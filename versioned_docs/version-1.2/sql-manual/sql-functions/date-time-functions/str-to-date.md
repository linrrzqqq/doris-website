---
{
    "title": "STR_TO_DATE",
    "language": "en"
}
---

## Str_to_date
### Description
#### Syntax

`DATETIME STR TWO DATES (VARCHAR STR, VARCHAR format)`


Convert STR to DATE type by format specified, if the conversion result does not return NULL. Note that the 'format' parameter specifies the format of the first parameter.

The `format` supported is consistent with [date_format](date-format.md)

### example

```
mysql> select str_to_date('2014-12-21 12:34:56', '%Y-%m-%d %H:%i:%s');
+---------------------------------------------------------+
| str_to_date('2014-12-21 12:34:56', '%Y-%m-%d %H:%i:%s') |
+---------------------------------------------------------+
| 2014-12-21 12:34:56                                     |
+---------------------------------------------------------+

mysql> select str_to_date('2014-12-21 12:34%3A56', '%Y-%m-%d %H:%i%%3A%s');
+--------------------------------------------------------------+
| str_to_date('2014-12-21 12:34%3A56', '%Y-%m-%d %H:%i%%3A%s') |
+--------------------------------------------------------------+
| 2014-12-21 12:34:56                                          |
+--------------------------------------------------------------+

mysql> select str_to_date('200442 Monday', '%X%V %W');
+-----------------------------------------+
| str_to_date('200442 Monday', '%X%V %W') |
+-----------------------------------------+
| 2004-10-18                              |
+-----------------------------------------+

mysql> select str_to_date("2020-09-01", "%Y-%m-%d %H:%i:%s");
+------------------------------------------------+
| str_to_date('2020-09-01', '%Y-%m-%d %H:%i:%s') |
+------------------------------------------------+
| 2020-09-01 00:00:00                            |
+------------------------------------------------+
1 row in set (0.01 sec)
```
### keywords

    STR_TO_DATE,STR,TO,DATE
