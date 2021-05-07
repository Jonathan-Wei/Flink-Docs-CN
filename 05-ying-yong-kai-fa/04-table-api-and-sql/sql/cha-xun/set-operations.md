---
description: 支持批、流
---

# Set Operations

## UNION 

`UNION`和`UNION ALL`返回在任意一个表中找到的行。`UNION`只取不同的行，而`UNION ALL`不从结果行中删除重复的行。

```sql
Flink SQL> create view t1(s) as values ('c'), ('a'), ('b'), ('b'), ('c');
Flink SQL> create view t2(s) as values ('d'), ('e'), ('a'), ('b'), ('b');

Flink SQL> (SELECT s FROM t1) UNION (SELECT s FROM t2);
+---+
|  s|
+---+
|  c|
|  a|
|  b|
|  d|
|  e|
+---+

Flink SQL> (SELECT s FROM t1) UNION ALL (SELECT s FROM t2);
+---+
|  c|
+---+
|  c|
|  a|
|  b|
|  b|
|  c|
|  d|
|  e|
|  a|
|  b|
|  b|
+---+
```

## INTERSECT 

`INTERSECT`和`INTERSECT ALL`返回在两个表中找到的行。`INTERSECT`只取不同的行，而`INTERSECT ALL`不从结果行中删除重复的行。

```sql
Flink SQL> (SELECT s FROM t1) INTERSECT (SELECT s FROM t2);
+---+
|  s|
+---+
|  a|
|  b|
+---+

Flink SQL> (SELECT s FROM t1) INTERSECT ALL (SELECT s FROM t2);
+---+
|  s|
+---+
|  a|
|  b|
|  b|
+---+
```

## EXCEPT 

`EXCEPT`和`EXCEPT ALL`返回在一个表中找到的行，但在另一个表中没有。`EXCEPT`只取不同的行，而`EXCEPT ALL`不从结果行中删除重复的行。

```sql
Flink SQL> (SELECT s FROM t1) EXCEPT (SELECT s FROM t2);
+---+
| s |
+---+
| c |
+---+

Flink SQL> (SELECT s FROM t1) EXCEPT ALL (SELECT s FROM t2);
+---+
| s |
+---+
| c |
| c |
+---+
```

## IN 

如果表达式存在于给定的表子查询中，则返回`true`。子查询表必须由一列组成。此列必须与表达式具有相同的数据类型。

```sql
SELECT user, amount
FROM Orders
WHERE product IN (
    SELECT product FROM NewProducts
)
```

 优化器将IN条件重写为联接和分组操作。对于流查询，根据不同输入行的数量，计算查询结果所需的状态可能会无限增长。您可以为查询配置提供适当的状态生存时间（TTL），以防止状态大小过大。请注意，这可能会影响查询结果的正确性。有关详细信息，请参见[查询配置](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/config/#table-exec-state-ttl)。

## EXISTS 

```sql
SELECT user, amount
FROM Orders
WHERE product EXISTS (
    SELECT product FROM NewProducts
)
```

如果子查询返回至少一行，则返回true。仅当该操作可以在联接和组操作中重写时才受支持。

优化器将`EXISTS`操作重写为联接和组操作。对于流查询，根据不同输入行的数量，计算查询结果所需的状态可能会无限增长。您可以为查询配置提供适当的状态生存时间（TTL），以防止状态大小过大。请注意，这可能会影响查询结果的正确性。有关详细信息，请参见[查询配置](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/config/#table-exec-state-ttl)。

