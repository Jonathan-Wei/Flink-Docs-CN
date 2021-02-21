# DESCRIBE语句

DESCRIBE语句用于描述表或视图的结构。

## 运行DESCRIBE语句

description语句可以通过TableEnvironment的executeSql\(\)方法执行。executeSql\(\)方法为成功的描述操作返回给定表的模式，否则将抛出异常。

下面的示例演示如何在TableEnvironment中运行description语句。

{% tabs %}
{% tab title=" Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tableEnv = TableEnvironment.create(settings);

// register a table named "Orders"
tableEnv.executeSql(
        "CREATE TABLE Orders (" +
        " `user` BIGINT NOT NULl," +
        " product VARCHAR(32)," +
        " amount INT," +
        " ts TIMESTAMP(3)," +
        " ptime AS PROCTIME()," +
        " PRIMARY KEY(`user`) NOT ENFORCED," +
        " WATERMARK FOR ts AS ts - INTERVAL '1' SECONDS" +
        ") with (...)");

// print the schema
tableEnv.executeSql("DESCRIBE Orders").print();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance()...
val tableEnv = TableEnvironment.create(settings)

// register a table named "Orders"
 tableEnv.executeSql(
        "CREATE TABLE Orders (" +
        " `user` BIGINT NOT NULl," +
        " product VARCHAR(32)," +
        " amount INT," +
        " ts TIMESTAMP(3)," +
        " ptime AS PROCTIME()," +
        " PRIMARY KEY(`user`) NOT ENFORCED," +
        " WATERMARK FOR ts AS ts - INTERVAL '1' SECONDS" +
        ") with (...)")

// print the schema
tableEnv.executeSql("DESCRIBE Orders").print()
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.new_instance()...
table_env = StreamTableEnvironment.create(env, settings)

# register a table named "Orders"
table_env.execute_sql( \
        "CREATE TABLE Orders (" 
        " `user` BIGINT NOT NULl," 
        " product VARCHAR(32),"
        " amount INT,"
        " ts TIMESTAMP(3),"
        " ptime AS PROCTIME(),"
        " PRIMARY KEY(`user`) NOT ENFORCED,"
        " WATERMARK FOR ts AS ts - INTERVAL '1' SECONDS"
        ") with (...)");

# print the schema
table_env.execute_sql("DESCRIBE Orders").print()
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE TABLE Orders (
>  `user` BIGINT NOT NULl,
>  product VARCHAR(32),
>  amount INT,
>  ts TIMESTAMP(3),
>  ptime AS PROCTIME(),
>  PRIMARY KEY(`user`) NOT ENFORCED,
>  WATERMARK FOR ts AS ts - INTERVAL '1' SECONDS
> ) with (
>  ...
> );
[INFO] Table has been created.

Flink SQL> DESCRIBE Orders;
```
{% endtab %}
{% endtabs %}

上面示例的结果是：

{% tabs %}
{% tab title="Java" %}
```text
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
|    name |                             type |  null |       key | computed column |                  watermark |
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
|    user |                           BIGINT | false | PRI(user) |                 |                            |
| product |                      VARCHAR(32) |  true |           |                 |                            |
|  amount |                              INT |  true |           |                 |                            |
|      ts |           TIMESTAMP(3) *ROWTIME* |  true |           |                 | `ts` - INTERVAL '1' SECOND |
|   ptime | TIMESTAMP(3) NOT NULL *PROCTIME* | false |           |      PROCTIME() |                            |
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
5 rows in set
```
{% endtab %}

{% tab title="Scala" %}
```text
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
|    name |                             type |  null |       key | computed column |                  watermark |
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
|    user |                           BIGINT | false | PRI(user) |                 |                            |
| product |                      VARCHAR(32) |  true |           |                 |                            |
|  amount |                              INT |  true |           |                 |                            |
|      ts |           TIMESTAMP(3) *ROWTIME* |  true |           |                 | `ts` - INTERVAL '1' SECOND |
|   ptime | TIMESTAMP(3) NOT NULL *PROCTIME* | false |           |      PROCTIME() |                            |
+---------+----------------------------------+-------+-----------+-----------------+----------------------------+
5 rows in set
```
{% endtab %}

{% tab title="Python" %}
    +---------+----------------------------------+-------+-----------+-----------------+----------------------------+
    |    name |                             type |  null |       key | computed column |                  watermark |
    +---------+----------------------------------+-------+-----------+-----------------+----------------------------+
    |    user |                           BIGINT | false | PRI(user) |                 |                            |
    | product |                      VARCHAR(32) |  true |           |                 |                            |
    |  amount |                              INT |  true |           |                 |                            |
    |      ts |           TIMESTAMP(3) *ROWTIME* |  true |           |                 | `ts` - INTERVAL '1' SECOND |
    |   ptime | TIMESTAMP(3) NOT NULL *PROCTIME* | false |           |      PROCTIME() |                            |
    +---------+----------------------------------+-------+-----------+-----------------+----------------------------+
    5 rows in set
{% endtab %}

{% tab title="" %}
    root
     |-- user: BIGINT NOT NULL
     |-- product: VARCHAR(32)
     |-- amount: INT
     |-- ts: TIMESTAMP(3) *ROWTIME*
     |-- ptime: TIMESTAMP(3) NOT NULL *PROCTIME* AS PROCTIME()
     |-- WATERMARK FOR ts AS `ts` - INTERVAL '1' SECOND
     |-- CONSTRAINT PK_3599338 PRIMARY KEY (user)
{% endtab %}
{% endtabs %}

## 语法

```sql
DESCRIBE [catalog_name.][db_name.]table_name
```

