# 概念&通用API

Table API和SQL集成在一个联合API中。此API的核心概念是`Table`用作查询的输入和输出。本文档显示了具有Table API和SQL查询的程序的常见结构，如何注册`Table`，如何查询`Table`以及如何发出`Table`。

## Table API和SQL程序结构

批处理和流式传输的所有Table API和SQL程序都遵循相同的模式。以下代码示例显示了Table API和SQL程序的常见结构。

{% tabs %}
{% tab title="Java" %}
```java
// for batch programs use ExecutionEnvironment instead of StreamExecutionEnvironment
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// create a TableEnvironment
// for batch programs use BatchTableEnvironment instead of StreamTableEnvironment
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// register a Table
tableEnv.registerTable("table1", ...)            // or
tableEnv.registerTableSource("table2", ...);     // or
tableEnv.registerExternalCatalog("extCat", ...);
// register an output Table
tableEnv.registerTableSink("outputTable", ...);

// create a Table from a Table API query
Table tapiResult = tableEnv.scan("table1").select(...);
// create a Table from a SQL query
Table sqlResult  = tableEnv.sqlQuery("SELECT ... FROM table2 ... ");

// emit a Table API result Table to a TableSink, same for SQL result
tapiResult.insertInto("outputTable");

// execute
env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
// for batch programs use ExecutionEnvironment instead of StreamExecutionEnvironment
val env = StreamExecutionEnvironment.getExecutionEnvironment

// create a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// register a Table
tableEnv.registerTable("table1", ...)           // or
tableEnv.registerTableSource("table2", ...)     // or
tableEnv.registerExternalCatalog("extCat", ...)
// register an output Table
tableEnv.registerTableSink("outputTable", ...);

// create a Table from a Table API query
val tapiResult = tableEnv.scan("table1").select(...)
// Create a Table from a SQL query
val sqlResult  = tableEnv.sqlQuery("SELECT ... FROM table2 ...")

// emit a Table API result Table to a TableSink, same for SQL result
tapiResult.insertInto("outputTable")

// execute
env.execute()
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**注意：**Table API和SQL查询可以轻松集成并嵌入到DataStream或DataSet程序中。查看[与DataStream和DataSet API](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#integration-with-datastream-and-dataset-api)的[集成](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#integration-with-datastream-and-dataset-api)部分，了解如何将DataStream和DataSet转换为Tables，反之亦然。
{% endhint %}

## 创建TableEnvironment

`TableEnvironment`是Table API和SQL集成的核心概念。它负责：

* 在内部目录中注册表 
* 注册外部目录 
* 执行SQL查询 
* 注册用户定义的\(标量、表或聚合\)函数 
* 将数据流或数据集转换为表 保存对ExecutionEnvironment或StreamExecutionEnvironment的引用

表总是绑定到特定的表环境。在同一个查询中组合不同表环境的表是不可能的，例如，联接或联合它们。

TableEnvironment是通过调用静态TableEnvironment. getTableEnvironment\(\)方法创建的，该方法带有一个StreamExecutionEnvironment或一个ExecutionEnvironment以及一个可选的TableConfig。TableConfig可用于配置TableEnvironment或自定义查询优化和转换过程\(参见[查询优化](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#query-optimization)\)。

{% tabs %}
{% tab title="Java" %}
```java
// ***************
// STREAMING QUERY
// ***************
StreamExecutionEnvironment sEnv = StreamExecutionEnvironment.getExecutionEnvironment();
// create a TableEnvironment for streaming queries
StreamTableEnvironment sTableEnv = TableEnvironment.getTableEnvironment(sEnv);

// ***********
// BATCH QUERY
// ***********
ExecutionEnvironment bEnv = ExecutionEnvironment.getExecutionEnvironment();
// create a TableEnvironment for batch queries
BatchTableEnvironment bTableEnv = TableEnvironment.getTableEnvironment(bEnv);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// ***************
// STREAMING QUERY
// ***************
val sEnv = StreamExecutionEnvironment.getExecutionEnvironment
// create a TableEnvironment for streaming queries
val sTableEnv = TableEnvironment.getTableEnvironment(sEnv)

// ***********
// BATCH QUERY
// ***********
val bEnv = ExecutionEnvironment.getExecutionEnvironment
// create a TableEnvironment for batch queries
val bTableEnv = TableEnvironment.getTableEnvironment(bEnv)
```
{% endtab %}
{% endtabs %}

## 在目录中注册表

`TableEnvironment`维护按名称注册的表的目录。有两种类型的表，_输入表_和_输出表_。输入表可以在表API和SQL查询中引用，并提供输入数据。输出表可用于将Table API或SQL查询的结果发送到外部系统。

可以从各种来源注册输入表：

* 现有的Table对象，通常是Table API或SQL查询的结果。 
* Table Souce，用于访问外部数据，如文件、数据库或消息传递系统。 
* 来自DataStream或DataSet程序的DataStream或DataSet。注册一个`DataStream`或`DataSet`在[与DataStream和DataSet API集成中](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#integration-with-datastream-and-dataset-api)讨论。

可以使用TableSink注册输出表。

### 注册表

Table在TableEnvironment中的注册如下:

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Table is the result of a simple projection query 
Table projTable = tableEnv.scan("X").select(...);

// register the Table projTable as table "projectedX"
tableEnv.registerTable("projectedTable", projTable);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// Table is the result of a simple projection query 
val projTable: Table = tableEnv.scan("X").select(...)

// register the Table projTable as table "projectedX"
tableEnv.registerTable("projectedTable", projTable)
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
注意：注册表的处理类似于关系数据库系统中的视图，定义的查询`Table`未经优化，但在另一个查询引用已注册的内容时将内联`Table。`如果多个查询引用相同的注册表，则每个引用查询都将内联并执行多次，即，注册表的结果将不会被共享。
{% endhint %}

### 注册TableSource

`TableSource`提供对存储在存储系统中的外部数据的访问，如数据库\(MySQL、HBase、…\)、具有特定编码的文件\(CSV、Apache \[Parquet、Avro、ORC\]、…\)或消息传递系统\(Apache Kafka、RabbitMQ、…\)。

Flink旨在为常见的数据格式和存储系统提供TableSource。请查看[Table Sources和Sinks](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sourceSinks.html)页面，获取受支持的TableSource列表以及如何构建自定义`TableSource`的说明。

`TableSource`在TableEnvironment中注册如下:

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// create a TableSource
TableSource csvSource = new CsvTableSource("/path/to/file", ...);

// register the TableSource as table "CsvTable"
tableEnv.registerTableSource("CsvTable", csvSource);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// create a TableSource
val csvSource: TableSource = new CsvTableSource("/path/to/file", ...)

// register the TableSource as table "CsvTable"
tableEnv.registerTableSource("CsvTable", csvSource)
```
{% endtab %}
{% endtabs %}

### 注册TableSink

已注册`TableSink`可用于将[Table API或SQL查询的结果](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#emit-a-table)发送到外部存储系统，例如数据库，键值存储，消息队列或文件系统（在不同的编码中，例如，CSV，Apache \[Parquet\] ，Avro，ORC\]，......）。

Flink旨在为通用数据格式和存储系统提供TableSinks。有关可用接收器的详细信息和如何实现自定义`TableSink`的说明，请参阅关于[Table Sources和Sink](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sourceSinks.html)页面的文档。

`TableSink`在TableEnvironment中注册如下:

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// create a TableSink
TableSink csvSink = new CsvTableSink("/path/to/file", ...);

// define the field names and types
String[] fieldNames = {"a", "b", "c"};
TypeInformation[] fieldTypes = {Types.INT, Types.STRING, Types.LONG};

// register the TableSink as table "CsvSinkTable"
tableEnv.registerTableSink("CsvSinkTable", fieldNames, fieldTypes, csvSink);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// create a TableSink
val csvSink: TableSink = new CsvTableSink("/path/to/file", ...)

// define the field names and types
val fieldNames: Array[String] = Array("a", "b", "c")
val fieldTypes: Array[TypeInformation[_]] = Array(Types.INT, Types.STRING, Types.LONG)

// register the TableSink as table "CsvSinkTable"
tableEnv.registerTableSink("CsvSinkTable", fieldNames, fieldTypes, csvSink)
```
{% endtab %}
{% endtabs %}

## 注册外部目录\(ExternalCatalog\)

外部目录可以提供有关外部数据库和表的信息，例如其名称，架构，统计信息以及有关如何访问存储在外部数据库，表或文件中的数据的信息。

可以通过实现`ExternalCatalog`接口创建外部目录，并在`TableEnvironment`以下内容中注册：

{% tabs %}
{% tab title="Java" %}
```text
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// create an external catalog
ExternalCatalog catalog = new InMemoryExternalCatalog();

// register the ExternalCatalog catalog
tableEnv.registerExternalCatalog("InMemCatalog", catalog);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// create an external catalog
val catalog: ExternalCatalog = new InMemoryExternalCatalog

// register the ExternalCatalog catalog
tableEnv.registerExternalCatalog("InMemCatalog", catalog)
```
{% endtab %}
{% endtabs %}

在表环境中注册之后，可以通过指定完整路径\(如catalog.database.table\)从Table API或SQL查询访问ExternalCatalog中定义的所有表。

目前，Flink为演示和测试目的提供了InMemoryExternalCatalog。但是，ExternalCatalog接口还可以用于将HCatalog或Metastore等目录连接到Table API。

## 查询表

### Table API

表API是Scala和Java的语言集成查询API。与SQL不同，查询不是作为字符串指定的，而是用宿主语言逐步组成的。

API基于表示表（流式或批处理）的表类，并提供了应用关系操作的方法。这些方法返回一个新的Table对象，它表示对输入Table应用关系操作的结果。一些关系操作由多个方法调用组成，例如table.group.\(...\)select\(\)，其中group.\(...\)指定表的分组，并select\(...\)表分组上的投影。

[表API](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/tableApi.html)文档介绍了支持流媒体和批次表中的所有表API操作。

以下示例是一个简单的Table API聚合查询：

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// register Orders table

// scan registered Orders table
Table orders = tableEnv.scan("Orders");
// compute revenue for all customers from France
Table revenue = orders
  .filter("cCountry === 'FRANCE'")
  .groupBy("cID, cName")
  .select("cID, cName, revenue.sum AS revSum");

// emit or convert Table
// execute query
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// register Orders table

// scan registered Orders table
val orders = tableEnv.scan("Orders")
// compute revenue for all customers from France
val revenue = orders
  .filter('cCountry === "FRANCE")
  .groupBy('cID, 'cName)
  .select('cID, 'cName, 'revenue.sum AS 'revSum)

// emit or convert Table
// execute query
```

{% hint style="danger" %}
注意:Scala表API使用Scala符号，这些符号以单引号\('\)开始引用表的属性。表API使用Scala implicits。为了使用Scala饮食转换要确保导入org.apache.flink.api.scala.\_和org.apache.flink.table.api.scala.\_
{% endhint %}
{% endtab %}
{% endtabs %}

### SQL

Flink的SQL集成基于[Apache Calcite](https://calcite.apache.org/)，它实现了SQL标准。SQL查询被指定为常规字符串。

该[SQL](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/sql.html)文件描述了Flink的流媒体和批量表的SQL支持。

以下示例显示如何指定查询并将结果作为a返回`Table`。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// register Orders table

// compute revenue for all customers from France
Table revenue = tableEnv.sqlQuery(
    "SELECT cID, cName, SUM(revenue) AS revSum " +
    "FROM Orders " +
    "WHERE cCountry = 'FRANCE' " +
    "GROUP BY cID, cName"
  );

// emit or convert Table
// execute query
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// register Orders table

// compute revenue for all customers from France
val revenue = tableEnv.sqlQuery("""
  |SELECT cID, cName, SUM(revenue) AS revSum
  |FROM Orders
  |WHERE cCountry = 'FRANCE'
  |GROUP BY cID, cName
  """.stripMargin)

// emit or convert Table
// execute query
```
{% endtab %}
{% endtabs %}

下面的示例说明如何指定将结果插入已注册表的更新查询。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// register "Orders" table
// register "RevenueFrance" output table

// compute revenue for all customers from France and emit to "RevenueFrance"
tableEnv.sqlUpdate(
    "INSERT INTO RevenueFrance " +
    "SELECT cID, cName, SUM(revenue) AS revSum " +
    "FROM Orders " +
    "WHERE cCountry = 'FRANCE' " +
    "GROUP BY cID, cName"
  );

// execute query
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// register "Orders" table
// register "RevenueFrance" output table

// compute revenue for all customers from France and emit to "RevenueFrance"
tableEnv.sqlUpdate("""
  |INSERT INTO RevenueFrance
  |SELECT cID, cName, SUM(revenue) AS revSum
  |FROM Orders
  |WHERE cCountry = 'FRANCE'
  |GROUP BY cID, cName
  """.stripMargin)

// execute query
```
{% endtab %}
{% endtabs %}

### 混合Table API和SQL

Table API和SQL查询可以轻松混合，因为它们都返回`Table`对象：

* 可以在`Table`SQL查询返回的对象上定义Table API 查询。
* 通过在TableEnvironment中[注册结果表](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#register-a-table)并在SQL查询的FROM子句中引用它，可以在表API查询的结果上定义SQL查询。

## 发出一张表

`Table`由`TableSink`写入发出。 `TableSink`是支持各种文件格式（例如CSV，Apache Parquet，Apache Avro），存储系统（例如，JDBC，Apache HBase，Apache Cassandra，Elasticsearch）或消息传递系统（例如，Apache Kafka，RabbitMQ）的通用接口。

批处理`Table`只能`BatchTableSink`写入，而流式处理`Table`需要`AppendStreamTableSink`，`RetractStreamTableSink`或`UpsertStreamTableSink`。

有关可用的Sink的详细信息和如何实现自定义Sink的说明，请参阅有关[Table Source和Sink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sourceSinks.html)的文档。

 `Table.insertInto(String tableName)` 方法将表发送到已注册的 `TableSink`。方法按名称从目录中查找 `TableSink`，并验证表的架构是否与 `TableSink`的架构相同。

以下示例展示如何发出`Table`：

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// create a TableSink
TableSink sink = new CsvTableSink("/path/to/file", fieldDelim = "|");

// register the TableSink with a specific schema
String[] fieldNames = {"a", "b", "c"};
TypeInformation[] fieldTypes = {Types.INT, Types.STRING, Types.LONG};
tableEnv.registerTableSink("CsvSinkTable", fieldNames, fieldTypes, sink);

// compute a result Table using Table API operators and/or SQL queries
Table result = ...
// emit the result Table to the registered TableSink
result.insertInto("CsvSinkTable");

// execute the program
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// create a TableSink
val sink: TableSink = new CsvTableSink("/path/to/file", fieldDelim = "|")

// register the TableSink with a specific schema
val fieldNames: Array[String] = Array("a", "b", "c")
val fieldTypes: Array[TypeInformation] = Array(Types.INT, Types.STRING, Types.LONG)
tableEnv.registerTableSink("CsvSinkTable", fieldNames, fieldTypes, sink)

// compute a result Table using Table API operators and/or SQL queries
val result: Table = ...

// emit the result Table to the registered TableSink
result.insertInto("CsvSinkTable")

// execute the program
```
{% endtab %}
{% endtabs %}

## 转换并执行查询

表API和SQL查询将转换为[DataStream](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html)或[DataSet](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch)程序，具体取决于它们的输入是流式还是批量输入。查询在内部表示为逻辑查询计划，并分为两个阶段：

1. 优化逻辑计划，
2. 转换为DataStream或DataSet程序。

在以下情况下转换Table API或SQL查询：

* Table被发送到TableSink，即，当调用Table.insertInto\(\)时。 
* 指定一个SQL更新查询，即，当调用`TableEnvironment.sqlUpdate()`时。 Table被转换为DataStream或DataSet\(请参阅[与DataStream和DataSet API集成](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#integration-with-dataStream-and-dataSet-api)\)。

一旦翻译，Table API或SQL查询像一个普通的数据流中或数据集处理程序，当`StreamExecutionEnvironment.execute()`或者`ExecutionEnvironment.execute()`被执行时调用。

## 与DataStream和DataSet API集成

Table API和SQL查询可以很容易地集成并嵌入到DataStream和DataSet程序中。例如,可以查询外部表\(例如从RDBMS\),做一些预处理,如过滤、投射、聚合,或加入元数据,然后使用DataStream或进一步处理数据，DataSet API（以及在这些API之上构建的任何库，例如CEP或Gelly）。相反，Table API或SQL查询也可以应用于DataStream或DataSet程序的结果。

这种交互可以通过将DataStream或DataSet转换为Table来实现，反之亦然。在本节中，我们将描述如何进行这些转换。

### Scala的隐式转换

Scala表API为DataSet、DataStream和Table类提供了隐式转换。除了为Scala DataStream API导入org.apache.flink.table.api.scala._之外，还导入org.apache.flink.api.scala._包来启用这些转换。

### 将DataStream或DataSet注册为Table

DataStream或DataSet可以在TableEnvironment中注册为表。结果表的模式取决于注册的DataStream或DataSet的数据类型。有关将[数据类型映射到表结构](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#mapping-of-data-types-to-table-schema)的详细信息，请参阅有关部分。

{% tabs %}
{% tab title="Java" %}
```java
// get StreamTableEnvironment
// registration of a DataSet in a BatchTableEnvironment is equivalent
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Long, String>> stream = ...

// register the DataStream as Table "myTable" with fields "f0", "f1"
tableEnv.registerDataStream("myTable", stream);

// register the DataStream as table "myTable2" with fields "myLong", "myString"
tableEnv.registerDataStream("myTable2", stream, "myLong, myString");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get TableEnvironment 
// registration of a DataSet is equivalent
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[(Long, String)] = ...

// register the DataStream as Table "myTable" with fields "f0", "f1"
tableEnv.registerDataStream("myTable", stream)

// register the DataStream as table "myTable2" with fields "myLong", "myString"
tableEnv.registerDataStream("myTable2", stream, 'myLong, 'myString)
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
注意:DataStreamTable的名称必须与^_DataStreamTable_\[0-9\]+模式不匹配，DataSetTable的名称必须与^_DataSetTable_\[0-9\]+模式不匹配。这些模式仅供内部使用。
{% endhint %}

### 将DataStream或DataSet转换为Table

它也可以直接转换为Table，而不是在TableEnvironment中注册一个DataStream或DataSet。如果您想在表API查询中使用该表，这非常方便。

{% tabs %}
{% tab title="Java" %}
```java
// get StreamTableEnvironment
// registration of a DataSet in a BatchTableEnvironment is equivalent
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Long, String>> stream = ...

// Convert the DataStream into a Table with default fields "f0", "f1"
Table table1 = tableEnv.fromDataStream(stream);

// Convert the DataStream into a Table with fields "myLong", "myString"
Table table2 = tableEnv.fromDataStream(stream, "myLong, myString");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get TableEnvironment
// registration of a DataSet is equivalent
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[(Long, String)] = ...

// convert the DataStream into a Table with default fields '_1, '_2
val table1: Table = tableEnv.fromDataStream(stream)

// convert the DataStream into a Table with fields 'myLong, 'myString
val table2: Table = tableEnv.fromDataStream(stream, 'myLong, 'myString)
```
{% endtab %}
{% endtabs %}

### 将Table转换为DataStream或DataSet

Table可以转换为DataStream或DataSet。通过这种方式，可以在Table API或SQL查询的结果上运行定制的DataStream或DataSet程序。

在将Table转换为DataStream或DataSet时，需要指定结果DataStream或DataSet的数据类型，即，将Table的行转换为的数据类型。通常最方便的转换类型是Row。以下列表概述了不同选项的功能:

* **Row**:字段按位置映射，字段数量任意，支持null值，没有类型安全访问。 
* **POJO**:字段按名称映射\(POJO字段必须命名为表字段\)、任意数量的字段、支持null值、类型安全访问。
* **Case Class**:字段按位置映射，不支持null值，类型安全访问。 
* **Tuple**:字段按位置映射，限制为22 \(Scala\)或25 \(Java\)字段，不支持null值，类型安全访问。 
* **Atomic Type**:表必须有一个字段，不支持null值，类型安全访问。

#### **将表转换为DataStream**

{% tabs %}
{% tab title="Java" %}
```java
// get StreamTableEnvironment. 
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Table with two fields (String name, Integer age)
Table table = ...

// convert the Table into an append DataStream of Row by specifying the class
DataStream<Row> dsRow = tableEnv.toAppendStream(table, Row.class);

// convert the Table into an append DataStream of Tuple2<String, Integer> 
//   via a TypeInformation
TupleTypeInfo<Tuple2<String, Integer>> tupleType = new TupleTypeInfo<>(
  Types.STRING(),
  Types.INT());
DataStream<Tuple2<String, Integer>> dsTuple = 
  tableEnv.toAppendStream(table, tupleType);

// convert the Table into a retract DataStream of Row.
//   A retract stream of type X is a DataStream<Tuple2<Boolean, X>>. 
//   The boolean field indicates the type of the change. 
//   True is INSERT, false is DELETE.
DataStream<Tuple2<Boolean, Row>> retractStream = 
  tableEnv.toRetractStream(table, Row.class);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get TableEnvironment. 
// registration of a DataSet is equivalent
val tableEnv = TableEnvironment.getTableEnvironment(env)

// Table with two fields (String name, Integer age)
val table: Table = ...

// convert the Table into an append DataStream of Row
val dsRow: DataStream[Row] = tableEnv.toAppendStream[Row](table)

// convert the Table into an append DataStream of Tuple2[String, Int]
val dsTuple: DataStream[(String, Int)] dsTuple = 
  tableEnv.toAppendStream[(String, Int)](table)

// convert the Table into a retract DataStream of Row.
//   A retract stream of type X is a DataStream[(Boolean, X)]. 
//   The boolean field indicates the type of the change. 
//   True is INSERT, false is DELETE.
val retractStream: DataStream[(Boolean, Row)] = tableEnv.toRetractStream[Row](table)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**注意：**[动态表](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/streaming/dynamic_tables.html)文档中给出了有关动态表及其属性的详细讨论。
{% endhint %}

#### **将Table转换为DataSet**

Table转换为DataSet如下：

{% tabs %}
{% tab title="Java" %}
```java
// get BatchTableEnvironment
BatchTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Table with two fields (String name, Integer age)
Table table = ...

// convert the Table into a DataSet of Row by specifying a class
DataSet<Row> dsRow = tableEnv.toDataSet(table, Row.class);

// convert the Table into a DataSet of Tuple2<String, Integer> via a TypeInformation
TupleTypeInfo<Tuple2<String, Integer>> tupleType = new TupleTypeInfo<>(
  Types.STRING(),
  Types.INT());
DataSet<Tuple2<String, Integer>> dsTuple = 
  tableEnv.toDataSet(table, tupleType);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get TableEnvironment 
// registration of a DataSet is equivalent
val tableEnv = TableEnvironment.getTableEnvironment(env)

// Table with two fields (String name, Integer age)
val table: Table = ...

// convert the Table into a DataSet of Row
val dsRow: DataSet[Row] = tableEnv.toDataSet[Row](table)

// convert the Table into a DataSet of Tuple2[String, Int]
val dsTuple: DataSet[(String, Int)] = tableEnv.toDataSet[(String, Int)](table)
```
{% endtab %}
{% endtabs %}

### 将数据类型映射到Table Schema

Flink的DataStream和DataSet api支持非常不同的类型。复合类型，如元组\(内置Scala和Flink Java元组\)、pojo、Scala case类和Flink的Row类型，允许嵌套的数据结构具有多个字段，可以在Table表达式中访问这些字段。其他类型被视为原子类型。下面，我们将描述Table API如何将这些类型转换为内部行表示，并展示将DataStream转换为Table的示例。

数据类型到Table Schema的映射可以以两种方式发生：**基于字段位置**或**基于字段名称**。

#### **基于位置的映射**

基于位置的映射可用于在保持字段顺序的同时为字段指定更有意义的名称。此映射可用于具有已定义字段顺序的组合数据类型以及原子类型。`Tuple`、`Row`和`Case Class`等复合数据类型具有这样的字段顺序。但是，必须根据字段名映射POJO的字段\(参见下一节\)。

在定义基于位置的映射时，输入数据类型中必须不存在指定的名称，否则API将假定映射应该基于字段名称进行。如果没有指定字段名，则使用组合类型的默认字段名和字段顺序，或者原子类型使用f0。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Long, Integer>> stream = ...

// convert DataStream into Table with default field names "f0" and "f1"
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with field names "myLong" and "myInt"
Table table = tableEnv.fromDataStream(stream, "myLong, myInt");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[(Long, Int)] = ...

// convert DataStream into Table with default field names "_1" and "_2"
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with field names "myLong" and "myInt"
val table: Table = tableEnv.fromDataStream(stream, 'myLong 'myInt)
```
{% endtab %}
{% endtabs %}

#### **基于名称的映射**

基于名称的映射可以用于任何数据类型，包括**POJO**。这是定义表模式映射最灵活的方法。映射中的所有字段都由name引用，可以使用别名as重命名。字段可以重新排序并投影。

如果没有指定字段名，则使用组合类型的默认字段名和字段顺序，或者原子类型使用f0。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Long, Integer>> stream = ...

// convert DataStream into Table with default field names "f0" and "f1"
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with field "f1" only
Table table = tableEnv.fromDataStream(stream, "f1");

// convert DataStream into Table with swapped fields
Table table = tableEnv.fromDataStream(stream, "f1, f0");

// convert DataStream into Table with swapped fields and field names "myInt" and "myLong"
Table table = tableEnv.fromDataStream(stream, "f1 as myInt, f0 as myLong");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[(Long, Int)] = ...

// convert DataStream into Table with default field names "_1" and "_2"
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with field "_2" only
val table: Table = tableEnv.fromDataStream(stream, '_2)

// convert DataStream into Table with swapped fields
val table: Table = tableEnv.fromDataStream(stream, '_2, '_1)

// convert DataStream into Table with swapped fields and field names "myInt" and "myLong"
val table: Table = tableEnv.fromDataStream(stream, '_2 as 'myInt, '_1 as 'myLong)
```
{% endtab %}
{% endtabs %}

#### **Atomic Types**

Flink将原语\(**Integer**、**Double**、**String**\)或泛型类型\(无法分析和分解的类型\)视为原子类型。原子类型的DataStream或数据集转换为具有单个属性的表。属性的类型是从原子类型推断出来的，可以指定属性的名称。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Long> stream = ...

// convert DataStream into Table with default field name "f0"
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with field name "myLong"
Table table = tableEnv.fromDataStream(stream, "myLong");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[Long] = ...

// convert DataStream into Table with default field name "f0"
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with field name "myLong"
val table: Table = tableEnv.fromDataStream(stream, 'myLong)
```
{% endtab %}
{% endtabs %}

#### **Tuple（Scala和Java）和Case  Class（仅限Scala）**

Flink支持Scala的内置元组，并为Java提供自己的元组类。两种元组的DataStream和DataSet都可以转换为表。可以通过为所有字段提供名称（基于位置的映射）来重命名字段。如果未指定字段名称，则使用默认字段名称。如果原始字段名（`f0`，`f1`，...为Flink元组和`_1`，`_2`..Scala元组）被引用时，API假设映射，而不是基于位置的基于域名的。基于名称的映射允许使用别名（`as`）重新排序字段和投影。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Long, String>> stream = ...

// convert DataStream into Table with default field names "f0", "f1"
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed field names "myLong", "myString" (position-based)
Table table = tableEnv.fromDataStream(stream, "myLong, myString");

// convert DataStream into Table with reordered fields "f1", "f0" (name-based)
Table table = tableEnv.fromDataStream(stream, "f1, f0");

// convert DataStream into Table with projected field "f1" (name-based)
Table table = tableEnv.fromDataStream(stream, "f1");

// convert DataStream into Table with reordered and aliased fields "myString", "myLong" (name-based)
Table table = tableEnv.fromDataStream(stream, "f1 as 'myString', f0 as 'myLong'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

val stream: DataStream[(Long, String)] = ...

// convert DataStream into Table with renamed default field names '_1, '_2
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with field names "myLong", "myString" (position-based)
val table: Table = tableEnv.fromDataStream(stream, 'myLong, 'myString)

// convert DataStream into Table with reordered fields "_2", "_1" (name-based)
val table: Table = tableEnv.fromDataStream(stream, '_2, '_1)

// convert DataStream into Table with projected field "_2" (name-based)
val table: Table = tableEnv.fromDataStream(stream, '_2)

// convert DataStream into Table with reordered and aliased fields "myString", "myLong" (name-based)
val table: Table = tableEnv.fromDataStream(stream, '_2 as 'myString, '_1 as 'myLong)

// define case class
case class Person(name: String, age: Int)
val streamCC: DataStream[Person] = ...

// convert DataStream into Table with default field names 'name, 'age
val table = tableEnv.fromDataStream(streamCC)

// convert DataStream into Table with field names 'myName, 'myAge (position-based)
val table = tableEnv.fromDataStream(streamCC, 'myName, 'myAge)

// convert DataStream into Table with reordered and aliased fields "myAge", "myName" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'age as 'myAge, 'name as 'myName)
```
{% endtab %}
{% endtabs %}

#### **POJO（Java和Scala）**

Flink支持POJO作为复合类型。[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#pojos)记录了决定POJO的规则。

在不指定字段名的情况下将`POJO DataStream`或`DataSet`转换为`Table`时，将使用原始POJO字段的名称。名称映射需要原始名称，不能按位置进行。字段可以使用别名\(使用as关键字\)重命名、重新排序和投影。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// Person is a POJO with fields "name" and "age"
DataStream<Person> stream = ...

// convert DataStream into Table with default field names "age", "name" (fields are ordered by name!)
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed fields "myAge", "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, "age as myAge, name as myName");

// convert DataStream into Table with projected field "name" (name-based)
Table table = tableEnv.fromDataStream(stream, "name");

// convert DataStream into Table with projected and renamed field "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, "name as myName");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// Person is a POJO with field names "name" and "age"
val stream: DataStream[Person] = ...

// convert DataStream into Table with default field names "age", "name" (fields are ordered by name!)
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with renamed fields "myAge", "myName" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'age as 'myAge, 'name as 'myName)

// convert DataStream into Table with projected field "name" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'name)

// convert DataStream into Table with projected and renamed field "myName" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'name as 'myName)
```
{% endtab %}
{% endtabs %}

#### **Row**

Row数据类型支持任意数量的字段和空值字段。字段名可以通过`RowTypeInfo`指定，也可以在将`Row DataStream`或`DataSet`转换为`Table`时指定。Row类型支持按位置和名称映射字段。可以通过为所有字段\(基于位置的映射\)提供名称来重命名字段，或者为投影/排序/重命名\(基于名称的映射\)单独选择字段。

{% tabs %}
{% tab title="Java" %}
```java
// get a StreamTableEnvironment, works for BatchTableEnvironment equivalently
StreamTableEnvironment tableEnv = TableEnvironment.getTableEnvironment(env);

// DataStream of Row with two fields "name" and "age" specified in `RowTypeInfo`
DataStream<Row> stream = ...

// convert DataStream into Table with default field names "name", "age"
Table table = tableEnv.fromDataStream(stream);

// convert DataStream into Table with renamed field names "myName", "myAge" (position-based)
Table table = tableEnv.fromDataStream(stream, "myName, myAge");

// convert DataStream into Table with renamed fields "myName", "myAge" (name-based)
Table table = tableEnv.fromDataStream(stream, "name as myName, age as myAge");

// convert DataStream into Table with projected field "name" (name-based)
Table table = tableEnv.fromDataStream(stream, "name");

// convert DataStream into Table with projected and renamed field "myName" (name-based)
Table table = tableEnv.fromDataStream(stream, "name as myName");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tableEnv = TableEnvironment.getTableEnvironment(env)

// DataStream of Row with two fields "name" and "age" specified in `RowTypeInfo`
val stream: DataStream[Row] = ...

// convert DataStream into Table with default field names "name", "age"
val table: Table = tableEnv.fromDataStream(stream)

// convert DataStream into Table with renamed field names "myName", "myAge" (position-based)
val table: Table = tableEnv.fromDataStream(stream, 'myName, 'myAge)

// convert DataStream into Table with renamed fields "myName", "myAge" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'name as 'myName, 'age as 'myAge)

// convert DataStream into Table with projected field "name" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'name)

// convert DataStream into Table with projected and renamed field "myName" (name-based)
val table: Table = tableEnv.fromDataStream(stream, 'name as 'myName)
```
{% endtab %}
{% endtabs %}

## 查询优化

Apache Flink利用Apache Calcite优化和翻译查询。当前执行的优化包括投影和过滤 下推、子查询去相关以及其他类型的查询重写。Flink还没有优化连接的顺序，但是按照查询中定义的顺序执行它们\(FROM子句中的表的顺序和/或WHERE子句中的连接谓词的顺序\)。

通过提供CalciteConfig对象，可以调整应用于不同阶段的优化规则集。这可以通过调用calciteConfig.createbuilder\(\)来创建，并通过调用tableEnv.getConfig.setCalciteConfig\(calciteConfig\)提供给TableEnvironment。

### 解释表

Table API提供了一种机制来解释计算表的逻辑和优化查询计划。这是通过TableEnvironment.explain\(table\)方法完成的。它返回一个字符串描述三个计划:

1. 关系查询的抽象语法树，即未优化的逻辑查询计划
2. 优化后的逻辑查询计划
3. 物理执行计划

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = TableEnvironment.getTableEnvironment(env);

DataStream<Tuple2<Integer, String>> stream1 = env.fromElements(new Tuple2<>(1, "hello"));
DataStream<Tuple2<Integer, String>> stream2 = env.fromElements(new Tuple2<>(1, "hello"));

Table table1 = tEnv.fromDataStream(stream1, "count, word");
Table table2 = tEnv.fromDataStream(stream2, "count, word");
Table table = table1
  .where("LIKE(word, 'F%')")
  .unionAll(table2);

String explanation = tEnv.explain(table);
System.out.println(explanation);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tEnv = TableEnvironment.getTableEnvironment(env)

val table1 = env.fromElements((1, "hello")).toTable(tEnv, 'count, 'word)
val table2 = env.fromElements((1, "hello")).toTable(tEnv, 'count, 'word)
val table = table1
  .where('word.like("F%"))
  .unionAll(table2)

val explanation: String = tEnv.explain(table)
println(explanation)
```
{% endtab %}
{% endtabs %}

```text
== Abstract Syntax Tree ==
LogicalUnion(all=[true])
  LogicalFilter(condition=[LIKE($1, 'F%')])
    LogicalTableScan(table=[[_DataStreamTable_0]])
  LogicalTableScan(table=[[_DataStreamTable_1]])

== Optimized Logical Plan ==
DataStreamUnion(union=[count, word])
  DataStreamCalc(select=[count, word], where=[LIKE(word, 'F%')])
    DataStreamScan(table=[[_DataStreamTable_0]])
  DataStreamScan(table=[[_DataStreamTable_1]])

== Physical Execution Plan ==
Stage 1 : Data Source
  content : collect elements with CollectionInputFormat

Stage 2 : Data Source
  content : collect elements with CollectionInputFormat

  Stage 3 : Operator
    content : from: (count, word)
    ship_strategy : REBALANCE

    Stage 4 : Operator
      content : where: (LIKE(word, 'F%')), select: (count, word)
      ship_strategy : FORWARD

      Stage 5 : Operator
        content : from: (count, word)
        ship_strategy : REBALANCE
```

