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

## 注册外部目录

## 查询表

## Emit一张表

## 转换并执行查询

## 与DataStream和DataSet API集成

## 查询优化

