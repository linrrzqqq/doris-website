---
{
    "title": "DROP-REPOSITORY",
    "language": "zh-CN"
}
---

## DROP-REPOSITORY

### Name

DROP  REPOSITORY

## 描述

该语句用于删除一个已创建的仓库。仅 root 或 superuser 用户可以删除仓库。

语法：

```sql
DROP REPOSITORY `repo_name`;
```

说明：

- 删除仓库，仅仅是删除该仓库在 Palo 中的映射，不会删除实际的仓库数据。删除后，可以再次通过指定相同的 LOCATION 映射到该仓库。

## 举例

1. 删除名为 bos_repo 的仓库：

```sql
DROP REPOSITORY `bos_repo`;
```

### Keywords

    DROP, REPOSITORY

### Best Practice

