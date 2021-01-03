# 生成水位线\(Watermarks\)

在本节中，你将了解Flink提供的用于处理**事件时间**时间戳和Watermarks的API 。有关_事件时间_，_处理时间_和_摄取时间的简介_，请参阅 [事件时间的简介](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/event_time.html)。

## Watermarks策略介绍

为了处理`Event Time`，Flink需要知道事件时间戳，这意味着流中的每个元素都需要分配它的事件时间戳。这通常是通过使用`TimestampAssigner`从元素中的某个字段访问/提取时间戳来实现的。

时间戳分配与生成`Watermarks`密切相关，后者告诉系统事件时间的进度。您可以通过指定`WatermarkGenerator`来配置这一点。

Flink API要求`WatermarkStrategy`同时包含`TimestampAssigner`和`WatermarkGenerator`。许多通用策略都可以作为`WatermarkStrategy`的静态方法开箱即用，但用户也可以在需要时构建自己的策略。

这是出于完整性考虑的接口：

```java
public interface WatermarkStrategy<T> extends TimestampAssignerSupplier<T>, WatermarkGeneratorSupplier<T>{

    /**
     * Instantiates a {@link TimestampAssigner} for assigning timestamps according to this
     * strategy.
     */
    @Override
    TimestampAssigner<T> createTimestampAssigner(TimestampAssignerSupplier.Context context);

    /**
     * Instantiates a WatermarkGenerator that generates watermarks according to this strategy.
     */
    @Override
    WatermarkGenerator<T> createWatermarkGenerator(WatermarkGeneratorSupplier.Context context);
}
```

如上所述，通常不需要自己实现这个接口，而是使用`WatermarkStrategy`上的静态助手方法来实现常见的`watermark`策略，或者将自定义的`TimestampAssigner`与`WatermarkGenerator`绑定在一起。例如，要使用有界的无序`watermark`和`lambda`函数作为时间戳分配器，你可以这样使用:

{% tabs %}
{% tab title="Java" %}
```java
WatermarkStrategy
        .<Tuple2<Long, String>>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withTimestampAssigner((event, timestamp) -> event.f0);
```
{% endtab %}

{% tab title="Scala" %}
```scala
WatermarkStrategy
  .forBoundedOutOfOrderness[(Long, String)](Duration.ofSeconds(20))
  .withTimestampAssigner(new SerializableTimestampAssigner[(Long, String)] {
    override def extractTimestamp(element: (Long, String), recordTimestamp: Long): Long = element._1
  })
```
{% endtab %}
{% endtabs %}

指定`TimestampAssigner`是可选的，而且在大多数情况下，实际上你并不想指定。例如，当使用Kafka或Kinesis时，你可以直接从Kafka/Kinesis记录中获取时间戳

我们将在`WatermarkGenerator`稍后的“[编写WatermarkGenerators”](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/event_timestamps_watermarks.html#writing-watermarkgenerators)中介绍该接口。

{% hint style="warning" %}
**注意**：自1970-01-01T00：00：00Z的Java时代以来，时间戳和水位线都指定为毫秒。
{% endhint %}

## 使用Watermarks策略

在Flink应用程序中有两个地方可以使用Watermark策略:1\)直接在`sources`上和2\)在`non-sources`操作之后。

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<MyEvent> stream = env.readFile(
        myFormat, myFilePath, FileProcessingMode.PROCESS_CONTINUOUSLY, 100,
        FilePathFilter.createDefaultFilter(), typeInfo);

DataStream<MyEvent> withTimestampsAndWatermarks = stream
        .filter( event -> event.severity() == WARNING )
        .assignTimestampsAndWatermarks(<watermark strategy>);

withTimestampsAndWatermarks
        .keyBy( (event) -> event.getGroup() )
        .timeWindow(Time.seconds(10))
        .reduce( (a, b) -> a.add(b) )
        .addSink(...);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

val stream: DataStream[MyEvent] = env.readFile(
         myFormat, myFilePath, FileProcessingMode.PROCESS_CONTINUOUSLY, 100,
         FilePathFilter.createDefaultFilter())

val withTimestampsAndWatermarks: DataStream[MyEvent] = stream
        .filter( _.severity == WARNING )
        .assignTimestampsAndWatermarks(<watermark strategy>)

withTimestampsAndWatermarks
        .keyBy( _.getGroup )
        .timeWindow(Time.seconds(10))
        .reduce( (a, b) => a.add(b) )
        .addSink(...)
```
{% endtab %}
{% endtabs %}

使用`WatermarkStrategy`方式获取一个流，并产生带有时间戳记的元素和水印的新流。如果原始流已经具有时间戳和/或水印，则时间戳分配器将覆盖它们。

## 处理空闲源

如果一个输入`split`/`partitions`/`shards`在一段时间内不携带事件，这意味着`WatermarkGenerator`也不会获得任何新的信息来作为Watermark的基础。我们称之为空闲输入或空闲源。这是一个问题，因为您的某些分区可能仍然承载事件。在这种情况下，`Watermark`将被抑制，因为它被计算为所有不同并行`Watermark`的最小值。

要处理这个问题，您可以使用一个水印策略来检测闲置并将输入标记为闲置。WatermarkStrategy为这提供了一个方便的助手:

{% tabs %}
{% tab title="Java" %}
```java
WatermarkStrategy
        .<Tuple2<Long, String>>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withIdleness(Duration.ofMinutes(1));
```
{% endtab %}

{% tab title="Scala" %}
```scala
WatermarkStrategy
  .forBoundedOutOfOrderness[(Long, String)](Duration.ofSeconds(20))
  .withIdleness(Duration.ofMinutes(1))
```
{% endtab %}
{% endtabs %}

## 编写WatermarkGenerators

`TimestampAssigner`是一个从事件中提取字段的简单函数，因此我们不需要详细查看它们。另一方面，`WatermarkGenerator`的编写有点复杂，我们将在接下来的两节中研究如何实现这一点。以下是`WatermarkGenerator`的接口:

```java
/**
 * The {@code WatermarkGenerator} generates watermarks either based on events or
 * periodically (in a fixed interval).
 *
 * <p><b>Note:</b> This WatermarkGenerator subsumes the previous distinction between the
 * {@code AssignerWithPunctuatedWatermarks} and the {@code AssignerWithPeriodicWatermarks}.
 */
@Public
public interface WatermarkGenerator<T> {

    /**
     * Called for every event, allows the watermark generator to examine and remember the
     * event timestamps, or to emit a watermark based on the event itself.
     */
    void onEvent(T event, long eventTimestamp, WatermarkOutput output);

    /**
     * Called periodically, and might emit a new watermark, or not.
     *
     * <p>The interval in which this method is called and Watermarks are generated
     * depends on {@link ExecutionConfig#getAutoWatermarkInterval()}.
     */
    void onPeriodicEmit(WatermarkOutput output);
}
```

有两种不同的水印生成方式:_`periodic`\(_周期型\)和\(_`punctuated`_\)标点型。

周期型生成器通常通过观察传入事件`onEvent()` ，然后在框架调用时发出水印`onPeriodicEmit()`。

标点型生成器将查看事件`onEvent()`并等待特殊的 _标记事件_或_标点_，这些_事件_或_标点_在流中携带`Watermark`信息。当看到这些事件之一时，它将立即发出`Watermark`。通常，标点型生成器不会发出来自`onPeriodicEmit()`的`Watermark`。

接下来，我们将研究如何为每种样式实现生成器。

### 编写周期型Watermark

周期型生成器观察流事件并周期性地生成`Watermark`水印（可能取决于流元素，或者纯粹基于处理时间）。

通过`ExecutionConfig.setAutoWatermarkInterval(...)`定义生成`Watermark`的间隔（每_n_毫秒）。生成器的`onPeriodicEmit()`方法每次都会被调用，并且如果返回的`Watermark`非空且大于先前的`Watermark`，则将发出新的`Watermark`。

在这里，我们显示了两个使用定期水印生成的`Watermark`生成器的简单示例。请注意，Flink附带了 `BoundedOutOfOrdernessWatermarks`，它的`WatermarkGenerator`工作原理与以下`BoundedOutOfOrdernessGenerator`所示类似。您可以[在这里](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/event_timestamp_extractors.html#assigners-allowing-a-fixed-amount-of-lateness)阅读有关使用它的[信息](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/event_timestamp_extractors.html#assigners-allowing-a-fixed-amount-of-lateness)。

{% tabs %}
{% tab title="Java" %}
```java
/**
 * This generator generates watermarks assuming that elements arrive out of order,
 * but only to a certain degree. The latest elements for a certain timestamp t will arrive
 * at most n milliseconds after the earliest elements for timestamp t.
 */
public class BoundedOutOfOrdernessGenerator implements AssignerWithPeriodicWatermarks<MyEvent> {

    private final long maxOutOfOrderness = 3500; // 3.5 seconds

    private long currentMaxTimestamp;

    @Override
    public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
        long timestamp = element.getCreationTime();
        currentMaxTimestamp = Math.max(timestamp, currentMaxTimestamp);
        return timestamp;
    }

    @Override
    public Watermark getCurrentWatermark() {
        // return the watermark as current highest timestamp minus the out-of-orderness bound
        return new Watermark(currentMaxTimestamp - maxOutOfOrderness);
    }
}

/**
 * This generator generates watermarks that are lagging behind processing time by a fixed amount.
 * It assumes that elements arrive in Flink after a bounded delay.
 */
public class TimeLagWatermarkGenerator implements AssignerWithPeriodicWatermarks<MyEvent> {

	private final long maxTimeLag = 5000; // 5 seconds

	@Override
	public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
		return element.getCreationTime();
	}

	@Override
	public Watermark getCurrentWatermark() {
		// return the watermark as current time minus the maximum time lag
		return new Watermark(System.currentTimeMillis() - maxTimeLag);
	}
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
/**
 * This generator generates watermarks assuming that elements arrive out of order,
 * but only to a certain degree. The latest elements for a certain timestamp t will arrive
 * at most n milliseconds after the earliest elements for timestamp t.
 */
class BoundedOutOfOrdernessGenerator extends AssignerWithPeriodicWatermarks[MyEvent] {

    val maxOutOfOrderness = 3500L // 3.5 seconds

    var currentMaxTimestamp: Long = _

    override def extractTimestamp(element: MyEvent, previousElementTimestamp: Long): Long = {
        val timestamp = element.getCreationTime()
        currentMaxTimestamp = max(timestamp, currentMaxTimestamp)
        timestamp
    }

    override def getCurrentWatermark(): Watermark = {
        // return the watermark as current highest timestamp minus the out-of-orderness bound
        new Watermark(currentMaxTimestamp - maxOutOfOrderness)
    }
}

/**
 * This generator generates watermarks that are lagging behind processing time by a fixed amount.
 * It assumes that elements arrive in Flink after a bounded delay.
 */
class TimeLagWatermarkGenerator extends AssignerWithPeriodicWatermarks[MyEvent] {

    val maxTimeLag = 5000L // 5 seconds

    override def extractTimestamp(element: MyEvent, previousElementTimestamp: Long): Long = {
        element.getCreationTime
    }

    override def getCurrentWatermark(): Watermark = {
        // return the watermark as current time minus the maximum time lag
        new Watermark(System.currentTimeMillis() - maxTimeLag)
    }
}
```
{% endtab %}
{% endtabs %}

### 编写标点型水位线

标点型`Watermark`生成器将观察事件流，并在看到携带`Watermark`信息的特殊元素时发出`Watermark`。

这就是你如何实现一个标点型生成器，每当事件表明它带有某个标记时就会发出`Watermark`:

{% tabs %}
{% tab title="Java" %}
```java
public class PunctuatedAssigner implements AssignerWithPunctuatedWatermarks<MyEvent> {

	@Override
	public long extractTimestamp(MyEvent element, long previousElementTimestamp) {
		return element.getCreationTime();
	}

	@Override
	public Watermark checkAndGetNextWatermark(MyEvent lastElement, long extractedTimestamp) {
		return lastElement.hasWatermarkMarker() ? new Watermark(extractedTimestamp) : null;
	}
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class PunctuatedAssigner extends AssignerWithPunctuatedWatermarks[MyEvent] {

	override def extractTimestamp(element: MyEvent, previousElementTimestamp: Long): Long = {
		element.getCreationTime
	}

	override def checkAndGetNextWatermark(lastElement: MyEvent, extractedTimestamp: Long): Watermark = {
		if (lastElement.hasWatermarkMarker()) new Watermark(extractedTimestamp) else null
	}
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
_注意：_可以在每个事件上生成水位线。然而，因为每个水位线在下游引起一些计算，所以过多的水位线会降低性能。
{% endhint %}

## Watermarks策略和Kafka连接器

当使用Apache Kafka作为数据源时，每个Kafka分区可能有一个简单的事件时间模式\(升序时间戳或有界的无序\)。然而，当使用来自Kafka的流时，多个分区常常被并行地使用，将来自分区的事件交织在一起，并破坏每个分区的模式\(这是Kafka的消费者客户端工作方式所固有的\)

在这种情况下，您可以使用Flink支持kafka分区的水位线生成。使用该特性，每个Kafka分区在Kafka使用者内部生成水位线，每个分区的水位线合并的方式与流shuffle时合并水位线的方式相同。

例如，如果事件时间戳严格按每个Kafka分区升序，则使用[升序时间戳水位线生成器](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamp_extractors.html#assigners-with-ascending-timestamps)生成每分区水位线 将产生完美的整体水位线。

下图显示了如何使用`per-Kafka-partition`水位线生成，以及在这种情况下水位线如何通过流数据流传播。

{% tabs %}
{% tab title="Java" %}
```java
FlinkKafkaConsumer09<MyType> kafkaSource = new FlinkKafkaConsumer09<>("myTopic", schema, props);
kafkaSource.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<MyType>() {

    @Override
    public long extractAscendingTimestamp(MyType element) {
        return element.eventTimestamp();
    }
});

DataStream<MyType> stream = env.addSource(kafkaSource);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val kafkaSource = new FlinkKafkaConsumer09[MyType]("myTopic", schema, props)
kafkaSource.assignTimestampsAndWatermarks(new AscendingTimestampExtractor[MyType] {
    def extractAscendingTimestamp(element: MyType): Long = element.eventTimestamp
})

val stream: DataStream[MyType] = env.addSource(kafkaSource)
```
{% endtab %}
{% endtabs %}

![](../../../.gitbook/assets/parallel_kafka_watermarks.svg)

## 操作符如何处理Watermarks

作为一般规则，操作符在将给定的`Watermark`转发到下游之前需要完全处理它。例如，`WindowOperator`将首先评估应触发的所有窗口，只有在产生了所有由水印触发的输出之后，水印本身才会被发送到下游。换句话说，由于`Watermark`的出现而产生的所有元素都将在`Watermark`之前发出。

相同的规则适用于`TwoInputStreamOperator`。但是，在这种情况下，操作符的当前水印被定义为其两个输入的最小值。

该行为的细节由 `OneInputStreamOperator#processWatermark`， `TwoInputStreamOperator#processWatermark1`和 `TwoInputStreamOperator#processWatermark2`方法的实现定义。

## 不建议使用的AssignerWithPeriodicWatermarks和AssignerWithPunctuatedWatermarks

在引入当前`WatermarkStrategy`， `TimestampAssigner`以及`WatermarkGenerator`的抽象之前，Flink使用`AssignerWithPeriodicWatermarks`和`AssignerWithPunctuatedWatermarks` 。你仍可在API中看到它们，但建议使用新接口，因为它们提供了更清晰的关注点分离，还统一了Watermarks生成的周期样式和标点样式。

