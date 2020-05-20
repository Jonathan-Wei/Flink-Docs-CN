# INSERT语句

## 运行INSERT语句

INSERT语句使用`TableEnvironment`的`sqlUpdate()`方法指定或在[SQL CLI中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sqlClient.html)执行。`sqlUpdate()`INSERT语句的方法是惰性执行，只有在`TableEnvironment.execute(jobName)`调用时才会执行。

以下示例显示了如何在`TableEnvironment`和SQL CLI 中和在SQL CLI中运行INSERT语句。

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance()...
TableEnvironment tEnv = TableEnvironment.create(settings);

// register a source table named "Orders" and a sink table named "RubberOrders"
tEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product VARCHAR, amount INT) WITH (...)");
tEnv.sqlUpdate("CREATE TABLE RubberOrders(product VARCHAR, amount INT) WITH (...)");

// run a SQL update query on the registered source table and emit the result to registered sink table
tEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val settings = EnvironmentSettings.newInstance()...
val tEnv = TableEnvironment.create(settings)

// register a source table named "Orders" and a sink table named "RubberOrders"
tEnv.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")
tEnv.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)")

// run a SQL update query on the registered source table and emit the result to registered sink table
tEnv.sqlUpdate(
  "INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="Python" %}
```python
settings = EnvironmentSettings.newInstance()...
table_env = TableEnvironment.create(settings)

# register a source table named "Orders" and a sink table named "RubberOrders"
table_env.sqlUpdate("CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...)")
table_env.sqlUpdate("CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...)")

# run a SQL update query on the registered source table and emit the result to registered sink table
table_env \
    .sqlUpdate("INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%'")
```
{% endtab %}

{% tab title="SQL CLI" %}
```sql
Flink SQL> CREATE TABLE Orders (`user` BIGINT, product STRING, amount INT) WITH (...);
[INFO] Table has been created.

Flink SQL> CREATE TABLE RubberOrders(product STRING, amount INT) WITH (...);

Flink SQL> SHOW TABLES;
Orders
RubberOrders

Flink SQL> INSERT INTO RubberOrders SELECT product, amount FROM Orders WHERE product LIKE '%Rubber%';
[INFO] Submitting SQL update statement to the cluster...
[INFO] Table update statement has been successfully submitted to the cluster:

```
{% endtab %}
{% endtabs %}

## 从Select查询INSERT

可以使用insert子句将查询结果插入表中。

### 语法

```sql
INSERT { INTO | OVERWRITE } [catalog_name.][db_name.]table_name [PARTITION part_spec] select_statement

part_spec:
  (part_col_name1=val1 [, part_col_name2=val2, ...])
```

 **OVERWRITE**

 `INSERT OVERWRITE`将覆盖表或分区中的所有现有数据。否则，将附加新数据。

 **PARTITION**

 `PARTITION` 子句应包含此插入的静态分区列。

#### 例子 <a id="examples"></a>

```sql
-- Creates a partitioned table
CREATE TABLE country_page_view (user STRING, cnt INT, date STRING, country STRING)
PARTITIONED BY (date, country)
WITH (...)

-- Appends rows into the static partition (date='2019-8-30', country='China')
INSERT INTO country_page_view PARTITION (date='2019-8-30', country='China')
  SELECT user, cnt FROM page_view_source;

-- Appends rows into partition (date, country), where date is static partition with value '2019-8-30',
-- country is dynamic partition whose value is dynamic determined by each row.
INSERT INTO country_page_view PARTITION (date='2019-8-30')
  SELECT user, cnt, country FROM page_view_source;

-- Overwrites rows into static partition (date='2019-8-30', country='China')
INSERT OVERWRITE country_page_view PARTITION (date='2019-8-30', country='China')
  SELECT user, cnt FROM page_view_source;

-- Overwrites rows into partition (date, country), where date is static partition with value '2019-8-30',
-- country is dynamic partition whose value is dynamic determined by each row.
INSERT OVERWRITE country_page_view PARTITION (date='2019-8-30')
  SELECT user, cnt, country FROM page_view_source;
```

## 将值插入表格

INSERT…VALUES语句可用于直接从SQL将数据插入表中。

### **语**法

```sql
INSERT { INTO | OVERWRITE } [catalog_name.][db_name.]table_name VALUES values_row [, values_row ...]

values_row:
    : (val1 [, val2, ...])
```

 **OVERWRITE**

`INSERT OVERWRITE`将覆盖表中的所有现有数据。否则，将附加新数据。

#### 例子 <a id="examples-1"></a>

```sql
CREATE TABLE students (name STRING, age INT, gpa DECIMAL(3, 2)) WITH (...);

INSERT INTO students
  VALUES ('fred flintstone', 35, 1.28), ('barney rubble', 32, 2.32);
```

