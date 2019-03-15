# 广播状态模式

[工作状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html)描述操作符状态，该状态在还原时均匀分布在操作符的并行任务之间，或与整个状态联合，用于初始化还原的并行任务。

第三种受支持的操作符状态是广播状态。引入`Broadcast State`是为了支持这样的用例:来自一个流的一些数据需要广播到所有下游任务，在这些任务中，这些数据被本地存储并用于处理另一个流上的所有传入元素。例如，广播状态可以作为一种自然匹配出现，您可以想象一个低吞吐量流，其中包含一组规则，我们希望对来自另一个流的所有元素进行评估。考虑到上述类型的用例，`Broadcast State`与其他操作符状态的不同之处在于:

1. 它有一个Map格式，
2. 它仅适用于具有_广播_流和_非广播_流的输入的特定操作符
3. 这样的操作符可以具有不同名称的_多个广播状态_。

## 提供的API

为了展示所提供的api，在展示其全部功能之前，我们将从一个示例开始。作为我们的运行示例，我们将使用这样的情况:我们有一个不同颜色和形状的对象流，我们希望找到遵循特定模式的相同颜色的对象对，例如一个矩形后面跟着一个三角形。我们假设这组有趣的模式会随着时间而发展。

在本例中，第一个流将包含具有颜色和形状属性的Item类型的元素。另一个流将包含规则。

从项目流开始，我们只需要按颜色键入它，因为我们想要相同颜色的对。这将确保相同颜色的元素最终出现在相同的物理机器上。

```java
// key the shapes by color
KeyedStream<Item, Color> colorPartitionedStream = shapeStream
                        .keyBy(new KeySelector<Shape, Color>(){...});
```

接下来看`Rules`，包含它们的流应该广播到所有下游任务，这些任务应该在本地存储它们，以便根据所有传入的`Item`对它们进行评估。下面的代码片段将使用广播规则流和提供的`MapStateDescriptor`创建规则存储的广播状态。

```java
// a map descriptor to store the name of the rule (string) and the rule itself.
MapStateDescriptor<String, Rule> ruleStateDescriptor = new MapStateDescriptor<>(
			"RulesBroadcastState",
			BasicTypeInfo.STRING_TYPE_INFO,
			TypeInformation.of(new TypeHint<Rule>() {}));
		
// broadcast the rules and create the broadcast state
BroadcastStream<Rule> ruleBroadcastStream = ruleStream
                        .broadcast(ruleStateDescriptor);
```

最后，为了对来自Item流的输入元素评估规则，我们需要:

1. 连接两个流
2. 指定我们的匹配检测逻辑。

将流\(键控的或非键控的\)与广播流连接可以通过在非广播流上调用`connect()`来完成，并将广播流作为参数。这将返回一个`BroadcastConnectedStream`，在它上面我们可以使用一种特殊类型的协处理器函数调用`process()`。该函数将包含我们的匹配逻辑。函数的确切类型取决于非广播流的类型:

* 如果**键控\(Keyed\)**，则函数是 `KeyedBroadcastProcessFunction`。
* 如果**是非键控\(non-keyed\)的**，则函数是 `BroadcastProcessFunction`。

鉴于我们的非广播流是键控的，以下代码段包含上述调用：

{% hint style="info" %}
应该在非广播流上调用connect，并将BroadcastStream作为参数。
{% endhint %}

```java
DataStream<String> output = colorPartitionedStream
                 .connect(ruleBroadcastStream)
                 .process(
                     
                     // type arguments in our KeyedBroadcastProcessFunction represent: 
                     //   1. the key of the keyed stream
                     //   2. the type of elements in the non-broadcast side
                     //   3. the type of elements in the broadcast side
                     //   4. the type of the result, here a string
                     
                     new KeyedBroadcastProcessFunction<Color, Item, Rule, String>() {
                         // my matching logic
                     }
                 );
```

### BroadcastProcessFunction和KeyedBroadcastProcessFunction

与`CoProcessFunction`的情况一样，这些函数有两种实现方法 processBroadcastElement\(\)负责处理广播流中的传入元素，processElement\(\)用于非广播流。 这些方法的完整签名如下：

```java
public abstract class BroadcastProcessFunction<IN1, IN2, OUT> extends BaseBroadcastProcessFunction {

    public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;

    public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;
}
```

```java
public abstract class KeyedBroadcastProcessFunction<KS, IN1, IN2, OUT> {

    public abstract void processElement(IN1 value, ReadOnlyContext ctx, Collector<OUT> out) throws Exception;

    public abstract void processBroadcastElement(IN2 value, Context ctx, Collector<OUT> out) throws Exception;

    public void onTimer(long timestamp, OnTimerContext ctx, Collector<OUT> out) throws Exception;
}
```

首先要注意的是，两个函数都需要实现`processBroadcastElement()`方法来处理广播端的元素，而`processElement()`则用于非广播端的元素。

这两种方法在提供的上下文中有所不同。 非广播侧是`ReadOnlyContext`，而广播侧是`Context`。

这两个上下文（以下枚举中的ctx）：

1. 允许访问广播状态： `ctx.getBroadcastState(MapStateDescriptor<K, V> stateDescriptor)`
2. 允许查询元素的时间戳：`ctx.timestamp()`，
3. 获取当前水位线： `ctx.currentWatermark()`
4. 得到当前的处理时间：`ctx.currentProcessingTime()`和
5. 向侧面输出发射元素：`ctx.output(OutputTag<X> outputTag, X value)`。

`getBroadcastState()`中的`stateDescriptor`应该与上面`.broadcast(ruleStateDescriptor)`中的`stateDescriptor`相同。

不同之处在于它们对广播状态的访问类型。广播端具有对其的读写访问权，而非广播端具有只读访问权\(即名称\)。原因是在Flink中没有跨任务通信。因此，为了保证广播状态中的内容在我们的操作符的所有并行实例中都是相同的，我们只向广播端提供读写访问，广播端在所有任务中都看到相同的元素，我们要求该端的每个传入元素的计算在所有任务中都是相同的。忽略此规则将破坏状态的一致性保证，导致不一致且通常难以调试结果。

{% hint style="info" %}
**注意：** \`processBroadcast（）\`中实现的逻辑必须在所有并行实例中具有相同的确定性行为！
{% endhint %}

最后，由于KeyedBroadcastProcessFunction是在一个键控流上操作的，它公开了一些BroadcastProcessFunction不可用的功能。那就是:

* `processElement()`方法中的`ReadOnlyContext`允许访问Flink的底层计时器服务，该服务允许注册事件和/或处理时间计时器。当计时器触发时，`onTimer()`\(如上所示\)通过`OnTimerContext`调用，`OnTimerContext`公开了与`ReadOnlyContext plus`相同的功能
* `processBroadcastElement()`方法中的`Context`包含`applyToKeyedState(StateDescriptor<S, VS> stateDescriptor, KeyedStateFunction<KS, S> function)`方法。这允许注册一个`KeyedStateFunction`，将其应用于与提供的`stateDescriptor`关联的所有键的所有状态。

{% hint style="info" %}
**注意：**只能在\`KeyedBroadcastProcessFunction\`的\`processElement（）\`中注册定时器。在\`processBroadcastElement（）\`方法中是不可能的，因为没有与广播元素相关联的键。
{% endhint %}

回到我们原来的例子，我们`KeyedBroadcastProcessFunction`可能看起来如下：

```java
new KeyedBroadcastProcessFunction<Color, Item, Rule, String>() {

    // store partial matches, i.e. first elements of the pair waiting for their second element
    // we keep a list as we may have many first elements waiting
    private final MapStateDescriptor<String, List<Item>> mapStateDesc =
        new MapStateDescriptor<>(
            "items",
            BasicTypeInfo.STRING_TYPE_INFO,
            new ListTypeInfo<>(Item.class));

    // identical to our ruleStateDescriptor above
    private final MapStateDescriptor<String, Rule> ruleStateDescriptor = 
        new MapStateDescriptor<>(
            "RulesBroadcastState",
            BasicTypeInfo.STRING_TYPE_INFO,
            TypeInformation.of(new TypeHint<Rule>() {}));

    @Override
    public void processBroadcastElement(Rule value,
                                        Context ctx,
                                        Collector<String> out) throws Exception {
        ctx.getBroadcastState(ruleStateDescriptor).put(value.name, value);
    }

    @Override
    public void processElement(Item value,
                               ReadOnlyContext ctx,
                               Collector<String> out) throws Exception {

        final MapState<String, List<Item>> state = getRuntimeContext().getMapState(mapStateDesc);
        final Shape shape = value.getShape();
    
        for (Map.Entry<String, Rule> entry :
                ctx.getBroadcastState(ruleStateDescriptor).immutableEntries()) {
            final String ruleName = entry.getKey();
            final Rule rule = entry.getValue();
    
            List<Item> stored = state.get(ruleName);
            if (stored == null) {
                stored = new ArrayList<>();
            }
    
            if (shape == rule.second && !stored.isEmpty()) {
                for (Item i : stored) {
                    out.collect("MATCH: " + i + " - " + value);
                }
                stored.clear();
            }
    
            // there is no else{} to cover if rule.first == rule.second
            if (shape.equals(rule.first)) {
                stored.add(value);
            }
    
            if (stored.isEmpty()) {
                state.remove(ruleName);
            } else {
                state.put(ruleName, stored);
            }
        }
    }
}
```

## 重要考虑因素

在描述提供的API之后，本节重点介绍使用广播状态时要记住的重要事项。具体如下：

* **没有跨任务通信**：如前所述，这就是为什么只有（Keyed）-BroadcastProcessFunction的广播端可以修改广播状态的内容。此外，用户必须确保所有任务以相同的方式为每个传入元素修改广播状态的内容。否则，不同的任务可能具有不同的内容，从而导致不一致的结果。
* **广播状态中的事件顺序可能因任务而异**：尽管广播流的元素保证所有元素将（最终）转到所有下游任务，但元素可能以不同的顺序到达每个任务。因此，每个传入元素的状态更新不得取决于传入事件的顺序。
* **所有任务都检查它们的广播状态**：虽然当检查点发生时，所有任务在广播状态中具有相同的元素（检查点障碍不会覆盖元素），但所有任务都检查它们的广播状态，而不仅仅是其中一个。这是一个设计决策，以避免在恢复期间从同一文件中读取所有任务（从而避免热点），尽管它的代价是将检查点状态的大小增加p（=并行度）。 Flink保证在恢复/重新缩放时不会有**重复数据**，也**不会丢失数据**。在具有相同或更小并行度的恢复的情况下，每个任务读取其检查点状态。在按比例放大时，每个任务都读取其自己的状态，其余任务（p\_new-p\_old）以循环方式读取先前任务的检查点。
* **没有RocksDB状态后端**：广播状态在运行时保留在内存中，并且应该相应地进行内存配置。这适用于所有操作符状态。

