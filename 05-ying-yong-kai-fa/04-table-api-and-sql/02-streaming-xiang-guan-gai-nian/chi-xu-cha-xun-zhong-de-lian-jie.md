# 持续查询中的关联

关联是批处理数据处理中常见且易于理解的操作，用于关联两个关系行。但是，[动态表](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html)上的关联语义不太明显甚至令人困惑。

因此，有几种方法可以使用Table API或SQL实际执行关联。

有关语法的更多信息，请查看[表API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#joins)和[SQL中](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#joins)的连接部分。

## 定期关联

常规关联是最通用的关联类型，在这种关联类型中，关联输入的任何一边的任何新记录或更改都是可见的，并且会影响整个连接结果。例如，如果左侧有一条新记录，它将与右侧的所有以前和将来的记录关联。

```sql
SELECT * FROM Orders
INNER JOIN Product
ON Orders.productId = Product.id
```

这些语义允许任何类型的更新\(插入、更新、删除\)输入表。

但是，这个操作有一个重要的含义:它要求关联输入的两侧永久保持在Flink的状态中。因此，如果一个或两个输入表都在持续增长，那么资源使用也会无限增长。

## 时间窗口关联

时间窗口连接由关联谓词定义，关联谓词检查输入记录的[时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)是否在某些时间限制内，即时间窗口。

```sql
SELECT *
FROM
  Orders o,
  Shipments s
WHERE o.id = s.orderId AND
      o.ordertime BETWEEN s.shiptime - INTERVAL '4' HOUR AND s.shiptime
```

与常规关联操作相比，这种关联只支持具有时间属性的仅追加表。由于时间属性是准一元递增的，Flink可以在不影响结果正确性的情况下从其状态中删除旧值。

## 时态表函数关联

与时态表的关联将仅追加表\(左输入/探测端\)与时态表\(右输入/构建端\)关联起来，即，一个随时间变化并跟踪其变化的表。有关时间表的更多信息，请查看相应的页面。

下面的示例显示了一个仅追加的表订单，该订单应该与不断变化的货币汇率表RatesHistory关联在一起。

Orders是一个仅用于追加的表，表示对给定金额和给定货币的付款。例如，在10:15有一个2欧元的订单。

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

RatesHistory代表了一个不断变化的货币对日元汇率的附表\(其汇率为1\)。从10:45到11:15是116。

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

假设我们要计算所有订单转换成共同货币\(日元\)的金额。

例如，我们希望使用给定行时间\(114\)的适当转换速率来转换以下顺序。

```sql
rowtime amount currency
======= ====== =========
10:15        2 Euro
```

如果不使用[时态表](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html)的概念，就需要编写如下查询：

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

借助RatesHistory上的时态表函数Rates，我们可以用SQL表示如下查询:

```sql
SELECT
  o.amount * r.rate AS amount
FROM
  Orders AS o,
  LATERAL TABLE (Rates(o.rowtime)) AS r
WHERE r.currency = o.currency
```

探测侧的每个记录将在探测侧记录的相关时间属性时与构建侧表的版本关联。为了支持在构建侧表上更新\(覆盖\)以前的值，表必须定义一个主键。

在我们的示例中，订单中的每个记录都将与时间o.rowtime中的速率版本相连接。前面已经将currency字段定义为rate的主键，并在我们的示例中用于连接两个表。如果查询使用的是处理时概念，那么在执行操作时，新添加的订单将始终与最新版本的速率相连接。

与[常规关联](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/joins.html#regular-joins)相比，这意味着如果在构建端有一条新记录，它不会影响连接的先前结果。这再次允许Flink限制必须保持在状态的元素的数量。

与[时间窗口关联](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/joins.html#time-windowed-joins)相比，时态表连接没有定义一个时间窗口，在这个时间窗口内记录将被连接。探测端记录总是在time属性指定的时间与构建端版本连接。因此，构建端的记录可能是任意旧的。随着时间的推移，先前的和不再需要的记录版本\(对于给定的主键\)将从状态中删除。

这种行为使得时态表连接成为用关系术语表示流充实的一个很好的候选。

### 用法

在[定义了时态表函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html#defining-temporal-table-function)之后，我们就可以开始使用它了。时态表函数的使用方法与普通表函数的使用方法相同。

下面的代码片段解决了从Orders表转换货币的激励问题:

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

注意:在[查询配置中](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html)定义的状态保留尚未为临时连接实现。这意味着计算查询结果所需的状态可能会无限增长，这取决于历史表中不同的主键的数量。

### 处理时间\(Processing-time\)时态关联

使用处理时间属性，不可能将_过去的_时间属性作为参数传递给时态表函数。根据定义，它始终是当前时间戳。因此，处理时间时态表函数的调用将始终返回基础表的最新已知版本，并且基础历史表中的任何更新也将立即覆盖当前值。

只有构建端记录的最新版本（相对于已定义的主键）保留在该状态中。构建端的更新将不会影响先前发出的关联结果。

可以将处理时间时间连接视为`HashMap<K, V>`存储构建端的所有记录的简单连接。当构建端的新记录具有与先前记录相同的密钥时，旧值只是被覆盖。来自探测器侧的每条记录始终根据最新/当前状态进行评估`HashMap`。

### 事件时间\(Event-time\)时态关联

利用事件时间属性（即行时属性），可以将_过去的_时间属性传递给时态表函数。这允许在共同的时间点关联两个表。

与处理时间时间连接相比，时态表不仅保持状态中的构建侧记录的最新版本（相对于定义的主键），而且存储自上一个水印以来的所有版本（由时间标识）。

例如，根据时态表的概念，将事件时间戳为12:30:00的传入行附加到探测侧表并在12:30:00与构建侧表的版本关联。因此，传入行只与时间戳小于或等于12:30:00的行关联，并根据主键应用更新，直到此时为止。

根据事件时间的定义，水印允许连接操作及时向前移动，并丢弃构建表的版本，这些版本不再是必需的，因为不期望有时间戳较低或相等的输入行。

## 与临时表关联

 与临时表的联接将任意表（左侧输入/探针侧）与临时表（右侧输入/构建侧）联接，即随时间变化的外部尺寸表。请检查相应的页面以获取有关时[态表的](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#temporal-table)更多信息。

{% hint style="danger" %}
**注意：**用户不能使用任意表作为时态表，但需要使用由LookupableTableSource支持的表。LookupableTableSource只能作为时态表用于时态连接。有关[如何定义LookupableTableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sourceSinks.html#defining-a-tablesource-with-lookupable)的更多细节，请参见该页。
{% endhint %}

下面的示例显示了一个订单流，该订单流应该与不断变化的`LatestRates`汇率表\(最近的基准\)相连接。

 `LatestRates`是以最新汇率实现的维度表。在时间`10:15`，`10:30`，`10:52`，`LatestRates`内容如下所示：

```text
10:15> SELECT * FROM LatestRates;

currency   rate
======== ======
US Dollar   102
Euro        114
Yen           1

10:30> SELECT * FROM LatestRates;

currency   rate
======== ======
US Dollar   102
Euro        114
Yen           1


10:52> SELECT * FROM LatestRates;

currency   rate
======== ======
US Dollar   102
Euro        116     <==== changed from 114 to 116
Yen           1
```

LastestRates在10:15和10:30时的内容相同。欧元汇率在10:52从114变为116。

Orders是一个只追加的表，它表示给定金额和给定货币的支付。例如，在10:15有一个2欧元的订单。

```text
SELECT * FROM Orders;

amount currency
====== =========
     2 Euro             <== arrived at time 10:15
     1 US Dollar        <== arrived at time 10:30
     2 Euro             <== arrived at time 10:52
```

鉴于我们想要计算所有订单转换成通用货币\(日元\)的金额。

例如，我们希望使用最新的汇率转换以下订单。结果将是:

```text
amount currency     rate   amout*rate
====== ========= ======= ============
     2 Euro          114          228    <== arrived at time 10:15
     1 US Dollar     102          102    <== arrived at time 10:30
     2 Euro          116          232    <== arrived at time 10:52
```

借助时态表关联，我们可以将这样的查询用SQL表示为:

```text
SELECT
  o.amout, o.currency, r.rate, o.amount * r.rate
FROM
  Orders AS o
  JOIN LatestRates FOR SYSTEM_TIME AS OF o.proctime AS r
  ON r.currency = o.currency
```

### 

### 用法

临时表联接的语法如下：

```sql
SELECT [column_list]
FROM table1 [AS <alias1>]
[LEFT] JOIN table2 FOR SYSTEM_TIME AS OF table1.proctime [AS <alias2>]
ON table1.column-name1 = table2.column-name1
```

目前，只支持内连接和左连接。表1中的FOR SYSTEM\_TIME。在时态表之后应遵循proctime。proctime是表1的[处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html#processing-time)。这意味着它在处理时从左表连接每个记录时获取时态表的快照。 

例如，在[定义时态表之后](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/temporal_tables.html#defining-temporal-table)，我们可以如下使用它。

```text
SELECT
  SUM(o_amount * r_rate) AS amount
FROM
  Orders
  JOIN LatestRates FOR SYSTEM_TIME AS OF o_proctime
  ON r_currency = o_currency
```

{% hint style="danger" %}
仅在Blink Planner中受支持。
{% endhint %}

{% hint style="danger" %}
仅在SQL中支持，而在Table API中尚不支持。
{% endhint %}

{% hint style="danger" %}
Flink当前不支持事件时间时态表联接。
{% endhint %}

