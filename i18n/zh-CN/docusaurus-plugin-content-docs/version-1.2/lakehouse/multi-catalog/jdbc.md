---
{
    "title": "JDBC",
    "language": "zh-CN"
}
---

# JDBC

JDBC Catalog 通过标准 JDBC 协议，连接其他数据源。

连接后，Doris 会自动同步数据源下的 Database 和 Table 的元数据，以便快速访问这些外部数据。

## 使用限制

1. 支持 MySQL、PostgreSQL、Oracle、SQLServer、Clickhouse、Doris、SAP HANA、Trino、OceanBase

## 创建 Catalog

1. MySQL

:::tip 提示
该功能自 Apache Doris  1.2 版本起支持
:::

```sql
CREATE CATALOG jdbc_mysql PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:mysql://127.0.0.1:3306/demo",
    "driver_url" = "mysql-connector-java-5.1.47.jar",
    "driver_class" = "com.mysql.jdbc.Driver"
)
```

2. PostgreSQL

:::tip 提示
该功能自 Apache Doris  1.2.2 版本起支持
:::

```sql
CREATE CATALOG jdbc_postgresql PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:postgresql://127.0.0.1:5449/demo",
    "driver_url" = "postgresql-42.5.1.jar",
    "driver_class" = "org.postgresql.Driver"
);
```

映射 PostgreSQL 时，Doris 的一个 Database 对应于 PostgreSQL 中指定 Catalog 下的一个 Schema（如示例中 `jdbc_url` 参数中 "demo"下的 schemas）。而 Doris 的 Database 下的 Table 则对应于 PostgreSQL 中，Schema 下的 Tables。即映射关系如下：

| Doris    | PostgreSQL |
|----------|------------|
| Catalog  | Database   | 
| Database | Schema     |
| Table    | Table      |

> Doris 通过 sql 语句`select nspname from pg_namespace where has_schema_privilege('<UserName>', nspname, 'USAGE');` 来获得 PG user 能够访问的所有 schema 并将其映射为 Doris 的 database

3. Oracle

:::tip 提示
该功能自 Apache Doris  1.2.2 版本起支持
:::

```sql
CREATE CATALOG jdbc_oracle PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:oracle:thin:@127.0.0.1:1521:helowin",
    "driver_url" = "ojdbc6.jar",
    "driver_class" = "oracle.jdbc.driver.OracleDriver"
);
```

映射 Oracle 时，Doris 的一个 Database 对应于 Oracle 中的一个 User。而 Doris 的 Database 下的 Table 则对应于 Oracle 中，该 User 下的有权限访问的 Table。即映射关系如下：

| Doris    | Oracle   |
|----------|----------|
| Catalog  | Database | 
| Database | User     |
| Table    | Table    |

4. Clickhouse

:::tip 提示
该功能自 Apache Doris  1.2.2 版本起支持
:::

```sql
CREATE CATALOG jdbc_clickhouse PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:clickhouse://127.0.0.1:8123/demo",
    "driver_url" = "clickhouse-jdbc-0.3.2-patch11-all.jar",
    "driver_class" = "com.clickhouse.jdbc.ClickHouseDriver"
);
```

5. SQLServer


:::tip 提示
该功能自 Apache Doris  1.2.2 版本起支持
:::

```sql
CREATE CATALOG sqlserver_catalog PROPERTIES (
    "type"="jdbc",
    "user"="SA",
    "password"="Doris123456",
    "jdbc_url" = "jdbc:sqlserver://localhost:1433;DataBaseName=doris_test",
    "driver_url" = "mssql-jdbc-11.2.3.jre8.jar",
    "driver_class" = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
);
```

映射 SQLServer 时，Doris 的一个 Database 对应于 SQLServer 中指定 Database（如示例中 `jdbc_url` 参数中的 "doris_test"）下的一个 Schema。而 Doris 的 Database 下的 Table 则对应于 SQLServer 中，Schema 下的 Tables。即映射关系如下：

| Doris    | SQLServer |
|----------|-----------|
| Catalog  | Database  | 
| Database | Schema    |
| Table    | Table     |

6. Doris


:::tip 提示
该功能自 Apache Doris  1.2.3 版本起支持
:::

Jdbc Catalog 也支持连接另一个 Doris 数据库：

```sql
CREATE CATALOG doris_catalog PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:mysql://127.0.0.1:9030?useSSL=false",
    "driver_url" = "mysql-connector-java-5.1.47.jar",
    "driver_class" = "com.mysql.jdbc.Driver"
);
```

目前 Jdbc Catalog 连接一个 Doris 数据库只支持用 5.x 版本的 jdbc jar 包。如果使用 8.x jdbc jar 包，可能会出现列类型无法匹配问题。

7. SAP HANA


:::tip 提示
该功能自 Apache Doris  1.2.3 版本起支持
:::

```sql
CREATE CATALOG hana_catalog PROPERTIES (
    "type"="jdbc",
    "user"="SYSTEM",
    "password"="SAPHANA",
    "jdbc_url" = "jdbc:sap://localhost:31515/TEST",
    "driver_url" = "ngdbc.jar",
    "driver_class" = "com.sap.db.jdbc.Driver"
)
```

| Doris    | SAP HANA |
|----------|----------|
| Catalog  | Database | 
| Database | Schema   |
| Table    | Table    |

8. Trino


:::tip 提示
该功能自 Apache Doris  1.2.4 版本起支持
:::

```sql
CREATE CATALOG trino_catalog PROPERTIES (
    "type"="jdbc",
    "user"="hadoop",
    "password"="",
    "jdbc_url" = "jdbc:trino://localhost:9000/hive",
    "driver_url" = "trino-jdbc-389.jar",
    "driver_class" = "io.trino.jdbc.TrinoDriver"
);
```

映射 Trino 时，Doris 的 Database 对应于 Trino 中指定 Catalog（如示例中 `jdbc_url` 参数中的 "hive"）下的一个 Schema。而 Doris 的 Database 下的 Table 则对应于 Trino 中 Schema 下的 Tables。即映射关系如下：

| Doris    | Trino   |
|----------|---------|
| Catalog  | Catalog | 
| Database | Schema  |
| Table    | Table   |

9. OceanBase

 

```sql
CREATE CATALOG jdbc_oceanbase_mysql PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:oceanbase://127.0.0.1:2881/demo",
    "driver_url" = "oceanbase-client-2.4.2.jar",
    "driver_class" = "com.oceanbase.jdbc.Drive",
    "oceanbase_mode" = "mysql"
)

CREATE CATALOG jdbc_oceanbase_oracle PROPERTIES (
    "type"="jdbc",
    "user"="root",
    "password"="123456",
    "jdbc_url" = "jdbc:oceanbase://127.0.0.1:2881/demo",
    "driver_url" = "oceanbase-client-2.4.2.jar",
    "driver_class" = "com.oceanbase.jdbc.Drive",
    "oceanbase_mode" = "oracle"
)
```

### 参数说明

| 参数                        | 是否必须 | 默认值     | 说明                                                               |
|---------------------------|------|---------|------------------------------------------------------------------- |
| `user`                    | 是    |         | 对应数据库的用户名                                                         |
| `password`                | 是    |         | 对应数据库的密码                                                          |
| `jdbc_url`                | 是    |         | JDBC 连接串                                                          |
| `driver_url`              | 是    |         | JDBC Driver Jar 包名称*                                              |
| `driver_class`            | 是    |         | JDBC Driver Class 名称                                              |
| `only_specified_database` | 否    | "false" | 指定是否只同步指定的 database                                               |
| `lower_case_table_names`  | 否    | "false" | 是否以小写的形式同步 jdbc 外部数据源的表名                                            |
| `oceanbase_mode`          | 否    | ""      | 当连接的外部数据源为 OceanBase 时，必须为其指定模式为 mysql 或 oracle                        |
| `include_database_list`   | 否    | ""      | 当 only_specified_database=true 时，指定同步多个 database，以','分隔。db 名称是大小写敏感的。 |
| `exclude_database_list`   | 否    | ""      | 当 only_specified_database=true 时，指定不需要同步的多个 database，以','分割。db 名称是大小写敏感的。|

> `driver_url` 可以通过以下三种方式指定：
> 
> 1. 文件名。如 `mysql-connector-java-5.1.47.jar`。需将 Jar 包预先存放在 FE 和 BE 部署目录的 `jdbc_drivers/` 目录下。系统会自动在这个目录下寻找。该目录的位置，也可以由 fe.conf 和 be.conf 中的 `jdbc_drivers_dir` 配置修改。
> 
> 2. 本地绝对路径。如 `file:///path/to/mysql-connector-java-5.1.47.jar`。需将 Jar 包预先存放在所有 FE/BE 节点指定的路径下。
> 
> 3. Http 地址。如：`https://doris-community-test-1308700295.cos.ap-hongkong.myqcloud.com/jdbc_driver/mysql-connector-java-5.1.47.jar`。系统会从这个 http 地址下载 Driver 文件。仅支持无认证的 http 服务。

> `only_specified_database`:
> 在 jdbc 连接时可以指定链接到哪个 database/schema, 如：mysql 中 jdbc_url 中可以指定 database, pg 的 jdbc_url 中可以指定 currentSchema。
>
> `include_database_list`:
> 当`only_specified_database=true`时，指定需要同步的 database，以','分割。默认为''，即不做任何过滤，同步所有 database。db 名称是大小写敏感的
>
> `exclude_database_list`:
> 当`only_specified_database=true`时，指定不需要同步的多个 database，以','分割。默认为''，即不做任何过滤，同步所有 database。db 名称是大小写敏感的。
>
> 当 `include_database_list` 和 `exclude_database_list` 有重合的 database 配置时，`exclude_database_list`会优先生效。
>
> 如果使用该参数时连接 oracle 数据库，要求使用 ojdbc8.jar 以上版本 jar 包。

## 数据查询

```sql
select * from mysql_catalog.mysql_database.mysql_table where k1 > 1000 and k3 ='term';
```
由于可能存在使用数据库内部的关键字作为字段名，为解决这种状况下仍能正确查询，所以在 SQL 语句中，会根据各个数据库的标准自动在字段名与表名上加上转义符。例如 MYSQL(``)、PostgreSQL("")、SQLServer([])、ORACLE("")，所以此时可能会造成字段名的大小写敏感，具体可以通过 explain sql，查看转义后下发到各个数据库的查询语句。

## 数据写入


:::tip 提示
该功能自 Apache Doris  1.2.2 版本起支持
:::

在 Doris 中建立 JDBC Catalog 后，可以通过 insert into 语句直接写入数据，也可以将 Doris 执行完查询之后的结果写入 JDBC Catalog，或者是从一个 JDBC 外表将数据导入另一个 JDBC 外表。


示例：

```sql
insert into mysql_catalog.mysql_database.mysql_table values(1, "doris");
insert into mysql_catalog.mysql_database.mysql_table select * from table;
```
### 事务

Doris 的数据是由一组 batch 的方式写入外部表的，如果中途导入中断，之前写入数据可能需要回滚。所以 JDBC 外表支持数据写入时的事务，事务的支持需要通过设置 session variable: `enable_odbc_transcation `。

```sql
set enable_odbc_transcation = true; 
```

事务保证了 JDBC 外表数据写入的原子性，但是一定程度上会降低数据写入的性能，可以考虑酌情开启该功能。

## 列类型映射

### MySQL

| MYSQL Type                                                                                                                                            | Doris Type  | Comment                                     |
|-------------------------------------------------------------------------------------------------------------------------------------------------------|-------------|---------------------------------------------|
| BOOLEAN                                                                                                                                               | BOOLEAN     |                                             |
| TINYINT                                                                                                                                               | TINYINT     |                                             |
| SMALLINT                                                                                                                                              | SMALLINT    |                                             |
| MEDIUMINT                                                                                                                                             | INT         |                                             |
| INT                                                                                                                                                   | INT         |                                             |
| BIGINT                                                                                                                                                | BIGINT      |                                             |
| UNSIGNED TINYINT                                                                                                                                      | SMALLINT    | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级 |
| UNSIGNED MEDIUMINT                                                                                                                                    | INT         | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级 | 
| UNSIGNED INT                                                                                                                                          | BIGINT      | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级 |
| UNSIGNED BIGINT                                                                                                                                       | LARGEINT    |                                             |
| FLOAT                                                                                                                                                 | FLOAT       |                                             |
| DOUBLE                                                                                                                                                | DOUBLE      |                                             |
| DECIMAL                                                                                                                                               | DECIMAL     |                                             |
| DATE                                                                                                                                                  | DATE        |                                             |
| TIMESTAMP                                                                                                                                             | DATETIME    |                                             |
| DATETIME                                                                                                                                              | DATETIME    |                                             |
| YEAR                                                                                                                                                  | SMALLINT    |                                             |
| TIME                                                                                                                                                  | STRING      |                                             |
| CHAR                                                                                                                                                  | CHAR        |                                             |
| VARCHAR                                                                                                                                               | VARCHAR     |                                             |
| TINYTEXT、TEXT、MEDIUMTEXT、LONGTEXT、TINYBLOB、BLOB、MEDIUMBLOB、LONGBLOB、TINYSTRING、STRING、MEDIUMSTRING、LONGSTRING、BINARY、VARBINARY、JSON、SET、BIT | STRING      |                                             |
| Other                                                                                                                                                 | UNSUPPORTED |                                             |

### PostgreSQL

 POSTGRESQL Type                          | Doris Type  | Comment                                                               |
|-----------------------------------------|-------------|-----------------------------------------------------------------------|
| boolean                                 | BOOLEAN     |                                                                       |
| smallint/int2                           | SMALLINT    |                                                                       | 
| integer/int4                            | INT         |                                                                       |
| bigint/int8                             | BIGINT      |                                                                       |
| decimal/numeric                         | DECIMAL     |                                                                       |
| real/float4                             | FLOAT       |                                                                       |
| double precision                        | DOUBLE      |                                                                       |
| smallserial                             | SMALLINT    |                                                                       |
| serial                                  | INT         |                                                                       |
| bigserial                               | BIGINT      |                                                                       |
| char                                    | CHAR        |                                                                       |
| varchar/text                            | STRING      |                                                                       |
| timestamp                               | DATETIME    |                                                                       |
| date                                    | DATE        |                                                                       |
| time                                    | STRING      |                                                                       |
| interval                                | STRING      |                                                                       |
| point/line/lseg/box/path/polygon/circle | STRING      |                                                                       |
| cidr/inet/macaddr                       | STRING      |                                                                       |
| bit/bit(n)/bit varying(n)               | STRING      | `bit`类型映射为 doris 的`STRING`类型，读出的数据是`true/false`, 而不是`1/0` |
| uuid/josnb                              | STRING      |                                                                       |
| Other                                   | UNSUPPORTED |                                                                       |

### Oracle

| ORACLE Type                       | Doris Type                           | Comment                                                                                                                                        |
|-----------------------------------|--------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------—｜
| number(p) / number(p,0)           | TINYINT/SMALLINT/INT/BIGINT/LARGEINT | Doris 会根据 p 的大小来选择对应的类型：`p < 3` -> `TINYINT`; `p < 5` -> `SMALLINT`; `p < 10` -> `INT`; `p < 19` -> `BIGINT`; `p > 19` -> `LARGEINT` |
| number(p,s), [ if(s>0 && p>s) ]   | DECIMAL(p,s)                         |                                                                                                                                                |
| number(p,s), [ if(s>0 && p < s) ] | DECIMAL(s,s)                         |                                                                                                                                                |
| number(p,s), [ if(s<0) ]          | TINYINT/SMALLINT/INT/BIGINT/LARGEINT | s<0 的情况下，Doris 会将 p 设置为 p+\|s\|，并进行和 number(p) / number(p,0) 一样的映射                                                                |
| number                            |                                      | Doris 目前不支持未指定 p 和 s 的 oracle 类型                                                                                                       |
| decimal                           | DECIMAL                              |                                                                                                                                                |
| float/real                        | DOUBLE                               |                                                                                                                                                |
| DATE                              | DATETIME                             |                                                                                                                                                |
| TIMESTAMP                         | DATETIME                             |                                                                                                                                                |
| CHAR/NCHAR                        | STRING                               |                                                                                                                                                |
| VARCHAR2/NVARCHAR2                | STRING                               |                                                                                                                                                |
| LONG/ RAW/ LONG RAW/ INTERVAL     | STRING                               |                                                                                                                                                |
| Other                             | UNSUPPORTED                          |                                                                                                                                                |

### SQLServer

| SQLServer Type                         | Doris Type    | Comment                                                  |
|----------------------------------------|---------------|----------------------------------------------------------|
| bit                                    | BOOLEAN       |                                                          |
| tinyint                                | SMALLINT      | SQLServer 的 tinyint 是无符号数，所以映射为 Doris 的 SMALLINT |
| smallint                               | SMALLINT      |                                                          |
| int                                    | INT           |                                                          |
| bigint                                 | BIGINT        |                                                          |
| real                                   | FLOAT         |                                                          |
| float                                  | DOUBLE        |                                                          |
| money                                  | DECIMAL(19,4) |                                                          |
| smallmoney                             | DECIMAL(10,4) |                                                          |
| decimal/numeric                        | DECIMAL       |                                                          |
| date                                   | DATE          |                                                          |
| datetime/datetime2/smalldatetime       | DATETIMEV2    |                                                          |
| char/varchar/text/nchar/nvarchar/ntext | STRING        |                                                          |
| binary/varbinary                       | STRING        |                                                          |
| time/datetimeoffset                    | STRING        |                                                          |
| Other                                  | UNSUPPORTED   |                                                          |


### Clickhouse

| ClickHouse Type                                      | Doris Type                  | Comment                                                                                  |
|------------------------------------------------------|-----------------------------|------------------------------------------------------------------------------------------|
| Bool                                                 | BOOLEAN                     |                                                                                          |
| String                                               | STRING                      |                                                                                          |
| Date/Date32                                          | DATEV2                      | Jdbc Catlog 连接 ClickHouse 时默认使用 DATEV2 类型                                          |
| DateTime/DateTime64                                  | DATETIMEV2                  | Jdbc Catlog 连接 ClickHouse 时默认使用 DATETIMEV2 类型                                      |
| Float32                                              | FLOAT                       |                                                                                          |
| Float64                                              | DOUBLE                      |                                                                                          |
| Int8                                                 | TINYINT                     |                                                                                          |
| Int16/UInt8                                          | SMALLINT                    | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级                                              |
| Int32/UInt16                                         | INT                         | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级                                              |
| Int64/Uint32                                         | BIGINT                      | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级                                              |
| Int128/UInt64                                        | LARGEINT                    | Doris 没有 UNSIGNED 数据类型，所以扩大一个数量级                                              |
| Int256/UInt128/UInt256                               | STRING                      | Doris 没有这个数量级的数据类型，采用 STRING 处理                                               |
| DECIMAL                                              | DECIMAL/DECIMALV3/STRING    | 将根据 Doris DECIMAL 字段的（precision, scale）和`enable_decimal_conversion`开关选择用何种类型  |
| Enum/IPv4/IPv6/UUID                                  | STRING                      | 在显示上 IPv4,IPv6 会额外在数据最前面显示一个`/`,需要自己用`split_part`函数处理                   |
|   Array                                              | ARRAY                       | Array 内部类型适配逻辑参考上述类型，不支持嵌套类型                                               |
| Other                                                | UNSUPPORTED                 |                                                                                           |

### Doris

| Doris Type            | Jdbc Catlog Doris Type    | Comment                                                                               |
|-----------------------|---------------------------|-----------------------------------------------------------------------------------------|
| BOOLEAN               | BOOLEAN                   | |
| TINYINT               | TINYINT                   | |
| SMALLINT              | SMALLINT                  | |
| INT                   | INT                       | |
| BIGINT                | BIGINT                    | |
| LARGEINT              | LARGEINT                  | |
| FLOAT                 | FLOAT                     | |
| DOUBLE                | DOUBLE                    | |
| DECIMAL / DECIMALV3   | DECIMAL/DECIMALV3/STRING | 将根据 Doris DECIMAL 字段的（precision, scale）和`enable_decimal_conversion`开关选择用何种类型 |
| DATE                  | DATEV2                   | Jdbc Catlog 连接 Doris 时默认使用 DATEV2 类型                                                |
| DATEV2                | DATEV2                    |  |
| DATETIME              | DATETIMEV2                | Jdbc Catlog 连接 Doris 时默认使用 DATETIMEV2 类型                                            |
| DATETIMEV2            | DATETIMEV2               | |
| CHAR                  | CHAR                      | |
| VARCHAR               | VARCHAR                   | |
| STRING                | STRING                    | |
| TEXT                  | STRING                    | |
| Other                 | UNSUPPORTED               |

### SAP HANA

| SAP HANA Type  | Doris Type               | Comment                                                                               |
|----------------|--------------------------|---------------------------------------------------------------------------------------|
| BOOLEAN        | BOOLEAN                  |                                                                                       |
| TINYINT        | TINYINT                  |                                                                                       |
| SMALLINT       | SMALLINT                 |                                                                                       |
| INTERGER       | INT                      |                                                                                       |
| BIGINT         | BIGINT                   |                                                                                       |
| SMALLDECIMAL   | DECIMALV3                |                                                                                       |
| DECIMAL        | DECIMAL/DECIMALV3/STRING | 将根据 Doris DECIMAL 字段的（precision, scale）和`enable_decimal_conversion`开关选择用何种类型 |
| REAL           | FLOAT                    |                                                                                       |
| DOUBLE         | DOUBLE                   |                                                                                       |
| DATE           | DATEV2                   | Jdbc Catlog 连接 HANA 时默认使用 DATEV2 类型                                                  |
| TIME           | TEXT                     |                                                                                       |
| TIMESTAMP      | DATETIMEV2               | Jdbc Catlog 连接 HANA 时默认使用 DATETIMEV2 类型                                              |
| SECONDDATE     | DATETIMEV2               | Jdbc Catlog 连接 HANA 时默认使用 DATETIMEV2 类型                                              |
| VARCHAR        | TEXT                     |                                                                                       |
| NVARCHAR       | TEXT                     |                                                                                       |
| ALPHANUM       | TEXT                     |                                                                                       |
| SHORTTEXT      | TEXT                     |                                                                                       |
| CHAR           | CHAR                     |                                                                                       |
| NCHAR          | CHAR                     |                                                                                       |

### Trino

| Trino Type                                           | Doris Type                     | Comment                                                                   |
|------------------------------------------------------|-------------------------------|---------------------------------------------------------------------------|
| boolean                                              | BOOLEAN                       |                                                                           |
| tinyint                                              | TINYINT                       |                                                                           |
| smallint                                             | SMALLINT                      |                                                                           |
| integer                                              | INT                           |                                                                           |
| bigint                                               | BIGINT                        |                                                                           |
| decimal                                              | DECIMAL/DECIMALV3/STRING      | 将根据 Doris DECIMAL 字段的（precision, scale）和`enable_decimal_conversion`开关选择用何种类型|
| real                                                 | FLOAT                         |                                                                           |
| double                                               | DOUBLE                        |                                                                           |
| date                                                 | DATE/DATEV2                   | Jdbc Catlog 连接 Trino 时默认使用 DATEV2 类型                                      |
| timestamp                                            | DATETIME/DATETIMEV2           | Jdbc Catlog 连接 Trino 时默认使用 DATETIMEV2 类型                                  |
| varchar                                              | TEXT                          |                                                                           |
| char                                                 | CHAR                          |                                                                           |
|   array                                              | ARRAY                         | Array 内部类型适配逻辑参考上述类型，不支持嵌套类型                                  |
| others                                               | UNSUPPORTED                   |                                                                           |

**Note:**
目前仅针对 Trino 连接的 Hive 做了测试，其他的 Trino 连接的数据源暂时未测试。

### OceanBase

MySQL 模式请参考 [MySQL 类型映射](#MySQL)
Oracle 模式请参考 [Oracle 类型映射](#Oracle)

## 常见问题

1. 除了 MySQL,Oracle,PostgreSQL,SQLServer,ClickHouse,SAP HANA,Trino,OceanBase 是否能够支持更多的数据库

    目前 Doris 只适配了 MySQL,Oracle,PostgreSQL,SQLServer,ClickHouse,SAP HANA,Trino,OceanBase. 关于其他的数据库的适配工作正在规划之中，原则上来说任何支持 JDBC 访问的数据库都能通过 JDBC 外表来访问。如果您有访问其他外表的需求，欢迎修改代码并贡献给 Doris。

2. 读写 MySQL 外表的 emoji 表情出现乱码

    Doris 进行 jdbc 外表连接时，由于 mysql 之中默认的 utf8 编码为 utf8mb3，无法表示需要 4 字节编码的 emoji 表情。这里需要在建立 mysql 外表时设置对应列的编码为 utf8mb4，设置服务器编码为 utf8mb4,JDBC Url 中的 characterEncoding 不配置.（该属性不支持 utf8mb4，配置了非 utf8mb4 将导致无法写入表情，因此要留空，不配置）

    可全局修改配置项
    
```
    修改 mysql 目录下的 my.ini 文件（linux 系统为 etc 目录下的 my.cnf 文件）
    [client]
    default-character-set=utf8mb4
    
    [mysql]
    设置 mysql 默认字符集
    default-character-set=utf8mb4
    
    [mysqld]
    设置 mysql 字符集服务器
    character-set-server=utf8mb4
    collation-server=utf8mb4_unicode_ci
    init_connect='SET NAMES utf8mb4
    
    修改对应表与列的类型
    ALTER TABLE table_name MODIFY  colum_name  VARCHAR(100) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    ALTER TABLE table_name CHARSET=utf8mb4;
    SET NAMES utf8mb4
```

3. 读 MySQL 外表时，DateTime="0000:00:00 00:00:00"异常报错："CAUSED BY: DataReadException: Zero date value prohibited"

    这是因为 JDBC 中对于该非法的 DateTime 默认处理为抛出异常，可以通过参数 `zeroDateTimeBehavior`控制该行为。
    
    可选参数为：`EXCEPTION`,`CONVERT_TO_NULL`,`ROUND`, 分别为：异常报错，转为 NULL 值，转为 "0001-01-01 00:00:00";
    
    可在 url 中添加：`"jdbc_url"="jdbc:mysql://IP:PORT/doris_test?zeroDateTimeBehavior=convertToNull"`

4. 读取 MySQL 外表或其他外表时，出现加载类失败

    如以下异常：
 
```
    failed to load driver class com.mysql.jdbc.driver in either of hikariconfig class loader
```
 
    这是因为在创建 resource 时，填写的 driver_class 不正确，需要正确填写，如上方例子为大小写问题，应填写为 `"driver_class" = "com.mysql.jdbc.Driver"`

5. 读取 MySQL 问题出现通信链路异常

    如果出现如下报错：

```
    ERROR 1105 (HY000): errCode = 2, detailMessage = PoolInitializationException: Failed to initialize pool: Communications link failure
    
    The last packet successfully received from the server was 7 milliseconds ago.  The last packet sent successfully to the server was 4 milliseconds ago.
    CAUSED BY: CommunicationsException: Communications link failure
        
    The last packet successfully received from the server was 7 milliseconds ago.  The last packet sent successfully to the server was 4 milliseconds ago.
    CAUSED BY: SSLHandshakeExcepti
```
    
    可查看 be 的 be.out 日志
    
    如果包含以下信息：
    
```
    WARN: Establishing SSL connection without server's identity verification is not recommended. 
    According to MySQL 5.5.45+, 5.6.26+ and 5.7.6+ requirements SSL connection must be established by default if explicit option isn't set. 
    For compliance with existing applications not using SSL the verifyServerCertificate property is set to 'false'. 
    You need either to explicitly disable SSL by setting useSSL=false, or set useSSL=true and provide truststore for server certificate verification.
```

    可在创建 Catalog 的 `jdbc_url` 把 JDBC 连接串最后增加 `?useSSL=false` ,如 `"jdbc_url" = "jdbc:mysql://127.0.0.1:3306/test?useSSL=false"`

6. 查询 MYSQL 的数据库报 OutOfMemoryError 的错误

    为减少内存的使用，在获取结果集时，每次仅获取 batchSize 的大小，这样一批一批的获取结果。而 MYSQL 默认是一次将结果全部加载到内存，
    设置的按批获取无法生效，需要主动显示的在 URL 中指定："jdbc_url"="jdbc:mysql://IP:PORT/doris_test?useCursorFetch=true"

7. 在使用 JDBC 查询过程中时，如果出现"CAUSED BY: SQLException OutOfMemoryError" 类似的错误

    如果 MYSQL 已经主动设置 useCursorFetch，可以在 be.conf 中修改 jvm_max_heap_size 的值，尝试增大 JVM 的内存，目前默认值为 1024M。

8. 使用 JDBC 查询 MYSQL 大数据量时，如果查询偶尔能够成功，偶尔会报如下错误，且出现该错误时 MYSQL 的连接被全部断开，无法连接到 MYSQL SERVER，过段时间后 mysql 又恢复正常，但是之前的连接都没了：

```
    ERROR 1105 (HY000): errCode = 2, detailMessage = [INTERNAL_ERROR]UdfRuntimeException: JDBC executor sql has error:
    CAUSED BY: CommunicationsException: Communications link failure
    The last packet successfully received from the server was 4,446 milliseconds ago. The last packet sent successfully to the server was 4,446 milliseconds ago.
```

    出现上述现象时，可能是 Mysql Server 自身的内存或 CPU 资源被耗尽导致 Mysql 服务不可用，可以尝试增大 Mysql Server 的内存或 CPU 配置。
 
