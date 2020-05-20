# ALTER语句

 ALTER语句用于修改[Catalog中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)已注册的表/视图/函数定义。

Flink SQL现在支持以下ALTER语句：

* ALTER TABLE
* ALTER DATABASE
* ALTER FUNCTION

## 运行一条ALTER语句

ALTER语句可以使用`TableEnvironment`的`sqlUpdate()`方法执行，也可以在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sqlClient.html)执行。`sqlUpdate()`对于成功的ALTER操作，该方法不返回任何内容，否则将引发异常。

下面的示例演示如何在TableEnvironment和SQL CLI中运行ALTER语句。

{% tabs %}
{% tab title="Java" %}
```javascript
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tableEnv = TableEnvironment.create(settings);

// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");

// a string array: ["Orders"]
String[] tables = tableEnv.listTable();

// rename "Orders" to "NewOrders"
tableEnv.sqlUpdate("ALTER TABLE Orders RENAME TO NewOrders;");

// a string array: ["NewOrders"]
String[] tables = tableEnv.listTable();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance()...
val tableEnv = TableEnvironment.create(settings)

// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");

// a string array: ["Orders"]
val tables = tableEnv.listTable()

// rename "Orders" to "NewOrders"
tableEnv.sqlUpdate("ALTER TABLE Orders RENAME TO NewOrders;")

// a string array: ["NewOrders"]
val tables = tableEnv.listTable()
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.newInstance()...
table_env = TableEnvironment.create(settings)

# a string array: ["Orders"]
tables = tableEnv.listTable()

# rename "Orders" to "NewOrders"
tableEnv.sqlUpdate("ALTER TABLE Orders RENAME TO NewOrders;")

# a string array: ["NewOrders"]
tables = tableEnv.listTable()
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> SHOW TABLES;
Orders

Flink SQL> ALTER TABLE Orders RENAME TO NewOrders;
[INFO] Table has been removed.

Flink SQL> SHOW TABLES;
NewOrders
```
{% endtab %}
{% endtabs %}

## ALTER TABLE

* 重命名表

```text
ALTER TABLE [catalog_name.][db_name.]table_name RENAME TO new_table_name
```

将给定的表名重命名为另一个新的表名。

* 设置或更改表属性

```text
ALTER TABLE [catalog_name.][db_name.]table_name SET (key1=val1, key2=val2, ...)
```

在指定的表中设置一个或多个属性。如果表中已经设置了特定属性，请用新属性覆盖旧值。

## ALTER DATABASE

```text
ALTER DATABASE [catalog_name.]db_name SET (key1=val1, key2=val2, ...)
```

在指定的数据库中设置一个或多个属性。如果数据库中已经设置了特定属性，请用新属性覆盖旧值。

## ALTER FUNCTION

```text
ALTER [TEMPORARY|TEMPORARY SYSTEM] FUNCTION 
  [IF EXISTS] [catalog_name.][db_name.]function_name 
  AS identifier [LANGUAGE JAVA|SCALA|
```

使用新的标识符\(JAVA/SCALA的完整类路径和可选语言标记\)修改一个目录函数。如果目录中不存在函数，则抛出异常。

 **TEMPORARY**

更改具有目录和数据库名称空间的临时目录功能，并覆盖目录功能。

**TEMPORARY SYSTEM**

更改没有名称空间的临时系统功能，并覆盖内置功能

 **IF EXISTS**

如果该功能不存在，则什么也不会发生。

 **LANGUAGE JAVA\|SCALA**

语言标签，用于指示flink运行时如何执行该功能。当前仅支持JAVA和SCALA，函数的默认语言为JAVA。

