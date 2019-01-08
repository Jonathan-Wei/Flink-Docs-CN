# 概述

## DataSet 和 DataStream

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

## 指定密钥

## 指定转换函数

## 支持的数据类型

## 累加器和计数器

累加器是具有**添加操作**和**最终累积结果的**简单构造，可在作业结束后使用。

最直接的累加器是一个**计数器**：您可以使用该`Accumulator.add(V value)`方法递增它 。在工作结束时，Flink将汇总（合并）所有部分结果并将结果发送给客户。在调试过程中，或者如果您想快速了解有关数据的更多信息，累加器非常有用。

Flink目前有以下**内置累加器**。它们中的每一个都实现了 [Accumulator](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Accumulator.java) 接口。

* [**IntCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/IntCounter.java)， [**LongCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/LongCounter.java) 和[ **DoubleCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/DoubleCounter.java)：请参阅下面的使用计数器的示例。
* [**直方图**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Histogram.java)：离散数量的区间的直方图实现。在内部，它只是一个从Integer到Integer的映射。您可以使用它来计算值的分布，例如字数统计程序的每行字数的分布。

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

每个作业的所有累加器共享一个命名空间。因此，您可以在作业的不同Operator功能中使用相同的累加器。Flink将在内部合并所有具有相同名称的累加器。

关于累加器和迭代的注释：目前累加器的结果仅在整个作业结束后才可用。我们还计划在下一次迭代中使前一次迭代的结果可用。您可以使用 [聚合器](https://github.com/apache/flink/blob/master//flink-java/src/main/java/org/apache/flink/api/java/operators/IterativeDataSet.java#L98) 来计算每次迭代统计信息，并根据此类统计信息确定迭代的终止。

**定制累加器：**

要实现自己的accumulator，只需编写accumulator接口的实现即可。如果您认为您的自定义累加器应该与Flink一起提供，请自由创建一个pull请求。

`Accumulator<V,R>`最灵活：它定义`V`要添加的值的类型`R`，以及最终结果的结果类型。例如，对于直方图，`V`是数字并且`R`是直方图。`SimpleAccumulator`适用于两种类型相同的情况，例如计数器。

