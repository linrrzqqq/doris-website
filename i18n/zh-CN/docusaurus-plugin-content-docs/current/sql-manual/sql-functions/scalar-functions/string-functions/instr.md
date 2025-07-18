---
{
    "title": "INSTR",
    "language": "zh-CN"
}
---

## 描述

返回 substr 在 str 中第一次出现的位置（从 1 开始计数）。特殊情况：

- 如果 substr 不在 str 中出现，则返回 0。

## 语法

```sql
INSTR ( <str> , <substr> )
```

## 参数

|参数     | 说明        |
|-------|-----------|
| `<str>`  | 需要查找的字符串  |
| `<substr>` | 需要被查找的字符串 |

## 返回值

参数 `<substr>` 在 `<str>` 中第一次出现的位置（从 1 开始计数）。特殊情况：

- 如果 `<substr>` 不在 `<str>` 中出现，则返回 0。

## 举例

```sql
SELECT INSTR("abc", "b"),INSTR("abc", "d")
```

```text
+-------------------+-------------------+
| instr('abc', 'b') | instr('abc', 'd') |
+-------------------+-------------------+
|                 2 |                 0 |
+-------------------+-------------------+
```