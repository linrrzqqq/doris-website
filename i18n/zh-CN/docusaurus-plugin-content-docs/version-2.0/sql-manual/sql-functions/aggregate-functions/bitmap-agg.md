---
{
    "title": "BITMAP_AGG",
    "language": "zh-CN"
}
---

## BITMAP_AGG
## 描述
## 语法

`BITMAP_AGG(expr)`

聚合 expr 的值（不包括任何空值）得到 bitmap。
expr 的类型需要为 TINYINT,SMALLINT,INT 和 BIGINT 类型。

## 举例
```
MySQL > select `n_nationkey`, `n_name`, `n_regionkey` from `nation`;
+-------------+----------------+-------------+
| n_nationkey | n_name         | n_regionkey |
+-------------+----------------+-------------+
|           0 | ALGERIA        |           0 |
|           1 | ARGENTINA      |           1 |
|           2 | BRAZIL         |           1 |
|           3 | CANADA         |           1 |
|           4 | EGYPT          |           4 |
|           5 | ETHIOPIA       |           0 |
|           6 | FRANCE         |           3 |
|           7 | GERMANY        |           3 |
|           8 | INDIA          |           2 |
|           9 | INDONESIA      |           2 |
|          10 | IRAN           |           4 |
|          11 | IRAQ           |           4 |
|          12 | JAPAN          |           2 |
|          13 | JORDAN         |           4 |
|          14 | KENYA          |           0 |
|          15 | MOROCCO        |           0 |
|          16 | MOZAMBIQUE     |           0 |
|          17 | PERU           |           1 |
|          18 | CHINA          |           2 |
|          19 | ROMANIA        |           3 |
|          20 | SAUDI ARABIA   |           4 |
|          21 | VIETNAM        |           2 |
|          22 | RUSSIA         |           3 |
|          23 | UNITED KINGDOM |           3 |
|          24 | UNITED STATES  |           1 |
+-------------+----------------+-------------+

MySQL > select n_regionkey, bitmap_to_string(bitmap_agg(n_nationkey)) from nation group by n_regionkey;
+-------------+---------------------------------------------+
| n_regionkey | bitmap_to_string(bitmap_agg(`n_nationkey`)) |
+-------------+---------------------------------------------+
|           4 | 4,10,11,13,20                               |
|           2 | 8,9,12,18,21                                |
|           1 | 1,2,3,17,24                                 |
|           0 | 0,5,14,15,16                                |
|           3 | 6,7,19,22,23                                |
+-------------+---------------------------------------------+

MySQL > select bitmap_count(bitmap_agg(n_nationkey))  from nation;
+-----------------------------------------+
| bitmap_count(bitmap_agg(`n_nationkey`)) |
+-----------------------------------------+
|                                      25 |
+-----------------------------------------+
```
### keywords
BITMAP_AGG
