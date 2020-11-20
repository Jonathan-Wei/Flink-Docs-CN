# 事件驱动应用

## 处理函数

### 介绍

 `ProcessFunction`将事件处理与计时器和状态相结合，使其成为流处理应用程序的强大构建块。这是使用Flink创建事件驱动的应用程序的基础。它与十分相似`RichFlatMapFunction`，但增加了计时器。

### 示例

如果你在“流式分析”培训中进行了实践练习，你会记得它使用TumblingEventTimeWindow计算每个驱动程序每小时的提示总和，如下所示：

```java
// compute the sum of the tips per hour for each driver
DataStream<Tuple3<Long, Long, Float>> hourlyTips = fares
        .keyBy((TaxiFare fare) -> fare.driverId)
        .window(TumblingEventTimeWindows.of(Time.hours(1)))
        .process(new AddTips());
```

用KeyedProcessFunction做同样的事情是相当简单和有教育意义的。让我们首先用以下代码替换上面的代码：

```java
// compute the sum of the tips per hour for each driver
DataStream<Tuple3<Long, Long, Float>> hourlyTips = fares
        .keyBy((TaxiFare fare) -> fare.driverId)
        .process(new PseudoWindow(Time.hours(1)));
```

 在此代码段中，一个名为`PseudoWindow`的`KeyedProcessFunction`被应用于键控流，其结果为`DataStream<Tuple3<Long, Long, Float>>`（与使用Flink内置时间窗口的实现所产生的相同类型的流）。

`PseudoWindow`的整体轮廓具有如下：

```java
// Compute the sum of the tips for each driver in hour-long windows.
// The keys are driverIds.
public static class PseudoWindow extends 
        KeyedProcessFunction<Long, TaxiFare, Tuple3<Long, Long, Float>> {

    private final long durationMsec;

    public PseudoWindow(Time duration) {
        this.durationMsec = duration.toMilliseconds();
    }

    @Override
    // Called once during initialization.
    public void open(Configuration conf) {
        . . .
    }

    @Override
    // Called as each fare arrives to be processed.
    public void processElement(
            TaxiFare fare,
            Context ctx,
            Collector<Tuple3<Long, Long, Float>> out) throws Exception {

        . . .
    }

    @Override
    // Called when the current watermark indicates that a window is now complete.
    public void onTimer(long timestamp, 
            OnTimerContext context, 
            Collector<Tuple3<Long, Long, Float>> out) throws Exception {

        . . .
    }
}
```

注意事项：

* 这里有几种类型的ProcessFunctions-这里是一个`KeyedProcessFunction`，但也有 `CoProcessFunctions`，`BroadcastProcessFunctions`等等。
* `KeyedProcessFunction`是一种`RichFunction`。作为`RichFunction`，它可以访问使用托管键控状态所需的`open` 和`getRuntimeContext`方法。
* 这里有两个回调要实现：`processElement`和`onTimer`。`processElement`在每个传入事件中被调用；`onTimer`在计时器触发时调用。这些可以是事件时间计时器，也可以是处理时间计时器。两者`processElement`和`onTimer`都提供了一个上下文对象，可用于与`TimerService`（以及其他事物）进行交互。两个回调也都传递了一个`Collector`，可用于发送结果。

#### **open\(\)**

```java
// Keyed, managed state, with an entry for each window, keyed by the window's end time.
// There is a separate MapState object for each driver.
private transient MapState<Long, Float> sumOfTips;

@Override
public void open(Configuration conf) {

    MapStateDescriptor<Long, Float> sumDesc =
            new MapStateDescriptor<>("sumOfTips", Long.class, Float.class);
    sumOfTips = getRuntimeContext().getMapState(sumDesc);
}
```

由于票价事件可能会无序到达，有时需要在计算完前一小时的结果之前处理事件一个小时。事实上，如果**水位线**延迟比窗口长度长得多，那么可能会有多个窗口同时打开，而不仅仅是两个窗口。这个实现通过使用MapState来支持这一点，MapState将每个窗口结束的时间戳映射到该窗口的提示总和。

#### **processElement\(\)**

```text
public void processElement(
        TaxiFare fare,
        Context ctx,
        Collector<Tuple3<Long, Long, Float>> out) throws Exception {

    long eventTime = fare.getEventTime();
    TimerService timerService = ctx.timerService();

    if (eventTime <= timerService.currentWatermark()) {
        // This event is late; its window has already been triggered.
    } else {
        // Round up eventTime to the end of the window containing this event.
        long endOfWindow = (eventTime - (eventTime % durationMsec) + durationMsec - 1);

        // Schedule a callback for when the window has been completed.
        timerService.registerEventTimeTimer(endOfWindow);

        // Add this fare's tip to the running total for that window.
        Float sum = sumOfTips.get(endOfWindow);
        if (sum == null) {
            sum = 0.0F;
        }
        sum += fare.tip;
        sumOfTips.put(endOfWindow, sum);
    }
}
```

注意事项：

* 迟到的事件会怎样？**水位线**后面（即较晚）的事件将被丢弃。如果你想做得更好，请考虑使用侧面输出，[下一节](https://ci.apache.org/projects/flink/flink-docs-release-1.11/learn-flink/event_driven.html#side-outputs)将对此进行说明。
* 本例使用`MapState`，其中key是时间戳，并为同一时间戳设置Timer。这是一个常见的模式；它使得在计时器触发时查找相关信息变得简单而高效。

#### **onTimer\(\)**

```text
public void onTimer(
        long timestamp, 
        OnTimerContext context, 
        Collector<Tuple3<Long, Long, Float>> out) throws Exception {

    long driverId = context.getCurrentKey();
    // Look up the result for the hour that just ended.
    Float sumOfTips = this.sumOfTips.get(timestamp);

    Tuple3<Long, Long, Float> result = Tuple3.of(driverId, timestamp, sumOfTips);
    out.collect(result);
    this.sumOfTips.remove(timestamp);
}
```

观察结果：

* 传递给`onTimer`的`OnTimerContext`上下文可用于确定当前键。
* 当当前水位线到达每个小时的结束时，就会触发我们的伪窗口`onTimer`。这个onTimer方法从`sumOfTips`中删除相关条目，其效果是使它无法适应延迟事件。这等效于在使用Flink的时间窗口时将allowedLateness设置为零。

### 性能考量

Flink提供了针对RocksDB优化的MapState和ListState类型。在可能的情况下，应该使用这些对象，而不是使用持有某种集合的ValueState对象。RocksDB状态后端可以不经过\(反\)序列化而附加到ListState，对于MapState来说，每个键/值对都是一个单独的RocksDB对象，因此MapState可以被有效地访问和更新。

## 侧面输出

### 介绍

想要从Flink操作符获得多个输出流有几个很好的理由，例如报告：

* 异常
* 格式错误的事件
* 延迟事件
* 操作警报，例如与外部服务的超时连接

侧面输出是实现此目的的便捷方法。除了错误报告之外，侧面输出也是实现流的n路拆分的好方法。

### 示例

现在可以对前一节中忽略的延迟事件进行处理。

侧面输出通道与OutputTag相关联。这些标记具有与端输出的DataStream类型对应的泛型类型，并且它们具有名称。

```java
private static final OutputTag<TaxiFare> lateFares = new OutputTag<TaxiFare>("lateFares") {};
```

上述展示的是一个静态的OutputTag，它可以在PseudoWindow的processElement方法中被引用。

```java
if (eventTime <= timerService.currentWatermark()) {
    // This event is late; its window has already been triggered.
    ctx.output(lateFares, fare);
} else {
    . . .
}
```

在作业的main方法中从该侧输出访问流时：

```java
// compute the sum of the tips per hour for each driver
SingleOutputStreamOperator hourlyTips = fares
        .keyBy((TaxiFare fare) -> fare.driverId)
        .process(new PseudoWindow(Time.hours(1)));

hourlyTips.getSideOutput(lateFares).print();
```

或者，可以使用两个具有相同名称的OutputTag来引用相同的侧面输出，但是如果这样做，它们必须具有相同的类型。

## 结束语

在此示例中，您已经了解了如何使用`ProcessFunction`重新实现一个简单的时间窗口。当然，如果Flink的内置窗口API可以满足您的需求，请继续使用它。但是，如果您发现自己正在考虑做一些与Flink的窗户扭曲的事情，请不要害怕自己动手。

此外，`ProcessFunctions`对于计算分析之外的许多其他用例也很有用。下面的动手练习提供了一个完全不同的示例。

ProcessFunctions的另一个常见用例是使过期状态过期。如果您回想一下“ [乘车和票价”练习](https://github.com/apache/flink-training/tree/release-1.11/rides-and-fares)，其中使用a`RichCoFlatMapFunction`来计算简单的联接，则示例解决方案假定“的士”和“的士票价”是一对一的完美匹配`rideId`。如果某个事件丢失，则该事件的另一个事件`rideId`将永远保持状态。而是可以将其实现为`KeyedCoProcessFunction`，并且可以使用计时器来检测和清除任何过时的状态。

