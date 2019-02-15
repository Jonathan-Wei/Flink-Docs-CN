# 概述

## Event Time / Processing Time / Ingestion Time <a id="event-time--processing-time--ingestion-time"></a>

Flink在流媒体程序中支持不同的时间概念:

**处理时间\(Processing Time\)：**处理时间是指执行相应操作的机器的系统时间。

当流程序在处理时间上运行时，所有基于时间的操作\(如时间窗口\)将使用运行各自操作符的机器的系统时间。每小时处理时间窗口将包括在系统时钟指示整小时之间到达特定操作符的所有记录。例如，如果应用程序在上午9:15开始运行，第一个小时处理时间窗口将包括上午9:15到10:00之间处理的事件，下一个窗口将包括上午10:00到11:00之间处理的事件，依此类推。

处理时间是最简单的时间概念，不需要流和机器之间的协调。它提供最佳性能和最低延迟。但是，在分布式和异步环境中，处理时间不提供确定性，因为它容易受到记录到达系统的速度（例如从消息队列）到记录在系统内的操作符之间流动的速度的影响，并中断（预定或其他）。

**事件时间\(Event Time\)：**事件时间是每个单独事件在其生成设备上发生的时间。这个时间通常在记录输入Flink之前嵌入到记录中，并且可以从每个记录中提取事件时间戳。在事件时间中，时间的进展取决于数据，而不是任何挂钟。事件时间程序必须指定如何生成事件时间水印\(WaterMark\)，这是表示事件时间进度的机制。这种水印机制将在[下一节](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html#event-time-and-watermarks)中进行描述。

在一个完美的世界中，事件时间处理将产生完全一致和确定的结果，无论事件何时到达，或它们的顺序如何。但是，除非事件是按顺序\(通过时间戳\)到达的，否则在等待无序事件时，事件时间处理会引起一些延迟。由于只能等待有限的一段时间，这就限制了事件时间应用程序的确定性。

假设所有数据都已到达，事件时间操作将按照预期的方式进行，即使在处理无序或延迟的事件或重新处理历史数据时，也会产生正确和一致的结果。例如，每小时事件时间窗口将包含所有带有属于该小时的事件时间戳的记录，而不管它们到达的顺序如何，也不管它们是在什么时候处理的。\(有关延迟事件的更多信息，请参见“[延迟事件](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html#late-elements)”一节\)。

请注意，有时当事件时间程序实时处理实时数据时，它们将使用一些处理时间操作，以确保及时地进行处理。

**摄入时间\(Ingestion time\)**:摄入时间是事件进入Flink的时间。在源操作符中，每个记录以时间戳的形式获取源的当前时间，基于时间的操作\(如时间窗口\)引用该时间戳。

从概念上讲，摄入时间介于事件时间和处理时间之间。与处理时间相比，它稍微昂贵一些，但是提供了更可预测的结果。由于摄入时间使用稳定的时间戳\(在源上分配一次\)，记录上的不同窗口操作将引用相同的时间戳，而在处理时间中，每个窗口操作人员可以将记录分配到不同的窗口\(基于本地系统时钟和任何传输延迟\)。

与事件时间相比，摄入时间程序不能处理任何无序事件或延迟数据，但程序不必指定如何生成水印。

在内部，摄食时间与事件时间非常相似，但具有自动时间戳分配和自动水印生成。

![](../../../.gitbook/assets/times_clocks.svg)

### 设定时间特征

Flink DataStream程序的第一部分通常设置基本_时间特性_。该设置定义了数据流源的行为方式（例如，它们是否将分配时间戳），以及窗口操作应该使用的时间概念`KeyedStream.timeWindow(Time.seconds(30))`。

下面的示例显示了一个Flink程序，它在每小时时间窗口中聚合事件。窗户的性能与时间特征相适应。

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

请注意，为了在事件时间运行此示例，程序需要使用直接为数据定义事件时间的源并自行发出水印，或者程序必须在源之后注入时间戳分配器和水印生成器。 这些函数描述了如何访问事件时间戳，以及事件流表现出的无序程度。

以下部分描述了时间戳和水印背后的一般机制。 有关如何在Flink DataStream API中使用时间戳分配和水印生成的指南，请参阅[生成时间戳/水印](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamps_watermarks.html)。

## EventTime和Watermark

_注意：Flink实现了数据流模型中的许多技术。有关事件时间和水印的详细介绍，请查看以下文章。_

* [Streaming 101](https://www.oreilly.com/ideas/the-world-beyond-batch-streaming-101) by Tyler Akidau
* The [Dataflow Model paper](https://research.google.com/pubs/archive/43864.pdf)

支持事件时间的流处理器需要一种方法来衡量事件时间的进度。 例如，当事件时间超过一小时结束时，需要通知构建每小时窗口的窗口操作符，以便操作员可以关闭正在进行的窗口。

事件时间可以独立于处理时间（由挂钟测量）进行。 例如，在一个程序中，操作符的当前事件时间可能略微滞后于处理时间（考虑到接收事件的延迟），而两者都以相同的速度进行。 另一方面，通过快速转发Kafka主题\(或另一个消息队列\)中已经缓冲的历史数据，另一个流程序可能只需要几秒钟的处理，就可以处理数周的事件时间。

Flink中测量事件时间进程的机制是水印。水印作为数据流的一部分流动并带有时间戳_t_。一个水印\(t\)宣告事件时间已达到时间t的流,这意味着不应该有来自流的具有时间戳t'&lt;= t的元素（即具有更长或等于水印的时间戳的事件）

下图展示了带有\(逻辑的\)时间戳和内联水印的事件流。在这个例子中，事件是按顺序排列的\(相对于它们的时间戳\)，这意味着水印只是流中的周期性标记。

![](../../../.gitbook/assets/stream_watermark_in_order.svg)

对于无序流，水印是至关重要的，如下所示，其中事件不是按照时间戳排序的。一般来说，水印是一种声明，通过流中的该点，到某个时间戳的所有事件都应该到达。一旦水印到达操作符，操作符可以将其内部事件时间时钟推进到水印的值。

![](../../../.gitbook/assets/stream_watermark_out_of_order.svg)

{% hint style="info" %}
请注意，事件时间由新创建的流元素\(或多个元素\)继承，这些流元素或来自生成它们的事件，或来自触发这些元素创建的水印。
{% endhint %}

### 并行流中的Watermark

在源函数处或之后生成水印。 源函数的每个并行子任务通常独立地生成其水印。 这些水印定义了该特定并行源的事件时间。

当水印流经流媒体程序时，它们会在他们到达的操作符处推进事件时间。 每当操作符提前其事件时间时，它为其后继操作符生成下游的新水印。

一些操作符使用多个输入流; 例如，一个`union`，或者跟随`keyBy（...）`或`partition（...）`函数的运算符。 这样一个操作符的当前事件时间是其输入流的事件时间的最小值。当输入流更新它们的事件时间时，操作符也会更新。

下图展示了一个事件和水印流经并行流的示例，以及跟踪事件时间的操作符。

![](../../../.gitbook/assets/parallel_streams_watermarks.svg)

{% hint style="info" %}
请注意，Kafka源支持每个分区的水印，可以在[此处](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamps_watermarks.html#timestamps-per-kafka-partition)阅读更多信息。
{% endhint %}

### 滞后元素

某些元素可能会违反水印条件，这意味着即使在水印\(t\)发生之后，还会出现更多带有时间戳t ' &lt;= t的元素。事实上，在许多实际的设置中，某些元素可以被任意延迟，因此不可能指定某个事件时间戳的所有元素发生的时间。此外，即使延迟是有界的，将水印延迟太多通常也是不可取的，因为它会在事件时间窗的评估中造成太多的延迟。

由于这个原因，流媒体程序可能会显式地预期一些延迟元素。延迟元素是在系统的事件时间时钟\(如水印所示\)已经超过延迟元素的时间戳的时间之后到达的元素。有关如何在事件时间窗口中处理延迟元素的更多信息，请参见“[允许延迟](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html#allowed-lateness)”。

### 闲置资源

目前，对于纯事件时间水印生成器，如果没有要处理的元素，则水印无法进行处理。 这意味着在输入数据存在间隙的情况下，事件时间将不会进行，例如窗口操作符将不会被触发，因此现有窗口将不能产生任何输出数据。

为了避免这种情况，可以使用定期水印分配器，它不仅基于元素时间戳进行分配。 示例解决方案可以是在不观察新事件一段时间之后切换到使用当前处理时间作为时间基础的分配器。

可以使用`SourceFunction.SourceContext #markAsTemporarilyIdle`将源标记为空闲。 有关详细信息，请参阅此方法的Javadoc以及`StreamStatus`。

### Watermark调试

有关在运行时调试Watermark的信息，请参阅[调试Windows和EventTime](https://ci.apache.org/projects/flink/flink-docs-master/monitoring/debugging_event_time.html)部分。

### 操作符如何处理Watermark

作为一般规则，操作符需要在向下游转发之前完全处理给定的水印。 例如，`WindowOperator`将首先评估应该触发哪些窗口，并且只有在产生由水印触发的所有输出之后，水印本身才会被发送到下游。 换句话说，由于出现水印而产生的所有元素将在水印之前发出。

同样的规则适用于`TwoInputStreamOperator`。 但是，在这种情况下，操作符的当前水印被定义为其两个输入的最小值。

此行为的详细信息由`OneInputStreamOperator＃processWatermark`，`TwoInputStreamOperator＃processWatermark1`和`TwoInputStreamOperator #processWatermark2`方法的实现定义。

