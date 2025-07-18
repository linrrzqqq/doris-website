---
{
    "title": "Sort Key and Prefix Index",
    "language": "en"
}
---

## Index Principles

Doris stores data in a structure similar to SSTable (Sorted String Table). This structure is an ordered data structure that can be sorted and stored according to one or more specified columns. In such a data structure, looking up conditions on all or part of the sorted columns is highly efficient.

In the Aggregate, Unique, and Duplicate data models, the underlying data storage is sorted according to the columns specified in the CREATE TABLE statement under AGGREGATE KEY, UNIQUE KEY, and DUPLICATE KEY respectively. These keys are referred to as sort keys. With sort keys, Doris can quickly locate the required data without scanning the entire table by specifying conditions on the sorted columns during a query, thereby reducing search complexity and speeding up the query.

Based on the sort keys, Doris introduces a prefix index. The prefix index is a sparse index. The data in the table forms a logical data block (Data Block) according to the corresponding number of rows. Each logical data block stores an index entry in the prefix index table, where the length of the index entry does not exceed 36 bytes. The entry content is the prefix composed of the sorted columns of the first row in the data block. When looking up the prefix index table, it helps determine the starting row number of the logical data block where the row data is located. Because the prefix index is relatively small, it can be fully cached in memory, allowing for quick data block location and significantly improving query efficiency.

:::tip

The first 36 bytes of a row in a data block are used as the prefix index for that row. When encountering a VARCHAR type, the prefix index is directly truncated. If the first column is VARCHAR, even if it does not reach 36 bytes, it will be directly truncated, and the subsequent columns will not be included in the prefix index.
:::

## Use Cases

Prefix indexes can speed up equality queries and range queries.

## Managing Indexes

There is no specific syntax to define a prefix index. When creating a table, the first 36 bytes of the table's Key are automatically taken as the prefix index.

### Recommendations for Prefix Index Selection

:::tip

Because the Key definition of a table is unique, a table has only one set of prefix indexes. Therefore, it is important to choose an appropriate prefix index when designing the table structure. The following recommendations can be considered:
1. Choose the fields most commonly used in WHERE filtering conditions as the Key.
2. Place the more frequently used fields at the front, as prefix indexes are only effective for fields in the WHERE condition that are part of the Key's prefix.

For queries that use other columns not covered by the prefix index as conditions, the efficiency may not meet the requirements. There are two solutions:
1. Create an inverted index on the columns that require accelerated queries, as a table can have many inverted indexes.
2. For DUPLICATE tables, multi-prefix indexes can be indirectly achieved by creating corresponding strongly consistent materialized views with adjusted column orders. For more details, refer to query acceleration/materialized views.

:::

## Using Indexes

Prefix indexes are used to accelerate equality and range queries in the WHERE clause. They automatically take effect when applicable, and there is no special syntax required.

The acceleration effect of the prefix index can be analyzed using the following metrics in the Query Profile:
- RowsKeyRangeFiltered: The number of rows filtered by the prefix index, which can be compared with other Rows values to analyze the filtering effect of the index.

## Example Usage

-   Suppose the sorted columns of a table are as follows, then the prefix index would be: user_id (8 Bytes) + age (4 Bytes) + message (prefix 20 Bytes).

| ColumnName     | Type         |
| -------------- | ------------ |
| user_id        | BIGINT       |
| age            | INT          |
| message        | VARCHAR(100) |
| max_dwell_time | DATETIME     |
| min_dwell_time | DATETIME     |

-   Suppose the sorted columns of a table are as follows, then the prefix index would be user_name (20 Bytes). Even if it does not reach 36 bytes, it is directly truncated due to encountering VARCHAR, and subsequent columns are not included.

| ColumnName     | Type         |
| -------------- | ------------ |
| user_name      | VARCHAR(20)  |
| age            | INT          |
| message        | VARCHAR(100) |
| max_dwell_time | DATETIME     |
| min_dwell_time | DATETIME     |

-   When our query condition is the prefix of the prefix index, it can significantly speed up the query. For example, in the first example, executing the following query:

```sql
SELECT * FROM table WHERE user_id = 1829239 AND age = 20;
```

This query will be much more efficient than the following query:

```sql
SELECT * FROM table WHERE age = 20;
```

Therefore, choosing the correct column order when creating a table can greatly improve query efficiency.
