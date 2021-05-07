---
description: 支持批
---

# LIMIT子句

`LIMIT`子句限制`SELECT`语句返回的行数。通常，此子句与ORDER BY结合使用以确保结果是确定的。

下面的示例选择`Orders`表中的前3行。

```text
SELECT *
FROM Orders
ORDER BY orderTime
LIMIT 3
```

