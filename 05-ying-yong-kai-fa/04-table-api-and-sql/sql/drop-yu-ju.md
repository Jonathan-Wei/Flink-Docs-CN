# DROP语句

DROP语句用于从当前[目录](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)或指定[目录中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/catalogs.html)删除已注册的表/视图/函数。

Flink SQL现在支持以下DROP语句：

* DROP TABLE
* DROP DATABASE
* DROP FUNCTION

## 运行DROP语句

DROP语句可以使用的`sqlUpdate()`方法执行，也可以`TableEnvironment`在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sqlClient.html)执行。`sqlUpdate()`对于成功的DROP操作，该方法不返回任何内容，否则将引发异常。

以下示例显示如何`TableEnvironment`和SQL CLI中运行DROP语句。

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tableEnv = TableEnvironment.create(settings);

// register a table named "Orders"
tableEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)");

// a string array: ["Orders"]
String[] tables = tableEnv.listTable();

// drop "Orders" table from catalog
tableEnv.sqlUpdate("DROP TABLE Orders");

// an empty string array
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

// drop "Orders" table from catalog
tableEnv.sqlUpdate("DROP TABLE Orders")

// an empty string array
val tables = tableEnv.listTable()
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.newInstance()...
table_env = TableEnvironment.create(settings)

# a string array: ["Orders"]
tables = tableEnv.listTable()

# drop "Orders" table from catalog
tableEnv.sqlUpdate("DROP TABLE Orders")

# an empty string array
tables = tableEnv.listTable()
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> SHOW TABLES;
Orders

Flink SQL> DROP TABLE Orders;
[INFO] Table has been removed.

Flink SQL> SHOW TABLES;
[INFO] Result was empty.
```
{% endtab %}
{% endtabs %}

## DROP TABLE

```text
DROP TABLE [IF EXISTS] [catalog_name.][db_name.]table_name
```

删除具有给定表名的表。如果要删除的表不存在，则会引发异常。

 **IF EXISTS**

如果该表不存在，则什么也不会发生。

## DROP DATABASE

```text
DROP DATABASE [IF EXISTS] [catalog_name.]db_name [ (RESTRICT | CASCADE) ]
```

删除具有给定数据库名称的数据库。如果要删除的数据库不存在，则会引发异常。

 **IF EXISTS**

如果数据库不存在，则什么都不会发生。

 **RESTRICT**

删除非空数据库将触发异常。默认启用。

**CASCADE**

删除非空数据库也会删除所有关联的表和函数。

## DROP FUNCTION

```text
DROP [TEMPORARY|TEMPORARY SYSTEM] FUNCTION [IF EXISTS] [catalog_name.][db_name.]function_name;
```

删除具有目录和数据库名称空间的目录函数。如果要删除的函数不存在，则抛出异常。

 **TEMPORARY**

删除具有目录和数据库名称空间的临时目录功能。

 **TEMPORARY SYSTEM**

删除没有名称空间的临时系统功能。

 **IF EXISTS**

如果函数不存在，则什么也不会发生

