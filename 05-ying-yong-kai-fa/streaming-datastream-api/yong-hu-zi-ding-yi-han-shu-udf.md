# 用户自定义函数（UDF）

大多数操作都需要用户定义的功能。本节列出了如何指定它们的不同方法。我们还将介绍`Accumulators`，可用于深入了解您的Flink应用程序。

{% tabs %}
{% tab title="Java" %}
## 实现借口

最基本的方法是实现提供的接口之一：

```java
class MyMapFunction implements MapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
data.map(new MyMapFunction());
```

## 匿名类

可以将函数作为匿名类传递：

```java
data.map(new MapFunction<String, Integer> () {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```

## Java 8 Lambdas

Flink还支持Java API中的Java 8 Lambda。

```java
data.filter(s -> s.startsWith("http://"));
```

```java
data.reduce((i1,i2) -> i1 + i2);
```

## Rich functions

所有需要用户定义函数的转换都可以将_Rich_函数作为参数。例如

```java
class MyMapFunction implements MapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
```

你可以写

```java
class MyMapFunction extends RichMapFunction<String, Integer> {
  public Integer map(String value) { return Integer.parseInt(value); }
};
```

并将函数照常传递给`map`转换：

```java
data.map(new MyMapFunction());
```

Rich函数也可以定义为匿名类：

```java
data.map (new RichMapFunction<String, Integer>() {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```
{% endtab %}

{% tab title="Scala" %}
## Lambda函数

如前面的示例中已经看到的，所有操作都接受lambda函数来描述该操作：

```scala
val data: DataSet[String] = // [...]
data.filter { _.startsWith("http://") }
```

```scala
val data: DataSet[Int] = // [...]
data.reduce { (i1,i2) => i1 + i2 }
// or
data.reduce { _ + _ }
```

## Rich Function

所有将lambda函数作为参数的转换都可以将_富_函数作为参数。例如

```scala
data.map { x => x.toInt }
```

你可以写

```scala
class MyMapFunction extends RichMapFunction[String, Int] {
  def map(in: String):Int = { in.toInt }
};
```

并将函数传递给`map`转换：

```scala
data.map(new MyMapFunction())
```

Rich Function也可以定义为匿名类：

```scala
data.map (new RichMapFunction[String, Int] {
  def map(in: String):Int = { in.toInt }
})
```
{% endtab %}
{% endtabs %}

除了用户定义的函数\(`map`、`reduce`等\)之外，Rich Function还提供了四种方法:`open`、`close`、`getRuntimeContext`和`setRuntimeContext`。对于参数化函数\(请参阅将[参数传递给函数](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/#passing-parameters-to-functions)\)、创建和结束本地状态、访问广播变量\(请参阅 [广播变量](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/#broadcast-variables)\)、访问运行时信息\(如累加器和计数器\(请参阅[累加器和计数器](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/user_defined_functions.html#accumulators--counters)\)以及关于Iterations的信息\(请参阅 [Iterations](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/batch/iterations.html)\)，这些都很有用。

## 累加器和计数器

累加器是具有**加法运算**和**最终累加结果的**简单结构，可在作业结束后使用。

最简单的累加器是一个**计数器**：你可以使用`Accumulator.add(V value)`方法将其递增 。在任务结束时，Flink将汇总（合并）所有部分结果，并将结果发送给客户端。累加器在调试过程中或如果您想快速查找有关数据的更多信息时非常有用。

Flink当前具有以下**内置累加器**。它们每个都实现 [累加器](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Accumulator.java) 接口。

* [**IntCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/IntCounter.java)， [**LongCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/LongCounter.java) 和[**DoubleCounter**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/DoubleCounter.java)：有关使用计数器的示例，请参见下文。
* [**直方图**](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/accumulators/Histogram.java)：离散数量的bin的直方图实现。在内部，它只是从Integer到Integer的映射。您可以使用它来计算值的分布，例如，单词计数程序的每行单词的分布。

**如何使用累加器：**

首先，您必须在要使用它的用户定义的转换函数中创建一个累加器对象（此处是一个计数器）。

```text
private IntCounter numLines = new IntCounter();
```

其次，您必须通常在**RichFunc**法中 注册累加器对象。您还可以在此处定义名称。  
  


