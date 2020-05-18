# 检查点\(Checkpointing\)

Flink中的每个函数和操作符都可以是**有状态**的\(有关详细信息，请参见[使用状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html)\)。有状态函数在单个元素/事件的处理过程中存储数据，使状态成为任何类型的更精细操作的关键构建块。

为了使状态容错，Flink需要对状态进行检查点。检查点允许Flink恢复流中的状态和位置，从而为应用程序提供与无故障执行相同的语义。

[流容错文档](https://ci.apache.org/projects/flink/flink-docs-master/internals/stream_checkpointing.html)详细描述了Flink流容错机制背后的技术。

## 先决条件

Flink的检查点机制与流和状态的持久存储交互。一般而言，它要求:

* 可以在一定时间内重播记录的持久\(或持久\)数据源。此类源的示例有持久消息队列\(例如Apache Kafka、RabbitMQ、Amazon Kinesis、谷歌PubSub\)或文件系统\(例如HDFS、S3、GFS、NFS、Ceph，……\)。 
* 状态的持久存储，通常是分布式文件系统\(例如，HDFS, S3, GFS, NFS, Ceph，…\)

## 启用和配置检查点

默认情况下，禁用检查点。为了使检查点，调用`enableCheckpointing(n)`上`StreamExecutionEnvironment`，其中n是以毫秒为单位的检查点间隔。

检查点的其他参数包括：

* _**完全一次与至少一次**_：您可以选择将模式传递给`enableCheckpointing(n)`方法，以在两个保证级别之间进行选择。对于大多数应用来说，恰好一次是优选的。至少一次可能与某些超低延迟（始终为几毫秒）的应用程序相关。
* _**Checkpoint timeout（检查点超时）**_：如果在指定时间之后仍未完成，则该检查点将会被中止。
* **检查点之间的最小时间**:为了确保流应用程序在检查点之间取得一定的进展，可以定义在检查点之间需要通过的时间。例如，如果将此值设置为5000，则无论检查点持续时间和检查点间隔如何，下一个检查点都将在上一个检查点完成后的5秒内启动。注意，这意味着检查点间隔永远不会小于此参数。

  通过定义“检查点之间的时间”而不是检查点间隔来配置应用程序通常更容易，因为“检查点之间的时间”不易受检查点有时需要比平均时间更长的事实的影响（例如，如果目标存储系统暂时很慢）。

  注意，这个值还意味着并发检查点的数量是1。

* **并发检查点的数量**:默认情况下，当一个检查点仍在运行时，系统不会触发另一个检查点。这确保拓扑不会在检查点上花费太多时间，也不会在处理流方面取得进展。可以允许多个重叠的检查站,这很有趣的管道有一定处理延迟\(例如,因为函数调用外部服务,需要一些时间来回应\),但仍然想做的非常频繁的检查点\(100毫秒\)处理文档很少失败。

  当定义检查点之间的最小时间时，不能使用此选项。

* **外部化检查点**:可以将定期检查点配置为在外部持久化。外部化检查点将它们的元数据写到持久存储中，并且在作业失败时不会自动清理。这样，如果你的工作失败了，你就有一个检查点可以继续。关于外部化检查点的部署说明中有更多详细信息。
* _**关于检查点错误的失败/继续任务**_：这确定如果在执行任务的检查点过程中发生错误，任务是否将失败。这是默认行为。或者，当禁用此选项时，任务将简单地拒绝检查点协调器的检查点并继续运行。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

// start a checkpoint every 1000 ms
env.enableCheckpointing(1000);

// advanced options:

// set mode to exactly-once (this is the default)
env.getCheckpointConfig().setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE);

// make sure 500 ms of progress happen between checkpoints
env.getCheckpointConfig().setMinPauseBetweenCheckpoints(500);

// checkpoints have to complete within one minute, or are discarded
env.getCheckpointConfig().setCheckpointTimeout(60000);

// allow only one checkpoint to be in progress at the same time
env.getCheckpointConfig().setMaxConcurrentCheckpoints(1);

// enable externalized checkpoints which are retained after job cancellation
env.getCheckpointConfig().enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()

// start a checkpoint every 1000 ms
env.enableCheckpointing(1000)

// advanced options:

// set mode to exactly-once (this is the default)
env.getCheckpointConfig.setCheckpointingMode(CheckpointingMode.EXACTLY_ONCE)

// make sure 500 ms of progress happen between checkpoints
env.getCheckpointConfig.setMinPauseBetweenCheckpoints(500)

// checkpoints have to complete within one minute, or are discarded
env.getCheckpointConfig.setCheckpointTimeout(60000)

// prevent the tasks from failing if an error happens in their checkpointing, the checkpoint will just be declined.
env.getCheckpointConfig.setFailTasksOnCheckpointingErrors(false)

// allow only one checkpoint to be in progress at the same time
env.getCheckpointConfig.setMaxConcurrentCheckpoints(1)
```
{% endtab %}

{% tab title="Python" %}
```python
env = StreamExecutionEnvironment.get_execution_environment()

# 每 1000ms 开始一次 checkpoint
env.enable_checkpointing(1000)

# 高级选项：

# 设置模式为精确一次 (这是默认值)
env.get_checkpoint_config().set_checkpointing_mode(CheckpointingMode.EXACTLY_ONCE)

# 确认 checkpoints 之间的时间会进行 500 ms
env.get_checkpoint_config().set_min_pause_between_checkpoints(500)

# Checkpoint 必须在一分钟内完成，否则就会被抛弃
env.get_checkpoint_config().set_checkpoint_timeout(60000)

# 同一时间只允许一个 checkpoint 进行
env.get_checkpoint_config().set_max_concurrent_checkpoints(1)

# 开启在 job 中止后仍然保留的 externalized checkpoints
env.get_checkpoint_config().enable_externalized_checkpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION)

# 允许在有更近 savepoint 时回退到 checkpoint
env.get_checkpoint_config().set_prefer_checkpoint_for_recovery(True)
```
{% endtab %}
{% endtabs %}

### 相关的配置项

可以通过conf/flink-conf.yaml设置更多参数和/或默认值\(详见[配置](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html)\):

| 参数 | 默认 | 描述 |
| :--- | :--- | :--- |
| **state.backend** | \(none\) | 状态后端用于存储和检查点状态。 |
| **state.backend.async** | true | 选择状态后端是否应在可能和可配置的情况下使用异步快照方法。某些状态后端可能不支持异步快照，或者仅支持异步快照，并忽略此选项。 |
| **state.backend.fs.memory-threshold** | 1024 | 状态数据文件的最小大小。小于该值的所有状态块都内联存储在根检查点元数据文件中。 |
| **state.backend.incremental** | false | 选项状态后端是否应该创建增量检查点\(如果可能的话\)。对于增量检查点，只存储与前一个检查点的差异，而不是完整的检查点状态。一些状态后端可能不支持增量检查点，并忽略此选项。 |
| **state.backend.local-recovery** | false | 此选项配置此状态后端的本地恢复。默认情况下，本地恢复是停用的。本地恢复目前只覆盖键控状态后端。目前，MemoryStateBackend不支持本地恢复，并忽略此选项。 |
| **state.checkpoints.dir** | \(none\) | 用于在Flink支持的文件系统中存储检查点的数据文件和元数据的默认目录。必须可以从所有参与的进程/节点（即所有TaskManagers和JobManagers）访问存储路径。 |
| **state.checkpoints.num-retained** | 1 | 要保留的已完成检查点的最大数量。 |
| **state.savepoints.dir** | \(none\) | 保存点的默认目录。由将后端写入文件系统的状态后端（MemoryStateBackend，FsStateBackend，RocksDBStateBackend）使用。 |
| **taskmanager.state.local.root-dirs** | \(none\) | 配置参数定义根目录，用于存储用于本地恢复的基于文件的状态。本地恢复目前只覆盖键控状态后端。目前，MemoryStateBackend不支持本地恢复，并忽略此选项 |

## 选择状态后端

Flink的[检查点机制](https://ci.apache.org/projects/flink/flink-docs-master/internals/stream_checkpointing.html)存储定时器和有状态运营商中所有状态的一致快照，包括连接器，窗口和任何[用户定义的状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html)。存储检查点的位置（例如，JobManager内存，文件系统，数据库）取决于配置的 **状态后端**。

默认情况下，状态保存在TaskManagers的内存中，检查点存储在JobManager的内存中。为了适当持久化大状态，Flink支持在其他状态后端中存储和检查点状态的各种方法。可以通过配置状态后端的选择`StreamExecutionEnvironment.setStateBackend(…)`。

有关可用状态后端的详细信息以及作业范围和群集范围配置的选项，请参阅[状态后端](https://ci.apache.org/projects/flink/flink-docs-master/ops/state/state_backends.html)。

## 迭代作业中的状态检查点

Flink目前仅为没有迭代的作业提供处理保证。在迭代作业上启用检查点会导致异常。为了强制对迭代程序进行检查点，用户在启用检查点时需要设置一个特殊标志：`env.enableCheckpointing(interval, force = true)`。

请注意，在失败期间，循环边缘中的记录（以及与它们相关的状态变化）将丢失。

## 重启策略

Flink支持不同的重启策略，可以控制在发生故障时如何重新启动作业。欲了解更多信息，请[重新启动策略](https://ci.apache.org/projects/flink/flink-docs-master/dev/restart_strategies.html)。



