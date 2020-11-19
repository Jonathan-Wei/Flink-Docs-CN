# 内置水位线生成器

如[生成水位线中所述](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/event_timestamps_watermarks.html)，Flink提供了抽象，允许程序员分配自己的时间戳并发出自己的水位线。更具体地说，可以通过实现`WatermarkGenerator`接口来做到这一点 。

为了进一步简化此类任务的编程工作，Flink附带了一些预先实现的时间戳分配器。本节提供了它们的列表。除了开箱即用的功能外，它们的实现还可以作为自定义实现的示例。

## 单调自增时间戳

_定期_生成水位线的最简单的特殊情况是给定源任务看到的时间戳以升序出现。在这种情况下，当前时间戳始终可以充当水位线，因为没有更早的时间戳会到达。

请注意，_每个并行数据源任务_仅需要使时间戳记递增。例如，如果在一个特定的设置中，一个并行数据源实例读取一个Kafka分区，则仅在每个Kafka分区内将时间戳递增是必要的。每当对并行流进行混洗，合并，连接或合并时，Flink的水位线合并机制将生成正确的水位线。

{% tabs %}
{% tab title="Java" %}
```text
WatermarkStrategy.forMonotonousTimestamps();
```
{% endtab %}

{% tab title="Scala" %}
```text
WatermarkStrategy.forMonotonousTimestamps()
```
{% endtab %}
{% endtabs %}

## 定额延迟

 周期性水位线生成的另一个示例是水位线在流中看到的最大（事件时间）时间戳落后固定时间量的情况。这种情况涵盖了预先知道流中可能遇到的最大延迟的场景，例如，当创建包含时间戳的元素的自定义源时，该时间戳在固定的时间段内传播以进行测试。对于这些情况，Flink提供了`BoundedOutOfOrdernessWatermarks` 生成器，该生成器将参数用作参数`maxOutOfOrderness`，即，在计算给定窗口的最终结果时，允许元素延迟到被忽略之前的最长时间。延迟与的结果相对应`t - t_w`，其中`t`元素的（事件时间）时间戳和`t_w`先前水位线的时间戳 。如果`lateness > 0`那么该元素将被认为是较晚的元素，默认情况下，在为其相应窗口计算作业结果时将其忽略。有关使用延迟元素的更多信息，请参见有关[允许延迟](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/operators/windows.html#allowed-lateness)的文档。

{% tabs %}
{% tab title="First Tab" %}
```text
WatermarkStrategy.forBoundedOutOfOrderness(Duration.ofSeconds(10));
```
{% endtab %}

{% tab title="Second Tab" %}
```text
WatermarkStrategy
  .forBoundedOutOfOrderness(Duration.ofSeconds(10))
```
{% endtab %}
{% endtabs %}

