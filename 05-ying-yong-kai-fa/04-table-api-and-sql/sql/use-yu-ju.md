# USE语句

USE语句用于设置当前数据库或catalog。

## 运行USE语句

USE语句可以使用TableEnvironment的executeSql\(\)方法执行。对于成功的使用操作，executeSql\(\)方法返回' OK '，否则将抛出异常。

下面的示例演示如何在TableEnvironment中运行USE语句。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
StreamTableEnvironment tEnv = StreamTableEnvironment.create(env);

// create a catalog
tEnv.executeSql("CREATE CATALOG cat1 WITH (...)");
tEnv.executeSql("SHOW CATALOGS").print();
// +-----------------+
// |    catalog name |
// +-----------------+
// | default_catalog |
// | cat1            |
// +-----------------+

// change default catalog
tEnv.executeSql("USE CATALOG cat1");

tEnv.executeSql("SHOW DATABASES").print();
// databases are empty
// +---------------+
// | database name |
// +---------------+
// +---------------+

// create a database
tEnv.executeSql("CREATE DATABASE db1 WITH (...)");
tEnv.executeSql("SHOW DATABASES").print();
// +---------------+
// | database name |
// +---------------+
// |        db1    |
// +---------------+

// change default database
tEnv.executeSql("USE db1");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
val tEnv = StreamTableEnvironment.create(env)

// create a catalog
tEnv.executeSql("CREATE CATALOG cat1 WITH (...)")
tEnv.executeSql("SHOW CATALOGS").print()
// +-----------------+
// |    catalog name |
// +-----------------+
// | default_catalog |
// | cat1            |
// +-----------------+

// change default catalog
tEnv.executeSql("USE CATALOG cat1")

tEnv.executeSql("SHOW DATABASES").print()
// databases are empty
// +---------------+
// | database name |
// +---------------+
// +---------------+

// create a database
tEnv.executeSql("CREATE DATABASE db1 WITH (...)")
tEnv.executeSql("SHOW DATABASES").print()
// +---------------+
// | database name |
// +---------------+
// |        db1    |
// +---------------+

// change default database
tEnv.executeSql("USE db1")
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.new_instance()...
table_env = StreamTableEnvironment.create(env, settings)

# create a catalog
table_env.execute_sql("CREATE CATALOG cat1 WITH (...)")
table_env.execute_sql("SHOW CATALOGS").print()
# +-----------------+
# |    catalog name |
# +-----------------+
# | default_catalog |
# | cat1            |
# +-----------------+

# change default catalog
table_env.execute_sql("USE CATALOG cat1")

table_env.execute_sql("SHOW DATABASES").print()
# databases are empty
# +---------------+
# | database name |
# +---------------+
# +---------------+

# create a database
table_env.execute_sql("CREATE DATABASE db1 WITH (...)")
table_env.execute_sql("SHOW DATABASES").print()
# +---------------+
# | database name |
# +---------------+
# |           db1 |
# +---------------+

# change default database
table_env.execute_sql("USE db1")
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE CATALOG cat1 WITH (...);
[INFO] Catalog has been created.

Flink SQL> SHOW CATALOGS;
default_catalog
cat1

Flink SQL> USE CATALOG cat1;

Flink SQL> SHOW DATABASES;

Flink SQL> CREATE DATABASE db1 WITH (...);
[INFO] Database has been created.

Flink SQL> SHOW DATABASES;
db1

Flink SQL> USE db1;SQ
```
{% endtab %}
{% endtabs %}

## USE CATLOAG

```text
USE CATALOG catalog_name
```

设置当前目录。所有未明确指定目录的后续命令都将使用此命令。如果提供的目录不存在，则会引发异常。当前的默认目录为`default_catalog`。

## USE

```text
USE [catalog_name.]database_name
```

设置当前数据库。所有未明确指定数据库的后续命令都将使用此命令。如果提供的数据库不存在，则会引发异常。当前的默认数据库是`default_database`。

