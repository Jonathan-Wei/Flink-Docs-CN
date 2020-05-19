# SQL

## DDL



{% hint style="info" %}
**注意：** Flink的DDL支持尚未完成。包含不受支持的SQL功能的查询会导致`TableException`。以下各节列出了批处理和流表上SQL DDL支持的功能。
{% endhint %}

### 指定DDL

以下示例显示如何指定SQL DDL。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
Table result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH (...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
val tableEnv = StreamTableEnvironment.create(env)

// SQL query with a registered table
// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
// run a SQL query on the Table and retrieve the result as a new Table
val result = tableEnv.sqlQuery(
  "SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");

// SQL update with a registered table
// register a TableSink
tableEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH ('connector.path'='/path/to/file' ...)");
// run a SQL update query on the Table and emit the result to the TableSink
tableEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
table_env = StreamTableEnvironment.create(env)

# SQL update with a registered table
# register a TableSink
table_env.sql_update("CREATE TABLE RubberOrders(product VARCHAR, amount INT) with (...)")
# run a SQL update query on the Table and emit the result to the TableSink
table_env \
    .sql_update("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}
{% endtabs %}

**PARTITIONED BY**

按指定的列对创建的表进行分区。如果此表用作文件系统接收器，则为每个分区创建一个目录。

**WITH OPTIONS**

用于创建表源/接收器的表属性。这些属性通常用于查找和创建基础连接器。

### 创建表

```sql
CREATE TABLE [catalog_name.][db_name.]table_name
  [(col_name1 col_type1 [COMMENT col_comment1], ...)]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name1, col_name2, ...)]
  WITH (key1=val1, key2=val2, ...)
```

### 删除表

```sql
DROP TABLE [IF EXISTS] [catalog_name.][db_name.]table_name
```

删除具有给定表名的表。如果要删除的表不存在，则抛出异常。

**如果存在**

如果表不存在，则不会发生任何事情。

## 数据类型

SQL运行时构建在Flink的DataSet和DataStream API之上。在内部，它还使用Flink的类型信息来定义数据类型。所有支持的类型列在`org.apache.flink.table.api.Types`中。下表总结了SQL类型、表API类型和生成的Java类之间的关系。

| Table API | SQL | Java类型 |
| :--- | :--- | :--- |
| Types.STRING | VARCHAR | java.lang.String |
| Types.BOOLEAN | BOOLEAN | java.lang.Boolean |
| Types.BYTE | TINYINT | java.lang.Byte |
| Types.SHORT | SMALLINT | java.lang.Short |
| Types.INT | INTEGER, INT | java.lang.Integer |
| Types.LONG | BIGINT | java.lang.Long |
| Types.FLOAT | REAL, FLOAT | java.lang.Float |
| Types.DOUBLE | DOUBLE | java.lang.Double |
| Types.DECIMAL | DECIMAL | java.math.BigDecimal |
| Types.SQL\_DATE | DATE | java.sql.Date |
| Types.SQL\_TIME | TIME | java.sql.Time |
| Types.SQL\_TIMESTAMP | TIMESTAMP\(3\) | java.sql.Timestamp |
| Types.INTERVAL\_MONTHS | INTERVAL YEAR TO MONTH | java.lang.Integer |
| Types.INTERVAL\_MILLIS | INTERVAL DAY TO SECOND\(3\) | java.lang.Long |
| Types.PRIMITIVE\_ARRAY | ARRAY | 例如 `int[]` |
| Types.OBJECT\_ARRAY | ARRAY | 例如 `java.lang.Byte[]` |
| Types.MAP | MAP | java.util.HashMap |
| Types.MULTISET | MULTISET | 例如，`java.util.HashMap<String, Integer>`表示`String`的多重集合 |
| Types.ROW | ROW | org.apache.flink.types.Row |

泛型类型和\(嵌套的\)复合类型\(例如：POJO、Tuple、Row、Scala Case Class\)也可以是行中的字段。

可以使用[值访问函数](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/functions.html#value-access-functions)访问具有任意嵌套的复合类型的字段。

泛型类型被视为黑盒，可以由[用户定义的函数](https://ci.apache.org/projects/flink/flink-docs-master/dev/table/udfs.html)传递或处理。





