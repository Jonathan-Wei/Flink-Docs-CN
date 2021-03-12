# Hive方言

从1.11.0开始，使用Hive 方言时，Flink允许用户以Hive语法编写SQL语句。通过提供与Hive语法的兼容性，我们旨在改善与Hive的互操作性，并减少用户需要在Flink和Hive之间切换以执行不同语句的情况。

## 使用Hive方言

Flink当前支持两种SQL方言：`default`和`hive`。必须先切换到Hive 方言，然后才能使用Hive语法编写。下面介绍如何使用SQL Client和Table API设置方言。还要注意，您可以为执行的每个语句动态切换方言。无需重新启动会话即可使用其他方言。

### SQL API

可以通过`table.sql-dialect`属性指定SQL Dialect。因此，您可以在yaml文件的`configuration`部分中为SQL客户端设置要使用的初始方言。

```yaml
execution:
  planner: blink
  type: batch
  result-mode: table

configuration:
  table.sql-dialect: hive
```

也可以在启动SQL Client后设置方言。

```sql
Flink SQL> set table.sql-dialect=hive; -- to use hive dialect
[INFO] Session property has been set.

Flink SQL> set table.sql-dialect=default; -- to use default dialect
[INFO] Session property has been set.
```

### Table API

您可以使用Table API为TableEnvironment设置方言。

{% tabs %}
{% tab title="Java" %}
```java
EnvironmentSettings settings = EnvironmentSettings.newInstance().useBlinkPlanner()...build();
TableEnvironment tableEnv = TableEnvironment.create(settings);
// to use hive dialect
tableEnv.getConfig().setSqlDialect(SqlDialect.HIVE);
// to use default dialect
tableEnv.getConfig().setSqlDialect(SqlDialect.DEFAULT);
```
{% endtab %}

{% tab title="Python" %}
```python
from pyflink.table import *

settings = EnvironmentSettings.new_instance().in_batch_mode().use_blink_planner().build()
t_env = BatchTableEnvironment.create(environment_settings=settings)

# to use hive dialect
t_env.get_config().set_sql_dialect(SqlDialect.HIVE)
# to use default dialect
t_env.get_config().set_sql_dialect(SqlDialect.DEFAULT)
```
{% endtab %}
{% endtabs %}

## DDL

本节列出了Hive方言支持的DDL。在这里，我们主要关注语法。您可以参考[Hive doc](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL) 了解每个DDL语句的语义。

### CATALOG

#### SHOW

```sql
SHOW CURRENT CATALOG;
```

### DATABASE

#### SHOW

```sql
SHOW DATABASES;
SHOW CURRENT DATABASE;
```

#### CREATE

```sql
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION fs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```

#### **ALTER**

**更新属性**

```sql
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);
```

**更新责任人**

```sql
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;
```

**更新位置**

```sql
ALTER (DATABASE|SCHEMA) database_name SET LOCATION fs_path;
```

**删除**

```sql
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

**Use**

```sql
USE database_name;
```

### TABLE

#### **Show**

```sql
SHOW TABLES;
```

#### **Create**

```sql
CREATE [EXTERNAL] TABLE [IF NOT EXISTS] table_name
  [(col_name data_type [column_constraint] [COMMENT col_comment], ... [table_constraint])]
  [COMMENT table_comment]
  [PARTITIONED BY (col_name data_type [COMMENT col_comment], ...)]
  [
    [ROW FORMAT row_format]
    [STORED AS file_format]
  ]
  [LOCATION fs_path]
  [TBLPROPERTIES (property_name=property_value, ...)]

row_format:
  : DELIMITED [FIELDS TERMINATED BY char [ESCAPED BY char]] [COLLECTION ITEMS TERMINATED BY char]
      [MAP KEYS TERMINATED BY char] [LINES TERMINATED BY char]
      [NULL DEFINED AS char]
  | SERDE serde_name [WITH SERDEPROPERTIES (property_name=property_value, ...)]

file_format:
  : SEQUENCEFILE
  | TEXTFILE
  | RCFILE
  | ORC
  | PARQUET
  | AVRO
  | INPUTFORMAT input_format_classname OUTPUTFORMAT output_format_classname

column_constraint:
  : NOT NULL [[ENABLE|DISABLE] [VALIDATE|NOVALIDATE] [RELY|NORELY]]

table_constraint:
  : [CONSTRAINT constraint_name] PRIMARY KEY (col_name, ...) [[ENABLE|DISABLE] [VALIDATE|NOVALIDATE] [RELY|NORELY]]
```

#### **Alter**

**重命名**

```sql
ALTER TABLE table_name RENAME TO new_table_name;
```

**更新属性**

```sql
ALTER TABLE table_name SET TBLPROPERTIES (property_name = property_value, property_name = property_value, ... );
```

**更新位置**

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET LOCATION fs_path;
```

如果`partition_spec`存在，那么它必须是一个完整的spec，也就是说，它有所有分区列的值。当它存在时，运算就会被应用到相应的分区上而不是表上。

**更新文件格式**

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET FILEFORMAT file_format;
```

如果`partition_spec`存在，那么它必须是一个完整的spec，也就是说，它有所有分区列的值。当它存在时，运算就会被应用到相应的分区上而不是表上。

**更新 SerDe 属性**

```sql
ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties];

ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties;

serde_properties:
  : (property_name = property_value, property_name = property_value, ... )
```

如果`partition_spec`存在，那么它必须是一个完整的spec，也就是说，它有所有分区列的值。当它存在时，运算就会被应用到相应的分区上而不是表上。

**Add Partitions**

```sql
ALTER TABLE table_name ADD [IF NOT EXISTS] (PARTITION partition_spec [LOCATION fs_path])+;
```

**Drop Partitions**

```sql
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec[, PARTITION partition_spec, ...];
```

**Add/Replace Columns**

```sql
ALTER TABLE table_name
  ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)
  [CASCADE|RESTRICT]
```

**Change Column**

```sql
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type
  [COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT];
```

#### **Drop**

```sql
DROP TABLE [IF EXISTS] table_name;
```

### VIEW

#### **Create**

```sql
CREATE VIEW [IF NOT EXISTS] view_name [(column_name, ...) ]
  [COMMENT view_comment]
  [TBLPROPERTIES (property_name = property_value, ...)]
  AS SELECT ...;
```

#### **Alter**

**NOTE**: 修改视图只能在Table API中工作，不支持通过SQL客户端。

**重命名**

```sql
ALTER VIEW view_name RENAME TO new_view_name;
```

**更新 Properties**

```sql
ALTER VIEW view_name SET TBLPROPERTIES (property_name = property_value, ... );
```

**通过Select更新**

```sql
ALTER VIEW view_name AS select_statement;
```

#### **Drop**

```sql
DROP VIEW [IF EXISTS] view_name;
```

### FUNCTION

#### **Show**

```sql
SHOW FUNCTIONS;
```

#### **Create**

```sql
CREATE FUNCTION function_name AS class_name;
```

#### **Drop**

```sql
DROP FUNCTION [IF EXISTS] function_name;
```

## DML

### INSERT

```sql
INSERT (INTO|OVERWRITE) [TABLE] table_name [PARTITION partition_spec] SELECT ...;
```

`partition_spec`如果存在，可以是完整的规范，也可以是部分的规范。如果`partition_spec`是部分的规范，可以省略动态分区列名。

## DQL

目前，Hive方言支持与DQL的Flink SQL相同的语法。有关更多详细信息，请参考 [Flink SQL查询](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/sql/queries.html)。并且建议切换到 `default`方言以执行DQL。

## 注意点

以下是使用Hive方言的一些注意事项。

* Hive方言只能用于操作Hive表，不能用于一般表。Hive方言应与[HiveCatalog](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/hive/hive_catalog.html)一起使用。
* 虽然所有Hive版本都支持相同的语法，但是是否有特定功能仍然取决于您使用的 [Hive版本](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/hive/#supported-hive-versions)。例如，仅在Hive-2.4.0或更高版本中支持更新数据库位置。
* Hive和方解石具有不同的保留关键字集。例如，`default`在Calcite中是保留关键字，在Hive中是非保留关键字。即使使用Hive方言，也必须使用反引号（\`）引用此类关键字，才能将其用作标识符。
* 由于扩大了查询的不兼容性，因此无法在Hive中查询在Flink中创建的视图。

