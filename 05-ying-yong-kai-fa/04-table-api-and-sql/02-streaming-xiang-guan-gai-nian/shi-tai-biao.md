# 时态表

时态表表示更改历史表上的\(参数化\)视图的概念，该视图在特定时间点返回表的内容。

Flink可以跟踪应用于底层仅追加表的更改，并允许在查询中的某个时间点访问表的内容。

## 动机

假设我们有表RatesHistory

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

“RatesHistory”代表了一个不断增长的货币对日元汇率的附表\(其汇率为1\)，例如，从9点到10点45分，欧元对日元的汇率为114。从10:45到11:15是116。

假设我们想要输出10:58时的所有当前汇率，我们需要以下SQL查询来计算结果表:

```sql
SELECT *
FROM RatesHistory AS r
WHERE r.rowtime = (
  SELECT MAX(rowtime)
  FROM RatesHistory AS r2
  WHERE r2.currency = r.currency
  AND r2.rowtime <= TIME '10:58');
```

相关子查询确定相应货币的最大时间小于或等于所需时间。外部查询列出具有最大时间戳的比率。

下表显示了这种计算的结果。在我们的例子中，考虑了10:45更新到欧元，但是在10:58的表格版本中没有考虑到11:15更新到欧元和新的英镑条目。

```sql
rowtime currency   rate
======= ======== ======
09:00   US Dollar   102
09:00   Yen           1
10:45   Euro        116
```

时态表的概念旨在简化此类查询，加速它们的执行，并减少Flink的状态使用。时态表是仅追加表上的参数化视图，它将仅追加表的行解释为表的更改日志，并在特定时间点提供该表的版本。将仅追加的表解释为更改日志需要指定主键属性和时间戳属性。主键决定哪些行被覆盖，时间戳决定行有效的时间。

在上面的例子中，currency是RatesHistory表的主键，rowtime是timestamp属性。

在Flink中，时态表由时态表函数表示。

## 时态表函数

为了访问时态表中的数据，必须传递一个[时间属性](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/time_attributes.html)，该属性确定将返回的表的版本。Flink使用[表函数](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/udfs.html#table-functions)的SQL语法提供了一种表示它的方法。

一旦定义，时态表函数接受单个时间参数timeAttribute并返回一组行。该集合包含与给定时间属性相关的所有现有主键的最新行版本。

假设我们定义了一个基于RatesHistory表的时态表函数Rates\(timeAttribute\)，我们可以通过以下方式查询这个函数:

```sql
SELECT * FROM Rates('10:15');

rowtime currency   rate
======= ======== ======
09:00   US Dollar   102
09:00   Euro        114
09:00   Yen           1

SELECT * FROM Rates('11:00');

rowtime currency   rate
======= ======== ======
09:00   US Dollar   102
10:45   Euro        116
09:00   Yen           1
```

对Rates\(timeAttribute\)的每个查询都将返回给定timeAttribute的Rates状态。

{% hint style="info" %}
目前，Flink不支持直接查询具有固定时间属性参数的时态表函数。目前，时态表函数只能用于连接。上面的示例用于直观地了解函数Rates\(timeAttribute\)返回什么。
{% endhint %}

有关如何连接时态表的更多信息，另请参阅有关[连续查询的连接](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/joins.html)的页面。  


### 定义时态表函数

下面的代码片段说明了如何从仅追加的表创建时态表函数。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.table.functions.TemporalTableFunction;
(...)

// Get the stream and table environments.
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = TableEnvironment.getTableEnvironment(env);

// Provide a static data set of the rates history table.
List<Tuple2<String, Long>> ratesHistoryData = new ArrayList<>();
ratesHistoryData.add(Tuple2.of("US Dollar", 102L));
ratesHistoryData.add(Tuple2.of("Euro", 114L));
ratesHistoryData.add(Tuple2.of("Yen", 1L));
ratesHistoryData.add(Tuple2.of("Euro", 116L));
ratesHistoryData.add(Tuple2.of("Euro", 119L));

// Create and register an example table using above data set.
// In the real setup, you should replace this with your own table.
DataStream<Tuple2<String, Long>> ratesHistoryStream = env.fromCollection(ratesHistoryData);
Table ratesHistory = tEnv.fromDataStream(ratesHistoryStream, "r_currency, r_rate, r_proctime.proctime");

tEnv.registerTable("RatesHistory", ratesHistory);

// Create and register a temporal table function.
// Define "r_proctime" as the time attribute and "r_currency" as the primary key.
TemporalTableFunction rates = ratesHistory.createTemporalTableFunction("r_proctime", "r_currency"); // <==== (1)
tEnv.registerFunction("Rates", rates);                                                              // <==== (2)
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Get the stream and table environments.
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tEnv = TableEnvironment.getTableEnvironment(env)

// Provide a static data set of the rates history table.
val ratesHistoryData = new mutable.MutableList[(String, Long)]
ratesHistoryData.+=(("US Dollar", 102L))
ratesHistoryData.+=(("Euro", 114L))
ratesHistoryData.+=(("Yen", 1L))
ratesHistoryData.+=(("Euro", 116L))
ratesHistoryData.+=(("Euro", 119L))

// Create and register an example table using above data set.
// In the real setup, you should replace this with your own table.
val ratesHistory = env
  .fromCollection(ratesHistoryData)
  .toTable(tEnv, 'r_currency, 'r_rate, 'r_proctime.proctime)

tEnv.registerTable("RatesHistory", ratesHistory)

// Create and register TemporalTableFunction.
// Define "r_proctime" as the time attribute and "r_currency" as the primary key.
val rates = ratesHistory.createTemporalTableFunction('r_proctime, 'r_currency) // <==== (1)
tEnv.registerFunction("Rates", rates)                                          // <==== (2)

```
{% endtab %}
{% endtabs %}

第\(1\)行创建一个rate时态表函数，它允许我们使用表API中的函数rate。

第\(2\)行在表环境中的名称Rates下注册这个函数，这允许我们在SQL中使用Rates函数

