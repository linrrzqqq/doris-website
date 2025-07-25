---
{
    "title": "CREATE-TABLE",
    "language": "en",
    "toc_min_heading_level": 2,
    "toc_max_heading_level": 4
}
---

## CREATE-TABLE

### Description

This command is used to create a table. The subject of this document describes the syntax for creating Doris self-maintained tables. For external table syntax, please refer to the [CREATE-EXTERNAL-TABLE](./CREATE-EXTERNAL-TABLE.md) document.

```sql
CREATE TABLE [IF NOT EXISTS] [database.]table
(
    column_definition_list
    [, index_definition_list]
)
[engine_type]
[keys_type]
[table_comment]
[partition_info]
distribution_desc
[rollup_list]
[properties]
[extra_properties]
```

#### column_definition_list

Column definition list:

`column_definition[, column_definition]`

* `column_definition`

    Column definition:

    `column_name column_type [KEY] [aggr_type] [NULL] [default_value] [column_comment]`

    * `column_type`

        Column type, the following types are supported:

        ```
        TINYINT (1 byte)
            Range: -2^7 + 1 ~ 2^7-1
        SMALLINT (2 bytes)
            Range: -2^15 + 1 ~ 2^15-1
        INT (4 bytes)
            Range: -2^31 + 1 ~ 2^31-1
        BIGINT (8 bytes)
            Range: -2^63 + 1 ~ 2^63-1
        LARGEINT (16 bytes)
            Range: -2^127 + 1 ~ 2^127-1
        FLOAT (4 bytes)
            Support scientific notation
        DOUBLE (12 bytes)
            Support scientific notation
        DECIMAL[(precision, scale)] (16 bytes)
            The decimal type with guaranteed precision. The default is DECIMAL(10, 0)
            precision: 1 ~ 27
            scale: 0 ~ 9
            Where the integer part is 1 ~ 18
            Does not support scientific notation
        DATE (3 bytes)
            Range: 0000-01-01 ~ 9999-12-31
        DATETIME (8 bytes)
            Range: 0000-01-01 00:00:00 ~ 9999-12-31 23:59:59
        CHAR[(length)]
            Fixed-length character string. Length range: 1 ~ 255. Default is 1
        VARCHAR[(length)]
            Variable length character string. Length range: 1 ~ 65533. Default is 65533
        HLL (1~16385 bytes)
            HyperLogLog column type, do not need to specify the length and default value. The length is controlled within the system according to the degree of data aggregation.
            Must be used with HLL_UNION aggregation type.
        BITMAP
            The bitmap column type does not need to specify the length and default value. Represents a collection of integers, and the maximum number of elements supported is 2^64-1.
            Must be used with BITMAP_UNION aggregation type.
        ```

    * `aggr_type`

        Aggregation type, the following aggregation types are supported:

        ```
        SUM: Sum. Applicable numeric types.
        MIN: Find the minimum value. Suitable for numeric types.
        MAX: Find the maximum value. Suitable for numeric types.
        REPLACE: Replace. For rows with the same dimension column, the index column will be imported in the order of import, and the last imported will replace the first imported.
        REPLACE_IF_NOT_NULL: non-null value replacement. The difference with REPLACE is that there is no replacement for null values. It should be noted here that the default value should be NULL, not an empty string. If it is an empty string, you should replace it with an empty string.
        HLL_UNION: The aggregation method of HLL type columns, aggregated by HyperLogLog algorithm.
        BITMAP_UNION: The aggregation mode of BIMTAP type columns, which performs the union aggregation of bitmaps.
        ```

    * `default_value`

        Default value of the column. If the load data does not specify a value for this column, the system will assign a default value to this column.
        
        The syntax is: `default default_value`。
        
        Currently, the default value supports two forms:

        1. The user specifies a fixed value, such as:

        ```SQL
        	k1 INT DEFAULT '1',
            k2 CHAR(10) DEFAULT 'aaaa'
        ```
        2. Keywords are provided by the system. Currently, the following keywords are supported: 
        
        ```SQL
            // This keyword is used only for DATETIME type. If the value is missing, the system assigns the current timestamp.
            dt DATETIME DEFAULT CURRENT_TIMESTAMP
        ```

    Example:

        ```
        k1 TINYINT,
        k2 DECIMAL(10,2) DEFAULT "10.5",
        k4 BIGINT NULL DEFAULT "1000" COMMENT "This is column k4",
        v1 VARCHAR(10) REPLACE NOT NULL,
        v2 BITMAP BITMAP_UNION,
        v3 HLL HLL_UNION,
        v4 INT SUM NOT NULL DEFAULT "1" COMMENT "This is column v4"
        ```

#### index_definition_list

Index list definition:

`index_definition[, index_definition]`

* `index_definition`

    Index definition:

    ```sql
    INDEX index_name (col_name) [USING INVERTED] COMMENT'xxxxxx'
    ```

    Example:

    ```sql
    INDEX idx1 (k1) USING INVERTED COMMENT "This is a inverted index1",
    INDEX idx2 (k2) USING INVERTED COMMENT "This is a inverted index2",
    ...
    ```

#### engine_type

Table engine type. All types in this document are OLAP. For other external table engine types, see [CREATE EXTERNAL TABLE](./CREATE-EXTERNAL-TABLE.md) document. Example:

    `ENGINE=olap`

#### keys_type

Data model.

`key_type(col1, col2, ...)`

`key_type` supports the following models:

* DUPLICATE KEY (default): The subsequent specified column is the sorting column.
* AGGREGATE KEY: The specified column is the dimension column.
* UNIQUE KEY: The subsequent specified column is the primary key column.

NOTE: when set table property `"enable_duplicate_without_keys_by_default" = "true"`, will create a duplicate model without sorting columns and prefix indexes by default.

Example:

```
DUPLICATE KEY(col1, col2),
AGGREGATE KEY(k1, k2, k3),
UNIQUE KEY(k1, k2)
```

#### table_comment

Table notes. Example:

    ```
    COMMENT "This is my first DORIS table"
    ```

#### partition_info

Partition information supports three writing methods:

1. LESS THAN: Only define the upper boundary of the partition. The lower bound is determined by the upper bound of the previous partition.

    ```
    PARTITION BY RANGE(col1[, col2, ...])
    (
        PARTITION partition_name1 VALUES LESS THAN MAXVALUE|("value1", "value2", ...),
        PARTITION partition_name2 VALUES LESS THAN MAXVALUE|("value1", "value2", ...)
    )
    ```

2. FIXED RANGE: Define the left closed and right open interval of the zone.

    ```
    PARTITION BY RANGE(col1[, col2, ...])
    (
        PARTITION partition_name1 VALUES [("k1-lower1", "k2-lower1", "k3-lower1",...), ("k1-upper1", "k2-upper1", "k3-upper1", ... )),
        PARTITION partition_name2 VALUES [("k1-lower1-2", "k2-lower1-2", ...), ("k1-upper1-2", MAXVALUE, ))
    )
    ```
           
:::tip Tips
This feature is supported since the Apache Doris 1.2 version
:::
    
3. MULTI RANGE: Multi build RANGE partitions,Define the left closed and right open interval of the zone, Set the time unit and step size, the time unit supports year, month, day, week and hour.

    ```
    PARTITION BY RANGE(col)
    (
       FROM ("2000-11-14") TO ("2021-11-14") INTERVAL 1 YEAR,
       FROM ("2021-11-14") TO ("2022-11-14") INTERVAL 1 MONTH,
       FROM ("2022-11-14") TO ("2023-01-03") INTERVAL 1 WEEK,
       FROM ("2023-01-03") TO ("2023-01-14") INTERVAL 1 DAY
    )
    ```
    


4. MULTI RANGE: Multi build integer RANGE partitions,Define the left closed and right open interval of the zone, and step size.

    ```
    PARTITION BY RANGE(int_col)
    (
        FROM (1) TO (100) INTERVAL 10
    )
    ```
    
#### distribution_desc

Define the data bucketing method.

1. Hash
   Syntax:
   `DISTRIBUTED BY HASH (k1[,k2 ...]) [BUCKETS num|auto]`
   Explain:
   Hash bucketing using the specified key column. 

2. Random
   Syntax:
   `DISTRIBUTED BY RANDOM [BUCKETS num|auto]`
   Explain:
   Use random numbers for bucketing.

#### rollup_list

Multiple materialized views (ROLLUP) can be created at the same time as the table is built.

`ROLLUP (rollup_definition[, rollup_definition, ...])`

* `rollup_definition`

    `rollup_name (col1[, col2, ...]) [DUPLICATE KEY(col1[, col2, ...])] [PROPERTIES("key" = "value")]`

    Example:

    ```
    ROLLUP (
        r1 (k1, k3, v1, v2),
        r2 (k1, v1)
    )
    ```

#### properties

Set table properties. The following attributes are currently supported:

* `replication_num`

   Number of copies. The default number of copies is 3. If the number of BE nodes is less than 3, you need to specify that the number of copies is less than or equal to the number of BE nodes.
    
   After version 0.15, this attribute will be automatically converted to the `replication_allocation` attribute, such as:

   `"replication_num" = "3"` will be automatically converted to `"replication_allocation" = "tag.location.default:3"`

* `replication_allocation`

    Set the copy distribution according to Tag. This attribute can completely cover the function of the `replication_num` attribute.

* `is_being_synced`  

    Used to identify whether this table is copied by CCR and is being synchronized by syncer. The default is `false`.  

    If set to `true`:  
    `colocate_with`, `storage_policy` properties will be erased  
    `dynamic partition`, `auto bucket` features will be disabled, that is, they will be displayed as enabled in `show create table`, but will not actually take effect. When `is_being_synced` is set to `false`, these features will resume working.  

    This property is for CCR peripheral modules only and should not be manually set during CCR synchronization.  

* `storage_medium/storage_cooldown_time`

   Data storage medium. `storage_medium` is used to declare the initial storage medium of the table data, and `storage_cooldown_time` is used to set the expiration time. Example:

   ```
   "storage_medium" = "SSD",
   "storage_cooldown_time" = "2020-11-20 00:00:00"
   ```

   This example indicates that the data is stored in the SSD and will be automatically migrated to the HDD storage after the expiration of 2020-11-20 00:00:00.

* `colocate_with`

   When you need to use the Colocation Join function, use this parameter to set the Colocation Group.

   `"colocate_with" = "group1"`

* `bloom_filter_columns`

   The user specifies the list of column names that need to be added to the Bloom Filter index. The Bloom Filter index of each column is independent, not a composite index.

   `"bloom_filter_columns" = "k1, k2, k3"`

* `in_memory`

   Deprecated.

* `function_column.sequence_col`

   When using the UNIQUE KEY model, you can specify a sequence column. When the KEY columns are the same, REPLACE will be performed according to the sequence column (the larger value replaces the smaller value, otherwise it cannot be replaced)

  The `function_column.sequence_col` is used to specify the mapping of the sequence column to a column in the table, which can be integral and time (DATE, DATETIME). The type of this column cannot be changed after creation. If `function_column.sequence_col` is set, `function_column.sequence_type` is ignored.

   `"function_column.sequence_col" ='column_name'`

* `function_column.sequence_type`

   When using the UNIQUE KEY model, you can specify a sequence column. When the KEY columns are the same, REPLACE will be performed according to the sequence column (the larger value replaces the smaller value, otherwise it cannot be replaced)

   Here we only need to specify the type of sequence column, support time type or integer type. Doris will create a hidden sequence column.

   `"function_column.sequence_type" ='Date'`

* `compression`

   The default compression method for Doris tables is LZ4. After version 1.1, it is supported to specify the compression method as ZSTD to obtain a higher compression ratio.

   `"compression"="zstd"`

* `light_schema_change`

   Whether to use the Light Schema Change optimization.
    
   If set to true, the addition and deletion of value columns can be done more quickly and synchronously.

   `"light_schema_change"="true"`

   This feature is enabled by default after v2.0.0.

* `disable_auto_compaction`

   Whether to disable automatic compaction for this table.

   If this property is set to 'true', the background automatic compaction process will skip all the tables of this table.

   `"disable_auto_compaction" = "false"`

* `enable_single_replica_compaction`

   Whether to enable single replica compaction for this table.

   If this property is set to 'true', all replicas of the tablet will only have one replica performing compaction, while the others fetch rowsets from that replica.

   `"enable_single_replica_compaction" = "false"`

* `enable_duplicate_without_keys_by_default`

   When `true`, if Unique, Aggregate, or Duplicate is not specified when creating a table, a Duplicate model table without sorting columns and prefix indexes will be created by default.

   `"enable_duplicate_without_keys_by_default" = "false"`

* `skip_write_index_on_load`

   Whether to enable skip inverted index on load for this table.

   If this property is set to 'true', skip writting index (only inverted index now) on first time load and delay writting 
   index to compaction. It can reduce CPU and IO resource usage for high throughput load.

   `"skip_write_index_on_load" = "false"`

* `compaction_policy`

    Configure the compaction strategy in the compression phase. Only support configuring the compaction policy as "time_series" or "size_based".

    time_series: When the disk size of a rowset accumulates to a certain threshold, version merging takes place. The merged rowset is directly promoted to the base compaction stage. This approach effectively reduces the write amplification rate of compaction, especially in scenarios with continuous imports in a time series context.

    In the case of time series compaction, the execution of compaction is adjusted using parameters that have the prefix time_series_compaction.

    `"compaction_policy" = ""`

* `time_series_compaction_goal_size_mbytes`

    Time series compaction policy will utilize this parameter to adjust the size of input files for each compaction. The output file size will be approximately equal to the input file size.

    `"time_series_compaction_goal_size_mbytes" = "1024"`

* `time_series_compaction_file_count_threshold`

    Time series compaction policy will utilize this parameter to adjust the minimum number of input files for each compaction.

    If the number of files in a tablet exceeds the configured threshold, it will trigger a compaction process.

    `"time_series_compaction_file_count_threshold" = "2000"`

* `time_series_compaction_time_threshold_seconds`

     When time series compaction policy is applied, a significant duration passes without a compaction being executed, a compaction will be triggered.

    `"time_series_compaction_time_threshold_seconds" = "3600"`

* `time_series_compaction_level_threshold`

    When time series compaction policy is applied, This parameter defaults to 1.  When set to 2, it is used to control the re-merging of segments that have been 
    
    merged once, ensuring that the segment size reaches the time_series_compaction_goal_size_mbytes, which can achieve the effect of reducing the number of 
    
    segments.

    `"time_series_compaction_level_threshold" = "2"`


* Dynamic partition related

   The relevant parameters of dynamic partition are as follows:

* `dynamic_partition.enable`: Used to specify whether the dynamic partition function at the table level is enabled. The default is true.
* `dynamic_partition.time_unit:` is used to specify the time unit for dynamically adding partitions, which can be selected as DAY (day), WEEK (week), MONTH (month), YEAR (year), HOUR (hour).
* `dynamic_partition.start`: Used to specify how many partitions to delete forward. The value must be less than 0. The default is Integer.MIN_VALUE.
* `dynamic_partition.end`: Used to specify the number of partitions created in advance. The value must be greater than 0.
* `dynamic_partition.prefix`: Used to specify the partition name prefix to be created. For example, if the partition name prefix is ​​p, the partition name will be automatically created as p20200108.
* `dynamic_partition.buckets`: Used to specify the number of partition buckets that are automatically created.
* `dynamic_partition.create_history_partition`: Whether to create a history partition.
* `dynamic_partition.history_partition_num`: Specify the number of historical partitions to be created.
* `dynamic_partition.reserved_history_periods`: Used to specify the range of reserved history periods.
    
### Example

1. Create a detailed model table

    ```sql
    CREATE TABLE example_db.table_hash
    (
        k1 TINYINT,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        k3 CHAR(10) COMMENT "string column",
        k4 INT NOT NULL DEFAULT "1" COMMENT "int column"
    )
    COMMENT "my first table"
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    ```

2. Create a detailed model table, partition, specify the sorting column, and set the number of copies to 1

    ```sql
    CREATE TABLE example_db.table_hash
    (
        k1 DATE,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        k3 CHAR(10) COMMENT "string column",
        k4 INT NOT NULL DEFAULT "1" COMMENT "int column"
    )
    DUPLICATE KEY(k1, k2)
    COMMENT "my first table"
    PARTITION BY RANGE(k1)
    (
        PARTITION p1 VALUES LESS THAN ("2020-02-01"),
        PARTITION p2 VALUES LESS THAN ("2020-03-01"),
        PARTITION p3 VALUES LESS THAN ("2020-04-01")
    )
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    PROPERTIES (
        "replication_num" = "1"
    );
    ```

3. Create a table with a unique model of the primary key, set the initial storage medium and cooling time

    ```sql
    CREATE TABLE example_db.table_hash
    (
        k1 BIGINT,
        k2 LARGEINT,
        v1 VARCHAR(2048),
        v2 SMALLINT DEFAULT "10"
    )
    UNIQUE KEY(k1, k2)
    DISTRIBUTED BY HASH (k1, k2) BUCKETS 32
    PROPERTIES(
        "storage_medium" = "SSD",
        "storage_cooldown_time" = "2015-06-04 00:00:00"
    );
    ```

4. Create an aggregate model table, using a fixed range partition description

    ```sql
    CREATE TABLE table_range
    (
        k1 DATE,
        k2 INT,
        k3 SMALLINT,
        v1 VARCHAR(2048) REPLACE,
        v2 INT SUM DEFAULT "1"
    )
    AGGREGATE KEY(k1, k2, k3)
    PARTITION BY RANGE (k1, k2, k3)
    (
        PARTITION p1 VALUES [("2014-01-01", "10", "200"), ("2014-01-01", "20", "300")),
        PARTITION p2 VALUES [("2014-06-01", "100", "200"), ("2014-07-01", "100", "300"))
    )
    DISTRIBUTED BY HASH(k2) BUCKETS 32
    ```

5. Create an aggregate model table with HLL and BITMAP column types

    ```sql
    CREATE TABLE example_db.example_table
    (
        k1 TINYINT,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        v1 HLL HLL_UNION,
        v2 BITMAP BITMAP_UNION
    )
    ENGINE=olap
    AGGREGATE KEY(k1, k2)
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    ```

6. Create two self-maintained tables of the same Colocation Group.

    ```sql
    CREATE TABLE t1 (
        id int(11) COMMENT "",
        value varchar(8) COMMENT ""
    )
    DUPLICATE KEY(id)
    DISTRIBUTED BY HASH(id) BUCKETS 10
    PROPERTIES (
        "colocate_with" = "group1"
    );

    CREATE TABLE t2 (
        id int(11) COMMENT "",
        value1 varchar(8) COMMENT "",
        value2 varchar(8) COMMENT ""
    )
    DUPLICATE KEY(`id`)
    DISTRIBUTED BY HASH(`id`) BUCKETS 10
    PROPERTIES (
        "colocate_with" = "group1"
    );
    ```

7. Create a table with inverted index and bloom filter index

    ```sql
    CREATE TABLE example_db.table_hash
    (
        k1 TINYINT,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        v1 CHAR(10) REPLACE,
        v2 INT SUM,
        INDEX k1_idx (k1) USING INVERTED COMMENT'my first index'
    )
    AGGREGATE KEY(k1, k2)
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    PROPERTIES (
        "bloom_filter_columns" = "k2"
    );
    ```

8. Create a dynamic partition table.

    The table creates partitions 3 days in advance every day, and deletes the partitions 3 days ago. For example, if today is `2020-01-08`, partitions named `p20200108`, `p20200109`, `p20200110`, `p20200111` will be created. The partition ranges are:

    ```
    [types: [DATE]; keys: [2020-01-08]; ‥types: [DATE]; keys: [2020-01-09];)
    [types: [DATE]; keys: [2020-01-09]; ‥types: [DATE]; keys: [2020-01-10];)
    [types: [DATE]; keys: [2020-01-10]; ‥types: [DATE]; keys: [2020-01-11];)
    [types: [DATE]; keys: [2020-01-11]; ‥types: [DATE]; keys: [2020-01-12];)
    ```

    ```sql
    CREATE TABLE example_db.dynamic_partition
    (
        k1 DATE,
        k2 INT,
        k3 SMALLINT,
        v1 VARCHAR(2048),
        v2 DATETIME DEFAULT "2014-02-04 15:36:00"
    )
    DUPLICATE KEY(k1, k2, k3)
    PARTITION BY RANGE (k1) ()
    DISTRIBUTED BY HASH(k2) BUCKETS 32
    PROPERTIES(
        "dynamic_partition.time_unit" = "DAY",
        "dynamic_partition.start" = "-3",
        "dynamic_partition.end" = "3",
        "dynamic_partition.prefix" = "p",
        "dynamic_partition.buckets" = "32"
    );
    ```

9. Create a table with a materialized view (ROLLUP).

    ```sql
    CREATE TABLE example_db.rolup_index_table
    (
        event_day DATE,
        siteid INT DEFAULT '10',
        citycode SMALLINT,
        username VARCHAR(32) DEFAULT'',
        pv BIGINT SUM DEFAULT '0'
    )
    AGGREGATE KEY(event_day, siteid, citycode, username)
    DISTRIBUTED BY HASH(siteid) BUCKETS 10
    ROLLUP (
        r1(event_day,siteid),
        r2(event_day,citycode),
        r3(event_day)
    )
    PROPERTIES("replication_num" = "3");
    ```

10. Set the replica of the table through the `replication_allocation` property.

    ```sql
    CREATE TABLE example_db.table_hash
    (
      k1 TINYINT,
    	k2 DECIMAL(10, 2) DEFAULT "10.5"
    )
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    PROPERTIES (
        "replication_allocation"="tag.location.group_a:1, tag.location.group_b:2"
    );
    ```
    ```sql
    CREATE TABLE example_db.dynamic_partition
    (
    	k1 DATE,
    	k2 INT,
    	k3 SMALLINT,
    	v1 VARCHAR(2048),
    	v2 DATETIME DEFAULT "2014-02-04 15:36:00"
    )
    PARTITION BY RANGE (k1) ()
    DISTRIBUTED BY HASH(k2) BUCKETS 32
    PROPERTIES(
        "dynamic_partition.time_unit" = "DAY",
        "dynamic_partition.start" = "-3",
        "dynamic_partition.end" = "3",
        "dynamic_partition.prefix" = "p",
        "dynamic_partition.buckets" = "32",
        "dynamic_partition.replication_allocation" = "tag.location.group_a:3"
     );
    ```

11. Set the table hot and cold separation policy through the `storage_policy` property.
    ```sql
        CREATE TABLE IF NOT EXISTS create_table_use_created_policy 
        (
            k1 BIGINT,
            k2 LARGEINT,
            v1 VARCHAR(2048)
        )
        UNIQUE KEY(k1)
        DISTRIBUTED BY HASH (k1) BUCKETS 3
        PROPERTIES(
            "storage_policy" = "test_create_table_use_policy",
            "replication_num" = "1"
        );
    ```
NOTE: Need to create the s3 resource and storage policy before the table can be successfully associated with the migration policy 

12. Add a hot and cold data migration strategy for the table partition
    ```sql
        CREATE TABLE create_table_partion_use_created_policy
        (
            k1 DATE,
            k2 INT,
            V1 VARCHAR(2048) REPLACE
        ) PARTITION BY RANGE (k1) (
            PARTITION p1 VALUES LESS THAN ("2022-01-01") ("storage_policy" = "test_create_table_partition_use_policy_1" ,"replication_num"="1"),
            PARTITION p2 VALUES LESS THAN ("2022-02-01") ("storage_policy" = "test_create_table_partition_use_policy_2" ,"replication_num"="1")
        ) DISTRIBUTED BY HASH(k2) BUCKETS 1;
    ```
NOTE: Need to create the s3 resource and storage policy before the table can be successfully associated with the migration policy 

:::tip Tips
This feature is supported since the Apache Doris 1.2 version
:::

13. Multi Partition by a partition desc
    ```sql
        CREATE TABLE create_table_multi_partion_date
        (
            k1 DATE,
            k2 INT,
            V1 VARCHAR(20)
        ) PARTITION BY RANGE (k1) (
            FROM ("2000-11-14") TO ("2021-11-14") INTERVAL 1 YEAR,
            FROM ("2021-11-14") TO ("2022-11-14") INTERVAL 1 MONTH,
            FROM ("2022-11-14") TO ("2023-01-03") INTERVAL 1 WEEK,
            FROM ("2023-01-03") TO ("2023-01-14") INTERVAL 1 DAY,
            PARTITION p_20230114 VALUES [('2023-01-14'), ('2023-01-15'))
        ) DISTRIBUTED BY HASH(k2) BUCKETS 1
        PROPERTIES(
            "replication_num" = "1"
        );
    ```
    ```sql
        CREATE TABLE create_table_multi_partion_date_hour
        (
            k1 DATETIME,
            k2 INT,
            V1 VARCHAR(20)
        ) PARTITION BY RANGE (k1) (
            FROM ("2023-01-03 12") TO ("2023-01-14 22") INTERVAL 1 HOUR
        ) DISTRIBUTED BY HASH(k2) BUCKETS 1
        PROPERTIES(
            "replication_num" = "1"
        );
    ```
    ```sql
        CREATE TABLE create_table_multi_partion_integer
        (
            k1 BIGINT,
            k2 INT,
            V1 VARCHAR(20)
        ) PARTITION BY RANGE (k1) (
            FROM (1) TO (100) INTERVAL 10
        ) DISTRIBUTED BY HASH(k2) BUCKETS 1
        PROPERTIES(
            "replication_num" = "1"
        );
    ```

NOTE: Multi Partition can be mixed with conventional manual creation of partitions. When using, you need to limit the partition column to only one, The default maximum number of partitions created in multi partition is 4096, This parameter can be adjusted in fe configuration `max_multi_partition_num`.



1.  Add a duplicate without sorting column table

    ```sql
    CREATE TABLE example_db.table_hash
    (
        k1 DATE,
        k2 DECIMAL(10, 2) DEFAULT "10.5",
        k3 CHAR(10) COMMENT "string column",
        k4 INT NOT NULL DEFAULT "1" COMMENT "int column"
    )
    COMMENT "duplicate without keys"
    PARTITION BY RANGE(k1)
    (
        PARTITION p1 VALUES LESS THAN ("2020-02-01"),
        PARTITION p2 VALUES LESS THAN ("2020-03-01"),
        PARTITION p3 VALUES LESS THAN ("2020-04-01")
    )
    DISTRIBUTED BY HASH(k1) BUCKETS 32
    PROPERTIES (
        "replication_num" = "1",
        "enable_duplicate_without_keys_by_default" = "true"
    );
    ```

### Keywords

    CREATE, TABLE

### Best Practice

#### Partitioning and bucketing

A table must specify the bucket column, but it does not need to specify the partition. For the specific introduction of partitioning and bucketing, please refer to the [Data Division](../../../../table-design/data-partitioning/dynamic-partition) document.

Tables in Doris can be divided into partitioned tables and non-partitioned tables. This attribute is determined when the table is created and cannot be changed afterwards. That is, for partitioned tables, you can add or delete partitions in the subsequent use process, and for non-partitioned tables, you can no longer perform operations such as adding partitions afterwards.

At the same time, partitioning columns and bucketing columns cannot be changed after the table is created. You can neither change the types of partitioning and bucketing columns, nor do any additions or deletions to these columns.

Therefore, it is recommended to confirm the usage method to build the table reasonably before building the table.

#### Dynamic Partition

The dynamic partition function is mainly used to help users automatically manage partitions. By setting certain rules, the Doris system regularly adds new partitions or deletes historical partitions. Please refer to [Dynamic Partition](../../../../table-design/data-partitioning/dynamic-partition) document for more help.

#### Materialized View

Users can create multiple materialized views (ROLLUP) while building a table. Materialized views can also be added after the table is built. It is convenient for users to create all materialized views at one time by writing in the table creation statement.

If the materialized view is created when the table is created, all subsequent data import operations will synchronize the data of the materialized view to be generated. The number of materialized views may affect the efficiency of data import.

If you add a materialized view in the subsequent use process, if there is data in the table, the creation time of the materialized view depends on the current amount of data.

For the introduction of materialized views, please refer to the document [materialized views](../../../../query/view-materialized-view/materialized-view).

#### Index

Users can create indexes on multiple columns while building a table. Indexes can also be added after the table is built.

If you add an index in the subsequent use process, if there is data in the table, you need to rewrite all the data, so the creation time of the index depends on the current data volume.

