# 重启策略

Flink支持不同的重新启动策略，这些策略控制在发生故障时如何重新启动作业。可以使用默认的重新启动策略启动集群，在未定义作业特定的重新启动策略时，始终使用该策略。如果使用重新启动策略提交作业，则此策略将覆盖群集的默认设置。

## 概述

默认重启策略是通过flink的配置文件`flink-conf.yaml`设置的。配置参数重新启动策略定义采用的策略。如果未启用检查点，则使用“不重启”策略。如果激活了检查点并且没有配置重新启动策略，则固定延迟策略与`Integer.max_value`重新启动尝试一起使用。请参阅以下可用重新启动策略列表，了解支持哪些值。

每个重启策略都有自己的一组参数，这些参数控制其行为。这些值也在配置文件中设置。每个重启策略的描述包含有关各自配置值的更多信息。

| 重启策略 | 重启策略值 |
| :--- | :--- |
| 固定延迟 | fixed-delay |
| 失败率 | failure-rate |
| 不重启 | none |

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

## 重启策略

以下部分介绍重新启动策略特定的配置选项。

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

```text
restart-strategy: failure-rate
```

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

