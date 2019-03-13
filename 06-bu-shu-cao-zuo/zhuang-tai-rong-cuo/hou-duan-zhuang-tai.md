# 状态后端

在使用[Data Stream API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html)编写的程序通常以各种形式保存状态：

* 窗口收集元素或聚合，直到触发它们
* 转换函数可以使用键/值状态接口来存储值
* 转换函数可以实现CheckpointedFunction接口，使其局部变量具有容错性。

另请参阅流API指南中的[状态部分](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/index.html)。

激活检查点时，检查点会持续保持此状态，以防止数据丢失并一致性恢复。状态如何在内部表示，以及在检查点上如何以及在何处保持**状态**取决于所选择的**状态后端**。

## 可用的状态后端

开箱即用，Flink捆绑了这些状态后端：

* _MemoryStateBackend_
* _FsStateBackend_
* _RocksDBStateBackend_

如果没有配置其他任何内容，系统将使用MemoryStateBackend。

### MemoryStateBackend

_MemoryStateBackend_保存数据在内部作为Java堆的对象。键/值状态和窗口运算符包含存储值，触发器等的哈希表。

在检查点时，此状态后端将对状态进行快照，并将其作为检查点确认消息的一部分发送到JobManager（主服务器），JobManager也将其存储在其堆上。

可以将MemoryStateBackend配置为使用异步快照。虽然我们强烈建议使用异步快照来避免阻塞管道，但请注意，默认情况下，此功能目前处于启用状态。要禁用此功能，用户可以在`MemoryStateBackend`构造函数中将相应的布尔标志实例化为`false`（这应该仅用于调试），例如：

```text
    new MemoryStateBackend(MAX_MEM_STATE_SIZE, false);
```

MemoryStateBackend的局限性：

* 默认情况下，每个状态的大小限制为5 MB。可以在`MemoryStateBackend`的构造函数中增加此值。
* 无论配置的最大状态大小如何，状态都不能大于akka帧大小（请参阅[配置](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html)）。
* 聚合状态必须适合于JobManager内存。

鼓励MemoryStateBackend用于：

* 本地开发和调试
* 几乎没有状态的作业，例如仅包含一次记录功能的作业（Map，FlatMap，Filter，...）。Kafka消费者需要很少的国家。

### FsStateBackend

FsStateBackend配置有文件系统URL（类型，地址，路径），例如“`hdfs://namenode:40010/flink/checkpoints`”或“`file:///data/flink/checkpoints`”。

FsStateBackend将正在运行的数据保存在TaskManager的内存中。 在检查点时，它将状态快照写入配置的文件系统和目录中的文件。 最小元数据存储在JobManager的内存中（或者，在高可用性模式下，存储在元数据检查点中）。

FsStateBackend _默认_使用_异步快照，_以避免在编写状态检查点时阻塞处理管道。要禁用此功能，用户可以在`FsStateBackend`在构造函数集中使用相应的布尔标志来实例化为`false`，例如：

```text
    new FsStateBackend(path, false);
```

鼓励FsStateBackend用于：

* 具有大状态，长窗口，大键/值状态的作业。
* 所有高可用性设置。

### RocksDBStateBackend

RocksDBStateBackend配置有文件系统URL（类型，地址，路径），例如“`hdfs://namenode:40010/flink/checkpoints`”或“`file:///data/flink/checkpoints`”。

RocksDBStateBackend在RocksDB数据库中保存正在进行的数据，该数据库（默认情况下）存储在TaskManager数据目录中。 在检查点时，整个RocksDB数据库将被保存到配置的文件系统和目录中。 最小元数据存储在JobManager的内存中（或者，在高可用性模式下，存储在元数据检查点中）。

RocksDBStateBackend始终执行异步快照。

RocksDBStateBackend的局限性：

* 由于RocksDB的JNI网桥API基于byte \[\]，因此每个密钥和每个值支持的最大大小为2 ^ 31个字节。 重要信息：在RocksDB中使用合并操作的状态（例如ListState）可以静默地累积&gt; 2 ^ 31字节的值大小，然后在下次检索时失败。 这是目前RocksDB JNI的一个限制。

鼓励RocksDBStateBackend用于：

* 具有非常大的状态，长窗口，大键/值状态的作业。
* 所有高可用性设置。

请注意，你可以保留的状态量仅受可用磁盘空间量的限制。 与将状态保持在内存中的`FsStateBackend`相比，这允许保持非常大的状态。 但是，这也意味着使用此状态后端可以实现的最大吞吐量更低。 对此后端的所有读/写都必须通过去/序列化来检索/存储状态对象，这也比使用基于堆的后端处理on-heap表示要昂贵得多。

RocksDBStateBackend是目前唯一提供增量检查点的后端（见[这里](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/large_state_tuning.html)）。

某些RocksDB本机指标可用但默认情况下已禁用，您可以[在此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html#rocksdb-native-metrics)找到完整文档

## 配置状态后端

如果您不指定任何内容，则默认状态后端是作业管理器。如果要为群集上的所有作业建立不同的默认值，可以通过在**flink-conf.yaml中**定义新的默认状态后端来**实现**。可以基于每个作业覆盖默认状态后端，如下所示。

### 设置每个作业状态后端

每个作业状态后端在作业的`StreamExecutionEnvironment`上设置，如下例所示：

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.setStateBackend(new FsStateBackend("hdfs://namenode:40010/flink/checkpoints"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment()
env.setStateBackend(new FsStateBackend("hdfs://namenode:40010/flink/checkpoints"))
```
{% endtab %}
{% endtabs %}

### 设置默认状态后端

可以在`flink-conf.yaml`中使用配置`state.backend`来指定默认状态后端。

配置项的可能值包括_jobmanager\(_`MemoryStateBackend`\)，_filesystem\(_`FsStateBackend`\)，_rocksdb\(_`RocksDBStateBackend`\)，或实现状态后端工厂[`FsStateBackendFactory`](https://github.com/apache/flink/blob/master/flink-runtime/src/main/java/org/apache/flink/runtime/state/filesystem/FsStateBackendFactory.java)的类的完全限定类名，例如`org.apache.flink.contrib.streaming.state.RocksDBStateBackendFactory`。

`state.checkpoints.dir`选项定义所有后端写入检查点数据和元数据文件的目录。可以[在此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/checkpoints.html#directory-structure)找到有关检查点目录结构的更多详细信息。

配置文件中的示例部分可能如下所示：

```text
# The backend that will be used to store operator state checkpoints

state.backend: filesystem

# Directory for storing checkpoints

state.checkpoints.dir: hdfs://namenode:40010/flink/checkpoints
```

### **RocksDB状态后端配置选项**

| Key | Default | Description |
| :--- | :--- | :--- |
| **state.backend.rocksdb.localdir** | \(none\) | RocksDB放置文件的本地目录（在TaskManager上） |
| **state.backend.rocksdb.timer-service.factory** | "HEAP" | 这确定了工厂的计时器服务状态实现。选项可以是基于RocksDB的HEAP（基于堆，默认）或ROCKSDB。 |

