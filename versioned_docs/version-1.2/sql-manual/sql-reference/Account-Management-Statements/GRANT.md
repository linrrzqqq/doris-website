---
{
    "title": "GRANT",
    "language": "en"
}
---

## GRANT

### Name

GRANT

### Description

The GRANT command has the following functions:

1. Grant the specified permissions to a user or role.
2. Grant the specified role to a user.

>Note that.
>
>"Grant specified roles to user" is supported in versions 2.0 and later

```sql
GRANT privilege_list ON priv_level TO user_identity [ROLE role_name]

GRANT privilege_list ON RESOURCE resource_name TO user_identity [ROLE role_name]

GRANT role_list TO user_identity
```

privilege_list is a list of privileges to be granted, separated by commas. Currently Doris supports the following permissions:

    NODE_PRIV: Cluster node operation permissions, including node online and offline operations. Only the root user has this permission and cannot be granted to other users.
    ADMIN_PRIV: All privileges except NODE_PRIV.
    GRANT_PRIV: Privilege for operation privileges. Including creating and deleting users, roles, authorization and revocation, setting passwords, etc.
    SELECT_PRIV: read permission on the specified database or table
    LOAD_PRIV: Import privileges on the specified database or table
    ALTER_PRIV: Schema change permission for the specified database or table
    CREATE_PRIV: Create permission on the specified database or table
    DROP_PRIV: drop privilege on the specified database or table
    USAGE_PRIV: access to the specified resource
    
    ALL and READ_WRITE in legacy permissions will be converted to: SELECT_PRIV,LOAD_PRIV,ALTER_PRIV,CREATE_PRIV,DROP_PRIV;
    READ_ONLY is converted to SELECT_PRIV.

Permission classification:

    1. Node Privilege: NODE_PRIV
    2. database table permissions: SELECT_PRIV, LOAD_PRIV, ALTER_PRIV, CREATE_PRIV, DROP_PRIV
    3. Resource permission: USAGE_PRIV

Priv_level supports the following four forms:

    1. *.*.* permissions can be applied to all catalogs, all databases and all tables in them
    2. ctl.*.* permissions can be applied to all databases and all tables in them
    3. ctl.db.* permissions can be applied to all tables under the specified database
    4. ctl.db.tbl permission can be applied to the specified table under the specified database
    
    The catalog or database, table specified here may be not exist.

resource_name supports the following two forms:

    1. * Permissions apply to all resources
    2. The resource permission applies to the specified resource
    
    The resource specified here can be a non-existing resource.

user_identity:

    The user_identity syntax here is the same as CREATE USER. And must be a user_identity created with CREATE USER. The host in user_identity can be a domain name. If it is a domain name, the effective time of the authority may be delayed by about 1 minute.
    
    You can also assign permissions to the specified ROLE, if the specified ROLE does not exist, it will be created automatically.

role_list is the list of roles to be assigned, separated by commas,the specified role must exist.

### Example

1. Grant permissions to all catalog and databases and tables to the user

   ```sql
   GRANT SELECT_PRIV ON *.*.* TO 'jack'@'%';
   ```

2. Grant permissions to the specified database table to the user

   ```sql
   GRANT SELECT_PRIV,ALTER_PRIV,LOAD_PRIV ON ctl1.db1.tbl1 TO 'jack'@'192.8.%';
   ```

3. Grant permissions to the specified database table to the role

   ```sql
   GRANT LOAD_PRIV ON ctl1.db1.* TO ROLE 'my_role';
   ```

4. Grant access to all resources to users

   ```sql
   GRANT USAGE_PRIV ON RESOURCE * TO 'jack'@'%';
   ```

5. Grant the user permission to use the specified resource

   ```sql
   GRANT USAGE_PRIV ON RESOURCE 'spark_resource' TO 'jack'@'%';
   ```

6. Grant access to specified resources to roles

   ```sql
   GRANT USAGE_PRIV ON RESOURCE 'spark_resource' TO ROLE 'my_role';
   ```

7. Grant the specified role to a user

    ```sql
    GRANT 'role1','role2' TO 'jack'@'%';
    ```

### Keywords

    GRANT

### Best Practice

