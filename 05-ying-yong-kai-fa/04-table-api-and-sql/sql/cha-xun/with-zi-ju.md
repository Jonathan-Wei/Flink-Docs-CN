---
description: 支持批、流
---

# WITH子句

`WITH`提供了一种编写用于较大查询的辅助语句的方法。这些语句（通常称为通用表表达式（CTE））可以认为是定义仅针对一个查询存在的临时视图。

该`WITH`语句的语法为：

```sql
WITH <with_item_definition> [ , ... ]
SELECT ... FROM ...;

<with_item_defintion>:
    with_item_name (column_name[, ...n]) AS ( <select_query> )
```

下面的示例定义一个公用表表达式`orders_with_total`，并在`GROUP BY`查询中使用它。

```sql
WITH orders_with_total AS (
    SELECT order_id, price + tax AS total
    FROM Orders
)
SELECT order_id, SUM(total)
FROM orders_with_total
GROUP BY order_id;
```

