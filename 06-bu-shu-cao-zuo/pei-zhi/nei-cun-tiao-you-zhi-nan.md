# 内存调优指南

 除了[主内存设置指南之外](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html)，本节还介绍了如何根据使用场景设置任务执行程序的内存，以及在这种情况下哪些选项很重要。

## 配置独立\(Standalone\)部署内存

建议为独立部署配置总Flink内存\(taskmanager.memory.flink.size\)或其组件，以便声明有多少内存分配给Flink本身。此外，如果JVM元空间导致问题，您可以调整它。

整个进程内存并不相关，因为JVM开销不是由Flink或部署环境控制的，在这种情况下，只有执行机器的物理资源才重要。

## 配置容器的内存

建议为容器化部署\( [Kubernetes](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/kubernetes.html)、 [Yarn](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/yarn_setup.html)或 [Mesos](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/mesos.html)\)配置总进程内存\(`taskmanager.memory.process.size`\)。它声明总共有多少内存应该分配给Flink JVM进程，并与所请求容器的大小相对应。

{% hint style="info" %}
 注意：如果配置了_Flink_的_总内存，则_ Flink将隐式添加JVM内存组件以导出_总的进程内存，_并请求具有该派生大小的内存的容器，另请参见[详细的内存模型](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)。
{% endhint %}

{% hint style="warning" %}
警告：如果Flink或用户代码分配的非托管堆外\(本机\)内存超过容器大小，作业可能会失败，因为部署环境可能会杀死有问题的容器。
{% endhint %}

 另请参见[容器内存超出](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_trouble.html#container-memory-exceeded)故障的描述。

## 配置状态后端的内存

 部署Flink流应用程序时，使用的[状态后端](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)类型将决定群集的最佳内存配置。

### 堆状态后端

 在运行无状态作业或使用堆状态后端（[MemoryStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-memorystatebackend) 或[FsStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-fsstatebackend)）时，请将[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)设置为零。这将确保为JVM上的用户代码分配最大的内存量。

### RocksDB状态后端

 [RocksDBStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-rocksdbstatebackend)使用本机内存。默认情况下，RocksDB设置为将本机内存分配限制为托管内存的大小。因此，为状态使用场景保留足够的托管内存是很重要的。如果禁用默认的RocksDB内存控制，RocksDB分配的内存超过请求的容器大小限制\(总进程内存\)，任务执行器就会在容器化部署中被杀死。

 另请参阅[如何调整RocksDB内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/large_state_tuning.html#tuning-rocksdb-memory) 和[state.backend.rocksdb.memory.managed](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#state-backend-rocksdb-memory-managed)。

## 配置批处理作业的内存

 Flink的批处理运算符利用[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)来更有效地运行。这样，可以直接对原始数据执行某些操作，而不必将其反序列化为Java对象。这意味着[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)配置会对应用程序的性能产生实际影响。Flink将尝试分配和使用与批处理作业配置相同的托管内存，但不会超出其限制。这可以防止发生`OutOfMemoryError`，因为Flink确切知道它必须利用多少内存。如果[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)不足，Flink将正常溢出到磁盘。



