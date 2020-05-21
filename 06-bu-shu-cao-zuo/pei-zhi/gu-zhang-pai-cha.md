# 故障排查

## IllegalConfigurationException

 如果您看到从_TaskExecutorProcessUtils_抛出_IllegalConfigurationException_，则通常表明存在无效的配置值（例如，负的内存大小，大于1的分数等）或配置冲突。检查与 异常消息中提到的[内存组件](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)相关的文档章节。

## OutOfMemoryError: Java heap space

 该异常通常表明JVM堆太小。您可以尝试通过增加[总内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-total-memory)或[任务堆内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#task-operator-heap-memory)来增加JVM堆大小。

{% hint style="info" %}
 注意：还可以增加[框架堆的内存，](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html#framework-memory)但是此选项是高级的，只有在您确定Flink框架本身需要更多的内存时，才应更改此选项。
{% endhint %}

## OutOfMemoryError: Direct buffer memory

 异常通常表明JVM _直接内存_限制太小或存在_直接内存泄漏_。检查用户代码或其他外部依赖项是否使用了JVM _直接内存_，并且已正确解决了该问题。您可以尝试通过调整[直接堆外内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)来增加其限制。另请参阅[如何配置堆外内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#configure-off-heap-memory-direct-or-native)和Flink设置的[JVM参数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html#jvm-parameters)。

## OutOfMemoryError: Metaspace

 该异常通常表明[JVM元空间限制](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html#jvm-parameters)配置得太小。您可以尝试增加[JVM metaspace选项](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-jvm-metaspace-size)。

## IOException: Insufficient number of network buffers

异常通常表明配置的[网络内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html) 大小不足。您可以尝试通过调整以下选项来增加_网络内存_：

* [`taskmanager.memory.network.min`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-min)
* [`taskmanager.memory.network.max`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-max)
* [`taskmanager.memory.network.fraction`](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#taskmanager-memory-network-fraction)

## Container Memory Exceeded

如果任务执行器容器尝试分配超出其请求大小（Yarn，Mesos或Kubernetes）的内存，则通常表明Flink没有预留足够的本机内存。当容器被部署环境杀死时，您可以通过使用外部监视系统或从错误消息中观察到此情况。

如果使用[RocksDBStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-rocksdbstatebackend)且禁用了内存控制，则可以尝试增加[托管内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html#managed-memory)。

或者，可以增加[JVM开销](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_detail.html)。另请参阅[如何为容器配置内存](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_tuning.html#configure-memory-for-containers)。

