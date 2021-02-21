# Joining

## 窗口关联

![](../../../.gitbook/assets/image%20%2866%29.png)

窗口关联联接共享一个公共键并位于同一窗口中的两个流的元素。这些窗口可以使用窗口分配程序定义，并对来自两个流的元素进行计算。

然后将两侧的元素传递给用户定义的`JoinFunction`或`FlatjoinFunction`，用户可以在其中发出满足关联条件的结果。

一般用法总结如下：

```text
stream.join(otherStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(<WindowAssigner>)
    .apply(<JoinFunction>)
```

关于语义的一些注释：

* 两个流元素的成对组合的创建行为类似于内部关联，这意味着如果一个流中的元素没有要关联的另一个流中的对应元素，则不会发出这些元素。
* 那些被关联的元素将作为它们的时间戳，最大的时间戳仍然位于各自的窗口中。例如，以\[5，10\]为边界的窗口将导致被关联元素的时间戳为9。

在下面的部分中，我们将使用一些示例性场景概述不同类型的窗口关联的行为。

### 滚动窗口关联

在执行滚动窗口关联时，具有公共键和公共滚动窗口的所有元素都作为成对组合进行联接，并传递给`JoinFunction`或`FlatJoinFunction`。因为它的行为类似于内部关联，所以一个流的元素在其滚动窗口中没有来自另一个流的元素不会被emit！

![](../../../.gitbook/assets/tumbling-window-join.svg)

如图所示，我们定义了一个2毫秒大小的滚动窗口，其结果是窗口形式为\[0,1\]、\[2,3\]、……图中显示了将传递给`JoinFunction`的每个窗口中所有元素的成对组合。请注意，在翻滚窗口\[6,7\]中不会发出任何信号，因为绿色流中不存在与橙色元素⑥和⑦相连的元素。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.windowing.assigners.TumblingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
 
...

DataStream<Integer> orangeStream = ...
DataStream<Integer> greenStream = ...

orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(TumblingEventTimeWindows.of(Time.milliseconds(2)))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

...

val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream.join(greenStream)
    .where(elem => /* select key */)
    .equalTo(elem => /* select key */)
    .window(TumblingEventTimeWindows.of(Time.milliseconds(2)))
    .apply { (e1, e2) => e1 + "," + e2 }

```
{% endtab %}
{% endtabs %}

### 滑动窗口关联

在执行滑动窗口关联时，所有具有公共键和公共滑动窗口的元素都作为成对组合进行关联，并传递给`JoinFunction`或`FlatJoinFunction`。在当前滑动窗口中，一个流中没有来自另一个流的元素的元素不会emit！请注意，某些元素可能在一个滑动窗口中关联而在另一个滑动窗口中不关联

![](../../../.gitbook/assets/sliding-window-join.svg)

在本例中，我们使用的滑动窗口的大小为2毫秒，并将其滑动1毫秒，从而导致滑动窗口\[-1，0\]，\[0，1\]，\[1，2\]，\[2，3\]，……X轴下方的关联元素是传递给每个滑动窗口的`JoinFunction`的元素。在这里，还可以看到例如，橙色②如何与窗口\[2,3\]中的绿色③关联，但不与窗口\[1,2\]中的任何内容关联。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

...

DataStream<Integer> orangeStream = ...
DataStream<Integer> greenStream = ...

orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(SlidingEventTimeWindows.of(Time.milliseconds(2) /* size */, Time.milliseconds(1) /* slide */))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.windowing.assigners.SlidingEventTimeWindows;
import org.apache.flink.streaming.api.windowing.time.Time;

...

val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream.join(greenStream)
    .where(elem => /* select key */)
    .equalTo(elem => /* select key */)
    .window(SlidingEventTimeWindows.of(Time.milliseconds(2) /* size */, Time.milliseconds(1) /* slide */))
    .apply { (e1, e2) => e1 + "," + e2 }

```
{% endtab %}
{% endtabs %}

### Session窗口关联

在执行会话窗口关联时，所有具有与“组合\(_combined_\)”满足会话条件时相同键的元素将以成对组合方式联接，并传递给`JoinFunction`或`FlatjoinFunction`。同样，这将执行内部关联，因此如果会话窗口只包含一个流中的元素，则不会发出任何输出！

![](../../../.gitbook/assets/session-window-join.svg)

此处，我们定义一个会话窗口关联，其中每个会话被至少1毫秒的间隔分割。有三个会话，在前两个会话中，来自两个流的关联元素被传递给`JoinFunction`。在第三个会话中，绿色流中没有元素，因此⑧和⑨没有关联！

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.windowing.assigners.EventTimeSessionWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
 
...

DataStream<Integer> orangeStream = ...
DataStream<Integer> greenStream = ...

orangeStream.join(greenStream)
    .where(<KeySelector>)
    .equalTo(<KeySelector>)
    .window(EventTimeSessionWindows.withGap(Time.milliseconds(1)))
    .apply (new JoinFunction<Integer, Integer, String> (){
        @Override
        public String join(Integer first, Integer second) {
            return first + "," + second;
        }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.windowing.assigners.EventTimeSessionWindows;
import org.apache.flink.streaming.api.windowing.time.Time;
 
...

val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream.join(greenStream)
    .where(elem => /* select key */)
    .equalTo(elem => /* select key */)
    .window(EventTimeSessionWindows.withGap(Time.milliseconds(1)))
    .apply { (e1, e2) => e1 + "," + e2 }
```
{% endtab %}
{% endtabs %}

## 区间关联

区间关联用一个公共键连接两个流的元素（现在我们称它们为A&B），其中流B的元素具有时间戳，该时间戳位于流A中元素时间戳的相对时间间隔内。

这也可以更正式地表示为：

`b.timestamp∈[a.timestamp+lowerbound;a.timestamp+upperbound]`

或

`a.timestamp+lowerbound<=b.timestamp<=a.timestamp+upperbound`。

其中A和B是共享一个公共密钥的A和B的元素。下界和上界都可以是负数或正数，只要下界总是小于或等于上界，区间关联当前只执行内部关联。

当一对元素传递给`ProcessJoinFunction`时，它们将被分配两个元素的较大时间戳（可通过`ProcessJoinFunction.context`访问）。

{% hint style="info" %}
注意：区间关联当前仅支持EventTime
{% endhint %}

![](../../../.gitbook/assets/interval-join.svg)

在上面的示例中，我们将两个流“橙色”和“绿色”连接起来，它们的下界为-2毫秒，上界为+1毫秒。默认情况下，这些边界是包含的，但是`.lowerboundexclusive()`和`.upperboundexclusive()`可以应用于更改行为。

再用更正式的符号来表示

```text
orangeElem.ts + lowerBound <= greenElem.ts <= orangeElem.ts + upperBound
```

如三角形所示：

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.time.Time;

...

DataStream<Integer> orangeStream = ...
DataStream<Integer> greenStream = ...

orangeStream
    .keyBy(<KeySelector>)
    .intervalJoin(greenStream.keyBy(<KeySelector>))
    .between(Time.milliseconds(-2), Time.milliseconds(1))
    .process (new ProcessJoinFunction<Integer, Integer, String(){

        @Override
        public void processElement(Integer left, Integer right, Context ctx, Collector<String> out) {
            out.collect(first + "," + second);
        }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction;
import org.apache.flink.streaming.api.windowing.time.Time;

...

val orangeStream: DataStream[Integer] = ...
val greenStream: DataStream[Integer] = ...

orangeStream
    .keyBy(elem => /* select key */)
    .intervalJoin(greenStream.keyBy(elem => /* select key */))
    .between(Time.milliseconds(-2), Time.milliseconds(1))
    .process(new ProcessJoinFunction[Integer, Integer, String] {
        override def processElement(left: Integer, right: Integer, ctx: ProcessJoinFunction[Integer, Integer, String]#Context, out: Collector[String]): Unit = {
         out.collect(left + "," + right); 
        }
      });
    });
```
{% endtab %}
{% endtabs %}

