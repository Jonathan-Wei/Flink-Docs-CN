# 工作状态

## 键控状态\(Keyed State\)和操作符状态\(Operator State\)

Flink有两种基本的状态：`Keyed State`和`Operator State`。

### 键控状态\(Keyed State\)

键控状态始终Key相关，只能在KeyedStream上的函数和操作符中使用。

可以将Keyed State视为已分区或分片的操作符状态，每个Key只有一个状态分区。每个键控状态在逻辑上绑定到&lt;parallel-operator-instance，key&gt;的唯一组合，并且由于每个键“属于”键控运算符的一个并行实例，我们可以将其简单地视为&lt;运算符，键&gt;

键控状态进一步组织成所谓的键组。键组是Flink重新分配键控状态的原子单元;键组的数量与定义的最大并行度完全相同。在执行过程中，每个键控操作符的并行实例都使用一个或多个键组的键。

### 操作符状态\(Operator State\)

对于操作符状态\(或非键控状态\)，每个操作符状态都绑定到一个并行操作符实例。Kafka连接器是在Flink中使用操作符状态的一个很好的例子。Kafka使用者的每个并行实例都维护一个主题分区和偏移量的映射作为其操作符状态。

当并行度改变时，操作符状态接口支持在并行操作符实例之间重新分布状态。可以有不同的重新分配方案。

## 原始和托管状态

_键控状态_\(Keyed State\)和_操作符状态_\(Operator State\)有两种形式：_托管状态_和_原始状态_。

_托管状态_由Flink运行时控制的数据结构表示，如：内部哈希表或RocksDB。  
例如“ValueState”，“ListState”等.Flink的运行时对状态进行编码并将它们写入检查点。

原始状态是操作符保存在自己的数据结构中的状态。当检查点时，它们只将字节序列写入检查点。Flink对状态的数据结构一无所知，只看到原始字节。

所有数据流功能都可以使用托管状态，但原始状态接口只能在实现操作符时使用。建议使用托管状态（而不是原始状态），因为在托管状态下，Flink能够在并行性更改时自动重新分配状态，并且还可以进行更好的内存管理。

{% hint style="danger" %}
注意：如果托管状态需要自定义序列化逻辑，请参阅[相应的指南](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/custom_serialization.html)以确保将来的兼容性。Flink的默认序列化器不需要特殊处理。
{% endhint %}

## 使用托管键控状态

托管键控状态接口提供对不同类型的状态的访问，这些状态的作用域都是当前输入元素的键。这意味着这种类型的状态只能在`KeyedStream`上使用，`KeyedStream`可以通过`stream.keyBy(…)`创建。

现在，我们将首先查看可用状态的不同类型，然后我们将了解如何在程序中使用它们。可用状态原语为:

* `ValueState<T>` ：这保留了一个可以更新和检索的值（如上所述，作用于输入元素的键的范围，因此操作看到的每个键可能有一个值）。可以使用update\(T\)设置该值，并使用T value\(\)检索该值。
* `ListState<T>`：这将保留元素列表。可以追加元素并在所有当前存储的元素上检索Iterable。使用`add(T)`或`addAll(List<T>)`添加元素，可以使用`Iterable<T> get()`检索Iterable。您还可以使用`update(List<T>)`覆盖现有列表
* `ReducingState<T>`：这保留一个值，表示添加到状态的所有值的聚合。该接口类似于ListState，但使用`add(T)`添加的元素使用指定的`ReduceFunction`缩减为聚合。
* `AggregatingState<IN, OUT>`：这保留一个值，表示添加到状态的所有值的聚合。与`ReducingState`相反，聚合类型可能与添加到状态的元素类型不同。接口与ListState相同，但使用`add(IN)`添加的元素使用指定的`AggregateFunction`进行聚合。
* `FoldingState<T, ACC>`：这保留一个值，表示添加到状态的所有值的聚合。与`ReducingState`相反，聚合类型可能与添加到状态的元素类型不同。该接口类似于ListState，但使用`add(T)`添加的元素使用指定的`FoldFunction`折叠为聚合。
* `MapState<UK, UV>`：这将保留映射列表。您可以将键值对放入状态，并在所有当前存储的映射上检索Iterable。使用`put(UK, UV)`或者`putAll(Map<UK, UV>)`添加映射。可以使用`get(UK)`检索与用户密钥关联的值。可以分别使用`entries()`，`keys()`和`values()`来检索映射，键和值的可迭代视图。

所有类型的状态都有一个clear\(\)方法，用于清除当前活动键\(即输入元素的键\)的状态。

{% hint style="danger" %}
注意：`FoldingState和FoldingStateDescriptor`已在Flink 1.4中弃用，将来将被完全删除。请使用`AggregatingState`而不是`AggregatingStateDescriptor`。
{% endhint %}

重要的是要记住，这些状态对象仅用于与状态接口。状态不一定存储在内部，但可能驻留在磁盘或其他位置。要记住的第二件事是，从状态获得的值取决于input元素的键。因此，如果所涉及的密钥不同，则在一次调用用户函数时获得的值可能与另一次调用中的值不同。

要获取状态句柄，您必须创建StateDescriptor。这保存了状态的名称（正如我们稍后将看到的，您可以创建几个状态，并且它们必须具有唯一的名称以便您可以引用它们），状态所持有的值的类型，并且可能是用户 - 指定的函数，例如`ReduceFunction`。根据要检索的状态类型，可以创建`ValueStateDescriptor`，`ListStateDescriptor`，`ReducingStateDescriptor`，`FoldingStateDescriptor`或`MapStateDescriptor`。

使用`RuntimeContext`访问`State`，因此只能在丰富的函数中使用。请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-master/dev/api_concepts.html#rich-functions)了解相关信息 RichFunction中可用的RuntimeContext具有以下访问状态的方法：

* `ValueState<T> getState(ValueStateDescriptor<T>)`
* `ReducingState<T> getReducingState(ReducingStateDescriptor<T>)`
* `ListState<T> getListState(ListStateDescriptor<T>)`
* `AggregatingState<IN, OUT> getAggregatingState(AggregatingStateDescriptor<IN, ACC, OUT>)`
* `FoldingState<T, ACC> getFoldingState(FoldingStateDescriptor<T, ACC>)`
* `MapState<UK, UV> getMapState(MapStateDescriptor<UK, UV>)`

这是一个FlatMapFunction的例子，它展示了所有部分是如何组合在一起的:

{% tabs %}
{% tab title="Java" %}
```java
public class CountWindowAverage extends RichFlatMapFunction<Tuple2<Long, Long>, Tuple2<Long, Long>> {

    /**
     * The ValueState handle. The first field is the count, the second field a running sum.
     */
    private transient ValueState<Tuple2<Long, Long>> sum;

    @Override
    public void flatMap(Tuple2<Long, Long> input, Collector<Tuple2<Long, Long>> out) throws Exception {

        // access the state value
        Tuple2<Long, Long> currentSum = sum.value();

        // update the count
        currentSum.f0 += 1;

        // add the second field of the input value
        currentSum.f1 += input.f1;

        // update the state
        sum.update(currentSum);

        // if the count reaches 2, emit the average and clear the state
        if (currentSum.f0 >= 2) {
            out.collect(new Tuple2<>(input.f0, currentSum.f1 / currentSum.f0));
            sum.clear();
        }
    }

    @Override
    public void open(Configuration config) {
        ValueStateDescriptor<Tuple2<Long, Long>> descriptor =
                new ValueStateDescriptor<>(
                        "average", // the state name
                        TypeInformation.of(new TypeHint<Tuple2<Long, Long>>() {}), // type information
                        Tuple2.of(0L, 0L)); // default value of the state, if nothing was set
        sum = getRuntimeContext().getState(descriptor);
    }
}

// this can be used in a streaming program like this (assuming we have a StreamExecutionEnvironment env)
env.fromElements(Tuple2.of(1L, 3L), Tuple2.of(1L, 5L), Tuple2.of(1L, 7L), Tuple2.of(1L, 4L), Tuple2.of(1L, 2L))
        .keyBy(0)
        .flatMap(new CountWindowAverage())
        .print();

// the printed output will be (1,4) and (1,5)
```
{% endtab %}

{% tab title="Scala" %}
```scala
class CountWindowAverage extends RichFlatMapFunction[(Long, Long), (Long, Long)] {

  private var sum: ValueState[(Long, Long)] = _

  override def flatMap(input: (Long, Long), out: Collector[(Long, Long)]): Unit = {

    // access the state value
    val tmpCurrentSum = sum.value

    // If it hasn't been used before, it will be null
    val currentSum = if (tmpCurrentSum != null) {
      tmpCurrentSum
    } else {
      (0L, 0L)
    }

    // update the count
    val newSum = (currentSum._1 + 1, currentSum._2 + input._2)

    // update the state
    sum.update(newSum)

    // if the count reaches 2, emit the average and clear the state
    if (newSum._1 >= 2) {
      out.collect((input._1, newSum._2 / newSum._1))
      sum.clear()
    }
  }

  override def open(parameters: Configuration): Unit = {
    sum = getRuntimeContext.getState(
      new ValueStateDescriptor[(Long, Long)]("average", createTypeInformation[(Long, Long)])
    )
  }
}


object ExampleCountWindowAverage extends App {
  val env = StreamExecutionEnvironment.getExecutionEnvironment

  env.fromCollection(List(
    (1L, 3L),
    (1L, 5L),
    (1L, 7L),
    (1L, 4L),
    (1L, 2L)
  )).keyBy(_._1)
    .flatMap(new CountWindowAverage())
    .print()
  // the printed output will be (1,4) and (1,5)

  env.execute("ExampleManagedState")
}
```
{% endtab %}
{% endtabs %}

这个例子实现了一个计数窗口。 我们通过第一个字段键入元组（在示例中都具有相同的键1）。 该函数将计数和运行总和存储在ValueState中。 一旦计数达到2，它将发出平均值并清除状态，以便从0开始。注意，如果我们在第一个字段中具有不同值的元组，则这将为每个不同的输入键保持不同的状态值

### 状态生存时间\(TTL\)

可以将生存时间\(TTL\)分配给任何类型的键控状态。如果TTL已配置，且状态值已过期，则将以最佳方式清理存储值，下面将对此进行更详细的讨论。

所有状态集合类型都支持每个条目的TTL。这意味着列表元素和映射条目将独立过期。

为了使用状态TTL，首先必须构建一个`StateTtlConfig`配置对象。然后，可以通过传递配置在任何状态描述符中启用TTL功能：

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.state.StateTtlConfig;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.time.Time;

StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build();
    
ValueStateDescriptor<String> stateDescriptor = new ValueStateDescriptor<>("text state", String.class);
stateDescriptor.enableTimeToLive(ttlConfig);
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.common.state.StateTtlConfig
import org.apache.flink.api.common.state.ValueStateDescriptor
import org.apache.flink.api.common.time.Time

val ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .setStateVisibility(StateTtlConfig.StateVisibility.NeverReturnExpired)
    .build
    
val stateDescriptor = new ValueStateDescriptor[String]("text state", classOf[String])
stateDescriptor.enableTimeToLive(ttlConfig)
```
{% endtab %}
{% endtabs %}

配置有几个选项需要考虑：

`newBuilder`方法的第一个参数是必需的，它是生存时间值。

更新类型在状态TTL刷新时配置\(默认情况下为OnCreateAndWrite\):

* `StateTtlConfig.UpdateType.OnCreateAndWrite` - 仅限创建和写入权限
* `StateTtlConfig.UpdateType.OnReadAndWrite` - 也读取访问权限

如果尚未清楚，状态可见性配置是否在读取访问时返回过期值（默认为NeverReturnExpired）：

* `StateTtlConfig.StateVisibility.NeverReturnExpired` - 永远不会返回过期的值
* `StateTtlConfig.StateVisibility.ReturnExpiredIfNotCleanedUp` - 如果仍然可用则返回

在NeverReturnExpired的情况下，过期状态表现得好像它不再存在，即使它仍然必须被删除。该选项对于在TTL之后数据必须严格不可读访问的用例非常有用，例如处理隐私敏感数据的应用程序。

另一个选项ReturnExpiredIfNotCleanedUp允许在清理之前返回过期状态。

**笔记：**

* 状态后端存储上次修改的时间戳和用户值，这意味着启用此功能会增加状态存储的消耗。堆状态后端存储一个附加的Java对象，该对象引用用户状态对象和内存中的一个基元长值。rocksdb状态后端为每个存储值、列表条目或映射条目添加8个字节。
* 当前仅支持引用处理时间的**TTL**。
* 尝试使用启用TTL的描述符恢复以前配置为不带TTL的状态，或者使用启用TTL的描述符恢复之前配置的状态，将导致兼容性失败和StateMigrationException。
* TTL配置不是检查点或保存点的一部分，而是Flink在当前运行的作业中如何处理检查点的一种方法。
* 只有用户值序列化程序可以处理空值时，具有TTL的映射状态当前才支持空用户值。如果序列化程序不支持空值，则可以使用NullableSerializer对其进行包装，但要以序列化形式中的额外字节为代价。

#### **清除过期状态**

目前，只有在显式读出过期值时才会删除过期值，例如通过调用`ValueState.value()`。

{% hint style="danger" %}
**注意：**这意味着默认情况下，如果未读取过期状态，则不会将其删除，这可能会导致状态不断增长。这可能在将来的版本中发生变化
{% endhint %}

此外，可以在获取完整状态快照时激活清理，这将减少其大小。在当前实现下，不会清理本地状态，但在从上一个快照恢复时，它不会包含已删除的过期状态。可以在StateTtlConfig中配置:

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.state.StateTtlConfig;
import org.apache.flink.api.common.time.Time;

StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .cleanupFullSnapshot()
    .build();
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.common.state.StateTtlConfig
import org.apache.flink.api.common.time.Time

val ttlConfig = StateTtlConfig
    .newBuilder(Time.seconds(1))
    .cleanupFullSnapshot
    .build
```
{% endtab %}
{% endtabs %}

此选项不适用于RocksDB状态后端中的增量检查点。

未来将添加更多策略，以便在后台自动清理过期状态。

### 在Scala DataStream API中声明

除了上面描述的接口之外，Scala API还具有有状态map\(\)或flatMap\(\)函数的快捷方式，在KeyedStream上具有单个ValueState。 用户函数在Option中获取ValueState的当前值，并且必须返回将用于更新状态的更新值。

```scala
val stream: DataStream[(String, Int)] = ...

val counts: DataStream[(String, Int)] = stream
  .keyBy(_._1)
  .mapWithState((in: (String, Int), count: Option[Int]) =>
    count match {
      case Some(c) => ( (in._1, c), Some(c + in._2) )
      case None => ( (in._1, 0), Some(in._2) )
    })
```

## 使用托管操作符状态

要使用托管操作符状态，有状态函数可以实现更通用的`CheckpointedFunction` 接口或`ListCheckpointed<T extends Serializable>`接口。

### **CheckpointedFunction**

`CheckpointedFunction`接口提供对具有不同重新分发方案的非键控状态的访问。它需要实现两种方法：

```java
void snapshotState(FunctionSnapshotContext context) throws Exception;

void initializeState(FunctionInitializationContext context) throws Exception;
```

每当必须执行检查点时，都会调用snapshotState\(\)。 每次初始化用户定义的函数时，都会调用对应的initializeState\(\)，即首次初始化函数时，或者当函数实际从早期检查点恢复时。 鉴于此，initializeState\(\)不仅是初始化不同类型状态的地方，而且还包括状态恢复逻辑。

目前，支持列表样式的托管操作符状态。 该状态应该是一个可序列化对象的List，彼此独立，因此有资格在重新缩放时重新分配。 换句话说，这些对象是可以重新分配非键控状态的最精细的粒度。 根据状态访问方法，定义了以下重新分发方案：

* **Even-split 重分配：**每个操作符返回一个状态元素列表。整个状态在逻辑上是所有列表的串联。在恢复/重新分配时，列表被平均分成与并行操作符一样多的子列表。每个操作符都会获得一个子列表，该子列表可以为空，也可以包含一个或多个元素。例如，如果并行度为1，则操作符的检查点状态包含元素element1和element2，当将并行度增加到2时，element1可能会出现在操作符实例0中，而element2则会出现在操作符实例1中。
* **Union 重分配：**每个操作符返回一个状态元素列表。整个状态在逻辑上是所有列表的串联。在恢复/重新分配时，每个操作符都会获得完整的状态元素列表。

下面是一个有状态SinkFunction的例子，它使用CheckpointedFunction在将元素发送到外部之前缓冲它们。它演示了基本的偶分裂再分配列表状态：

{% tabs %}
{% tab title="Java" %}
```java
public class BufferingSink
        implements SinkFunction<Tuple2<String, Integer>>,
                   CheckpointedFunction {

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
        ListStateDescriptor<Tuple2<String, Integer>> descriptor =
            new ListStateDescriptor<>(
                "buffered-elements",
                TypeInformation.of(new TypeHint<Tuple2<String, Integer>>() {}));

        checkpointedState = context.getOperatorStateStore().getListState(descriptor);

        if (context.isRestored()) {
            for (Tuple2<String, Integer> element : checkpointedState.get()) {
                bufferedElements.add(element);
            }
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class BufferingSink(threshold: Int = 0)
  extends SinkFunction[(String, Int)]
    with CheckpointedFunction {

  @transient
  private var checkpointedState: ListState[(String, Int)] = _

  private val bufferedElements = ListBuffer[(String, Int)]()

  override def invoke(value: (String, Int)): Unit = {
    bufferedElements += value
    if (bufferedElements.size == threshold) {
      for (element <- bufferedElements) {
        // send it to the sink
      }
      bufferedElements.clear()
    }
  }

  override def snapshotState(context: FunctionSnapshotContext): Unit = {
    checkpointedState.clear()
    for (element <- bufferedElements) {
      checkpointedState.add(element)
    }
  }

  override def initializeState(context: FunctionInitializationContext): Unit = {
    val descriptor = new ListStateDescriptor[(String, Int)](
      "buffered-elements",
      TypeInformation.of(new TypeHint[(String, Int)]() {})
    )

    checkpointedState = context.getOperatorStateStore.getListState(descriptor)

    if(context.isRestored) {
      for(element <- checkpointedState.get()) {
        bufferedElements += element
      }
    }
  }

}
```
{% endtab %}
{% endtabs %}

initializeState方法以FunctionInitializationContext作为参数。这用于初始化非键控状态“容器”。这些是ListState类型的容器，其中非键控状态对象将在检查点上存储。

注意状态是如何初始化的，类似于键控状态，使用一个状态描述符，其中包含状态名和关于状态持有的值的类型的信息:

{% tabs %}
{% tab title="Java" %}
```java
ListStateDescriptor<Tuple2<String, Integer>> descriptor =
    new ListStateDescriptor<>(
        "buffered-elements",
        TypeInformation.of(new TypeHint<Tuple2<Long, Long>>() {}));

checkpointedState = context.getOperatorStateStore().getListState(descriptor);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val descriptor = new ListStateDescriptor[(String, Long)](
    "buffered-elements",
    TypeInformation.of(new TypeHint[(String, Long)]() {})
)

checkpointedState = context.getOperatorStateStore.getListState(descriptor)
```
{% endtab %}
{% endtabs %}

状态访问方法的命名约定包含其重新分布模式及其状态结构。例如，要在还原时使用联合重分发模式的列表状态，可以使用`getUnionListState(descriptor)`访问该状态。如果方法名不包含重分发模式，例如getListState\(descriptor\)，它仅仅意味着将使用基本的均分重分发模式。

在初始化容器之后，我们使用上下文的`isRestored()`方法检查失败后是否正在恢复。如果是这样`true`，_即_我们正在恢复，则应用恢复逻辑。

如修改后的`BufferingSink`的代码所示，状态初始化期间恢复的`ListState`保存在类变量中，以备将来在`Snapshotstate()`中使用。在这里，`ListState`（由先前的检查点包含的所有对象）被清除，然后被我们想要检查点的新对象填充。

另外，键控状态也可以在`initializeState()`方法中初始化。这可以使用提供的`FunctionInitializationContext`来完成。

**ListCheckpointed**

ListCheckpointed接口是CheckpointedFunction的一个更有限的变体，它仅支持在恢复时使用even-split重新分配方案的列表样式状态。 它还需要实现两种方法：

```java
List<T> snapshotState(long checkpointId, long timestamp) throws Exception;

void restoreState(List<T> state) throws Exception;
```

在`snapshotState()`上，操作符应该向检查点返回一个对象列表，而`restoreState`必须在恢复时处理这个列表。如果状态不可重分区，则始终可以在`snapshotState()`中返回`Collections.singletonList(MY_STATE)`。

### 有状态的源函数

与其他操作符相比，有状态源需要更多的关注。为了更新状态和输出集合的原子性\(对于失败/恢复的精确一次语义来说是必需的\)，用户需要从源上下文获取一个锁。

{% tabs %}
{% tab title="Java" %}
```java
public static class CounterSource
        extends RichParallelSourceFunction<Long>
        implements ListCheckpointed<Long> {

    /**  current offset for exactly once semantics */
    private Long offset = 0L;

    /** flag for job cancellation */
    private volatile boolean isRunning = true;

    @Override
    public void run(SourceContext<Long> ctx) {
        final Object lock = ctx.getCheckpointLock();

        while (isRunning) {
            // output and state update are atomic
            synchronized (lock) {
                ctx.collect(offset);
                offset += 1;
            }
        }
    }

    @Override
    public void cancel() {
        isRunning = false;
    }

    @Override
    public List<Long> snapshotState(long checkpointId, long checkpointTimestamp) {
        return Collections.singletonList(offset);
    }

    @Override
    public void restoreState(List<Long> state) {
        for (Long s : state)
            offset = s;
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class CounterSource
       extends RichParallelSourceFunction[Long]
       with ListCheckpointed[Long] {

  @volatile
  private var isRunning = true

  private var offset = 0L

  override def run(ctx: SourceFunction.SourceContext[Long]): Unit = {
    val lock = ctx.getCheckpointLock

    while (isRunning) {
      // output and state update are atomic
      lock.synchronized({
        ctx.collect(offset)

        offset += 1
      })
    }
  }

  override def cancel(): Unit = isRunning = false

  override def restoreState(state: util.List[Long]): Unit =
    for (s <- state) {
      offset = s
    }

  override def snapshotState(checkpointId: Long, timestamp: Long): util.List[Long] =
    Collections.singletonList(offset)

}
```
{% endtab %}
{% endtabs %}

当Flink完全确认检查点与外界通信时，某些操作符可能需要这些信息。 在这种情况下，请参阅`org.apache.flink.runtime.state.CheckpointListener`接口。

