---
{
"title": "DECIMAL",
"language": "zh-CN"
}
---

## 描述
    DECIMAL(M[,D])
    高精度定点数，M 代表一共有多少个有效数字(precision)，D 代表小数位有多少数字(scale)，
    有效数字 M 的范围是 [1, 38]，小数位数字数量 D 的范围是 [0, precision]。

    默认值为 DECIMAL(9, 0)。

### 精度推演

DECIMAL 有一套很复杂的类型推演规则，针对不同的表达式，会应用不同规则进行精度推断。

#### 四则运算

* 加法 / 减法：DECIMAL(a, b) + DECIMAL(x, y) -> DECIMAL(max(a - b, x - y) + max(b, y) + 1, max(b, y))。
* 乘法：DECIMAL(a, b) + DECIMAL(x, y) -> DECIMAL(a + x, b + y)。
* 除法：DECIMAL(p1, s1) + DECIMAL(p2, s2) -> DECIMAL(p1 + s2 + div_precision_increment, s1 + div_precision_increment)。div_precision_increment 默认为 4。
  值得注意的是，除法计算的过程是
  DECIMAL(p1, s1) / DECIMAL(p2, s2) 先转换成 DECIMAL(p1 + s2 + div_precision_increment, s1 + s2 ) /  DECIMAL(p2, s2)  然后再进行计算，所以可能会出现
  DECIMAL(p1 + s2 + div_precision_increment, s1 + div_precision_increment) 是满足 DECIMAL 的范围，但是由于先转换成了 DECIMAL(p1 + s2 + div_precision_increment, s1 + s2 )
  导致超出范围，目前 Doris 的处理是转成 Double 进行计算


#### 聚合运算

* SUM / MULTI_DISTINCT_SUM: SUM(DECIMAL(a, b)) -> DECIMAL(38, b).
* AVG: AVG(DECIMAL(a, b)) -> DECIMAL(38, max(b, 4)).

#### 默认规则

除上述提到的函数外，其余表达式都使用默认规则进行精度推演。即对于表达式 `expr(DECIMAL(a, b))`，结果类型同样也是 DECIMAL(a, b)。

#### 调整结果精度

不同用户对 DECIMAL 的精度要求各不相同，上述规则为当前 Doris 的默认行为，如果用户**有不同的精度需求，可以通过以下方式进行精度调整**：
1. 如果期望的结果精度大于默认精度，可以通过调整入参精度来调整结果精度。例如用户期望计算`AVG(col)`得到 DECIMAL(x, y) 作为结果，其中`col`的类型为 DECIMAL(a, b)，则可以改写表达式为`AVG(CAST(col as DECIMAL(x, y)))`。
2. 如果期望的结果精度小于默认精度，可以通过对输出结果求近似得到想要的精度。例如用户期望计算`AVG(col)`得到 DECIMAL(x, y) 作为结果，其中`col`的类型为 DECIMAL(a, b)，则可以改写表达式为`ROUND(AVG(col), y)`。

### 为什么需要 DECIMAL

Doris 中的 DECIMAL 是真正意义上的高精度定点数，Decimal 有以下核心优势：
1. 可表示范围更大。DECIMAL 中 precision 和 scale 的取值范围都进行了明显扩充。
2. 性能更高。老版本的 DECIMAL 在内存中需要占用 16 bytes，在存储中占用 12 bytes，而 DECIMAL 进行了自适应调整（如下表格）。
```
+----------------------+-------------------+
|     precision        | 占用空间（内存/磁盘）|
+----------------------+-------------------+
| 0 < precision <= 9   |      4 bytes      |
+----------------------+-------------------+
| 9 < precision <= 18  |      8 bytes      |
+----------------------+-------------------+
| 18 < precision <= 38 |     16 bytes      |
+----------------------+-------------------+
```
3. 更完备的精度推演。对于不同的表达式，应用不同的精度推演规则对结果的精度进行推演。

