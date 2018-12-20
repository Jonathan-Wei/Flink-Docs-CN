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

CountMapper是RichFlatMapFunction假定grouped-by-key输入流的形式\(word,1\)。该函数为每个传入密钥（ValueState counter）保留一个计数器，如果某个单词的出现次数超过用户提供的阈值，则发出包含单词本身和发生次数的元组。

BufferingSink是一个SinkFunction，它接收元素（可能是CountMapper的输出）并缓存它们，直到达到某个用户指定的阈值，然后将它们发送到最终接收器。这是避免对数据库或外部存储系统进行许多昂贵调用的常用方法。为了以容错方式执行缓冲，缓冲元素被保存在一个列表（bufferedElements）中，该列表周期性地被检查点。

### 状态API迁移

为了利用Flink.1.2的新特性，应该修改上面的代码以使用新的状态抽象。在完成这些更改之后，您将能够更改作业的并行性（按比例放大或缩小），并且保证您的作业的新版本将从其前任停止的地方开始。

**键控状态\(Keyed State\)**:在深入研究迁移过程的细节之前需要注意的一点是，如果您的函数**只有键控状态**，那么Flink 1.1中完全相同的代码也适用于Flink 1.2，它完全支持新特性和完全向后兼容。可以仅为了更好的代码组织而进行更改，但这只是风格问题。

如上所述，本节的其余部分将重点介绍**非键控状态\(no-keyed State\)**。

#### **重新缩放和新状态抽象**

第一个修改是从旧`Checkpointed<T extends Serializable>`状态接口到新状态接口的转换。在Flink 1.2中，有状态函数可以实现更通用的`CheckpointedFunction` 接口或`ListCheckpointed<T extends Serializable>`接口，它在语义上更接近旧接口 `Checkpointed`。

在这两种情况下，非键控状态都是可序列化对象的列表，彼此独立，因此在重新缩放时有资格进行重新分配。换句话说，这些对象是可以重新分区非键控状态的最佳粒度。例如，如果并行度为1，则缓冲池的检查点状态包含元素\(test1, 2\)和\(test2, 2\)，当将并行度增加到2时，\(test1, 2\)可能最终出现在任务0中，而\(test2, 2\)将进入任务1。

有关键控状态和非键控状态重新缩放的原则的更多详细信息，请参阅[状态文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/index.html)。

**ListCheckpointed** 

listcheckpoint接口需要实现两种方法:

```text
List<T> snapshotState(long checkpointId, long timestamp) throws Exception;

void restoreState(List<T> state) throws Exception;
```

它们的语义与旧Checkpointed接口中的对应部分相同。唯一的区别在于，现在snapshotState\(\)应该如前所述返回一个对象列表到检查点，并且restoreState必须在恢复时处理这个列表。如果状态不可重新分区，则始终可以在snapshotState\(\)中返回Collections.singletonList\(MY\_STATE\)。BufferingSink的更新代码包括：

```text
public class BufferingSinkListCheckpointed implements
        SinkFunction<Tuple2<String, Integer>>,
        ListCheckpointed<Tuple2<String, Integer>>,
        CheckpointedRestoring<ArrayList<Tuple2<String, Integer>>> {

    private final int threshold;

    private transient ListState<Tuple2<String, Integer>> checkpointedState;

    private List<Tuple2<String, Integer>> bufferedElements;

    public BufferingSinkListCheckpointed(int threshold) {
        this.threshold = threshold;
        this.bufferedElements = new ArrayList<>();
    }

    @Override
    public void invoke(Tuple2<String, Integer> value) throws Exception {
        this.bufferedElements.add(value);
        if (bufferedElements.size() == threshold) {
            for (Tuple2<String, Integer> element: bufferedElements) {
                // send it to the sink
            }
            bufferedElements.clear();
        }
    }

    @Override
    public List<Tuple2<String, Integer>> snapshotState(
            long checkpointId, long timestamp) throws Exception {
        return this.bufferedElements;
    }

    @Override
    public void restoreState(List<Tuple2<String, Integer>> state) throws Exception {
        if (!state.isEmpty()) {
            this.bufferedElements.addAll(state);
        }
    }

    @Override
    public void restoreState(ArrayList<Tuple2<String, Integer>> state) throws Exception {
        // this is from the CheckpointedRestoring interface.
        this.bufferedElements.addAll(state);
    }
}

```

如代码所示，更新后的函数还实现了CheckpointedRestoring接口。这是为了向后兼容的原因，更多的细节将在本节的最后解释。

**CheckpointedFunction** 

CheckpointedFunction接口再次需要实现两种方法:

```text
void snapshotState(FunctionSnapshotContext context) throws Exception;

void initializeState(FunctionInitializationContext context) throws Exception;
```

与Flink 1.1中一样，每当执行检查点时就调用snapshotState\(\)，但是现在每次初始化用户定义的函数时都调用initializeState\(\)\(它是restoreState\(\)的对应部分\)，而不仅仅是在从故障中恢复的情况下。鉴于此，initializeState\(\)不仅是初始化不同类型的状态的地方，而且还包括状态恢复逻辑。下面介绍用于BufferingSink的CheckpointedFunction接口的实现。

```text
public class BufferingSink implements SinkFunction<Tuple2<String, Integer>>,
        CheckpointedFunction, CheckpointedRestoring<ArrayList<Tuple2<String, Integer>>> {

    private final int threshold;

    private transient ListState<Tuple2<String, Integer>> checkpointedState;

    private List<Tuple2<String, Integer>> bufferedElements;

    public BufferingSink(int threshold) {
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
    public void snapshotState(FunctionSnapshotContext context) throws Exception {
        checkpointedState.clear();
        for (Tuple2<String, Integer> element : bufferedElements) {
            checkpointedState.add(element);
        }
    }

    @Override
    public void initializeState(FunctionInitializationContext context) throws Exception {
        checkpointedState = context.getOperatorStateStore().
            getSerializableListState("buffered-elements");

        if (context.isRestored()) {
            for (Tuple2<String, Integer> element : checkpointedState.get()) {
                bufferedElements.add(element);
            }
        }
    }

    @Override
    public void restoreState(ArrayList<Tuple2<String, Integer>> state) throws Exception {
        // this is from the CheckpointedRestoring interface.
        this.bufferedElements.addAll(state);
    }
}
```

initializeState以FunctionInitializationContext作为参数。这用于初始化非键控状态“容器”。这是一个ListState类型的容器，其中非键控状态对象将存储在检查点上:

`this.checkpointedState = context.getOperatorStateStore().getSerializableListState("buffered-elements");`

在初始化容器之后，我们使用上下文的isrestore\(\)方法检查失败后是否正在恢复。如果这是真的，即我们正在恢复，则应用恢复逻辑。

如修改后的BufferingSink代码所示，在状态初始化期间恢复的这个ListState保存在一个类变量中，以备将来在snapshotState\(\)中使用。在那里，ListState将清除前一个检查点包含的所有对象，然后填充我们希望通过检查点的新对象。

作为附注，键控状态也可以在initializeState\(\)方法中初始化。这可以使用作为参数给出的FunctionInitializationContext来完成，而不是使用Flink 1.1的RuntimeContext。如果在CountMapper示例中使用CheckpointedFunction接口，则可以删除旧的open\(\)方法，并且新的snapshotState\(\)和initializeState\(\)方法如下所示：

```text
public class CountMapper extends RichFlatMapFunction<Tuple2<String, Integer>, Tuple2<String, Integer>>
        implements CheckpointedFunction {

    private transient ValueState<Integer> counter;

    private final int numberElements;

    public CountMapper(int numberElements) {
        this.numberElements = numberElements;
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

    @Override
    public void snapshotState(FunctionSnapshotContext context) throws Exception {
        // all managed, nothing to do.
    }

    @Override
    public void initializeState(FunctionInitializationContext context) throws Exception {
        counter = context.getKeyedStateStore().getState(
            new ValueStateDescriptor<>("counter", Integer.class, 0));
    }
}
```

请注意，snapshotState\(\)方法是空的，因为Flink本身负责检查点上的快照托管键控状态。

#### 向后兼容Flink1.1

到目前为止，我们已经了解了如何修改函数来利用Flink 1.2引入的新特性。剩下的问题是“我能否确保我修改后的\(Flink 1.2\)作业将从我从Flink 1.1停止运行的作业开始?”

答案是肯定的，方法很简单。对于键控状态，您不需要做任何事情。Flink将负责从Flink 1.1恢复状态。对于非键控状态，新函数必须实现CheckpointedRestoring接口，如上面的代码所示。这有一个方法，即来自Flink 1.1的旧Checkpointed接口的常见restoreState\(\)。如BufferingSink的修改代码所示，restoreState\(\)方法与其前身相同。

### 对齐处理时间窗口操作符

在Flink 1.1中，只有在没有指定驱逐器或触发器的情况下操作处理时间时，键控流上的命令timeWindow\(\)才会实例化一种特殊类型的窗口操作符。这可以是AggregatingProcessingTimeWindowOperator，也可以是AccumulatingProcessingTimeWindowOperator。这两个操作符都称为对齐窗口操作符，因为它们假定输入元素按顺序到达。这在处理时间操作时是有效的，因为元素在到达窗口操作符时将得到时钟时间戳。这些操作符被限制为使用内存状态后端，并且已经优化了用于存储利用顺序输入元素到达的每个窗口元素的数据结构。

在Flink 1.2中，对齐的窗口操作符是不受欢迎的，所有窗口操作都通过通用窗口操作符进行。这种迁移不需要更改Flink 1.1作业的代码，因为Flink将透明地读取Flink 1.1保存点中对齐的窗口操作符存储的状态，将其转换为与通用窗口操作符兼容的格式，并使用通用窗口操作符恢复执行。

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

