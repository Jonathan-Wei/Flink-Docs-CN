# 侧输出

除了来自DataStream操作的主流之外，还可以生成任意数量的附加侧输出结果流。结果流中的数据类型不必与主流中的数据类型匹配，而且不同侧输出的类型也可能不同。当您希望拆分\(通常需要复制\)数据流时，然后从每个流中过滤出不想要的数据时，此操作非常有用。

在使用侧输出时，首先需要定义一个`OutputTag`，用于标识侧输出流:

{% tabs %}
{% tab title="Java" %}
```java
// this needs to be an anonymous inner class, so that we can analyze the type
OutputTag<String> outputTag = new OutputTag<String>("side-output") {};
```
{% endtab %}

{% tab title="Scala" %}
```scala
val outputTag = OutputTag[String]("side-output")
```
{% endtab %}
{% endtabs %}

注意`OutputTag`是如何根据侧输出流包含的元素类型键入的。

可以通过以下函数将数据发送到侧输出

* [ProcessFunction](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/process_function.html)
* CoProcessFunction
* [ProcessWindowFunction](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html#processwindowfunction)
* ProcessAllWindowFunction

您可以使用上下文参数\(在上述函数中公开给用户\)将数据发送到OutputTag标识的一个侧输出。下面是一个从`ProcessFunction`发送端输出数据的示例:

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Integer> input = ...;

final OutputTag<String> outputTag = new OutputTag<String>("side-output"){};

SingleOutputStreamOperator<Integer> mainDataStream = input
  .process(new ProcessFunction<Integer, Integer>() {

      @Override
      public void processElement(
          Integer value,
          Context ctx,
          Collector<Integer> out) throws Exception {
        // emit data to regular output
        out.collect(value);

        // emit data to side output
        ctx.output(outputTag, "sideout-" + String.valueOf(value));
      }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[Int] = ...
val outputTag = OutputTag[String]("side-output")

val mainDataStream = input
  .process(new ProcessFunction[Int, Int] {
    override def processElement(
        value: Int,
        ctx: ProcessFunction[Int, Int]#Context,
        out: Collector[Int]): Unit = {
      // emit data to regular output
      out.collect(value)

      // emit data to side output
      ctx.output(outputTag, "sideout-" + String.valueOf(value))
    }
  })
```
{% endtab %}
{% endtabs %}

要检索侧输出流，可以在DataStream操作的结果上使用`getSideOutput(OutputTag)`。这将给你一个DataStream，输入到输出流的结果:

{% tabs %}
{% tab title="Java" %}
```java
final OutputTag<String> outputTag = new OutputTag<String>("side-output"){};

SingleOutputStreamOperator<Integer> mainDataStream = ...;

DataStream<String> sideOutputStream = mainDataStream.getSideOutput(outputTag);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val outputTag = OutputTag[String]("side-output")

val mainDataStream = ...

val sideOutputStream: DataStream[String] = mainDataStream.getSideOutput(outputTag)
```
{% endtab %}
{% endtabs %}

