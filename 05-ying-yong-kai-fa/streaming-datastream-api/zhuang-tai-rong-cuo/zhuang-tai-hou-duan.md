# 状态后端

Flink提供了不同的状态后端，用于指定状态的存储方式和位置

状态可以位于Java的堆上或堆外。根据您的状态后端，Flink还可以管理应用程序的状态，这意味着Flink处理内存管理\(如果需要，可能会溢出到磁盘\)，以允许应用程序保存非常大的状态。默认情况下，配置文件为`flink-conf.yaml`确定所有Flink作业的状态后端。

但是，可以在每个作业的基础上覆盖默认状态后端，如下所示。

有关可用状态后端，其优点，限制和配置参数的详细信息，请参阅[部署和操作中](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/state_backends.html)的相应部分。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(...);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
env.setStateBackend(...)
```
{% endtab %}
{% endtabs %}

