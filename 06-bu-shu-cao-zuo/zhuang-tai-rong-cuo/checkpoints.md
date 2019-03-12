# Checkpoints

## 概述

检查点通过允许恢复状态和相应的流位置使Flink中的状态容错，从而为应用程序提供与无故障执行相同的语义。

请参阅[检查点](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/state/checkpointing.html)，了解如何为程序启用和配置检查点。

## 保留检查点

默认情况下，检查点不会保留，仅用于从失败中恢复作业。取消程序时会删除它们。但是，您可以配置要保留的定期检查点。根据配置 ，当作业失败或取消时，_不会_自动清除这些_保留的_检查点。这样，如果你的工作失败，你将有一个检查点来恢复。

```java
CheckpointConfig config = env.getCheckpointConfig();
config.enableExternalizedCheckpoints(ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION);
```

`ExternalizedCheckpointCleanup`模式配置取消作业时检查点发生的情况：

* **`ExternalizedCheckpointCleanup.RETAIN_ON_CANCELLATION`**：取消作业时保留检查点。请注意，在这种情况下，您必须在取消后手动清理检查点状态。
* **`ExternalizedCheckpointCleanup.DELETE_ON_CANCELLATION`**：取消作业时删除检查点。仅当作业失败时，检查点状态才可用。

### 目录结构

与[保存点](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html)类似，检查点由元数据文件和一些其他数据文件组成，具体取决于状态后端。元数据文件和数据文件存储在配置文件中配置的`state.checkpoints.dir`目录中，也可以为代码中的每个作业指定。

#### **通过配置文件全局配置**

```text
state.checkpoints.dir: hdfs:///checkpoints/
```

#### **在构造状态后端时为每个作业配置**

```java
env.setStateBackend(new RocksDBStateBackend("hdfs:///checkpoints-data/");
```

### **与**保存点的差异

检查点与[保存](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html)点有一些差异：

* 使用状态后端特定的\(低级\)数据格式，可以是增量的。 
* 不支持Flink的特定特性，如重新扫描。

### 从保留的检查点恢复

可以使用检查点的元数据文件从检查点恢复作业，就像从保存点恢复作业一样\(请参阅[保存点恢复指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/cli.html#restore-a-savepoint)\)。注意，如果元数据文件不是自包含的，JobManager需要访问它引用的数据文件\(参见上面的[目录结构](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/checkpoints.html#directory-structure)\)。

