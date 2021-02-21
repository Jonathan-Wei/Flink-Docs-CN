# 处理函数

## ProcessFunction

`ProcessFunction`是一个低级流处理操作，可以访问所有（非循环）流应用程序的基本构建块：

* 事件（流元素）
* 状态（容错，一致，仅在键控流上）
* 定时器（事件时间和处理时间，仅限键控流）

`过程函数(ProcessFunction)` 可以被认为一种提供了对有键状态\(keyed state\)和定时器\(timers\)访问的 `FlatMapFunction`。每在输入流中收到一个事件，过程函数就会被触发来对事件进行处理。

对于容错的状态\(state\), `过程函数(ProcessFunction)` 可以通过 `RuntimeContext` 访问Flink's [有键状态\(keyed state\)](https://github.com/flink-china/flink-china-doc/blob/master/dev/stream/state.html), 就像其它状态函数能够访问有键状态\(keyed state\)一样.

定时器则允许程序对处理时间和[事件时间\(event time\)](https://github.com/flink-china/flink-china-doc/blob/master/dev/event_time.html)的改变做出反应。每次对 `processElement(...)` 的调用都能拿到一个`上下文(Context)`对象,这个对象能访问到所处理元素事件时间的时间戳,还有 _定时服务器\(TimerService\)_ 。`定时服务器(TimerService)`可以为尚未发生的处理时间或事件时间实例注册回调函数。当一个定时器到达特定的时间实例时，`onTimer(...)`方法就会被调用。在这个函数的调用期间，所有的状态\(states\)都会再次对应定时器被创建时key所属的states，同时被触发的回调函数也能操作这些状态。

{% hint style="info" %}
注意 如果你希望访问有键状态\(keyed state\)和定时器\(timers\),你必须在一个键型流\(keyed stream\)上使用`过程函数(ProcessFunction)`:
{% endhint %}

```text
stream.keyBy(...).process(new MyProcessFunction())
```

## 低层级关联\(Low-level Joins\)

为了在两个输入源实现低层次的操作，应用可以使用 `CoProcessFunction`。该函数绑定了连个不同的输入源并且会对从两个输入源中得到的记录分别调用 `processElement1(...)` 和 `processElement2(...)` 方法。

可以按下面的步骤来实现一个低层典型的连接操作：

* 为一个\(或两个\)输入源创建一个状态\(state\)对象
* 在从输入源收到元素时更新这个状态\(state\)对象
* 在从另一个输入源接收到元素时，扫描这个state对象并产出连接的结果

比如，你正在把顾客数据和交易数据做一个连接，并且为顾客数据保存了状态\(state\)。如果你担心因为事件乱序导致不能得到完整和准确的连接结果，你可以用定时器来 控制，当顾客数据的水位线\(watermark\)时间超过了那笔交易的时间时，再进行计算和产出连接的结果。

## 示例

下面的例子中每一个键维护了一个计数，并且会把一分钟\(事件时间\)内没有更新的键/值对输出:

* 计数、键和最后一次更新时间存储在该键隐式持有的 `ValueState` 中
* 对于每一条记录，`过程函数(ProcessFunction)` 会对这个键对应的 `ValueState` 增加计数器的值，并且调整最后一次更新时间
* 该 `过程函数(ProcessFunction)` 也会注册一个一分钟\(事件时间\)后的回调函数
* 每一次回调触发时，它会检查回调事件的时间戳和存在 `ValueState` 中最后一次更新的时间戳是否符合要求\(比如，在过去的一分钟没有再发生更新\)，如果符合要求则会把键/计数对传出来

{% hint style="info" %}
注意 这个简单的列子本来可以通过会话窗口来实现。我们在这里使用 `过程函数(ProcessFunction)` 来举例说明它的基本模式。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.ProcessFunction;
import org.apache.flink.streaming.api.functions.ProcessFunction.Context;
import org.apache.flink.streaming.api.functions.ProcessFunction.OnTimerContext;
import org.apache.flink.util.Collector;


// the source data stream
DataStream<Tuple2<String, String>> stream = ...;

// apply the process function onto a keyed stream
DataStream<Tuple2<String, Long>> result = stream
    .keyBy(0)
    .process(new CountWithTimeoutFunction());

/**
 * The data type stored in the state
 */
public class CountWithTimestamp {

    public String key;
    public long count;
    public long lastModified;
}

/**
 * The implementation of the ProcessFunction that maintains the count and timeouts
 */
public class CountWithTimeoutFunction extends ProcessFunction<Tuple2<String, String>, Tuple2<String, Long>> {

    /** The state that is maintained by this process function */
    private ValueState<CountWithTimestamp> state;

    @Override
    public void open(Configuration parameters) throws Exception {
        state = getRuntimeContext().getState(new ValueStateDescriptor<>("myState", CountWithTimestamp.class));
    }

    @Override
    public void processElement(Tuple2<String, String> value, Context ctx, Collector<Tuple2<String, Long>> out)
            throws Exception {

        // retrieve the current count
        CountWithTimestamp current = state.value();
        if (current == null) {
            current = new CountWithTimestamp();
            current.key = value.f0;
        }

        // update the state's count
        current.count++;

        // set the state's timestamp to the record's assigned event time timestamp
        current.lastModified = ctx.timestamp();

        // write the state back
        state.update(current);

        // schedule the next timer 60 seconds from the current event time
        ctx.timerService().registerEventTimeTimer(current.lastModified + 60000);
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Tuple2<String, Long>> out)
            throws Exception {

        // get the state for the key that scheduled the timer
        CountWithTimestamp result = state.value();

        // check if this is an outdated timer or the latest timer
        if (timestamp == result.lastModified + 60000) {
            // emit the state on timeout
            out.collect(new Tuple2<String, Long>(result.key, result.count));
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.api.common.state.ValueState
import org.apache.flink.api.common.state.ValueStateDescriptor
import org.apache.flink.streaming.api.functions.ProcessFunction
import org.apache.flink.streaming.api.functions.ProcessFunction.Context
import org.apache.flink.streaming.api.functions.ProcessFunction.OnTimerContext
import org.apache.flink.util.Collector

// the source data stream
val stream: DataStream[Tuple2[String, String]] = ...

// apply the process function onto a keyed stream
val result: DataStream[Tuple2[String, Long]] = stream
  .keyBy(0)
  .process(new CountWithTimeoutFunction())

/**
  * The data type stored in the state
  */
case class CountWithTimestamp(key: String, count: Long, lastModified: Long)

/**
  * The implementation of the ProcessFunction that maintains the count and timeouts
  */
class CountWithTimeoutFunction extends ProcessFunction[(String, String), (String, Long)] {

  /** The state that is maintained by this process function */
  lazy val state: ValueState[CountWithTimestamp] = getRuntimeContext
    .getState(new ValueStateDescriptor[CountWithTimestamp]("myState", classOf[CountWithTimestamp]))


  override def processElement(value: (String, String), ctx: Context, out: Collector[(String, Long)]): Unit = {
    // initialize or retrieve/update the state

    val current: CountWithTimestamp = state.value match {
      case null =>
        CountWithTimestamp(value._1, 1, ctx.timestamp)
      case CountWithTimestamp(key, count, lastModified) =>
        CountWithTimestamp(key, count + 1, ctx.timestamp)
    }

    // write the state back
    state.update(current)

    // schedule the next timer 60 seconds from the current event time
    ctx.timerService.registerEventTimeTimer(current.lastModified + 60000)
  }

  override def onTimer(timestamp: Long, ctx: OnTimerContext, out: Collector[(String, Long)]): Unit = {
    state.value match {
      case CountWithTimestamp(key, count, lastModified) if (timestamp == lastModified + 60000) =>
        out.collect((key, count))
      case _ =>
    }
  }
}
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
注意：在Flink 1.4.0之前，当从处理时间计时器调用时，`processFunction.ontimer()`方法将当前处理时间设置为事件时间戳。这种行为非常微妙，用户可能不会注意到。这是有问题的，因为处理时间戳是不确定的，并且不与`Watermarks`对齐。此外，用户实现的逻辑依赖于这个错误的时间戳，很可能是无意中出错的。所以我们决定把它修复。升级到1.4.0后，使用此错误事件时间戳的Flink作业将失败，用户应根据正确的逻辑调整其作业
{% endhint %}

## KeyedProcessFunction

`KeyedProcessFunction`作为`ProcessFunction`的扩展，可以在其`onTimer(...)` 方法中访问计时器的Key。

{% tabs %}
{% tab title="Java" %}
```java
@Override
public void onTimer(long timestamp, OnTimerContext ctx, Collector<OUT> out) throws Exception {
    K key = ctx.getCurrentKey();
    // ...
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
override def onTimer(timestamp: Long, ctx: OnTimerContext, out: Collector[OUT]): Unit = {
  var key = ctx.getCurrentKey
  // ...
}
```
{% endtab %}
{% endtabs %}

## 计时器\(Timers\)

两种类型的计时器（处理时间和事件时间）都由`TimerService`在内部维护并排队等待执行。

TimerService对应每个键和时间戳。即，每个键和时间戳最多有一个计时器。如果为相同的时间戳注册了多个计时器，则只调用onTimer\(\)方法一次。

{% hint style="info" %}
Flink同步调用`ontimer()`和`processElement()`。因此，用户不必担心状态的并发修改。
{% endhint %}

### 容错

计时器与应用程序的状态一起具有容错和检查点功能。在故障恢复或从保存点启动应用程序时，计时器将被恢复。

{% hint style="info" %}
注意：检查点处理时间计时器应该在恢复之前触发，但将立即触发。当应用程序从故障中恢复或从保存点启动时，可能会发生这种情况。
{% endhint %}

{% hint style="info" %}
注意，计时器总是异步地进行检查点，除了RocksDB后端/与增量快照/与基于堆的计时器的组合之外\(将使用FLINK-10026进行解析\)。注意，大量的计时器会增加检查点时间，因为计时器是检查点状态的一部分。有关如何减少计时器数量的建议，请参阅“计时器合并”一节。
{% endhint %}

### 计时器合并

因为Flink只为每个键和时间戳维护一个计时器，所以可以通过减少计时器的分辨率来合并它们，从而减少计时器的数量。

对于计时器分辨率为1秒\(事件或处理时间\)的情况，可以将目标时间四舍五入为完整秒。计时器的触发时间最多提前1秒，但不会晚于所请求的毫秒精度。因此，每个键和秒最多有一个计时器。

{% tabs %}
{% tab title="Java" %}
```java
long coalescedTime = ((ctx.timestamp() + timeout) / 1000) * 1000;
ctx.timerService().registerProcessingTimeTimer(coalescedTime);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val coalescedTime = ((ctx.timestamp + timeout) / 1000) * 1000
ctx.timerService.registerProcessingTimeTimer(coalescedTime)
```
{% endtab %}
{% endtabs %}

因为事件时间计时器只在有水印的时候触发，你也可以使用当前的一个来调度和合并这些计时器和下一个水印:

{% tabs %}
{% tab title="Java" %}
```java
long coalescedTime = ctx.timerService().currentWatermark() + 1;
ctx.timerService().registerEventTimeTimer(coalescedTime);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val coalescedTime = ctx.timerService.currentWatermark + 1
ctx.timerService.registerEventTimeTimer(coalescedTime)
```
{% endtab %}
{% endtabs %}

也可以按以下方式停止和删除计时器：

停止处理时间计时器：

{% tabs %}
{% tab title="Java" %}
```java
long timestampOfTimerToStop = ...
ctx.timerService().deleteProcessingTimeTimer(timestampOfTimerToStop);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val timestampOfTimerToStop = ...
ctx.timerService.deleteProcessingTimeTimer(timestampOfTimerToStop)
```
{% endtab %}
{% endtabs %}

停止事件时间计时器：

{% tabs %}
{% tab title="Java" %}
```java
long timestampOfTimerToStop = ...
ctx.timerService().deleteEventTimeTimer(timestampOfTimerToStop);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val timestampOfTimerToStop = ...
ctx.timerService.deleteEventTimeTimer(timestampOfTimerToStop)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
注意：如果未注册具有给定时间戳记的计时器，则停止计时器无效。
{% endhint %}

