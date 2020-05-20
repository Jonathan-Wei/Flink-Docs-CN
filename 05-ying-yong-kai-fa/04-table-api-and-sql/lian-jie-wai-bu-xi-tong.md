# 连接外部系统

Flink的Table API和SQL程序可以连接到其他外部系统，以便读取和写入批处理表和流表。表源提供对存储在外部系统（例如数据库，键值存储，消息队列或文件系统）中的数据的访问。表接收器将表发送到外部存储系统。根据源和接收器的类型，它们支持不同的格式，如CSV，Parquet或ORC。

本页介绍如何声明内置表源和/或表接收器，并在Flink中注册它们。注册源或接收器后，可以通过Table API和SQL语句访问它。

{% hint style="danger" %}
**注意:**如果要实现自己的_自定义_表源或接收器，请查看[用户定义的源和接收器页面](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sourceSinks.html)。
{% endhint %}

## 依赖

下表列出了所有可用的连接器和格式。它们的相互兼容性在[表连接器](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html#table-connectors)和[表格格式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html#table-formats)的相应部分中标记。下表提供了使用构建自动化工具（如Maven或SBT）和带有SQL JAR包的SQL Client的两个项目的依赖关系信息。

### 连接器

| 名称 | 版本 | Maven依赖 | SQL Client JAR |
| :--- | :--- | :--- | :--- |
| Filesystem |  | Built-in | Built-in |
| Elasticsearch | 6 | flink-connector-elasticsearch6 | \`\`[Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-elasticsearch6_2.11/1.7.2/flink-connector-elasticsearch6_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.8 | flink-connector-kafka-0.8 | Not available |
| Apache Kafka | 0.9 | flink-connector-kafka-0.9 | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.9_2.11/1.7.2/flink-connector-kafka-0.9_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.10 | flink-connector-kafka-0.10 | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.10_2.11/1.7.2/flink-connector-kafka-0.10_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.11 | flink-connector-kafka-0.11 | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.11_2.11/1.7.2/flink-connector-kafka-0.11_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.11+ \(`universal`\) | flink-connector-kafka | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka_2.11/1.7.2/flink-connector-kafka_2.11-1.7.2-sql-jar.jar) |
| HBase | 1.4.3 | flink-hbase |  [Download](https://repo.maven.apache.org/maven2/org/apache/flink/flink-hbase_2.11/1.10.0/flink-hbase_2.11-1.10.0.jar) |
| JDBC |  | flink-jdbc |  |

### 格式

| 名称 | Maven依赖 | SQL Client JAR |
| :--- | :--- | :--- |
| Old CSV \(for files\) | Built-in | Built-in |
| CSV \(for Kafka\) | flink-csv |  [Download](https://repo.maven.apache.org/maven2/org/apache/flink/flink-csv/1.10.0/flink-csv-1.10.0-sql-jar.jar) |
| JSON | flink-json | [Download](http://central.maven.org/maven2/org/apache/flink/flink-json/1.7.2/flink-json-1.7.2-sql-jar.jar) |
| Apache Avro | flink-avro | [Download](http://central.maven.org/maven2/org/apache/flink/flink-avro/1.7.2/flink-avro-1.7.2-sql-jar.jar) |

## 概述

从Flink 1.6开始，与外部系统的连接声明与实际实现分开。

也可以指定连接

* **以编程方式**在表和SQL API的**org.apache.flink.table.descriptors**下使用**Descriptor**
* 或**声明性地**通过SQL客户端的[YAML配置文件](http://yaml.org/)。

这不仅可以更好地统一API和SQL Client，还可以在[自定义实现的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sourceSinks.html)情况下[实现](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sourceSinks.html)更好的可扩展性，而无需更改实际声明。

每个声明都类似于`SQL CREATE TABLE`语句。 可以预先定义表的名称，表的模式，连接器以及用于连接到外部系统的数据格式。

连接器描述了存储表数据的外部系统。 可以在此处声明Apacha Kafka或常规文件系统等存储系统。 连接器可能已经提供了具有字段和架构的固定格式。

某些系统支持不同的数据格式。例如，存储在Kafka或文件中的表可以使用CSV，JSON或Avro对其行进行编码。数据库连接器可能需要此处的表模式。每个连接器都记录了存储系统是否需要定义格式。不同的系统还需要不同类型的格式（例如，面向列的格式与面向行的格式）。该文档说明了哪些格式类型和连接器兼容。

表模式定义了向SQL查询公开的表的模式。它描述了源如何将数据格式映射到表模式，反之亦然。模式可以访问连接器或格式定义的字段。它可以使用一个或多个字段来提取或插入时间属性。如果输入字段没有确定性字段顺序，则模式清楚地定义列名称，它们的顺序和原点。

后续部分将更详细地介绍每个定义部分（连接器，格式和模式）。以下示例展示了如何传递它们：

{% tabs %}
{% tab title="DDL" %}
```sql
tableEnvironment.sqlUpdate(
    "CREATE TABLE MyTable (\n" +
    "  ...    -- declare table schema \n" +
    ") WITH (\n" +
    "  'connector.type' = '...',  -- declare connector specific properties\n" +
    "  ...\n" +
    "  'update-mode' = 'append',  -- declare update mode\n" +
    "  'format.type' = '...',     -- declare format specific properties\n" +
    "  ...\n" +
    ")");
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
tableEnvironment
  .connect(...)
  .withFormat(...)
  .withSchema(...)
  .inAppendMode()
  .createTemporaryTable("MyTable")
```
{% endtab %}

{% tab title="Python" %}
```python
table_environment \
    .connect(...) \
    .with_format(...) \
    .with_schema(...) \
    .in_append_mode() \
    .create_temporary_table("MyTable")
```
{% endtab %}

{% tab title="YAML" %}
```yaml
name: MyTable
type: source
update-mode: append
connector: ...
format: ...
schema: ...
```
{% endtab %}
{% endtabs %}

表的类型（源，接收器或两者）确定表的注册方式。 如果是表类型，则表源和表接收器都以相同的名称注册。 从逻辑上讲，这意味着我们可以读取和写入这样的表，类似于常规DBMS中的表。

 对于流查询，[更新模式](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html#update-mode)声明了如何在动态表和存储系统之间进行通信以进行连续查询。

以下代码显示了如何连接到Kafka以读取Avro记录的完整示例。

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  -- declare the schema of the table
  `user` BIGINT,
  message STRING,
  ts STRING
) WITH (
  -- declare the external system to connect to
  'connector.type' = 'kafka',
  'connector.version' = '0.10',
  'connector.topic' = 'topic_name',
  'connector.startup-mode' = 'earliest-offset',
  'connector.properties.zookeeper.connect' = 'localhost:2181',
  'connector.properties.bootstrap.servers' = 'localhost:9092',

  -- specify the update-mode for streaming tables
  'update-mode' = 'append',

  -- declare a format for this system
  'format.type' = 'avro',
  'format.avro-schema' = '{
                            "namespace": "org.myorganization",
                            "type": "record",
                            "name": "UserMessage",
                            "fields": [
                                {"name": "ts", "type": "string"},
                                {"name": "user", "type": "long"},
                                {"name": "message", "type": ["string", "null"]}
                            ]
                         }'
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
tableEnvironment
  // declare the external system to connect to
  .connect(
    new Kafka()
      .version("0.10")
      .topic("test-input")
      .startFromEarliest()
      .property("zookeeper.connect", "localhost:2181")
      .property("bootstrap.servers", "localhost:9092")
  )

  // declare a format for this system
  .withFormat(
    new Avro()
      .avroSchema(
        "{" +
        "  \"namespace\": \"org.myorganization\"," +
        "  \"type\": \"record\"," +
        "  \"name\": \"UserMessage\"," +
        "    \"fields\": [" +
        "      {\"name\": \"timestamp\", \"type\": \"string\"}," +
        "      {\"name\": \"user\", \"type\": \"long\"}," +
        "      {\"name\": \"message\", \"type\": [\"string\", \"null\"]}" +
        "    ]" +
        "}"
      )
  )

  // declare the schema of the table
  .withSchema(
    new Schema()
      .field("rowtime", DataTypes.TIMESTAMP(3))
        .rowtime(new Rowtime()
          .timestampsFromField("timestamp")
          .watermarksPeriodicBounded(60000)
        )
      .field("user", DataTypes.BIGINT())
      .field("message", DataTypes.STRING())
  )

  // create a table with given name
  .createTemporaryTable("MyUserTable");
```
{% endtab %}

{% tab title="Python" %}
```python
table_environment \
    .connect(  # declare the external system to connect to
        Kafka()
        .version("0.10")
        .topic("test-input")
        .start_from_earliest()
        .property("zookeeper.connect", "localhost:2181")
        .property("bootstrap.servers", "localhost:9092")
    ) \
    .with_format(  # declare a format for this system
        Avro()
        .avro_schema(
            "{"
            "  \"namespace\": \"org.myorganization\","
            "  \"type\": \"record\","
            "  \"name\": \"UserMessage\","
            "    \"fields\": ["
            "      {\"name\": \"timestamp\", \"type\": \"string\"},"
            "      {\"name\": \"user\", \"type\": \"long\"},"
            "      {\"name\": \"message\", \"type\": [\"string\", \"null\"]}"
            "    ]"
            "}"
        )
    ) \
    .with_schema(  # declare the schema of the table
        Schema()
        .field("rowtime", DataTypes.TIMESTAMP(3))
        .rowtime(
            Rowtime()
            .timestamps_from_field("timestamp")
            .watermarks_periodic_bounded(60000)
        )
        .field("user", DataTypes.BIGINT())
        .field("message", DataTypes.STRING())
    ) \
    .create_temporary_table("MyUserTable")
    # register as source, sink, or both and under a name
```
{% endtab %}

{% tab title="YAML" %}
```yaml
tables:
  - name: MyUserTable      # name the new table
    type: source           # declare if the table should be "source", "sink", or "both"

    # declare the external system to connect to
    connector:
      type: kafka
      version: "0.10"
      topic: test-input
      startup-mode: earliest-offset
      properties:
        zookeeper.connect: localhost:2181
        bootstrap.servers: localhost:9092

    # declare a format for this system
    format:
      type: avro
      avro-schema: >
        {
          "namespace": "org.myorganization",
          "type": "record",
          "name": "UserMessage",
            "fields": [
              {"name": "ts", "type": "string"},
              {"name": "user", "type": "long"},
              {"name": "message", "type": ["string", "null"]}
            ]
        }

    # declare the schema of the table
    schema:
      - name: rowtime
        data-type: TIMESTAMP(3)
        rowtime:
          timestamps:
            type: from-field
            from: ts
          watermarks:
            type: periodic-bounded
            delay: "60000"
      - name: user
        data-type: BIGINT
      - name: message
        data-type: STRING
```
{% endtab %}
{% endtabs %}

两种方式都将所需的连接属性转换为标准化的基于字符串的键值对。所谓的[表工厂](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sourceSinks.html#define-a-tablefactory)根据键值对创建已配置的表源，表接收器和相应的格式。搜索完全匹配的一个表工厂时，将考虑通过Java [服务提供商接口（SPI）](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html)可以找到的所有表工厂。

如果找不到给定属性的工厂或多个工厂匹配，则将引发异常，并提供有关考虑的工厂和支持的属性的其他信息。

## 表格式

表模式定义类的名称和类型，类似于`SQL CREATE TABLE`语句的列定义。 此外，可以指定列的表示方式和表格数据编码格式的字段。 如果列的名称应与输入/输出格式不同，则字段的来源可能很重要。 例如，列`user_name`应从JSON格式引用字段`$$-user-name`。 此外，需要模式将类型从外部系统映射到Flink的表示。 对于表接收器，它确保仅将具有有效模式的数据写入外部系统。

以下示例展示了没有时间属性的简单模式以及输入/输出到表列的一对一字段映射。

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withSchema(
  new Schema()
    .field("MyField1", DataTypes.INT())     // required: specify the fields of the table (in this order)
    .field("MyField2", DataTypes.STRING())
    .field("MyField3", DataTypes.BOOLEAN())
)
```
{% endtab %}

{% tab title="Python" %}
```python
.with_schema(
    Schema()
    .field("MyField1", DataTypes.INT())  # required: specify the fields of the table (in this order)
    .field("MyField2", DataTypes.STRING())
    .field("MyField3", DataTypes.BOOLEAN())
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
schema:
  - name: MyField1    # required: specify the fields of the table (in this order)
    data-type: INT
  - name: MyField2
    data-type: STRING
  - name: MyField3
    data-type: BOOLEAN
```
{% endtab %}
{% endtabs %}

对于每个字段，除了列的名称和类型之外，还可以声明以下属性：

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withSchema(
  new Schema()
    .field("MyField1", DataTypes.TIMESTAMP(3))
      .proctime()      // optional: declares this field as a processing-time attribute
    .field("MyField2", DataTypes.TIMESTAMP(3))
      .rowtime(...)    // optional: declares this field as a event-time attribute
    .field("MyField3", DataTypes.BOOLEAN())
      .from("mf3")     // optional: original field in the input that is referenced/aliased by this field
)
```
{% endtab %}

{% tab title="Python" %}
```python
.with_schema(
    Schema()
    .field("MyField1", DataTypes.TIMESTAMP(3))
    .proctime()  # optional: declares this field as a processing-time attribute
    .field("MyField2", DataTypes.TIMESTAMP(3))
    .rowtime(...)  # optional: declares this field as a event-time attribute
    .field("MyField3", DataTypes.BOOLEAN())
    .from_origin_field("mf3")  # optional: original field in the input that is referenced/aliased by this field
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
schema:
  - name: MyField1
    data-type: TIMESTAMP(3)
    proctime: true    # optional: boolean flag whether this field should be a processing-time attribute
  - name: MyField2
    data-type: TIMESTAMP(3)
    rowtime: ...      # optional: wether this field should be a event-time attribute
  - name: MyField3
    data-type: BOOLEAN
    from: mf3         # optional: original field in the input that is referenced/aliased by this field
```
{% endtab %}
{% endtabs %}

使用无界流表时，时间属性是必不可少的。因此，处理时间和事件时间（也称为“行时”）属性都可以定义为模式的一部分。

有关Flink中时间处理的更多信息，特别是事件时间，我们建议使用常规[事件时间部分](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)。

### RowTime属性

为了控制表的事件时间行​​为，Flink提供了预定义的时间戳提取器和水印策略。

支持以下时间戳提取器：

{% tabs %}
{% tab title="Java/Scala" %}
```java
// Converts an existing LONG or SQL_TIMESTAMP field in the input into the rowtime attribute.
.rowtime(
  new Rowtime()
    .timestampsFromField("ts_field")    // required: original field name in the input
)

// Converts the assigned timestamps from a DataStream API record into the rowtime attribute
// and thus preserves the assigned timestamps from the source.
// This requires a source that assigns timestamps (e.g., Kafka 0.10+).
.rowtime(
  new Rowtime()
    .timestampsFromSource()
)

// Sets a custom timestamp extractor to be used for the rowtime attribute.
// The extractor must extend `org.apache.flink.table.sources.tsextractors.TimestampExtractor`.
.rowtime(
  new Rowtime()
    .timestampsFromExtractor(...)
)
```
{% endtab %}

{% tab title="Python" %}
```python
# Converts an existing BIGINT or TIMESTAMP field in the input into the rowtime attribute.
.rowtime(
    Rowtime()
    .timestamps_from_field("ts_field")  # required: original field name in the input
)

# Converts the assigned timestamps into the rowtime attribute
# and thus preserves the assigned timestamps from the source.
# This requires a source that assigns timestamps (e.g., Kafka 0.10+).
.rowtime(
    Rowtime()
    .timestamps_from_source()
)

# Sets a custom timestamp extractor to be used for the rowtime attribute.
# The extractor must extend `org.apache.flink.table.sources.tsextractors.TimestampExtractor`.
# Due to python can not accept java object, so it requires a full-qualified class name of the extractor.
.rowtime(
    Rowtime()
    .timestamps_from_extractor(...)
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
# Converts an existing BIGINT or TIMESTAMP field in the input into the rowtime attribute.
rowtime:
  timestamps:
    type: from-field
    from: "ts_field"                 # required: original field name in the input

# Converts the assigned timestamps from a DataStream API record into the rowtime attribute
# and thus preserves the assigned timestamps from the source.
rowtime:
  timestamps:
    type: from-source
```
{% endtab %}
{% endtabs %}

支持以下水印策略：

{% tabs %}
{% tab title="Java/Scala" %}
```java
// Sets a watermark strategy for ascending rowtime attributes. Emits a watermark of the maximum
// observed timestamp so far minus 1. Rows that have a timestamp equal to the max timestamp
// are not late.
.rowtime(
  new Rowtime()
    .watermarksPeriodicAscending()
)

// Sets a built-in watermark strategy for rowtime attributes which are out-of-order by a bounded time interval.
// Emits watermarks which are the maximum observed timestamp minus the specified delay.
.rowtime(
  new Rowtime()
    .watermarksPeriodicBounded(2000)    // delay in milliseconds
)

// Sets a built-in watermark strategy which indicates the watermarks should be preserved from the
// underlying DataStream API and thus preserves the assigned watermarks from the source.
.rowtime(
  new Rowtime()
    .watermarksFromSource()
)
```
{% endtab %}

{% tab title="Python" %}
```python
# Sets a watermark strategy for ascending rowtime attributes. Emits a watermark of the maximum
# observed timestamp so far minus 1. Rows that have a timestamp equal to the max timestamp
# are not late.
.rowtime(
    Rowtime()
    .watermarks_periodic_ascending()
)

# Sets a built-in watermark strategy for rowtime attributes which are out-of-order by a bounded time interval.
# Emits watermarks which are the maximum observed timestamp minus the specified delay.
.rowtime(
    Rowtime()
    .watermarks_periodic_bounded(2000)  # delay in milliseconds
)

# Sets a built-in watermark strategy which indicates the watermarks should be preserved from the
# underlying DataStream API and thus preserves the assigned watermarks from the source.
.rowtime(
    Rowtime()
    .watermarks_from_source()
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
# Sets a watermark strategy for ascending rowtime attributes. Emits a watermark of the maximum
# observed timestamp so far minus 1. Rows that have a timestamp equal to the max timestamp
# are not late.
rowtime:
  watermarks:
    type: periodic-ascending

# Sets a built-in watermark strategy for rowtime attributes which are out-of-order by a bounded time interval.
# Emits watermarks which are the maximum observed timestamp minus the specified delay.
rowtime:
  watermarks:
    type: periodic-bounded
    delay: ...                # required: delay in milliseconds

# Sets a built-in watermark strategy which indicates the watermarks should be preserved from the
# underlying DataStream API and thus preserves the assigned watermarks from the source.
rowtime:
  watermarks:
    type: from-source
```
{% endtab %}
{% endtabs %}

确保始终声明时间戳和水印。 触发基于时间的操作需要水印。

### 类型字符串

 因为`DataType`仅在编程语言中可用，所以支持在YAML文件中定义类型字符串。类型字符串与SQL中的类型声明相同，请参见“ [数据类型”](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/types.html)页面，了解如何在SQL中声明类型。

## 更新模式

对于流式查询，需要声明如何[在动态表和外部连接器](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#continuous-queries)之间执行[转换](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#continuous-queries)。更新模式指定应与外部系统交换哪种消息：

**追加模式：**在追加模式下，动态表和外部连接器仅交换`INSERT`消息。

**回退模式：**在回退模式下，动态表和外部连接器交换`ADD`和`RETRACT`消息。 `INSERT`更改被编码为`ADD`消息，`DELETE`更改被编码为`RETRACT`消息，`UPDATE`更改被编码为更新（前一个）行的`RETRACT`消息和更新（新）行的`ADD`消息。在此模式下，不能定义键而不是`upsert`模式。但是，每次更新都包含两个效率较低的消息。

**Upsert模式：**在`upsert`模式下，动态表和外部连接器交换`UPSERT`和`DELETE`消息。此模式需要一个（可能是复合的）唯一密钥，通过该密钥可以传播更新。外部连接器需要知道唯一键属性才能正确应用消息。 `INSERT`和`UPDATE`更改被编码为`UPSERT`消息。 `DELETE`更改为`DELETE`消息。与回退流的主要区别在于`UPDATE`更改使用单个消息进行编码，因此更有效。

{% hint style="danger" %}
**注意：**每个连接器的文档都说明了支持哪些更新模式。
{% endhint %}

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyTable (
 ...
) WITH (
 'update-mode' = 'append'  -- otherwise: 'retract' or 'upsert'
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.connect(...)
  .inAppendMode()    // otherwise: inUpsertMode() or inRetractMode()
```
{% endtab %}

{% tab title="Python" %}
```python
.connect(...) \
    .in_append_mode()  # otherwise: in_upsert_mode() or in_retract_mode()
```
{% endtab %}

{% tab title="YAML" %}
```yaml
tables:
  - name: ...
    update-mode: append    # otherwise: "retract" or "upsert"
```
{% endtab %}
{% endtabs %}

有关更多信息，另请参阅[常规流概述文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#continuous-queries)。

## 表连接器

Flink提供了一组用于连接外部系统的连接器。

请注意，并非所有连接器都可用于批量和流式传输。 此外，并非每个流连接器都支持每种流模式。 因此，相应地标记每个连接器。 格式标记表示连接器需要特定类型的格式。

### 文件系统连接器

{% hint style="info" %}
**来源：**批量   
**来源：**流媒体附加模式   
**接收器：**批量   
**接收器：**流式附加模式   
**格式：**仅限Old CSV
{% endhint %}

文件系统连接器允许从本地或分布式文件系统进行读写。文件系统可以定义为：

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  ...
) WITH (
  'connector.type' = 'filesystem',                -- required: specify to connector type
  'connector.path' = 'file:///path/to/whatever',  -- required: path to a file or directory
  'format.type' = '...',                          -- required: file system connector requires to specify a format,
  ...                                             -- currently only 'csv' format is supported.
                                                  -- Please refer to old CSV format part of Table Formats section for more details.
)  
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.connect(
  new FileSystem()
    .path("file:///path/to/whatever")    // required: path to a file or directory
)
.withFormat(                             // required: file system connector requires to specify a format,
  ...                                    // currently only OldCsv format is supported.
)                                        // Please refer to old CSV format part of Table Formats section for more details.
```
{% endtab %}

{% tab title="Python" %}
```python
.connect(
    FileSystem()
    .path("file:///path/to/whatever")  # required: path to a file or directory
)
.withFormat(                           # required: file system connector requires to specify a format,
  ...                                  # currently only OldCsv format is supported.
)                                      # Please refer to old CSV format part of Table Formats section for more details.
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: filesystem
  path: "file:///path/to/whatever"    # required: path to a file or directory
format:                               # required: file system connector requires to specify a format,
  ...                                 # currently only 'csv' format is supported.
                                      # Please refer to old CSV format part of Table Formats section for more details.
```
{% endtab %}
{% endtabs %}

文件系统连接器本身包含在Flink中，不需要额外的依赖项。需要指定相应的格式，以便从文件系统读取和写入行。

{% hint style="danger" %}
**注意：**确保包含[特定于Flink文件系统的依赖项](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/filesystems.html)。
{% endhint %}

{% hint style="danger" %}
**注意：**用于流式传输的文件系统源和接收器仅是实验性的。 将来，我们将支持实际的流用例，即目录监控和桶输出。
{% endhint %}

### Kafka连接器

{% hint style="info" %}
**来源：**流附加模式   
**接收器：**流附加模式   
**格式：**序列化模式   
**格式：**反序列化模式
{% endhint %}

Kafka连接器允许从Apache Kafka主题读取和写入。定义如下：

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  ...
) WITH (
  'connector.type' = 'kafka',       

  'connector.version' = '0.11',     -- required: valid connector versions are
                                    -- "0.8", "0.9", "0.10", "0.11", and "universal"

  'connector.topic' = 'topic_name', -- required: topic name from which the table is read

  'connector.properties.zookeeper.connect' = 'localhost:2181', -- required: specify the ZooKeeper connection string
  'connector.properties.bootstrap.servers' = 'localhost:9092', -- required: specify the Kafka server connection string
  'connector.properties.group.id' = 'testGroup', --optional: required in Kafka consumer, specify consumer group
  'connector.startup-mode' = 'earliest-offset',    -- optional: valid modes are "earliest-offset", 
                                                   -- "latest-offset", "group-offsets", 
                                                   -- or "specific-offsets"

  -- optional: used in case of startup mode with specific offsets
  'connector.specific-offsets' = 'partition:0,offset:42;partition:1,offset:300',

  'connector.sink-partitioner' = '...',  -- optional: output partitioning from Flink's partitions 
                                         -- into Kafka's partitions valid are "fixed" 
                                         -- (each Flink partition ends up in at most one Kafka partition),
                                         -- "round-robin" (a Flink partition is distributed to 
                                         -- Kafka partitions round-robin)
                                         -- "custom" (use a custom FlinkKafkaPartitioner subclass)
  -- optional: used in case of sink partitioner custom
  'connector.sink-partitioner-class' = 'org.mycompany.MyPartitioner',
  
  'format.type' = '...',                 -- required: Kafka connector requires to specify a format,
  ...                                    -- the supported formats are 'csv', 'json' and 'avro'.
                                         -- Please refer to Table Formats section for more details.
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.connect(
  new Kafka()
    .version("0.11")    // required: valid connector versions are
                        //   "0.8", "0.9", "0.10", "0.11", and "universal"
    .topic("...")       // required: topic name from which the table is read

    // optional: connector specific properties
    .property("zookeeper.connect", "localhost:2181")
    .property("bootstrap.servers", "localhost:9092")
    .property("group.id", "testGroup")

    // optional: select a startup mode for Kafka offsets
    .startFromEarliest()
    .startFromLatest()
    .startFromSpecificOffsets(...)

    // optional: output partitioning from Flink's partitions into Kafka's partitions
    .sinkPartitionerFixed()         // each Flink partition ends up in at-most one Kafka partition (default)
    .sinkPartitionerRoundRobin()    // a Flink partition is distributed to Kafka partitions round-robin
    .sinkPartitionerCustom(MyCustom.class)    // use a custom FlinkKafkaPartitioner subclass
)
.withFormat(                                  // required: Kafka connector requires to specify a format,
  ...                                         // the supported formats are Csv, Json and Avro.
)                                             // Please refer to Table Formats section for more details.
```
{% endtab %}

{% tab title="Python" %}
```python
.connect(
    Kafka()
    .version("0.11")  # required: valid connector versions are
                      # "0.8", "0.9", "0.10", "0.11", and "universal"
    .topic("...")     # required: topic name from which the table is read
    
    # optional: connector specific properties
    .property("zookeeper.connect", "localhost:2181")
    .property("bootstrap.servers", "localhost:9092")
    .property("group.id", "testGroup")

    # optional: select a startup mode for Kafka offsets
    .start_from_earliest()
    .start_from_latest()
    .start_from_specific_offsets(...)

    # optional: output partitioning from Flink's partitions into Kafka's partitions
    .sink_partitioner_fixed()        # each Flink partition ends up in at-most one Kafka partition (default)
    .sink_partitioner_round_robin()  # a Flink partition is distributed to Kafka partitions round-robin
    .sink_partitioner_custom("full.qualified.custom.class.name")  # use a custom FlinkKafkaPartitioner subclass
)
.withFormat(                         # required: Kafka connector requires to specify a format,
  ...                                # the supported formats are Csv, Json and Avro.
)                                    # Please refer to Table Formats section for more details.
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: kafka
  version: "0.11"     # required: valid connector versions are
                      #   "0.8", "0.9", "0.10", "0.11", and "universal"
  topic: ...          # required: topic name from which the table is read

  properties:
    zookeeper.connect: localhost:2181  # required: specify the ZooKeeper connection string
    bootstrap.servers: localhost:9092  # required: specify the Kafka server connection string
    group.id: testGroup                # optional: required in Kafka consumer, specify consumer group

  startup-mode: ...                                               # optional: valid modes are "earliest-offset", "latest-offset",
                                                                  # "group-offsets", or "specific-offsets"
  specific-offsets: partition:0,offset:42;partition:1,offset:300  # optional: used in case of startup mode with specific offsets
  
  sink-partitioner: ...    # optional: output partitioning from Flink's partitions into Kafka's partitions
                           # valid are "fixed" (each Flink partition ends up in at most one Kafka partition),
                           # "round-robin" (a Flink partition is distributed to Kafka partitions round-robin)
                           # "custom" (use a custom FlinkKafkaPartitioner subclass)
  sink-partitioner-class: org.mycompany.MyPartitioner  # optional: used in case of sink partitioner custom
  
  format:                  # required: Kafka connector requires to specify a format,
    ...                    # the supported formats are "csv", "json" and "avro".
                           # Please refer to Table Formats section for more details.
```
{% endtab %}
{% endtabs %}

**指定开始读取位置：**默认情况下，Kafka源将开始从Zookeeper或Kafka代理中的已提交组偏移量中读取数据。您可以指定其他起始位置，这些位置对应于[Kafka消费者起始位置配置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/kafka.html#kafka-consumers-start-position-configuration)部分中的[配置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/kafka.html#kafka-consumers-start-position-configuration)。

**Flink-Kafka Sink Partitioning：**默认情况下，**Kafka接收**器最多写入与其自身并行性一样多的分区（接收器的每个并行实例只写入一个分区）。为了将写入分发到更多分区或控制行路由到分区，可以提供自定义接收器分区器。循环分区器可用于避免不平衡的分区。但是，它会在所有Flink实例和所有Kafka代理之间产生大量网络连接。

**一致性保证：**默认情况下，如果在[启用](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/checkpointing.html#enabling-and-configuring-checkpointing)了[检查点的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/checkpointing.html#enabling-and-configuring-checkpointing)情况下执行查询，则Kafka接收器会使用至少一次保证将数据提取到Kafka主题中。

**Kafka 0.10+时间戳：**自Kafka 0.10起，Kafka消息的时间戳作为元数据，指定记录何时写入Kafka主题。通过分别在YAML和Java / Scala中选择，可以将这些时间戳用于[行时属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html#defining-the-schema)。`timestamps: from-sourcetimestampsFromSource()`

**Kafka 0.11+版本控制：**自Flink 1.7起，Kafka连接器定义应独立于硬编码的Kafka版本。使用连接器版本`universal`作为Flink的Kafka连接器的通配符，该连接器与从0.11开始的所有Kafka版本兼容。

确保添加特定于版本的Kafka依赖项。此外，需要指定相应的格式以便从Kafka读取和写入行。

### Elasticsearch连接器

{% hint style="info" %}
**接收器：**流式附加模式   
**接收器：**流式Upsert模式   
**格式：**仅限JSON
{% endhint %}

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  ...
) WITH (
  'connector.type' = 'elasticsearch', -- required: specify this table type is elasticsearch
  
  'connector.version' = '6',          -- required: valid connector versions are "6"
  
  'connector.hosts' = 'http://host_name:9092;http://host_name:9093',  -- required: one or more Elasticsearch hosts to connect to

  'connector.index' = 'MyUsers',       -- required: Elasticsearch index

  'connector.document-type' = 'user',  -- required: Elasticsearch document type

  'update-mode' = 'append',            -- optional: update mode when used as table sink.           

  'connector.key-delimiter' = '$',     -- optional: delimiter for composite keys ("_" by default)
                                       -- e.g., "$" would result in IDs "KEY1$KEY2$KEY3"

  'connector.key-null-literal' = 'n/a',  -- optional: representation for null fields in keys ("null" by default)

  'connector.failure-handler' = '...',   -- optional: failure handling strategy in case a request to 
                                         -- Elasticsearch fails ("fail" by default).
                                         -- valid strategies are 
                                         -- "fail" (throws an exception if a request fails and
                                         -- thus causes a job failure), 
                                         -- "ignore" (ignores failures and drops the request),
                                         -- "retry-rejected" (re-adds requests that have failed due 
                                         -- to queue capacity saturation), 
                                         -- or "custom" for failure handling with a
                                         -- ActionRequestFailureHandler subclass

  -- optional: configure how to buffer elements before sending them in bulk to the cluster for efficiency
  'connector.flush-on-checkpoint' = 'true',   -- optional: disables flushing on checkpoint (see notes below!)
                                              -- ("true" by default)
  'connector.bulk-flush.max-actions' = '42',  -- optional: maximum number of actions to buffer 
                                              -- for each bulk request
  'connector.bulk-flush.max-size' = '42 mb',  -- optional: maximum size of buffered actions in bytes
                                              -- per bulk request
                                              -- (only MB granularity is supported)
  'connector.bulk-flush.interval' = '60000',  -- optional: bulk flush interval (in milliseconds)
  'connector.bulk-flush.backoff.type' = '...',       -- optional: backoff strategy ("disabled" by default)
                                                      -- valid strategies are "disabled", "constant",
                                                      -- or "exponential"
  'connector.bulk-flush.backoff.max-retries' = '3',  -- optional: maximum number of retries
  'connector.bulk-flush.backoff.delay' = '30000',    -- optional: delay between each backoff attempt
                                                      -- (in milliseconds)

  -- optional: connection properties to be used during REST communication to Elasticsearch
  'connector.connection-max-retry-timeout' = '3',     -- optional: maximum timeout (in milliseconds)
                                                      -- between retries
  'connector.connection-path-prefix' = '/v1'          -- optional: prefix string to be added to every
                                                      -- REST communication
                                                      
  'format.type' = '...',   -- required: Elasticsearch connector requires to specify a format,
  ...                      -- currently only 'json' format is supported.
                           -- Please refer to Table Formats section for more details.
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.connect(
  new Elasticsearch()
    .version("6")                      // required: valid connector versions are "6"
    .host("localhost", 9200, "http")   // required: one or more Elasticsearch hosts to connect to
    .index("MyUsers")                  // required: Elasticsearch index
    .documentType("user")              // required: Elasticsearch document type

    .keyDelimiter("$")        // optional: delimiter for composite keys ("_" by default)
                              //   e.g., "$" would result in IDs "KEY1$KEY2$KEY3"
    .keyNullLiteral("n/a")    // optional: representation for null fields in keys ("null" by default)

    // optional: failure handling strategy in case a request to Elasticsearch fails (fail by default)
    .failureHandlerFail()          // optional: throws an exception if a request fails and causes a job failure
    .failureHandlerIgnore()        //   or ignores failures and drops the request
    .failureHandlerRetryRejected() //   or re-adds requests that have failed due to queue capacity saturation
    .failureHandlerCustom(...)     //   or custom failure handling with a ActionRequestFailureHandler subclass

    // optional: configure how to buffer elements before sending them in bulk to the cluster for efficiency
    .disableFlushOnCheckpoint()    // optional: disables flushing on checkpoint (see notes below!)
    .bulkFlushMaxActions(42)       // optional: maximum number of actions to buffer for each bulk request
    .bulkFlushMaxSize("42 mb")     // optional: maximum size of buffered actions in bytes per bulk request
                                   //   (only MB granularity is supported)
    .bulkFlushInterval(60000L)     // optional: bulk flush interval (in milliseconds)

    .bulkFlushBackoffConstant()    // optional: use a constant backoff type
    .bulkFlushBackoffExponential() //   or use an exponential backoff type
    .bulkFlushBackoffMaxRetries(3) // optional: maximum number of retries
    .bulkFlushBackoffDelay(30000L) // optional: delay between each backoff attempt (in milliseconds)

    // optional: connection properties to be used during REST communication to Elasticsearch
    .connectionMaxRetryTimeout(3)  // optional: maximum timeout (in milliseconds) between retries
    .connectionPathPrefix("/v1")   // optional: prefix string to be added to every REST communication
)
.withFormat(                      // required: Elasticsearch connector requires to specify a format,
  ...                             // currently only Json format is supported.
                                  // Please refer to Table Formats section for more details.
)    
```
{% endtab %}

{% tab title="Python" %}
```python
.connect(
    Elasticsearch()
    .version("6")                      # required: valid connector versions are "6"
    .host("localhost", 9200, "http")   # required: one or more Elasticsearch hosts to connect to
    .index("MyUsers")                  # required: Elasticsearch index
    .document_type("user")             # required: Elasticsearch document type

    .key_delimiter("$")       # optional: delimiter for composite keys ("_" by default)
                              #   e.g., "$" would result in IDs "KEY1$KEY2$KEY3"
    .key_null_literal("n/a")  # optional: representation for null fields in keys ("null" by default)

    # optional: failure handling strategy in case a request to Elasticsearch fails (fail by default)
    .failure_handler_fail()             # optional: throws an exception if a request fails and causes a job failure
    .failure_handler_ignore()           #   or ignores failures and drops the request
    .failure_handler_retry_rejected()   #   or re-adds requests that have failed due to queue capacity saturation
    .failure_handler_custom(...)        #   or custom failure handling with a ActionRequestFailureHandler subclass

    # optional: configure how to buffer elements before sending them in bulk to the cluster for efficiency
    .disable_flush_on_checkpoint()      # optional: disables flushing on checkpoint (see notes below!)
    .bulk_flush_max_actions(42)         # optional: maximum number of actions to buffer for each bulk request
    .bulk_flush_max_size("42 mb")       # optional: maximum size of buffered actions in bytes per bulk request
                                        #   (only MB granularity is supported)
    .bulk_flush_interval(60000)         # optional: bulk flush interval (in milliseconds)

    .bulk_flush_backoff_constant()      # optional: use a constant backoff type
    .bulk_flush_backoff_exponential()   #   or use an exponential backoff type
    .bulk_flush_backoff_max_retries(3)  # optional: maximum number of retries
    .bulk_flush_backoff_delay(30000)    # optional: delay between each backoff attempt (in milliseconds)

    # optional: connection properties to be used during REST communication to Elasticsearch
    .connection_max_retry_timeout(3)    # optional: maximum timeout (in milliseconds) between retries
    .connection_path_prefix("/v1")      # optional: prefix string to be added to every REST communication
)
.withFormat(                      // required: Elasticsearch connector requires to specify a format,
  ...                             // currently only Json format is supported.
                                  // Please refer to Table Formats section for more details.
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: elasticsearch
  version: 6                                            # required: valid connector versions are "6"
    hosts: http://host_name:9092;http://host_name:9093  # required: one or more Elasticsearch hosts to connect to
    index: "MyUsers"        # required: Elasticsearch index
    document-type: "user"   # required: Elasticsearch document type

    key-delimiter: "$"      # optional: delimiter for composite keys ("_" by default)
                            #   e.g., "$" would result in IDs "KEY1$KEY2$KEY3"
    key-null-literal: "n/a" # optional: representation for null fields in keys ("null" by default)

    # optional: failure handling strategy in case a request to Elasticsearch fails ("fail" by default)
    failure-handler: ...    # valid strategies are "fail" (throws an exception if a request fails and
                            #   thus causes a job failure), "ignore" (ignores failures and drops the request),
                            #   "retry-rejected" (re-adds requests that have failed due to queue capacity
                            #   saturation), or "custom" for failure handling with a
                            #   ActionRequestFailureHandler subclass

    # optional: configure how to buffer elements before sending them in bulk to the cluster for efficiency
    flush-on-checkpoint: true   # optional: disables flushing on checkpoint (see notes below!) ("true" by default)
    bulk-flush:
      max-actions: 42           # optional: maximum number of actions to buffer for each bulk request
      max-size: 42 mb           # optional: maximum size of buffered actions in bytes per bulk request
                                #   (only MB granularity is supported)
      interval: 60000           # optional: bulk flush interval (in milliseconds)
      back-off:                 # optional: backoff strategy ("disabled" by default)
        type: ...               #   valid strategies are "disabled", "constant", or "exponential"
        max-retries: 3          # optional: maximum number of retries
        delay: 30000            # optional: delay between each backoff attempt (in milliseconds)

    # optional: connection properties to be used during REST communication to Elasticsearch
    connection-max-retry-timeout: 3   # optional: maximum timeout (in milliseconds) between retries
    connection-path-prefix: "/v1"     # optional: prefix string to be added to every REST communication
    
    format:                     # required: Elasticsearch connector requires to specify a format,
      ...                       # currently only "json" format is supported.
                                # Please refer to Table Formats section for more details.
```
{% endtab %}
{% endtabs %}

**批量刷新：**有关可选刷新参数特征的更多信息，请参阅[相应的低级文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/elasticsearch.html)。

**禁用检查点上的刷新：**禁用时，接收器不会等待Elasticsearch在检查点上确认所有待处理的操作请求。因此，接收器不会为至少一次传递动作请求提供任何强有力的保证。

**关键字提取：** Flink自动从查询中提取有效关键字。例如，查询`SELECT a, b, c FROM t GROUP BY a, b`定义组合键的字段`a`和`b`。Elasticsearch连接器通过使用键分隔符连接查询中定义的顺序中的所有键字段，为每一行生成文档ID字符串。可以定义关键字段的空文字的自定义表示。

{% hint style="danger" %}
**注意** JSON格式定义了如何为外部系统编码文档，因此，必须将其添加为[依赖项](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html#formats)。
{% endhint %}

### HBase连接器

{% hint style="info" %}
来源：批处理   
接收器：批处理   
接收器：流式追加模式   
接收器：流式 追加模式  
临时连接：同步模式
{% endhint %}

HBase连接器允许读取和写入HBase群集。

连接器可以在[upsert模式下](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html#update-modes)运行，以使用[查询定义](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/dynamic_tables.html#table-to-stream-conversion)的[密钥](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/dynamic_tables.html#table-to-stream-conversion)与外部系统交换UPSERT / DELETE消息。

对于仅追加查询，连接器还可以在[追加模式下](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html#update-modes)操作，以仅与外部系统交换INSERT消息。

连接器可以定义如下：

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  hbase_rowkey_name rowkey_type,
  hbase_column_family_name1 ROW<...>,
  hbase_column_family_name2 ROW<...>
) WITH (
  'connector.type' = 'hbase', -- required: specify this table type is hbase
  
  'connector.version' = '1.4.3',          -- required: valid connector versions are "1.4.3"
  
  'connector.table-name' = 'hbase_table_name',  -- required: hbase table name
  
  'connector.zookeeper.quorum' = 'localhost:2181', -- required: HBase Zookeeper quorum configuration
  'connector.zookeeper.znode.parent' = '/test',    -- optional: the root dir in Zookeeper for HBase cluster.
                                                   -- The default value is "/hbase".

  'connector.write.buffer-flush.max-size' = '10mb', -- optional: writing option, determines how many size in memory of buffered
                                                    -- rows to insert per round trip. This can help performance on writing to JDBC
                                                    -- database. The default value is "2mb".

  'connector.write.buffer-flush.max-rows' = '1000', -- optional: writing option, determines how many rows to insert per round trip.
                                                    -- This can help performance on writing to JDBC database. No default value,
                                                    -- i.e. the default flushing is not depends on the number of buffered rows.

  'connector.write.buffer-flush.interval' = '2s',   -- optional: writing option, sets a flush interval flushing buffered requesting
                                                    -- if the interval passes, in milliseconds. Default value is "0s", which means
                                                    -- no asynchronous flush thread will be scheduled.
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.connect(
  new HBase()
    .version("1.4.3")                      // required: currently only support "1.4.3"
    .tableName("hbase_table_name")         // required: HBase table name
    .zookeeperQuorum("localhost:2181")     // required: HBase Zookeeper quorum configuration
    .zookeeperNodeParent("/test")          // optional: the root dir in Zookeeper for HBase cluster.
                                           // The default value is "/hbase".
    .writeBufferFlushMaxSize("10mb")       // optional: writing option, determines how many size in memory of buffered
                                           // rows to insert per round trip. This can help performance on writing to JDBC
                                           // database. The default value is "2mb".
    .writeBufferFlushMaxRows(1000)         // optional: writing option, determines how many rows to insert per round trip.
                                           // This can help performance on writing to JDBC database. No default value,
                                           // i.e. the default flushing is not depends on the number of buffered rows.
    .writeBufferFlushInterval("2s")        // optional: writing option, sets a flush interval flushing buffered requesting
                                           // if the interval passes, in milliseconds. Default value is "0s", which means
                                           // no asynchronous flush thread will be scheduled.
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: hbase
  version: "1.4.3"               # required: currently only support "1.4.3"
  
  table-name: "hbase_table_name" # required: HBase table name
  
  zookeeper:
    quorum: "localhost:2181"     # required: HBase Zookeeper quorum configuration
    znode.parent: "/test"        # optional: the root dir in Zookeeper for HBase cluster.
                                 # The default value is "/hbase".
  
  write.buffer-flush:
    max-size: "10mb"             # optional: writing option, determines how many size in memory of buffered
                                 # rows to insert per round trip. This can help performance on writing to JDBC
                                 # database. The default value is "2mb".
    max-rows: 1000               # optional: writing option, determines how many rows to insert per round trip.
                                 # This can help performance on writing to JDBC database. No default value,
                                 # i.e. the default flushing is not depends on the number of buffered rows.
    interval: "2s"               # optional: writing option, sets a flush interval flushing buffered requesting
                                 # if the interval passes, in milliseconds. Default value is "0s", which means
                                 # no asynchronous flush thread will be scheduled.
```
{% endtab %}
{% endtabs %}

### JDBC连接器

{% hint style="info" %}
来源：批处理   
接收器：批处理   
接收器：流式追加模式   
接收器：流式 追加模式  
临时连接：同步模式
{% endhint %}

**支持的驱动**

| Name | Group Id | Artifact Id | JAR |
| :--- | :--- | :--- | :--- |
| MySQL | mysql | mysql-connector-java | [Download](https://repo.maven.apache.org/maven2/mysql/mysql-connector-java/) |
| PostgreSQL | org.postgresql | postgresql | [Download](https://jdbc.postgresql.org/download.html) |
| Derby | org.apache.derby | derby | [Download](http://db.apache.org/derby/derby_downloads.html) |

连接器可以定义如下：

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  ...
) WITH (
  'connector.type' = 'jdbc', -- required: specify this table type is jdbc
  
  'connector.url' = 'jdbc:mysql://localhost:3306/flink-test', -- required: JDBC DB url
  
  'connector.table' = 'jdbc_table_name',  -- required: jdbc table name
  
  'connector.driver' = 'com.mysql.jdbc.Driver', -- optional: the class name of the JDBC driver to use to connect to this URL. 
                                                -- If not set, it will automatically be derived from the URL.

  'connector.username' = 'name', -- optional: jdbc user name and password
  'connector.password' = 'password',
  
  -- scan options, optional, used when reading from table

  -- These options must all be specified if any of them is specified. In addition, partition.num must be specified. They
  -- describe how to partition the table when reading in parallel from multiple tasks. partition.column must be a numeric,
  -- date, or timestamp column from the table in question. Notice that lowerBound and upperBound are just used to decide
  -- the partition stride, not for filtering the rows in table. So all rows in the table will be partitioned and returned.
  -- This option applies only to reading.
  'connector.read.partition.column' = 'column_name', -- optional, name of the column used for partitioning the input.
  'connector.read.partition.num' = '50', -- optional, the number of partitions.
  'connector.read.partition.lower-bound' = '500', -- optional, the smallest value of the first partition.
  'connector.read.partition.upper-bound' = '1000', -- optional, the largest value of the last partition.
  
  'connector.read.fetch-size' = '100', -- optional, Gives the reader a hint as to the number of rows that should be fetched
                                       -- from the database when reading per round trip. If the value specified is zero, then
                                       -- the hint is ignored. The default value is zero.

  -- lookup options, optional, used in temporary join
  'connector.lookup.cache.max-rows' = '5000', -- optional, max number of rows of lookup cache, over this value, the oldest rows will
                                              -- be eliminated. "cache.max-rows" and "cache.ttl" options must all be specified if any
                                              -- of them is specified. Cache is not enabled as default.
  'connector.lookup.cache.ttl' = '10s', -- optional, the max time to live for each rows in lookup cache, over this time, the oldest rows
                                        -- will be expired. "cache.max-rows" and "cache.ttl" options must all be specified if any of
                                        -- them is specified. Cache is not enabled as default.
  'connector.lookup.max-retries' = '3', -- optional, max retry times if lookup database failed

  -- sink options, optional, used when writing into table
  'connector.write.flush.max-rows' = '5000', -- optional, flush max size (includes all append, upsert and delete records), 
                                             -- over this number of records, will flush data. The default value is "5000".
  'connector.write.flush.interval' = '2s', -- optional, flush interval mills, over this time, asynchronous threads will flush data.
                                           -- The default value is "0s", which means no asynchronous flush thread will be scheduled. 
  'connector.write.max-retries' = '3' -- optional, max retry times if writing records to database failed
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: jdbc
  url: "jdbc:mysql://localhost:3306/flink-test"     # required: JDBC DB url
  table: "jdbc_table_name"        # required: jdbc table name
  driver: "com.mysql.jdbc.Driver" # optional: the class name of the JDBC driver to use to connect to this URL.
                                  # If not set, it will automatically be derived from the URL.

  username: "name"                # optional: jdbc user name and password
  password: "password"
  
  read: # scan options, optional, used when reading from table
    partition: # These options must all be specified if any of them is specified. In addition, partition.num must be specified. They
               # describe how to partition the table when reading in parallel from multiple tasks. partition.column must be a numeric,
               # date, or timestamp column from the table in question. Notice that lowerBound and upperBound are just used to decide
               # the partition stride, not for filtering the rows in table. So all rows in the table will be partitioned and returned.
               # This option applies only to reading.
      column: "column_name" # optional, name of the column used for partitioning the input.
      num: 50               # optional, the number of partitions.
      lower-bound: 500      # optional, the smallest value of the first partition.
      upper-bound: 1000     # optional, the largest value of the last partition.
    fetch-size: 100         # optional, Gives the reader a hint as to the number of rows that should be fetched
                            # from the database when reading per round trip. If the value specified is zero, then
                            # the hint is ignored. The default value is zero.
  
  lookup: # lookup options, optional, used in temporary join
    cache:
      max-rows: 5000 # optional, max number of rows of lookup cache, over this value, the oldest rows will
                     # be eliminated. "cache.max-rows" and "cache.ttl" options must all be specified if any
                     # of them is specified. Cache is not enabled as default.
      ttl: "10s"     # optional, the max time to live for each rows in lookup cache, over this time, the oldest rows
                     # will be expired. "cache.max-rows" and "cache.ttl" options must all be specified if any of
                     # them is specified. Cache is not enabled as default.
    max-retries: 3   # optional, max retry times if lookup database failed
  
  write: # sink options, optional, used when writing into table
      flush:
        max-rows: 5000 # optional, flush max size (includes all append, upsert and delete records), 
                       # over this number of records, will flush data. The default value is "5000".
        interval: "2s" # optional, flush interval mills, over this time, asynchronous threads will flush data.
                       # The default value is "0s", which means no asynchronous flush thread will be scheduled. 
      max-retries: 3   # optional, max retry times if writing records to database failed.
```
{% endtab %}
{% endtabs %}

### Hive连接器

{% hint style="info" %}
来源：批次   
接收器：批次
{% endhint %}

请参考[Hive集成](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/)。

## 表格式

Flink提供了一组可与表连接器一起使用的表格式。

格式标记表示与连接器匹配的格式类型。

### CSV格式

{% hint style="info" %}
格式：序列化模式   
格式：反序列化模式
{% endhint %}

CSV格式允许读取和写入以逗号分隔的行。

{% tabs %}
{% tab title="DDL" %}
```sql
CREATE TABLE MyUserTable (
  ...
) WITH (
  'format.type' = 'csv',                  -- required: specify the schema type

  'format.fields.0.name' = 'lon',         -- optional: define the schema explicitly using type information.
  'format.fields.0.data-type' = 'FLOAT',  -- This overrides default behavior that uses table's schema as format schema.
  'format.fields.1.name' = 'rideTime',
  'format.fields.1.data-type' = 'TIMESTAMP(3)',

  'format.field-delimiter' = ';',         -- optional: field delimiter character (',' by default)
  'format.line-delimiter' = U&'\000D\000A',  -- optional: line delimiter ("\n" by default; otherwise
                                             -- "\r" or "\r\n" are allowed), unicode is supported if the delimiter
                                             -- is an invisible special character,
                                             -- e.g. U&'\000D' is the unicode representation of carriage return "\r"
                                             -- e.g. U&'\000A' is the unicode representation of line feed "\n"
  'format.quote-character' = '''',        -- optional: quote character for enclosing field values ('"' by default)
  'format.allow-comments' = 'true',       -- optional: ignores comment lines that start with "#"
                                          -- (disabled by default);
                                          -- if enabled, make sure to also ignore parse errors to allow empty rows
  'format.ignore-parse-errors' = 'true',  -- optional: skip fields and rows with parse errors instead of failing;
                                          -- fields are set to null in case of errors
  'format.array-element-delimiter' = '|', -- optional: the array element delimiter string for separating
                                          -- array and row element values (";" by default)
  'format.escape-character' = '\\',       -- optional: escape character for escaping values (disabled by default)
  'format.null-literal' = 'n/a'           -- optional: null literal string that is interpreted as a
                                          -- null value (disabled by default)
)
```
{% endtab %}

{% tab title="Java/Scala" %}
```java
.withFormat(
  new Csv()

    // optional: define the schema explicitly using type information. This overrides default
    // behavior that uses table's schema as format schema.
    .schema(Type.ROW(...))

    .fieldDelimiter(';')         // optional: field delimiter character (',' by default)
    .lineDelimiter("\r\n")       // optional: line delimiter ("\n" by default;
                                 //   otherwise "\r", "\r\n", or "" are allowed)
    .quoteCharacter('\'')        // optional: quote character for enclosing field values ('"' by default)
    .allowComments()             // optional: ignores comment lines that start with '#' (disabled by default);
                                 //   if enabled, make sure to also ignore parse errors to allow empty rows
    .ignoreParseErrors()         // optional: skip fields and rows with parse errors instead of failing;
                                 //   fields are set to null in case of errors
    .arrayElementDelimiter("|")  // optional: the array element delimiter string for separating
                                 //   array and row element values (";" by default)
    .escapeCharacter('\\')       // optional: escape character for escaping values (disabled by default)
    .nullLiteral("n/a")          // optional: null literal string that is interpreted as a
                                 //   null value (disabled by default)
)
```
{% endtab %}

{% tab title="Python" %}
```python
.with_format(
    Csv()

    # optional: define the schema explicitly using type information. This overrides default
    # behavior that uses table's schema as format schema.
    .schema(DataTypes.ROW(...))

    .field_delimiter(';')          # optional: field delimiter character (',' by default)
    .line_delimiter("\r\n")        # optional: line delimiter ("\n" by default;
                                   #   otherwise "\r", "\r\n", or "" are allowed)
    .quote_character('\'')         # optional: quote character for enclosing field values ('"' by default)
    .allow_comments()              # optional: ignores comment lines that start with '#' (disabled by default);
                                   #   if enabled, make sure to also ignore parse errors to allow empty rows
    .ignore_parse_errors()         # optional: skip fields and rows with parse errors instead of failing;
                                   #   fields are set to null in case of errors
    .array_element_delimiter("|")  # optional: the array element delimiter string for separating
                                   #   array and row element values (";" by default)
    .escape_character('\\')        # optional: escape character for escaping values (disabled by default)
    .null_literal("n/a")           # optional: null literal string that is interpreted as a
                                   #   null value (disabled by default)
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
format:
  type: csv

  # optional: define the schema explicitly using type information. This overrides default
  # behavior that uses table's schema as format schema.
  schema: "ROW(lon FLOAT, rideTime TIMESTAMP)"

  field-delimiter: ";"         # optional: field delimiter character (',' by default)
  line-delimiter: "\r\n"       # optional: line delimiter ("\n" by default;
                               #   otherwise "\r", "\r\n", or "" are allowed)
  quote-character: "'"         # optional: quote character for enclosing field values ('"' by default)
  allow-comments: true         # optional: ignores comment lines that start with "#" (disabled by default);
                               #   if enabled, make sure to also ignore parse errors to allow empty rows
  ignore-parse-errors: true    # optional: skip fields and rows with parse errors instead of failing;
                               #   fields are set to null in case of errors
  array-element-delimiter: "|" # optional: the array element delimiter string for separating
                               #   array and row element values (";" by default)
  escape-character: "\\"       # optional: escape character for escaping values (disabled by default)
  null-literal: "n/a"          # optional: null literal string that is interpreted as a
                               #   null value (disabled by default)
```
{% endtab %}
{% endtabs %}

下表列出了可以读取和写入的受支持类型：

| 支持的Flink SQL类型 |
| :--- |
| `ROW` |
| `VARCHAR` |
| `ARRAY[_]` |
| `INT` |
| `BIGINT` |
| `FLOAT` |
| `DOUBLE` |
| `BOOLEAN` |
| `DATE` |
| `TIME` |
| `TIMESTAMP` |
| `DECIMAL` |
| `NULL` （尚不支持） |

### JSON格式

{% hint style="info" %}
**格式：**序列化架构   
**格式：**反序列化架构
{% endhint %}

JSON格式允许读写与给定格式模式对应的JSON数据。格式模式可以定义为Flink类型、JSON模式，也可以从所需的表模式派生。Flink类型支持更类似SQL的定义，并映射到相应的SQL数据类型。JSON模式允许更复杂和嵌套的结构。

如果格式模式等于表模式，还可以自动派生模式。这只允许定义模式信息一次。格式的名称、类型和字段顺序由表的模式决定。如果时间属性的原点不是字段，则忽略它们。表模式中的from定义被解释为按格式重命名的字段。

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withFormat(
  new Json()
    .failOnMissingField(true)   // optional: flag whether to fail if a field is missing or not, false by default

    // required: define the schema either by using type information which parses numbers to corresponding types
    .schema(Type.ROW(...))

    // or by using a JSON schema which parses to DECIMAL and TIMESTAMP
    .jsonSchema(
      "{" +
      "  type: 'object'," +
      "  properties: {" +
      "    lon: {" +
      "      type: 'number'" +
      "    }," +
      "    rideTime: {" +
      "      type: 'string'," +
      "      format: 'date-time'" +
      "    }" +
      "  }" +
      "}"
    )

    // or use the table's schema
    .deriveSchema()
)

```
{% endtab %}

{% tab title="YAML" %}
```yaml
format:
  type: json
  fail-on-missing-field: true   # optional: flag whether to fail if a field is missing or not, false by default

  # required: define the schema either by using a type string which parses numbers to corresponding types
  schema: "ROW(lon FLOAT, rideTime TIMESTAMP)"

  # or by using a JSON schema which parses to DECIMAL and TIMESTAMP
  json-schema: >
    {
      type: 'object',
      properties: {
        lon: {
          type: 'number'
        },
        rideTime: {
          type: 'string',
          format: 'date-time'
        }
      }
    }

  # or use the table's schema
  derive-schema: true
```
{% endtab %}
{% endtabs %}

下表展示了JSON模式类型到Flink SQL类型的映射：

| JSON模式 | Flink SQL |
| :--- | :--- |
| `object` | `ROW` |
| `boolean` | `BOOLEAN` |
| `array` | `ARRAY[_]` |
| `number` | `DECIMAL` |
| `integer` | `DECIMAL` |
| `string` | `VARCHAR` |
| `带格式的字符串: date-time` | `TIMESTAMP` |
| `带格式的字符串: date` | `DATE` |
| `带格式的字符串: time` | `TIME` |
| `字符串编码: base64` | `ARRAY[TINYINT]` |
| `null` | `NULL` （尚不支持） |

目前，Flink仅支持[JSON模式规范](http://json-schema.org/)draft-07的子集。 联盟类型（以及allOf，anyOf，not）尚不支持。 oneOf和类型数组仅支持指定为空性。

支持链接到文档中的通用定义的简单引用，如下面更复杂的示例所示：

```scheme
{
  "definitions": {
    "address": {
      "type": "object",
      "properties": {
        "street_address": {
          "type": "string"
        },
        "city": {
          "type": "string"
        },
        "state": {
          "type": "string"
        }
      },
      "required": [
        "street_address",
        "city",
        "state"
      ]
    }
  },
  "type": "object",
  "properties": {
    "billing_address": {
      "$ref": "#/definitions/address"
    },
    "shipping_address": {
      "$ref": "#/definitions/address"
    },
    "optional_address": {
      "oneOf": [
        {
          "type": "null"
        },
        {
          "$ref": "#/definitions/address"
        }
      ]
    }
  }
}
```

缺少字段处理:默认情况下，缺少的JSON字段被设置为null。您可以启用严格的JSON解析，如果缺少字段，它将取消源\(和查询\)。

确保将JSON格式添加为依赖项。

### Apache Avro格式

{% hint style="info" %}
**格式：**序列化架构   
**格式：**反序列化架构
{% endhint %}

[Apache的Avro](https://avro.apache.org/)格式允许读取和写入对应于给定的Avro格式模式的数据。格式模式可以定义为Avro特定记录的完全限定类名，也可以定义为Avro架构字符串。如果使用类名，则在运行时期间类必须在类路径中可用。

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withFormat(
  new Avro()

    // required: define the schema either by using an Avro specific record class
    .recordClass(User.class)

    // or by using an Avro schema
    .avroSchema(
      "{" +
      "  \"type\": \"record\"," +
      "  \"name\": \"test\"," +
      "  \"fields\" : [" +
      "    {\"name\": \"a\", \"type\": \"long\"}," +
      "    {\"name\": \"b\", \"type\": \"string\"}" +
      "  ]" +
      "}"
    )
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
format:
  type: avro

  # required: define the schema either by using an Avro specific record class
  record-class: "org.organization.types.User"

  # or by using an Avro schema
  avro-schema: >
    {
      "type": "record",
      "name": "test",
      "fields" : [
        {"name": "a", "type": "long"},
        {"name": "b", "type": "string"}
      ]
    }
```
{% endtab %}
{% endtabs %}

Avro类型映射到相应的SQL数据类型。仅支持联合类型以指定可为空性，否则它们将转换为`ANY`类型。下表展示了映射关系：

| Avro架构 | Flink SQL |
| :--- | :--- |
| `record` | `ROW` |
| `enum` | `VARCHAR` |
| `array` | `ARRAY[_]` |
| `map` | `MAP[VARCHAR, _]` |
| `union` | 非null类型或 `ANY` |
| `fixed` | `ARRAY[TINYINT]` |
| `string` | `VARCHAR` |
| `bytes` | `ARRAY[TINYINT]` |
| `int` | `INT` |
| `long` | `BIGINT` |
| `float` | `FLOAT` |
| `double` | `DOUBLE` |
| `boolean` | `BOOLEAN` |
| `int with logicalType: date` | `DATE` |
| `int with logicalType: time-millis` | `TIME` |
| `int with logicalType: time-micros` | `INT` |
| `long with logicalType: timestamp-millis` | `TIMESTAMP` |
| `long with logicalType: timestamp-micros` | `BIGINT` |
| `bytes with logicalType: decimal` | `DECIMAL` |
| `fixed with logicalType: decimal` | `DECIMAL` |
| `null` | `NULL` （尚不支持） |

Avro使用[Joda-Time](http://www.joda.org/joda-time/)来表示特定记录类中的逻辑日期和时间类型。Joda-Time依赖不是Flink分发的一部分。因此，请确保Joda-Time在运行时期间与您的特定记录类一起位于类路径中。通过模式字符串指定的Avro格式不需要Joda-Time。

确保添加Apache Avro依赖项。

## 深层的TableSources和TableSinks

尚未将以下表源和接收器迁移（或尚未完全迁移）到新的统一接口。

这些是`TableSource`Flink提供的附加功能：

| **类名称** | **Maven依赖** | **批量？** | **流？** | **描述** |
| :--- | :--- | :--- | :--- | :--- |
| `OrcTableSource` | `flink-orc` | ÿ | ñ | 一个`TableSource`ORC文件。 |

这些是`TableSink`Flink提供的附加功能：

| **类名称** | **Maven依赖** | **批量？** | **流？** | **描述** |
| :--- | :--- | :--- | :--- | :--- |
| `CsvTableSink` | `flink-table` | ÿ | 附加 | CSV文件的简单接收器。 |
| `JDBCAppendTableSink` | `flink-jdbc` | ÿ | 附加 | 将表写入JDBC表。 |
| `CassandraAppendTableSink` | `flink-connector-cassandra` | ñ | 附加 | 将表写入Cassandra表。 |

### OrcTableSource

OrcTableSource读取[ORC文件](https://orc.apache.org/)。 ORC是结构化数据的文件格式，并以压缩的列式表示形式存储数据。 ORC非常高效，支持投影和滤波器下推。

创建OrcTableSource，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
// create Hadoop Configuration
Configuration config = new Configuration();

OrcTableSource orcTableSource = OrcTableSource.builder()
  // path to ORC file(s). NOTE: By default, directories are recursively scanned.
  .path("file:///path/to/data")
  // schema of ORC files
  .forOrcSchema("struct<name:string,addresses:array<struct<street:string,zip:smallint>>>")
  // Hadoop configuration
  .withConfiguration(config)
  // build OrcTableSource
  .build();

```
{% endtab %}

{% tab title="Scala" %}
```scala
// create Hadoop Configuration
val config = new Configuration()

val orcTableSource = OrcTableSource.builder()
  // path to ORC file(s). NOTE: By default, directories are recursively scanned.
  .path("file:///path/to/data")
  // schema of ORC files
  .forOrcSchema("struct<name:string,addresses:array<struct<street:string,zip:smallint>>>")
  // Hadoop configuration
  .withConfiguration(config)
  // build OrcTableSource
  .build()

```
{% endtab %}
{% endtabs %}

注意：OrcTableSource尚不支持ORC的Union类型。

### CsvTableSink

CsvTableSink向一个或多个CSV文件发出表。

接收器仅支持仅附加流表。 它不能用于发出不断更新的表。 有关详细信息，请参阅[表到流转换的文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#table-to-stream-conversion)。 发出流表时，行至少写入一次（如果启用了检查点），并且CsvTableSink不会将输出文件拆分为存储桶文件，而是连续写入相同的文件。

{% tabs %}
{% tab title="Java" %}
```java
CsvTableSink sink = new CsvTableSink(
    path,                  // output path
    "|",                   // optional: delimit files by '|'
    1,                     // optional: write to a single file
    WriteMode.OVERWRITE);  // optional: override existing files

tableEnv.registerTableSink(
  "csvOutputTable",
  // specify table schema
  new String[]{"f0", "f1"},
  new TypeInformation[]{Types.STRING, Types.INT},
  sink);

Table table = ...
table.insertInto("csvOutputTable");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val sink: CsvTableSink = new CsvTableSink(
    path,                             // output path
    fieldDelim = "|",                 // optional: delimit files by '|'
    numFiles = 1,                     // optional: write to a single file
    writeMode = WriteMode.OVERWRITE)  // optional: override existing files

tableEnv.registerTableSink(
  "csvOutputTable",
  // specify table schema
  Array[String]("f0", "f1"),
  Array[TypeInformation[_]](Types.STRING, Types.INT),
  sink)

val table: Table = ???
table.insertInto("csvOutputTable")
```
{% endtab %}
{% endtabs %}

### JDBCAppendTableSink

`JDBCAppendTableSink`将表发送到JDBC连接。 接收器仅支持仅附加流表。 它不能用于发出不断更新的表。 有关详细信息，请参阅[表到流转换的文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#table-to-stream-conversion)。

`JDBCAppendTableSink`将每个Table行至少插入一次数据库表（如果启用了检查点）。 但是，您可以使用`REPLACE`或`INSERT OVERWRITE`指定插入查询以执行对数据库的`upsert`写入。

要使用`JDBC`接收器，必须将JDBC连接器依赖项（`flink-jdbc`）添加到项目中。 然后，您可以使用`JDBCAppendSinkBuilder`创建接收器：

{% tabs %}
{% tab title="Java" %}
```java
JDBCAppendTableSink sink = JDBCAppendTableSink.builder()
  .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")
  .setDBUrl("jdbc:derby:memory:ebookshop")
  .setQuery("INSERT INTO books (id) VALUES (?)")
  .setParameterTypes(INT_TYPE_INFO)
  .build();

tableEnv.registerTableSink(
  "jdbcOutputTable",
  // specify table schema
  new String[]{"id"},
  new TypeInformation[]{Types.INT},
  sink);

Table table = ...
table.insertInto("jdbcOutputTable");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val sink: JDBCAppendTableSink = JDBCAppendTableSink.builder()
  .setDrivername("org.apache.derby.jdbc.EmbeddedDriver")
  .setDBUrl("jdbc:derby:memory:ebookshop")
  .setQuery("INSERT INTO books (id) VALUES (?)")
  .setParameterTypes(INT_TYPE_INFO)
  .build()

tableEnv.registerTableSink(
  "jdbcOutputTable",
  // specify table schema
  Array[String]("id"),
  Array[TypeInformation[_]](Types.INT),
  sink)

val table: Table = ???
table.insertInto("jdbcOutputTable")
```
{% endtab %}
{% endtabs %}

与使用`JDBCOutputFormat`类似，必须显式指定JDBC驱动程序的名称，JDBC URL，要执行的查询以及JDBC表的字段类型。

### CassandraAppendTableSink

`CassandraAppendTableSink`向`Cassandra`表发出一个表。 接收器仅支持仅附加流表。 它不能用于发出不断更新的表。 有关详细信息，请参阅[表到流转换的文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#table-to-stream-conversion)。

如果启用了检查点，`CassandraAppendTableSink`会将所有行至少插入一次`Cassandra`表中。 但是，您可以将查询指定为`upsert`查询。

要使用`CassandraAppendTableSink`，必须将`Cassandra`连接器依赖项（`flink-connector-cassandra`）添加到项目中。 下面的示例展示示了如何使用`CassandraAppendTableSink`。

{% tabs %}
{% tab title="Java" %}
```java
ClusterBuilder builder = ... // configure Cassandra cluster connection

CassandraAppendTableSink sink = new CassandraAppendTableSink(
  builder,
  // the query must match the schema of the table
  "INSERT INTO flink.myTable (id, name, value) VALUES (?, ?, ?)");

tableEnv.registerTableSink(
  "cassandraOutputTable",
  // specify table schema
  new String[]{"id", "name", "value"},
  new TypeInformation[]{Types.INT, Types.STRING, Types.DOUBLE},
  sink);

Table table = ...
table.insertInto(cassandraOutputTable);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val builder: ClusterBuilder = ... // configure Cassandra cluster connection

val sink: CassandraAppendTableSink = new CassandraAppendTableSink(
  builder,
  // the query must match the schema of the table
  "INSERT INTO flink.myTable (id, name, value) VALUES (?, ?, ?)")

tableEnv.registerTableSink(
  "cassandraOutputTable",
  // specify table schema
  Array[String]("id", "name", "value"),
  Array[TypeInformation[_]](Types.INT, Types.STRING, Types.DOUBLE),
  sink)

val table: Table = ???
table.insertInto(cassandraOutputTable)
```
{% endtab %}
{% endtabs %}

