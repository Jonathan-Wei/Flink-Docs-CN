# 持续查询中的关联

## 定期关联



```sql
SELECT * FROM Orders
INNER JOIN Product
ON Orders.productId = Product.id
```

## 时间窗口关联



```sql
SELECT *
FROM
  Orders o,
  Shipments s
WHERE o.id = s.orderId AND
      o.ordertime BETWEEN s.shiptime - INTERVAL '4' HOUR AND s.shiptime
```

## 时态表关联



```sql
SELECT * FROM Orders;

rowtime amount currency
======= ====== =========
10:15        2 Euro
10:30        1 US Dollar
10:32       50 Yen
10:52        3 Euro
11:04        5 US Dollar
```

```sql
SELECT * FROM RatesHistory;

rowtime currency   rate
======= ======== ======
09:00   US Dollar   102
09:00   Euro        114
09:00   Yen           1
10:45   Euro        116
11:15   Euro        119
11:49   Pounds      108
```

```sql
rowtime amount currency
======= ====== =========
10:15        2 Euro
```

```sql
SELECT
  SUM(o.amount * r.rate) AS amount
FROM Orders AS o,
  RatesHistory AS r
WHERE r.currency = o.currency
AND r.rowtime = (
  SELECT MAX(rowtime)
  FROM RatesHistory AS r2
  WHERE r2.currency = o.currency
  AND r2.rowtime <= o.rowtime);
```

```sql
SELECT
  o.amount * r.rate AS amount
FROM
  Orders AS o,
  LATERAL TABLE (Rates(o.rowtime)) AS r
WHERE r.currency = o.currency
```

### 用法

{% tabs %}
{% tab title="SQL" %}
```sql
SELECT
  SUM(o_amount * r_rate) AS amount
FROM
  Orders,
  LATERAL TABLE (Rates(o_proctime))
WHERE
  r_currency = o_currency
```
{% endtab %}

{% tab title="Java" %}
```java
Table result = orders
    .join(new Table(tEnv, "rates(o_proctime)"), "o_currency = r_currency")
    .select("(o_amount * r_rate).sum as amount");
```
{% endtab %}

{% tab title="Scala" %}
```scala
Table result = orders
    .join(new Table(tEnv, "rates(o_proctime)"), "o_currency = r_currency")
    .select("(o_amount * r_rate).sum as amount");
```
{% endtab %}
{% endtabs %}

### 处理时间\(Processing-time\)时态关联

### 事件时间\(Event-time\)时态关联

