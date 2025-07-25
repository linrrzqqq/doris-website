---
{
    "title": "SHOW-FUNCTIONS",
    "language": "zh-CN"
}
---

## SHOW-FUNCTIONS

### Name

SHOW FUNCTIONS

## 描述

查看数据库下所有的自定义(系统提供)的函数。如果用户指定了数据库，那么查看对应数据库的，否则直接查询当前会话所在数据库

需要对这个数据库拥有 `SHOW` 权限

语法

```sql
SHOW [FULL] [BUILTIN] FUNCTIONS [IN|FROM db] [LIKE 'function_pattern']
```

Parameters

>`full`:表示显示函数的详细信息
>`builtin`:表示显示系统提供的函数
>`db`: 要查询的数据库名字
>`function_pattern`: 用来过滤函数名称的参数

语法

```sql
SHOW GLOBAL [FULL] FUNCTIONS [LIKE 'function_pattern']
```

Parameters

>`global`:表示要展示的是全局函数
>`full`:表示显示函数的详细信息
>`function_pattern`: 用来过滤函数名称的参数

**注意: "global"关键字在v2.0版本及以后才可用**

## 举例

```sql
mysql> show full functions in testDb\G
*************************** 1. row ***************************
        Signature: my_add(INT,INT)
      Return Type: INT
    Function Type: Scalar
Intermediate Type: NULL
       Properties: {"symbol":"_ZN9doris_udf6AddUdfEPNS_15FunctionContextERKNS_6IntValES4_","object_file":"http://host:port/libudfsample.so","md5":"cfe7a362d10f3aaf6c49974ee0f1f878"}
*************************** 2. row ***************************
        Signature: my_count(BIGINT)
      Return Type: BIGINT
    Function Type: Aggregate
Intermediate Type: NULL
       Properties: {"object_file":"http://host:port/libudasample.so","finalize_fn":"_ZN9doris_udf13CountFinalizeEPNS_15FunctionContextERKNS_9BigIntValE","init_fn":"_ZN9doris_udf9CountInitEPNS_15FunctionContextEPNS_9BigIntValE","merge_fn":"_ZN9doris_udf10CountMergeEPNS_15FunctionContextERKNS_9BigIntValEPS2_","md5":"37d185f80f95569e2676da3d5b5b9d2f","update_fn":"_ZN9doris_udf11CountUpdateEPNS_15FunctionContextERKNS_6IntValEPNS_9BigIntValE"}
*************************** 3. row ***************************
        Signature: id_masking(BIGINT)
      Return Type: VARCHAR
    Function Type: Alias
Intermediate Type: NULL
       Properties: {"parameter":"id","origin_function":"concat(left(`id`, 3), `****`, right(`id`, 4))"}

3 rows in set (0.00 sec)
mysql> show builtin functions in testDb like 'year%';
+---------------+
| Function Name |
+---------------+
| year          |
| years_add     |
| years_diff    |
| years_sub     |
+---------------+
2 rows in set (0.00 sec)
    
mysql> show global full functions\G;
*************************** 1. row ***************************
        Signature: decimal(ALL, INT, INT)
      Return Type: VARCHAR
    Function Type: Alias
Intermediate Type: NULL
       Properties: {"parameter":"col, precision, scale","origin_function":"CAST(`col` AS decimal(`precision`, `scale`))"}
*************************** 2. row ***************************
        Signature: id_masking(BIGINT)
      Return Type: VARCHAR
    Function Type: Alias
Intermediate Type: NULL
       Properties: {"parameter":"id","origin_function":"concat(left(`id`, 3), `****`, right(`id`, 4))"}
2 rows in set (0.00 sec)
    
mysql> show global functions ;
+---------------+
| Function Name |
+---------------+
| decimal       |
| id_masking    |
+---------------+
2 rows in set (0.00 sec)

```

### Keywords

    SHOW, FUNCTIONS

### Best Practice

