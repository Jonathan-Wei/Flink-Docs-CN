# Catalogs

## Catalog类型

Catalog提供元数据，如数据库、表、分区、视图以及访问存储在数据库或其他外部系统中的数据所需的函数和信息。

数据处理最关键的方面之一是管理元数据。它可以是临时表之类的临时元数据，也可以是在表环境中注册的udf。或者永久的元数据，比如在Hive Metastore中。Catalog提供了一个统一的API来管理元数据，并使其可以通过表API和SQL查询访问。

### GenericInMemory Catalog

GenericInMemoryCatalog是一个目录在内存中的实现。所有对象只在会话的生命周期内可用。

### Hive Catalog

HiveCatalog有两个目的:作为纯Flink元数据的持久存储，以及作为读取和写入现有Hive元数据的接口。Flink的 [Hive文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/index.html)提供了设置目录和与现有Hive安装接口的完整细节。

{% hint style="danger" %}
注意： Hive Metastore以小写形式存储所有元对象名称。这与`GenericInMemoryCatalog`区分大小写不同
{% endhint %}

### 用户自定义Catalog

Catalog是可插入的，用户可以通过实现Catalog接口开发自定义目录。要在SQL CLI中使用自定义目录，用户应该通过实现CatalogFactory接口来开发Catalog及其相应的 `CatalogFactory` 。

 `CatalogFactory` 定义了一组属性，用于在SQL CLI引导时配置Catalog。这组属性将被传递给发现服务，该服务将尝试将这些属性与CatalogFactory匹配，并启动相应的catalog实例。

## 如何创建Flink表并将其注册到Catalog

### 使用SQL DDL

用户可以使用SQL DDL通过Table API和SQL在Catalog中创建表。

{% tabs %}
{% tab title="Table API \(Java\)" %}
```java
TableEnvironment tableEnv = ...

// Create a HiveCatalog 
Catalog catalog = new HiveCatalog("myhive", null, "<path_of_hive_conf>", "<hive_version>");

// Register the catalog
tableEnv.registerCatalog("myhive", catalog);

// Create a catalog database
tableEnv.sqlUpdate("CREATE DATABASE mydb WITH (...)");

// Create a catalog table
tableEnv.sqlUpdate("CREATE TABLE mytable (name STRING, age INT) WITH (...)");

tableEnv.listTables(); // should return the tables in current catalog and database.
```
{% endtab %}

{% tab title="SQL CLI\(Java\)" %}
```sql
// the catalog should have been registered via yaml file
Flink SQL> CREATE DATABASE mydb WITH (...);

Flink SQL> CREATE TABLE mytable (name STRING, age INT) WITH (...);

Flink SQL> SHOW TABLES;
mytable
```
{% endtab %}
{% endtabs %}

 有关详细信息，请查看[Flink SQL CREATE DDL](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sql/create.html)。

### 使用Java / Scala / Python API

用户可以使用Java，Scala或Python API以编程方式创建Catalog表。

{% tabs %}
{% tab title="Java" %}
```java
TableEnvironment tableEnv = ...

// Create a HiveCatalog 
Catalog catalog = new HiveCatalog("myhive", null, "<path_of_hive_conf>", "<hive_version>");

// Register the catalog
tableEnv.registerCatalog("myhive", catalog);

// Create a catalog database 
catalog.createDatabase("mydb", new CatalogDatabaseImpl(...))

// Create a catalog table
TableSchema schema = TableSchema.builder()
    .field("name", DataTypes.STRING())
    .field("age", DataTypes.INT())
    .build();

catalog.createTable(
        new ObjectPath("mydb", "mytable"), 
        new CatalogTableImpl(
            schema,
            new Kafka()
                .version("0.11")
                ....
                .startFromEarlist(),
            "my comment"
        )
    );
    
List<String> tables = catalog.listTables("mydb"); // tables should contain "mytable"
```
{% endtab %}
{% endtabs %}

## Catalog API

 注意：此处仅列出Catalog程序API。用户可以使用SQL DDL实现许多相同的功能。有关详细的DDL信息，请参阅[SQL CREATE DDL](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sql/create.html)。

### 数据库操作

{% tabs %}
{% tab title="Java" %}
```java
// create database
catalog.createDatabase("mydb", new CatalogDatabaseImpl(...), false);

// drop database
catalog.dropDatabase("mydb", false);

// alter database
catalog.alterDatabase("mydb", new CatalogDatabaseImpl(...), false);

// get databse
catalog.getDatabase("mydb");

// check if a database exist
catalog.databaseExists("mydb");

// list databases in a catalog
catalog.listDatabases("mycatalog");
```
{% endtab %}
{% endtabs %}

### 表操作

{% tabs %}
{% tab title="Java" %}
```java
// create table
catalog.createTable(new ObjectPath("mydb", "mytable"), new CatalogTableImpl(...), false);

// drop table
catalog.dropTable(new ObjectPath("mydb", "mytable"), false);

// alter table
catalog.alterTable(new ObjectPath("mydb", "mytable"), new CatalogTableImpl(...), false);

// rename table
catalog.renameTable(new ObjectPath("mydb", "mytable"), "my_new_table");

// get table
catalog.getTable("mytable");

// check if a table exist or not
catalog.tableExists("mytable");

// list tables in a database
catalog.listTables("mydb");
```
{% endtab %}
{% endtabs %}

### 查询操作

{% tabs %}
{% tab title="Java" %}
```java
// create view
catalog.createTable(new ObjectPath("mydb", "myview"), new CatalogViewImpl(...), false);

// drop view
catalog.dropTable(new ObjectPath("mydb", "myview"), false);

// alter view
catalog.alterTable(new ObjectPath("mydb", "mytable"), new CatalogViewImpl(...), false);

// rename view
catalog.renameTable(new ObjectPath("mydb", "myview"), "my_new_view");

// get view
catalog.getTable("myview");

// check if a view exist or not
catalog.tableExists("mytable");

// list views in a database
catalog.listViews("mydb");
```
{% endtab %}
{% endtabs %}

### 分区操作

{% tabs %}
{% tab title="Java" %}
```java
// create view
catalog.createPartition(
    new ObjectPath("mydb", "mytable"),
    new CatalogPartitionSpec(...),
    new CatalogPartitionImpl(...),
    false);

// drop partition
catalog.dropPartition(new ObjectPath("mydb", "mytable"), new CatalogPartitionSpec(...), false);

// alter partition
catalog.alterPartition(
    new ObjectPath("mydb", "mytable"),
    new CatalogPartitionSpec(...),
    new CatalogPartitionImpl(...),
    false);

// get partition
catalog.getPartition(new ObjectPath("mydb", "mytable"), new CatalogPartitionSpec(...));

// check if a partition exist or not
catalog.partitionExists(new ObjectPath("mydb", "mytable"), new CatalogPartitionSpec(...));

// list partitions of a table
catalog.listPartitions(new ObjectPath("mydb", "mytable"));

// list partitions of a table under a give partition spec
catalog.listPartitions(new ObjectPath("mydb", "mytable"), new CatalogPartitionSpec(...));

// list partitions of a table by expression filter
catalog.listPartitions(new ObjectPath("mydb", "mytable"), Arrays.asList(epr1, ...));
```
{% endtab %}
{% endtabs %}

### 函数操作

{% tabs %}
{% tab title="Java" %}
```java
// create function
catalog.createFunction(new ObjectPath("mydb", "myfunc"), new CatalogFunctionImpl(...), false);

// drop function
catalog.dropFunction(new ObjectPath("mydb", "myfunc"), false);

// alter function
catalog.alterFunction(new ObjectPath("mydb", "myfunc"), new CatalogFunctionImpl(...), false);

// get function
catalog.getFunction("myfunc");

// check if a function exist or not
catalog.functionExists("myfunc");

// list functions in a database
catalog.listFunctions("mydb");
```
{% endtab %}
{% endtabs %}

## 用于Catalog的Table API和SQL

### 注册Catalog

用户可以访问一个名为default\_catalog的默认内存Catalog，该Catalog总是在默认情况下创建的。默认情况下，这个目录有一个名为default\_database的数据库。用户还可以将其他Catalog注册到现有的Flink会话中。

{% tabs %}
{% tab title="Java/Scala" %}
```text
tableEnv.registerCatalog(new CustomCatalog("myCatalog"));
```
{% endtab %}

{% tab title="YAML" %}
 使用YAML定义的所有目录必须提供指定目录类型的类型属性。以下类型支持开箱即用。

| Catalog | Type Value |
| :--- | :--- |
| GenericInMemory | generic\_in\_memory |
| Hive | hive |

```text
catalogs:
   - name: myCatalog
     type: custom_catalog
     hive-conf-dir: ...
```
{% endtab %}
{% endtabs %}

### 更改当前Catalog和数据库

Flink将始终在当前Catalog和数据库中搜索表，视图和UDF。

{% tabs %}
{% tab title="Java/Scala" %}
```java
tableEnv.useCatalog("myCatalog");
tableEnv.useDatabase("myDb");
```
{% endtab %}

{% tab title="SQL" %}
```sql
Flink SQL> USE CATALOG myCatalog;
Flink SQL> USE myDB;
```
{% endtab %}
{% endtabs %}

通过在catalog.database.object表单中提供完全限定的名称，可以访问来自非当前编目的编目的元数据。

{% tabs %}
{% tab title="Java/Scala" %}
```java
tableEnv.from("not_the_current_catalog.not_the_current_db.my_table");
```
{% endtab %}

{% tab title="SQL" %}
```sql
Flink SQL> SELECT * FROM not_the_current_catalog.not_the_current_db.my_table;
```
{% endtab %}
{% endtabs %}

### 列出可用的Catalog

{% tabs %}
{% tab title="Java/Scala" %}
```java
tableEnv.listCatalogs();
```
{% endtab %}

{% tab title="SQL" %}
```sql
Flink SQL> show catalogs;
```
{% endtab %}
{% endtabs %}

### 列出可用的数据库

{% tabs %}
{% tab title="Java/Scala" %}
```text
tableEnv.listDatabases();
```
{% endtab %}

{% tab title="SQL" %}
```sql
Flink SQL> show databases;
```
{% endtab %}
{% endtabs %}

### 列出可用表

{% tabs %}
{% tab title="Java/Scala" %}
```text
tableEnv.listTables();
```
{% endtab %}

{% tab title="SQL" %}
```sql
Flink SQL> show tables;
```
{% endtab %}
{% endtabs %}

