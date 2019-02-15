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

Flink Kafka Consumer需要知道如何将Kafka中的二进制数据转换为Java / Scala对象。

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

### Kafka Consumer容错

启用Flink的检查点后，Flink Kafka Consumer将使用主题中的记录，并以一致的方式定期检查其所有Kafka offset以及其他操作的状态。如果作业失败，Flink会将流式程序恢复到最新检查点的状态，并从存储在检查点中的偏移量开始重新读取来自Kafka的记录。

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

如果未启用检查点，Kafka使用者将定期向Zookeeper提交偏移量。

### Kafka Consumer Topic及分区发现

#### 分区发现

#### Topic发现

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

## Kafka Producer

{% tabs %}
{% tab title="Java" %}
```text
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

### Kafka Producer分区计划

### Kafka Producer容错

## 在Kafka 0.10中使用Kafka时间戳和Flink EventTime

```java
public long extractTimestamp(Long element, long previousElementTimestamp) {
    return previousElementTimestamp;
}
```

```java
FlinkKafkaProducer010.FlinkKafkaProducer010Configuration config = FlinkKafkaProducer010.writeToKafkaWithTimestamps(streamWithTimestamps, topic, new SimpleStringSchema(), standardProps);
config.setWriteTimestampToKafka(true);
```

## Kafka Connector指标

## 启用Kerberos身份验证（仅适用于0.9及更高版本）

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

