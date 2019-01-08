# 概述

## DataSet 和 DataStream

Flink程序是实现分布式集合转换的常规程序（例如，过滤，映射，更新状态，加入，分组，定义窗口，聚合）。集合最初是从源创建的（例如，通过读取文件，kafka topic或从本地内存中的集合）。结果通过Sink返回，接收器可以将数据写入（分布式）文件或标准输出（例如，命令行终端）。 Flink程序可以在各种环境中运行，独立运行或嵌入其他程序中。执行可以在本地JVM中执行，也可以在许多计算机的集群上执行。

根据数据源的类型，即有界或无界源，可以编写批处理程序或流程序，其中DataSet API用于批处理，DataStream API用于流式处理。本指南将介绍两种API共有的基本概念，但请参阅我们的[流媒体指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html)和[批处理指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html)，了解有关使用每个API编写程序的具体信息。

{% hint style="info" %}
注意：在展示如何使用API的实际示例时，我们将使用StreamingExecutionEnvironment和DataStream API。数据集API中的概念完全相同，只是替换为ExecutionEnvironment和DataSet。
{% endhint %}

## Flink程序剖析

Flink程序看起来像转换数据集合的常规程序。每个程序由相同的基本部分组成:

1. 获得一个execution environment
2. 加载/创建初始数据
3. 指定此数据上的转换
4. 指定放置计算结果的位置
5. 触发程序执行

{% tabs %}
{% tab title="Java" %}
我们现在将概述每个步骤，请参阅相应部分以获取更多详细信息。请注意，Java DataSet API的所有核心类都可以在[org.apache.flink.api.java](https://github.com/apache/flink/blob/master//flink-java/src/main/java/org/apache/flink/api/java)包中找到， 而Java DataStream API的类可以在[org.apache.flink.streaming.api中](https://github.com/apache/flink/blob/master//flink-streaming-java/src/main/java/org/apache/flink/streaming/api)找到 。

`StreamExecutionEnvironment`是所有Flink程序的基础。您可以使用以下静态方法获取一个`StreamExecutionEnvironment`：

```java
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(String host, int port, String... jarFiles)
```
{% endtab %}

{% tab title="Scala" %}
我们现在将概述每个步骤，请参阅相应部分以获取更多详细信息。请注意，Scala DataSet API的所有核心类都可以在[org.apache.flink.api.scala](https://github.com/apache/flink/blob/master//flink-scala/src/main/scala/org/apache/flink/api/scala)包中找到， 而Scala DataStream API的类可以在[org.apache.flink.streaming.api.scala中](https://github.com/apache/flink/blob/master//flink-streaming-scala/src/main/scala/org/apache/flink/streaming/api/scala)找到 。

  
`StreamExecutionEnvironment`是所有Flink程序的基础。您可以使用以下静态方法获取一个`StreamExecutionEnvironment`：

```scala
getExecutionEnvironment()

createLocalEnvironment()

createRemoteEnvironment(host: String, port: Int, jarFiles: String*)
```
{% endtab %}
{% endtabs %}

## 惰性评估

所有Flink程序都是延迟执行的:当程序的主方法执行时，数据加载和转换不会直接发生。相反，每个操作都被创建并添加到程序的计划中。当执行环境上的`execute()`调用显式地触发执行时，才会实际执行操作。程序是在本地执行还是在集群上执行取决于执行环境的类型。

惰性评估允许您构建复杂的程序，Flink作为一个整体规划的单元执行这些程序。

## 指定密钥

一些转换\(join、coGroup、keyBy、groupBy\)要求在元素集合上定义一个键。其他转换\(Reduce、GroupReduce、Aggregate、Windows\)允许在应用数据之前对它们进行分组。

DataSet被分组为：

```text
DataSet<...> input = // [...]
DataSet<...> reduced = input
  .groupBy(/*define key here*/)
  .reduceGroup(/*do something*/);
```

DataStream可以在使用的时候指定key

```text
DataStream<...> input = // [...]
DataStream<...> windowed = input
  .keyBy(/*define key here*/)
  .window(/*window specification*/);
```

Flink的数据模型不是基于键值对的。因此，不需要将数据集类型物理地打包为键和值。键是“虚拟的”:它们被定义为实际数据上的函数，以指导分组操作符。

{% hint style="info" %}
注意:在下面的讨论中，我们将使用DataStream API和keyBy。对于DataSet API，只需用DataSet和groupBy替换即可。
{% endhint %}

### 为Tuple定义Key

最简单的情况是对元组的一个或多个字段进行分组:

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Tuple3<Integer,String,Long>> input = // [...]
KeyedStream<Tuple3<Integer,String,Long>,Tuple> keyed = input.keyBy(0)
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[(Int, String, Long)] = // [...]
val keyed = input.keyBy(0)
```
{% endtab %}
{% endtabs %}

Tuple在第一个字段（整数类型）上分组。

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Tuple3<Integer,String,Long>> input = // [...]
KeyedStream<Tuple3<Integer,String,Long>,Tuple> keyed = input.keyBy(0,1)
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataSet[(Int, String, Long)] = // [...]
val grouped = input.groupBy(0,1)
```
{% endtab %}
{% endtabs %}

在这里，我们将元组分组到由第一个字段和第二个字段组成的组合键上。

关于嵌套Tuple的注意事项:如果您有一个带有嵌套元组的DataStream，例如:

```text
DataStream<Tuple3<Tuple2<Integer, Float>,String,Long>> ds;
```

指定keyBy\(0\)将导致系统使用完整的Tuple2作为键\(以整数和浮点数作为键\)。如果您想“导航”到嵌套的Tuple2中，您必须使用下面解释的字段表达式键。

### 使用Field Expressions定义键



## 支持的数据类型

Flink对可以在DataSet或DataStream中的元素类型进行了一些限制。原因是系统分析类型以确定有效的执行策略。

有六种不同类别的数据类型：

1. **Java Tuples**和**Scala Case Class**
2. **Java POJO**
3. **原始类型**
4. **常规类型**
5. **Values**
6. **Hadoop Writables**
7. **特殊类型**

### Tuples及Case Class

{% tabs %}
{% tab title="Java" %}
Tuple是包含固定数量的具有各种类型的字段的复合类型。Java API提供`Tuple1`最多到`Tuple25`。Tuple的每个字段都可以是包含更多Tuple的任意Flink类型，从而产生嵌套Tuple。可以使用字段名称直接访问Tuple的字段`tuple.f4`，或使用通用getter方法 `tuple.getField(int position)`。字段索引从0开始。请注意，这与Scala Tuple形成鲜明对比，但它与Java的常规索引更为一致。

```java
DataStream<Tuple2<String, Integer>> wordCounts = env.fromElements(
    new Tuple2<String, Integer>("hello", 1),
    new Tuple2<String, Integer>("world", 2));

wordCounts.map(new MapFunction<Tuple2<String, Integer>, Integer>() {
    @Override
    public Integer map(Tuple2<String, Integer> value) throws Exception {
        return value.f1;
    }
});

wordCounts.keyBy(0); // also valid .keyBy("f0")
```
{% endtab %}

{% tab title="Scala" %}
Scala ****Case Class（和Scala Tuple是Case Class的特例）是包含固定数量的具有各种类型的字段的复合类型。Tuple字段通过其偏移名称来寻址，例如`_1`第一个字段。Case Class字段按名称访问。

```scala
case class WordCount(word: String, count: Int)
val input = env.fromElements(
    WordCount("hello", 1),
    WordCount("world", 2)) // Case Class Data Set

input.keyBy("word")// key by field expression "word"

val input2 = env.fromElements(("hello", 1), ("world", 2)) // Tuple2 Data Set

input2.keyBy(0, 1) // key by field positions 0 and 1
```
{% endtab %}
{% endtabs %}

### POJO

如果满足以下要求，则Flink将Java和Scala类视为特殊的POJO数据类型：

* Class必须public。
* 它必须有一个没有参数的public构造函数（默认构造函数）。
* 所有字段都是共public的，或者必须通过getter和setter函数访问。对于一个名为`foo`，getter和setter方法的字段必须命名`getFoo()`和`setFoo()`。
* Flink必须支持字段的类型。目前，Flink使用[Avro](http://avro.apache.org/)序列化任意对象（例如`Date`）。

Flink分析了POJO类型的结构，即它了解了POJO的字段。因此，POJO类型比一般类型更容易使用。此外，Flink可以比一般类型更有效地处理POJO。

以下示例显示了一个包含两个公共字段的简单POJO。

{% tabs %}
{% tab title="Java" %}
```java
public class WordWithCount {

    public String word;
    public int count;

    public WordWithCount() {}

    public WordWithCount(String word, int count) {
        this.word = word;
        this.count = count;
    }
}

DataStream<WordWithCount> wordCounts = env.fromElements(
    new WordWithCount("hello", 1),
    new WordWithCount("world", 2));

wordCounts.keyBy("word"); // key by field expression "word"
```
{% endtab %}

{% tab title="Scala" %}
```scala
class WordWithCount(var word: String, var count: Int) {
    def this() {
      this(null, -1)
    }
}

val input = env.fromElements(
    new WordWithCount("hello", 1),
    new WordWithCount("world", 2)) // Case Class Data Set

input.keyBy("word")// key by field expression "word"
```
{% endtab %}
{% endtabs %}

### 原始类型

Flink支持所有Java和Scala的原始类型，如`Integer`，`String`和`Double`。

### 常规类型

Flink支持大多数Java和Scala类（API和自定义）。限制适用于包含无法序列化的字段的类，如文件指针，I / O流或其他本机资源。遵循Java Beans约定的类通常可以很好地工作。

所有未标识为POJO类型的类（请参阅上面的POJO要求）都由Flink作为常规类类型处理。Flink将这些数据类型视为黑盒子，并且无法访问其内容（即，用于有效排序）。使用序列化框架[Kryo](https://github.com/EsotericSoftware/kryo)对常规类型进行反序列化。

### Values

值类型手动描述其序列化和反序列化。它们不是通过通用的序列化框架，而是通过实现带有读写方法的`org.apache.flinktypes.value`接口为这些操作提供自定义代码。当通用序列化效率很低时，使用值类型是合理的。举个例子，将元素的稀疏向量实现为一个数组的数值类型。知道数组大部分是零的，就可以对非零元素使用特殊的编码，而通用的序列化只需编写所有数组元素。

`org.apache.flinktypes.copyableValue`接口以类似的方式支持手动内部克隆逻辑。

Flink附带了对应于基本数据类型的预定义值类型。（`byteValue`、`shortValue`、`intValue`、`longValue`、`floatValue`、`doubleValue`、`stringValue`、`charValue`、`booleanValue`）。这些值类型充当基本数据类型的可变变体：它们的值可以更改，允许程序员重用对象并释放垃圾收集器的压力。

### **Hadoop Writables**

可以使用实现该`org.apache.hadoop.Writable`接口的类型。`write()`和`readFields()`方法中定义的序列化逻辑将用于序列化。

### **特殊类型**

可以使用的特殊类型，包括Scala的`Either`，`Option`和`Try`。Java API有自己的自定义实现`Either`。与Scala的`Either`类似，它代表两种可能类型的值，Left或Right。 `Either`可用于错误处理或需要输出两种不同类型记录的运算符。

### **类型擦除和类型推断**

{% hint style="info" %}
_注意：本节仅适用于Java。_
{% endhint %}

Java编译器在编译后抛弃了大部分泛型类型信息。这在Java中称为类型擦除。这意味着在运行时，对象的实例不再知道其泛型类型。例如，`DataStream` 和`DataStream` 的实例与JVM看起来相同。

Flink在准备执行程序时（当调用程序的主要方法时）需要类型信息。 Flink Java API尝试重建以各种方式丢弃的类型信息，并将其显式存储在数据集和运算符中。您可以通过`DataStream.getType()`检索类型。该方法返回`TypeInformation`的一个实例，这是Flink表示类型的内部方式。

类型推断有其局限性，在某些情况下需要程序员的“合作”。这方面的示例是从集合创建数据集的方法，例如`ExecutionEnvironment.fromCollection()`，您可以在其中传递描述类型的参数。但是像MapFunction 这样的通用函数也可能需要额外的类型信息。

`ResultTypeQueryable`接口可以通过输入格式和函数实现，以明确告知API其返回类型。调用函数的输入类型通常可以通过先前操作的结果类型来推断。

## 累加器和计数器

累加器是具有**添加操作**和**最终累积结果的**简单构造，可在作业结束后使用。

最直接的累加器是一个**计数器**：您可以使用该`Accumulator.add(V value)`方法递增它 。在工作结束时，Flink将汇总（合并）所有部分结果并将结果发送给客户。在调试过程中，或者如果您想快速了解有关数据的更多信息，累加器非常有用。

Flink目前有以下**内置累加器**。它们中的每一个都实现了 [Accumulator](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Accumulator.java) 接口。

* [**IntCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/IntCounter.java)， [**LongCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/LongCounter.java) 和[ **DoubleCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/DoubleCounter.java)：请参阅下面的使用计数器的示例。
* [**直方图**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Histogram.java)：离散数量的区间的直方图实现。在内部，它只是一个从`Integer`到`Integer`的映射。您可以使用它来计算值的分布，例如字数统计程序的每行字数的分布。

**如何使用累加器：**

首先，您必须在要使用它的用户定义转换函数中创建累加器对象（此处为计数器）。

```text
private IntCounter numLines = new IntCounter();
```

其次，您必须注册累加器对象，通常在_富_函数的`open()`方法中 。在这里您还可以定义名称。

```text
getRuntimeContext().addAccumulator("num-lines", this.numLines);
```

现在可以在运算符函数中的任何位置使用累加器，包括在`open()`和 `close()`方法中。

```text
this.numLines.add(1);
```

整个结果将存储在`JobExecutionResult`从`execute()`执行环境的方法返回的对象中（当前这仅在执行等待作业完成时才有效）。

```text
myJobExecutionResult.getAccumulatorResult("num-lines")
```

每个作业的所有累加器共享一个命名空间。因此，您可以在作业的不同`Operator`功能中使用相同的累加器。Flink将在内部合并所有具有相同名称的累加器。

关于累加器和迭代的注释：目前累加器的结果仅在整个作业结束后才可用。我们还计划在下一次迭代中使前一次迭代的结果可用。您可以使用 [聚合器](https://github.com/apache/flink/blob/master//flink-java/src/main/java/org/apache/flink/api/java/operators/IterativeDataSet.java#L98) 来计算每次迭代统计信息，并根据此类统计信息确定迭代的终止。

**定制累加器：**

要实现自己的`accumulator`，只需编写`accumulator`接口的实现即可。如果您认为您的自定义累加器应该与Flink一起提供，请自由创建一个pull请求。

您可以选择实现 [Accumulator](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Accumulator.java) 或[SimpleAccumulator](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/SimpleAccumulator.java)。

`Accumulator<V,R>`最灵活：它定义`V`要添加的值的类型`R`，以及最终结果的结果类型。例如，对于直方图，`V`是数字并且`R`是直方图。`SimpleAccumulator`适用于两种类型相同的情况，例如计数器。

