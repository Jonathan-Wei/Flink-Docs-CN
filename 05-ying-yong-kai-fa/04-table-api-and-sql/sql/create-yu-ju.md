# CREATE语句

CREATE语句用于将表/视图/函数注册到当前[目录](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)或指定[目录中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)。注册的表/视图/函数可以在SQL查询中使用。

Flink SQL现在支持以下CREATE语句：

* CREATE TABLE
* CREATE DATABASE
* CREATE FUNCTION

## 运行CREATE语句

CREATE语句可以使用`TableEnvironment`的`sqlUpdate()`方法执行，也可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sqlClient.html)执行。`sqlUpdate()`对于成功的CREATE操作，该方法不返回任何内容，否则将引发异常。

以下示例显示了如何在`TableEnvironment`和SQL CLI 中运行CREATE语句。

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tableEnv = TableEnvironment.create(settings);

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
Table result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance()...
val tableEnv = TableEnvironment.create(settings)

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
val result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH ('connector.path'='/path/to/file' ...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.newInstance()...
table_env = TableEnvironment.create(settings)

# SQL query with a registered table
# register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");
# run a SQL query on the Table and retrieve the result as a new Table
result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

# SQL update with a registered table
# register a TableSink
table_env.sql_update("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)")
# run a SQL update query on the Table and emit the result to the TableSink
table_env \
    .sql_update("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> CREATE TABLE RubberOrders (product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%';
[INFO] Submitting SQL update statement to the cluster...
```
{% endtab %}
{% endtabs %}

## CREATE TABLE

```sql
CREATE TABLE [catalog_name.][db_name.]table_name
  (
    { <column_definition> | <computed_column_definition> }[ , ...n]
    [ <watermark_definition> ]
  )
  [COMMENT table_comment]
  [PARTITIONED BY (partition_column_name1, partition_column_name2, ...)]
  WITH (key1=val1, key2=val2, ...)

<column_definition>:
  column_name column_type [COMMENT column_comment]

<computed_column_definition>:
  column_name AS computed_column_expression [COMMENT column_comment]

<watermark_definition>:
  WATERMARK FOR rowtime_column_name AS watermark_strategy_expression
```

用给定名称创建一个表。如果目录中已经存在具有相同名称的表，则会引发异常。

  **计算列\(COMPUTED COLUMN\)**

 计算列是使用语法“ `column_name AS computed_column_expression`” 生成的虚拟列。它由使用同一表中其他列的非查询表达式生成，并且未物理存储在表中。例如，可以将计算列定义为`cost AS price * quantity`。该表达式可以包含物理列，常量，函数或变量的任意组合。表达式不能包含子查询。

在Flink中，计算列通常用于在CREATE TABLE语句中定义时间属性。可以使用系统的PROCTIME\(\)函数通过“proc AS PROCTIME\(\)”轻松定义处理时间属性。另一方面， 计算列可用于派生事件时间列，因为事件时间列可能需要从现有字段派生，例如，原始字段不是TIMESTAMP\(3\)类型或嵌套在JSON字符串中。

{% hint style="info" %}
注意：

* 在源表上定义的计算列是在从源表读取后计算的，可以在以下SELECT查询语句中使用它。
* 计算列不能是INSERT语句的目标。在INSERT语句中，SELECT子句的模式应该与没有计算列的目标表的模式匹配。
{% endhint %}

  **水印\(WATERMARK\)**

水印定义表的事件时间属性，并采用“`WATERMARK FOR rowtime_column_name AS watermark_strategy_expression`”的形式。

`rowtime_column_name`定义了一个已存在的列，它被标记为表的事件时间属性。列的类型必须是`TIMESTAMP(3)`，并且是模式中的顶级列。它可能是一个计算列。

`watermark_strategy_expression`定义了水印生成策略。它允许任意非查询表达式\(包括计算列\)计算水印。表达式返回类型必须是`TIMESTAMP(3)`，它表示从大纪元\(Epoch\)开始的时间戳。

使用事件时间语义时，表必须包含事件时间属性和水印策略。

Flink提供了几种常用的水印策略。

* 严格提升时间戳记：`WATERMARK FOR rowtime_column AS rowtime_column`。

  发出到目前为止已观察到的最大时间戳的水印。时间戳小于最大时间戳的行不会迟到。

* 升序时间戳记：`WATERMARK FOR rowtime_column AS rowtime_column - INTERVAL '0.001' SECOND`。

  发出到目前为止最大观测到的时间戳减负1的水印。时间戳等于或小于最大时间戳的行不会迟到。

* 受限制的时间戳记为限：`WATERMARK FOR rowtime_column AS rowtime_column - INTERVAL 'string' timeUnit`。

  发出水印，它是观察到的最大时间戳减去指定的延迟，例如，`WATERMARK FOR rowtime_column AS rowtime_column - INTERVAL '5' SECOND`是5秒的延迟水印策略。

```sql
CREATE TABLE Orders (
    user BIGINT,
    product STRING,
    order_time TIMESTAMP(3),
    WATERMARK FOR order_time AS order_time - INTERVAL '5' SECOND
) WITH ( . . . );
```

  **分区\(PARTITIONED BY\)**

按指定的列对创建的表进行分区。如果将此表用作文件系统接收器，则会为每个分区创建一个目录。

 **WITH OPTIONS**

用于创建表源/接收器的表属性。这些属性通常用于查找和创建基础连接器。

 expression的键和值`key1=val1`都应为字符串文字。有关不同连接器的所有受支持表属性，请参阅“ [连接到外部系统”中的](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html)详细信息。

**注意：**表名可以采用三种格式：

* `catalog_name.db_name.table_name`
* `db_name.table_name`
* `table_name`

对于`catalog_name.db_name.table_name`，该表将被注册到名为“ `catalog_name`”的目录和名为“ `db_name`”的数据库的元存储库中；  
****对于`db_name.table_name`，该表将被注册到执行表环境和名为“ `db_name`”的数据库的当前目录中；  
对于`table_name`，该表将被注册到执行表环境的当前目录和数据库中。

**注意：**使用`CREATE` 语句注册的表既可以用作表源，也可以用作表接收器，在DML中引用它之前，我们无法确定是将其用作源还是接收器。  


## CREATE DATABASE

```sql
CREATE DATABASE [IF NOT EXISTS] [catalog_name.]db_name
  [COMMENT database_comment]
  WITH (key1=val1, key2=val2, ...)
```

创建具有给定数据库属性的数据库。如果目录中已经存在具有相同名称的数据库，则会引发异常。

 **IF NOT EXISTS**

如果数据库已经存在，则什么都不会发生。

 **WITH OPTIONS**

数据库属性，用于存储与此数据库有关的额外信息。expression的键和值`key1=val1`都应为字符串文字。

## CREATE FUNCTION

```sql
CREATE [TEMPORARY|TEMPORARY SYSTEM] FUNCTION 
  [IF NOT EXISTS] [catalog_name.][db_name.]function_name 
  AS identifier [LANGUAGE JAVA|SCALA]
```

创建一个具有目录和数据库名称空间的目录功能，该名称空间的标识符是JAVA / SCALA的完整类路径和可选的语言标记。如果目录中已经存在相同名称的函数，则将引发异常。

 **TEMPORARY**

创建具有目录和数据库名称空间并覆盖目录功能的临时目录功能。

 **TEMPORARY SYSTEM**

创建没有名称空间的临时系统函数，并覆盖内置函数

 **IF NOT EXISTS**

如果该功能已经存在，则什么也不会发生。

 **LANGUAGE JAVA\|SCALA**

语言标签，用于指示Flink运行时如何执行该功能。当前仅支持JAVA和SCALA，函数的默认语言为JAVA。

