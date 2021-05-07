---
description: 支持批、流
---

# ORDER BY子句

`ORDER BY`子句使结果行根据指定的表达式进行排序。如果根据最左边的表达式两行相等，则根据下一个表达式对它们进行比较，依此类推。如果根据所有指定的表达式，它们是相等的，则以依赖于实现的顺序返回它们。

在流模式下运行时，表的主要排序顺序必须在[时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/concepts/time_attributes/)上升序。后续所有的排序可以自由选择。但是在批处理模式下没有此限制。

```sql
SELECT *
FROM Orders
ORDER BY order_time, order_id
```

