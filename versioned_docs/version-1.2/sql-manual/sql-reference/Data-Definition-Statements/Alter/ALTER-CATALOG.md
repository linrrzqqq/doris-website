---
{
    "title": "ALTER-CATALOG",
    "language": "en"
}
---

## ALTER-CATALOG

### Name

<version since="1.2">

ALTER CATALOG

</version>

### Description

This statement is used to set properties of the specified catalog. (administrator only)

1) Rename the catalog

```sql
ALTER CATALOG catalog_name RENAME new_catalog_name;
```

illustrate:
- The builtin catalog `internal` cannot be renamed
- Only the one who has at least Alter privilege can rename a catalog
- After renaming the catalog, use the REVOKE and GRANT commands to modify the appropriate user permissions

2) Modify / add properties for the catalog

```sql
ALTER CATALOG catalog_name SET PROPERTIES ('key1' = 'value1' [, 'key' = 'value2']); 
```

Update values of specified keys. If a key does not exist in the catalog properties, it will be added. 

illustrate:
- property `type` cannot be modified.
- properties of builtin catalog `internal` cannot be modified.

### Example

1. rename catalog ctlg_hive to hive

```sql
ALTER CATALOG ctlg_hive RENAME hive;
```

3. modify property `hive.metastore.uris` of catalog hive

```sql
ALTER CATALOG hive SET PROPERTIES ('hive.metastore.uris'='thrift://172.21.0.1:9083');
```

### Keywords

ALTER,CATALOG,RENAME,PROPERTY

### Best Practice

