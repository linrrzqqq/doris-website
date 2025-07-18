---
{
    "title": "ST_GEOMETRYFROMWKB",
    "language": "en"
}
---

## Description

Converts a standard figure WKB (Well-known binary) to the corresponding memory geometry

## Alias

- ST_GEOMFROMWKB

## Syntax

```sql
ST_GEOMETRYFROMWKB( <wkb>)
```
## Parameters

| Parameters | Instructions |
|------------|---------|
| `<wkb>`  | The memory form of the graph |

## Return Value

The corresponding geometric storage form of WKB

## Examples

```sql
select ST_AsText(ST_GeometryFromWKB(ST_AsBinary(ST_Point(24.7, 56.7))));
```

```text
+------------------------------------------------------------------+
| st_astext(st_geometryfromwkb(st_asbinary(st_point(24.7, 56.7)))) |
+------------------------------------------------------------------+
| POINT (24.7 56.7)                                                |
+------------------------------------------------------------------+
```

```sql
select ST_AsText(ST_GeomFromWKB(ST_AsBinary(ST_Point(24.7, 56.7))));
```

```text
+--------------------------------------------------------------+
| st_astext(st_geomfromwkb(st_asbinary(st_point(24.7, 56.7)))) |
+--------------------------------------------------------------+
| POINT (24.7 56.7)                                            |
+--------------------------------------------------------------+
```

```sql
select ST_AsText(ST_GeometryFromWKB(ST_AsBinary(ST_GeometryFromText("LINESTRING (1 1, 2 2)"))));
```

```text
+------------------------------------------------------------------------------------------+
| st_astext(st_geometryfromwkb(st_asbinary(st_geometryfromtext('LINESTRING (1 1, 2 2)')))) |
+------------------------------------------------------------------------------------------+
| LINESTRING (1 1, 2 2)                                                                    |
+------------------------------------------------------------------------------------------+
```

```sql
select ST_AsText(ST_GeometryFromWKB(ST_AsBinary(ST_Polygon("POLYGON ((114.104486 22.547119,114.093758 22.547753,114.096504 22.532057,114.104229 22.539826,114.106203 22.542680,114.104486 22.547119))"))));
```

```text
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| st_astext(st_geometryfromwkb(st_asbinary(st_polygon('POLYGON ((114.104486 22.547119,114.093758 22.547753,114.096504 22.532057,114.104229 22.539826,114.106203 22.542680,114.104486 22.547119))')))) |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| POLYGON ((114.104486 22.547119, 114.093758 22.547753, 114.096504 22.532057, 114.104229 22.539826, 114.106203 22.54268, 114.104486 22.547119))                                                       |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

```sql
select ST_AsText(ST_GeomFromWKB(ST_AsBinary(ST_Polygon("POLYGON ((114.104486 22.547119,114.093758 22.547753,114.096504 22.532057,114.104229 22.539826,114.106203 22.542680,114.104486 22.547119))"))));
```

```text
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| st_astext(st_geomfromwkb(st_asbinary(st_polygon('POLYGON ((114.104486 22.547119,114.093758 22.547753,114.096504 22.532057,114.104229 22.539826,114.106203 22.542680,114.104486 22.547119))')))) |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| POLYGON ((114.104486 22.547119, 114.093758 22.547753, 114.096504 22.532057, 114.104229 22.539826, 114.106203 22.54268, 114.104486 22.547119))                                                   |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```