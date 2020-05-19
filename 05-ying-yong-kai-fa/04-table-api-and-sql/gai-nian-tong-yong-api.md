# 概念&通用API

Table API和SQL集成在一个联合API中。此API的核心概念是`Table`用作查询的输入和输出。本文档显示了具有Table API和SQL查询的程序的常见结构，如何注册`Table`，如何查询`Table`以及如何发出`Table`。

## 两个Planner之间的主要区别

1. Blink将批处理作业视为流处理的特殊情况。因此，表和数据集之间的转换也不受支持，批处理作业不会被转换为DateSet程序，而是转换为DataStream程序，这与流作业相同。
2. Blink Planner不支持BatchTableSource，使用有界的StreamTableSource代替它。
3. Blink Planner只支持全新的目录，不支持已弃用的ExternalCatalog。
4. 老Planner和Blink Planner的`FilterableTableSource`的实现是不兼容的。旧的计划表将向下推`PlannerExpressions`到`FilterableTableSource`中，而Blink计划表将向下推 `Expressions`
5.  基于字符串的键值配置选项（有关详细信息，请参阅有关[配置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/config.html)的文档）仅用于Blink计划器。
6.  两个Planner的`PlannerConfig`实现（`CalciteConfig`）不同。
7.  Blink Planner会将多个接收器优化为一个DAG（仅在上支持`TableEnvironment`，而不在上支持`StreamTableEnvironment`）。旧的Planner将始终将每个接收器优化为一个新的DAG，其中所有DAG彼此独立。
8. 旧的Planner现在不支持目录统计信息，而Blink Planner则支持。

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

{% tab title="Python" %}
```python
# create a TableEnvironment for specific planner batch or streaming
table_env = ... # see "Create a TableEnvironment" section

# register a Table
table_env.connect(...).create_temporary_table("table1")

# register an output Table
table_env.connect(...).create_temporary_table("outputTable")

# create a Table from a Table API query
tapi_result = table_env.from_path("table1").select(...)
# create a Table from a SQL query
sql_result  = table_env.sql_query("SELECT ... FROM table1 ...")

# emit a Table API result Table to a TableSink, same for SQL result
tapi_result.insert_into("outputTable")

# execute
table_env.execute("python_job")
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**注意：**Table API和SQL查询可以轻松集成并嵌入到DataStream或DataSet程序中。查看[与DataStream和DataSet API](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#integration-with-datastream-and-dataset-api)的[集成](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#integration-with-datastream-and-dataset-api)部分，了解如何将DataStream和DataSet转换为Tables，反之亦然。
{% endhint %}

## 创建TableEnvironment

`TableEnvironment`是Table API和SQL集成的核心概念。它负责：

*  `Table`在内部目录中注册 
* 注册目录 
* 加载可插拔模块
* 执行SQL查询 
* 注册用户定义的\(标量、表或聚合\)函数 
*  将`DataStream`或`DataSet`转换为`Table`
* 保存对ExecutionEnvironment或StreamExecutionEnvironment的引用

表总是绑定到特定的表环境。在同一个查询中组合不同表环境的表是不可能的，例如，联接或联合它们。

TableEnvironment是通过调用静态TableEnvironment. getTableEnvironment\(\)方法创建的，该方法带有一个StreamExecutionEnvironment或一个ExecutionEnvironment以及一个可选的TableConfig。TableConfig可用于配置TableEnvironment或自定义查询优化和转换过程\(参见[查询优化](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/common.html#query-optimization)\)。

确保选择与您的编程语言相匹配的特定计划器`BatchTableEnvironment`/`StreamTableEnvironment`

如果两个Planner jar都在类路径上（默认行为），则应明确设置当前要在程序中使用的Planner。

{% tabs %}
{% tab title="Java" %}
```java
// **********************
// FLINK STREAMING QUERY
// **********************
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.java.StreamTableEnvironment;

EnvironmentSettings fsSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build();
StreamExecutionEnvironment fsEnv = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment fsTableEnv = StreamTableEnvironment.create(fsEnv, fsSettings);
// or TableEnvironment fsTableEnv = TableEnvironment.create(fsSettings);

// ******************
// FLINK BATCH QUERY
// ******************
import org.apache.flink.api.java.ExecutionEnvironment;
import org.apache.flink.table.api.java.BatchTableEnvironment;

ExecutionEnvironment fbEnv = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment fbTableEnv = BatchTableEnvironment.create(fbEnv);

// **********************
// BLINK STREAMING QUERY
// **********************
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.java.StreamTableEnvironment;

StreamExecutionEnvironment bsEnv = StreamExecutionEnvironment.getExecutionEnvironment();
EnvironmentSettings bsSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();
StreamTableEnvironment bsTableEnv = StreamTableEnvironment.create(bsEnv, bsSettings);
// or TableEnvironment bsTableEnv = TableEnvironment.create(bsSettings);

// ******************
// BLINK BATCH QUERY
// ******************
import org.apache.flink.table.api.EnvironmentSettings;
import org.apache.flink.table.api.TableEnvironment;

EnvironmentSettings bbSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build();
TableEnvironment bbTableEnv = TableEnvironment.create(bbSettings);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// **********************
// FLINK STREAMING QUERY
// **********************
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.table.api.EnvironmentSettings
import org.apache.flink.table.api.scala.StreamTableEnvironment

val fsSettings = EnvironmentSettings.newInstance().useOldPlanner().inStreamingMode().build()
val fsEnv = StreamExecutionEnvironment.getExecutionEnvironment
val fsTableEnv = StreamTableEnvironment.create(fsEnv, fsSettings)
// or val fsTableEnv = TableEnvironment.create(fsSettings)

// ******************
// FLINK BATCH QUERY
// ******************
import org.apache.flink.api.scala.ExecutionEnvironment
import org.apache.flink.table.api.scala.BatchTableEnvironment

val fbEnv = ExecutionEnvironment.getExecutionEnvironment
val fbTableEnv = BatchTableEnvironment.create(fbEnv)

// **********************
// BLINK STREAMING QUERY
// **********************
import org.apache.flink.streaming.api.scala.StreamExecutionEnvironment
import org.apache.flink.table.api.EnvironmentSettings
import org.apache.flink.table.api.scala.StreamTableEnvironment

val bsEnv = StreamExecutionEnvironment.getExecutionEnvironment
val bsSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build()
val bsTableEnv = StreamTableEnvironment.create(bsEnv, bsSettings)
// or val bsTableEnv = TableEnvironment.create(bsSettings)

// ******************
// BLINK BATCH QUERY
// ******************
import org.apache.flink.table.api.{EnvironmentSettings, TableEnvironment}

val bbSettings = EnvironmentSettings.newInstance().useBlinkPlanner().inBatchMode().build()
val bbTableEnv = TableEnvironment.create(bbSettings)
```
{% endtab %}

{% tab title="Python" %}
```python
# **********************
# FLINK STREAMING QUERY
# **********************
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment, EnvironmentSettings

f_s_env = StreamExecutionEnvironment.get_execution_environment()
f_s_settings = EnvironmentSettings.new_instance().use_old_planner().in_streaming_mode().build()
f_s_t_env = StreamTableEnvironment.create(f_s_env, environment_settings=f_s_settings)

# ******************
# FLINK BATCH QUERY
# ******************
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import BatchTableEnvironment

f_b_env = ExecutionEnvironment.get_execution_environment()
f_b_t_env = BatchTableEnvironment.create(f_b_env, table_config)

# **********************
# BLINK STREAMING QUERY
# **********************
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment, EnvironmentSettings

b_s_env = StreamExecutionEnvironment.get_execution_environment()
b_s_settings = EnvironmentSettings.new_instance().use_blink_planner().in_streaming_mode().build()
b_s_t_env = StreamTableEnvironment.create(b_s_env, environment_settings=b_s_settings)

# ******************
# BLINK BATCH QUERY
# ******************
from pyflink.table import EnvironmentSettings, BatchTableEnvironment

b_b_settings = EnvironmentSettings.new_instance().use_blink_planner().in_batch_mode().build()
b_b_t_env = BatchTableEnvironment.create(environment_settings=b_b_settings)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
 **注意：**如果`/lib`目录中只有一个计划器jar ，则可以使用`useAnyPlanner(`

对于python使用`use_any_planner`）创建指定的`EnvironmentSettings`。
{% endhint %}

## 通过Catalogs创建表

 `TableEnvironment`维护使用标识符创建的表目录的映射。每个标识符由3部分组成：目录名称，数据库名称和对象名称。如果未指定目录或数据库，则将使用当前默认值（请参阅[表标识符扩展](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html#table-identifier-expanding)部分中的示例）。

 表可以是虚拟（`VIEWS`）或常规（`TABLES`）。`VIEWS`可以从现有`Table`对象创建，通常是Table API或SQL查询的结果。`TABLES`描述外部数据，例如文件，数据库表或消息队列**。**

### 临时表与永久表

表可以是临时的，并与单个Flink会话的生命周期相关，也可以是永久的，并且在多个Flink会话和群集中可见。

 永久表需要一个[目录](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)（例如Hive Metastore）来维护有关表的元数据。创建永久表后，连接到目录的任何Flink会话都可以看到该表，并且该表将一直存在，直到明确删除该表为止。

另一方面，临时表始终存储在内存中，并且仅在它们在其中创建的Flink会话期间存在。这些表对其他会话不可见。它们未绑定到任何目录或数据库，但可以在一个目录或数据库的名称空间中创建。如果删除了临时表的相应数据库，则不会删除这些临时表。

#### **遮蔽**

可以使用与现有永久表相同的标识符注册一个临时表。只要存在临时表，该临时表就会覆盖该永久表，并使该永久表不可访问。具有该标识符的所有查询将针对临时表执行。

这可能对实验有用。它允许首先对临时表运行完全相同的查询，该临时表例如仅具有数据的一个子集，或者对数据进行混淆。一旦验证查询正确无误，就可以针对实际生产表运行该查询。

### 创建表

#### 虚拟表

表API对象对应于SQL术语中的视图\(虚拟表\)。它封装了一个逻辑查询计划。它可以在目录中创建如下:

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

{% tab title="Python" %}
```python
# get a TableEnvironment
table_env = ... # see "Create a TableEnvironment" section

# table is the result of a simple projection query 
proj_table = table_env.from_path("X").select(...)

# register the Table projTable as table "projectedTable"
table_env.register_table("projectedTable", proj_table)
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**注意**：表对象类似于关系数据库系统中的视图。定义表的查询未优化，但当另一个查询引用已注册表时，将内联该查询。如果多个查询引用同一个已注册表，则每个引用查询都将被内联并执行多次，即，注册表的结果将不会被共享。
{% endhint %}

#### 连接器表

还可以通过连接器声明从关系数据库创建已知的表。连接器描述了存储表数据的外部系统。可以在此处声明诸如Apacha Kafka之类的存储系统或常规文件系统。

{% tabs %}
{% tab title="Java" %}
```java
tableEnvironment
  .connect(...)
  .withFormat(...)
  .withSchema(...)
  .inAppendMode()
  .createTemporaryTable("MyTable")
```
{% endtab %}

{% tab title="Scala" %}
```scala
tableEnvironment
  .connect(...)
  .withFormat(...)
  .withSchema(...)
  .inAppendMode()
  .createTemporaryTable("MyTable")
```
{% endtab %}

{% tab title="Python" %}
```python
table_environment \
    .connect(...) \
    .with_format(...) \
    .with_schema(...) \
    .in_append_mode() \
    .create_temporary_table("MyTable")
```
{% endtab %}

{% tab title="DDL" %}
```text
tableEnvironment.sqlUpdate("CREATE [TEMPORARY] TABLE MyTable (...) WITH (...)")
```
{% endtab %}
{% endtabs %}

### 扩展表标识符

表总是用由目录、数据库和表名组成的三部分标识符注册。

用户可以将其中的一个目录和一个数据库设置为“当前目录”和“当前数据库”。有了它们，上面提到的3-parts标识符中的前两个部分可以是可选的——如果没有提供它们，将引用当前目录和当前数据库。用户可以通过表API或SQL切换当前目录和当前数据库。

 标识符遵循SQL要求，这意味着可以使用反引号（`````）对其进行转义。此外，所有SQL保留关键字都必须转义。

{% tabs %}
{% tab title="Java" %}
```java
TableEnvironment tEnv = ...;
tEnv.useCatalog("custom_catalog");
tEnv.useDatabase("custom_database");

Table table = ...;

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("exampleView", table);

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_database.exampleView", table);

// register the view named 'View' in the catalog named 'custom_catalog' in the
// database named 'custom_database'. 'View' is a reserved keyword and must be escaped.  
tableEnv.createTemporaryView("`View`", table);

// register the view named 'example.View' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("`example.View`", table);

// register the view named 'exampleView' in the catalog named 'other_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_catalog.other_database.exampleView", table);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// get a TableEnvironment
val tEnv: TableEnvironment = ...;
tEnv.useCatalog("custom_catalog")
tEnv.useDatabase("custom_database")

val table: Table = ...;

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("exampleView", table)

// register the view named 'exampleView' in the catalog named 'custom_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_database.exampleView", table)

// register the view named 'View' in the catalog named 'custom_catalog' in the
// database named 'custom_database'. 'View' is a reserved keyword and must be escaped.  
tableEnv.createTemporaryView("`View`", table)

// register the view named 'example.View' in the catalog named 'custom_catalog'
// in the database named 'custom_database' 
tableEnv.createTemporaryView("`example.View`", table)

// register the view named 'exampleView' in the catalog named 'other_catalog'
// in the database named 'other_database' 
tableEnv.createTemporaryView("other_catalog.other_database.exampleView", table)
```
{% endtab %}
{% endtabs %}

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
```python
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
{% endtab %}

{% tab title="Python" %}
```text
# get a TableEnvironment
table_env = # see "Create a TableEnvironment" section

# register Orders table

# scan registered Orders table
orders = table_env.from_path("Orders")
# compute revenue for all customers from France
revenue = orders \
    .filter("cCountry === 'FRANCE'") \
    .group_by("cID, cName") \
    .select("cID, cName, revenue.sum AS revSum")

# emit or convert Table
# execute query

```
{% endtab %}
{% endtabs %}

### SQL

Flink的SQL集成基于实现SQL标准的[Apache Calcite](https://calcite.apache.org/)。SQL查询被指定为常规字符串。

该[SQL](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sql/index.html)文件描述弗林克的流媒体和批量表的SQL支持。

以下示例说明如何指定查询并以返回结果`Table`。

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

{% tab title="Python" %}
```python
# get a TableEnvironment
table_env = ... # see "Create a TableEnvironment" section

# register Orders table

# compute revenue for all customers from France
revenue = table_env.sql_query(
    "SELECT cID, cName, SUM(revenue) AS revSum "
    "FROM Orders "
    "WHERE cCountry = 'FRANCE' "
    "GROUP BY cID, cName"
)

# emit or convert Table
# execute query
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

{% tab title="" %}
```python
# get a TableEnvironment
table_env = ... # see "Create a TableEnvironment" section

# register "Orders" table
# register "RevenueFrance" output table

# compute revenue for all customers from France and emit to "RevenueFrance"
table_env.sql_update(
    "INSERT INTO RevenueFrance "
    "SELECT cID, cName, SUM(revenue) AS revSum "
    "FROM Orders "
    "WHERE cCountry = 'FRANCE' "
    "GROUP BY cID, cName"
)

# execute query
```
{% endtab %}
{% endtabs %}

### 混合Table API和SQL

Table API和SQL查询可以轻松混合，因为它们都返回`Table`对象：

* 可以在`TableSQL`查询返回的对象上定义Table API 查询。
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

{% tab title="Python" %}
```python
# get a TableEnvironment
table_env = ... # see "Create a TableEnvironment" section

# create a TableSink
t_env.connect(FileSystem().path("/path/to/file")))
    .with_format(Csv()
                 .field_delimiter(',')
                 .deriveSchema())
    .with_schema(Schema()
                 .field("a", DataTypes.INT())
                 .field("b", DataTypes.STRING())
                 .field("c", DataTypes.BIGINT()))
    .create_temporary_table("CsvSinkTable")

# compute a result Table using Table API operators and/or SQL queries
result = ...

# emit the result Table to the registered TableSink
result.insert_into("CsvSinkTable")

# execute the program
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

转换后，将像常规DataStream或DataSet程序一样处理Table API或SQL查询，当`StreamExecutionEnvironment.execute()`或者`ExecutionEnvironment.execute()`被执行时调用。

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

#### **将Table转换为DataStream**

Table结果由流查询动态更新。即，随着新记录到达查询的输入流，它会发生变化。因此，将这样的动态查询转换成的DataStream需要对表的更新进行编码。

将**Table**转换为**DataStream**有两种模式:

1. **追加模式**：仅在动态`Table`仅通过`INSERT`更改进行修改的情况下才能使用此模式，即仅追加并且以前发出的结果从不更新。
2. **缩回模式**：始终可以使用此模式。它使用标志进行编码`INSERT`和`DELETE`更改`boolean`。

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

Table API提供了一种机制来解释计算表的逻辑和优化查询计划。这是通过TableEnvironment.explain\(table\)方法完成的。 `explain(table)`返回给定计划`Table`。`explain()`返回多个接收器计划的结果，主要用于Blink Planner。它返回一个字符串描述三个计划:

1. 关系查询的抽象语法树，即未优化的逻辑查询计划
2. 优化后的逻辑查询计划
3. 物理执行计划

 以下代码展示了一个示例以及`Table`使用给出的相应输出`explain(table)`：

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

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

table1 = t_env.from_elements([(1, "hello")], ["count", "word"])
table2 = t_env.from_elements([(1, "hello")], ["count", "word"])
table = table1 \
    .where("LIKE(word, 'F%')") \
    .union_all(table2)

explanation = t_env.explain(table)
print(explanation)
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

 以下代码显示了一个示例以及使用的多Sink计划的相应输出`explain()`：

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner().inStreamingMode().build();
TableEnvironment tEnv = TableEnvironment.create(settings);

final Schema schema = new Schema()
    .field("count", DataTypes.INT())
    .field("word", DataTypes.STRING());

tEnv.connect(new FileSystem("/source/path1"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySource1");
tEnv.connect(new FileSystem("/source/path2"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySource2");
tEnv.connect(new FileSystem("/sink/path1"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySink1");
tEnv.connect(new FileSystem("/sink/path2"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySink2");

Table table1 = tEnv.from("MySource1").where("LIKE(word, 'F%')");
table1.insertInto("MySink1");

Table table2 = table1.unionAll(tEnv.from("MySource2"));
table2.insertInto("MySink2");

String explanation = tEnv.explain(false);
System.out.println(explanation);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance.useBlinkPlanner.inStreamingMode.build
val tEnv = TableEnvironment.create(settings)

val schema = new Schema()
    .field("count", DataTypes.INT())
    .field("word", DataTypes.STRING())

tEnv.connect(new FileSystem("/source/path1"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySource1")
tEnv.connect(new FileSystem("/source/path2"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySource2")
tEnv.connect(new FileSystem("/sink/path1"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySink1")
tEnv.connect(new FileSystem("/sink/path2"))
    .withFormat(new Csv().deriveSchema())
    .withSchema(schema)
    .createTemporaryTable("MySink2")

val table1 = tEnv.from("MySource1").where("LIKE(word, 'F%')")
table1.insertInto("MySink1")

val table2 = table1.unionAll(tEnv.from("MySource2"))
table2.insertInto("MySink2")

val explanation = tEnv.explain(false)
println(explanation)
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.new_instance().use_blink_planner().in_streaming_mode().build()
t_env = TableEnvironment.create(environment_settings=settings)

schema = Schema()
    .field("count", DataTypes.INT())
    .field("word", DataTypes.STRING())

t_env.connect(FileSystem().path("/source/path1")))
    .with_format(Csv().deriveSchema())
    .with_schema(schema)
    .create_temporary_table("MySource1")
t_env.connect(FileSystem().path("/source/path2")))
    .with_format(Csv().deriveSchema())
    .with_schema(schema)
    .create_temporary_table("MySource2")
t_env.connect(FileSystem().path("/sink/path1")))
    .with_format(Csv().deriveSchema())
    .with_schema(schema)
    .create_temporary_table("MySink1")
t_env.connect(FileSystem().path("/sink/path2")))
    .with_format(Csv().deriveSchema())
    .with_schema(schema)
    .create_temporary_table("MySink2")

table1 = t_env.from_path("MySource1").where("LIKE(word, 'F%')")
table1.insert_into("MySink1")

table2 = table1.union_all(t_env.from_path("MySource2"))
table2.insert_into("MySink2")

explanation = t_env.explain()
print(explanation)
```
{% endtab %}
{% endtabs %}

多Sink计划的结果是

```text
== Abstract Syntax Tree ==
LogicalSink(name=[MySink1], fields=[count, word])
+- LogicalFilter(condition=[LIKE($1, _UTF-16LE'F%')])
   +- LogicalTableScan(table=[[default_catalog, default_database, MySource1, source: [CsvTableSource(read fields: count, word)]]])

LogicalSink(name=[MySink2], fields=[count, word])
+- LogicalUnion(all=[true])
   :- LogicalFilter(condition=[LIKE($1, _UTF-16LE'F%')])
   :  +- LogicalTableScan(table=[[default_catalog, default_database, MySource1, source: [CsvTableSource(read fields: count, word)]]])
   +- LogicalTableScan(table=[[default_catalog, default_database, MySource2, source: [CsvTableSource(read fields: count, word)]]])

== Optimized Logical Plan ==
Calc(select=[count, word], where=[LIKE(word, _UTF-16LE'F%')], reuse_id=[1])
+- TableSourceScan(table=[[default_catalog, default_database, MySource1, source: [CsvTableSource(read fields: count, word)]]], fields=[count, word])

Sink(name=[MySink1], fields=[count, word])
+- Reused(reference_id=[1])

Sink(name=[MySink2], fields=[count, word])
+- Union(all=[true], union=[count, word])
   :- Reused(reference_id=[1])
   +- TableSourceScan(table=[[default_catalog, default_database, MySource2, source: [CsvTableSource(read fields: count, word)]]], fields=[count, word])

== Physical Execution Plan ==
Stage 1 : Data Source
	content : collect elements with CollectionInputFormat

	Stage 2 : Operator
		content : CsvTableSource(read fields: count, word)
		ship_strategy : REBALANCE

		Stage 3 : Operator
			content : SourceConversion(table:Buffer(default_catalog, default_database, MySource1, source: [CsvTableSource(read fields: count, word)]), fields:(count, word))
			ship_strategy : FORWARD

			Stage 4 : Operator
				content : Calc(where: (word LIKE _UTF-16LE'F%'), select: (count, word))
				ship_strategy : FORWARD

				Stage 5 : Operator
					content : SinkConversionToRow
					ship_strategy : FORWARD

					Stage 6 : Operator
						content : Map
						ship_strategy : FORWARD

Stage 8 : Data Source
	content : collect elements with CollectionInputFormat

	Stage 9 : Operator
		content : CsvTableSource(read fields: count, word)
		ship_strategy : REBALANCE

		Stage 10 : Operator
			content : SourceConversion(table:Buffer(default_catalog, default_database, MySource2, source: [CsvTableSource(read fields: count, word)]), fields:(count, word))
			ship_strategy : FORWARD

			Stage 12 : Operator
				content : SinkConversionToRow
				ship_strategy : FORWARD

				Stage 13 : Operator
					content : Map
					ship_strategy : FORWARD

					Stage 7 : Data Sink
						content : Sink: CsvTableSink(count, word)
						ship_strategy : FORWARD

						Stage 14 : Data Sink
							content : Sink: CsvTableSink(count, word)
							ship_strategy : FORWARD
```

