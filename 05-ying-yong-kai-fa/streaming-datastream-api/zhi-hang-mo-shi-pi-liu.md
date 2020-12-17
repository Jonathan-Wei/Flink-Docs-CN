# 执行模式\(批/流\)

DataStream API支持不同的运行时执行模式，您可以根据用例的要求和工作特征从中选择运行模式。



## 什么时候可以/应该使用BATCH执行模式？

## 配置BATCH执行模式

可以通过`execution.runtime-mode`配置设置执行模式。有以下三种模式：

* `STREAMING`：经典的DataStream执行模式（默认）
* `BATCH`：在DataStream API上以批处理方式执行
* `AUTOMATIC`：让系统根据来源的边界来决定

可以通过的命令行参数进行配置`bin/flink run ...`，也可以在创建/配置时以编程方式进行配置`StreamExecutionEnvironment`。

通过命令行配置执行模式的方法如下：

```text
$ bin/flink run -Dexecution.runtime-mode=BATCH examples/streaming/WordCount.jar
```

本示例说明如何在代码中配置执行模式：

```text
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```

{% hint style="info" %}
**注意：** 建议用户不要在程序中设置运行时模式，而应在提交应用程序时使用命令行来设置运行时模式。保持应用程序代码的自由配置可提供更大的灵活性，因为可以在任何执行模式下执行同一应用程序。
{% endhint %}

## 执行行为

本节概述了`BATCH` 执行模式的执行行为，并将其与`STREAMING`执行模式进行了对比。有关更多详细信息，请参阅引入此功能的[FLIP](https://cwiki.apache.org/confluence/x/4i94CQ)： [FLIP-134](https://cwiki.apache.org/confluence/x/4i94CQ)和 [FLIP-140](https://cwiki.apache.org/confluence/x/kDh4CQ)。

### 任务调度和网络Shuffle

Flink作业由在数据流图中连接在一起的不同算子组成。系统决定如何安排在不同进程/机器（TaskManager）上执行这些操作的方式，以及如何在它们之间对数据进行随机（发送）。



另一方面，诸如`keyBy()`或`rebalance()`之类的操作要求在任务的不同并行实例之间对数据进行混洗。这会引起网络混乱。

我们将使用此示例来说明任务调度和网络传输中的差异：

```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

DataStreamSource<String> source = env.fromElements(...);

source.name("source")
	.map(...).name("map1")
	.map(...).name("map2")
	.rebalance()
	.map(...).name("map3")
	.map(...).name("map4")
	.keyBy((value) -> value)
	.map(...).name("map5")
	.map(...).name("map6")
	.sinkTo(...).name("sink");
```



对于上面的示例，Flink将操作分组为如下任务：

* 任务1： `source`，`map1`和`map2`
* 任务2 ：`map3`、`map4`
* 任务3： `map5`，`map6`和`sink`

我们在任务1和2之间以及任务2和3之间进行了网络改组。这是该工作的直观表示：

![](../../.gitbook/assets/datastream-example-job-graph.svg)

### 状态后端/状态

### EventTime/Watermarks

### 处理事件

### 故障恢复

## 重要注意事项

### 检查点

### 广播状态

### 编写自定义运算符

{% hint style="info" %}
**注意：** 自定义运算符是Apache Flink的高级用法模式。对于大多数用例，请考虑改用（keyed）处理函数。
{% endhint %}

