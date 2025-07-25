---
{
    "title": "自动分桶",
    "language": "zh-CN"
}
---

用户经常设置不合适的 bucket，导致各种问题，这里提供一种方式，来自动设置分桶数。当前只对 OLAP 表生效。

:::caution
注意：这个功能在被 CCR 同步时将会失效。如果这个表是被 CCR 复制而来的，即 PROPERTIES 中包含`is_being_synced = true`时，在`show create table`中会显示开启状态，但不会实际生效。当`is_being_synced`被设置为 `false` 时，这些功能将会恢复生效，但`is_being_synced`属性仅供 CCR 外围模块使用，在 CCR 同步的过程中不要手动设置。
:::

以往创建分桶时需要手动设定分桶数，而自动分桶推算功能是 Apache Doris 可以动态地推算分桶个数，使得分桶数始终保持在一个合适范围内，让用户不再操心桶数的细枝末节。首先说明一点，为了方便阐述该功能，该部分会将桶拆分为两个时期的桶，即初始分桶以及后续分桶；这里的初始和后续只是本文为了描述清楚该功能而采用的术语，Apache Doris 分桶本身没有初始和后续之分。从上文中创建分桶一节我们知道，BUCKET_DESC 非常简单，但是需要指定分桶个数；而在自动分桶推算功能上，BUCKET_DESC 的语法直接将分桶数改成"Auto"，并新增一个 Properties 配置即可：

```sql
-- 旧版本指定分桶个数的创建语法
DISTRIBUTED BY HASH(site) BUCKETS 20

-- 新版本使用自动分桶推算的创建语法
DISTRIBUTED BY HASH(site) BUCKETS AUTO
properties("estimate_partition_size" = "2G")
```

新增的配置参数 estimate_partition_size 用于指定单个分区的数据量。该参数为可选项，若未设置，Doris 会默认将 estimate_partition_size 取值为 10GB。如前文所述，在物理层面上，一个分桶对应一个 Tablet。为了获得最佳性能，建议每个 Tablet 的大小控制在 1GB 至 15GB 之间。

那么自动分桶推算是如何保证 Tablet 大小处于这个范围内的呢？

- 若是整体数据量较小，则分桶数不要设置过多

- 若是整体数据量较大，则应使桶数跟总的磁盘块数相关，充分利用每台 BE 机器和每块磁盘的能力 

## 初始分桶推算 
1.  首先根据数据量计算桶数 N。具体做法是将 estimate_partition_size 的数值除以 5（因为以文本格式存储时，Doris 通常有 5:1 的数据压缩比），然后根据分区大小估算桶数。在存算一体架构下，默认每 5GB 分区大小对应一个分桶；在存算分离架构下，默认每 10GB 分区大小对应一个分桶。该默认值可通过 FE 配置项 autobucket_partition_size_per_bucket_gb 进行调整。最终得到的结果如下：

  ```Plain
  (, 100MB)，则取 N=1
  [100MB, 1GB)，则取 N=2
  [1GB, )，则每(autobucket_partition_size_per_bucket_gb) GB 一个分桶
  ```

2. 根据 BE 节点数以及每个 BE 节点的磁盘容量，计算出桶数 M。

  ```Plain
  其中每个 BE 节点算 1，每 50G 的磁盘容量算 1，
  M 的计算规则为：M = BE 节点数 * ( 一块磁盘块大小 / 50GB) *磁盘块数 
  举例：有 3 台 BE，每台 BE 都有 4 块 500GB 的磁盘，那么 M = 3 (500GB / 50GB) 4 = 120
  ```

3. 得到最终的分桶个数计算逻辑： 

  ```Plain
  先计算一个中间值 x = min(M, N, 128)， 
  如果 x < N 并且 x < BE 节点个数，则最终分桶为 y 即 BE 节点个数；
  否则最终分桶数为 x
  ```

4. x = max(x, autobucket_min_buckets), 这里 autobucket_min_buckets 是在 Config 中配置的，默认是 1

  上述过程伪代码表现形式为：

  ```Plain
  int N = 计算 N 值;
  int M = 计算 M 值;

  int y = BE 节点个数;
  int x = min(M, N, 128);

  if (x < N && x < y) {
    return y;
  }
  return x;
  ```

5. 示例：有了上述算法，咱们再引入一些例子来更好地理解这部分逻辑。

  ```Plain
  case1:
  数据量 100 MB，10 台 BE 机器，2TB *3 块盘
  数据量 N = 1
  BE 磁盘 M = 10* (2TB/50GB) * 3 = 1230
  x = min(M, N, 128) =  1
  最终：1

  case2:
  数据量 1GB, 3 台 BE 机器，500GB *2 块盘盘
  数据量 N = 2
  BE 磁盘 M = 3* (500GB/50GB) * 2 = 60
  x = min(M, N, 128) =  2
  最终：2

  case3:
  数据量 100GB，3 台 BE 机器，500GB *2 块盘
  数据量 N = 20
  BE 磁盘 M = 3* (500GB/50GB) * 2 = 60
  x = min(M, N, 128) =  20
  最终：20

  case4:
  数据量 500GB，3 台 BE 机器，1TB *1 块盘
  数据量 N = 100
  BE 磁盘 M = 3* (1TB /50GB) * 1 = 6060
  x = min(M, N, 128) =  63
  最终：63

  case5:
  数据量 500GB，10 台 BE 机器，2TB *3 块盘*3 块盘
  数据量 N =  100
  BE 磁盘 M = 10* (2TB / 50GB) * 3 = 1230
  x = min(M, N, 128) =  100
  最终：100

  case 6:
  数据量 1TB，10 台 BE 机器，2TB *3 块盘
  数据量 N =  205
  BE 磁盘 M = 10* (2TB / 50GB) * 3 = 1230
  x = min(M, N, 128) =  128
  最终：128

  case 7:
  数据量 500GB，1 台 BE 机器，100TB *1 块盘
  数据量 N = 100
  BE 磁盘 M =  1* (100TB / 50GB) * 1 = 2048
  x = min(M, N, 128) =  100
  最终：100

  case 8:
  数据量 1TB, 200 台 BE 机器，4TB *7 块盘
  数据量 N = 205
  BE 磁盘 M = 200* (4TB / 50GB) * 7 = 114800
  x = min(M, N, 128) =  128
  最终：200
  ```

## 后续分桶推算 

上述是关于初始分桶的计算逻辑，后续分桶数因为已经有了一定的分区数据，可以根据已有的分区数据量来进行评估。后续分桶数会根据最多前 7 个分区数据量的 EMA（短期指数移动平均线）值，作为 estimate_partition_size 进行评估。此时计算分桶有两种计算方式，假设以天来分区，往前数第一天分区大小为 S7，往前数第二天分区大小为 S6，依次类推到 S1。

- 如果 7 天内的分区数据每日严格递增，则此时会取趋势值

  有 6 个 delta 值，分别是

  ```Plain
  S7 - S6 = delta1,
  S6 - S5 = delta2,
  ...
  S2 - S1 = delta6
  ```

  由此得到 ema(delta) 值：那么，今天的 estimate_partition_size = S7 + ema(delta)。

- 非第一种的情况，此时直接取前几天的 EMA 平均值

  今天的 estimate_partition_size = EMA(S1, ..., S7)。

## 说明

根据上述算法，初始分桶个数以及后续分桶个数都能被计算出来。跟之前只能指定固定分桶数不同，由于业务数据的变化，有可能前面分区的分桶数和后面分区的分桶数不一样，这对用户是透明的，用户无需关心每一分区具体的分桶数是多少，而这一自动推算的功能会让分桶数更加合理。

开启 autobucket 之后，在`show create table`的时候看到的 schema 也是`BUCKETS AUTO`.如果想要查看确切的 bucket 数，可以通过`show partitions from ${table};`来查看。
