# 执行配置

StreamExecutionEnvironment包含ExecutionConfig，它允许为运行时设置特定于作业的配置值。要更改影响所有作业的默认值，请参见[Configuration](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html)。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
ExecutionConfig executionConfig = env.getConfig();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment
var executionConfig = env.getConfig
```
{% endtab %}
{% endtabs %}

可以使用以下配置选项:\(默认为粗体）

* **`enableClosureCleaner()`**/ `disableClosureCleaner()`。默认情况下启用闭包清理器。闭包清理器删除Flink程序中对周围类匿名函数的不需要的引用。禁用闭包清除器后，可能会发生匿名用户函数引用周围的类，通常不是Serializable。这将导致序列化器出现异常。
* `getParallelism()`/ `setParallelism(int parallelism)`设置作业的默认并行度。
* `getMaxParallelism()`/ `setMaxParallelism(int parallelism)`设置作业的默认最大并行度。此设置确定最大并行度并指定动态缩放的上限。
* `getNumberOfExecutionRetries()`/ `setNumberOfExecutionRetries(int numberOfExecutionRetries)`设置重新执行失败任务的次数。值为零可有效禁用容错。值为`-1`表示应使用系统默认值（在配置中定义）。这已弃用，请改用[重启策略](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/restart_strategies.html)。
* `getExecutionRetryDelay()`/ `setExecutionRetryDelay(long executionRetryDelay)`将系统在作业失败后等待的延迟设置为毫秒，然后重新执行该作业。在任务管理器上成功停止所有任务之后，延迟开始，一旦延迟过去，任务将重新启动。此参数用于延迟重新执行，以便在尝试重新执行之前充分显示与超时相关的故障\(如未完全超时的断开连接\)，并立即由于相同的问题再次失败。此参数仅在执行重试次数为一个或多个时才有效果。这已弃用，请改用[重启策略](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/restart_strategies.html)。
* `getExecutionMode()`/ `setExecutionMode()`。默认执行模式为PIPELINED。设置执行模式以执行程序。执行模式定义数据交换是以批处理还是以流的方式执行。
* `enableForceKryo()`/ **`disableForceKryo`**。Kryo在默认情况下不是强制的。强制`GenericTypeInformation`为`POJO`使用`Kryo`序列化器，尽管我们可以将它们作为`POJO`进行分析。在某些情况下，这可能更可取。例如，当Flink的内部序列化器不能正确处理`POJO`时。
* `enableForceAvro()`/ **`disableForceAvro()`**。默认情况下，Avro不是强制的。强制Flink `AvroTypeInformation`使用`Avro`序列化器而不是`Kryo`来序列化`Avro pojo`。
* `enableObjectReuse()`/ **`disableObjectReuse()`**默认情况下，对象不会在Flink中重复使用。启用对象重用模式将指示运行时重用用户对象以获得更好的性能。请记住，当操作的用户代码功能不知道此行为时，这可能会导致错误。
* **`enableSysoutLogging()`**/ `disableSysoutLogging()`JobManager状态更新被打印到`System.out`中。默认情况下。此设置允许禁用此行为。
* `getGlobalJobParameters()`/ `setGlobalJobParameters()`此方法允许用户将自定义对象设置为作业的全局配置。由于`ExecutionConfig`在所有用户定义的函数中都可以访问，因此这是一种在作业中全局提供配置的简单方法。
* `addDefaultKryoSerializer(Class<?> type, Serializer<?> serializer)`注册给定类型的Kryo序列化器实例。
* `addDefaultKryoSerializer(Class<?> type, Class<? extends Serializer<?>> serializerClass)`注册给定类型的Kryo序列化器类。
* `registerTypeWithKryoSerializer(Class<?> type, Serializer<?> serializer)`使用Kryo注册给定类型并为其指定序列化程序。通过使用Kryo注册类型，类型的序列化将更加有效。
* `registerKryoType(Class<?> type)`如果该类型最终被Kryo序列化，那么它将在Kryo中注册，以确保只编写标记\(整数id\)。如果一个类型没有在Kryo中注册，那么它的整个类名将在每个实例中序列化，从而导致更高的I/O成本。
* `registerPojoType(Class<?> type)`用序列化堆栈注册给定的类型。如果该类型最终被序列化为POJO，那么该类型将被注册到POJO序列化器中。如果该类型最终被Kryo序列化，那么它将在Kryo中注册，以确保只编写标记。如果一个类型没有在Kryo中注册，那么它的整个类名将在每个实例中序列化，从而导致更高的I/O成本。

注意，在registerKryoType\(\)中注册的类型对Flink的Kryo序列化器实例不可用。

* `disableAutoTypeRegistration()`默认情况下启用自动类型注册。自动类型注册是注册用户代码与Kryo和POJO序列化器使用的所有类型（包括子类型）。
* `setTaskCancellationInterval(long interval)`设置在连续尝试取消正在运行的任务之间等待的间隔\(以毫秒为单位\)。当一个任务被取消时，如果任务线程在一定的时间内没有终止，则创建一个新线程，该线程定期调用任务线程上的`interrupt()`。该参数指的是对`interrupt()`的连续调用之间的时间，默认设置为**30000**毫秒，即**30**秒。

`RuntimeContext`在`Rich*`函数中可以通过`getRuntimeContext()`方法访问，它还允许在所有用户定义的函数中访问`ExecutionConfig`。

