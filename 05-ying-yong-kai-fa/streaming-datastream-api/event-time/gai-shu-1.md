# 概述

## Event Time / Processing Time / Ingestion Time <a id="event-time--processing-time--ingestion-time"></a>

![](../../../.gitbook/assets/times_clocks.svg)

### 设定时间特征

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime);

// alternatively:
// env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime);
// env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<MyEvent> stream = env.addSource(new FlinkKafkaConsumer09<MyEvent>(topic, schema, props));

stream
    .keyBy( (event) -> event.getUser() )
    .timeWindow(Time.hours(1))
    .reduce( (a, b) -> a.add(b) )
    .addSink(...);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment

env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime)

// alternatively:
// env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime)
// env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

val stream: DataStream[MyEvent] = env.addSource(new FlinkKafkaConsumer09[MyEvent](topic, schema, props))

stream
    .keyBy( _.getUser )
    .timeWindow(Time.hours(1))
    .reduce( (a, b) => a.add(b) )
    .addSink(...)
```
{% endtab %}
{% endtabs %}

## EventTime和Watermark

![](../../../.gitbook/assets/stream_watermark_in_order.svg)

![](../../../.gitbook/assets/stream_watermark_out_of_order.svg)

### 并行流中的Watermark

![](../../../.gitbook/assets/parallel_streams_watermarks.svg)

### 迟到元素

### 闲置资源

### Watermark调试

有关在运行时调试Watermark的信息，请参阅[调试Windows和EventTime](https://ci.apache.org/projects/flink/flink-docs-master/monitoring/debugging_event_time.html)部分。

### 运算符如何处理Watermark

