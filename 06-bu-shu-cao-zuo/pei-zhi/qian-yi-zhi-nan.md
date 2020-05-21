# 迁移指南

 1.10版[中任务管理器](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html)的[内存设置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html)发生了很大变化。许多配置选项已删除或它们的语义已更改。本指南将帮助您将内存配置从Flink [&lt;= _1.9_](https://ci.apache.org/projects/flink/flink-docs-release-1.9/ops/mem_setup.html)迁移到&gt; = _1.10_。

{% hint style="warning" %}
警告:回顾本指南非常重要，因为旧的和新的内存配置可能导致不同大小的内存组件。如果您试图重用1.10之前较老版本的Flink配置，可能会影响应用程序的功能、性能，甚至配置失败。
{% endhint %}

{% hint style="info" %}
 注意：在版本_1.10_之前，Flink完全不需要设置与内存相关的选项，因为它们都具有默认值。在[新的内存配置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)要求，下列选项至少一个子集显式配置，否则配置将失败：
{% endhint %}

* [`taskmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-flink-size)
* [`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-process-size)
* [`taskmanager.memory.task.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-task-heap-size) 和 [`taskmanager.memory.managed.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-size)

Flink附带的[默认`flink-conf.yaml`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#default-configuration-in-flink-confyaml)设置[`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-process-size) 可以使默认内存配置保持一致。

该[表格](https://docs.google.com/spreadsheets/d/1mJaMkMPfDJJ-w6nMXALYmTc4XxiV30P5U7DzgwLkSoE)还可以帮助评估和比较旧内存和新内存计算的结果。

## 配置选项的变更

本章简要地列出了在1.10版本中引入的对Flink内存配置选项的所有更改。关于迁移到新配置选项的更多细节，本文还参考了其他章节。

下面的配置项完全被删除了。如果它们仍然被使用，它们将被忽略。

| 删除的配置项 | 注意 |
| :--- | :--- |
| **taskmanager.memory.fraction** |  检查新配置项[taskmanager.memory. management .fraction](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-fraction%29)的描述。新配置项具有不同的语义，通常需要调整已弃用配置项的值。另请参阅[如何迁移托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory)。 |
| **taskmanager.memory.off-heap** |  不再支持堆上托管的内存。另请参阅[如何迁移托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory) |
| **taskmanager.memory.preallocate** |  不再支持预分配，并且总是延迟分配托管内存。另请参阅[如何迁移托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory) |

下面的配置项是不推荐的，但如果它们仍然被使用，它们将被解释为向后兼容的新配置项:

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x4E0D;&#x63A8;&#x8350;&#x4F7F;&#x7528;&#x7684;&#x914D;&#x7F6E;&#x9879;</th>
      <th
      style="text-align:left">&#x89E3;&#x91CA;&#x4E3A;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>taskmanager.heap.size</b>
      </td>
      <td style="text-align:left">
        <ul>
          <li>&#x7528;&#x4E8E;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/cluster_setup.html">&#x72EC;&#x7ACB;&#x90E8;&#x7F72;&#x7684;</a>
            <a
            href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-flink-size">taskmanager.memory.flink.size</a>
          </li>
          <li>&#x7528;&#x4E8E;&#x5BB9;&#x5668;&#x5316;&#x90E8;&#x7F72;&#x7684;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-process-size">taskmanager.memory.process.size</a>
          </li>
        </ul>
        <p>&#x53E6;&#x8BF7;&#x53C2;&#x9605;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#total-memory-previously-heap-memory">&#x5982;&#x4F55;&#x8FC1;&#x79FB;&#x603B;&#x5185;&#x5B58;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.memory.size</b>
      </td>
      <td style="text-align:left"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-size">taskmanager.memory.managed.size</a>&#xFF0C;&#x53E6;&#x8BF7;&#x53C2;&#x9605;
        <a
        href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory">&#x5982;&#x4F55;&#x8FC1;&#x79FB;&#x6258;&#x7BA1;&#x5185;&#x5B58;</a>&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.network.memory.min</b>
      </td>
      <td style="text-align:left"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-min">taskmanager.memory.network.min</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.network.memory.max</b>
      </td>
      <td style="text-align:left"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-max">taskmanager.memory.network.max</a>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.network.memory.fraction</b>
      </td>
      <td style="text-align:left"><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-fraction">taskmanager.memory.network.fraction</a>
      </td>
    </tr>
  </tbody>
</table> 尽管网络内存配置没有发生太大的变化，但建议检查其配置。如果其他内存组件有新的大小，它可以改变，例如，网络可以是总内存的一部分。 请参阅[新的详细内存模型](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)。

容器截止配置选项[`containerized.heap-cutoff-ratio`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/config.html#containerized-heap-cutoff-ratio) 和和[`containerized.heap-cutoff-min`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/config.html#containerized-heap-cutoff-min)不再对任务管理器进程有效，但它们对于作业管理器进程仍具有相同的语义。另请参阅[如何迁移容器边界](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#container-cut-off-memory)。

## 总内存（以前为堆内存）

之前负责Flink使用的总内存的选项是`taskmanager.heap.size`或`taskmanager.heap.mb`。尽管有这样的命名，但它们不仅包括JVM堆，还包括其他堆外内存组件。这些选项已被弃用。

 Mesos集成还有一个具有相同语义的单独选项：`mesos.resourcemanager.tasks.mem`不建议使用。

如果使用上述遗留选项而未指定相应的新选项，则它们将直接转换为以下新配置项：

* [`taskmanager.memory.flink.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-flink-size)独立部署的总Flink内存
* [`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-process-size)容器化部署（纱线或Mesos）的总进程内存

建议使用新配置项，而不是旧选项，因为在之后的版本中它们可能会完全删除。

请参阅[如何配置总内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)。

## JVM堆内存

JVM堆内存以前包括托管内存\(如果配置为在堆上\)和其他包括堆内存的任何其他用法的内存。 其余部分总是隐式地导出为总内存的剩余部分，请参阅[如何迁移托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory)。

现在，如果只配置了Flink总内存或总进程内存，那么JVM堆也派生为从总内存中减去所有其他组件后剩下的剩余部分， 请参阅[如何配置总内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)。

 此外，您现在可以更直接地控制分配给操作员任务（[`taskmanager.memory.task.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-task-heap-size)）的JVM堆，请参阅[任务（操作）堆内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#task-operator-heap-memory)。如果为流作业选择了JVM状态堆内存，则堆状态后端（[MemoryStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-memorystatebackend) 或[FsStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-fsstatebackend)）也会使用JVM堆内存。

 现在，JVM堆的一部分始终为Flink框架（[`taskmanager.memory.framework.heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-framework-heap-size)）保留。请参阅[框架内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html#framework-memory)。

## 托管内存

请参阅[如何配置托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)。

### 显式大小

先前配置托管内存大小（`taskmanager.memory.size`）的选项已重命名为 [`taskmanager.memory.managed.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-size)，并已弃用。建议使用新配置项，因为旧的配置项将会在之后版本中删除。

### 分数

如果没有显式设置，则可以预先将托管内存指定为总内存减去网络内存和容器截止\(仅用于Yarn和Mesos部署\)的分数\(taskmanager.memory.fraction\)。 此选项已被完全删除，如果仍然使用，将无效。请改用新选项[`taskmanager.memory.managed.fraction`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-fraction)。如果 [`taskmanager.memory.managed.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-managed-size)未通过显式设置[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)的大小，则此新选项会将[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)设置为[Flink内存总数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)的指定分数。

### RocksDB状态

 如果为流作业选择了[RocksDBStateBackend，](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-rocksdbstatebackend)那么它的本机内存消耗现在应该计入[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)中。。RocksDB内存分配受[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)大小的限制。这样可以防止容器在Yarn或Mesos上的损坏。您可以通过将[state.backend.rocksdb.memory.managed](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#state-backend-rocksdb-memory-managed)设置`false` 来禁用RocksDB内存控制。

请参阅[如何迁移容器边界](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#container-cut-off-memory)。

### 其他变化

此外，还进行了以下更改：

* [托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)现在总是脱离堆。配置选项`taskmanager.memory.off-heap`已删除，将不再起作用。
* 托管内存现在使用本机内存，而本机内存不是直接内存。这意味着托管内存不再计入JVM直接内存限制中。
* 托管内存现在总是被延迟分配。配置选项 `taskmanager.memory.preallocate`已删除，将不再起作用。

## 容器截止内存

对于容器化部署，可以在前面指定一个截止内存。此内存可以用于未占的内存分配。那些不是由Flink直接控制的依赖项是那些分配的主要来源，例如RocksDB、JVM的内部机制等。 它不再可用，并且相关的配置选项（`containerized.heap-cutoff-ratio`和`containerized.heap-cutoff-min`）将不再对任务管理器进程产生影响。新的内存模型引入了更具体的内存组件，进一步解决了这些问题。

在使用rocksdbstate后端的流作业中，RocksDB本机内存消耗现在应该作为托管内存的一部分来考虑。RocksDB内存分配也受配置的[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)大小限制。请参阅[迁移托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_migration.html#managed-memory)以及[如何配置托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)。

现在，其他直接或本机堆外内存消费者可以通过以下新的配置选项来解决:

* 任务堆外内存（[`taskmanager.memory.task.off-heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-task-off-heap-size)）
* 框架堆外内存（[`taskmanager.memory.framework.off-heap.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-framework-off-heap-size)）
* JVM metaspace（[`taskmanager.memory.jvm-metaspace.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-jvm-metaspace-size)）
* JVM overhead（另请参阅新的[内存模型](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)[详细](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)）

{% hint style="info" %}
注意：作业管理器仍然有容器截止内存配置选项。上面提到的配置选项对作业管理器仍然有效，方式与以前一样。
{% endhint %}

## flink-conf.yaml中的默认配置

本节介绍`flink-conf.yaml`Flink随附的默认设置的更改。

默认情况下，在`flink-conf.yaml`配置文件中总内存（`taskmanager.heap.size`）替换为[`taskmanager.memory.process.size`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-process-size)。该值也从1024Mb增加到1568Mb。

请参阅[如何配置总内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)。

