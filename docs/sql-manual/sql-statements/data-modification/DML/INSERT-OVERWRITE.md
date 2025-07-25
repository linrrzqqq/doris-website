---
{
    "title": "INSERT OVERWRITE",
    "language": "en"
}


---

## Description

The function of this statement is to overwrite a table or some partitions of a table

```sql
INSERT OVERWRITE table table_name
    [ PARTITION (p1, ... | *) ]
    [ WITH LABEL label]
    [ (column [, ...]) ]
    [ [ hint [, ...] ] ]
    { VALUES ( { expression | DEFAULT } [, ...] ) [, ...] | query }
```

 Parameters

> table_name: the destination table to overwrite. This table must exist. It can be of the form `db_name.table_name`
>
> partitions: the table partitions that needs to be overwritten. The following two formats are supported
>
> > 1. partition names. must be one of the existing partitions in `table_name` separated by a comma
> > 2. asterisk(*)。Enable [auto-detect-partition](#overwrite-auto-detect-partition). The write operation will automatically detect the partitions involved in the data and overwrite those partitions. This format is supported since Apache Doris 2.1.3 version.
>
> label: specify a label for the Insert task
>
> column_name: the specified destination column must be one of the existing columns in `table_name`
>
> expression: the corresponding expression that needs to be assigned to a column
>
> DEFAULT: let the column use the default value
>
> query: a common query, the result of the query will overwrite the target.
>
> hint: some indicator used to indicate the execution behavior of `INSERT`. You can choose one of this values: `/*+ STREAMING */`, `/*+ SHUFFLE */` or `/*+ NOSHUFFLE */`.
>
> 1. STREAMING: At present, it has no practical effect and is only reserved for compatibility with previous versions. (In the previous version, adding this hint would return a label, but now it defaults to returning a label)
> 2. SHUFFLE: When the target table is a partition table, enabling this hint will do repartiiton.
> 3. NOSHUFFLE: Even if the target table is a partition table, repartiiton will not be performed, but some other operations will be performed to ensure that the data is correctly dropped into each partition.

Notice:

1. In the current version, the session variable `enable_insert_strict` is set to `true` by default. If some data that does not conform to the format of the target table is filtered out during the execution of the `INSERT OVERWRITE` statement, such as when overwriting a partition and not all partition conditions are satisfied, overwriting the target table will fail.
2. The `INSERT OVERWRITE` statement first creates a new table, inserts the data to be overwritten into the new table, and then atomically replaces the old table with the new table and modifies its name. Therefore, during the process of overwriting the table, the data in the old table can still be accessed normally until the overwriting is completed.

### For Auto Partition Table

If the target table of the INSERT OVERWRITE is an autopartitioned table, the behaviour is controlled by the [Session Variable](../../session/variable/SET-VARIABLE.md)  `enable_auto_create_when_overwrite` controls the behaviour as follows:

1. If PARTITION is not specified (overwrite the whole table), when `enable_auto_create_when_overwrite` is `true`, the table is overwritten and partitions are created according to the table's auto-partitioning rules for data that does not have a corresponding partition, and those datas is admit. If `enable_auto_create_when_overwrite` is `false`, data for which no partition is found will accumulate error rows until it fails.
2. If an overwrite PARTITION is specified, the AUTO PARTITION table behaves as a normal partitioned table during this process, and data that does not satisfy the conditions of an existing partition is filtered instead of creating a new partition.
3. If you specify PARTITION as `partition(*)` (auto detect partition and overwrite), when `enable_auto_create_when_overwrite` is `true`, for the data that have corresponding partitions in the table, overwrite their corresponding partitions, and leave the other existing partitions unchanged. At the same time, for data without corresponding partitions, create partitions according to the table's auto-partitioning rules, and accommodate the data without corresponding partitions. If `enable_auto_create_when_overwrite` is `false`, data for which no partition is found will accumulate error rows until it fails.

In versions without `enable_auto_create_when_overwrite`, the behaviour is as if the variable had a value of `false`.

The quick check conclusion is as follows:

1. For auto-partition tables that have `enable_auto_create_when_overwrite` enabled:

|    | Partitions to Overwrite | Clear Other Partitions | Auto Create for Unpartitioned Data |
|-|-|-|-|
| Unlabeled (Full Table) | All | √ | √ |
| Designated Partition | Explicit partition | × | × |
| `partition(*)` | The partition that the data belongs to | × | √ |

2. For trivial tables, auto-partition tables disabled `enable_auto_create_when_overwrite`:

|    | Partitions to Overwrite | Clear Other Partitions | Auto Create for Unpartitioned Data |
|-|-|-|-|
| Unlabeled (Full Table) | All | √ | × |
| Designated Partition | Explicit partition | × | × |
| `partition(*)` | The partition that the data belongs to | × | × |

Examples are shown below:

```sql
mysql> create table auto_list(
    ->              k0 varchar null
    ->          )
    ->          auto partition by list (k0)
    ->          (
    ->              PARTITION p1 values in (("Beijing"), ("BEIJING")),
    ->              PARTITION p2 values in (("Shanghai"), ("SHANGHAI")),
    ->              PARTITION p3 values in (("xxx"), ("XXX")),
    ->              PARTITION p4 values in (("list"), ("LIST")),
    ->              PARTITION p5 values in (("1234567"), ("7654321"))
    ->          )
    ->          DISTRIBUTED BY HASH(`k0`) BUCKETS 1
    ->          properties("replication_num" = "1");
Query OK, 0 rows affected (0.14 sec)

mysql> insert into auto_list values ("Beijing"),("Shanghai"),("xxx"),("list"),("1234567");
Query OK, 5 rows affected (0.22 sec)

mysql> insert overwrite table auto_list partition(*) values ("BEIJING"), ("new1");
Query OK, 2 rows affected (0.28 sec)

mysql> select * from auto_list;
+----------+ --- p1 is overwritten, new1 gets the new partition, and the other partitions remain unchanged.
| k0       |
+----------+
| 1234567  |
| BEIJING  |
| list     |
| xxx      |
| new1     |
| Shanghai |
+----------+
6 rows in set (0.48 sec)

mysql> insert overwrite table auto_list values ("SHANGHAI"), ("new2");
Query OK, 2 rows affected (0.17 sec)

mysql> select * from auto_list;
+----------+ --- The whole table is overwritten, and new2 gets the new partition.
| k0       |
+----------+
| new2     |
| SHANGHAI |
+----------+
2 rows in set (0.15 sec)
```

## Example

Assuming there is a table named `test`. The table contains two columns `c1` and `c2`, and two partitions `p1` and `p2`

```sql
CREATE TABLE IF NOT EXISTS test (
  `c1` int NOT NULL DEFAULT "1",
  `c2` int NOT NULL DEFAULT "4"
) ENGINE=OLAP
UNIQUE KEY(`c1`)
PARTITION BY LIST (`c1`)
(
PARTITION p1 VALUES IN ("1","2","3"),
PARTITION p2 VALUES IN ("4","5","6")
)
DISTRIBUTED BY HASH(`c1`) BUCKETS 3
PROPERTIES (
  "replication_allocation" = "tag.location.default: 1",
  "in_memory" = "false",
  "storage_format" = "V2"
);
```

### Overwrite Table

1. Overwrite the `test` table using the form of `VALUES`.

   ```sql
   // Single-row overwrite.
   INSERT OVERWRITE table test VALUES (1, 2);
   INSERT OVERWRITE table test (c1, c2) VALUES (1, 2);
   INSERT OVERWRITE table test (c1, c2) VALUES (1, DEFAULT);
   INSERT OVERWRITE table test (c1) VALUES (1);
   // Multi-row overwrite.
   INSERT OVERWRITE table test VALUES (1, 2), (3, 2 + 2);
   INSERT OVERWRITE table test (c1, c2) VALUES (1, 2), (3, 2 * 2);
   INSERT OVERWRITE table test (c1, c2) VALUES (1, DEFAULT), (3, DEFAULT);
   INSERT OVERWRITE table test (c1) VALUES (1), (3);
   ```

- The first and second statements have the same effect. If the target column is not specified during overwriting, the column order in the table will be used as the default target column. After the overwrite is successful, there is only one row of data in the `test` table.
- The third and fourth statements have the same effect. The unspecified column `c2` will be overwritten with the default value 4. After the overwrite is successful, there is only one row of data in the `test` table.
- The fifth and sixth statements have the same effect. Expressions (such as `2+2`, `2*2`) can be used in the statement. The result of the expression will be computed during the execution of the statement and then overwritten into the `test` table. After the overwrite is successful, there are two rows of data in the `test` table.
- The seventh and eighth statements have the same effect. The unspecified column `c2` will be overwritten with the default value 4. After the overwrite is successful, there are two rows of data in the `test` table.

2. Overwrite the `test` table in the form of a query statement. The data format of the `test2` table and the `test` table must be consistent. If they are not consistent, implicit data type conversion will be triggered.

   ```sql
   INSERT OVERWRITE table test SELECT * FROM test2;
   INSERT OVERWRITE table test (c1, c2) SELECT * from test2;
   ```

- The first and second statements have the same effect. The purpose of these statements is to take data from the `test2` table and overwrite the `test` table with the taken data. After the overwrite is successful, the data in the `test` table will be consistent with the data in the `test2` table.

3. Overwrite the `test` table and specify a label.

   ```sql
   INSERT OVERWRITE table test WITH LABEL `label1` SELECT * FROM test2;
   INSERT OVERWRITE table test WITH LABEL `label2` (c1, c2) SELECT * from test2;
   ```

- Users can use the `SHOW LOAD;` command to check the status of the job imported by this `label`. It should be noted that the label is unique.

### Overwrite Table Partition

When using INSERT OVERWRITE to rewrite partitions, we actually encapsulate the following three steps into a single transaction and execute it. If it fails halfway through, the operations that have been performed will be rolled back:

1. Assuming that partition `p1` is specified to be rewritten, first create an empty temporary partition `pTMP` with the same structure as the target partition to be rewritten.
2. Write data to `pTMP`.
3. replace `p1` with the `pTMP` atom

The following is examples:

1. Overwrite partitions `P1` and `P2` of the `test` table using the form of `VALUES`.

   ```sql
   // Single-row overwrite.
   INSERT OVERWRITE table test PARTITION(p1,p2) VALUES (1, 2);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1, c2) VALUES (1, 2);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1, c2) VALUES (1, DEFAULT);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1) VALUES (1);
   // Multi-row overwrite.
   INSERT OVERWRITE table test PARTITION(p1,p2) VALUES (1, 2), (4, 2 + 2);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1, c2) VALUES (1, 2), (4, 2 * 2);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1, c2) VALUES (1, DEFAULT), (4, DEFAULT);
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1) VALUES (1), (4);
   ```

   Unlike overwriting an entire table, the above statements are overwriting partitions in the table. Partitions can be overwritten one at a time or multiple partitions can be overwritten at once. It should be noted that only data that satisfies the corresponding partition filtering condition can be overwritten successfully. If there is data in the overwritten data that does not satisfy any of the partitions, the overwrite will fail. An example of a failure is shown below.

   ```sql
   INSERT OVERWRITE table test PARTITION(p1,p2) VALUES (7, 2);
   ```

   The data overwritten by the above statements (`c1=7`) does not satisfy the conditions of partitions `P1` and `P2`, so the overwrite will fail.

2. Overwrite partitions `P1` and `P2` of the `test` table in the form of a query statement. The data format of the `test2` table and the `test` table must be consistent. If they are not consistent, implicit data type conversion will be triggered.

   ```sql
   INSERT OVERWRITE table test PARTITION(p1,p2) SELECT * FROM test2;
   INSERT OVERWRITE table test PARTITION(p1,p2) (c1, c2) SELECT * from test2;
   ```

3. Overwrite partitions `P1` and `P2` of the `test` table and specify a label.

   ```sql
   INSERT OVERWRITE table test PARTITION(p1,p2) WITH LABEL `label3` SELECT * FROM test2;
   INSERT OVERWRITE table test PARTITION(p1,p2) WITH LABEL `label4` (c1, c2) SELECT * from test2;
   ```

### Overwrite Auto Detect Partition

> This feature is available since version 2.1.3.

When the PARTITION clause specified by the INSERT OVERWRITE command is `PARTITION(*)`, this overwrite will automatically detect the partition where the data is located. Example:

```sql
mysql> create table test(
    -> k0 int null
    -> )
    -> partition by range (k0)
    -> (
    -> PARTITION p10 values less than (10),
    -> PARTITION p100 values less than (100),
    -> PARTITION pMAX values less than (maxvalue)
    -> )
    -> DISTRIBUTED BY HASH(`k0`) BUCKETS 1
    -> properties("replication_num" = "1");
Query OK, 0 rows affected (0.11 sec)

mysql> insert into test values (1), (2), (15), (100), (200);
Query OK, 5 rows affected (0.29 sec)

mysql> select * from test order by k0;
+------+
| k0   |
+------+
|    1 |
|    2 |
|   15 |
|  100 |
|  200 |
+------+
5 rows in set (0.23 sec)

mysql> insert overwrite table test partition(*) values (3), (1234);
Query OK, 2 rows affected (0.24 sec)

mysql> select * from test order by k0;
+------+
| k0   |
+------+
|    3 |
|   15 |
| 1234 |
+------+
3 rows in set (0.20 sec)
```

As you can see, all data in partitions `p10` and `pMAX`, where data 3 and 1234 are located, are overwritten, while partition `p100` remains unchanged. This operation can be interpreted as syntactic sugar for specifying a specific partition to be overwritten by the PARTITION clause during an INSERT OVERWRITE operation, which is implemented in the same way as [specify a partition to overwrite](#overwrite-table-partition). The `PARTITION(*)` syntax eliminates the need to manually fill in all the partition names when overwriting a large number of partitions.

## Keywords

INSERT OVERWRITE, OVERWRITE, AUTO DETECT