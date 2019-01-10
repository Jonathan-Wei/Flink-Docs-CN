# 容错

Flink的容错机制在出现故障时恢复程序并继续执行它们。此类故障包括机器硬件故障，网络故障，瞬态程序故障等。

## 批处理容错

DataSet API中程序的容错性是通过重试失败的执行来实现的。Flink在宣告作业失败之前重试执行的次数可以通过执行重试参数进行配置。如果值为0，则意味着故障容忍度失效。

若要激活容错，请将执行重试设置为大于零的值。一个常见的选择是3。

这个例子展示了如何配置Flink数据集程序的执行重试。

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setNumberOfExecutionRetries(3);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setNumberOfExecutionRetries(3)
```
{% endtab %}
{% endtabs %}

还可以在`flink-conf.yaml`中定义执行重试次数和延迟重试的默认值:

```text
execution-retries.default: 3
```

## 延迟重试

可以将执行重试配置为延迟。延迟重试意味着在执行失败之后，重新执行不会立即启动，而是在一定的延迟之后。

当程序与外部系统交互时\(例如连接或挂起的事务在尝试重新执行之前应该达到超时\)，延迟重试是有帮助的。

您可以设置每个程序的延迟重试如下\(示例展示的是DataStream API— DataSet API的工作原理类似\):

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().setExecutionRetryDelay(5000); // 5000 milliseconds delay
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.getConfig.setExecutionRetryDelay(5000) // 5000 milliseconds delay
```
{% endtab %}
{% endtabs %}

还可以在`flink-conf.yaml`中定义重试延迟的默认值:

```text
execution-retries.delay: 10 s
```

