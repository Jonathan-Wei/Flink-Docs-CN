# 生成水位线\(Watermarks\)

此部分与在**事件时间**运行的程序相关。有关_事件时间_， _处理时间_和_摄入时间_的[介绍](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html)，请参阅[事件时间简介](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html)。

要处理_事件时间_，流式传输程序需要设置相应地_时间特性_。

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```
{% endtab %}

{% tab title="" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()
env.set_stream_time_characteristic(TimeCharacteristic.EventTime)
```
{% endtab %}
{% endtabs %}

## 分配时间戳

为了处理_事件时间_，Flink需要知道事件的_时间戳_，这意味着流中的每个元素都需要_分配_其事件时间戳。这通常通过从元素中的某个字段访问/提取时间戳来完成。

时间戳分配与生成水位线密切相关，水位线告诉系统事件时间的进展。

有两种方法可以分配时间戳并生成水位线：

1. 直接在数据流源中
2. 通过时间戳分配器/水位线生成器：在Flink中，时间戳分配器还定义要发出的水位线

{% hint style="info" %}
自1970-01-01T00：00：00Z的Java时代以来，时间戳和水位线都指定为毫秒。
{% endhint %}

### 带时间戳和水位线\(Watermark\)的源函数\(SourceFunction\)

流源\(Source\)可以直接为它们生成的元素分配时间戳，它们还可以发出水位线。完成此操作后，不需要时间戳分配器。注意，如果使用了时间戳分配器，则源提供的任何时间戳和水位线都将被覆盖。

要直接将时间戳分配给源中的元素，源必须使用`SourceContext`上的`collectWithTimestamp(…)`方法。要生成水位线，源必须调用`emitWatermark(Watermark)`函数。

下面是一个\(非检查点\)源的简单示例，它分配时间戳并生成水位线:

{% tabs %}
{% tab title="Java" %}
```java
@Override
public void run(SourceContext<MyType> ctx) throws Exception {
	while (/* condition */) {
		MyType next = getNext();
		ctx.collectWithTimestamp(next, next.getEventTimestamp());

		if (next.hasWatermarkTime()) {
			ctx.emitWatermark(new Watermark(next.getWatermarkTime()));
		}
	}
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
override def run(ctx: SourceContext[MyType]): Unit = {
	while (/* condition */) {
		val next: MyType = getNext()
		ctx.collectWithTimestamp(next, next.eventTimestamp)

		if (next.hasWatermarkTime) {
			ctx.emitWatermark(new Watermark(next.getWatermarkTime))
		}
	}
}
```
{% endtab %}
{% endtabs %}

### 时间戳分配器/水位线\(Watermark\)生成器

时间戳分配程序获取一个流并生成一个带有时间戳元素和水位线的新流。如果原始流已经有时间戳和/或水位线，则时间戳分配程序将覆盖它们。

时间戳分配器通常在数据源之后立即指定，但并非严格要求这样做。例如，常见的模式是在时间戳分配器之前解析（_MapFunction_）和过滤（_FilterFunction_）。在任何情况下，需要在事件时间的第一个操作之前指定时间戳分配器（例如第一个窗口操作）。作为一种特殊情况，当使用Kafka作为流式传输作业的源时，Flink允许在源（或消费者）本身内指定时间戳分配器/水位线发射器。有关如何操作的更多信息，请参阅 [Kafka Connector文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/kafka.html)。

{% hint style="info" %}
**注意：**本节的其余部分介绍了程序员必须实现的主要接口，以便创建自己的时间戳提取器/水位线发射器。要查看Flink附带的预先实现的提取器，请参阅 [预定义的时间戳提取器/](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamp_extractors.html)水位线[发射器](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamp_extractors.html)页面。
{% endhint %}

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
        .assignTimestampsAndWatermarks(new MyTimestampsAndWatermarks());

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
        .assignTimestampsAndWatermarks(new MyTimestampsAndWatermarks())

withTimestampsAndWatermarks
        .keyBy( _.getGroup )
        .timeWindow(Time.seconds(10))
        .reduce( (a, b) => a.add(b) )
        .addSink(...)
```
{% endtab %}
{% endtabs %}

#### 周期性水位线\(Watermark\)

`AssignerWithPeriodicWatermarks` 分配时间戳并定期生成水位线（可能取决于流元素，或纯粹基于处理时间）。

生成水位线的时间间隔\(每n毫秒\)是通过`ExecutionConfig.setAutoWatermarkInterval(…)`定义的。每次都会调用assigner的`getCurrentWatermark()`方法，如果返回的水位线是非空且比前一个水位线大，则会发出一个新的水位线。

这里我们展示了两个使用周期性水位线生成的时间戳分配器的简单示例。请注意，Flink附带了一个`BoundedOutOfOrdernessTimestampExtractor`，类似于下面所示的`BoundedOutOfOrdernessGenerator`，您可以在[这里](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamp_extractors.html#assigners-allowing-a-fixed-amount-of-lateness)阅读相关内容。

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

#### 间断性水位线

若要在某个事件指示可能生成新水位线时生成水位线，请使用assignerwith标点水位线。对于这个类，Flink首先调用`extractTimestamp(…)`方法为元素分配一个时间戳，然后立即调用该元素上的`checkAndGetNextWatermark(…)`方法。

`checkAndGetNextWatermark(…)`方法将传递在`extractTimestamp(…)`方法中分配的时间戳，并可以决定是否要生成水位线。每当`checkAndGetNextWatermark(…)`方法返回一个非空水位线，并且该水位线大于最新的前一个水位线时，就会发出新的水位线。

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

## 每个Kafka分区的时间戳

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

