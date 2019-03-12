# Kafka

此连接器提供对[Apache Kafka](https://kafka.apache.org/)服务的事件流的访问。

Flink提供特殊的Kafka连接器，用于从/向Kafka Topic读取和写入数据。Flink Kafka Consumer集成了Flink的检查点机制，可提供一次性处理语义。为实现这一目标，Flink并不完全依赖卡夫卡的消费者群体偏移跟踪，而是在内部跟踪和检查这些偏移。

请为您的用例和环境选择一个包（maven artifact id）和类名。对于大多数用户来说，`FlinkKafkaConsumer08`（部分`flink-connector-kafka`）较为合适。

然后，导入maven项目中的连接器：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kafka_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

{% hint style="info" %}
请注意，流连接器目前不是二进制发行版的一部分。在[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking.html)查看如何与之关联以进行集群执行。
{% endhint %}

## 安装Apache Kafka

* 按照[Kafka快速入门](https://kafka.apache.org/documentation.html#quickstart)的说明下载代码并启动服务器（每次启动应用程序前都需要启动Zookeeper和Kafka服务器）。
* 如果Kafka和Zookeeper服务器在远程机器上运行，那么必须在`config/server.properties`

  文件设置`advertised.host.name`为本机的IP地址。

## Kafka 1.0.0+连接器

从Flink 1.7开始，有一个新的通用Kafka连接器，它不跟踪特定的Kafka主要版本。相反，它在Flink发布时跟踪最新版本的Kafka。

如果您的Kafka代理版本是1.0.0或更高版本，则应使用此Kafka连接器。如果使用旧版本的Kafka（0.11,0.10,0.9或0.8），则应使用与代理版本对应的连接器。

### 兼容性

通过Kafka客户端API和Broker的兼容性保证，通用Kafka连接器与较旧和较新的Kafka Broker兼容。它与Kafka Broker版本0.11.0或更高版本兼容，具体取决于所使用的功能。有关Kafka兼容性的详细信息，请参阅[Kafka文档](https://kafka.apache.org/protocol.html#protocol_compatibility)。

### 用法

要使用通用Kafka连接器，请为其添加依赖关系：

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kafka_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>

```

然后实例化新的source（`FlinkKafkaConsumer`）和sink（`FlinkKafkaProducer`）。除了从模块和类名中删除特定的Kafka版本之外，API向后兼容Kafka 0.11连接器。

## Kafka Consumer

Flink的Kafka消费者被称为`FlinkKafkaConsumer08`（或Kafka 0.9.0.x版本的09，或仅`FlinkKafkaConsumer`适用于Kafka&gt; = 1.0.0版本）。它提供对一个或多个Kafka主题的访问。

构造函数接受以下参数：

1. Topic名称/Topic名称列表
2. DeserializationSchema / KeyedDeserializationSchema用于反序列化来自Kafka的数据
3. Kafka消费者的属性。需要以下属性：
   * “bootstrap.servers”（以逗号分隔的Kafka经纪人名单）
   * “zookeeper.connect”（逗号分隔的Zookeeper服务器列表）（**仅Kafka 0.8需要**）
   * “group.id”消费者群组的ID

例：

{% tabs %}
{% tab title="Java" %}
```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
// only required for Kafka 0.8
properties.setProperty("zookeeper.connect", "localhost:2181");
properties.setProperty("group.id", "test");
DataStream<String> stream = env
	.addSource(new FlinkKafkaConsumer08<>("topic", new SimpleStringSchema(), properties));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
// only required for Kafka 0.8
properties.setProperty("zookeeper.connect", "localhost:2181")
properties.setProperty("group.id", "test")
stream = env
    .addSource(new FlinkKafkaConsumer08[String]("topic", new SimpleStringSchema(), properties))
    .print()
```
{% endtab %}
{% endtabs %}

### DeserializationSchema

Flink Kafka Consumer需要知道如何将Kafka中的二进制数据转换为Java / Scala对象。`DeserializationSchema`允许用户指定这样的模式。为每个Kafka消息调用`T deserialize(byte[] message)` 方法，从Kafka传递值。

从`AbstractDeserializationSchema`开始通常很有帮助，它负责将生成的Java/Scala类型描述为Flink的类型系统。实现普通反序列化模式的用户需要自己实现`getProducedType(…)`方法。

对于访问Kafka消息的键和值，`KeyedDeserializationSchema`具有以下反序列化方法“`T deserialize（byte [] messageKey，byte [] message，String topic，int partition，long offset）`”。

为方便起见，Flink提供以下模式：

1. `TypeInformationSerializationSchema`（和`TypeInformationKeyValueSerializationSchema`）创建基于Flink的模式`TypeInformation`。如果Flink编写和读取数据，这将非常有用。此模式是其他通用序列化方法的高性能Flink替代方案。
2. `JsonDeserializationSchema`（和`JSONKeyValueDeserializationSchema`）将序列化的JSON转换为ObjectNode对象，可以使用`objectNode.get(“field”).as(Int/String/…)()`从中访问字段。KeyValue objectNode包含“key”和“value”字段，其中包含所有字段，以及公开此消息的偏移量/分区/主题的可选“元数据”字段。
3. `AvroDeserializationSchema`它使用静态提供的模式读取使用Avro格式序列化的数据。它可以从Avro生成的类（`AvroDeserializationSchema.forSpecific(...)`）推断出架构，也可以`GenericRecords` 使用手动提供的架构（with `AvroDeserializationSchema.forGeneric(...)`）。此反序列化架构要求序列化记录不包含嵌入式架构。
   * 此模式还有一个版本，可以在[Confluent Schema Registry中](https://docs.confluent.io/current/schema-registry/docs/index.html)查找编写器的模式（用于编写记录的 [模式）](https://docs.confluent.io/current/schema-registry/docs/index.html)。使用这些反序列化模式记录将使用从模式注册表中检索的模式进行读取，并转换为静态提供的模式（通过`ConfluentRegistryAvroDeserializationSchema.forGeneric(...)`或`ConfluentRegistryAvroDeserializationSchema.forSpecific(...)`）。

要使用此反序列化模式，必须添加以下附加依赖项：

{% tabs %}
{% tab title="AvroDeserializationSchema" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-avro</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```
{% endtab %}

{% tab title="ConfluentRegistryAvroDeserializationSchema" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-avro-confluent-registry</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```
{% endtab %}
{% endtabs %}

当遇到因任何原因无法反序列化的损坏消息时，有两个选项 - 从`deserialize(...)`方法中抛出异常将导致作业失败并重新启动，或返回`null`以允许Flink Kafka消费者以默认方式跳过损坏的消息。请注意，由于消费者的容错能力（请参阅下面的部分以获取更多详细信息），在损坏的消息上失败作业将使消费者尝试再次反序列化消息。因此，如果反序列化仍然失败，则消费者将在该损坏的消息上进入不间断重启和失败循环。

### Kafka Consumer开始位置配置

Flink Kafka Consumer允许配置如何确定Kafka分区的起始位置。

例：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

FlinkKafkaConsumer08<String> myConsumer = new FlinkKafkaConsumer08<>(...);
myConsumer.setStartFromEarliest();     // start from the earliest record possible
myConsumer.setStartFromLatest();       // start from the latest record
myConsumer.setStartFromTimestamp(...); // start from specified epoch timestamp (milliseconds)
myConsumer.setStartFromGroupOffsets(); // the default behaviour

DataStream<String> stream = env.addSource(myConsumer);
...
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

val myConsumer = new FlinkKafkaConsumer08[String](...)
myConsumer.setStartFromEarliest()      // start from the earliest record possible
myConsumer.setStartFromLatest()        // start from the latest record
myConsumer.setStartFromTimestamp(...)  // start from specified epoch timestamp (milliseconds)
myConsumer.setStartFromGroupOffsets()  // the default behaviour

val stream = env.addSource(myConsumer)
...
```
{% endtab %}
{% endtabs %}

Flink Kafka Consumer的所有版本都具有上述明确的起始位置配置方法。

* `setStartFromGroupOffsets`（默认行为）：从Kafka broker（或Zookeeper for Kafka 0.8）中的消费者组（`group.id`在消费者属性中设置）提交的偏移量开始读取分区。如果找不到分区的偏移量，`auto.offset.reset`将使用属性中的设置。
* `setStartFromEarliest()`/ `setStartFromLatest()`：从最早/最新记录开始。在这些模式下，Kafka中的承诺偏移将被忽略，不会用作起始位置。
* `setStartFromTimestamp(long)`：从指定的时间戳开始。对于每个分区，时间戳大于或等于指定时间戳的记录将用作起始位置。如果分区的最新记录早于时间戳，则只会从最新记录中读取分区。在此模式下，Kafka中的已提交偏移将被忽略，不会用作起始位置。

还可以为每个分区指定消费者应该从哪个偏移量开始:

{% tabs %}
{% tab title="Java" %}
```text
Map<KafkaTopicPartition, Long> specificStartOffsets = new HashMap<>();
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 0), 23L);
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 1), 31L);
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 2), 43L);

myConsumer.setStartFromSpecificOffsets(specificStartOffsets);
```
{% endtab %}

{% tab title="Scala" %}
```scala
Map<KafkaTopicPartition, Long> specificStartOffsets = new HashMap<>();
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 0), 23L);
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 1), 31L);
specificStartOffsets.put(new KafkaTopicPartition("myTopic", 2), 43L);

myConsumer.setStartFromSpecificOffsets(specificStartOffsets);
```
{% endtab %}
{% endtabs %}

上面的示例将消费者配置为从主题 myTopic的0、1和2分区的指定偏移量开始。偏移量值应该是消费者应该为每个分区读取的下一条记录。注意，如果消费者需要读取在提供的偏移量映射中没有指定偏移量的分区，那么它将返回到该特定分区的默认组偏移量行为\(即`setStartFromGroupOffsets()`\)。

请注意，当作业从故障中自动恢复或使用保存点手动恢复时，这些起始位置配置方法不会影响起始位置。在恢复时，每个Kafka分区的起始位置由存储在保存点或检查点中的偏移量确定（有关检查点的信息，请参阅下一节以启用消费者的容错）。

### Kafka Consumer容错

启用Flink的检查点后，Flink Kafka Consumer将使用主题中的记录，并以一致的方式定期检查其所有Kafka offset以及其他操作的状态。如果作业失败，Flink会将流式程序恢复到最新检查点的状态，并从存储在检查点中的偏移量开始重新读取来自Kafka的记录。

因此，绘制检查点的间隔定义了在失败的情况下，程序最多需要返回多少。

要使用容错Kafka消费者，需要在执行环境中启用拓扑的检查点:

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // checkpoint every 5000 msecs
```
{% endtab %}

{% tab title="Scala" %}
```scala
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // checkpoint every 5000 msecs
```
{% endtab %}
{% endtabs %}

另请注意，如果有足够的处理插槽可用于重新启动拓扑，则Flink只能重新启动拓扑。因此，如果拓扑由于丢失了TaskManager而失败，那么之后仍然需要有足够的可用插槽。YARN上的Flink支持自动重启丢失的YARN容器。

如果未启用检查点，Kafka消费者将定期向Zookeeper提交偏移量。

### Kafka Consumer Topic及分区发现

#### 分区发现

Flink Kafka Consumer支持发现动态创建的Kafka分区，并保证仅完全消费一次。在初始检索分区元数据之后（即，当作业开始运行时）发现的所有分区将从最早可能的偏移量中消耗。

默认情况下，禁用分区发现。要启用它，请`flink.partition-discovery.interval-millis`在提供的属性配置中设置非负值，表示以毫秒为单位的发现间隔。

{% hint style="danger" %}
限制：当从Flink 1.3.x之前的Flink版本的保存点还原消费者时，无法在还原运行时启用分区发现。如果启用，则还原将失败并出现异常。在这种情况下，为了使用分区发现，请首先在Flink 1.3.x中使用保存点，然后再从中恢复。
{% endhint %}

#### Topic发现

在更高级别，Flink Kafka Consumer还能够使用正则表达式基于主题名称的模式匹配来发现主题。请参阅以下示例：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
properties.setProperty("group.id", "test");

FlinkKafkaConsumer011<String> myConsumer = new FlinkKafkaConsumer011<>(
    java.util.regex.Pattern.compile("test-topic-[0-9]"),
    new SimpleStringSchema(),
    properties);

DataStream<String> stream = env.addSource(myConsumer);
...

```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
properties.setProperty("group.id", "test")

val myConsumer = new FlinkKafkaConsumer08[String](
  java.util.regex.Pattern.compile("test-topic-[0-9]"),
  new SimpleStringSchema,
  properties)

val stream = env.addSource(myConsumer)
...

```
{% endtab %}
{% endtabs %}

在上面的示例中，当作业开始运行时，消费者将订阅名称与指定正则表达式匹配的所有主题\(从`test-topic-`开始，以一位数字结束\)。

要允许消费者在作业开始运行后动态发现创建的主题，请为`flink. parti-discovery.interval-millis`设置一个非负值。这允许消费者发现具有与指定模式匹配的名称的新主题的分区。

### Kafka消费者偏移量提交行为配置

Flink Kafka Consumer允许配置如何将偏移提交回Kafka代理（或0.8中的Zookeeper）的行为。请注意，Flink Kafka Consumer不依赖于提交的偏移量来实现容错保证。提交的偏移量仅是用于暴露消费者进展以进行监控的手段。

配置偏移提交行为的方式不同，具体取决于是否为作业启用了检查点。

* _禁用_检查_点：_如果禁用了检查点，则Flink Kafka Consumer依赖于内部使用的Kafka客户端的自动定期偏移提交功能。因此，要禁用或启用偏移提交，只需将`enable.auto.commit`（或`auto.commit.enable` Kafka 0.8）/ `auto.commit.interval.ms`键设置为所提供`Properties`配置中的适当值。
* _启用_检查_点：_如果启用了检查点，则Flink Kafka Consumer将在检查点完成时提交存储在检查点状态中的偏移量。这可确保Kafka broker中的提交偏移量与检查点状态中的偏移量一致。用户可以通过调用消费者的方法来选择禁用或启用偏移提交`setCommitOffsetsOnCheckpoints(boolean)`（默认情况下，行为是`true`）。请注意，在此方案中，`Properties`完全忽略自动定期偏移提交设置。

### Kafka消费者和时间戳提取/水印排放

在许多情况下，记录的时间戳（显式或隐式）嵌入记录本身。另外，用户可能想要周期性地或以不规则的方式发送水印，例如基于包含当前事件时间水印的Kafka流中的特殊记录。对于这些情况，Flink Kafka Consumer允许指定一个`AssignerWithPeriodicWatermarks`或一个`AssignerWithPunctuatedWatermarks`。

可以按[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/apis/streaming/event_timestamps_watermarks.html)所述指定自定义时间戳提取器/水印发射器，或使用 [预定义的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/apis/streaming/event_timestamp_extractors.html)时间戳提取器/水印发射器 。完成后，可以通过以下方式将其传递给消费者：

{% tabs %}
{% tab title="Java" %}
```java
Properties properties = new Properties();
properties.setProperty("bootstrap.servers", "localhost:9092");
// only required for Kafka 0.8
properties.setProperty("zookeeper.connect", "localhost:2181");
properties.setProperty("group.id", "test");

FlinkKafkaConsumer08<String> myConsumer =
    new FlinkKafkaConsumer08<>("topic", new SimpleStringSchema(), properties);
myConsumer.assignTimestampsAndWatermarks(new CustomWatermarkEmitter());

DataStream<String> stream = env
	.addSource(myConsumer)
	.print();

```
{% endtab %}

{% tab title="Scala" %}
```scala
val properties = new Properties()
properties.setProperty("bootstrap.servers", "localhost:9092")
// only required for Kafka 0.8
properties.setProperty("zookeeper.connect", "localhost:2181")
properties.setProperty("group.id", "test")

val myConsumer = new FlinkKafkaConsumer08[String]("topic", new SimpleStringSchema(), properties)
myConsumer.assignTimestampsAndWatermarks(new CustomWatermarkEmitter())
stream = env
    .addSource(myConsumer)
    .print()
```
{% endtab %}
{% endtabs %}

在内部，每个Kafka分区执行一个分配器实例。当指定这样的分配器时，对于从Kafka读取的每个记录，调用`extractTimestamp(T element, long previousElementTimestamp)`为记录分配时间戳， 并且调用`Watermark getCurrentWatermark()`（用于定期）或`Watermark checkAndGetNextWatermark(T lastElement, long extractedTimestamp)`（用于标点符号）以确定是否应该发出新的水印并且时间戳。

**注意**：如果水印分配器依赖于从Kafka读取的记录来推进其水印（通常是这种情况），则所有主题和分区都需要具有连续的记录流。否则，整个应用程序的水印无法前进，并且所有基于时间的操作（例如时间窗口或具有计时器的功能）都无法取得进展。单个空闲Kafka分区会导致此行为。计划进行Flink改进以防止这种情况发生（参见[FLINK-5479：FlinkKafkaConsumer中的每分区水印应考虑空闲分区](https://issues.apache.org/jira/browse/FLINK-5479)）。同时，可能的解决方法是将_心跳消息发送_到所有消耗的分区，从而推进空闲分区的水印。

## Kafka Producer

Flink的Kafka Producer被称为`FlinkKafkaProducer011`（或`010`Kafka 0.10.0.x版本等，或仅`FlinkKafkaProducer`适用于Kafka&gt; = 1.0.0版本）。它允许将记录流写入一个或多个Kafka主题。

例：

{% tabs %}
{% tab title="Java" %}
```java
DataStream<String> stream = ...;

FlinkKafkaProducer011<String> myProducer = new FlinkKafkaProducer011<String>(
        "localhost:9092",            // broker list
        "my-topic",                  // target topic
        new SimpleStringSchema());   // serialization schema

// versions 0.10+ allow attaching the records' event timestamp when writing them to Kafka;
// this method is not available for earlier Kafka versions
myProducer.setWriteTimestampToKafka(true);

stream.addSink(myProducer);

```
{% endtab %}

{% tab title="Scala" %}
```scala
val stream: DataStream[String] = ...

val myProducer = new FlinkKafkaProducer011[String](
        "localhost:9092",         // broker list
        "my-topic",               // target topic
        new SimpleStringSchema)   // serialization schema

// versions 0.10+ allow attaching the records' event timestamp when writing them to Kafka;
// this method is not available for earlier Kafka versions
myProducer.setWriteTimestampToKafka(true)

stream.addSink(myProducer)

```
{% endtab %}
{% endtabs %}

上面的示例演示了创建Flink Kafka Producer以将流写入单个Kafka目标主题的基本用法。对于更高级的用法，还有其他构造函数变体允许提供以下内容：

* _提供自定义属性_：生产者允许为内部提供自定义属性配置`KafkaProducer`。有关如何配置Kafka Producers的详细信息，请参阅[Apache Kafka文档](https://kafka.apache.org/documentation.html)。
* _自定义分区程序_：要将记录分配给特定的分区，可以向构造函数提供`FlinkKafkaPartitioner`的实现。将为流中的每个记录调用此分区器，以确定应该将记录发送到目标主题的哪个确切分区。有关详细信息，请参阅[Kafka Producer Partitioning Scheme](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/kafka.html#kafka-producer-partitioning-scheme)。
* _高级序列化模式_：与使用者类似，生产者还允许使用调用的高级序列化模式`KeyedSerializationSchema`，该模式允许单独序列化键和值。它还允许覆盖目标主题，以便一个生产者实例可以将数据发送到多个主题。

### Kafka Producer分区方案

默认情况下，如果没有为Flink Kafka生成器指定自定义分区器，则生成器将使用`FlinkFixedPartitioner`将每个Flink Kafka生成器并行的子任务映射到单个Kafka分区\(即， sink子任务接收到的所有记录都将位于同一个Kafka分区中\)。

可以通过扩展`FlinkKafkaPartitioner`类来实现自定义分区器。所有Kafka版本的构造函数都允许在实例化生成器时提供自定义分区器。注意，分区器实现必须是可序列化的，因为它们将跨Flink节点传输。另外，请记住，分区程序中的任何状态都会因为作业失败而丢失，因为分区程序不是生产者的检查点状态的一部分。

还可以完全避免使用和使用类型的分区器，并简单地让Kafka通过其附加的键对写入的记录进行分区\(根据使用提供的序列化模式为每个记录确定的键\)。为此，在实例化生成器时提供一个空自定义分区器。重要的是提供`null`作为自定义分区器;如上所述，如果没有指定自定义分区程序，则使用`FlinkFixedPartitioner`。

### Kafka Producer容错

#### **卡夫卡0.8**

在0.9之前，Kafka没有提供任何机制来保证至少一次或完全一次的语义。

#### **卡夫卡0.9和0.10**

启用Flink的检查点时，`FlinkKafkaProducer09`并`FlinkKafkaProducer010` 能提供在-至少一次传输保证。

除了启用Flink的检查点之外，还应该适当地配置setter方法`setLogFailuresOnly(boolean)`和`setFlushOnCheckpoint(boolean)`。

* `setLogFailuresOnly(boolean)`：默认情况下，此设置为`false`。启用此选项将使生产者仅记录失败而不是捕获和重新抛出它们。这基本上记录了成功的记录，即使它从未写入目标Kafka主题。必须至少禁用一次。
* `setFlushOnCheckpoint(boolean)`：默认情况下，此设置为`true`。启用此功能后，Flink的检查点将在检查点成功之前等待Kafka确认检查点时的任何即时记录。这可确保检查点之前的所有记录都已写入Kafka。必须至少启用一次。

总之，Kafka生成器默认情况下对0.9和0.10版本至少有一次保证，`setLogFailureOnly`设置为false, `setFlushOnCheckpoint`设置为true

**注意**：默认情况下，重试次数设置为“0”。这意味着当`setLogFailuresOnly`设置为时`false`，生产者会在错误\(包括leader更改\)时立即失败。默认情况下，该值设置为“0”，以避免目标主题中由重试引起的重复消息。对于经常更改代理的大多数生产环境，我们建议将重试次数设置为更高的值。

**注意**：目前Kafka的生产者不支持事务，因此Flink无法保证一次性交付Kafka主题。

#### **卡夫卡0.11和更新**

启用Flink的检查点后，`FlinkKafkaProducer011`（`FlinkKafkaProducer`对于Kafka&gt; = 1.0.0版本）可以提供**完全**一次性交付保证。

除了启用Flink的检查点，您还可以通过将适当的`semantic`参数传递给`FlinkKafkaProducer011`（`FlinkKafkaProducer`对于Kafka&gt; = 1.0.0版本）来选择三种不同的操作模式：

* `Semantic.NONE`：Flink不保证任何东西。生成的记录可能会丢失，也可能会被复制。
* `Semantic.AT_LEAST_ONCE`（默认设置）：类似于`setFlushOnCheckpoint(true)`在 `FlinkKafkaProducer010`。这可以保证不会丢失任何记录（尽管它们可以重复）。
* `Semantic.EXACTLY_ONCE`：使用Kafka事务提供完全一次的语义。无论何时使用事务写入Kafka，都不要忘记为消耗Kafka记录的任何应用程序设置所需的`isolation.level`（`read_committed` 或`read_uncommitted`- 后者是默认值）。

**注意事项**

Semantic.EXACTLY\_ONCE模式依赖于从检查点恢复后提交在接受检查点之前启动的事务的能力。如果Flink应用程序崩溃和完全重新启动之间的时间比较长，那么Kafka的事务超时将会导致数据丢失\(Kafka会自动中止超过超时时间的事务\)。考虑到这一点，请根据预期停机时间适当配置事务超时时间。

Kafka代理在默认情况下具有`transaction.max.timeout.ms`设置为15分钟。此属性不允许为大于其值的生成者设置事务的超时时间。默认情况下，FlinkKafkaProducer011设置`transaction.timeout.ms`属性设置为1小时，因此`transaction.max.timeout.ms`在使用`Semantic.EXACTLY_ONCE`模式之前应该增加 该属性。

在KafkaConsumer的read\_committed模式中，任何没有完成\(既没有中止也没有完成\)的事务都将阻塞来自给定Kafka主题的所有读取，从而导致所有未完成的事务被阻塞。换句话说，在遵循以下事件序列之后：

* 用户启动transaction1并使用它编写一些记录 
* 用户启动transaction2并使用它编写了一些进一步的记录 
* 用户提交transaction2

即使`transaction2`已经提交了记录，在`transaction1`提交或中止之前，消费者也不会看到它们。这有两个含义：

* 首先，在Flink应用程序的正常工作期间，用户可以预期Kafka主题中生成的记录的可见性会延迟，等于已完成检查点之间的平均时间。
* 其次，在Flink应用程序失败的情况下，读者将阻止此应用程序编写的主题，直到应用程序重新启动或配置的事务超时时间过去为止。此注释仅适用于有多个agent/应用程序写入同一Kafka主题的情况。

**注意**： `Semantic.EXACTLY_ONCE`模式对每个`FlinkKafkaProducer011`实例使用固定大小的KafkaProducers池。每个检查点使用其中一个生产者。如果并发检查点的数量超过池大小，`FlinkKafkaProducer011` 将引发异常并将使整个应用程序失败。请相应地配置最大池大小和最大并发检查点数。

**注意**：`Semantic.EXACTLY_ONCE` 采取所有可能的措施，不留下任何延迟的事务，这些事务可能会阻止消费者阅读Kafka主题的内容。但是，如果在第一个检查点之前Flink应用程序失败，在重新启动此类应用程序之后，系统中没有关于以前池大小的信息。因此，在第一个检查点完成之前缩小Flink应用程序是不安全的，因为大于`FlinkKafkaProducer011.SAFE_SCALE_DOWN_FACTOR`。

## 在Kafka 0.10中使用Kafka时间戳和Flink EventTime

自Apache Kafka 0.10+以来，Kafka的消息可以携带 [时间戳](https://cwiki.apache.org/confluence/display/KAFKA/KIP-32+-+Add+timestamps+to+Kafka+message)，指示事件发生的时间（请参阅[Apache Flink中的“事件时间”](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/event_time.html)）或消息写入Kafka代理的时间。

如果Flink中的时间特征设置为`TimeCharacteristic.EventTime`（`StreamExecutionEnvironment.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)`），那么`FlinkKafkaConsumer010`将发出带有时间戳的记录。

Kafka的消费者不排放水印。要发出水印，可以使用`assignTimestampsAndWatermarks`方法，使用“Kafka消费者和时间戳提取/水印排放”中描述的机制。

使用Kafka的时间戳时，无需定义时间戳提取器。`extractTimestamp()`方法的`previousElementTimestamp`参数包含Kafka消息携带的时间戳。

Kafka消费者的时间戳提取器如下所示：

```java
public long extractTimestamp(Long element, long previousElementTimestamp) {
    return previousElementTimestamp;
}
```

如果设置`setWriteTimestampToKafka(true)`， `FlinkKafkaProducer010`只发出记录时间戳。

```java
FlinkKafkaProducer010.FlinkKafkaProducer010Configuration config = FlinkKafkaProducer010.writeToKafkaWithTimestamps(streamWithTimestamps, topic, new SimpleStringSchema(), standardProps);
config.setWriteTimestampToKafka(true);
```

## Kafka Connector指标

Flink的Kafka连接器通过Flink的[度量系统](https://ci.apache.org/projects/flink/flink-docs-release-1.7/monitoring/metrics.html)提供一些指标来分析连接器的行为。生产者通过Flink的度量系统为所有支持的版本导出Kafka的内部指标。消费者从Kafka 0.9版开始导出所有指标。Kafka文档在其[文档中](http://kafka.apache.org/documentation/#selector_monitoring)列出了所有导出的指标。

除了这些度量之外，所有使用者还公开每个主题分区的`current-offsets`和`committed-offsets`。`current-offsets`是指分区中的当前偏移量。这指的是我们成功检索和发出的最后一个元素的偏移量。`committed-offsets`是最后一个提交偏移量。

Flink中的Kafka消费者将偏移量提交给Zookeeper \(Kafka 0.8\)或Kafka Broker\(Kafka 0.9+\)。如果禁用检查点，则定期提交偏移量。通过检查点，一旦流拓扑中的所有操作符确认他们已经为自己的状态创建了检查点，提交就会发生。这为用户提供了提交给Zookeeper或Broker的偏移量的至少一次语义。对于指向Flink的偏移量检查，系统提供了一次准确的保证。

提交给ZK或Broker的偏移量也可以用于跟踪Kafka消费者的读取进度。每个分区中提交的偏移量和最新偏移量之间的差异称为消费者延迟。如果Flink拓扑使用来自主题的数据的速度比添加新数据的速度慢，那么延迟就会增加，用户就会落后。对于大型生产部署，我们建议监视该指标，以避免延迟增加。

## 启用Kerberos身份验证（仅适用于0.9及更高版本）

Flink通过Kafka连接器提供了一流的支持，可以对Kerberos配置的Kafka安装进行身份验证。只需在Flink的flink-conf.yaml中配置支持Kafka的Kerberos身份验证，如下所示:

1. 通过设置以下内容配置Kerberos凭据 -
   * `security.kerberos.login.use-ticket-cache`：默认情况下为`true`，Flink将尝试在`kinit`管理的票据缓存中使用Kerberos凭证。请注意，在YARN上部署的Flink作业中使用Kafka连接器时，使用票证缓存的Kerberos授权将不起作用。使用Mesos进行部署时也是如此，因为Mesos部署不支持使用票证缓存进行授权。
   * `security.kerberos.login.keytab`and `security.kerberos.login.principal`：要改为使用Kerberos keytabs，请为这两个属性设置值。
2. 附加`KafkaClient`到`security.kerberos.login.contexts`：这告诉Flink将配置的Kerberos凭据提供给Kafka登录上下文以用于Kafka身份验证。

启用基于Kerberos的Flink安全性后，只需在提供的属性配置中包含以下两个设置（通过传递给内部Kafka客户端），即可使用Flink Kafka Consumer或Producer向Kafka进行身份验证：

* 设置`security.protocol`为`SASL_PLAINTEXT`（默认值`NONE`）：用于与Kafka代理进行通信的协议。使用独立Flink部署时，您也可以使用`SASL_SSL`; 请[在此处](https://kafka.apache.org/documentation/#security_configclients)查看如何为SSL配置Kafka客户端。
* 设置`sasl.kerberos.service.name`为`kafka`（默认值`kafka`）：此值应与`sasl.kerberos.service.name`用于Kafka代理配置的值相匹配。客户端和服务器配置之间的服务名称不匹配将导致身份验证失败。

有关Kerberos安全性的Flink配置的更多信息，请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html)。您还可以[在此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/security-kerberos.html)找到有关Flink内部如何设置基于Kerberos的安全性的更多详细信息。

## 故障排除

> 如果您在使用Flink时遇到Kafka问题，请记住Flink只包装 [KafkaConsumer](https://kafka.apache.org/documentation/#consumerapi)或 [KafkaProducer](https://kafka.apache.org/documentation/#producerapi) ，您的问题可能独立于Flink，有时可以通过升级Kafka Broker，重新配置Kafka Broker或重新配置Flink中的KafkaConsumer或KafkaProducer来解决。下面列出了一些常见问题的例子。

### 数据丢失

根据您的Kafka配置，即使在Kafka确认写入后您仍然可能会遇到数据丢失。特别要记住Kafka配置中的以下属性：

* `acks`
* `log.flush.interval.messages`
* `log.flush.interval.ms`
* `log.flush.*`

上述选项的默认值很容易导致数据丢失。有关更多说明，请参阅Kafka文档。

### UnknownTopicOrPartitionException

导致此错误的一个可能原因是当新的领导者选举发生时，例如在重新启动Kafka经纪人之后或期间。这是一个可重试的异常，因此Flink作业应该能够重启并恢复正常运行。还可以通过更改生产者设置中的重试属性来规避此问题。然而，这可能会导致消息的重新排序，如果不需要，可以通过设置`max.in.flight.request.per`为1来避免这种情况。

