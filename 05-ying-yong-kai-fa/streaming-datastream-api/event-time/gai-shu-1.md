# 概述

 在本节中，您将学习编写时间感知的Flink程序。请看一下[及时流处理](https://ci.apache.org/projects/flink/flink-docs-release-1.11/concepts/timely-stream-processing.html)以了解及时流处理背后的概念。

 有关如何在Flink程序中使用时间的信息，请参阅 [windowing](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/operators/windows.html)和 [ProcessFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/operators/process_function.html)。

使用_事件时间_处理的先决条件是设置正确的_时间特性_。该设置定义了数据流源的行为方式（例如，是否分配时间戳），以及窗口操作应使用什么时间概念，例如`KeyedStream.timeWindow(Time.seconds(30))`。

你可以使用`StreamExecutionEnvironment.setStreamTimeCharacteristic()`设置时间特性 ：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<MyEvent> stream = env.addSource(new FlinkKafkaConsumer<MyEvent>(topic, schema, props));

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

env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

val stream: DataStream[MyEvent] = env.addSource(new FlinkKafkaConsumer[MyEvent](topic, schema, props))

stream
    .keyBy( _.getUser )
    .timeWindow(Time.hours(1))
    .reduce( (a, b) => a.add(b) )
    .addSink(...
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()

env.set_stream_time_characteristic(TimeCharacteristic.EventTime)

# alternatively:
# env.set_stream_time_characteristic(TimeCharacteristic.IngestionTime)
# env.set_stream_time_characteristic(TimeCharacteristic.ProcessingTime)
```
{% endtab %}
{% endtabs %}

请注意，为了在事件时间中运行此示例，程序需要使用直接为数据定义事件时间并本身发出水印的源，或者程序必须在源之后注入时间戳分配器和水印生成器。这些函数描述了如何访问事件时间戳，以及事件流呈现出何种程度的乱序。

