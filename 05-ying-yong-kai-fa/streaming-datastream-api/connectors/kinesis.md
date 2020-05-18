# Kinesis

 Kinesis连接器提供对[Amazon AWS Kinesis Streams的](http://aws.amazon.com/kinesis/streams/)访问。

要使用连接器，请将以下Maven依赖项添加到您的项目中:

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-connector-kinesis_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

{% hint style="danger" %}
在Flink 1.10.0版本之前，Flink -connector-kinesis\_2.11依赖于在Amazon软件许可下授权的代码。链接到先前版本的“flink-connector-kinesis”将把这段代码包含到您的应用程序中。
{% endhint %}

## 使用Amazon Kinesis Streams服务

 按照[Amazon Kinesis Streams开发人员指南](https://docs.aws.amazon.com/streams/latest/dev/learning-kinesis-module-one-create-stream.html) 中的说明设置Kinesis流。确保创建适当的IAM策略和用户以读取/写入Kinesis流。

## Kinesis 消费者

FlinkKinesisConsumer是一个完全并行的流数据源，它订阅同一个AWS服务区域内的多个AWS Kinesis流，并且可以在作业运行时透明地处理流的重新分片。使用者的每个子任务负责从多个Kinesis分片中获取数据记录。每个子任务获取的分片数量会随着分片的关闭和由Kinesis创建而改变。

在使用Kinesis流中的数据之前，请确保在AWS仪表板中创建的所有流的状态均为“ ACTIVE”。

{% tabs %}
{% tab title="Java" %}
```java
Properties consumerConfig = new Properties();
consumerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1");
consumerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id");
consumerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key");
consumerConfig.put(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST");

StreamExecutionEnvironment env = StreamExecutionEnvironment.getEnvironment();

DataStream<String> kinesis = env.addSource(new FlinkKinesisConsumer<>(
    "kinesis_stream_name", new SimpleStringSchema(), consumerConfig));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val consumerConfig = new Properties()
consumerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1")
consumerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id")
consumerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key")
consumerConfig.put(ConsumerConfigConstants.STREAM_INITIAL_POSITION, "LATEST")

val env = StreamExecutionEnvironment.getEnvironment

val kinesis = env.addSource(new FlinkKinesisConsumer[String](
    "kinesis_stream_name", new SimpleStringSchema, consumerConfig))
```
{% endtab %}
{% endtabs %}

上面是一个使用消费者的简单示例。消费者的配置由java.util.Properties实例提供。配置键可以在`AWSConfigConstants`\(特定于aws的参数\)和`ConsumerConfigConstants` \(Kinesis消费者参数\)中找到。这个例子演示了在AWS区域“us-east-1”消费一个单驱动流。AWS凭据是使用基本方法提供的，其中AWS访问密钥ID和秘密访问密钥直接在配置中提供\(其他选项是设置`AWSConfigConstants`\)。`AWS_CREDENTIALS_PROVIDER`提供`ENV_VAR`、`SYS_PROP`、`PROFILE`、`ASSUME_ROLE`和`AUTO`\)。此外，数据是从Kinesis流的最新位置消费的\(另一个选项是设置`ConsumerConfigConstants`。`STREAM_INITIAL_POSITION`到`TRIM_HORIZON`，它允许用户从尽可能早的记录开始读取Kinesis流\)。

消费者的其他可选配置键可以在`ConsumerConfigConstants`中找到。

请注意，Flink Kinesis Customer Source配置并行度可以完全独立于Kinesis流中的分片总数。当分片的数量大于消费者的并行度时，每个使用者子任务可以订阅多个分片;否则，如果分片的数量小于消费者的并行度，那么一些消费者子任务就会处于空闲状态，并等待分配新的分片\(即分片\)。，当流被重新分片以增加分片数量以满足更高的Kinesis服务吞吐量时\)。

还要注意，当分片id不是连续的\(由于Kinesis中的动态分片\)时，将分片分配给子任务可能不是最优的。对于赋值倾斜导致消费严重不平衡的情况，可以在消费者上设置`KinesisShardAssigner`的自定义实现。

### 配置起始位置

Flink Kinesis消费者目前提供以下选项来配置从哪里开始读取Kinesis流，只需设置ConsumerConfigConstants。STREAM\_INITIAL\_POSITION到所提供的配置属性中的以下值之一。 即可（这些选项的命名完全遵循[AWS Kinesis Streams服务使用](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html#API_GetShardIterator_RequestSyntax)的命名）：

* `LATEST`：从最新记录开始读取所有流的所有分片。
* `TRIM_HORIZON`：从可能的最早记录开始读取所有流的所有分片（根据保留设置，Kinesis可能会修剪数据）。
* `AT_TIMESTAMP`：从指定的时间戳开始读取所有流的所有分片。还必须通过`ConsumerConfigConstants.STREAM_INITIAL_TIMESTAMP`在以下日期模式之一中提供的值，在配置属性中指定时间戳记：
  * 一个非负的double值，表示自Unix纪元以来经过的秒数（例如`1459799926.480`）。
  * 用户定义的模式，这是一个有效的图案为`SimpleDateFormat`通过提供`ConsumerConfigConstants.STREAM_TIMESTAMP_DATE_FORMAT`。如果`ConsumerConfigConstants.STREAM_TIMESTAMP_DATE_FORMAT`未定义，则默认模式将是`yyyy-MM-dd'T'HH:mm:ss.SSSXXX` （例如，时间戳记值为，`2016-04-04`并且模式`yyyy-MM-dd`由用户指定，或者时间戳记值`2016-04-04T19:58:46.480-00:00`未指定模式）。

### 用户定义的状态更新语义一次的容错能力

当Flink的检查点启用时，Flink Kinesis消费者将使用跃动流中的分片记录，并定期检查每个分片的进程。如果作业失败，Flink将把流程序恢复到最新的完整检查点状态，并从存储在检查点的进程开始，重新使用来自Kinesis分片的记录。

因此，绘制检查点的间隔定义了程序在出现故障时最多需要返回多少。

要使用容错Kinesis消费者，需要在执行环境中启用拓扑检查点:

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // checkpoint every 5000 msecs
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
env.enableCheckpointing(5000) // checkpoint every 5000 msecs
```
{% endtab %}
{% endtabs %}

另外需要注意，Flink只能在有足够的处理槽可用时重新启动拓扑。因此，如果拓扑由于任务管理器的丢失而失败，那么之后一定还有足够的插槽可用。纱线上的Flink支持自动重新启动丢失的Yarn容器。

### 使用事件时间消费记录

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```
{% endtab %}
{% endtabs %}

如果流拓扑选择将事件时间概念用于记录时间戳，则默认情况下将使用近似到达时间戳。这个时间戳被Kinesis附加到记录上，一旦它们被streams成功接收并存储。注意，这个时间戳通常被称为Kinesis服务器端时间戳，并不能保证准确性或顺序正确性\(例如，时间戳可能不总是升序的\)。

用户可以选择使用自定义时间戳覆盖此默认值，如这里所述，或者使用来自预定义时间戳的时间戳。这样做之后，可以通过以下方式传递给消费者:

{% tabs %}
{% tab title="Java" %}
```java
FlinkKinesisConsumer<String> consumer = new FlinkKinesisConsumer<>(
    "kinesis_stream_name",
    new SimpleStringSchema(),
    kinesisConsumerConfig);
consumer.setPeriodicWatermarkAssigner(new CustomAssignerWithPeriodicWatermarks());
DataStream<String> stream = env
	.addSource(consumer)
	.print();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val consumer = new FlinkKinesisConsumer[String](
    "kinesis_stream_name",
    new SimpleStringSchema(),
    kinesisConsumerConfig);
consumer.setPeriodicWatermarkAssigner(new CustomAssignerWithPeriodicWatermarks());
val stream = env
	.addSource(consumer)
	.print();
```
{% endtab %}
{% endtabs %}

在内部，每个分片/消费者线程执行一个分配程序实例\(请参阅下面的线程模型\)。当指定了一个分配者时，对于从Kinesis读取的每个记录，将调用extractTimestamp\(T元素，long previousElementTimestamp\)来为记录分配一个时间戳，并调用getCurrentWatermark\(\)来确定shard的新水印。使用者子任务的水印随后被确定为其所有分片的最小水印，并定期发出。per分片水印对于处理分片之间不同的消费速度至关重要，否则可能会导致依赖于水印的下游逻辑出现问题，例如不正确的后期数据删除。

默认情况下，如果分片不提供新记录，水印就会停止。房地产ConsumerConfigConstants。SHARD\_IDLE\_INTERVAL\_MILLIS可以通过超时来避免这个潜在的问题，超时将允许水印继续前进，尽管存在空闲分片。

### 分片消费者事件时间对齐

Flink Kinesis消费者可选地支持并行消费者子任务\(及其线程\)之间的同步，以避免跨源的事件时间同步中描述的事件时间倾斜相关问题。

若要启用同步，请在使用者上设置水印跟踪器:

```java
JobManagerWatermarkTracker watermarkTracker =
    new JobManagerWatermarkTracker("myKinesisSource");
consumer.setWatermarkTracker(watermarkTracker);
```

`JobManagerWatermarkTracker`将使用一个全局聚合来同步每个子任务的水印。每个子任务使用一个`shard`队列来控制记录向下游发送的速率，该速率基于队列中的下一个记录与全局水印之间的距离。

“`emit ahead`”限制是通过 `ConsumerConfigConstants.WATERMARK_LOOKAHEAD_MILLIS`配置的。值越小，歪斜越小，吞吐量也越小。更大的值将允许子任务在等待全局水印前进之前继续前进。

吞吐量方程中的另一个变量是跟踪器传播水印的频率。时间间隔可以通过  `ConsumerConfigConstants.WATERMARK_SYNC_MILLIS`来配置。较小的值减少发射器等待，并以增加与作业管理器的通信为代价。

由于发生歪斜时记录会累积在队列中，因此需要增加内存消耗。多少取决于平均记录大小。对于较大的尺寸，可能需要通过`ConsumerConfigConstants.WATERMARK_SYNC_QUEUE_CAPACITY`调整发射器队列容量。

### 线程模型

Flink Kinesis Consumer使用多个线程进行分片发现和数据消费。

对于shard发现，每个并行的消费者子任务都有一个线程，该线程不断地查询Kinesis以获取shard信息，即使子任务最初在启动使用者时没有要读取的分片。换句话说，如果使用者以10的并行度运行，那么将会有10个线程不断地查询Kinesis，而不管订阅流中分片的总数。

对于数据消费，将创建一个线程来消费每个发现的分片。当由于流重新分片导致负责使用的分片关闭时，线程将终止。换句话说，每个打开的分片始终只有一个线程。

### 使用内部的Kinesis API

 Flink Kinesis Consumer 内部使用[AWS Java SDK](http://aws.amazon.com/sdk-for-java/)来调用Kinesis API，以进行分片发现和数据使用。由于Amazon 对API上的[Kinesis Streams](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)的[服务限制](http://docs.aws.amazon.com/streams/latest/dev/service-sizes-and-limits.html)，消费者将与用户正在运行的其他非Flink消费应用程序竞争。以下是使用者调用的API列表，其中描述了使用者如何使用API​​，以及有关如何处理由于这些服务限制而导致Flink Kinesis使用者可能出现的任何错误或警告的信息。

* [_ListShards_](https://docs.aws.amazon.com/kinesis/latest/APIReference/API_ListShards.html)：每个并行消费者子任务中的单个线程经常调用此方法，以发现由于流重新分片而导致的任何新分片。默认情况下，使用者以10秒的间隔执行分片发现，并且将无限期重试，直到获得Kinesis的结果。如果这干扰了其他非Flink消费应用程序，则用户可以通过`ConsumerConfigConstants.SHARD_DISCOVERY_INTERVAL_MILLIS`在提供的配置属性中设置的值来减慢调用此API的速度。这会将发现间隔设置为其他值。请注意，此设置将直接影响发现新分片并开始使用该分片的最大延迟，因为在间隔期间将不会发现分片。
* [_GetShardIterator_](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html)：仅在启动每个分片消费线程时才调用一次，如果Kinesis抱怨API的事务限制已超过，则重试，默认尝试3次。请注意，由于此API的速率限制是每个分片（而不是每个流）的速率限制，因此使用者本身不应超过该限制。通常，如果发生这种情况，用户可以尝试降低调用这个API的任何其他非flink消费应用程序的速度，或者通过设置前缀为`ConsumerConfigConstants.SHARD`_`GETITERATOR`_`*`的键来修改消费者中这个API调用的重试行为。
* [_GetRecords_](http://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetRecords.html)：每个分片消费线程都会不断调用它来从Kinesis获取记录。当一个分片具有多个并发使用者时（当有任何其他非flink消费应用程序在运行时），可能会超出每个分片速率限制。默认情况下，在每次调用此API时，如果Kinesis抱怨该API的数据大小/事务限制已超过，则消费者将重试，默认为3次尝试。用户可以尝试减慢其他不使用Flink的应用程序，或者在提供的配置属性中

  通过设置`ConsumerConfigConstants.SHARD_GETRECORDS_MAX`和`ConsumerConfigConstants.SHARD_GETRECORDS_INTERVAL_MILLIS`

  来调整使用方的吞吐量。设置前者将调整每个使用线程尝试从每个调用的分片中获取的最大记录数（默认值为10,000），而后者将修改每次获取之间的睡眠间隔（默认值为200）。消费者在调用这个API时的重试行为也可以通过使用以ConsumerConfigConstants.SHARD_GETRECORDS_\*前缀的其他键来修改。

## Kinesis生产者

`FlinkKinesisProducer`使用[KinesisProducer库\(KPL\)](http://docs.aws.amazon.com/streams/latest/dev/developing-producers-with-kpl.html)将来自Flink流的数据放入到Kinesis流中。

请注意，生产者没有参与Flink的检查点，并且不提供精确的一次处理保证。此外，Kinesis生成器不保证记录是按照分片顺序编写的\(有关详细信息，请参阅此处和此处\)。

在失败或重新分片的情况下，数据将被再次写入Kinesis，导致重复。这种行为通常称为“至少一次”语义。

要将数据放入Kinesis流，请确保在AWS仪表板中将该流标记为“ACTIVE”。

要使监控工作正常，访问流的用户需要访问CloudWatch服务。

{% tabs %}
{% tab title="Java" %}
```java
Properties producerConfig = new Properties();
// Required configs
producerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1");
producerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id");
producerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key");
// Optional configs
producerConfig.put("AggregationMaxCount", "4294967295");
producerConfig.put("CollectionMaxCount", "1000");
producerConfig.put("RecordTtl", "30000");
producerConfig.put("RequestTimeout", "6000");
producerConfig.put("ThreadPoolSize", "15");

// Disable Aggregation if it's not supported by a consumer
// producerConfig.put("AggregationEnabled", "false");
// Switch KinesisProducer's threading model
// producerConfig.put("ThreadingModel", "PER_REQUEST");

FlinkKinesisProducer<String> kinesis = new FlinkKinesisProducer<>(new SimpleStringSchema(), producerConfig);
kinesis.setFailOnError(true);
kinesis.setDefaultStream("kinesis_stream_name");
kinesis.setDefaultPartition("0");

DataStream<String> simpleStringStream = ...;
simpleStringStream.addSink(kinesis);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val producerConfig = new Properties()
// Required configs
producerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1")
producerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id")
producerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key")
// Optional KPL configs
producerConfig.put("AggregationMaxCount", "4294967295")
producerConfig.put("CollectionMaxCount", "1000")
producerConfig.put("RecordTtl", "30000")
producerConfig.put("RequestTimeout", "6000")
producerConfig.put("ThreadPoolSize", "15")

// Disable Aggregation if it's not supported by a consumer
// producerConfig.put("AggregationEnabled", "false")
// Switch KinesisProducer's threading model
// producerConfig.put("ThreadingModel", "PER_REQUEST")

val kinesis = new FlinkKinesisProducer[String](new SimpleStringSchema, producerConfig)
kinesis.setFailOnError(true)
kinesis.setDefaultStream("kinesis_stream_name")
kinesis.setDefaultPartition("0")

val simpleStringStream = ...
simpleStringStream.addSink(kinesis)
```
{% endtab %}
{% endtabs %}

上面是一个使用生成器的简单示例。要初始化`FlinkKinesisProducer`，用户需要通过`java.util.Properties`实例传递`AWS_REGION`、`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`。实例属性。用户还可以将KPL的配置作为可选参数传入，以定制底层`FlinkKinesisProducer`的KPL。完整的KPL配置和解释列表可以在这里找到。这个例子演示了在AWS区域“us-east-1”产生一个单驱动流。

如果用户没有指定任何KPL配置和值，`FlinkKinesisProducer`将使用KPL的默认配置值，`RateLimit`除外。`RateLimit`限制分片允许的最大看跌率，作为后端限制的百分比。KPL的默认值是150，但是它使KPL过于频繁地抛出`Ratelimitdedexception`，并因此打破Flink sink。因此，`FlinkKinesisProducer`将KPL的默认值改写为100。

它还支持`KinesisSerializationSchema`，而不是`SerializationSchema`。`KinesisSerializationSchema`允许将数据发送到多个流。这是使用`KinesisSerializationSchema.getTargetStream(T element)`方法完成的。如果返回null，将指示生产者将元素写入默认流。否则，将使用返回的流名称。

### 线程模型

从Flink 1.4.0开始，`FlinkKinesisProducer`将其默认底层KPL从每个请求一个线程的模式切换到线程池模式。KPL在线程池模式下使用队列和线程池来执行对Kinesis的请求。这限制了KPL的本机进程可能创建的线程数量，因此大大降低了CPU利用率并提高了效率。**因此，我们强烈建议Flink用户使用线程池模型**。默认的线程池大小是10。用户可以在java.util.Properties实例中设置池大小。其关键字为`ThreadPoolSize`，如上例所示。

通过在java.util.Properties实例中设置`ThreadingModel`和`PER_REQUEST`的键-值对，用户仍然可以切换回每个请求一个线程的模式。如上述示例中注释掉的代码所示。

### 背压

默认情况下，`FlinkKinesisProducer`不会反压。相反，由于速率限制\(每个分片每秒1 MB\)而无法发送的记录被缓冲在一个无限制的队列中，并在它们的RecordTtl过期时被丢弃。

为了避免数据丢失，您可以通过限制内部队列的大小来启用反加压:

```text
// 200 Bytes per record, 1 shard
kinesis.setQueueLimit(500);
```

`queueLimit`的值取决于预期的记录大小。要选择一个好的值，考虑到Kinesis的速度限制为每个分片每秒1MB。如果缓冲的记录值小于一秒，那么队列可能无法满负荷运行。对于默认的`RecordMaxBufferedTime`为100ms，每个分片的队列大小为100kB就足够了。然后可以通过以下方式计算`queueLimit`

```text
queue limit = (number of shards * queue size per shard) / record size
```

例如，对于每个记录200字节和8个分片，队列限制4000是一个很好的起点。如果队列大小限制了吞吐量\(每个切分低于每秒1MB\)，请尝试稍微增加队列限制。

## 使用自定义Kinesis Endpoint

有时需要让Flink作为消费者或生产者对抗Kinesis VPC终端或Kinesalite等非aws的Kinesis终端;这在执行Flink应用程序的功能测试时特别有用。通常由Flink配置中的AWS区域集推断的AWS端点必须通过配置属性覆盖。

 要覆盖AWS终端节点，请设置`AWSConfigConstants.AWS_ENDPOINT`和`AWSConfigConstants.AWS_REGION`属性。该区域将用于对端点URL进行签名。

{% tabs %}
{% tab title="Java" %}
```java
Properties producerConfig = new Properties();
producerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1");
producerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id");
producerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key");
producerConfig.put(AWSConfigConstants.AWS_ENDPOINT, "http://localhost:4567");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val producerConfig = new Properties()
producerConfig.put(AWSConfigConstants.AWS_REGION, "us-east-1")
producerConfig.put(AWSConfigConstants.AWS_ACCESS_KEY_ID, "aws_access_key_id")
producerConfig.put(AWSConfigConstants.AWS_SECRET_ACCESS_KEY, "aws_secret_access_key")
producerConfig.put(AWSConfigConstants.AWS_ENDPOINT, "http://localhost:4567")
```
{% endtab %}
{% endtabs %}

