# 配置

所有配置都在`conf/flink-conf.yaml`中完成，该配置应该是YAML键值对的平面集合，格式为`key:value`。

启动Flink进程时，将分析并评估配置。 对配置文件的变更要求重新启动相关进程。

开箱即用的配置将使用默认的Java安装。 如果要手动覆盖要使用的Java运行时，则可以在`conf/flink-conf.yaml`中手动设置环境变量JAVA\_HOME或配置键env.java.home。

## 基本设置

默认配置支持在不进行任何更改的情况下启动单节点Flink会话群集。本节中的选项是基本的分布式Flink设置中最常用的配置项。

#### 主机名/端口号

这些选项仅对独立应用程序或会话部署（简单的独立或Kubernetes）是必需的。

如果将Flink与Yarn、Mesos或活动的Kubernetes集成一起使用，则会自动发现主机名和端口。

* `rest.address`，`rest.port`：客户端使用这两个配置来连接到Flink。将其设置为作业运行主服务器（JobManager）的主机名，指向Flink Master的REST接口前端的\(Kubernetes\)服务的主机名。
* `jobmanager.rpc.address`（默认为_“localhost”_）和`jobmanager.rpc.port`（默认为_6123\)_配置项由TaskManager用于连接到JobManager/ResourceManager。将其设置为主服务器\(JobManager\)运行的主机名，或设置为Flink主服务器\(JobManager\)的\(Kubernetes底层\)服务的主机名。在[具有高可用性的设置中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/jobmanager_high_availability.html)，使用leader选举机制自动发现此选项时，将忽略此选项。

#### 内存大小

默认内存大小支持简单的流/批处理应用程序，但是太低而无法为更复杂的应用程序提供良好的性能。

* `jobmanager.heap.size`：设置_Flink Master_（JobManager / ResourceManager / Dispatcher）JVM堆的大小。
* `taskmanager.memory.process.size`：TaskManager进程的总大小，包括所有内容。Flink将为JVM自身的内存需求（元空间和其他空间）减去一些内存，并在其组件（网络，托管内存，JVM Heap等）之间自动分配和配置其余内存。

这些值配置为内存大小，例如_1536m_或_2g_。

#### 并行度

* `taskmanager.numberOfTaskSlots`：TaskManager提供的插槽数_（默认值：1）_。每个插槽可以执行一项任务或管道。TaskManager中具有多个插槽可以帮助分摊并行任务或管道中的某些恒定开销（JVM，应用程序库或网络连接的开销）。有关详细信息，请参见[任务插槽和资源](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/runtime.html#task-slots-and-resources)概念部分。

  一个更好的起点是运行更多的更小的TaskManager，每个插槽有一个良好的起点，并且可以实现任务之间的最佳隔离。将相同的资源分配给具有更多插槽的较少的较大TaskManager，可以帮助提高资源利用率，但代价是任务之间的隔离性较弱（更多任务共享同一JVM）。

* `parallelism.default`：当未在任何地方指定并行性时使用的默认并行性_（默认值：1）_。

#### 检查点

可以直接在Flink作业或应用程序中的代码中配置检查点。在应用程序不进行任何配置的情况下，将这些值放在配置中会将它们定义为默认值。

* `state.backend`：要使用的状态后端。这定义了用于拍摄快照的数据结构机制。常用值为`filesystem`和 `rocksdb`。
* `state.checkpoints.dir`：要向其写入检查点的目录。这将采用_`s3://mybucket/flink-app / checkpoints`_或_`hdfs://namenode:port/flink/checkpoints`之_类的路径URI 。
* `state.savepoints.dir`：保存点的默认目录。采用类似于的路径URI `state.checkpoints.dir`。

#### Web UI

* `web.submit.enable`：启用通过Flink UI上载和启动作业_（默认为true）_。请注意，即使禁用此选项，会话群集仍会通过REST请求（HTTP调用）接受作业。此标志仅保护功能部件在UI中上载作业。
* `web.upload.dir`：存储上载作业的目录。仅当`web.submit.enable`为true 时使用。

#### 其他



## 通用设置选项

_配置Flink应用程序或集群的常用配置项。_

### 主机和端口

用于为不同的Flink组件配置主机名和端口的配置项。



### 容错能力

### 检查点及状态后端

### 高可用性

### 内存配置

### 其他配置

## 安全

### SSL协议

### 与外部系统进行身份验证

## 资源编排框架 <a id="resource-orchestration-frameworks"></a>

### Yarn

### Kubernetes

### Mesos

## 状态后端

### RocksDB状态后端

## 指标

### RocksDB本地指标

## History Server <a id="history-server"></a>

## 实验性 <a id="experimental"></a>

## 调试&专家调优

## JVM和日志记录选项

## 转发环境变量

## 不推荐使用的配置项

## 备份



