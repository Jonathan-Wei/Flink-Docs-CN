# SHOW语句

SHOW语句用于列出所有catalog、或当前catalog中的所有数据库，或当前catalog和当前数据库的所有表/视图，或显示当前目录和数据库，或列出所有函数，包括临时系统函数，系统当前catalog和当前数据库中的函数，临时catalog函数以及catalog函数。

Flink SQL现在支持以下SHOW语句：

* SHOW CATALOGS
* SHOW CURRENT CATALOG
* SHOW DATABASES
* SHOW CURRENT DATABASE
* SHOW TABLES
* SHOW VIEWS
* SHOW FUNCTIONS

## 运行一个SHOW语句

可以使用`TableEnvironment`的`executeSql()`方法执行`SHOW`语句。`executeSql()`方法为成功的SHOW操作返回对象，否则将抛出异常。

以下示例演示如何在`TableEnvironment`中运行`show`语句。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

// show catalogs
tEnv.executeSql("SHOW CATALOGS").print();
// +-----------------+
// |    catalog name |
// +-----------------+
// | default_catalog |
// +-----------------+

// show current catalog
tEnv.executeSql("SHOW CURRENT CATALOG").print();
// +----------------------+
// | current catalog name |
// +----------------------+
// |      default_catalog |
// +----------------------+

// show databases
tEnv.executeSql("SHOW DATABASES").print();
// +------------------+
// |    database name |
// +------------------+
// | default_database |
// +------------------+

// show current database
tEnv.executeSql("SHOW CURRENT DATABASE").print();
// +-----------------------+
// | current database name |
// +-----------------------+
// |      default_database |
// +-----------------------+

// create a table
tEnv.executeSql("CREATE TABLE my_table (...) WITH (...)");
// show tables
tEnv.executeSql("SHOW TABLES").print();
// +------------+
// | table name |
// +------------+
// |   my_table |
// +------------+

// create a view
tEnv.executeSql("CREATE VIEW my_view AS ...");
// show views
tEnv.executeSql("SHOW VIEWS").print();
// +-----------+
// | view name |
// +-----------+
// |   my_view |
// +-----------+

// show functions
tEnv.executeSql("SHOW FUNCTIONS").print();
// +---------------+
// | function name |
// +---------------+
// |           mod |
// |        sha256 |
// |           ... |
// +---------------+
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
val tEnv = StreamTableEnvironment.create(env)

// show catalogs
tEnv.executeSql("SHOW CATALOGS").print()
// +-----------------+
// |    catalog name |
// +-----------------+
// | default_catalog |
// +-----------------+

// show databases
tEnv.executeSql("SHOW DATABASES").print()
// +------------------+
// |    database name |
// +------------------+
// | default_database |
// +------------------+

// create a table
tEnv.executeSql("CREATE TABLE my_table (...) WITH (...)")
// show tables
tEnv.executeSql("SHOW TABLES").print()
// +------------+
// | table name |
// +------------+
// |   my_table |
// +------------+

// create a view
tEnv.executeSql("CREATE VIEW my_view AS ...")
// show views
tEnv.executeSql("SHOW VIEWS").print()
// +-----------+
// | view name |
// +-----------+
// |   my_view |
// +-----------+

// show functions
tEnv.executeSql("SHOW FUNCTIONS").print()
// +---------------+
// | function name |
// +---------------+
// |           mod |
// |        sha256 |
// |           ... |
// +---------------+
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.new_instance()...
table_env = StreamTableEnvironment.create(env, settings)

# show catalogs
table_env.execute_sql("SHOW CATALOGS").print()
# +-----------------+
# |    catalog name |
# +-----------------+
# | default_catalog |
# +-----------------+

# show databases
table_env.execute_sql("SHOW DATABASES").print()
# +------------------+
# |    database name |
# +------------------+
# | default_database |
# +------------------+

# create a table
table_env.execute_sql("CREATE TABLE my_table (...) WITH (...)")
# show tables
table_env.execute_sql("SHOW TABLES").print()
# +------------+
# | table name |
# +------------+
# |   my_table |
# +------------+

# create a view
table_env.execute_sql("CREATE VIEW my_view AS ...")
# show views
table_env.execute_sql("SHOW VIEWS").print()
# +-----------+
# | view name |
# +-----------+
# |   my_view |
# +-----------+

# show functions
table_env.execute_sql("SHOW FUNCTIONS").print()
# +---------------+
# | function name |
# +---------------+
# |           mod |
# |        sha256 |
# |           ... |
# +---------------+
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> SHOW CATALOGS;
default_catalog

Flink SQL> SHOW DATABASES;
default_database

Flink SQL> CREATE TABLE my_table (...) WITH (...);
[INFO] Table has been created.

Flink SQL> SHOW TABLES;
my_table

Flink SQL> CREATE VIEW my_view AS ...;
[INFO] View has been created.

Flink SQL> SHOW VIEWS;
my_view

Flink SQL> SHOW FUNCTIONS;
mod
sha256
...
```
{% endtab %}
{% endtabs %}

## 显示Catalog

```text
SHOW CATALOGS
```

显示所有Catalog。

## 显示当前Catalog

```text
SHOW CURRENT CATALOG
```

显示当前Catalog。

### 显示数据库 <a id="show-databases"></a>

```text
SHOW DATABASES
```

显示当前Catalog中的所有数据库。

### 显示当前数据库 <a id="show-current-database"></a>

```text
SHOW CURRENT DATABASE
```

显示当前数据库。

### 显示表 <a id="show-tables"></a>

```text
SHOW TABLES
```

显示当前Catalog和当前数据库中的所有表。

### 显示视图 <a id="show-views"></a>

```text
SHOW VIEWS
```

显示当前Catalog和当前数据库中的所有视图。

### 显示函数 <a id="show-functions"></a>

```text
SHOW FUNCTIONS
```

在当前Catalog和当前数据库中显示所有函数，包括临时系统函数，系统函数，临时Catalog函数和Catalog函数。  


