# 任务失败恢复

当任务失败时，Flink需要重新启动失败的任务和其他受影响的任务，以将作业恢复到正常状态。

重新启动策略和故障转移策略用于控制任务重新启动。重新启动策略决定是否以及何时可以重新启动失败/受影响的任务。故障转移策略决定应重新启动哪些任务以恢复作业。

## 重新启动策略

可以使用默认的重启策略来启动集群，该默认重启策略通常在未定义作业特定的重启策略时使用。如果使用重新启动策略提交作业，则此策略将覆盖集群的默认设置。

默认重启策略是通过flink的配置文件`flink-conf.yaml`设置的。配置参数重新启动策略定义采用的策略。如果未启用检查点，则使用“不重启”策略。如果激活了检查点并且没有配置重新启动策略，则固定延迟策略与`Integer.max_value`重新启动尝试一起使用。请参阅以下可用重新启动策略列表，了解支持哪些值。

每个重启策略都有自己的一组参数，这些参数控制其行为。这些值也在配置文件中设置。每个重启策略的描述包含有关各自配置值的更多信息。



| Key | Default | Type | Description |
| :--- | :--- | :--- | :--- |


<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>restart-strategy</b>
      </th>
      <th style="text-align:left">(none)</th>
      <th style="text-align:left">String</th>
      <th style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x5728;&#x4F5C;&#x4E1A;&#x5931;&#x8D25;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x4F7F;&#x7528;&#x7684;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x7B56;&#x7565;&#x3002;
          <br
          />&#x53EF;&#x63A5;&#x53D7;&#x7684;&#x503C;&#x4E3A;&#xFF1A;</p>
        <ul>
          <li><code>none</code>&#xFF0C;<code>off</code>&#xFF0C;<code>disable</code>&#xFF1A;&#x6CA1;&#x6709;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x7B56;&#x7565;&#x3002;</li>
          <li><code>fixeddelay</code>&#xFF0C;<code>fixed-delay</code>&#xFF1A;&#x56FA;&#x5B9A;&#x5EF6;&#x8FDF;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x7B56;&#x7565;&#x3002;&#x53EF;&#x4EE5;&#x5728;
            <a
            href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#fixed-delay-restart-strategy">&#x6B64;&#x5904;</a>&#x627E;&#x5230;&#x66F4;&#x591A;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#x3002;</li>
          <li><code>failurerate</code>&#xFF0C;<code>failure-rate</code>&#xFF1A;&#x5931;&#x8D25;&#x7387;&#x91CD;&#x542F;&#x7B56;&#x7565;&#x3002;&#x53EF;&#x4EE5;&#x5728;
            <a
            href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#failure-rate-restart-strategy">&#x6B64;&#x5904;</a>&#x627E;&#x5230;&#x66F4;&#x591A;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#x3002;</li>
        </ul>
        <p>&#x5982;&#x679C;&#x7981;&#x7528;&#x68C0;&#x67E5;&#x70B9;&#xFF0C;&#x5219;&#x9ED8;&#x8BA4;&#x503C;&#x4E3A;<code>none</code>&#x3002;&#x5982;&#x679C;&#x68C0;&#x67E5;&#x70B9;&#x5DF2;&#x542F;&#x7528;&#xFF0C;&#x9ED8;&#x8BA4;&#x503C;&#x662F;<code>fixed-delay</code>&#x4E0E;<code>Integer.MAX_VALUE</code>&#x91CD;&#x542F;&#x5C1D;&#x8BD5;&#x548C;&#x201C; <code>1 s</code>&#x201D;&#x5EF6;&#x8FDF;&#x3002;</p>
      </th>
    </tr>
  </thead>
  <tbody></tbody>
</table>

除了定义默认的重启策略之外，还可以为每个Flink作业定义特定的重启策略。通过在`ExecutionEnvironment`上调用`setRestartStrategy`方法以编程方式设置此重新启动策略。请注意，这也适用于`StreamExecutionEnvironment`。

以下示例展示了我们如何为我们的工作设置固定延迟重启策略。如果发生故障，系统会尝试重新启动作业3次，并在连续重启尝试之间等待10秒。

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // number of restart attempts
  Time.of(10, TimeUnit.SECONDS) // delay
));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // number of restart attempts
  Time.of(10, TimeUnit.SECONDS) // delay
))
```
{% endtab %}
{% endtabs %}

### 固定延迟重启策略

固定延迟重启策略尝试给定次数重新启动作业。如果超过最大尝试次数，则作业最终会失败。在两次连续重启尝试之间，重启策略等待一段固定的时间。

通过在`flink-conf.yaml`中设置以下配置参数，可以默认启用此策略。

```text
restart-strategy: fixed-delay
```

| 配置参数 | 描述 | 默认值 |
| :--- | :--- | :--- |
| restart-strategy.fixed-delay.attempts | 声明Flink在作业失败之前重试执行的次数。 | 1，或者`Integer.MAX_VALUE`\(通过检查点激活\) |
| restart-strategy.fixed-delay.delay | 延迟重试意味着在执行失败之后，重新执行不会立即启动，而是在一定的延迟之后。当程序与外部系统交互时\(例如连接或挂起的事务在尝试重新执行之前应该达到超时\)，延迟重试是有帮助的。 | `akka.ask.timeout`，如果通过检查点激活，则为10秒 |

例如：

```text
restart-strategy.fixed-delay.attempts: 3
restart-strategy.fixed-delay.delay: 10 s
```

固定延迟重启策略也可以通过编程方式设置：

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // number of restart attempts
  Time.of(10, TimeUnit.SECONDS) // delay
));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.fixedDelayRestart(
  3, // number of restart attempts
  Time.of(10, TimeUnit.SECONDS) // delay
))
```
{% endtab %}
{% endtabs %}

### 故障率重启策略

故障率重启策略在故障后重新启动作业，但是当`failure rate`超过（每个时间间隔的故障）时，作业最终会失败。在两次连续重启尝试之间，重启策略等待一段固定的时间。

通过在`flink-conf.yaml`中设置以下配置参数，可以默认启用此策略。

{% code title="" %}
```text
restart-strategy: failure-rate
```
{% endcode %}

| 配置参数 | 描述 | 默认值 |
| :--- | :--- | :--- |
| restart-strategy.failure-rate.max-failures-per-interval | 失败作业之前给定时间间隔内的最大重启次数 | 1 |
| restart-strategy.failure-rate.failure-rate-interval | 测量故障率的时间间隔。 | 1分钟 |
| restart-strategy.failure-rate.delay | 两次连续重启尝试之间的延迟 | akka.ask.timeout |

```text
restart-strategy.failure-rate.max-failures-per-interval: 3
restart-strategy.failure-rate.failure-rate-interval: 5 min
restart-strategy.failure-rate.delay: 10 s
```

故障率重启策略也可以通过编程方式设置：

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.failureRateRestart(
  3, // max failures per interval
  Time.of(5, TimeUnit.MINUTES), //time interval for measuring failure rate
  Time.of(10, TimeUnit.SECONDS) // delay
));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.failureRateRestart(
  3, // max failures per unit
  Time.of(5, TimeUnit.MINUTES), //time interval for measuring failure rate
  Time.of(10, TimeUnit.SECONDS) // delay
))
```
{% endtab %}
{% endtabs %}

### 无重启战略

作业直接失败，不尝试重启。

```text
restart-strategy: none
```

也可以通过编程方式设置`no restart`策略：

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.setRestartStrategy(RestartStrategies.noRestart());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment()
env.setRestartStrategy(RestartStrategies.noRestart())
```
{% endtab %}
{% endtabs %}

### 后备重启策略

使用群集定义的重新启动策略。这对于启用检查点的流式传输程序很有帮助。默认情况下，如果没有定义其他重启策略，则选择固定延迟重启策略。

## 故障转移策略

 Flink支持不同的故障转移策略，可以通过Flink的配置文件`flink-conf.yaml`中配置参数_`jobmanager.execution.failover-strategy`对其_进行配置。

| 故障转移策略 | Value for jobmanager.execution.failover-strategy |
| :--- | :--- |
| 重启所有 | full |
| 重启管道区域 | region |

### 重启所有故障转移策略

此策略重新启动作业中的所有任务以从任务失败中恢复。

### 重新启动流水线区域故障转移策略

此策略将任务分为不相交的区域。当检测到任务故障时，此策略将计算必须重新启动以从故障中恢复的最小区域集。与“重新启动所有故障转移策略”相比，对于某些作业，这可能导致要重新启动的任务更少。

区域是一组通过流水线数据交换进行通信的任务。也就是说，批处理数据交换表示区域的边界。

* DataStream作业或流表/SQL作业中的所有数据交换都是流水线的。
* 默认情况下，批处理表/SQL作业中的所有数据交换都是批处理的。
* 数据集作业中的数据交换类型由ExecutionMode决定，可以通过ExecutionConfig设置该模式

重启区域如下:

* 包含失败任务的区域将重新启动。
* 如果结果分区不可用，而某个将要重新启动的区域又需要它，那么生成结果分区的区域也将重新启动。
* 如果要重新启动某个区域，则该区域的所有消费区域也将重新启动。这是为了保证数据一致性，因为不确定的处理或分区可能导致不同的分区。

