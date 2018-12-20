# API迁移向导

## 从Flink 1.3+迁移到Flink 1.7

### 序列化器快照的API更改

这主要适用于为其状态实现自定义类型序列化器的用户。

旧的TypeSerializerConfigSnapshot抽象现在已经被弃用，并且在将来将被完全删除，取而代之的是新的TypeSerializerSnapshot。有关如何迁移的详细信息和指南，请参见[从Flink 1.7之前的已弃用序列化器快照api迁移](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/custom_serialization.html#migration-from-deprecated-serializer-snapshot-apis-before-Flink-1.7)。

## 从Flink 1.2迁移到Flink 1.3

自Flink 1.2以来，已经更改了一些api。大多数更改都记录在它们的特定文档中。以下是升级到Flink 1.3时的API更改和迁移细节链接的综合列表。

### `TypeSerializer` 接口变更

这主要适用于为其状态实现自定义类型序列化器的用户。

自Flink 1.3以来，又添加了两个与保存点恢复之间的序列化器兼容性相关的方法。有关如何实现这些方法的详细信息，请参见[处理序列化器升级和兼容性](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/custom_serialization.html#handling-serializer-upgrades-and-compatibility)。

### ProcessFunction始终是RichFunction

在Flink 1.2中，引入了ProcessFunction及其丰富的变体RichProcessFunction。自从Flink 1.3之后，RichProcessFunction被移除，ProcessFunction现在始终是一个RichFunction，可以访问生命周期方法和运行时上下文。

### Flink CEP库API更改

Flink 1.3中的CEP库附带了许多新功能，这些功能导致API发生了一些变化。有关详细信息，请访问[CEP迁移文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/libs/cep.html#migrating-from-an-older-flink-version)。

### 从Flink核心工件中删除了Logger依赖项

在Flink 1.3中，为了确保用户可以使用他们自己的自定义日志记录框架，核心Flink工件现在已经清除了特定的日志记录器依赖项。

示例和quickstart原型已经指定了记录器，不应该受到影响。对于其他自定义项目，请确保添加日志记录器依赖项。例如，你可以在maven工程的pom.xml文件中添加:

```markup
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.7</version>
</dependency>

<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

## Flink1.1迁移到Flink1.2

如[状态文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html)中所述，Flink有两种类型的状态：**键控**状态 和**非键控**状态（也称为**操作员**状态）。这两种类型都可用于运算符和用户定义的函数。本文档将指导您完成将Flink 1.1功能代码迁移到Flink 1.2的过程，并介绍Flink 1.2中引入的一些重要内部更改，这些更改涉及Flink 1.1中对齐窗口运算符的弃用（请参阅[对齐处理时间窗口运算符](https://ci.apache.org/projects/flink/flink-docs-master/dev/migration.html#aligned-processing-time-window-operators)）。

迁移过程将有两个目标：

1. 允许您的功能利用Flink 1.2中引入的新功能，例如重新缩放，
2. 确保您的新Flink 1.2作业能够从其Flink 1.1前身生成的保存点恢复执行。

按照本指南中的步骤操作后，您可以将正在运行的作业从Flink 1.1迁移到Flink 1.2，只需在Flink 1.1作业中使用[savepoint](https://ci.apache.org/projects/flink/flink-docs-master/ops/state/savepoints.html)并将其作为起点提供给Flink 1.2作业。这将允许Flink 1.2作业从其Flink 1.1前任中断的位置恢复执行。

### 用户函数示例

作为本文其余部分的运行示例，我们将使用CountMapper和BufferingSink函数。第一个是具有**键控**状态的函数的示例，而第二个是具有**非键控**状态的函数。Flink 1.1中上述两个功能的代码如下：

```java
public class CountMapper extends RichFlatMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>> {

    private transient ValueState<Integer> counter;

    private final int numberElements;

    public CountMapper(int numberElements) {
        this.numberElements = numberElements;
    }

    @Override
    public void open(Configuration parameters) throws Exception {
        counter = getRuntimeContext().getState(
            new ValueStateDescriptor<>("counter", Integer.class, 0));
    }

    @Override
    public void flatMap(Tuple2<String, Integer> value, Collector<Tuple2<String, Integer>> out) throws Exception {
        int count = counter.value() + 1;
        counter.update(count);

        if (count % numberElements == 0) {
            out.collect(Tuple2.of(value.f0, count));
            counter.update(0); // reset to 0
        }
    }
}

public class BufferingSink implements SinkFunction<Tuple2<String, Integer>>,
    Checkpointed<ArrayList<Tuple2<String, Integer>>> {

    private final int threshold;

    private ArrayList<Tuple2<String, Integer>> bufferedElements;

    BufferingSink(int threshold) {
        this.threshold = threshold;
        this.bufferedElements = new ArrayList<>();
    }

    @Override
    public void invoke(Tuple2<String, Integer> value) throws Exception {
        bufferedElements.add(value);
        if (bufferedElements.size() == threshold) {
            for (Tuple2<String, Integer> element: bufferedElements) {
                // send it to the sink
            }
            bufferedElements.clear();
        }
    }

    @Override
    public ArrayList<Tuple2<String, Integer>> snapshotState(
        long checkpointId, long checkpointTimestamp) throws Exception {
        return bufferedElements;
    }

    @Override
    public void restoreState(ArrayList<Tuple2<String, Integer>> state) throws Exception {
        bufferedElements.addAll(state);
    }
}
```

### 状态API迁移

### 对齐处理时间窗口操作符

{% hint style="info" %}
虽然不推荐使用，但是您仍然可以通过专门为此目的引入的windows赋值器来使用Flink 1.2中对齐的窗口操作符。这些assigners分别是`SlidingAlignedProcessingTimeWindows`和`TumblingAlignedProcessingTimeWindows`assigners，分别用于滑动窗口和滚动窗口。使用对齐窗口的Flink 1.2作业必须是一个新作业，因为在使用这些操作符时无法从Flink 1.1保存点恢复执行。
{% endhint %}

{% hint style="danger" %}
对齐的窗口运算符不提供**重新缩放**功能，**也不**提供与Flink 1.1的**向后兼容性**
{% endhint %}

在Flink 1.2中使用对齐窗口运算符的代码如下所示：

{% tabs %}
{% tab title="Java" %}
```text
// for tumbling windows
DataStream<Tuple2<String, Integer>> window1 = source
    .keyBy(0)
    .window(TumblingAlignedProcessingTimeWindows.of(Time.of(1000, TimeUnit.MILLISECONDS)))
    .apply(your-function)

// for sliding windows
DataStream<Tuple2<String, Integer>> window1 = source
    .keyBy(0)
    .window(SlidingAlignedProcessingTimeWindows.of(Time.seconds(1), Time.milliseconds(100)))
    .apply(your-function)
```
{% endtab %}

{% tab title="Scala" %}
```scala
// for tumbling windows
val window1 = source
    .keyBy(0)
    .window(TumblingAlignedProcessingTimeWindows.of(Time.of(1000, TimeUnit.MILLISECONDS)))
    .apply(your-function)

// for sliding windows
val window2 = source
    .keyBy(0)
    .window(SlidingAlignedProcessingTimeWindows.of(Time.seconds(1), Time.milliseconds(100)))
    .apply(your-function)
```
{% endtab %}
{% endtabs %}

