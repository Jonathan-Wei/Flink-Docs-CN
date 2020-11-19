# 概述

Flink中的DataStream程序是常规程序，可对数据流执行转换（例如，过滤，更新状态，定义窗口，聚合）。最初从各种来源（例如，消息队列，套接字流，文件）创建数据流。结果通过接收器返回，接收器可以将数据写入文件或标准输出（例如命令行终端）。Flink程序可以在各种上下文中运行，独立运行或嵌入其他程序中。执行可以在本地JVM或许多计算机的群集中进行。

请参阅[基本概念](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/api_concepts.html)，以了解Flink API的基本概念。

为了创建自己的Flink DataStream程序，我们建议您从Flink程序的解剖开始， 并逐步添加自己的流转换。其余部分用作其他操作和高级功能的参考。

## 什么是数据流

DataStream API的名称来自于`DataStream`类，DataStream类用于表示Flink程序中的一组数据。你可以将它们视为可包含重复项的不可变数据集合。这些数据可以是有限的，也可以是无限的，用于处理它们的API是相同的。

就用法而言，DataStream与常规Java集合类似，但在一些关键方面有很大不同。它们是不可变的，这意味着一旦创建了它们，你就不能添加或删除元素。你也可以不能检查内部的元素，而是只能使用DataStream API操作对它们进行操作，这些操作也称为转换。

你可以通过在Flink程序中添加源来创建初始DataStream。然后可以从中派生出新的流，并通过使用诸如map、filter等API方法将它们组合起来。

## Flink程序剖析

Flink程序看起来就像转换数据表的常规程序。每个程序都由相同的基本部分组成:

1. 获得`execution environment`，
2. 加载/创建初始数据，
3. 指定对此数据的转换，
4. 指定将计算结果放在何处，
5. 触发程序执行

{% tabs %}
{% tab title="Java" %}
现在，我们将对每个步骤进行概述，有关更多详细信息，请参阅相应的章节。请注意，可以在[org.apache.flink.streaming.api中](https://github.com/apache/flink/blob/master//flink-streaming-java/src/main/java/org/apache/flink/streaming/api)找到Java DataStream API的所有核心类。

`StreamExecutionEnvironment`是所有Flink程序的基础。可以使用以下静态方法获得一个`StreamExecutionEnvironment`：

```text
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(String host, int port, String... jarFiles)
```



```text
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<String> text = env.readTextFile("file:///path/to/file");
```



```text
DataStream<String> input = ...;

DataStream<Integer> parsed = input.map(new MapFunction<String, Integer>() {
    @Override
    public Integer map(String value) {
        return Integer.parseInt(value);
    }
});
```



```text
writeAsText(String path)

print()
```



```text
final JobClient jobClient = env.executeAsync();

final JobExecutionResult jobExecutionResult = jobClient.getJobExecutionResult(userClassloader).get();
```
{% endtab %}

{% tab title="Scala" %}
现在，我们将对每个步骤进行概述，有关更多详细信息，请参阅相应的章节。请注意，可以在[org.apache.flink.streaming.api.scala中](https://github.com/apache/flink/blob/master//flink-streaming-scala/src/main/scala/org/apache/flink/streaming/api/scala)找到Scala DataStream API的所有核心类。

`StreamExecutionEnvironment`是所有Flink程序的基础。可以使用以下静态方法获得一个`StreamExecutionEnvironment`：

```text
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(host: String, port: Int, jarFiles: String*)
```



```text
val env = StreamExecutionEnvironment.getExecutionEnvironment()

val text: DataStream[String] = env.readTextFile("file:///path/to/file")
```



```text
val input: DataSet[String] = ...

val mapped = input.map { x => x.toInt }
```



```text
writeAsText(path: String)

print()
```



```text
final JobClient jobClient = env.executeAsync();

final JobExecutionResult jobExecutionResult = jobClient.getJobExecutionResult(userClassloader).get();
```
{% endtab %}
{% endtabs %}

## 示例程序

以下程序是流窗口字数统计应用程序的完整工作示例，它在5秒窗口中对来自Web Socket的单词进行计数。可以复制并粘贴代码以在本地运行它。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.windowing.time.Time;
import org.apache.flink.util.Collector;

public class WindowWordCount {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Tuple2<String, Integer>> dataStream = env
                .socketTextStream("localhost", 9999)
                .flatMap(new Splitter())
                .keyBy(0)
                .timeWindow(Time.seconds(5))
                .sum(1);

        dataStream.print();

        env.execute("Window WordCount");
    }

    public static class Splitter implements FlatMapFunction<String, Tuple2<String, Integer>> {
        @Override
        public void flatMap(String sentence, Collector<Tuple2<String, Integer>> out) throws Exception {
            for (String word: sentence.split(" ")) {
                out.collect(new Tuple2<String, Integer>(word, 1));
            }
        }
    }

}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time

object WindowWordCount {
  def main(args: Array[String]) {

    val env = StreamExecutionEnvironment.getExecutionEnvironment
    val text = env.socketTextStream("localhost", 9999)

    val counts = text.flatMap { _.toLowerCase.split("\\W+") filter { _.nonEmpty } }
      .map { (_, 1) }
      .keyBy(0)
      .timeWindow(Time.seconds(5))
      .sum(1)

    counts.print()

    env.execute("Window Stream WordCount")
  }
}
```
{% endtab %}
{% endtabs %}

要运行示例程序，首先从终端使用netcat启动输入流：

```text
nc -lk 9999
```

只需输入一些单词即可，输入多个单词以请按回车键后再输入。所输入的单词将是单词计数程序的输入。如果要查看大于1的计数，请在5秒内反复键入相同的单词（如果无法快速输入，则将窗口大小从扩大，大于5秒）。

## 数据源\(Data Souces\)

Source是程序读取输入的地方。您可以使用`StreamExecutionEnvironment.addSource(sourceFunction)`将Source添加到程序中。Flink附带了许多预先实现的`SourceFunction`，Flink附带了许多预先实现的源函数，但您可以通过实现`SourceFunction` 非并行源，或通过实现`ParallelSourceFunction`接口或扩展`RichParallelSourceFunction`并行源来编写自己的自定义源。

有几个预定义的流源可以从`StreamExecutionEnvironment`访问:

### 基于文件的：

{% tabs %}
{% tab title="Java" %}
* `readTextFile(path)`- 逐行读取文本文件，即符合`TextInputFormat`规范的文件，并将其作为字符串返回。
* `readFile(fileInputFormat, path)` - 按指定的文件输入格式指定读取（一次）文件。
* `readFile(fileInputFormat, path, watchType, interval, pathFilter, typeInfo)` - 这是前两个方法在内部调用的方法。它根据给定的`FileInputFormat`读取路径中的文件。根据所提供的`watchType`，这个源可以定期监视\(每隔一段时间\)新数据的路径

  `(FileProcessingMode.PROCESS_CONTINUOUSLY)`，或者处理当前路径中的数据并退出`(FileProcessingMode.PROCESS_ONCE)`。使用`pathFilter`，用户可以进一步排除正在处理的文件。
{% endtab %}

{% tab title="Scala" %}
* `readTextFile(path)`- 逐行读取文本文件，即符合`TextInputFormat`规范的文件，并将其作为字符串返回。
* `readFile(fileInputFormat, path)` - 按指定的文件输入格式指定读取（一次）文件。
* `readFile(fileInputFormat, path, watchType, interval, pathFilter)` - 这是前两个方法在内部调用的方法。它根据给定的`FileInputFormat`读取路径中的文件。根据所提供的`watchType`，这个源可以定期监视\(每隔一段时间\)新数据的路径

  `(FileProcessingMode.PROCESS_CONTINUOUSLY)`，或者处理当前路径中的数据并退出`(FileProcessingMode.PROCESS_ONCE)`。使用`pathFilter`，用户可以进一步排除正在处理的文件。
{% endtab %}
{% endtabs %}

_实现：_

在底层，Flink将文件读取过程分为两个子任务，即目录监视和数据读取。这些子任务都由单独的实体实现。监视由单个非并行\(并行度= 1\)任务实现，而读取由多个并行运行的任务执行。后者的并行性等于作业的并行性。单个监视任务的作用是扫描目录\(根据watchType定期或仅扫描一次\)，查找要处理的文件，将它们分割，并将这些分割分配给下游的读取器。读者是将阅读实际数据的人。每个分割仅由一个读取器读取，而读取器可以逐个读取多个分割。

{% hint style="info" %}
重要提示:

1. 1. 如果watchType设置为**FileProcessingMode.PROCESS\_CONTINUOUSLY**，则在修改文件时，将完全重新处理其内容。这可以打破“完全一次”的语义，因为在文件末尾附加数据将导致其所有内容被重新处理。 
   2. 如果watchType设置为**FileProcessingMode.PROCESS\_ONCE**，则源扫描路径一次并退出，而不等待读者完成读取文件内容。当然读者将继续阅读，直到读取所有文件内容。在该点之后关闭源将导致不再有检查点。这可能会导致节点发生故障后恢复速度变慢，因为作业将从上一个检查点恢复读取。 
{% endhint %}

### 基于Socket

`socketTextStream` - 从Socket读取。元素可以用分隔符分隔。

### 基于集合

{% tabs %}
{% tab title="Java" %}
* `fromCollection(Collection)`——从Java.util.Collection创建数据流。集合中的所有元素必须是相同类型的。
* `fromCollection(Iterator, Class)`——从迭代器创建数据流。该类指定迭代器返回的元素的数据类型。
* `fromElements(T…)`——给定的对象序列中创建数据流。所有对象必须具有相同的类型。
* `fromParallelCollection(SplittableIterator,Class)`——并行地从迭代器创建数据流。该类指定迭代器返回的元素的数据类型。
* `generateSequence(from,to)`——在给定的区间内并行生成数字序列。
{% endtab %}

{% tab title="Scala" %}
* `fromCollection(Seq)`——从Java.util.Collection创建数据流。集合中的所有元素必须是相同类型的。
* `fromCollection(Iterator)`——从迭代器创建数据流。该类指定迭代器返回的元素的数据类型。
* `fromElements(elements:_*)`——给定的对象序列中创建数据流。所有对象必须具有相同的类型。
* `fromParallelCollection(SplittableIterator)`——并行地从迭代器创建数据流。该类指定迭代器返回的元素的数据类型。
* `generateSequence(from,to)`——在给定的区间内并行生成数字序列。
{% endtab %}
{% endtabs %}

### 自定义

* `addSource` - 附加新的`SourceFunction`。例如，要从Apache Kafka中读取，您可以使用 `addSource(new FlinkKafkaConsumer08<>(...))`。请参阅[连接器](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/)以获取更多详

## DataStream转换

有关可用流转换的概述，请参阅[运算符](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/index.html)。

## 数据接收器\(Data Sinks\)

Data Sink接收数据流并将其转发到文件、Socket、外部系统或打印它们。Flink带有各种内置输出格式，这些格式封装在`DataStreams`上的操作后面：

* `writeAsText()`/ `TextOutputFormat`- 按字符串顺序写入元素。通过调用每个元素的_`toString()`_方法获得字符串。
* `writeAsCsv(...)`/ `CsvOutputFormat`- 将元组写为逗号分隔值文件。行和字段分隔符是可配置的。每个字段的值来自对象的_`toString()`_方法。
* `print()`/ `printToErr()` - 在标准输出/标准错误流上打印每个元素的_`toString()`_值。还可以选择在输出之前提供前缀\(msg\)。这可以帮助区分不同的打印调用。如果并行度大于1，则输出还将加上生成输出的任务的标识符。
* `writeUsingOutputFormat()`/ `FileOutputFormat`- 自定义文件输出的方法和基类。支持自定义对象到字节的转换。
* `writeToSocket` - 根据`SerializationSchema`将元素写入Socket
* `addSink` - 调用自定义接收器功能。Flink捆绑了其他系统（如Apache Kafka）的连接器，这些系统实现为接收器功能。

{% hint style="danger" %}
注意，DataStream上的`write*()`方法主要用于调试。它们不参与Flink的检查点，这意味着这些函数通常具有至少一次的语义。目标系统的数据刷新取决于`OutputFormat`的实现。这意味着并非所有发送到`OutputFormat`的元素都立即显示在目标系统中。此外，在失败的情况下，这些记录可能会丢失。
{% endhint %}

为了可靠、准确地将流交付到文件系统中，请使用`flink-connector-filesystem`。另外，通过. `addsink(...)`方法的自定义实现可以参与Flink的检查点，实现精确的一次语义。

## 迭代\(Iterations\)

{% tabs %}
{% tab title="Java" %}
迭代流程序实现一个步骤函数并将其嵌入`IterativeStream`中。由于`DataStream`程序可能永远执行，因此没有最大迭代次数。相反，需要指定流的哪个部分反馈到迭代，哪个部分使用`split`转换或转发到下游`filter`。这里，我们展示了一个使用过滤器的示例。首先，我们定义一个`IterativeStream`

```java
IterativeStream<Integer> iteration = input.iterate();
```

然后，我们使用一系列转换指定将在循环内执行的逻辑（这里是一个简单的`map`转换）

```java
DataStream<Integer> iterationBody = iteration.map(/* this is executed many times */);
```

要结束一个迭代并定义迭代尾部，调用`IterativeStream`的`closeWith(feedbackStream)`方法。给`closeWith`函数的数据流将反馈给迭代头。一种常见的模式是使用筛选器来分离被反馈的流部分和向前传播的流部分。例如，这些过滤器可以定义“终止”逻辑，其中允许元素向下传播而不是被反馈。

```java
iteration.closeWith(iterationBody.filter(/* one part of the stream */));
DataStream<Integer> output = iterationBody.filter(/* some other part of the stream */);
```

例如，这里有一个程序，它连续地从一系列整数中减去1，直到它们达到零：

```java
DataStream<Long> someIntegers = env.generateSequence(0, 1000);

IterativeStream<Long> iteration = someIntegers.iterate();

DataStream<Long> minusOne = iteration.map(new MapFunction<Long, Long>() {
  @Override
  public Long map(Long value) throws Exception {
    return value - 1 ;
  }
});

DataStream<Long> stillGreaterThanZero = minusOne.filter(new FilterFunction<Long>() {
  @Override
  public boolean filter(Long value) throws Exception {
    return (value > 0);
  }
});

iteration.closeWith(stillGreaterThanZero);

DataStream<Long> lessThanZero = minusOne.filter(new FilterFunction<Long>() {
  @Override
  public boolean filter(Long value) throws Exception {
    return (value <= 0);
  }
});
```
{% endtab %}

{% tab title="Scala" %}
迭代流程序实现一个步骤函数并将其嵌入`IterativeStream`中。由于`DataStream`程序可能永远执行，因此没有最大迭代次数。相反，需要指定流的哪个部分反馈到迭代，哪个部分使用`split`转换或转发到下游`filter。`在这里，我们展示了一个示例迭代，其中主体\(重复计算的部分\)是一个简单的映射转换，通过使用过滤器转发到下游的元素来区分反馈的元素。

```scala
val iteratedStream = someDataStream.iterate(
  iteration => {
    val iterationBody = iteration.map(/* this is executed many times */)
    (iterationBody.filter(/* one part of the stream */), iterationBody.filter(/* some other part of the stream */))
})
```

例如，这里有一个程序，它连续地从一系列整数中减去1，直到它们达到零：

```scala
val someIntegers: DataStream[Long] = env.generateSequence(0, 1000)

val iteratedStream = someIntegers.iterate(
  iteration => {
    val minusOne = iteration.map( v => v - 1)
    val stillGreaterThanZero = minusOne.filter (_ > 0)
    val lessThanZero = minusOne.filter(_ <= 0)
    (stillGreaterThanZero, lessThanZero)
  }
)
```
{% endtab %}
{% endtabs %}

## 执行参数

`StreamExecutionEnvironment`包含`ExecutionConfig`，它允许为运行时设置特定于作业的配置值。

更多相关参数的说明，请参阅[执行配置](https://ci.apache.org/projects/flink/flink-docs-master/dev/execution_configuration.html)。这些参数特别适用于DataStream API:

* `setAutoWatermarkInterval(long milliseconds)`：设置自动Watermark发射的间隔。您可以使用获取当前值`long getAutoWatermarkInterval()`

### 容错

[State＆Checkpointing](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/checkpointing.html)描述了如何启用和配置Flink的检查点机制。

### 控制延迟

默认情况下，元素不会在网络上逐个传输\(这会导致不必要的网络流量\)，而是被缓冲。缓冲区的大小\(实际上是在机器之间传输的\)可以在Flink配置文件中设置。虽然这种方法可以很好地优化吞吐量，但当传入流的速度不够快时，它可能会导致延迟问题。为了控制吞吐量和延迟，可以在执行环境\(或单个操作符\)上使用`env.setBufferTimeout(timeoutMillis)`设置缓冲区被填满的最大等待时间。在此之后，即使缓冲区没有满，也会自动发送缓冲区。此超时的默认值是100 ms。

用法：

{% tabs %}
{% tab title="Java" %}
```java
LocalStreamEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();
env.setBufferTimeout(timeoutMillis);

env.generateSequence(1,10).map(new MyMapper()).setBufferTimeout(timeoutMillis);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env: LocalStreamEnvironment = StreamExecutionEnvironment.createLocalEnvironment
env.setBufferTimeout(timeoutMillis)

env.generateSequence(1,10).map(myMap).setBufferTimeout(timeoutMillis)
```
{% endtab %}
{% endtabs %}

## Debugging

在分布式集群中运行流程序之前，最好确保所实现的算法能够正常工作。因此，实现数据分析程序通常是检查结果、调试和改进的增量过程。

通过支持IDE中的本地调试、测试数据的注入和结果数据的收集，Flink提供了显著简化数据分析程序开发过程的特性。本节给出了一些如何简化Flink程序开发的提示。

### 本地执行环境

`LocalStreamEnvironment`在它创建的JVM进程中启动Flink系统。如果从IDE启动`LocalEnvironment`，可以在代码中设置断点并轻松调试程序。

创建并使用`LocalEnvironment`如下所示:

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();

DataStream<String> lines = env.addSource(/* some source */);
// build your program

env.execute();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.createLocalEnvironment()

val lines = env.addSource(/* some source */)
// build your program

env.execute()
```
{% endtab %}
{% endtabs %}

### Collection数据源

Flink提供了特殊的数据源，这些数据源由Java集合支持，以方便测试。一旦程序经过测试，Source和Sink可以很容易地被读取/写入外部系统的Source和Sink替换。

集合数据源可以使用如下：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.createLocalEnvironment();

// Create a DataStream from a list of elements
DataStream<Integer> myInts = env.fromElements(1, 2, 3, 4, 5);

// Create a DataStream from any Java collection
List<Tuple2<String, Integer>> data = ...
DataStream<Tuple2<String, Integer>> myTuples = env.fromCollection(data);

// Create a DataStream from an Iterator
Iterator<Long> longIt = ...
DataStream<Long> myLongs = env.fromCollection(longIt, Long.class);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.createLocalEnvironment()

// Create a DataStream from a list of elements
val myInts = env.fromElements(1, 2, 3, 4, 5)

// Create a DataStream from any Collection
val data: Seq[(String, Int)] = ...
val myTuples = env.fromCollection(data)

// Create a DataStream from an Iterator
val longIt: Iterator[Long] = ...
val myLongs = env.fromCollection(longIt)
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**注意：**目前，集合数据源要求数据类型和迭代器实现`Serializable`。此外，集合数据源不能并行执行（并行度= 1）。
{% endhint %}

### Iterator Data Sink

Flink还提供了一个Sink，用于收集DataStream结果以进行测试和调试。它可以使用如下

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.streaming.experimental.DataStreamUtils

DataStream<Tuple2<String, Integer>> myResult = ...
Iterator<Tuple2<String, Integer>> myOutput = DataStreamUtils.collect(myResult)
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.streaming.experimental.DataStreamUtils
import scala.collection.JavaConverters.asScalaIteratorConverter

val myResult: DataStream[(String, Int)] = ...
val myOutput: Iterator[(String, Int)] = DataStreamUtils.collect(myResult.javaStream).asScala
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
**注意：** `flink-streaming-contrib`模块已从Flink 1.5.0中删除。相关的类已被移入`flink-streaming-java`和`flink-streaming-scala`。
{% endhint %}

