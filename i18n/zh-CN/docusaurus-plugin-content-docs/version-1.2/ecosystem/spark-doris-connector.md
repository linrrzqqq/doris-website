---
{
  "title": "Spark Doris Connector",
  "language": "zh-CN"
}
---

# Spark Doris Connector

Spark Doris Connector 可以支持通过 Spark 读取 Doris 中存储的数据，也支持通过 Spark 写入数据到 Doris。

代码库地址：https://github.com/apache/doris-spark-connector

- 支持从`Doris`中读取数据
- 支持`Spark DataFrame`批量/流式 写入`Doris`
- 可以将`Doris`表映射为`DataFrame`或者`RDD`，推荐使用`DataFrame`。
- 支持在`Doris`端完成数据过滤，减少数据传输量。

## 版本兼容

| Connector | Spark               | Doris       | Java | Scala      |
|-----------|---------------------|-------------|------|------------|
| 1.3.1     | 3.4 ~ 3.1, 2.4, 2.3 | 1.0 +       | 8    | 2.12, 2.11 |
| 1.2.0     | 3.2, 3.1, 2.3       | 1.0 +       | 8    | 2.12, 2.11 |
| 1.1.0     | 3.2, 3.1, 2.3       | 1.0 +       | 8    | 2.12, 2.11 |
| 1.0.1     | 3.1, 2.3            | 0.12 - 0.15 | 8    | 2.12, 2.11 |

## 使用

### Maven
```
<dependency>
    <groupId>org.apache.doris</groupId>
    <artifactId>spark-doris-connector-3.4_2.12</artifactId>
    <version>1.3.2</version>
</dependency>
```

**备注**

1. 请根据不同的 Spark 和 Scala 版本替换相应的 Connector 版本。

2. 也可从[这里](https://repo.maven.apache.org/maven2/org/apache/doris/)下载相关版本 jar 包。

### 编译

编译时，可直接运行 `sh build.sh`，具体可参考这里。

编译成功后，会在 `dist` 目录生成目标 jar 包，如：spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar。 将此文件复制到 `Spark` 的 `ClassPath` 中即可使用 `Spark-Doris-Connector`。 例如，`Local` 模式运行的 `Spark`，将此文件放入 `jars/` 文件夹下。`Yarn`集群模式运行的`Spark`，则将此文件放入预部署包中。
也可以


2. 在源码目录下执行：
   `sh build.sh`
   根据提示输入你需要的 Scala 与 Spark 版本进行编译。

编译成功后，会在 `dist` 目录生成目标 jar 包，如：`spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar`。
将此文件复制到 `Spark` 的 `ClassPath` 中即可使用 `Spark-Doris-Connector`。

例如，`Local` 模式运行的 `Spark`，将此文件放入 `jars/` 文件夹下。`Yarn`集群模式运行的`Spark`，则将此文件放入预部署包中。

例如将 `spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar` 上传到 hdfs 并在 `spark.yarn.jars` 参数上添加 hdfs 上的 Jar包路径
```shell
1. 上传 `spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar` 到 hdfs。

hdfs dfs -mkdir /spark-jars/
hdfs dfs -put /your_local_path/spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar /spark-jars/

2. 在集群中添加 `spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar` 依赖。
spark.yarn.jars=hdfs:///spark-jars/spark-doris-connector-3.2_2.12-1.2.0-SNAPSHOT.jar

```

## 使用示例

### 读取

#### SQL

```sql
CREATE
TEMPORARY VIEW spark_doris
USING doris
OPTIONS(
  "table.identifier"="$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME",
  "fenodes"="$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT",
  "user"="$YOUR_DORIS_USERNAME",
  "password"="$YOUR_DORIS_PASSWORD"
);

SELECT *
FROM spark_doris;
```

#### DataFrame

```java
val dorisSparkDF = spark.read.format("doris")
  .option("doris.table.identifier", "$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME")
  .option("doris.fenodes", "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
  .option("user", "$YOUR_DORIS_USERNAME")
  .option("password", "$YOUR_DORIS_PASSWORD")
  .load()

dorisSparkDF.show(5)
```

#### RDD

```java
import org.apache.doris.spark._

val dorisSparkRDD = sc.dorisRDD(
  tableIdentifier = Some("$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME"),
  cfg = Some(Map(
    "doris.fenodes" -> "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT",
    "doris.request.auth.user" -> "$YOUR_DORIS_USERNAME",
    "doris.request.auth.password" -> "$YOUR_DORIS_PASSWORD"
  ))
)

dorisSparkRDD.collect()
```

#### pySpark

```java
dorisSparkDF = spark.read.format("doris")
  .option("doris.table.identifier", "$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME")
  .option("doris.fenodes", "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
  .option("user", "$YOUR_DORIS_USERNAME")
  .option("password", "$YOUR_DORIS_PASSWORD")
  .load()
// show 5 lines data 
dorisSparkDF.show(5)
```

### 写入

#### SQL

```sql
CREATE
TEMPORARY VIEW spark_doris
USING doris
OPTIONS(
  "table.identifier"="$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME",
  "fenodes"="$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT",
  "user"="$YOUR_DORIS_USERNAME",
  "password"="$YOUR_DORIS_PASSWORD"
);

INSERT INTO spark_doris
VALUES ("VALUE1", "VALUE2", ...);
# or
INSERT INTO spark_doris
SELECT *
FROM YOUR_TABLE
# or
INSERT OVERWRITE 
SELECT *
FROM YOUR_TABLE 
```

#### DataFrame(batch/stream)

```java
// batch sink
val mockDataDF = List(
  (3, "440403001005", "21.cn"),
  (1, "4404030013005", "22.cn"),
  (33, null, "23.cn")
).toDF("id", "mi_code", "mi_name")
mockDataDF.show(5)

mockDataDF.write.format("doris")
  .option("doris.table.identifier", "$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME")
  .option("doris.fenodes", "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
  .option("user", "$YOUR_DORIS_USERNAME")
  .option("password", "$YOUR_DORIS_PASSWORD")
  //其它选项
  //指定你要写入的字段
  .option("doris.write.fields", "$YOUR_FIELDS_TO_WRITE")
  // 支持设置 Overwrite 模式来覆盖数据
  // .option("save_mode", SaveMode.Overwrite)
  .save()

// stream sink(StructuredStreaming)

// 结果 DataFrame 和 doris 表相同的结构化数据, 配置方式和批量模式一致。
val sourceDf = spark.readStream.
       .format("your_own_stream_source")
       .load()

val resultDf = sourceDf.<transformations>

resultDf.writeStream
      .format("doris")
      .option("checkpointLocation", "$YOUR_CHECKPOINT_LOCATION")
      .option("doris.table.identifier", "$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME")
      .option("doris.fenodes", "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
      .option("user", "$YOUR_DORIS_USERNAME")
      .option("password", "$YOUR_DORIS_PASSWORD")
      .start()
      .awaitTermination()

// 结果 DataFrame 中存在某一列的数据可以直接写入的，比如符合导入规范的 Kafka 消息中的 value 值

val kafkaSource = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "$YOUR_KAFKA_SERVERS")
  .option("startingOffsets", "latest")
  .option("subscribe", "$YOUR_KAFKA_TOPICS")
  .load()
kafkaSource.selectExpr("CAST(value as STRING)")
  .writeStream
  .format("doris")
  .option("checkpointLocation", "$YOUR_CHECKPOINT_LOCATION")
  .option("doris.table.identifier", "$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME")
  .option("doris.fenodes", "$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
  .option("user", "$YOUR_DORIS_USERNAME")
  .option("password", "$YOUR_DORIS_PASSWORD")
  // 设置该选项可以将 Kafka 消息中的 value 列不经过处理直接写入
  .option("doris.sink.streaming.passthrough", "true")
  .option("doris.sink.properties.format", "json")
  // 其他选项
  .start()
  .awaitTermination()
```

### Java 示例

`samples/doris-demo/spark-demo/` 下提供了 Java
版本的示例，可供参考，[这里](https://github.com/apache/incubator-doris/tree/master/samples/doris-demo/spark-demo)

## 配置

### 通用配置项

| Key                                   | Default Value           | Comment                                                                                             |
|---------------------------------------|-------------------------|-----------------------------------------------------------------------------------------------------|
| doris.fenodes                         | --                | Doris FE http 地址，支持多个地址，使用逗号分隔                                                                                                                                                                                                                                          |
| doris.table.identifier                | --                | Doris 表名，如：db1.tbl1                                                                                                                                                                                                                                                     |
| doris.request.retries                 | 3                 | 向 Doris 发送请求的重试次数                                                                                                                                                                                                                                                         |
| doris.request.connect.timeout.ms      | 30000             | 向 Doris 发送请求的连接超时时间                                                                                                                                                                                                                                                       |
| doris.request.read.timeout.ms         | 30000             | 向 Doris 发送请求的读取超时时间                                                                                                                                                                                                                                                       |
| doris.request.query.timeout.s         | 3600              | 查询 doris 的超时时间，默认值为 1 小时，-1 表示无超时限制                                                                                                                                                                                                                                          |
| doris.request.tablet.size             | Integer.MAX_VALUE | 一个 RDD Partition 对应的 Doris Tablet 个数。<br />此数值设置越小，则会生成越多的 Partition。从而提升 Spark 侧的并行度，但同时会对 Doris 造成更大的压力。                                                                                                                                                                       |
| doris.read.field                      | --                | 读取 Doris 表的列名列表，多列之间使用逗号分隔                                                                                                                                                                                                                                                |
| doris.batch.size                      | 4064              | 一次从 BE 读取数据的最大行数。增大此数值可减少 Spark 与 Doris 之间建立连接的次数。<br />从而减轻网络延迟所带来的额外时间开销。                                                                                                                                                                                                   |
| doris.exec.mem.limit                  | 2147483648        | 单个查询的内存限制。默认为 2GB，单位为字节                                                                                                                                                                                                                                                 |
| doris.deserialize.arrow.async         | false             | 是否支持异步转换 Arrow 格式到 spark-doris-connector 迭代所需的 RowBatch                                                                                                                                                                                                                      |
| doris.deserialize.queue.size          | 64                | 异步转换 Arrow 格式的内部处理队列，当 doris.deserialize.arrow.async 为 true 时生效                                                                                                                                                                                                               |
| doris.write.fields                    | --                | 指定写入 Doris 表的字段或者字段顺序，多列之间使用逗号分隔。<br />默认写入时要按照 Doris 表字段顺序写入全部字段。                                                                                                                                                                                                          |
| doris.sink.batch.size                 | 100000            | 单次写 BE 的最大行数                                                                                                                                                                                                                                                              |
| doris.sink.max-retries                | 0                 | 写 BE 失败之后的重试次数，从 1.3.0 版本开始， 默认值为 0，即默认不进行重试。当设置该参数大于 0 时，会进行批次级别的失败重试，会在 Spark Executor 内存中缓存 `doris.sink.batch.size` 所配置大小的数据，可能需要适当增大内存分配。                                                    |       
| doris.sink.properties.format          | csv               | Stream Load 的数据格式。<br/>共支持 3 种格式：csv，json，arrow <br/> [更多参数详情](https://doris.apache.org/zh-CN/docs/data-operate/import/stream-load-manual/) |
| doris.sink.properties.*               | --                | Stream Load 的导入参数。<br/>例如:<br/>指定列分隔符：`'doris.sink.properties.column_separator' = ','`等<br/> [更多参数详情](https://doris.apache.org/zh-CN/docs/data-operate/import/stream-load-manual/) |
| doris.sink.task.partition.size        | --                | Doris 写入任务对应的 Partition 个数。Spark RDD 经过过滤等操作，最后写入的 Partition 数可能会比较大，但每个 Partition 对应的记录数比较少，导致写入频率增加和计算资源浪费。<br/>此数值设置越小，可以降低 Doris 写入频率，减少 Doris 合并压力。该参数配合 doris.sink.task.use.repartition 使用。                                                                        |
| doris.sink.task.use.repartition       | false             | 是否采用 repartition 方式控制 Doris 写入 Partition 数。默认值为 false，采用 coalesce 方式控制（注意：如果在写入之前没有 Spark action 算子，可能会导致整个计算并行度降低）。<br/>如果设置为 true，则采用 repartition 方式（注意：可设置最后 Partition 数，但会额外增加 shuffle 开销）。                                                                         |
| doris.sink.batch.interval.ms          | 50                | 每个批次 sink 的间隔时间，单位 ms。                                                                                                                                                                                                                                                    |
| doris.sink.enable-2pc                 | false             | 是否开启两阶段提交。开启后将会在作业结束时提交事务，而部分任务失败时会将所有预提交状态的事务会滚。                                                                                                                                                |
| doris.sink.auto-redirect              | true              | 是否重定向 StreamLoad 请求。开启后 StreamLoad 将通过 FE 写入，不再显式获取 BE 信息。                                                                                                                                                                                                                         |

### SQL 和 Dataframe 专有配置

| Key                             | Default Value | Comment                                                                |
|---------------------------------|---------------|------------------------------------------------------------------------|
| user                            | --            | 访问 Doris 的用户名                                                            |
| password                        | --            | 访问 Doris 的密码                                                             |
| doris.filter.query.in.max.count | 100           | 谓词下推中，in 表达式 value 列表元素最大数量。超过此数量，则 in 表达式条件过滤在 Spark 侧处理。                    |
| doris.ignore-type               | --            | 指在定临时视图中，读取 schema 时要忽略的字段类型。<br/> 例如，'doris.ignore-type'='bitmap,hll' |

### Structured Streaming 专有配置

| Key                              | Default Value | Comment                                                          |
| -------------------------------- | ------------- | ---------------------------------------------------------------- |
| doris.sink.streaming.passthrough | false         | 将第一列的值不经过处理直接写入。                                      |

### RDD 专有配置

| Key                         | Default Value | Comment                                      |
|-----------------------------|---------------|----------------------------------------------|
| doris.request.auth.user     | --            | 访问 Doris 的用户名                                  |
| doris.request.auth.password | --            | 访问 Doris 的密码                                   |
| doris.filter.query          | --            | 过滤读取数据的表达式，此表达式透传给 Doris。Doris 使用此表达式完成源端数据过滤。 |


## Doris 和 Spark 列类型映射关系

| Doris Type | Spark Type                       |
|------------|----------------------------------|
| NULL_TYPE  | DataTypes.NullType               |
| BOOLEAN    | DataTypes.BooleanType            |
| TINYINT    | DataTypes.ByteType               |
| SMALLINT   | DataTypes.ShortType              |
| INT        | DataTypes.IntegerType            |
| BIGINT     | DataTypes.LongType               |
| FLOAT      | DataTypes.FloatType              |
| DOUBLE     | DataTypes.DoubleType             |
| DATE       | DataTypes.DateType               |
| DATETIME   | DataTypes.StringType<sup>1</sup> |
| DECIMAL    | DecimalType                      |
| CHAR       | DataTypes.StringType             |
| LARGEINT   | DecimalType                      |
| VARCHAR    | DataTypes.StringType             |
| TIME       | DataTypes.DoubleType             |
| HLL        | Unsupported datatype             |
| Bitmap     | Unsupported datatype             |

* 注：Connector 中，将`DATETIME`映射为`String`。由于`Doris`底层存储引擎处理逻辑，直接使用时间类型时，覆盖的时间范围无法满足需求。所以使用 `String` 类型直接返回对应的时间可读文本。


## 常见问题
1. **如何写入 Bitmap 类型？**

在 Spark SQL 中，通过 insert into 方式写入数据时，如果 doris 的目标表中包含 `BITMAP` 或 `HLL` 类型的数据时，需要设置参数 `doris.ignore-type` 为对应类型，并通过 `doris.write.fields` 对列进行映射转换，使用方式如下：
> BITMAP
> ```sql
> CREATE TEMPORARY VIEW spark_doris
> USING doris
> OPTIONS(
> "table.identifier"="$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME",
> "fenodes"="$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT",
> "user"="$YOUR_DORIS_USERNAME",
> "password"="$YOUR_DORIS_PASSWORD"
> "doris.ignore-type"="bitmap",
> "doris.write.fields"="col1,col2,col3,bitmap_col2=to_bitmap(col2),bitmap_col3=bitmap_hash(col3)"
> );
> ```
> HLL
> ```sql
> CREATE TEMPORARY VIEW spark_doris
> USING doris
> OPTIONS(
> "table.identifier"="$YOUR_DORIS_DATABASE_NAME.$YOUR_DORIS_TABLE_NAME",
> "fenodes"="$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT",
> "user"="$YOUR_DORIS_USERNAME",
> "password"="$YOUR_DORIS_PASSWORD"
> "doris.ignore-type"="hll",
> "doris.write.fields"="col1,hll_col1=hll_hash(col1)"
> );
> ```

2. **如何使用overwrite写入？**

从 1.3.0 版本开始，支持 overwrite 模式写入（只支持全表级别的数据覆盖），具体使用方式如下
> DataFrame
> ```java
> resultDf.format("doris")
>   .option("doris.fenodes","$YOUR_DORIS_FE_HOSTNAME:$YOUR_DORIS_FE_RESFUL_PORT")
>   // your own options
>   .option("save_mode", SaveMode.Overwrite)
>   .save()
> ```
>
> SQL
> ```sql
> INSERT OVERWRITE your_target_table
> SELECT * FROM your_source_table
> ```


