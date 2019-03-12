# Savepoints

## 什么是Savepoint？Savepoint如何与Checkpoint不同？

Savepoint是流作业执行状态的一致映像，它是通过Flink的检查点机制创建的。你可以使用保存点来停止和恢复、分叉或更新Flink作业。保存点由两部分组成:一个目录，其中包含稳定存储\(例如HDFS、S3、…\)上的\(通常较大的\)二进制文件，以及一个\(相对较小的\)元数据文件。稳定存储上的文件表示作业执行状态映像的净数据。Savepoint的元数据文件以绝对路径的形式包含\(主要\)指向稳定存储中属于保存点的所有文件的指针。

{% hint style="warning" %}
**注意：**为了允许程序和Flink版本之间的升级，请务必查看以下有关[为操作符分配ID的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#assigning-operator-ids)部分。
{% endhint %}

从概念上讲，Flink的保存点不同于检查点，就像备份不同于传统数据库系统中的恢复日志一样。检查点的主要目的是在作业意外失败时提供恢复机制。检查点的生命周期由Flink管理，也就是说，检查点是由Flink创建、拥有和发布的，不需要用户交互。作为一种恢复和定期触发的方法，检查点实现的两个主要设计目标是:i\)尽可能轻量地创建，ii\)尽可能快地恢复。针对这些目标的优化可以利用某些属性，例如，在执行尝试之间作业代码不会更改。检查点通常在用户终止作业之后被删除\(除非显式地配置为保留的检查点\)。

与此相反，保存点由用户创建、拥有和删除。它们的用例用于计划、手动备份和恢复。例如，这可能是Flink版本的更新、更改作业图、更改并行度、为红/蓝部署创建第二个作业，等等。当然，保存点必须在作业终止后仍然存在。从概念上讲，生成和恢复保存点的成本可能会更高一些，并且更关注于可移植性和对作业的前面提到的更改的支持。

抛开这些概念上的差异，检查点和保存点的当前实现基本上使用相同的代码并产生相同的“格式”。然而，目前有一个例外，我们可能在未来引入更多的差异。例外情况是带有RocksDB状态后端的增量检查点。它们使用一些RocksDB内部格式，而不是Flink的本机保存点格式。与保存点相比，这使得它们成为更轻量级检查点机制的第一个实例。

## 分配操作符ID

**强烈建议**按照本节的描述调整你的程序，以便将来能够升级你的程序。所需的主要更改是通过`uid(String)`方法手动指定操作符id。这些id用于限定每个操作符的状态。

```text
DataStream<String> stream = env.
  // Stateful source (e.g. Kafka) with ID
  .addSource(new StatefulSource())
  .uid("source-id") // ID for the source operator
  .shuffle()
  // Stateful mapper with ID
  .map(new StatefulMapper())
  .uid("mapper-id") // ID for the mapper
  // Stateless printing sink
  .print(); // Auto-generated ID
```

如果不手动指定id，它们将自动生成。只要这些id不变，就可以从保存点自动恢复。生成的id依赖于程序的结构，并且对程序更改很敏感。因此，强烈建议手动分配这些id。

### 保存点状态

可以将保存点视为`Operator ID -> State`包含每个有状态运算符的映射：

```text
Operator ID | State
------------+------------------------
source-id   | State of StatefulSource
mapper-id   | State of StatefulMapper
```

在上面的示例中，打印接收器是无状态的，因此不是保存点状态的一部分。默认情况下，我们尝试将保存点的每个条目映射回新程序。

## 操作

您可以使用[命令行客户端](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/cli.html#savepoints)，以_触发保存点_，_取消作业用的保存点_，_从保存点恢复_和_处置保存点_。

使用Flink&gt; = 1.2.0，也可以使用webui _从保存点恢复_。

### 触发保存点

触发保存点时，会创建一个新的保存点目录，其中将存储数据和元数据。可以通过[配置默认目标目录](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#configuration)或使用触发器命令指定自定义目标目录来控制此目录的位置（请参阅[`:targetDirectory`参数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#trigger-a-savepoint)）。

{% hint style="warning" %}
**注意：**目标目录必须是JobManager（s）和TaskManager（例如分布式文件系统上的位置）可访问的位置。
{% endhint %}

例如，使用`FsStateBackend`或`RocksDBStateBackend`：

```text
# Savepoint target directory
/savepoints/

# Savepoint directory
/savepoints/savepoint-:shortjobid-:savepointid/

# Savepoint file contains the checkpoint meta data
/savepoints/savepoint-:shortjobid-:savepointid/_metadata

# Savepoint state
/savepoints/savepoint-:shortjobid-:savepointid/...
```

{% hint style="info" %}
**注意：** 虽然看起来好像可以移动保存点，但由于`_metadata`文件中的绝对路径，目前无法进行保存。请按照[FLINK-5778](https://issues.apache.org/jira/browse/FLINK-5778)了解取消此限制的进度。
{% endhint %}

请注意，如果使用`MemoryStateBackend`，则元数据_和_保存点状态将存储在`_metadata`文件中。由于它是自包含的，可以移动文件并从任何位置恢复。

{% hint style="warning" %}
注意:不建议移动或删除正在运行的作业的最后一个保存点，因为这可能会干扰故障恢复。保存点对精确一次的接收器有副作用，因此为了确保精确一次的语义，如果在最后一个保存点之后没有检查点，那么保存点将用于恢复。
{% endhint %}

**触发保存点**

```text
$ bin/flink savepoint :jobId [:targetDirectory]
```

这将触发具有ID的作业的保存点`:jobId`，并返回创建的保存点的路径。您需要此路径来还原和部署保存点。

**使用YARN触发保存点**

```text
$ bin/flink savepoint :jobId [:targetDirectory] -yid :yarnAppId
```

这将触发具有ID:jobId和YARN应用程序ID:yarnAppId的作业的保存点，并返回创建的保存点的路径。

**使用Savepoint取消作业**

```text
$ bin/flink cancel -s [:targetDirectory] :jobId
```

这将以原子方式触发具有ID的作业的保存点`:jobid`并取消作业。此外，您可以指定目标文件系统目录以存储保存点。该目录需要可由JobManager和TaskManager访问。

### 从保存点恢复

```text
$ bin/flink run -s :savepointPath [:runArgs]
```

这将提交作业并指定要从中恢复的保存点。您可以指定保存点目录或`_metadata`文件的路径。

#### **允许Non-Restored状态**

默认情况下，**恢复**操作将尝试将保存点的所有状态映射回您要还原的程序。如果删除了运算符，则可以通过`--allowNonRestoredState`（short `-n`:\)选项跳过无法映射到新程序的状态：

```text
$ bin/flink run -s :savepointPath -n [:runArgs]
```

### 处理保存点

```text
$ bin/flink savepoint -d :savepointPath
```

这将处理存储的保存点`:savepointPath`。

请注意，还可以通过常规的文件系统操作手动删除保存点，而不影响其他保存点或检查点\(请回忆一下，每个保存点都是自包含的\)。在Flink 1.2之前，使用上面的savepoint命令执行这个任务比较繁琐。

### 配置

可以通过`state.savepoints.dir`配置默认保存点目标目录。触发保存点时，此目录将用于存储保存点。可以通过使用触发器命令指定自定义目标目录来覆盖默认值（请参阅[`:targetDirectory`参数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/state/savepoints.html#trigger-a-savepoint)）。

```text
# Default savepoint target directory
state.savepoints.dir: hdfs:///flink/savepoints
```

如果既未配置缺省值也未指定自定义目标目录，则触发保存点将失败。

{% hint style="warning" %}
**注意：**目标目录必须是JobManager（s）和TaskManager（例如分布式文件系统上的位置）可访问的位置。
{% endhint %}

## 常见问题

### 我应该为我的任务中的所有操作员分配ID吗？

根据经验，是的。严格地说，仅通过uid方法将id分配给作业中的有状态操作符就足够了。保存点只包含这些操作符的状态，无状态操作符不属于保存点。

在实践中，建议将其分配给所有操作符，因为Flink的一些内置操作符\(如窗口操作符\)也是有状态的，而且不清楚哪些内置操作符实际上是有状态的，哪些不是。如果您完全确定某个操作符是无状态的，那么可以跳过uid方法。

### 如果我在任务中添加一个需要状态的新运算符，会发生什么？

当向作业中添加新操作符时，它将在没有任何状态的情况下初始化。保存点包含每个有状态操作符的状态。无状态操作符根本不是保存点的一部分。新操作符的行为类似于无状态操作符。

### 如果我从任务中删除一个具有状态的操作符，会发生什么?

默认情况下，保存点恢复将尝试将所有状态匹配回已恢复的作业。如果从包含已删除操作符的状态的保存点进行恢复，则此操作将失败。

可以通过run命令设置`——allowNonRestoredState (short: -n)`来允许非恢复状态:

```text
$ bin/flink run -s :savepointPath -n [:runArgs]
```

### 如果我在任务中重新排序有状态操作符，会发生什么?

如果将id分配给这些操作符，它们将像往常一样恢复。

如果没有分配id，有状态操作符的自动生成的id很可能在重新排序之后发生更改。这将导致您无法从以前的保存点进行恢复。

### 如果我添加、删除或重新排序作业中没有状态的操作符，会发生什么?

如果将id分配给有状态操作符，则无状态操作符不会影响保存点恢复。

如果没有分配id，有状态操作符的自动生成的id很可能在重新排序之后发生更改。这将导致您无法从以前的保存点进行恢复。

### 当我在恢复时改变程序的并行度会发生什么?

如果保存点是用Flink &gt;= 1.2.0触发的，并且不使用检查点之类的弃用状态API，那么可以简单地从保存点恢复程序并指定一个新的并行性。

如果正在从Flink &lt; 1.2.0触发的保存点恢复，或者使用现在已经废弃的api，那么首先必须将作业迁移到Flink &gt;= 1.2.0，然后才能更改并行性。参见升级作业和Flink版本指南。

### 我可以将保存点文件移动到稳定存储上吗?

对这个问题的快速回答是“否”，因为出于技术原因，元数据文件将稳定存储上的文件引用为绝对路径。更长的答案是:如果由于某种原因必须移动文件，有两种可能的解决方法。首先，更简单但可能更危险的是，您可以使用编辑器在元数据文件中查找旧路径，并用新路径替换它们。其次，可以使用SavepointV2Serializer类作为起点，以编程方式使用新路径读取、操作和重写元数据文件。



