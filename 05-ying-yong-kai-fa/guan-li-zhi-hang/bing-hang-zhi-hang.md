# 并行执行

本节介绍如何在Flink中配置程序的并行执行。Flink程序由多个任务（转换/运算符，数据源和接收器）组成。任务被分成几个并行实例以供执行，每个并行实例处理任务输入数据的子集。任务的并行实例数称为_并行度_。

如果要使用[SavePoint](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html)，还应考虑设置`maximum parallelism`（或`max parallelism`）。从SavePoint恢复时，您可以更改特定运算符或整个程序的并行度，此设置指定并行度的上限。这是必需的，因为在内部将分区状态转换为键组，并且我们不能拥有`+Inf`多个密钥组，因为这会对性能产生不利影响。

## 设置并行度

可以在不同级别的Flink中指定任务的并行度：

### Operator级别

可以通过调用其`setParallelism()`方法来定义单个运算符，数据源或数据接收器的并行性 。例如：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStream<String> text = [...]
DataStream<Tuple2<String, Integer>> wordCounts = text
    .flatMap(new LineSplitter())
    .keyBy(0)
    .timeWindow(Time.seconds(5))
    .sum(1).setParallelism(5);

wordCounts.print();

env.execute("Word Count Example");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment

val text = [...]
val wordCounts = text
    .flatMap{ _.split(" ") map { (_, 1) } }
    .keyBy(0)
    .timeWindow(Time.seconds(5))
    .sum(1).setParallelism(5)
wordCounts.print()

env.execute("Word Count Example")
```
{% endtab %}
{% endtabs %}

### 执行环境级别

如前所述，Flink程序是在执行环境的上下文中执行的。执行环境为它执行的所有操作符、数据源和数据接收器定义了默认的并行性。可以通过显式配置操作符的并行性来覆盖执行环境的并行性。

可以通过调用`setParallelism()`方法来指定执行环境的默认并行性 。要以并行方式执行所有运算符，数据源和数据接收器，请按如下方式设置执行环境的默认并行度：

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setParallelism(3);

DataStream<String> text = [...]
DataStream<Tuple2<String, Integer>> wordCounts = [...]
wordCounts.print();

env.execute("Word Count Example");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
env.setParallelism(3)

val text = [...]
val wordCounts = text
    .flatMap{ _.split(" ") map { (_, 1) } }
    .keyBy(0)
    .timeWindow(Time.seconds(5))
    .sum(1)
wordCounts.print()

env.execute("Word Count Example")
```
{% endtab %}
{% endtabs %}

### 客户端级别

在提交作业到Flink时，可以在客户端设置并行性。客户端既可以是Java，也可以是Scala程序。Flink的命令行接口（command line interface，CLI）就是这样一个客户机的例子。

对于CLI客户端，可以使用指定parallelism参数`-p`。例如：

```text
./bin/flink run -p 10 ../examples/*WordCount-java*.jar
```

在Java/Scala程序中，并行度设置如下：

{% tabs %}
{% tab title="Java" %}
```java
try {
    PackagedProgram program = new PackagedProgram(file, args);
    InetSocketAddress jobManagerAddress = RemoteExecutor.getInetFromHostport("localhost:6123");
    Configuration config = new Configuration();

    Client client = new Client(jobManagerAddress, config, program.getUserCodeClassLoader());

    // set the parallelism to 10 here
    client.run(program, 10, true);

} catch (ProgramInvocationException e) {
    e.printStackTrace();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
try {
    PackagedProgram program = new PackagedProgram(file, args)
    InetSocketAddress jobManagerAddress = RemoteExecutor.getInetFromHostport("localhost:6123")
    Configuration config = new Configuration()

    Client client = new Client(jobManagerAddress, new Configuration(), program.getUserCodeClassLoader())

    // set the parallelism to 10 here
    client.run(program, 10, true)

} catch {
    case e: Exception => e.printStackTrace
}
```
{% endtab %}
{% endtabs %}

### 系统级别

可以通过设置`parallelism.default`属性来定义所有执行环境的系统范围默认并行度 `./conf/flink-conf.yaml`。有关详细信息，请参阅 [配置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html)文档

## 设置最大并行度

可以在设置并行度的地方设置最大并行度\(客户端级别和系统级别除外\)。不是调用`setParallelism()`，而是调用`setMaxParallelism()`来设置最大并行度。

最大并行度的默认设置大致为`operatorparallelism+(operatorparallelism/2)`，下限为127，上限为32768。

{% hint style="danger" %}
注意，将最大并行性设置为非常大的值可能会对性能造成不利影响，因为某些状态后端必须保持与键组数量（这是可重新调整状态的内部实现机制）成比例的内部数据结构。
{% endhint %}

