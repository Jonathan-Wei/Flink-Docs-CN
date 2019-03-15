# 预定义Timestamp Extractors / Watermark Emitters

正如时间戳和水位线处理中所描述的，Flink提供了抽象，允许程序员分配自己的时间戳并发出自己的水位线。更具体地说，可以通过实现其中一个`AssignerWithPeriodicWatermarks`和`AssignerWithPunctuatedWatermarks`接口来实现，具体取决于用例。简而言之，第一个会周期性地发出水位线，而第二个会根据传入记录的某些属性发出水位线，例如每当流中遇到特殊元素时。

为了进一步简化此类任务的编程工作，Flink提供了一些预先实现的时间戳分配器。本节提供了它们的列表。除了它们的开箱即用功能之外，它们的实现还可以作为定制实现的示例。

## **具有递增时间戳的分配者**

定期水位线生成的最简单的特殊情况是给定源任务看到的时间戳按升序发生的情况。 在这种情况下，当前时间戳始终可以充当水位线，因为没有更早的时间戳会到达。

{% hint style="info" %}
注意，只需要将每个并行数据源任务的时间戳递增。
{% endhint %}

例如，如果在特定设置中，一个并行数据源实例读取一个Kafka分区，则只需要在每个Kafka分区中时间戳递增。Flink的水位线合并机制将在并行流被打乱、联合、连接或合并时生成正确的水位线。机制将在并行流被打乱、联合、连接或合并时生成正确的水位线。

{% tabs %}
{% tab title="Java" %}
```java
DataStream<MyEvent> stream = ...

DataStream<MyEvent> withTimestampsAndWatermarks =
    stream.assignTimestampsAndWatermarks(new AscendingTimestampExtractor<MyEvent>() {

        @Override
        public long extractAscendingTimestamp(MyEvent element) {
            return element.getCreationTime();
        }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
val stream: DataStream[MyEvent] = ...

val withTimestampsAndWatermarks = stream.assignAscendingTimestamps( _.getCreationTime )
```
{% endtab %}
{% endtabs %}

## 允许固定数量的迟到的分配者

定期水位线生成的另一个例子是当水位线滞后于在流中看到的最大（事件 - 时间）时间戳一段固定的时间。这种情况包括预先知道流中可能遇到的最大延迟的情况，例如，在创建包含时间戳的元素的自定义源时，这些元素在固定的时间段内传播以进行测试。对于这些情况，Flink提供`BoundedOutOfOrdernessTimestampExtractor`，它将`maxOutOfOrderness`作为参数，即在计算给定窗口的最终结果时，在忽略元素之前允许元素延迟的最长时间。延迟对应于t-t\_w的结果，其中t是元素的（事件 - 时间）时间戳，t\_w是前一个水位线的时间戳。如果lateness&gt; 0，那么该元素被认为是迟的，并且在计算其对应窗口的作业结果时默认被忽略。有关使用延迟元素的更多信息，请参阅有关允许延迟的文档。

{% tabs %}
{% tab title="Java" %}
```java
DataStream<MyEvent> stream = ...

DataStream<MyEvent> withTimestampsAndWatermarks =
    stream.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor<MyEvent>(Time.seconds(10)) {

        @Override
        public long extractTimestamp(MyEvent element) {
            return element.getCreationTime();
        }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
val stream: DataStream[MyEvent] = ...

val withTimestampsAndWatermarks = stream.assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[MyEvent](Time.seconds(10))( _.getCreationTime ))
```
{% endtab %}
{% endtabs %}

