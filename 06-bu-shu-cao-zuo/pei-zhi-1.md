# 配置

配置项所有配置都在`conf/flink-conf.yaml`中完成，该配置应该是YAML键值对的平面集合，格式为`key:value`。

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

* `io.tmp.dirs`：Flink放置本地数据的目录，默认为系统临时目录（`java.io.tmpdir`属性）。如果配置了目录列表，则Flink将在目录中轮换文件。

  默认情况下，放在这些目录中的数据包括RocksDB创建的文件，溢出的中间结果（批处理算法）和缓存的jar文件。

  持久性/恢复不依赖此数据，但是如果删除此数据，通常会导致重量级的恢复操作。因此，建议将其设置为不会自动定期清除的目录。

  默认情况下，Yarn，Mesos和Kubernetes设置会自动将此值配置为本地工作目录。

## 通用设置选项

_配置Flink应用程序或集群的常用配置项。_

### 主机和端口

用于为不同的Flink组件配置主机名和端口的配置项。

| 配置项 | 默认值 | 类型 | 描默认 |
| :--- | :--- | :--- | :--- |
| **jobmanager.rpc.address** | \(none\) | String | The config parameter defining the network address to connect to for communication with the job manager. This value is only interpreted in setups where a single JobManager with static name or address exists \(simple standalone setups, or container setups with dynamic service name resolution\). It is not used in many high-availability setups, when a leader-election service \(like ZooKeeper\) is used to elect and discover the JobManager leader from potentially multiple standby JobManagers. |
| **jobmanager.rpc.port** | 6123 | Integer | The config parameter defining the network port to connect to for communication with the job manager. Like jobmanager.rpc.address, this value is only interpreted in setups where a single JobManager with static name/address and port exists \(simple standalone setups, or container setups with dynamic service name resolution\). This config option is not used in many high-availability setups, when a leader-election service \(like ZooKeeper\) is used to elect and discover the JobManager leader from potentially multiple standby JobManagers. |
| **rest.address** | \(none\) | String | The address that should be used by clients to connect to the server. |
| **rest.bind-address** | \(none\) | String | The address that the server binds itself. |
| **rest.bind-port** | "8081" | String | The port that the server binds itself. Accepts a list of ports \(“50100,50101”\), ranges \(“50100-50200”\) or a combination of both. It is recommended to set a range of ports to avoid collisions when multiple Rest servers are running on the same machine. |
| **rest.port** | 8081 | Integer | The port that the client connects to. If rest.bind-port has not been specified, then the REST server will bind to this port. |
| **taskmanager.data.port** | 0 | Integer | The task manager’s port used for data exchange operations. |
| **taskmanager.host** | \(none\) | String | The address of the network interface that the TaskManager binds to. This option can be used to define explicitly a binding address. Because different TaskManagers need different values for this option, usually it is specified in an additional non-shared TaskManager-specific config file. |
| **taskmanager.rpc.port** | "0" | String | The task manager’s IPC port. Accepts a list of ports \(“50100,50101”\), ranges \(“50100-50200”\) or a combination of both. It is recommended to set a range of ports to avoid collisions when multiple TaskManagers are running on the same machine. |

### 容错能力

以下配置项控制Flink在执行过程中发生故障时的重启行为。通过在`flink-conf.yaml`中配置这些选项，可以定义集群的默认重启策略。

仅当尚未通过`ExecutionConfig`设置特定于作业的重启策略时，默认重启策略才会生效。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x914D;&#x7F6E;&#x9879;</th>
      <th style="text-align:left">&#x9ED8;&#x8BA4;&#x503C;</th>
      <th style="text-align:left">&#x7C7B;&#x578B;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>restart-strategy</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>Defines the restart strategy to use in case of job failures.
          <br />Accepted values are:</p>
        <ul>
          <li><code>none</code>, <code>off</code>, <code>disable</code>: No restart strategy.</li>
          <li><code>fixeddelay</code>, <code>fixed-delay</code>: Fixed delay restart
            strategy. More details can be found <a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#fixed-delay-restart-strategy">here</a>.</li>
          <li><code>failurerate</code>, <code>failure-rate</code>: Failure rate restart
            strategy. More details can be found <a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#failure-rate-restart-strategy">here</a>.</li>
        </ul>
        <p>If checkpointing is disabled, the default value is <code>none</code>. If
          checkpointing is enabled, the default value is <code>fixed-delay</code> with <code>Integer.MAX_VALUE</code> restart
          attempts and &apos;<code>1 s</code>&apos; delay.</p>
      </td>
    </tr>
  </tbody>
</table>

 **固定延迟重启策略**

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **restart-strategy.fixed-delay.attempts** | 1 | Integer | The number of times that Flink retries the execution before the job is declared as failed if `restart-strategy` has been set to `fixed-delay`. |
| **restart-strategy.fixed-delay.delay** | 1 s | Duration | Delay between two consecutive restart attempts if `restart-strategy` has been set to `fixed-delay`. Delaying the retries can be helpful when the program interacts with external systems where for example connections or pending transactions should reach a timeout before re-execution is attempted. It can be specified using notation: "1 min", "20 s" |

 **故障率重启策略**

| **配置项** | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **restart-strategy.failure-rate.delay** | 1 s | Duration | Delay between two consecutive restart attempts if `restart-strategy` has been set to `failure-rate`. It can be specified using notation: "1 min", "20 s" |
| **restart-strategy.failure-rate.failure-rate-interval** | 1 min | Duration | Time interval for measuring failure rate if `restart-strategy` has been set to `failure-rate`. It can be specified using notation: "1 min", "20 s" |
| **restart-strategy.failure-rate.max-failures-per-interval** | 1 | Integer | Maximum number of restarts in given time interval before failing a job if `restart-strategy` has been set to `failure-rate`. |

### 检查点及状态后端

以下是配置项项控制状态后端和检查点行为的基本设置。

这些选项仅适用于以连续流方式执行的作业/应用程序。以批处理方式执行的作业/应用程序不使用状态后端和检查点，而是使用为批处理优化的不同内部数据结构。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend** | \(none\) | String | The state backend to be used to store and checkpoint state. |
| **state.checkpoints.dir** | \(none\) | String | The default directory used for storing the data files and meta data of checkpoints in a Flink supported filesystem. The storage path must be accessible from all participating processes/nodes\(i.e. all TaskManagers and JobManagers\). |
| **state.savepoints.dir** | \(none\) | String | The default directory for savepoints. Used by the state backends that write savepoints to file systems \(MemoryStateBackend, FsStateBackend, RocksDBStateBackend\). |
| **state.backend.incremental** | false | Boolean | Option whether the state backend should create incremental checkpoints, if possible. For an incremental checkpoint, only a diff from the previous checkpoint is stored, rather than the complete checkpoint state. Some state backends may not support incremental checkpoints and ignore this option. |
| **state.backend.local-recovery** | false | Boolean | This option configures local recovery for this state backend. By default, local recovery is deactivated. Local recovery currently only covers keyed state backends. Currently, MemoryStateBackend does not support local recovery and ignore this option. |
| **state.checkpoints.num-retained** | 1 | Integer | The maximum number of completed checkpoints to retain. |
| **taskmanager.state.local.root-dirs** | \(none\) | String | The config parameter defining the root directories for storing file-based state for local recovery. Local recovery currently only covers keyed state backends. Currently, MemoryStateBackend does not support local recovery and ignore this option |

### 高可用性

这里的高可用性是指主控（JobManager）进程从故障中恢复的能力。

JobManager确保跨TaskManager恢复期间的一致性。为了使JobManager自身一致地恢复，外部服务必须存储最少数量的恢复元数据（例如“上次提交的检查点的ID”），并帮助选择和锁定哪个JobManager是领导者（避免出现脑裂情况） ）。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **high-availability** | "NONE" | String | Defines high-availability mode used for the cluster execution. To enable high-availability, set this mode to "ZOOKEEPER" or specify FQN of factory class. |
| **high-availability.cluster-id** | "/default" | String | The ID of the Flink cluster, used to separate multiple Flink clusters from each other. Needs to be set for standalone clusters but is automatically inferred in YARN and Mesos. |
| **high-availability.storageDir** | \(none\) | String | File system path \(URI\) where Flink persists metadata in high-availability setups. |

 **ZooKeeper的高可用性配置选项**

| **配置项** | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **high-availability.zookeeper.path.root** | "/flink" | String | The root path under which Flink stores its entries in ZooKeeper. |
| **high-availability.zookeeper.quorum** | \(none\) | String | The ZooKeeper quorum to use, when running Flink in a high-availability mode with ZooKeeper. |

### 内存配置

这些配置值控制TaskManager和JobManager使用内存的方式。

Flink尝试使用户免受配置JVM进行数据密集型处理的复杂性的影响。在大多数情况下，用户只需要设置值`taskmanager.memory.process.size`或`taskmanager.memory.flink.size`（取决于设置的方式），并可能通过调整JVM堆与托管内存的比率`taskmanager.memory.managed.fraction`。下面的其他选项可用于执行性能调整和修复与内存相关的错误。

有关这些选项如何交互的详细说明，请参阅[TaskManager内存配置文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_setup.html)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **taskmanager.memory.flink.size** | \(none\) | MemorySize | Total Flink Memory size for the TaskExecutors. This includes all the memory that a TaskExecutor consumes, except for JVM Metaspace and JVM Overhead. It consists of Framework Heap Memory, Task Heap Memory, Task Off-Heap Memory, Managed Memory, and Network Memory. See also 'taskmanager.memory.process.size' for total process memory size configuration. |
| **taskmanager.memory.framework.heap.size** | 128 mb | MemorySize | Framework Heap Memory size for TaskExecutors. This is the size of JVM heap memory reserved for TaskExecutor framework, which will not be allocated to task slots. |
| **taskmanager.memory.framework.off-heap.size** | 128 mb | MemorySize | Framework Off-Heap Memory size for TaskExecutors. This is the size of off-heap memory \(JVM direct memory and native memory\) reserved for TaskExecutor framework, which will not be allocated to task slots. The configured value will be fully counted when Flink calculates the JVM max direct memory size parameter. |
| **taskmanager.memory.jvm-metaspace.size** | 256 mb | MemorySize | JVM Metaspace Size for the TaskExecutors. |
| **taskmanager.memory.jvm-overhead.fraction** | 0.1 | Float | Fraction of Total Process Memory to be reserved for JVM Overhead. This is off-heap memory reserved for JVM overhead, such as thread stack space, compile cache, etc. This includes native memory but not direct memory, and will not be counted when Flink calculates JVM max direct memory size parameter. The size of JVM Overhead is derived to make up the configured fraction of the Total Process Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of JVM Overhead can be explicitly specified by setting the min/max size to the same value. |
| **taskmanager.memory.jvm-overhead.max** | 1 gb | MemorySize | Max JVM Overhead size for the TaskExecutors. This is off-heap memory reserved for JVM overhead, such as thread stack space, compile cache, etc. This includes native memory but not direct memory, and will not be counted when Flink calculates JVM max direct memory size parameter. The size of JVM Overhead is derived to make up the configured fraction of the Total Process Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of JVM Overhead can be explicitly specified by setting the min/max size to the same value. |
| **taskmanager.memory.jvm-overhead.min** | 192 mb | MemorySize | Min JVM Overhead size for the TaskExecutors. This is off-heap memory reserved for JVM overhead, such as thread stack space, compile cache, etc. This includes native memory but not direct memory, and will not be counted when Flink calculates JVM max direct memory size parameter. The size of JVM Overhead is derived to make up the configured fraction of the Total Process Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of JVM Overhead can be explicitly specified by setting the min/max size to the same value. |
| **taskmanager.memory.managed.fraction** | 0.4 | Float | Fraction of Total Flink Memory to be used as Managed Memory, if Managed Memory size is not explicitly specified. |
| **taskmanager.memory.managed.size** | \(none\) | MemorySize | Managed Memory size for TaskExecutors. This is the size of off-heap memory managed by the memory manager, reserved for sorting, hash tables, caching of intermediate results and RocksDB state backend. Memory consumers can either allocate memory from the memory manager in the form of MemorySegments, or reserve bytes from the memory manager and keep their memory usage within that boundary. If unspecified, it will be derived to make up the configured fraction of the Total Flink Memory. |
| **taskmanager.memory.network.fraction** | 0.1 | Float | Fraction of Total Flink Memory to be used as Network Memory. Network Memory is off-heap memory reserved for ShuffleEnvironment \(e.g., network buffers\). Network Memory size is derived to make up the configured fraction of the Total Flink Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of Network Memory can be explicitly specified by setting the min/max size to the same value. |
| **taskmanager.memory.network.max** | 1 gb | MemorySize | Max Network Memory size for TaskExecutors. Network Memory is off-heap memory reserved for ShuffleEnvironment \(e.g., network buffers\). Network Memory size is derived to make up the configured fraction of the Total Flink Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of Network Memory can be explicitly specified by setting the min/max to the same value. |
| **taskmanager.memory.network.min** | 64 mb | MemorySize | Min Network Memory size for TaskExecutors. Network Memory is off-heap memory reserved for ShuffleEnvironment \(e.g., network buffers\). Network Memory size is derived to make up the configured fraction of the Total Flink Memory. If the derived size is less/greater than the configured min/max size, the min/max size will be used. The exact size of Network Memory can be explicitly specified by setting the min/max to the same value. |
| **taskmanager.memory.process.size** | \(none\) | MemorySize | Total Process Memory size for the TaskExecutors. This includes all the memory that a TaskExecutor consumes, consisting of Total Flink Memory, JVM Metaspace, and JVM Overhead. On containerized setups, this should be set to the container memory. See also 'taskmanager.memory.flink.size' for total Flink memory size configuration. |
| **taskmanager.memory.task.heap.size** | \(none\) | MemorySize | Task Heap Memory size for TaskExecutors. This is the size of JVM heap memory reserved for tasks. If not specified, it will be derived as Total Flink Memory minus Framework Heap Memory, Task Off-Heap Memory, Managed Memory and Network Memory. |
| **taskmanager.memory.task.off-heap.size** | 0 bytes | MemorySize | Task Off-Heap Memory size for TaskExecutors. This is the size of off heap memory \(JVM direct memory and native memory\) reserved for tasks. The configured value will be fully counted when Flink calculates the JVM max direct memory size parameter. |

### 其他配置

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **fs.default-scheme** | \(none\) | String | The default filesystem scheme, used for paths that do not declare a scheme explicitly. May contain an authority, e.g. host:port in case of an HDFS NameNode. |
| **io.tmp.dirs** | 'LOCAL\_DIRS' on Yarn. '\_FLINK\_TMP\_DIR' on Mesos. System.getProperty\("java.io.tmpdir"\) in standalone. | String | Directories for temporary files, separated by",", "\|", or the system's java.io.File.pathSeparator. |

## 安全

用于配置Flink的安全性和与外部系统的安全交互的选项。

### SSL协议

 Flink的网络连接可以通过SSL进行保护。有关详细的设置指南和背景，请参考[SSL设置文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/security-ssl.html)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **security.ssl.algorithms** | "TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA" | String | The comma separated list of standard SSL algorithms to be supported. Read more [here](http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#ciphersuites) |
| **security.ssl.internal.cert.fingerprint** | \(none\) | String | The sha1 fingerprint of the internal certificate. This further protects the internal communication to present the exact certificate used by Flink.This is necessary where one cannot use private CA\(self signed\) or there is internal firm wide CA is required |
| **security.ssl.internal.enabled** | false | Boolean | Turns on SSL for internal network communication. Optionally, specific components may override this through their own settings \(rpc, data transport, REST, etc\). |
| **security.ssl.internal.key-password** | \(none\) | String | The secret to decrypt the key in the keystore for Flink's internal endpoints \(rpc, data transport, blob server\). |
| **security.ssl.internal.keystore** | \(none\) | String | The Java keystore file with SSL Key and Certificate, to be used Flink's internal endpoints \(rpc, data transport, blob server\). |
| **security.ssl.internal.keystore-password** | \(none\) | String | The secret to decrypt the keystore file for Flink's for Flink's internal endpoints \(rpc, data transport, blob server\). |
| **security.ssl.internal.truststore** | \(none\) | String | The truststore file containing the public CA certificates to verify the peer for Flink's internal endpoints \(rpc, data transport, blob server\). |
| **security.ssl.internal.truststore-password** | \(none\) | String | The password to decrypt the truststore for Flink's internal endpoints \(rpc, data transport, blob server\). |
| **security.ssl.protocol** | "TLSv1.2" | String | The SSL protocol version to be supported for the ssl transport. Note that it doesn’t support comma separated list. |
| **security.ssl.rest.authentication-enabled** | false | Boolean | Turns on mutual SSL authentication for external communication via the REST endpoints. |
| **security.ssl.rest.cert.fingerprint** | \(none\) | String | The sha1 fingerprint of the rest certificate. This further protects the rest REST endpoints to present certificate which is only used by proxy serverThis is necessary where once uses public CA or internal firm wide CA |
| **security.ssl.rest.enabled** | false | Boolean | Turns on SSL for external communication via the REST endpoints. |
| **security.ssl.rest.key-password** | \(none\) | String | The secret to decrypt the key in the keystore for Flink's external REST endpoints. |
| **security.ssl.rest.keystore** | \(none\) | String | The Java keystore file with SSL Key and Certificate, to be used Flink's external REST endpoints. |
| **security.ssl.rest.keystore-password** | \(none\) | String | The secret to decrypt the keystore file for Flink's for Flink's external REST endpoints. |
| **security.ssl.rest.truststore** | \(none\) | String | The truststore file containing the public CA certificates to verify the peer for Flink's external REST endpoints. |
| **security.ssl.rest.truststore-password** | \(none\) | String | The password to decrypt the truststore for Flink's external REST endpoints. |
| **security.ssl.verify-hostname** | true | Boolean | Flag to enable peer’s hostname verification during ssl handshake. |

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



