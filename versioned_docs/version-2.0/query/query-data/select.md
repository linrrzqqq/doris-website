---
{
    "title": "Select Manual",
    "language": "en"
}
---

## Select

Select syntax

```
SELECT
    [hint_statement, ...]
    [ALL | DISTINCT | DISTINCTROW | ALL EXCEPT ( col_name1 [, col_name2, col_name3, ...] )]
    select_expr [, select_expr ...]
    [FROM table_references
      [PARTITION partition_list]
      [TABLET tabletid_list]
      [TABLESAMPLE sample_value [ROWS | PERCENT]
        [REPEATABLE pos_seek]]
    [WHERE where_condition]
    [GROUP BY [GROUPING SETS | ROLLUP | CUBE] {col_name | expr | position}]
    [HAVING where_condition]
    [ORDER BY {col_name | expr | position}
      [ASC | DESC], ...]
    [LIMIT {[offset,] row_count | row_count OFFSET offset}]
    [INTO OUTFILE 'file_name']
```

## Syntax explanation

- `select_expr, ...`Specifies the columns to retrieve and display in the result set. Aliases can be used, and `as` is optional.
- `table_references`Specifies the target tables for retrieval, which can be one or more tables (including temporary tables generated by subqueries).
- `where_definition`Specifies the retrieval conditions (expressions). If a WHERE clause exists, its conditions filter the row data. The `where_condition` is an expression that evaluates to true for each row to be selected. If there is no WHERE clause, the statement selects all rows. In a WHERE expression, you can use any MySQL-supported functions and operators except aggregate functions.
- `ALL | DISTINCT`Filters the result set. `ALL` selects all rows, while `DISTINCT` or `DISTINCTROW` filters out duplicate rows. The default is `ALL`.
- `ALL EXCEPT`Filters the result set from `ALL` by specifying one or more column names to exclude from the full result set. All matching column names will be ignored in the output.
- `INTO OUTFILE 'file_name'`Saves the result set to a new file (which must not exist beforehand), with differences in the saved format.
- `Group by having`Groups the result set by one or more columns. If `HAVING` is present, it filters the groups produced by `GROUP BY`. Extensions to `GROUP BY` such as `GROUPING SETS`, `ROLLUP`, and `CUBE` are available and detailed in the [GROUPING SETS](https://doris.apache.org/community/design/grouping_sets_design).
- `Order by`Sorts the final result set. `ORDER BY` sorts the result set by comparing values in one or more columns. Sorting operations can be time-consuming and resource-intensive because all data needs to be sent to a single node for sorting. Sorting requires more memory compared to non-sorted operations. If you need to return the top N sorted results, use the `LIMIT` clause. 
- `Limit n`Limits the number of rows in the output result set. `LIMIT m,n` means to start outputting from the mth row and return n records. Using `LIMIT m,n` is meaningful only when combined with `ORDER BY`, otherwise the data returned may be inconsistent each time the query is executed.
- `Having`The `HAVING` clause does not filter rows in the table but filters the results produced by aggregate functions. Typically, `HAVING` is used with aggregate functions (such as `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()`) and the `GROUP BY` clause.
- `SELECT` supports explicit partition selection using `PARTITION`, which includes a list of partitions or subpartitions (or both) following the table name in `table_reference`.
- `[TABLET tids] TABLESAMPLE n [ROWS | PERCENT] [REPEATABLE seek]`Limits the number of rows read from a table in the `FROM` clause by pseudo-randomly selecting several tablets based on the specified number of rows or percentage. `REPEATABLE` with a specified seed allows the same sample to be returned again. Alternatively, Tablet IDs can be manually specified, but this is only applicable to OLAP tables.
- `hint_statement`Using hints before the select list can influence the optimizer's behavior to obtain a desired execution plan. For more information, refer to the [joinHint Document](../join-optimization/join-hint).

## Syntax constraints

- `SELECT` can also be used to retrieve calculated rows without referencing any tables.
- All clauses must strictly follow the above format. A `HAVING` clause must come after the `GROUP BY` clause and before the `ORDER BY` clause.
- The alias keyword `AS` is optional. Aliases can be used in `GROUP BY`, `ORDER BY,` and `HAVING`.
- `WHERE` clause: Executes the `WHERE` statement to determine which rows should be included in the `GROUP BY` section, while `HAVING` is used to determine which rows from the result set should be used.
- The `HAVING` clause can reference aggregate functions, such as `count, sum, max, min, avg`, while the `WHERE` clause cannot. However, the `WHERE` clause can reference other functions besides aggregate functions. Column aliases cannot be used in the `WHERE` clause to define conditions.
- Following `GROUP BY` with `WITH ROLLUP` allows for one or more aggregations of the results.

## Join syntax

Doris supports the following JOIN syntax.

```
JOIN
table_references:
    table_reference [, table_reference] …
table_reference:
    table_factor
  | join_table
table_factor:
    tbl_name [[AS] alias]
        [{USE|IGNORE|FORCE} INDEX (key_list)]
  | ( table_references )
  | { OJ table_reference LEFT OUTER JOIN table_reference
        ON conditional_expr }
join_table:
    table_reference [INNER | CROSS] JOIN table_factor [join_condition]
  | table_reference LEFT [OUTER] JOIN table_reference join_condition
  | table_reference NATURAL [LEFT [OUTER]] JOIN table_factor
  | table_reference RIGHT [OUTER] JOIN table_reference join_condition
  | table_reference NATURAL [RIGHT [OUTER]] JOIN table_factor
join_condition:
    ON conditional_expr
```

UNION:

```
SELECT ...
UNION [ALL| DISTINCT] SELECT ......
[UNION [ALL| DISTINCT] SELECT ...]
```

`UNION` is used to combine the results of multiple `SELECT` statements into a single result set. The column names from the first `SELECT` statement are used as the column names for the returned result. The selected columns listed in the corresponding positions of each `SELECT` statement should have the same data type. (For example, the first column selected in the first statement should have the same type as the first column selected in the other statements.)

By default, `UNION` removes duplicate rows from the result. The optional `DISTINCT` keyword has no effect beyond the default, as it also specifies duplicate row removal. Using the optional `ALL` keyword, no duplicate row removal occurs, and the result includes all matching rows from all `SELECT` statements.

INTERSECT:

```sql
SELECT ...
INTERSECT [DISTINCT] SELECT ......
[INTERSECT [DISTINCT] SELECT ...]
```

`INTERSECT` is used to return the intersection of results from multiple `SELECT` statements, with duplicate results removed.
The effect of `INTERSECT` is equivalent to `INTERSECT DISTINCT`. The `ALL` keyword is not supported.
Each `SELECT` query must return the same number of columns, And when the column types are inconsistent, they will be `CAST` to the same type.

EXCEPT/MINUS:

```sql
SELECT ...
EXCEPT [DISTINCT] SELECT ......
[EXCEPT [DISTINCT] SELECT ...]
```

The `EXCEPT` clause is used to return the complement between the results of multiple queries, meaning it returns the data from the left query that does not exist in the right query, with duplicates removed.
`EXCEPT` is functionally equivalent to `MINUS`.
The effect of `EXCEPT` is the same as `EXCEPT DISTINCT`. The `ALL` keyword is not supported.
Each `SELECT` query must return the same number of columns, And when the column types are inconsistent, they will be `CAST` to the same type.

WITH:

To specify a common table expression, use a `WITH` clause with one or more comma-separated subclauses. Each subclause provides a subquery that generates a result set and associates a name with the subquery. The following example defines CTEs named `cte1` and `cte2` in the `WITH` clause, and refers to them in the top-level `SELECT `following the WITH clause.

```
WITH
  cte1 AS (SELECT a，b FROM table1),
  cte2 AS (SELECT c，d FROM table2)
SELECT b，d FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```

In statements that include this `WITH` clause, each CTE name can be referenced to access the corresponding CTE result set. CTE names can be referenced in other CTEs, allowing CTEs to be defined based on other CTEs. Currently, recursive CTEs are not supported.

## Example

- Query the names of students whose ages are 18, 20, and 25.

```
select Name from student where age in (18,20,25);
```

- ALL EXCEPT 

```
-- Query all information except for the age of the students.
select * except(age) from student; 
```

- GROUP BY 

```
--Query the tb_book table, group by type, and calculate the average price for each category of books.
select type,avg(price) from tb_book group by type;
```

- DISTINCT 

```
--Query the tb_book table and remove duplicate type data.
select distinct type from tb_book;
```

- ORDER BY 

Sort the query results in ascending order (by default) or descending order (DESC). In ascending order, NULL values should appear at the beginning, and in descending order, NULL values should appear at the end.

```
--Query all records from the tb_book table, sort them in descending order by id, and display only the first three records.
select * from tb_book order by id desc limit 3;
```

- LIKE 

`LIKE` can perform fuzzy queries with two wildcards: `%` and `_`. The `%` wildcard matches one or more characters, while the `_` wildcard matches a single character.

```
-- Find all books where the second character is 'h'.
select * from tb_book where name like('_h%');
```

- LIMIT (Limit the number of result rows.)

```
-- Display 3 records in descending order.
select * from tb_book order by price desc limit 3;

Display 4 records starting from id=1
select * from tb_book where id limit 1,4;
```

- CONCAT (Concatenate multiple columns

```
--Concatenate 'name' and 'price' into a new string for output.
select id,concat(name,":",price) as info,type from tb_book;
```

- Functions and expressions

```
--Calculate the total price of each category of books in the tb_book table.
select sum(price) as total,type from tb_book group by type;
--20% off the price
select *,(price * 0.8) as "20% off" from tb_book;
```

- UNION 

```
SELECT a FROM t1 WHERE a = 10 AND B = 1 ORDER by a LIMIT 10
UNION
SELECT a FROM t2 WHERE a = 11 AND B = 2 ORDER by a LIMIT 10;
```

- INTERSECT

```sql
SELECT a FROM t1 WHERE a = 10 AND B = 1 ORDER by a LIMIT 10
INTERSECT
SELECT a FROM t2 WHERE a = 11 AND B = 2 ORDER by a LIMIT 10;
```

- EXCEPT

```sql
SELECT a FROM t1 WHERE a = 10 AND B = 1 ORDER by a LIMIT 10
EXCEPT
SELECT a FROM t2 WHERE a = 11 AND B = 2 ORDER by a LIMIT 10;
```

- WITH clause

```
WITH cte AS
(
  SELECT 1 AS col1, 2 AS col2
  UNION ALL
  SELECT 3, 4
)
SELECT col1, col2 FROM cte;
```

- JOIN 

```
SELECT * FROM t1 LEFT JOIN (t2, t3, t4)
                 ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```

the same as

```
SELECT * FROM t1 LEFT JOIN (t2 CROSS JOIN t3 CROSS JOIN t4)
                 ON (t2.a = t1.a AND t3.b = t1.b AND t4.c = t1.c)
```

- INNER JOIN

```
SELECT t1.name, t2.salary
  FROM employee AS t1 INNER JOIN info AS t2 ON t1.name = t2.name;

SELECT t1.name, t2.salary
  FROM employee t1 INNER JOIN info t2 ON t1.name = t2.name;
```

- LEFT JOIN

```
SELECT left_tbl.*
  FROM left_tbl LEFT JOIN right_tbl ON left_tbl.id = right_tbl.id
  WHERE right_tbl.id IS NULL;
```

- RIGHT JOIN

```
mysql SELECT * FROM t1 RIGHT JOIN t2 ON (t1.a = t2.a);
+------+------+------+------+
| a    | b    | a    | c    |
+------+------+------+------+
|    2 | y    |    2 | z    |
| NULL | NULL |    3 | w    |
+------+------+------+------+
```

- TABLESAMPLE

```
--Randomly sample 1000 rows in t1 using pseudo-random method. Note that the actual process is to select several Tablets based on the statistical information of the table, and the total number of rows in the selected Tablets may be greater than 1000. Therefore, if you want to return exactly 1000 rows, you need to add a Limit clause.
SELECT * FROM t1 TABLET(10001) TABLESAMPLE(1000 ROWS) REPEATABLE 2 limit 1000;
```

## Best practice

- Additional explanation about the SELECT clause:
  - An alias can be specified for `select_expr` using `AS alias_name`. The alias serves as the column name for the expression and can be used in `GROUP BY`, `ORDER BY`, or `HAVING` clauses.
  - The `table_references` after `FROM` indicate one or multiple tables involved in the query. If multiple tables are listed, a `JOIN` operation will be performed. Each specified table can be assigned an alias.
  - The selected columns after `SELECT` can be referenced in `ORDER BY` and `GROUP BY` clauses using column names, column aliases, or integers representing the column position (starting from 1).

```
SELECT college, region, seed FROM tournament
  ORDER BY region, seed;

SELECT college, region AS r, seed AS s FROM tournament
  ORDER BY r, s;

SELECT college, region, seed FROM tournament
  ORDER BY 2, 3;
```

- If `ORDER BY` appears in a subquery and is also applied to the outer query, the outermost `ORDER BY` takes precedence.
- When using `GROUP BY`, the grouped columns are automatically sorted in ascending order (as if an `ORDER BY` clause followed with the same columns). To avoid the overhead caused by the automatic sorting of `GROUP BY`, adding `ORDER BY NULL` can solve the issue:

```
SELECT a, COUNT(b) FROM test_table GROUP BY a ORDER BY NULL;
```

- When sorting columns in a `SELECT` statement using `ORDER BY` or `GROUP BY`, the server only sorts the values using the initial number of bytes indicated by the `max_sort_length` system variable.
- The `HAVING` clause is typically applied at the end, just before the result set is returned to the client, and it is not optimized (whereas `LIMIT` is applied after `HAVING`).
- According to the SQL standard, `HAVING` must reference columns that are either in the `GROUP BY` list or used in aggregate functions. However, MySQL extends this by allowing `HAVING` to reference columns from the `SELECT` clause list and columns from outer subqueries.
- If a column referenced in `HAVING` is ambiguous, a warning will be generated. In the following statement, `col2` is ambiguous:

```
SELECT COUNT(col1) AS col2 FROM t GROUP BY col2 HAVING col2 = 2;
```

Do not use `HAVING` where `WHERE` should be used. `HAVING` is intended to be used with `GROUP BY`.

The `HAVING` clause can reference aggregate functions, whereas `WHERE` cannot.

```
SELECT user, MAX(salary) FROM users
  GROUP BY user HAVING MAX(salary)  10;
```

- The `LIMIT` clause can be used to restrict the number of rows returned by a `SELECT` statement. `LIMIT` can have one or two parameters, both of which must be non-negative integers.

```
-- Retrieve rows 6 to 15 from the result set.
SELECT * FROM tbl LIMIT 5,10;
-- If you want to retrieve all rows starting from a certain offset, you can set a very large constant as the second parameter. The following query retrieves all data starting from the 96th row:
SELECT * FROM tbl LIMIT 95,18446744073709551615;
-- If LIMIT has only one parameter, then the parameter specifies the number of rows that should be retrieved, and the offset is defaulted to 0, which means starting from the first row.
```

- `SELECT...INTO` allows the query results to be written to a file.
- Modifiers for the `SELECT` keyword:
  - Primarily used for removing duplicates.
  - The `ALL` and `DISTINCT` modifiers specify whether to remove duplicate rows (not a specific column) from the result set.
  - `ALL` is the default modifier, meaning all rows that meet the criteria will be retrieved.
  - `DISTINCT `removes duplicate rows.
- Key advantages of subqueries:
  - Subqueries enable structured queries, allowing each part of a statement to be isolated.
  - Some operations require complex joins and associations. Subqueries provide alternative methods to perform these operations.
- Accelerating queries:
  - Utilize Doris's partitioning and bucketing as data filtering conditions to reduce the data scanning range as much as possible.
  - Make full use of Doris's prefix index fields as data filtering conditions to speed up query performance.
- UNION:Using only the `union` keyword has the same effect as using `union distinct`. Since deduplication can be memory-intensive, using `union all` for queries can result in faster performance and reduced memory consumption. If users want to perform `order by` and `limit` operations on the returned result set, they should place the `union` operation within a subquery, then select from that subquery, and finally, place the subquery along with `order by` outside.

```
select * from (select age from student_01 union all select age from student_02) as t1 
order by age limit 4;

+-------------+
|     age     |
+-------------+
|      18     |
|      19     |
|      20     |
|      21     |
+-------------+
4 rows in set (0.01 sec)
```

- JOIN
  - In addition to supporting equi-join in inner join conditions, non-equi-join is also supported. However, for performance considerations, it is recommended to use equi-join.
  - Other types of joins only support equi-join.