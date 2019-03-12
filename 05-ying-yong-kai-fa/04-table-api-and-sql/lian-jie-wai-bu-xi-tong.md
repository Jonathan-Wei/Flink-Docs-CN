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
| Elasticsearch | 6 | `flink-connector-elasticsearch6` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-elasticsearch6_2.11/1.7.2/flink-connector-elasticsearch6_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.8 | `flink-connector-kafka-0.8` | Not available |
| Apache Kafka | 0.9 | `flink-connector-kafka-0.9` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.9_2.11/1.7.2/flink-connector-kafka-0.9_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.10 | `flink-connector-kafka-0.10` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.10_2.11/1.7.2/flink-connector-kafka-0.10_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.11 | `flink-connector-kafka-0.11` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka-0.11_2.11/1.7.2/flink-connector-kafka-0.11_2.11-1.7.2-sql-jar.jar) |
| Apache Kafka | 0.11+ \(`universal`\) | `flink-connector-kafka` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-connector-kafka_2.11/1.7.2/flink-connector-kafka_2.11-1.7.2-sql-jar.jar) |

### 格式

| 名称 | Maven依赖 | SQL Client JAR |
| :--- | :--- | :--- |
| CSV | Built-in | Built-in |
| JSON | `flink-json` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-json/1.7.2/flink-json-1.7.2-sql-jar.jar) |
| Apache Avro | `flink-avro` | [Download](http://central.maven.org/maven2/org/apache/flink/flink-avro/1.7.2/flink-avro-1.7.2-sql-jar.jar) |

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
{% tab title="Java/Scala" %}
```java
tableEnvironment
  .connect(...)
  .withFormat(...)
  .withSchema(...)
  .inAppendMode()
  .registerTableSource("MyTable")
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

对于流式查询，更新模式声明如何在动态表和存储系统之间进行通信以进行连续查询。

以下代码显示了如何连接到Kafka以读取Avro记录的完整示例。

{% tabs %}
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
        "}" +
      )
  )

  // declare the schema of the table
  .withSchema(
    new Schema()
      .field("rowtime", Types.SQL_TIMESTAMP)
        .rowtime(new Rowtime()
          .timestampsFromField("timestamp")
          .watermarksPeriodicBounded(60000)
        )
      .field("user", Types.LONG)
      .field("message", Types.STRING)
  )

  // specify the update-mode for streaming tables
  .inAppendMode()

  // register as source, sink, or both and under a name
  .registerTableSource("MyUserTable");
```
{% endtab %}

{% tab title="YAML" %}
```yaml
tables:
  - name: MyUserTable      # name the new table
    type: source           # declare if the table should be "source", "sink", or "both"
    update-mode: append    # specify the update-mode for streaming tables

    # declare the external system to connect to
    connector:
      type: kafka
      version: "0.10"
      topic: test-input
      startup-mode: earliest-offset
      properties:
        - key: zookeeper.connect
          value: localhost:2181
        - key: bootstrap.servers
          value: localhost:9092

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
        type: TIMESTAMP
        rowtime:
          timestamps:
            type: from-field
            from: ts
          watermarks:
            type: periodic-bounded
            delay: "60000"
      - name: user
        type: BIGINT
      - name: message
        type: VARCHAR
```
{% endtab %}
{% endtabs %}

在两种方式中，所需的连接属性都转换为规范化的，基于字符串的键值对。 所谓的表工厂从键值对创建配置的表源，表接收器和相应的格式。 在搜索完全匹配的表工厂时，会考虑通过Java的服务提供程序接口（SPI）找到的所有表工厂。

如果找不到工厂或多个工厂匹配给定的属性，则会抛出一个异常，其中包含有关已考虑的工厂和支持的属性的其他信息。

## 表格式

表模式定义类的名称和类型，类似于`SQL CREATE TABLE`语句的列定义。 此外，可以指定列的表示方式和表格数据编码格式的字段。 如果列的名称应与输入/输出格式不同，则字段的来源可能很重要。 例如，列`user_name`应从JSON格式引用字段`$$-user-name`。 此外，需要模式将类型从外部系统映射到Flink的表示。 对于表接收器，它确保仅将具有有效模式的数据写入外部系统。

以下示例展示了没有时间属性的简单模式以及输入/输出到表列的一对一字段映射。

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withSchema(
  new Schema()
    .field("MyField1", Types.INT)     // required: specify the fields of the table (in this order)
    .field("MyField2", Types.STRING)
    .field("MyField3", Types.BOOLEAN)
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
schema:
  - name: MyField1    # required: specify the fields of the table (in this order)
    type: INT
  - name: MyField2
    type: VARCHAR
  - name: MyField3
    type: BOOLEAN
```
{% endtab %}
{% endtabs %}

对于每个字段，除了列的名称和类型之外，还可以声明以下属性：

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withSchema(
  new Schema()
    .field("MyField1", Types.SQL_TIMESTAMP)
      .proctime()      // optional: declares this field as a processing-time attribute
    .field("MyField2", Types.SQL_TIMESTAMP)
      .rowtime(...)    // optional: declares this field as a event-time attribute
    .field("MyField3", Types.BOOLEAN)
      .from("mf3")     // optional: original field in the input that is referenced/aliased by this field
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
schema:
  - name: MyField1
    type: TIMESTAMP
    proctime: true    # optional: boolean flag whether this field should be a processing-time attribute
  - name: MyField2
    type: TIMESTAMP
    rowtime: ...      # optional: wether this field should be a event-time attribute
  - name: MyField3
    type: BOOLEAN
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

{% tab title="YAML" %}
```text
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

由于类型信息仅在编程语言中可用，因此支持在YAML文件中定义以下类型字符串：

```yaml
VARCHAR
BOOLEAN
TINYINT
SMALLINT
INT
BIGINT
FLOAT
DOUBLE
DECIMAL
DATE
TIME
TIMESTAMP
MAP<fieldtype, fieldtype>        # generic map; e.g. MAP<VARCHAR, INT> that is mapped to Flink's MapTypeInfo
MULTISET<fieldtype>              # multiset; e.g. MULTISET<VARCHAR> that is mapped to Flink's MultisetTypeInfo
PRIMITIVE_ARRAY<fieldtype>       # primitive array; e.g. PRIMITIVE_ARRAY<INT> that is mapped to Flink's PrimitiveArrayTypeInfo
OBJECT_ARRAY<fieldtype>          # object array; e.g. OBJECT_ARRAY<POJO(org.mycompany.MyPojoClass)> that is mapped to
                                 #   Flink's ObjectArrayTypeInfo
ROW<fieldtype, ...>              # unnamed row; e.g. ROW<VARCHAR, INT> that is mapped to Flink's RowTypeInfo
                                 #   with indexed fields names f0, f1, ...
ROW<fieldname fieldtype, ...>    # named row; e.g., ROW<myField VARCHAR, myOtherField INT> that
                                 #   is mapped to Flink's RowTypeInfo
POJO<class>                      # e.g., POJO<org.mycompany.MyPojoClass> that is mapped to Flink's PojoTypeInfo
ANY<class>                       # e.g., ANY<org.mycompany.MyClass> that is mapped to Flink's GenericTypeInfo
ANY<class, serialized>           # used for type information that is not supported by Flink's Table & SQL API
```

## 更新模式

对于流式查询，需要声明如何[在动态表和外部连接器](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#continuous-queries)之间执行[转换](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/dynamic_tables.html#continuous-queries)。更新模式指定应与外部系统交换哪种消息：

**追加模式：**在追加模式下，动态表和外部连接器仅交换`INSERT`消息。

**回退模式：**在回退模式下，动态表和外部连接器交换`ADD`和`RETRACT`消息。 `INSERT`更改被编码为`ADD`消息，`DELETE`更改被编码为`RETRACT`消息，`UPDATE`更改被编码为更新（前一个）行的`RETRACT`消息和更新（新）行的`ADD`消息。在此模式下，不能定义键而不是`upsert`模式。但是，每次更新都包含两个效率较低的消息。

**Upsert模式：**在`upsert`模式下，动态表和外部连接器交换`UPSERT`和`DELETE`消息。此模式需要一个（可能是复合的）唯一密钥，通过该密钥可以传播更新。外部连接器需要知道唯一键属性才能正确应用消息。 `INSERT`和`UPDATE`更改被编码为`UPSERT`消息。 `DELETE`更改为`DELETE`消息。与回退流的主要区别在于`UPDATE`更改使用单个消息进行编码，因此更有效。

{% hint style="danger" %}
**注意：**每个连接器的文档都说明了支持哪些更新模式。
{% endhint %}

{% tabs %}
{% tab title="Java/Scala" %}
```java
.connect(...)
  .inAppendMode()    // otherwise: inUpsertMode() or inRetractMode()
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
**格式：**仅限CSV
{% endhint %}

文件系统连接器允许从本地或分布式文件系统进行读写。文件系统可以定义为：

{% tabs %}
{% tab title="Java/Scala" %}
```java
.connect(
  new FileSystem()
    .path("file:///path/to/whatever")    // required: path to a file or directory
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: filesystem
  path: "file:///path/to/whatever"    # required: path to a file or directory
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
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: kafka
  version: "0.11"     # required: valid connector versions are
                      #   "0.8", "0.9", "0.10", "0.11", and "universal"
  topic: ...          # required: topic name from which the table is read

  properties:         # optional: connector specific properties
    - key: zookeeper.connect
      value: localhost:2181
    - key: bootstrap.servers
      value: localhost:9092
    - key: group.id
      value: testGroup

  startup-mode: ...   # optional: valid modes are "earliest-offset", "latest-offset",
                      # "group-offsets", or "specific-offsets"
  specific-offsets:   # optional: used in case of startup mode with specific offsets
    - partition: 0
      offset: 42
    - partition: 1
      offset: 300

  sink-partitioner: ...    # optional: output partitioning from Flink's partitions into Kafka's partitions
                           # valid are "fixed" (each Flink partition ends up in at most one Kafka partition),
                           # "round-robin" (a Flink partition is distributed to Kafka partitions round-robin)
                           # "custom" (use a custom FlinkKafkaPartitioner subclass)
  sink-partitioner-class: org.mycompany.MyPartitioner  # optional: used in case of sink partitioner custom
```
{% endtab %}
{% endtabs %}

### Elasticsearch连接器

{% hint style="info" %}
**接收器：**流式附加模式   
**接收器：**流式Upsert模式   
**格式：**仅限JSON
{% endhint %}

{% tabs %}
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
```
{% endtab %}

{% tab title="YAML" %}
```yaml
connector:
  type: elasticsearch
  version: 6                # required: valid connector versions are "6"
    hosts:                  # required: one or more Elasticsearch hosts to connect to
      - hostname: "localhost"
        port: 9200
        protocol: "http"
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
```
{% endtab %}
{% endtabs %}

## 表格式

Flink提供了一组可与表连接器一起使用的表格式。

格式标记表示与连接器匹配的格式类型。

### CSV格式

CSV格式允许读取和写入以逗号分隔的行。

{% tabs %}
{% tab title="Java/Scala" %}
```java
.withFormat(
  new Csv()
    .field("field1", Types.STRING)    // required: ordered format fields
    .field("field2", Types.TIMESTAMP)
    .fieldDelimiter(",")              // optional: string delimiter "," by default
    .lineDelimiter("\n")              // optional: string delimiter "\n" by default
    .quoteCharacter('"')              // optional: single character for string values, empty by default
    .commentPrefix('#')               // optional: string to indicate comments, empty by default
    .ignoreFirstLine()                // optional: ignore the first line, by default it is not skipped
    .ignoreParseErrors()              // optional: skip records with parse error instead of failing by default
)
```
{% endtab %}

{% tab title="YAML" %}
```yaml
format:
  type: csv
  fields:                    # required: ordered format fields
    - name: field1
      type: VARCHAR
    - name: field2
      type: TIMESTAMP
  field-delimiter: ","       # optional: string delimiter "," by default
  line-delimiter: "\n"       # optional: string delimiter "\n" by default
  quote-character: '"'       # optional: single character for string values, empty by default
  comment-prefix: '#'        # optional: string to indicate comments, empty by default
  ignore-first-line: false   # optional: boolean flag to ignore the first line, by default it is not skipped
  ignore-parse-errors: true  # optional: skip records with parse error instead of failing by default
```
{% endtab %}
{% endtabs %}

CSV格式包含在Flink中，不需要其他依赖项。

{% hint style="danger" %}
**注意：**目前写入行的CSV格式有限。仅支持自定义字段分隔符作为可选参数。
{% endhint %}

### JSON格式

{% hint style="info" %}
**格式：**序列化架构   
**格式：**反序列化架构
{% endhint %}

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

### Apache Avro格式

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

