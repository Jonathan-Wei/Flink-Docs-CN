---
description: 支持批、流
---

# SELECT DISTINCT

如果指定了SELECT DISTINCT，则从结果集中删除所有重复的行\(每组重复的行保留一行\)。

```text
SELECT DISTINCT id FROM Orders
```

 对于流查询，计算查询结果所需的状态可能会无限增长。状态大小取决于不同行的数量。您可以为查询配置提供适当的状态生存时间（TTL），以防止状态大小过大。请注意，这可能会影响查询结果的正确性。有关详细信息，请参见[查询配置](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/config/#table-exec-state-ttl)。

