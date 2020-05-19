# 本地执行

即使在单个Java虚拟机中，Flink也可以在一台机器上运行。这允许用户在本地测试和调试Flink程序。本节概述了本地执行机制。

本地环境和执行程序允许您在本地Java虚拟机中运行Flink程序，或在任何JVM中作为现有程序的一部分运行。只需点击IDE的“运行”按钮，即可在本地启动大多数示例。

Flink支持两种不同的本地执行。`LocalExecutionEnvironment`运行时将启动完整的Flink，包括`JobManager`和`TaskManager`。这些包括内存管理和在集群模式下执行的所有内部算法。

`CollectionEnvironment`正在Java集合上执行Flink程序。这种模式不会启动完整的Flink运行时，因此执行的开销非常低，而且轻量级。因此执行的开销和轻量级都非常低。例如，`DataSet.map()`通过将`map()`函数应用于Java列表中的所有元素来执行转换。

## 调试

如果您在本地运行Flink程序，您也可以像调试任何其他Java程序一样调试程序。您可以使用`System.out.println()`写出一些内部变量，也可以使用调试器。可以在其中设置断点`map()`，`reduce()`以及所有其他方法。另请参阅Java API文档中的[调试部分](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/index.html#debugging)，以获取Java API中的测试和本地调试实用程序指南。

## Maven依赖

如果使用Maven做项目程序开发，则必须`flink-clients`使用此依赖项添加模块：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-clients_2.11</artifactId>
  <version>1.10.0</version>
</dependency>
```

## 本地环境

`LocalEnvironment`是Flink程序的本地执行句柄。使用它在本地JVM中运行一个程序——独立的或嵌入到其他程序中。

本地环境通过`ExecutionEnvironment.createLocalEnvironment()`方法实例化。默认情况下，它将使用与您的机器的CPU内核\(硬件上下文\)一样多的本地线程来执行。您可以选择指定所需的并行性。可以使用`enableLogging()`/`disableLogging()`将本地环境配置为登录控制台。

在大多数情况下，调用`ExecutionEnvironment.getExecutionEnvironment()`是更好的方法。当程序在本地启动\(在命令行界面之外\)时，该方法返回`LocalEnvironment`，当命令行界面调用程序时，该方法返回预配置的集群执行环境。

```java
public static void main(String[] args) throws Exception {
    ExecutionEnvironment env = ExecutionEnvironment.createLocalEnvironment();

    DataSet<String> data = env.readTextFile("file:///path/to/file");

    data
        .filter(new FilterFunction<String>() {
            public boolean filter(String value) {
                return value.startsWith("http://");
            }
        })
        .writeAsText("file:///path/to/result");

    JobExecutionResult res = env.execute();
}
```

执行完成后返回的`JobExecutionResult`对象，包含程序运行时间和累加器结果。

`LocalEnvironment`还允许向Flink传递自定义配置值。

```java
Configuration conf = new Configuration();
conf.setFloat(ConfigConstants.TASK_MANAGER_MEMORY_FRACTION_KEY, 0.5f);
final ExecutionEnvironment env = ExecutionEnvironment.createLocalEnvironment(conf);
```

{% hint style="info" %}
_注意：_本地执行环境不会启动任何Web前端来监视执行。
{% endhint %}

## 集合环境

使用CollectionEnvironment在Java集合上执行是一种执行Flink程序的低开销方法。这种模式的典型用例是自动化测试、调试和代码重用。

对于更具交互性的情况，用户也可以使用为批处理实现的算法。在Java应用服务器中，可以使用稍微更改过的Flink程序变体来处理传入的请求。

**基于集合执行的框架**

```java
public static void main(String[] args) throws Exception {
    // initialize a new Collection-based execution environment
    final ExecutionEnvironment env = new CollectionEnvironment();

    DataSet<User> users = env.fromCollection( /* get elements from a Java Collection */);

    /* Data Set transformations ... */

    // retrieve the resulting Tuple2 elements into a ArrayList.
    Collection<...> result = new ArrayList<...>();
    resultDataSet.output(new LocalCollectionOutputFormat<...>(result));

    // kick off execution.
    env.execute();

    // Do some work with the resulting ArrayList (=Collection).
    for(... t : result) {
        System.err.println("Result = "+t);
    }
}
```

该`flink-examples-batch`模块包含一个完整的示例，称为`CollectionExecutionExample`。

{% hint style="info" %}
注意：基于集合的Flink程序的执行仅适用于适合JVM堆的小数据。集合上的执行不是多线程的，只使用一个线程。
{% endhint %}

