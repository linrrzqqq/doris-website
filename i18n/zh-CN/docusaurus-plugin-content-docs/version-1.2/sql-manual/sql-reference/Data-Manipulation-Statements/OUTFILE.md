---
{
    "title": "OUTFILE",
    "language": "zh-CN"
}
---

## OUTFILE

### Name

OURFILE

## 描述

`SELECT INTO OUTFILE` 命令用于将查询结果导出为文件。目前支持通过 Broker 进程, 通过 S3 协议, 或直接通过 HDFS 协议，导出到远端存储，如 HDFS，S3，BOS，COS（腾讯云）上。
    
语法：

```
query_stmt
INTO OUTFILE "file_path"
[format_as]
[properties]
```

说明：

1. file_path
   
    ```
    file_path 指向文件存储的路径以及文件前缀。如 `hdfs://path/to/my_file_`。
    
    最终的文件名将由 `my_file_`，文件序号以及文件格式后缀组成。其中文件序号由0开始，数量为文件被分割的数量。如：
    my_file_abcdefg_0.csv
    my_file_abcdefg_1.csv
    my_file_abcdegf_2.csv
    ```
    
2. format_as

    ```
    FORMAT AS CSV
    ```

    指定导出格式. 支持 CSV、PARQUET、CSV_WITH_NAMES、CSV_WITH_NAMES_AND_TYPES、ORC. 默认为 CSV。

    > 注：PARQUET、CSV_WITH_NAMES、CSV_WITH_NAMES_AND_TYPES、ORC 在 1.2 版本开始支持。
    
3. properties
   
    ```
    指定相关属性。目前支持通过 Broker 进程, 或通过 S3 协议进行导出。
    
    语法：
    [PROPERTIES ("key"="value", ...)]
    支持如下属性：
    column_separator: 列分隔符。<version since="1.2.0">支持多字节分隔符，如："\\x01", "abc"</version>
    line_delimiter: 行分隔符。<version since="1.2.0">支持多字节分隔符，如："\\x01", "abc"</version>
    max_file_size: 单个文件大小限制，如果结果超过这个值，将切割成多个文件。
    
    Broker 相关属性需加前缀 `broker.`:
    broker.name: broker名称
    broker.hadoop.security.authentication: 指定认证方式为 kerberos
    broker.kerberos_principal: 指定 kerberos 的 principal
    broker.kerberos_keytab: 指定 kerberos 的 keytab 文件路径。该文件必须为 Broker 进程所在服务器上的文件的绝对路径。并且可以被 Broker 进程访问
    
    HDFS 相关属性:
    fs.defaultFS: namenode 地址和端口
    hadoop.username: hdfs 用户名
    dfs.nameservices: name service名称，与hdfs-site.xml保持一致
    dfs.ha.namenodes.[nameservice ID]: namenode的id列表,与hdfs-site.xml保持一致
    dfs.namenode.rpc-address.[nameservice ID].[name node ID]: Name node的rpc地址，数量与namenode数量相同，与hdfs-site.xml保持一致
    dfs.client.failover.proxy.provider.[nameservice ID]: HDFS客户端连接活跃namenode的java类，通常是"org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"

    对于开启kerberos认证的Hadoop 集群，还需要额外设置如下 PROPERTIES 属性:
    dfs.namenode.kerberos.principal: HDFS namenode 服务的 principal 名称
    hadoop.security.authentication: 认证方式设置为 kerberos
    hadoop.kerberos.principal: 设置 Doris 连接 HDFS 时使用的 Kerberos 主体
    hadoop.kerberos.keytab: 设置 keytab 本地文件路径

    S3 协议则直接执行 S3 协议配置即可:
    AWS_ENDPOINT
    AWS_ACCESS_KEY
    AWS_SECRET_KEY
    AWS_REGION
    use_path_stype: (选填) 默认为false 。S3 SDK 默认使用 virtual-hosted style 方式。但某些对象存储系统可能没开启或不支持virtual-hosted style 方式的访问，此时可以添加 use_path_style 参数来强制使用 path style 访问方式。
    ```

## 举例

1. 使用 broker 方式导出，将简单查询结果导出到文件 `hdfs://path/to/result.txt`。指定导出格式为 CSV。使用 `my_broker` 并设置 kerberos 认证信息。指定列分隔符为 `,`，行分隔符为 `\n`。
    
    ```sql
    SELECT * FROM tbl
    INTO OUTFILE "hdfs://path/to/result_"
    FORMAT AS CSV
    PROPERTIES
    (
        "broker.name" = "my_broker",
        "broker.hadoop.security.authentication" = "kerberos",
        "broker.kerberos_principal" = "doris@YOUR.COM",
        "broker.kerberos_keytab" = "/home/doris/my.keytab",
        "column_separator" = ",",
        "line_delimiter" = "\n",
        "max_file_size" = "100MB"
    );
    ```
    
    最终生成文件如如果不大于 100MB，则为：`result_0.csv`。
    如果大于 100MB，则可能为 `result_0.csv, result_1.csv, ...`。
    
2. 将简单查询结果导出到文件 `hdfs://path/to/result.parquet`。指定导出格式为 PARQUET。使用 `my_broker` 并设置 kerberos 认证信息。
    
    ```sql
    SELECT c1, c2, c3 FROM tbl
    INTO OUTFILE "hdfs://path/to/result_"
    FORMAT AS PARQUET
    PROPERTIES
    (
        "broker.name" = "my_broker",
        "broker.hadoop.security.authentication" = "kerberos",
        "broker.kerberos_principal" = "doris@YOUR.COM",
        "broker.kerberos_keytab" = "/home/doris/my.keytab",
        "schema"="required,int32,c1;required,byte_array,c2;required,byte_array,c2"
    );
    ```
    
    查询结果导出到parquet文件需要明确指定`schema`。
    
3. 将 CTE 语句的查询结果导出到文件 `hdfs://path/to/result.txt`。默认导出格式为 CSV。使用 `my_broker` 并设置 hdfs 高可用信息。使用默认的行列分隔符。
    
    ```sql
    WITH
    x1 AS
    (SELECT k1, k2 FROM tbl1),
    x2 AS
    (SELECT k3 FROM tbl2)
    SELEC k1 FROM x1 UNION SELECT k3 FROM x2
    INTO OUTFILE "hdfs://path/to/result_"
    PROPERTIES
    (
        "broker.name" = "my_broker",
        "broker.username"="user",
        "broker.password"="passwd",
        "broker.dfs.nameservices" = "my_ha",
        "broker.dfs.ha.namenodes.my_ha" = "my_namenode1, my_namenode2",
        "broker.dfs.namenode.rpc-address.my_ha.my_namenode1" = "nn1_host:rpc_port",
        "broker.dfs.namenode.rpc-address.my_ha.my_namenode2" = "nn2_host:rpc_port",
        "broker.dfs.client.failover.proxy.provider.my_ha" = "org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider"
    );
    ```
    
    最终生成文件如如果不大于 1GB，则为：`result_0.csv`。
    如果大于 1GB，则可能为 `result_0.csv, result_1.csv, ...`。
    
4. 将 UNION 语句的查询结果导出到文件 `bos://bucket/result.txt`。指定导出格式为 PARQUET。使用 `my_broker` 并设置 hdfs 高可用信息。PARQUET 格式无需指定列分割符。
    导出完成后，生成一个标识文件。
    
    ```sql
    SELECT k1 FROM tbl1 UNION SELECT k2 FROM tbl1
    INTO OUTFILE "bos://bucket/result_"
    FORMAT AS PARQUET
    PROPERTIES
    (
        "broker.name" = "my_broker",
        "broker.bos_endpoint" = "http://bj.bcebos.com",
        "broker.bos_accesskey" = "xxxxxxxxxxxxxxxxxxxxxxxxxx",
        "broker.bos_secret_accesskey" = "yyyyyyyyyyyyyyyyyyyyyyyyyy",
        "schema"="required,int32,k1;required,byte_array,k2"
    );
    ```
    
5. 将 select 语句的查询结果导出到文件 `s3a://${bucket_name}/path/result.txt`。指定导出格式为 csv。
    导出完成后，生成一个标识文件。
    
    ```sql
    select k1,k2,v1 from tbl1 limit 100000
    into outfile "s3a://my_bucket/export/my_file_"
    FORMAT AS CSV
    PROPERTIES
    (
        "broker.name" = "hdfs_broker",
        "broker.fs.s3a.access.key" = "xxx",
        "broker.fs.s3a.secret.key" = "xxxx",
        "broker.fs.s3a.endpoint" = "https://cos.xxxxxx.myqcloud.com/",
        "column_separator" = ",",
        "line_delimiter" = "\n",
        "max_file_size" = "1024MB",
        "success_file_name" = "SUCCESS"
    )
    ```
    
    最终生成文件如如果不大于 1GB，则为：`my_file_0.csv`。
    如果大于 1GB，则可能为 `my_file_0.csv, result_1.csv, ...`。
    在cos上验证
    
        1. 不存在的path会自动创建
        2. access.key/secret.key/endpoint需要和cos的同学确认。尤其是endpoint的值，不需要填写bucket_name。
    
6. 使用 s3 协议导出到 bos，并且并发导出开启。
    
    ```sql
    set enable_parallel_outfile = true;
    select k1 from tb1 limit 1000
    into outfile "s3://my_bucket/export/my_file_"
    format as csv
    properties
    (
        "AWS_ENDPOINT" = "http://s3.bd.bcebos.com",
        "AWS_ACCESS_KEY" = "xxxx",
        "AWS_SECRET_KEY" = "xxx",
        "AWS_REGION" = "bd"
    )
    ```
    
    最终生成的文件前缀为 `my_file_{fragment_instance_id}_`。
    
7. 使用 s3 协议导出到 bos，并且并发导出 session 变量开启。
    注意：但由于查询语句带了一个顶层的排序节点，所以这个查询即使开启并发导出的 session 变量，也是无法并发导出的。
    
    ```sql
    set enable_parallel_outfile = true;
    select k1 from tb1 order by k1 limit 1000
    into outfile "s3://my_bucket/export/my_file_"
    format as csv
    properties
    (
        "AWS_ENDPOINT" = "http://s3.bd.bcebos.com",
        "AWS_ACCESS_KEY" = "xxxx",
        "AWS_SECRET_KEY" = "xxx",
        "AWS_REGION" = "bd"
    )
    ```
    
8. 使用 hdfs 方式导出，将简单查询结果导出到文件 `hdfs://${host}:${fileSystem_port}/path/to/result.txt`。指定导出格式为 CSV，用户名为work。指定列分隔符为 `,`，行分隔符为 `\n`。
    
    ```sql
    -- fileSystem_port默认值为9000
    SELECT * FROM tbl
    INTO OUTFILE "hdfs://${host}:${fileSystem_port}/path/to/result_"
    FORMAT AS CSV
    PROPERTIES
    (
        "fs.defaultFS" = "hdfs://ip:port",
        "hadoop.username" = "work"
    );
    ```

    如果Hadoop 集群开启高可用并且启用 Kerberos 认证，可以参考如下SQL语句：

    ```sql
    SELECT * FROM tbl
    INTO OUTFILE "hdfs://path/to/result_"
    FORMAT AS CSV
    PROPERTIES
    (
    'fs.defaultFS'='hdfs://hacluster/',
    'dfs.nameservices'='hacluster',
    'dfs.ha.namenodes.hacluster'='n1,n2',
    'dfs.namenode.rpc-address.hacluster.n1'='192.168.0.1:8020',
    'dfs.namenode.rpc-address.hacluster.n2'='192.168.0.2:8020',
    'dfs.client.failover.proxy.provider.hacluster'='org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider',
    'dfs.namenode.kerberos.principal'='hadoop/_HOST@REALM.COM'
    'hadoop.security.authentication'='kerberos',
    'hadoop.kerberos.principal'='doris_test@REALM.COM',
    'hadoop.kerberos.keytab'='/path/to/doris_test.keytab'
    );
    ```
    
    最终生成文件如如果不大于 100MB，则为：`result_0.csv`。
    如果大于 100MB，则可能为 `result_0.csv, result_1.csv, ...`。

9. 将 select 语句的查询结果导出到腾讯云cos的文件 `cosn://${bucket_name}/path/result.txt`。指定导出格式为 csv。
    导出完成后，生成一个标识文件。

    ```sql
    select k1,k2,v1 from tbl1 limit 100000
    into outfile "cosn://my_bucket/export/my_file_"
    FORMAT AS CSV
    PROPERTIES
    (
        "broker.name" = "broker_name",
        "broker.fs.cosn.userinfo.secretId" = "xxx",
        "broker.fs.cosn.userinfo.secretKey" = "xxxx",
        "broker.fs.cosn.bucket.endpoint_suffix" = "https://cos.xxxxxx.myqcloud.com/",
        "column_separator" = ",",
        "line_delimiter" = "\n",
        "max_file_size" = "1024MB",
        "success_file_name" = "SUCCESS"
    )
    ```

### keywords
    SELECT, INTO, OUTFILE

### Best Practice

1. 导出数据量和导出效率

   该功能本质上是执行一个 SQL 查询命令。最终的结果是单线程输出的。所以整个导出的耗时包括查询本身的耗时，和最终结果集写出的耗时。如果查询较大，需要设置会话变量 `query_timeout` 适当的延长查询超时时间。

2. 导出文件的管理

   Doris 不会管理导出的文件。包括导出成功的，或者导出失败后残留的文件，都需要用户自行处理。

3. 导出到本地文件

   导出到本地文件的功能不适用于公有云用户，仅适用于私有化部署的用户。并且默认用户对集群节点有完全的控制权限。Doris 对于用户填写的导出路径不会做合法性检查。如果 Doris 的进程用户对该路径无写权限，或路径不存在，则会报错。同时出于安全性考虑，如果该路径已存在同名的文件，也会导出失败。

   Doris 不会管理导出到本地的文件，也不会检查磁盘空间等。这些文件需要用户自行管理，如清理等。

4. 结果完整性保证

   该命令是一个同步命令，因此有可能在执行过程中任务连接断开了，从而无法获知导出任务是否正常结束及导出数据是否完整。此时可以使用 `success_file_name` 参数要求任务成功后，在目录下生成一个成功文件标识。用户可以通过这个文件，来判断导出是否正常结束。
