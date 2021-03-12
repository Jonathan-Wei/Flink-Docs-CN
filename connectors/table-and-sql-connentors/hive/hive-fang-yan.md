# Hive方言

从1.11.0开始，使用Hive 方言时，Flink允许用户以Hive语法编写SQL语句。通过提供与Hive语法的兼容性，我们旨在改善与Hive的互操作性，并减少用户需要在Flink和Hive之间切换以执行不同语句的情况。

## 使用Hive方言

Flink当前支持两种SQL方言：`default`和`hive`。必须先切换到Hive 方言，然后才能使用Hive语法编写。下面介绍如何使用SQL Client和Table API设置方言。还要注意，您可以为执行的每个语句动态切换方言。无需重新启动会话即可使用其他方言。

### SQL API

可以通过`table.sql-dialect`属性指定SQL Dialect。因此，您可以在yaml文件的`configuration`部分中为SQL客户端设置要使用的初始方言。

```text
execution:
  planner: blink
  type: batch
  result-mode: table

configuration:
  table.sql-dialect: hive
```

也可以在启动SQL Client后设置方言。

```text
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

**显示**

```text
SHOW CURRENT CATALOG;
```

### DATABASE

**显示**

```text
SHOW DATABASES;
SHOW CURRENT DATABASE;
```

**创建**

```text
CREATE (DATABASE|SCHEMA) [IF NOT EXISTS] database_name
  [COMMENT database_comment]
  [LOCATION fs_path]
  [WITH DBPROPERTIES (property_name=property_value, ...)];
```

**修改**

**更新属性**

```text
ALTER (DATABASE|SCHEMA) database_name SET DBPROPERTIES (property_name=property_value, ...);
```

**更新责任人**

```text
ALTER (DATABASE|SCHEMA) database_name SET OWNER [USER|ROLE] user_or_role;
```

**更新位置**

```text
ALTER (DATABASE|SCHEMA) database_name SET LOCATION fs_path;
```

**删除**

```text
DROP (DATABASE|SCHEMA) [IF EXISTS] database_name [RESTRICT|CASCADE];
```

**Use**

```text
USE database_name;
```

### TABLE

**Show**

```text
SHOW TABLES;
```

**Create**

```text
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

**Alter**

**Rename**

```text
ALTER TABLE table_name RENAME TO new_table_name;
```

**Update Properties**

```text
ALTER TABLE table_name SET TBLPROPERTIES (property_name = property_value, property_name = property_value, ... );
```

**Update Location**

```text
ALTER TABLE table_name [PARTITION partition_spec] SET LOCATION fs_path;
```

The `partition_spec`, if present, needs to be a full spec, i.e. has values for all partition columns. And when it’s present, the operation will be applied to the corresponding partition instead of the table.

**Update File Format**

```text
ALTER TABLE table_name [PARTITION partition_spec] SET FILEFORMAT file_format;
```

The `partition_spec`, if present, needs to be a full spec, i.e. has values for all partition columns. And when it’s present, the operation will be applied to the corresponding partition instead of the table.

**Update SerDe Properties**

```text
ALTER TABLE table_name [PARTITION partition_spec] SET SERDE serde_class_name [WITH SERDEPROPERTIES serde_properties];

ALTER TABLE table_name [PARTITION partition_spec] SET SERDEPROPERTIES serde_properties;

serde_properties:
  : (property_name = property_value, property_name = property_value, ... )
```

The `partition_spec`, if present, needs to be a full spec, i.e. has values for all partition columns. And when it’s present, the operation will be applied to the corresponding partition instead of the table.

**Add Partitions**

```text
ALTER TABLE table_name ADD [IF NOT EXISTS] (PARTITION partition_spec [LOCATION fs_path])+;
```

**Drop Partitions**

```text
ALTER TABLE table_name DROP [IF EXISTS] PARTITION partition_spec[, PARTITION partition_spec, ...];
```

**Add/Replace Columns**

```text
ALTER TABLE table_name
  ADD|REPLACE COLUMNS (col_name data_type [COMMENT col_comment], ...)
  [CASCADE|RESTRICT]
```

**Change Column**

```text
ALTER TABLE table_name CHANGE [COLUMN] col_old_name col_new_name column_type
  [COMMENT col_comment] [FIRST|AFTER column_name] [CASCADE|RESTRICT];
```

**Drop**

```text
DROP TABLE [IF EXISTS] table_name;
```

### VIEW

**Create**

```text
CREATE VIEW [IF NOT EXISTS] view_name [(column_name, ...) ]
  [COMMENT view_comment]
  [TBLPROPERTIES (property_name = property_value, ...)]
  AS SELECT ...;
```

**Alter**

**NOTE**: Altering view only works in Table API, but not supported via SQL client.

**Rename**

```text
ALTER VIEW view_name RENAME TO new_view_name;
```

**Update Properties**

```text
ALTER VIEW view_name SET TBLPROPERTIES (property_name = property_value, ... );
```

**Update As Select**

```text
ALTER VIEW view_name AS select_statement;
```

**Drop**

```text
DROP VIEW [IF EXISTS] view_name;
```

### FUNCTION

**Show**

```text
SHOW FUNCTIONS;
```

**Create**

```text
CREATE FUNCTION function_name AS class_name;
```

**Drop**

```text
DROP FUNCTION [IF EXISTS] function_name;
```

## DML

### INSERT

```text
INSERT (INTO|OVERWRITE) [TABLE] table_name [PARTITION partition_spec] SELECT ...;
```

The `partition_spec`, if present, can be either a full spec or partial spec. If the `partition_spec` is a partial spec, the dynamic partition column names can be omitted.

## DQL

At the moment, Hive dialect supports the same syntax as Flink SQL for DQLs. Refer to [Flink SQL queries](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/sql/queries.html) for more details. And it’s recommended to switch to `default` dialect to execute DQLs.

## 注意点

The following are some precautions for using the Hive dialect.

* Hive dialect should only be used to manipulate Hive tables, not generic tables. And Hive dialect should be used together with a [HiveCatalog](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/hive/hive_catalog.html).
* While all Hive versions support the same syntax, whether a specific feature is available still depends on the [Hive version](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/table/connectors/hive/#supported-hive-versions) you use. For example, updating database location is only supported in Hive-2.4.0 or later.
* Hive and Calcite have different sets of reserved keywords. For example, `default` is a reserved keyword in Calcite and a non-reserved keyword in Hive. Even with Hive dialect, you have to quote such keywords with backtick \( \` \) in order to use them as identifiers.
* Due to expanded query incompatibility, views created in Flink cannot be queried in Hive.

