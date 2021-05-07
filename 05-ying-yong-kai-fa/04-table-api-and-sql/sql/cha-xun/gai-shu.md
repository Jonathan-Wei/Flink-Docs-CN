# 概述

使用`TableEnvironment`的`sqlQuery()`方法指定SQL查询。该方法以表的形式返回SQL查询的结果。表可以用于后续的[SQL和表API查询](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#mixing-table-api-and-sql)，可以[转换为`DataSet`或`DataStream`](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#integration-with-datastream-and-dataset-api)，也可以写入[TableSink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#emit-a-table)\)。可以无缝地混合SQL和Table API查询，并对其进行整体优化，并将其转换为单个程序。

为了访问SQL查询中的表，必须在`TableEnvironment`中注册表。可以从[TableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource), [Table](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-table), [DataStream, 或 DataSet](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-datastream-or-dataset-as-table).注册表。另外，用户还可以在`TableEnvironment`中注册外部Catalog，以指定数据源的位置。

为了方便起见，`Table.toString()`在`TableEnvironment`中以惟一的名称自动注册表并返回该名称。因此，表对象可以直接内联到SQL查询中\(通过字符串连接\)，如下面的示例所示。

{% hint style="info" %}
注意：包含不受支持的SQL功能的查询会导致`TableException`。以下各节列出了批处理表和流表上SQL的受支持功能。
{% endhint %}

## 指定查询

下面的示例展示如何在注册表和内联表上指定SQL查询。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// ingest a DataStream from an external source
DataStream<Tuple3<Long, String, Integer>> ds = env.addSource(...);

// SQL query with an inlined (unregistered) table
Table table = tableEnv.fromDataStream(ds, "user, product, amount");
Table result = tableEnv.sqlQuery(
  "SELECT SUM(amount) FROM " + table + " WHERE product LIKE '%Rubber%'");

// SQL query with a registered table
// register the DataStream as view "Orders"
tableEnv.createTemporaryView("Orders", ds, "user, product, amount");
// run a SQL query on the Table and retrieve the result as a new Table
Table result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// create and register a TableSink
final Schema schema = new Schema()
    .field("product", DataTypes.STRING())
    .field("amount", DataTypes.INT());

tableEnv.connect(new FileSystem("/path/to/file"))
    .withFormat(...)
    .withSchema(schema)
    .createTemporaryTable("RubberOrders");

// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = StreamTableEnvironment.create(env)

// read a DataStream from an external source
val ds: DataStream[(Long, String, Integer)] = env.addSource(...)

// SQL query with an inlined (unregistered) table
val table = ds.toTable(tableEnv, 'user, 'product, 'amount)
val result = tableEnv.sqlQuery(
  s"SELECT SUM(amount) FROM $table WHERE product LIKE '%Rubber%'")

// SQL query with a registered table
// register the DataStream under the name "Orders"
tableEnv.createTemporaryView("Orders", ds, 'user, 'product, 'amount)
// run a SQL query on the Table and retrieve the result as a new Table
val result2 = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")

// SQL update with a registered table
// create and register a TableSink
val schema = new Schema()
    .field("product", DataTypes.STRING())
    .field("amount", DataTypes.INT())

tableEnv.connect(new FileSystem("/path/to/file"))
    .withFormat(...)
    .withSchema(schema)
    .createTemporaryTable("RubberOrders")

// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)

# SQL query with an inlined (unregistered) table
# elements data type: BIGINT, STRING, BIGINT
table = table_env.from_elements(..., ['user', 'product', 'amount'])
result = table_env \
    .sql_query("SELECT SUM(amount) FROM %s WHERE product LIKE '%%Rubber%%'" % table)

# SQL update with a registered table
# create and register a TableSink
t_env.connect(FileSystem().path("/path/to/file")))
    .with_format(Csv()
                 .field_delimiter(',')
                 .deriveSchema())
    .with_schema(Schema()
                 .field("product", DataTypes.STRING())
                 .field("amount", DataTypes.BIGINT()))
    .create_temporary_table("RubberOrders")

# run a SQL update query on the Table and emit the result to the TableSink
table_env \
    .sql_update("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}
{% endtabs %}

## 执行查询

可以执行SELECT语句或VALUES语句，通过TableEnvironment.executeSql\(\)方法将内容收集到本地。该方法将SELECT语句\(或VALUES语句\)的结果作为TableResult返回。与SELECT语句类似，可以使用Table.execute\(\)方法执行Table对象，将查询的内容收集到本地客户端。collect\(\)方法返回一个可关闭的行迭代器。除非已收集所有结果数据，否则选择作业将不会完成。我们应该主动关闭作业，以避免通过CloseableIterator\#close\(\)方法泄漏资源。我们还可以通过`TableResult.print()`方法将选择结果打印到客户端控制台。TableResult中的结果数据只能访问一次。因此， `collect()` 和`print()` 不能相互调用。

在不同的检查点设置下， `TableResult.collect()` 和  `TableResult.print()`的行为略有不同\(要为流作业启用检查点，请参阅 [checkpointing config](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/deployment/config/#checkpointing)\)。

* 对于没有检查点的批处理作业或流作业，`TableResult.collect()` 和`TableResult.print()`

  既不能精确地保证一次，也不能至少保证一次。查询结果生成后，客户端可以立即访问它们，但当作业失败并重新启动时将引发异常。

* 对于具有精确一次检查点的流作业，`TableResult.collect()` 和`TableResult.print()`

  保证端到端精确一次的记录传递。只有在相应的检查点完成后，客户端才能访问结果。

* 对于具有至少一次检查点的流作业，  `TableResult.collect()` 和`TableResult.print()`保证端到端至少一次的记录传递。查询结果一旦产生，客户端就可以立即访问它们，但同样的结果也可能被多次交付。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env, settings);

tableEnv.executeSql("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");

// execute SELECT statement
TableResult tableResult1 = tableEnv.executeSql("SELECT * FROM Orders");
// use try-with-resources statement to make sure the iterator will be closed automatically
try (CloseableIterator<Row> it = tableResult1.collect()) {
    while(it.hasNext()) {
        Row row = it.next();
        // handle row
    }
}

// execute Table
TableResult tableResult2 = tableEnv.sqlQuery("SELECT * FROM Orders").execute();
tableResult2.print();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
val tableEnv = StreamTableEnvironment.create(env, settings)
// enable checkpointing
tableEnv.getConfig.getConfiguration.set(
  ExecutionCheckpointingOptions.CHECKPOINTING_MODE, CheckpointingMode.EXACTLY_ONCE)
tableEnv.getConfig.getConfiguration.set(
  ExecutionCheckpointingOptions.CHECKPOINTING_INTERVAL, Duration.ofSeconds(10))

tableEnv.executeSql("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")

// execute SELECT statement
val tableResult1 = tableEnv.executeSql("SELECT * FROM Orders")
val it = tableResult1.collect()
try while (it.hasNext) {
  val row = it.next
  // handle row
}
finally it.close() // close the iterator to avoid resource leak

// execute Table
val tableResult2 = tableEnv.sqlQuery("SELECT * FROM Orders").execute()
tableResult2.print()
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env, settings)
# enable checkpointing
table_env.get_config().get_configuration().set_string("execution.checkpointing.mode", "EXACTLY_ONCE")
table_env.get_config().get_configuration().set_string("execution.checkpointing.interval", "10s")

table_env.execute_sql("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")

# execute SELECT statement
table_result1 = table_env.execute_sql("SELECT * FROM Orders")
table_result1.print()

# execute Table
table_result2 = table_env.sql_query("SELECT * FROM Orders").execute()
table_result2.print()

```
{% endtab %}
{% endtabs %}

## 语法

Flink使用[Apache Calcite](https://calcite.apache.org/docs/reference.html)解析SQL，支持标准ANSI SQL。

以下BNF语法描述了批处理和流查询中支持的SQL功能的超集。“[操作”](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/overview/#operations)部分显示了受支持功能的示例，并指示仅批处理或流查询支持哪些功能。

```sql
query:
  values
  | {
      select
      | selectWithoutFrom
      | query UNION [ ALL ] query
      | query EXCEPT query
      | query INTERSECT query
    }
    [ ORDER BY orderItem [, orderItem ]* ]
    [ LIMIT { count | ALL } ]
    [ OFFSET start { ROW | ROWS } ]
    [ FETCH { FIRST | NEXT } [ count ] { ROW | ROWS } ONLY]

orderItem:
  expression [ ASC | DESC ]

select:
  SELECT [ ALL | DISTINCT ]
  { * | projectItem [, projectItem ]* }
  FROM tableExpression
  [ WHERE booleanExpression ]
  [ GROUP BY { groupItem [, groupItem ]* } ]
  [ HAVING booleanExpression ]
  [ WINDOW windowName AS windowSpec [, windowName AS windowSpec ]* ]

selectWithoutFrom:
  SELECT [ ALL | DISTINCT ]
  { * | projectItem [, projectItem ]* }

projectItem:
  expression [ [ AS ] columnAlias ]
  | tableAlias . *

tableExpression:
  tableReference [, tableReference ]*
  | tableExpression [ NATURAL ] [ LEFT | RIGHT | FULL ] JOIN tableExpression [ joinCondition ]

joinCondition:
  ON booleanExpression
  | USING '(' column [, column ]* ')'

tableReference:
  tablePrimary
  [ matchRecognize ]
  [ [ AS ] alias [ '(' columnAlias [, columnAlias ]* ')' ] ]

tablePrimary:
  [ TABLE ] tablePath [ dynamicTableOptions ] [systemTimePeriod] [[AS] correlationName]
  | LATERAL TABLE '(' functionName '(' expression [, expression ]* ')' ')'
  | UNNEST '(' expression ')'

tablePath:
  [ [ catalogName . ] schemaName . ] tableName

systemTimePeriod:
  FOR SYSTEM_TIME AS OF dateTimeExpression

dynamicTableOptions:
  /*+ OPTIONS(key=val [, key=val]*) */

key:
  stringLiteral

val:
  stringLiteral

values:
  VALUES expression [, expression ]*

groupItem:
  expression
  | '(' ')'
  | '(' expression [, expression ]* ')'
  | CUBE '(' expression [, expression ]* ')'
  | ROLLUP '(' expression [, expression ]* ')'
  | GROUPING SETS '(' groupItem [, groupItem ]* ')'

windowRef:
    windowName
  | windowSpec

windowSpec:
    [ windowName ]
    '('
    [ ORDER BY orderItem [, orderItem ]* ]
    [ PARTITION BY expression [, expression ]* ]
    [
        RANGE numericOrIntervalExpression {PRECEDING}
      | ROWS numericExpression {PRECEDING}
    ]
    ')'

matchRecognize:
      MATCH_RECOGNIZE '('
      [ PARTITION BY expression [, expression ]* ]
      [ ORDER BY orderItem [, orderItem ]* ]
      [ MEASURES measureColumn [, measureColumn ]* ]
      [ ONE ROW PER MATCH ]
      [ AFTER MATCH
            ( SKIP TO NEXT ROW
            | SKIP PAST LAST ROW
            | SKIP TO FIRST variable
            | SKIP TO LAST variable
            | SKIP TO variable )
      ]
      PATTERN '(' pattern ')'
      [ WITHIN intervalLiteral ]
      DEFINE variable AS condition [, variable AS condition ]*
      ')'

measureColumn:
      expression AS alias

pattern:
      patternTerm [ '|' patternTerm ]*

patternTerm:
      patternFactor [ patternFactor ]*

patternFactor:
      variable [ patternQuantifier ]

patternQuantifier:
      '*'
  |   '*?'
  |   '+'
  |   '+?'
  |   '?'
  |   '??'
  |   '{' { [ minRepeat ], [ maxRepeat ] } '}' ['?']
  |   '{' repeat '}'
```

## Operations

* [WITH clause](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/with/)
* [SELECT & WHERE](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/select/)
* [SELECT DISTINCT](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/select-distinct/)
* [Windowing TVF](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-tvf/)
* [Window Aggregation](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-agg/)
* [Group Aggregation](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/group-agg/)
* [Over Aggregation](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/over-agg/)
* [Joins](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/joins/)
* [Set Operations](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/set-ops/)
* [ORDER BY clause](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/orderby/)
* [LIMIT clause](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/limit/)
* [Top-N](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/topn/)
* [Window Top-N](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/window-topn/)
* [Deduplication](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/deduplication/)
* [Pattern Recognition](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/table/sql/queries/match_recognize/)

