---
{
"title": "SHA1",
"language": "zh-CN"
}
---

## 描述

使用 SHA1 算法对信息进行摘要处理。

## 别名
SHA

## 语法

``` sql
SHA1( <str> )
```

## 参数

| 参数      | 说明          |
|---------|-------------|
| `<str>` | 需要被计算 sha1 的值 |

## 返回值

返回输入字符串的 sha1 值


## 示例

```sql
select sha("123"), sha1("123");
```

```text
+------------------------------------------+------------------------------------------+
| sha1('123')                              | sha1('123')                              |
+------------------------------------------+------------------------------------------+
| 40bd001563085fc35165329ea1ff5c5ecbdbbeef | 40bd001563085fc35165329ea1ff5c5ecbdbbeef |
+------------------------------------------+------------------------------------------+
```
