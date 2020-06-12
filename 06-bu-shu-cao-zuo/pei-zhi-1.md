# 配置

类型类型p配置项描述描述配置项所有配置都在`conf/flink-conf.yaml`中完成，该配置应该是YAML键值对的平面集合，格式为`key:value`。

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

 **ZooKeeper认证/授权**

当连接到安全的ZooKeeper仲裁时，这些选项是必需的。

| **配置项** | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **zookeeper.sasl.disable** | false | Boolean |  |
| **zookeeper.sasl.login-context-name** | "Client" | String |  |
| **zookeeper.sasl.service-name** | "zookeeper" | String |  |

 **基于Kerberos的身份验证/授权**

有关设置指南和外部系统列表，请参考[Flink和Kerberos](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/security-kerberos.html)文档，Flink可以通过Kerberos对自己进行身份验证。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **security.kerberos.login.contexts** | \(none\) | String | A comma-separated list of login contexts to provide the Kerberos credentials to \(for example, \`Client,KafkaClient\` to use the credentials for ZooKeeper authentication and for Kafka authentication\) |
| **security.kerberos.login.keytab** | \(none\) | String | Absolute path to a Kerberos keytab file that contains the user credentials. |
| **security.kerberos.login.principal** | \(none\) | String | Kerberos principal name associated with the keytab. |
| **security.kerberos.login.use-ticket-cache** | true | Boolean | Indicates whether to read from your Kerberos ticke |

## 资源编排框架 <a id="resource-orchestration-frameworks"></a>

本节包含与将Flink集成到资源编配框架\(如Kubernetes、Yarn、Mesos等\)相关的选项。

注意，将Flink与资源编排框架集成并不总是必要的。例如，您可以轻松地在Kubernetes上部署Flink应用程序，而不需要Flink知道它在Kubernetes上运行\(这里不指定任何Kubernetes配置选项\)。有关示例，请参阅此[配置安装指南](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/kubernetes.html)。

对于Flink本身主动从协调器请求和释放资源的设置，此部分中的选项是必需的。

### Yarn

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
      <td style="text-align:left"><b>yarn.application-attempt-failures-validity-interval</b>
      </td>
      <td style="text-align:left">10000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Time window in milliseconds which defines the number of application attempt
        failures when restarting the AM. Failures which fall outside of this window
        are not being considered. Set this value to -1 in order to count globally.
        See <a href="https://hortonworks.com/blog/apache-hadoop-yarn-hdp-2-2-fault-tolerance-features-long-running-services/">here</a> for
        more information.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application-attempts</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Number of ApplicationMaster restarts. Note that that the entire Flink
        cluster will restart and the YARN Client will loose the connection. Also,
        the JobManager address will change and you&#x2019;ll need to set the JM
        host:port manually. It is recommended to leave this option at 1.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application-master.port</b>
      </td>
      <td style="text-align:left">&quot;0&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">With this configuration option, users can specify a port, a range of ports
        or a list of ports for the Application Master (and JobManager) RPC port.
        By default we recommend using the default value (0) to let the operating
        system choose an appropriate port. In particular when multiple AMs are
        running on the same physical host, fixed port assignments prevent the AM
        from starting. For example when running Flink on YARN on an environment
        with a restrictive firewall, this option allows specifying a range of allowed
        ports.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.id</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The YARN application id of the running yarn cluster. This is the YARN
        cluster where the pipeline is going to be executed.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.name</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">A custom name for your YARN application.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.node-label</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Specify YARN node label for the YARN application.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.priority</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">A non-negative integer indicating the priority for submitting a Flink
        YARN application. It will only take effect if YARN priority scheduling
        setting is enabled. Larger integer corresponds with higher priority. If
        priority is negative or set to &apos;-1&apos;(default), Flink will unset
        yarn priority setting and use cluster default priority. Please refer to
        YARN&apos;s official documentation for specific settings required to enable
        priority scheduling for the targeted YARN version.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.queue</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The YARN queue on which to put the current pipeline.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.application.type</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">A custom type for your YARN application..</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.appmaster.rpc.address</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The hostname or address where the application master RPC system is listening.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.appmaster.rpc.port</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The port where the application master RPC system is listening.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.appmaster.vcores</b>
      </td>
      <td style="text-align:left">1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The number of virtual cores (vcores) used by YARN application master.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.containers.vcores</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The number of virtual cores (vcores) per YARN container. By default, the
        number of vcores is set to the number of slots per TaskManager, if set,
        or to 1, otherwise. In order for this parameter to be used your cluster
        must have CPU scheduling enabled. You can do this by setting the <code>org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.flink-dist-jar</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The location of the Flink dist jar.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.heartbeat.container-request-interval</b>
      </td>
      <td style="text-align:left">500</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">
        <p>Time between heartbeats with the ResourceManager in milliseconds if Flink
          requests containers:</p>
        <ul>
          <li>The lower this value is, the faster Flink will get notified about container
            allocations since requests and allocations are transmitted via heartbeats.</li>
          <li>The lower this value is, the more excessive containers might get allocated
            which will eventually be released but put pressure on Yarn.</li>
        </ul>
        <p>If you observe too many container allocations on the ResourceManager,
          then it is recommended to increase this value. See <a href="https://issues.apache.org/jira/browse/YARN-1902">this link</a> for
          more information.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.heartbeat.interval</b>
      </td>
      <td style="text-align:left">5</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">Time between heartbeats with the ResourceManager in seconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.maximum-failed-containers</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Maximum number of containers the system is going to reallocate in case
        of a failure.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.per-job-cluster.include-user-jar</b>
      </td>
      <td style="text-align:left">&quot;ORDER&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Defines whether user-jars are included in the system class path for per-job-clusters
        as well as their positioning in the path. They can be positioned at the
        beginning (&quot;FIRST&quot;), at the end (&quot;LAST&quot;), or be positioned
        based on their name (&quot;ORDER&quot;). &quot;DISABLED&quot; means the
        user-jars are excluded from the system class path.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.properties-file.location</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">When a Flink job is submitted to YARN, the JobManager&#x2019;s host and
        the number of available processing slots is written into a properties file,
        so that the Flink client is able to pick those details up. This configuration
        parameter allows changing the default location of that file (for example
        for environments sharing a Flink installation between users).</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.ship-directories</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">A semicolon-separated list of directories to be shipped to the YARN cluster.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>yarn.tags</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">A comma-separated list of tags to apply to the Flink YARN application.</td>
    </tr>
  </tbody>
</table>

### Kubernetes

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **kubernetes.cluster-id** | \(none\) | String | The cluster id used for identifying the unique flink cluster. If it's not set, the client will generate a random UUID name. |
| **kubernetes.config.file** | \(none\) | String | The kubernetes config file will be used to create the client. The default is located at ~/.kube/config |
| **kubernetes.container-start-command-template** | "%java% %classpath% %jvmmem% %jvmopts% %logging% %class% %args% %redirects%" | String | Template for the kubernetes jobmanager and taskmanager container start invocation. |
| **kubernetes.container.image** | "flink:latest" | String | Image to use for Flink containers. |
| **kubernetes.container.image.pull-policy** | "IfNotPresent" | String | Kubernetes image pull policy. Valid values are Always, Never, and IfNotPresent. The default policy is IfNotPresent to avoid putting pressure to image repository. |
| **kubernetes.entry.path** | "/opt/flink/bin/kubernetes-entry.sh" | String | The entrypoint script of kubernetes in the image. It will be used as command for jobmanager and taskmanager container. |
| **kubernetes.flink.conf.dir** | "/opt/flink/conf" | String | The flink conf directory that will be mounted in pod. The flink-conf.yaml, log4j.properties, logback.xml in this path will be overwritten from config map. |
| **kubernetes.flink.log.dir** | "/opt/flink/log" | String | The directory that logs of jobmanager and taskmanager be saved in the pod. |
| **kubernetes.jobmanager.cpu** | 1.0 | Double | The number of cpu used by job manager |
| **kubernetes.jobmanager.service-account** | "default" | String | Service account that is used by jobmanager within kubernetes cluster. The job manager uses this service account when requesting taskmanager pods from the API server. |
| **kubernetes.namespace** | "default" | String | The namespace that will be used for running the jobmanager and taskmanager pods. |
| **kubernetes.rest-service.exposed.type** | "LoadBalancer" | String | It could be ClusterIP/NodePort/LoadBalancer\(default\). When set to ClusterIP, the rest servicewill not be created. |
| **kubernetes.service.create-timeout** | "1 min" | String | Timeout used for creating the service. The timeout value requires a time-unit specifier \(ms/s/min/h/d\). |
| **kubernetes.taskmanager.cpu** | -1.0 | Double | The number of cpu used by task manager. By default, the cpu is set to the number of slots per TaskManager |

### Mesos

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
      <td style="text-align:left"><b>mesos.failover-timeout</b>
      </td>
      <td style="text-align:left">604800</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The failover timeout in seconds for the Mesos scheduler, after which running
        tasks are automatically shut down.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.master</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>The Mesos master URL. The value should be in one of the following forms:</p>
        <ul>
          <li>host:port</li>
          <li>zk://host1:port1,host2:port2,.../path</li>
          <li>zk://username:password@host1:port1,host2:port2,.../path</li>
          <li>file:///path/to/file</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.artifactserver.port</b>
      </td>
      <td style="text-align:left">0</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The config parameter defining the Mesos artifact server port to use. Setting
        the port to 0 will let the OS choose an available port.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.artifactserver.ssl.enabled</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Enables SSL for the Flink artifact server. Note that security.ssl.enabled
        also needs to be set to true encryption to enable encryption.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.declined-offer-refuse-duration</b>
      </td>
      <td style="text-align:left">5000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Amount of time to ask the Mesos master to not resend a declined resource
        offer again. This ensures a declined resource offer isn&apos;t resent immediately
        after being declined</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.name</b>
      </td>
      <td style="text-align:left">&quot;Flink&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Mesos framework name</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.principal</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Mesos framework principal</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.role</b>
      </td>
      <td style="text-align:left">&quot;*&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Mesos framework role definition</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.secret</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Mesos framework secret</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.user</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Mesos framework user</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.tasks.port-assignments</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Comma-separated list of configuration keys which represent a configurable
        port. All port keys will dynamically get a port assigned through Mesos.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.unused-offer-expiration</b>
      </td>
      <td style="text-align:left">120000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Amount of time to wait for unused expired offers before declining them.
        This ensures your scheduler will not hoard unuseful offers.</td>
    </tr>
  </tbody>
</table>

 **Mesos TaskManager**

| **配置项** | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **mesos.constraints.hard.hostattribute** | \(none\) | String | Constraints for task placement on Mesos based on agent attributes. Takes a comma-separated list of key:value pairs corresponding to the attributes exposed by the target mesos agents. Example: az:eu-west-1a,series:t2 |
| **mesos.resourcemanager.tasks.bootstrap-cmd** | \(none\) | String | A command which is executed before the TaskManager is started. |
| **mesos.resourcemanager.tasks.container.docker.force-pull-image** | false | Boolean | Instruct the docker containerizer to forcefully pull the image rather than reuse a cached version. |
| **mesos.resourcemanager.tasks.container.docker.parameters** | \(none\) | String | Custom parameters to be passed into docker run command when using the docker containerizer. Comma separated list of "key=value" pairs. The "value" may contain '='. |
| **mesos.resourcemanager.tasks.container.image.name** | \(none\) | String | Image name to use for the container. |
| **mesos.resourcemanager.tasks.container.type** | "mesos" | String | Type of the containerization used: “mesos” or “docker”. |
| **mesos.resourcemanager.tasks.container.volumes** | \(none\) | String | A comma separated list of \[host\_path:\]container\_path\[:RO\|RW\]. This allows for mounting additional volumes into your container. |
| **mesos.resourcemanager.tasks.cpus** | 0.0 | Double | CPUs to assign to the Mesos workers. |
| **mesos.resourcemanager.tasks.disk** | 0 | Integer | Disk space to assign to the Mesos workers in MB. |
| **mesos.resourcemanager.tasks.gpus** | 0 | Integer | GPUs to assign to the Mesos workers. |
| **mesos.resourcemanager.tasks.hostname** | \(none\) | String | Optional value to define the TaskManager’s hostname. The pattern \_TASK\_ is replaced by the actual id of the Mesos task. This can be used to configure the TaskManager to use Mesos DNS \(e.g. \_TASK\_.flink-service.mesos\) for name lookups. |
| **mesos.resourcemanager.tasks.taskmanager-cmd** | "$FLINK\_HOME/bin/mesos-taskmanager.sh" | String |  |
| **mesos.resourcemanager.tasks.uris** | \(none\) | String | A comma separated list of URIs of custom artifacts to be downloaded into the sandbox of Mesos workers. |
| **taskmanager.numberOfTaskSlots** | 1 | Integer | The number of parallel operator or user function instances that a single TaskManager can run. If this value is larger than 1, a single TaskManager takes multiple instances of a function or operator. That way, the TaskManager can utilize multiple CPU cores, but at the same time, the available memory is divided between the different operator or function instances. This value is typically proportional to the number of physical CPU cores that the TaskManager's machine has \(e.g., equal to the number of cores, or half the number of cores\). |

## 状态后端

 请参阅[状态后端文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)以及[状态后端](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)的背景。

### RocksDB状态后端

 这些是配置RocksDB状态后端通常需要的选项。有关高级低级配置和故障排除所必需的选项，请参见[Advanced RocksDB后端部分](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#advanced-rocksdb-state-backends-options)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend.rocksdb.memory.fixed-per-slot** | \(none\) | MemorySize | The fixed total amount of memory, shared among all RocksDB instances per slot. This option overrides the 'state.backend.rocksdb.memory.managed' option when configured. If neither this option, nor the 'state.backend.rocksdb.memory.managed' optionare set, then each RocksDB column family state has its own memory caches \(as controlled by the column family options\). |
| **state.backend.rocksdb.memory.high-prio-pool-ratio** | 0.1 | Double | The fraction of cache memory that is reserved for high-priority data like index, filter, and compression dictionary blocks. This option only has an effect when 'state.backend.rocksdb.memory.managed' or 'state.backend.rocksdb.memory.fixed-per-slot' are configured. |
| **state.backend.rocksdb.memory.managed** | true | Boolean | If set, the RocksDB state backend will automatically configure itself to use the managed memory budget of the task slot, and divide the memory over write buffers, indexes, block caches, etc. That way, the three major uses of memory of RocksDB will be capped. |
| **state.backend.rocksdb.memory.write-buffer-ratio** | 0.5 | Double | The maximum amount of memory that write buffers may take, as a fraction of the total shared memory. This option only has an effect when 'state.backend.rocksdb.memory.managed' or 'state.backend.rocksdb.memory.fixed-per-slot' are configured. |
| **state.backend.rocksdb.timer-service.factory** | "ROCKSDB" | String | This determines the factory for timer service state implementation. Options are either HEAP \(heap-based\) or ROCKSDB for an implementation based on RocksDB. |

## 指标

 请参阅[指标系统文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/metrics.html)以了解Flink指标基础结构的背景。

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
      <td style="text-align:left"><b>metrics.fetcher.update-interval</b>
      </td>
      <td style="text-align:left">10000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Update interval for the metric fetcher used by the web UI in milliseconds.
        Decrease this value for faster updating metrics. Increase this value if
        the metric fetcher causes too much load. Setting this value to 0 disables
        the metric fetching completely.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.internal.query-service.port</b>
      </td>
      <td style="text-align:left">&quot;0&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The port range used for Flink&apos;s internal metric query service. Accepts
        a list of ports (&#x201C;50100,50101&#x201D;), ranges(&#x201C;50100-50200&#x201D;)
        or a combination of both. It is recommended to set a range of ports to
        avoid collisions when multiple Flink components are running on the same
        machine. Per default Flink will pick a random port.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.internal.query-service.thread-priority</b>
      </td>
      <td style="text-align:left">1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The thread priority used for Flink&apos;s internal metric query service.
        The thread is created by Akka&apos;s thread pool executor. The range of
        the priority is from 1 (MIN_PRIORITY) to 10 (MAX_PRIORITY). Warning, increasing
        this value may bring the main Flink components down.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.latency.granularity</b>
      </td>
      <td style="text-align:left">&quot;operator&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>Defines the granularity of latency metrics. Accepted values are:</p>
        <ul>
          <li>single - Track latency without differentiating between sources and subtasks.</li>
          <li>operator - Track latency while differentiating between sources, but not
            subtasks.</li>
          <li>subtask - Track latency while differentiating between sources and subtasks.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.latency.history-size</b>
      </td>
      <td style="text-align:left">128</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">Defines the number of measured latencies to maintain at each operator.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.latency.interval</b>
      </td>
      <td style="text-align:left">0</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Defines the interval at which latency tracking marks are emitted from
        the sources. Disables latency tracking if set to 0 or a negative value.
        Enabling this feature can significantly impact the performance of the cluster.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.reporter.&lt;name&gt;.&lt;parameter&gt;</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Configures the parameter &lt;parameter&gt; for the reporter named &lt;name&gt;.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.reporter.&lt;name&gt;.class</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The reporter class to use for the reporter named &lt;name&gt;.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.reporter.&lt;name&gt;.interval</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The reporter interval to use for the reporter named &lt;name&gt;.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.reporters</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">An optional list of reporter names. If configured, only reporters whose
        name matches any of the names in the list will be started. Otherwise, all
        reporters that could be found in the configuration will be started.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.delimiter</b>
      </td>
      <td style="text-align:left">&quot;.&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Delimiter used to assemble the metric identifier.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.jm</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.jobmanager&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
        to a JobManager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.jm.job</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.jobmanager.&lt;job_name&gt;&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
        to a job on a JobManager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.operator</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.taskmanager.&lt;tm_id&gt;.&lt;job_name&gt;.&lt;operator_name&gt;.&lt;subtask_index&gt;&quot;</td>
      <td
      style="text-align:left">String</td>
        <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
          to an operator.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.task</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.taskmanager.&lt;tm_id&gt;.&lt;job_name&gt;.&lt;task_name&gt;.&lt;subtask_index&gt;&quot;</td>
      <td
      style="text-align:left">String</td>
        <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
          to a task.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.tm</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.taskmanager.&lt;tm_id&gt;&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
        to a TaskManager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.scope.tm.job</b>
      </td>
      <td style="text-align:left">&quot;&lt;host&gt;.taskmanager.&lt;tm_id&gt;.&lt;job_name&gt;&quot;</td>
      <td
      style="text-align:left">String</td>
        <td style="text-align:left">Defines the scope format string that is applied to all metrics scoped
          to a job on a TaskManager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.system-resource</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Flag indicating whether Flink should report system resource metrics such
        as machine&apos;s CPU, memory or network usage.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>metrics.system-resource-probing-interval</b>
      </td>
      <td style="text-align:left">5000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Interval between probing of system resource metrics specified in milliseconds.
        Has an effect only when &apos;metrics.system-resource&apos; is enabled.</td>
    </tr>
  </tbody>
</table>

### RocksDB本地指标

对于使用RocksDB状态后端的应用程序，Flink可以报告来自RocksDB本机代码的指标。这里的度量范围是操作符，然后进一步细分为列族;值被报告为无符号的long。

{% hint style="warning" %}
注意:启用RocksDB的本机指标可能会导致性能下降，应该仔细设置。
{% endhint %}

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend.rocksdb.metrics.actual-delayed-write-rate** | false | Boolean | Monitor the current actual delayed write rate. 0 means no delay. |
| **state.backend.rocksdb.metrics.background-errors** | false | Boolean | Monitor the number of background errors in RocksDB. |
| **state.backend.rocksdb.metrics.block-cache-capacity** | false | Boolean | Monitor block cache capacity. |
| **state.backend.rocksdb.metrics.block-cache-pinned-usage** | false | Boolean | Monitor the memory size for the entries being pinned in block cache. |
| **state.backend.rocksdb.metrics.block-cache-usage** | false | Boolean | Monitor the memory size for the entries residing in block cache. |
| **state.backend.rocksdb.metrics.column-family-as-variable** | false | Boolean | Whether to expose the column family as a variable. |
| **state.backend.rocksdb.metrics.compaction-pending** | false | Boolean | Track pending compactions in RocksDB. Returns 1 if a compaction is pending, 0 otherwise. |
| **state.backend.rocksdb.metrics.cur-size-active-mem-table** | false | Boolean | Monitor the approximate size of the active memtable in bytes. |
| **state.backend.rocksdb.metrics.cur-size-all-mem-tables** | false | Boolean | Monitor the approximate size of the active and unflushed immutable memtables in bytes. |
| **state.backend.rocksdb.metrics.estimate-live-data-size** | false | Boolean | Estimate of the amount of live data in bytes. |
| **state.backend.rocksdb.metrics.estimate-num-keys** | false | Boolean | Estimate the number of keys in RocksDB. |
| **state.backend.rocksdb.metrics.estimate-pending-compaction-bytes** | false | Boolean | Estimated total number of bytes compaction needs to rewrite to get all levels down to under target size. Not valid for other compactions than level-based. |
| **state.backend.rocksdb.metrics.estimate-table-readers-mem** | false | Boolean | Estimate the memory used for reading SST tables, excluding memory used in block cache \(e.g.,filter and index blocks\) in bytes. |
| **state.backend.rocksdb.metrics.is-write-stopped** | false | Boolean | Track whether write has been stopped in RocksDB. Returns 1 if write has been stopped, 0 otherwise. |
| **state.backend.rocksdb.metrics.mem-table-flush-pending** | false | Boolean | Monitor the number of pending memtable flushes in RocksDB. |
| **state.backend.rocksdb.metrics.num-deletes-active-mem-table** | false | Boolean | Monitor the total number of delete entries in the active memtable. |
| **state.backend.rocksdb.metrics.num-deletes-imm-mem-tables** | false | Boolean | Monitor the total number of delete entries in the unflushed immutable memtables. |
| **state.backend.rocksdb.metrics.num-entries-active-mem-table** | false | Boolean | Monitor the total number of entries in the active memtable. |
| **state.backend.rocksdb.metrics.num-entries-imm-mem-tables** | false | Boolean | Monitor the total number of entries in the unflushed immutable memtables. |
| **state.backend.rocksdb.metrics.num-immutable-mem-table** | false | Boolean | Monitor the number of immutable memtables in RocksDB. |
| **state.backend.rocksdb.metrics.num-live-versions** | false | Boolean | Monitor number of live versions. Version is an internal data structure. See RocksDB file version\_set.h for details. More live versions often mean more SST files are held from being deleted, by iterators or unfinished compactions. |
| **state.backend.rocksdb.metrics.num-running-compactions** | false | Boolean | Monitor the number of currently running compactions. |
| **state.backend.rocksdb.metrics.num-running-flushes** | false | Boolean | Monitor the number of currently running flushes. |
| **state.backend.rocksdb.metrics.num-snapshots** | false | Boolean | Monitor the number of unreleased snapshots of the database. |
| **state.backend.rocksdb.metrics.size-all-mem-tables** | false | Boolean | Monitor the approximate size of the active, unflushed immutable, and pinned immutable memtables in bytes. |
| **state.backend.rocksdb.metrics.total-sst-files-size** | false | Boolean | Monitor the total size \(bytes\) of all SST files.WARNING: may slow down online queries if there are too many files. |

## History Server <a id="history-server"></a>

历史记录服务器保留已完成作业的信息（图形，运行时，统计信息）。要启用它，您必须在JobManager（`jobmanager.archive.fs.dir`）中启用“作业存档” 。

有关详细信息，请参见[History Server Docs](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/historyserver.html)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **historyserver.archive.clean-expired-jobs** | false | Boolean | Whether HistoryServer should cleanup jobs that are no longer present \`historyserver.archive.fs.dir\`. |
| **historyserver.archive.fs.dir** | \(none\) | String | Comma separated list of directories to fetch archived jobs from. The history server will monitor these directories for archived jobs. You can configure the JobManager to archive jobs to a directory via \`jobmanager.archive.fs.dir\`. |
| **historyserver.archive.fs.refresh-interval** | 10000 | Long | Interval in milliseconds for refreshing the archived job directories. |
| **historyserver.web.address** | \(none\) | String | Address of the HistoryServer's web interface. |
| **historyserver.web.port** | 8082 | Integer | Port of the HistoryServers's web interface. |
| **historyserver.web.refresh-interval** | 10000 | Long | The refresh interval for the HistoryServer web-frontend in milliseconds. |
| **historyserver.web.ssl.enabled** | false | Boolean | Enable HTTPs access to the HistoryServer web frontend. This is applicable only when the global SSL flag security.ssl.enabled is set to true. |
| **historyserver.web.tmpdir** | \(none\) | String | This configuration parameter allows defining the Flink web directory to be used by the history server web interface. The web interface will copy its static files into the directory. |

## 实验性 <a id="experimental"></a>

 _Flink中的实验功能选项。_

### 可查询状态

 _可查询状态_是一项实验性功能，可让_你_访问Flink的内部状态，如键/值存储。有关详细信息，请参见可查询[状态文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/stream/state/queryable_state.html)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **queryable-state.client.network-threads** | 0 | Integer | Number of network \(Netty's event loop\) Threads for queryable state client. |
| **queryable-state.enable** | false | Boolean | Option whether the queryable state proxy and server should be enabled where possible and configurable. |
| **queryable-state.proxy.network-threads** | 0 | Integer | Number of network \(Netty's event loop\) Threads for queryable state proxy. |
| **queryable-state.proxy.ports** | "9069" | String | The port range of the queryable state proxy. The specified range can be a single port: "9123", a range of ports: "50100-50200", or a list of ranges and ports: "50100-50200,50300-50400,51234". |
| **queryable-state.proxy.query-threads** | 0 | Integer | Number of query Threads for queryable state proxy. Uses the number of slots if set to 0. |
| **queryable-state.server.network-threads** | 0 | Integer | Number of network \(Netty's event loop\) Threads for queryable state server. |
| **queryable-state.server.ports** | "9067" | String | The port range of the queryable state server. The specified range can be a single port: "9123", a range of ports: "50100-50200", or a list of ranges and ports: "50100-50200,50300-50400,51234". |
| **queryable-state.server.query-threads** | 0 | Integer | Number of query Threads for queryable state server. Uses the number of slots if set to 0.配置项默认值 |

## 调试&专家调优

下面的选项是针对专业用户和用于修复/调试问题的。大多数情况下不需要配置这些选项。

### 类加载

Flink为加载到会话集群的作业动态加载代码。此外，Flink尝试从应用程序隐藏类路径中的许多依赖项。这有助于减少应用程序代码和类路径中的依赖关系之间的依赖关系冲突。

有关详细信息，请参阅[调试类加载文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/debugging_classloading.html)。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **classloader.parent-first-patterns.additional** | \(none\) | String | A \(semicolon-separated\) list of patterns that specifies which classes should always be resolved through the parent ClassLoader first. A pattern is a simple prefix that is checked against the fully qualified class name. These patterns are appended to "classloader.parent-first-patterns.default". |
| **classloader.parent-first-patterns.default** | "java.;scala.;org.apache.flink.;com.esotericsoftware.kryo;org.apache.hadoop.;javax.annotation.;org.slf4j;org.apache.log4j;org.apache.logging;org.apache.commons.logging;ch.qos.logback;org.xml;javax.xml;org.apache.xerces;org.w3c" | String | A \(semicolon-separated\) list of patterns that specifies which classes should always be resolved through the parent ClassLoader first. A pattern is a simple prefix that is checked against the fully qualified class name. This setting should generally not be modified. To add another pattern we recommend to use "classloader.parent-first-patterns.additional" instead. |
| **classloader.resolve-order** | "child-first" | String | Defines the class resolution strategy when loading classes from user code, meaning whether to first check the user code jar \("child-first"\) or the application classpath \("parent-first"\). The default settings indicate to load classes first from the user code jar, which means that user code jars can include and load different dependencies than Flink uses \(transitively\). |

### 高级状态后端选项

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend.async** | true | Boolean | Option whether the state backend should use an asynchronous snapshot method where possible and configurable. Some state backends may not support asynchronous snapshots, or only support asynchronous snapshots, and ignore this option. |
| **state.backend.fs.memory-threshold** | 1024 | Integer | The minimum size of state data files. All state chunks smaller than that are stored inline in the root checkpoint metadata file. |
| **state.backend.fs.write-buffer-size** | 4096 | Integer | The default size of the write buffer for the checkpoint streams that write to file systems. The actual write buffer size is determined to be the maximum of the value of this option and option 'state.backend.fs.memory-threshold'. |

### 高级RocksDB状态后端选项

调整RocksDB和RocksDB检查点的高级选项。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend.rocksdb.checkpoint.transfer.thread.num** | 1 | Integer | The number of threads \(per stateful operator\) used to transfer \(download and upload\) files in RocksDBStateBackend. |
| **state.backend.rocksdb.localdir** | \(none\) | String | The local directory \(on the TaskManager\) where RocksDB puts its files. |
| **state.backend.rocksdb.options-factory** | "org.apache.flink.contrib.streaming.state.DefaultConfigurableOptionsFactory" | String | The options factory class for RocksDB to create DBOptions and ColumnFamilyOptions. The default options factory is org.apache.flink.contrib.streaming.state.DefaultConfigurableOptionsFactory, and it would read the configured options which provided in 'RocksDBConfigurableOptions'. |
| **state.backend.rocksdb.predefined-options** | "DEFAULT" | String | The predefined settings for RocksDB DBOptions and ColumnFamilyOptions by Flink community. Current supported candidate predefined-options are DEFAULT, SPINNING\_DISK\_OPTIMIZED, SPINNING\_DISK\_OPTIMIZED\_HIGH\_MEM or FLASH\_SSD\_OPTIMIZED. Note that user customized options and options from the OptionsFactory are applied on top of these predefined ones. |

 **RocksDB可配置选项**

 这些选项可对ColumnFamilies的行为和资源进行细粒度控制。随着`state.backend.rocksdb.memory.managed`和`state.backend.rocksdb.memory.fixed-per-slot`（Apache Flink 1.10）的引入，只需要使用此处的选项进行高级性能调整。也可以在应用程序中通过`RocksDBStateBackend.setOptions(PptionsFactory)`指定这些选项。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **state.backend.rocksdb.block.blocksize** | \(none\) | MemorySize | The approximate size \(in bytes\) of user data packed per block. RocksDB has default blocksize as '4KB'. |
| **state.backend.rocksdb.block.cache-size** | \(none\) | MemorySize | The amount of the cache for data blocks in RocksDB. RocksDB has default block-cache size as '8MB'. |
| **state.backend.rocksdb.compaction.level.max-size-level-base** | \(none\) | MemorySize | The upper-bound of the total size of level base files in bytes. RocksDB has default configuration as '256MB'. |
| **state.backend.rocksdb.compaction.level.target-file-size-base** | \(none\) | MemorySize | The target file size for compaction, which determines a level-1 file size. RocksDB has default configuration as '64MB'. |
| **state.backend.rocksdb.compaction.level.use-dynamic-size** | \(none\) | Boolean | If true, RocksDB will pick target size of each level dynamically. From an empty DB, RocksDB would make last level the base level, which means merging L0 data into the last level, until it exceeds max\_bytes\_for\_level\_base. And then repeat this process for second last level and so on. RocksDB has default configuration as 'false'. For more information, please refer to [RocksDB's doc.](https://github.com/facebook/rocksdb/wiki/Leveled-Compaction#level_compaction_dynamic_level_bytes-is-true) |
| **state.backend.rocksdb.compaction.style** | \(none\) | EnumPossible values: \[LEVEL, UNIVERSAL, FIFO\] | The specified compaction style for DB. Candidate compaction style is LEVEL, FIFO or UNIVERSAL, and RocksDB choose 'LEVEL' as default style. |
| **state.backend.rocksdb.files.open** | \(none\) | Integer | The maximum number of open files \(per TaskManager\) that can be used by the DB, '-1' means no limit. RocksDB has default configuration as '-1'. |
| **state.backend.rocksdb.thread.num** | \(none\) | Integer | The maximum number of concurrent background flush and compaction jobs \(per TaskManager\). RocksDB has default configuration as '1'. |
| **state.backend.rocksdb.write-batch-size** | 2 mb | MemorySize | The max size of the consumed memory for RocksDB batch write, will flush just based on item count if this config set to 0. |
| **state.backend.rocksdb.writebuffer.count** | \(none\) | Integer | Tne maximum number of write buffers that are built up in memory. RocksDB has default configuration as '2'. |
| **state.backend.rocksdb.writebuffer.number-to-merge** | \(none\) | Integer | The minimum number of write buffers that will be merged together before writing to storage. RocksDB has default configuration as '1'. |
| **state.backend.rocksdb.writebuffer.size** | \(none\) | MemorySize | The amount of data built up in memory \(backed by an unsorted log on disk\) before converting to a sorted on-disk files. RocksDB has default writebuffer size as '64MB'. |

### 高级容错配置项

这些参数可以帮助解决与故障转移相关的问题，以及错误地将组件视为失败的问题。

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
      <td style="text-align:left"><b>cluster.io-executor.pool-size</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The pool size of io executor for cluster entry-point and mini cluster.
        It&apos;s undefined by default and will use the number of CPU cores (hardware
        contexts) that the cluster entry-point JVM has access to.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>cluster.registration.error-delay</b>
      </td>
      <td style="text-align:left">10000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The pause made after an registration attempt caused an exception (other
        than timeout) in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>cluster.registration.initial-timeout</b>
      </td>
      <td style="text-align:left">100</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Initial registration timeout between cluster components in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>cluster.registration.max-timeout</b>
      </td>
      <td style="text-align:left">30000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Maximum registration timeout between cluster components in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>cluster.registration.refused-registration-delay</b>
      </td>
      <td style="text-align:left">30000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The pause made after the registration attempt was refused in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>cluster.services.shutdown-timeout</b>
      </td>
      <td style="text-align:left">30000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The shutdown timeout for cluster services like executors in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>heartbeat.interval</b>
      </td>
      <td style="text-align:left">10000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Time interval for requesting heartbeat from sender side.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>heartbeat.timeout</b>
      </td>
      <td style="text-align:left">50000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Timeout for requesting and receiving heartbeat for both sender and receiver
        sides.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.execution.failover-strategy</b>
      </td>
      <td style="text-align:left">region</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>This option specifies how the job computation recovers from task failures.
          Accepted values are:</p>
        <ul>
          <li>&apos;full&apos;: Restarts all tasks to recover the job.</li>
          <li>&apos;region&apos;: Restarts all tasks that could be affected by the task
            failure. More details can be found <a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#restart-pipelined-region-failover-strategy">here</a>.</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### 高级调度配置项

这些参数可以帮助微调特定情况下的调度。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **cluster.evenly-spread-out-slots** | false | Boolean | Enable the slot spread out allocation strategy. This strategy tries to spread out the slots evenly across all available `TaskExecutors`. |
| **slot.idle.timeout** | 50000 | Long | The timeout in milliseconds for a idle slot in Slot Pool. |
| **slot.request.timeout** | 300000 | Long | The timeout in milliseconds for requesting a slot from Slot Pool. |

### 高级的高可用配置

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **high-availability.jobmanager.port** | "0" | String | The port \(range\) used by the Flink Master for its RPC connections in highly-available setups. In highly-available setups, this value is used instead of 'jobmanager.rpc.port'.A value of '0' means that a random free port is chosen. TaskManagers discover this port through the high-availability services \(leader election\), so a random port or a port range works without requiring any additional means of service discovery. |

### 高级的高可用性ZooKeeper配置项



| Key | Default | Type | Description |
| :--- | :--- | :--- | :--- |
| **high-availability.zookeeper.client.acl** | "open" | String | Defines the ACL \(open\|creator\) to be configured on ZK node. The configuration value can be set to “creator” if the ZooKeeper server configuration has the “authProvider” property mapped to use SASLAuthenticationProvider and the cluster is configured to run in secure mode \(Kerberos\). |
| **high-availability.zookeeper.client.connection-timeout** | 15000 | Integer | Defines the connection timeout for ZooKeeper in ms. |
| **high-availability.zookeeper.client.max-retry-attempts** | 3 | Integer | Defines the number of connection retries before the client gives up. |
| **high-availability.zookeeper.client.retry-wait** | 5000 | Integer | Defines the pause between consecutive retries in ms. |
| **high-availability.zookeeper.client.session-timeout** | 60000 | Integer | Defines the session timeout for the ZooKeeper session in ms. |
| **high-availability.zookeeper.path.checkpoint-counter** | "/checkpoint-counter" | String | ZooKeeper root path \(ZNode\) for checkpoint counters. |
| **high-availability.zookeeper.path.checkpoints** | "/checkpoints" | String | ZooKeeper root path \(ZNode\) for completed checkpoints. |
| **high-availability.zookeeper.path.jobgraphs** | "/jobgraphs" | String | ZooKeeper root path \(ZNode\) for job graphs |
| **high-availability.zookeeper.path.latch** | "/leaderlatch" | String | Defines the znode of the leader latch which is used to elect the leader. |
| **high-availability.zookeeper.path.leader** | "/leader" | String | Defines the znode of the leader which contains the URL to the leader and the current leader session ID. |
| **high-availability.zookeeper.path.mesos-workers** | "/mesos-workers" | String | The ZooKeeper root path for persisting the Mesos worker information. |
| **high-availability.zookeeper.path.running-registry** | "/running\_job\_registry/" | String |  |

### 高级安全SSL配置项

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
      <td style="text-align:left"><b>security.ssl.internal.close-notify-flush-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The timeout (in ms) for flushing the `close_notify` that was triggered
        by closing a channel. If the `close_notify` was not flushed in the given
        timeout the channel will be closed forcibly. (-1 = use system default)</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.handshake-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The timeout (in ms) during SSL handshake. (-1 = use system default)</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.session-cache-size</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The size of the cache used for storing SSL session objects. According
        to https://github.com/netty/netty/issues/832, you should always set this
        to an appropriate number to not run into a bug with stalling IO threads
        during garbage collection. (-1 = use system default).</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.internal.session-timeout</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The timeout (in ms) for the cached SSL session objects. (-1 = use system
        default)</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>security.ssl.provider</b>
      </td>
      <td style="text-align:left">&quot;JDK&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>The SSL engine provider to use for the ssl transport:</p>
        <ul>
          <li><code>JDK</code>: default Java-based SSL engine</li>
          <li><code>OPENSSL</code>: openSSL-based SSL engine using system libraries</li>
        </ul>
        <p><code>OPENSSL</code> is based on <a href="http://netty.io/wiki/forked-tomcat-native.html#wiki-h2-4">netty-tcnative</a> and
          comes in two flavours:</p>
        <ul>
          <li>dynamically linked: This will use your system&apos;s openSSL libraries
            (if compatible) and requires <code>opt/flink-shaded-netty-tcnative-dynamic-*.jar</code> to
            be copied to <code>lib/</code>
          </li>
          <li>statically linked: Due to potential licensing issues with openSSL (see
            <a
            href="https://issues.apache.org/jira/browse/LEGAL-393">LEGAL-393</a>), we cannot ship pre-built libraries. However, you can build
              the required library yourself and put it into <code>lib/</code>:
              <br /><code>git clone https://github.com/apache/flink-shaded.git &amp;amp;&amp;amp; cd flink-shaded &amp;amp;&amp;amp; mvn clean package -Pinclude-netty-tcnative-static -pl flink-shaded-netty-tcnative-static</code>
          </li>
        </ul>
      </td>
    </tr>
  </tbody>
</table>

### REST端点和客户端的高级选项

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **rest.await-leader-timeout** | 30000 | Long | The time in ms that the client waits for the leader address, e.g., Dispatcher or WebMonitorEndpoint |
| **rest.client.max-content-length** | 104857600 | Integer | The maximum content length in bytes that the client will handle. |
| **rest.connection-timeout** | 15000 | Long | The maximum time in ms for the client to establish a TCP connection. |
| **rest.idleness-timeout** | 300000 | Long | The maximum time in ms for a connection to stay idle before failing. |
| **rest.retry.delay** | 3000 | Long | The time in ms that the client waits between retries \(See also \`rest.retry.max-attempts\`\). |
| **rest.retry.max-attempts** | 20 | Integer | The number of retries the client will attempt if a retryable operations fails. |
| **rest.server.max-content-length** | 104857600 | Integer | The maximum content length in bytes that the server will handle. |
| **rest.server.numThreads** | 4 | Integer | The number of threads for the asynchronous processing of requests. |
| **rest.server.thread-priority** | 5 | Integer | Thread priority of the REST server's executor for processing asynchronous requests. Lowering the thread priority will give Flink's main components more CPU time whereas increasing will allocate more time for the REST server's processing. |

### Flink Web UI的高级选项

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **web.access-control-allow-origin** | "\*" | String | Access-Control-Allow-Origin header for all responses from the web-frontend. |
| **web.backpressure.cleanup-interval** | 600000 | Integer | Time, in milliseconds, after which cached stats are cleaned up if not accessed. |
| **web.backpressure.delay-between-samples** | 50 | Integer | Delay between samples to determine back pressure in milliseconds. |
| **web.backpressure.num-samples** | 100 | Integer | Number of samples to take to determine back pressure. |
| **web.backpressure.refresh-interval** | 60000 | Integer | Time, in milliseconds, after which available stats are deprecated and need to be refreshed \(by resampling\). |
| **web.checkpoints.history** | 10 | Integer | Number of checkpoints to remember for recent history. |
| **web.history** | 5 | Integer | Number of archived jobs for the JobManager. |
| **web.log.path** | \(none\) | String | Path to the log file \(may be in /log for standalone but under log directory when using YARN\). |
| **web.refresh-interval** | 3000 | Long | Refresh interval for the web-frontend in milliseconds. |
| **web.submit.enable** | true | Boolean | Flag indicating whether jobs can be uploaded and run from the web-frontend. |
| **web.timeout** | 600000 | Long | Timeout for asynchronous operations by the web monitor in milliseconds. |
| **web.tmpdir** | System.getProperty\("java.io.tmpdir"\) | String | Flink web directory which is used by the webmonitor. |
| **web.upload.dir** | \(none\) | String | Directory for uploading the job jars. If not specified a dynamic directory will be used under the directory specified by JOB\_MANAGER\_WEB\_TMPDIR\_KEY. |

### 完整的Flink Master配置项

 **Master / JobManager**

<table>
  <thead>
    <tr>
      <th style="text-align:left"><b>&#x914D;&#x7F6E;&#x9879;</b>
      </th>
      <th style="text-align:left">&#x9ED8;&#x8BA4;&#x503C;</th>
      <th style="text-align:left">&#x7C7B;&#x578B;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>jobmanager.archive.fs.dir</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">Dictionary for JobManager to store the archives of completed jobs.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.execution.attempts-history-size</b>
      </td>
      <td style="text-align:left">16</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The maximum number of prior execution attempts kept in history.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.execution.failover-strategy</b>
      </td>
      <td style="text-align:left">region</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>This option specifies how the job computation recovers from task failures.
          Accepted values are:</p>
        <ul>
          <li>&apos;full&apos;: Restarts all tasks to recover the job.</li>
          <li>&apos;region&apos;: Restarts all tasks that could be affected by the task
            failure. More details can be found <a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/task_failure_recovery.html#restart-pipelined-region-failover-strategy">here</a>.</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.heap.size</b>
      </td>
      <td style="text-align:left">&quot;1024m&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">JVM heap size for the JobManager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.rpc.address</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The config parameter defining the network address to connect to for communication
        with the job manager. This value is only interpreted in setups where a
        single JobManager with static name or address exists (simple standalone
        setups, or container setups with dynamic service name resolution). It is
        not used in many high-availability setups, when a leader-election service
        (like ZooKeeper) is used to elect and discover the JobManager leader from
        potentially multiple standby JobManagers.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobmanager.rpc.port</b>
      </td>
      <td style="text-align:left">6123</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The config parameter defining the network port to connect to for communication
        with the job manager. Like jobmanager.rpc.address, this value is only interpreted
        in setups where a single JobManager with static name/address and port exists
        (simple standalone setups, or container setups with dynamic service name
        resolution). This config option is not used in many high-availability setups,
        when a leader-election service (like ZooKeeper) is used to elect and discover
        the JobManager leader from potentially multiple standby JobManagers.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobstore.cache-size</b>
      </td>
      <td style="text-align:left">52428800</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The job store cache size in bytes which is used to keep completed jobs
        in memory.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobstore.expiration-time</b>
      </td>
      <td style="text-align:left">3600</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The time in seconds after which a completed job expires and is purged
        from the job store.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>jobstore.max-capacity</b>
      </td>
      <td style="text-align:left">2147483647</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The max number of completed jobs that can be kept in the job store.</td>
    </tr>
  </tbody>
</table>

 **Blob Server**

Blob **Server**是Flink Master / JobManager中的组件。它用于分发太大而无法附加到RPC消息且受益于缓存的对象（例如Jar文件或大型序列化代码对象）。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **blob.client.connect.timeout** | 0 | Integer | The connection timeout in milliseconds for the blob client. |
| **blob.client.socket.timeout** | 300000 | Integer | The socket timeout in milliseconds for the blob client. |
| **blob.fetch.backlog** | 1000 | Integer | The config parameter defining the backlog of BLOB fetches on the JobManager. |
| **blob.fetch.num-concurrent** | 50 | Integer | The config parameter defining the maximum number of concurrent BLOB fetches that the JobManager serves. |
| **blob.fetch.retries** | 5 | Integer | The config parameter defining number of retires for failed BLOB fetches. |
| **blob.offload.minsize** | 1048576 | Integer | The minimum size for messages to be offloaded to the BlobServer. |
| **blob.server.port** | "0" | String | The config parameter defining the server port of the blob service. |
| **blob.service.cleanup.interval** | 3600 | Long | Cleanup interval of the blob caches at the task managers \(in seconds\). |
| **blob.service.ssl.enabled** | true | Boolean | Flag to override ssl support for the blob service transport. |
| **blob.storage.directory** | \(none\) | String | The config parameter defining the storage directory to be used by the blob server. |

 **ResourceManager**

以下配置项控制基本的资源管理器行为，而与所使用的资源编排管理框架（YARN，Mesos等）无关。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **containerized.heap-cutoff-min** | 600 | Integer | Minimum amount of heap memory to remove in Job Master containers, as a safety margin. |
| **containerized.heap-cutoff-ratio** | 0.25 | Float | Percentage of heap space to remove from Job Master containers \(YARN / Mesos / Kubernetes\), to compensate for other JVM memory usage. |
| **resourcemanager.job.timeout** | "5 minutes" | String | Timeout for jobs which don't have a job manager as leader assigned. |
| **resourcemanager.rpc.port** | 0 | Integer | Defines the network port to connect to for communication with the resource manager. By default, the port of the JobManager, because the same ActorSystem is used. Its not possible to use this configuration key to define port ranges. |
| **resourcemanager.standalone.start-up-time** | -1 | Long | Time in milliseconds of the start-up period of a standalone cluster. During this time, resource manager of the standalone cluster expects new task executors to be registered, and will not fail slot requests that can not be satisfied by any current registered slots. After this time, it will fail pending and new coming requests immediately that can not be satisfied by registered slots. If not set, 'slotmanager.request-timeout' will be used by default. |
| **resourcemanager.taskmanager-timeout** | 30000 | Long | The timeout for an idle task manager to be released. |

### 完整的TaskManager配置项

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
      <td style="text-align:left"><b>task.cancellation.interval</b>
      </td>
      <td style="text-align:left">30000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Time interval between two successive task cancellation attempts in milliseconds.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>task.cancellation.timeout</b>
      </td>
      <td style="text-align:left">180000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Timeout in milliseconds after which a task cancellation times out and
        leads to a fatal TaskManager error. A value of 0 deactivates the watch
        dog.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>task.cancellation.timers.timeout</b>
      </td>
      <td style="text-align:left">7500</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">Time we wait for the timers in milliseconds to finish all pending timer
        threads when the stream task is cancelled.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.data.port</b>
      </td>
      <td style="text-align:left">0</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The task manager&#x2019;s port used for data exchange operations.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.data.ssl.enabled</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Enable SSL support for the taskmanager data transport. This is applicable
        only when the global flag for internal SSL (security.ssl.internal.enabled)
        is set to true</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.debug.memory.log</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Flag indicating whether to start a thread, which repeatedly logs the memory
        usage of the JVM.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.debug.memory.log-interval</b>
      </td>
      <td style="text-align:left">5000</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">The interval (in ms) for the log thread to log the current memory usage.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.host</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The address of the network interface that the TaskManager binds to. This
        option can be used to define explicitly a binding address. Because different
        TaskManagers need different values for this option, usually it is specified
        in an additional non-shared TaskManager-specific config file.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.jvm-exit-on-oom</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Whether to kill the TaskManager when the task thread throws an OutOfMemoryError.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.memory.segment-size</b>
      </td>
      <td style="text-align:left">32 kb</td>
      <td style="text-align:left">MemorySize</td>
      <td style="text-align:left">Size of memory buffers used by the network stack and the memory manager.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.network.bind-policy</b>
      </td>
      <td style="text-align:left">&quot;ip&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">
        <p>The automatic address binding policy used by the TaskManager if &quot;taskmanager.host&quot;
          is not set. The value should be one of the following:</p>
        <ul>
          <li>&quot;name&quot; - uses hostname as binding address</li>
          <li>&quot;ip&quot; - uses host&apos;s ip address as binding address</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.numberOfTaskSlots</b>
      </td>
      <td style="text-align:left">1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The number of parallel operator or user function instances that a single
        TaskManager can run. If this value is larger than 1, a single TaskManager
        takes multiple instances of a function or operator. That way, the TaskManager
        can utilize multiple CPU cores, but at the same time, the available memory
        is divided between the different operator or function instances. This value
        is typically proportional to the number of physical CPU cores that the
        TaskManager&apos;s machine has (e.g., equal to the number of cores, or
        half the number of cores).</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.registration.initial-backoff</b>
      </td>
      <td style="text-align:left">500 ms</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">The initial registration backoff between two consecutive registration
        attempts. The backoff is doubled for each new registration attempt until
        it reaches the maximum registration backoff.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.registration.max-backoff</b>
      </td>
      <td style="text-align:left">30 s</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">The maximum registration backoff between two consecutive registration
        attempts. The max registration backoff requires a time unit specifier (ms/s/min/h/d).</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.registration.refused-backoff</b>
      </td>
      <td style="text-align:left">10 s</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">The backoff after a registration has been refused by the job manager before
        retrying to connect.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.registration.timeout</b>
      </td>
      <td style="text-align:left">5 min</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">Defines the timeout for the TaskManager registration. If the duration
        is exceeded without a successful registration, then the TaskManager terminates.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>taskmanager.rpc.port</b>
      </td>
      <td style="text-align:left">&quot;0&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">The task manager&#x2019;s IPC port. Accepts a list of ports (&#x201C;50100,50101&#x201D;),
        ranges (&#x201C;50100-50200&#x201D;) or a combination of both. It is recommended
        to set a range of ports to avoid collisions when multiple TaskManagers
        are running on the same machine.</td>
    </tr>
  </tbody>
</table>

 **数据传输网络堆栈**

这些选项用于处理taskmanager之间的流和批处理数据交换的网络堆栈。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **taskmanager.network.blocking-shuffle.compression.enabled** | false | Boolean | Boolean flag indicating whether the shuffle data will be compressed for blocking shuffle mode. Note that data is compressed per buffer and compression can incur extra CPU overhead, so it is more effective for IO bounded scenario when data compression ratio is high. Currently, shuffle data compression is an experimental feature and the config option can be changed in the future. |
| **taskmanager.network.blocking-shuffle.type** | "file" | String | The blocking shuffle type, either "mmap" or "file". The "auto" means selecting the property type automatically based on system memory architecture \(64 bit for mmap and 32 bit for file\). Note that the memory usage of mmap is not accounted by configured memory limits, but some resource frameworks like yarn would track this memory usage and kill the container once memory exceeding some threshold. Also note that this option is experimental and might be changed future. |
| **taskmanager.network.detailed-metrics** | false | Boolean | Boolean flag to enable/disable more detailed metrics about inbound/outbound network queue lengths. |
| **taskmanager.network.memory.buffers-per-channel** | 2 | Integer | Maximum number of network buffers to use for each outgoing/incoming channel \(subpartition/input channel\).In credit-based flow control mode, this indicates how many credits are exclusive in each input channel. It should be configured at least 2 for good performance. 1 buffer is for receiving in-flight data in the subpartition and 1 buffer is for parallel serialization. |
| **taskmanager.network.memory.floating-buffers-per-gate** | 8 | Integer | Number of extra network buffers to use for each outgoing/incoming gate \(result partition/input gate\). In credit-based flow control mode, this indicates how many floating credits are shared among all the input channels. The floating buffers are distributed based on backlog \(real-time output buffers in the subpartition\) feedback, and can help relieve back-pressure caused by unbalanced data distribution among the subpartitions. This value should be increased in case of higher round trip times between nodes and/or larger number of machines in the cluster. |
| **taskmanager.network.netty.client.connectTimeoutSec** | 120 | Integer | The Netty client connection timeout. |
| **taskmanager.network.netty.client.numThreads** | -1 | Integer | The number of Netty client threads. |
| **taskmanager.network.netty.num-arenas** | -1 | Integer | The number of Netty arenas. |
| **taskmanager.network.netty.sendReceiveBufferSize** | 0 | Integer | The Netty send and receive buffer size. This defaults to the system buffer size \(cat /proc/sys/net/ipv4/tcp\_\[rw\]mem\) and is 4 MiB in modern Linux. |
| **taskmanager.network.netty.server.backlog** | 0 | Integer | The netty server connection backlog. |
| **taskmanager.network.netty.server.numThreads** | -1 | Integer | The number of Netty server threads. |
| **taskmanager.network.netty.transport** | "nio" | String | The Netty transport type, either "nio" or "epoll" |
| **taskmanager.network.request-backoff.initial** | 100 | Integer | Minimum backoff in milliseconds for partition requests of input channels. |
| **taskmanager.network.request-backoff.max** | 10000 | Integer | Maximum backoff in milliseconds for partition requests of input channels. |

### RPC / Akka

Flink将Akka用于组件（JobManager / TaskManager / ResourceManager）之间的RPC。Flink不使用Akka进行数据传输。

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **akka.ask.timeout** | "10 s" | String | Timeout used for all futures and blocking Akka calls. If Flink fails due to timeouts then you should try to increase this value. Timeouts can be caused by slow machines or a congested network. The timeout value requires a time-unit specifier \(ms/s/min/h/d\). |
| **akka.client-socket-worker-pool.pool-size-factor** | 1.0 | Double | The pool size factor is used to determine thread pool size using the following formula: ceil\(available processors \* factor\). Resulting size is then bounded by the pool-size-min and pool-size-max values. |
| **akka.client-socket-worker-pool.pool-size-max** | 2 | Integer | Max number of threads to cap factor-based number to. |
| **akka.client-socket-worker-pool.pool-size-min** | 1 | Integer | Min number of threads to cap factor-based number to. |
| **akka.client.timeout** | "60 s" | String | Timeout for all blocking calls on the client side. |
| **akka.fork-join-executor.parallelism-factor** | 2.0 | Double | The parallelism factor is used to determine thread pool size using the following formula: ceil\(available processors \* factor\). Resulting size is then bounded by the parallelism-min and parallelism-max values. |
| **akka.fork-join-executor.parallelism-max** | 64 | Integer | Max number of threads to cap factor-based parallelism number to. |
| **akka.fork-join-executor.parallelism-min** | 8 | Integer | Min number of threads to cap factor-based parallelism number to. |
| **akka.framesize** | "10485760b" | String | Maximum size of messages which are sent between the JobManager and the TaskManagers. If Flink fails because messages exceed this limit, then you should increase it. The message size requires a size-unit specifier. |
| **akka.jvm-exit-on-fatal-error** | true | Boolean | Exit JVM on fatal Akka errors. |
| **akka.log.lifecycle.events** | false | Boolean | Turns on the Akka’s remote logging of events. Set this value to 'true' in case of debugging. |
| **akka.lookup.timeout** | "10 s" | String | Timeout used for the lookup of the JobManager. The timeout value has to contain a time-unit specifier \(ms/s/min/h/d\). |
| **akka.retry-gate-closed-for** | 50 | Long | Milliseconds a gate should be closed for after a remote connection was disconnected. |
| **akka.server-socket-worker-pool.pool-size-factor** | 1.0 | Double | The pool size factor is used to determine thread pool size using the following formula: ceil\(available processors \* factor\). Resulting size is then bounded by the pool-size-min and pool-size-max values. |
| **akka.server-socket-worker-pool.pool-size-max** | 2 | Integer | Max number of threads to cap factor-based number to. |
| **akka.server-socket-worker-pool.pool-size-min** | 1 | Integer | Min number of threads to cap factor-based number to. |
| **akka.ssl.enabled** | true | Boolean | Turns on SSL for Akka’s remote communication. This is applicable only when the global ssl flag security.ssl.enabled is set to true. |
| **akka.startup-timeout** | \(none\) | String | Timeout after which the startup of a remote component is considered being failed. |
| **akka.tcp.timeout** | "20 s" | String | Timeout for all outbound connections. If you should experience problems with connecting to a TaskManager due to a slow network, you should increase this value. |
| **akka.throughput** | 15 | Integer | Number of messages that are processed in a batch before returning the thread to the pool. Low values denote a fair scheduling whereas high values can increase the performance at the cost of unfairness. |
| **akka.transport.heartbeat.interval** | "1000 s" | String | Heartbeat interval for Akka’s transport failure detector. Since Flink uses TCP, the detector is not necessary. Therefore, the detector is disabled by setting the interval to a very high value. In case you should need the transport failure detector, set the interval to some reasonable value. The interval value requires a time-unit specifier \(ms/s/min/h/d\). |
| **akka.transport.heartbeat.pause** | "6000 s" | String | Acceptable heartbeat pause for Akka’s transport failure detector. Since Flink uses TCP, the detector is not necessary. Therefore, the detector is disabled by setting the pause to a very high value. In case you should need the transport failure detector, set the pause to some reasonable value. The pause value requires a time-unit specifier \(ms/s/min/h/d\). |
| **akka.transport.threshold** | 300.0 | Double | Threshold for the transport failure detector. Since Flink uses TCP, the detector is not necessary and, thus, the threshold is set to a high value. |

## JVM和日志记录配置项

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **env.hadoop.conf.dir** | \(none\) | String | Path to hadoop configuration directory. It is required to read HDFS and/or YARN configuration. You can also set it via environment variable. |
| **env.java.opts** | \(none\) | String | Java options to start the JVM of all Flink processes with. |
| **env.java.opts.client** | \(none\) | String | Java options to start the JVM of the Flink Client with. |
| **env.java.opts.historyserver** | \(none\) | String | Java options to start the JVM of the HistoryServer with. |
| **env.java.opts.jobmanager** | \(none\) | String | Java options to start the JVM of the JobManager with. |
| **env.java.opts.taskmanager** | \(none\) | String | Java options to start the JVM of the TaskManager with. |
| **env.log.dir** | \(none\) | String | Defines the directory where the Flink logs are saved. It has to be an absolute path. \(Defaults to the log directory under Flink’s home\) |
| **env.log.max** | 5 | Integer | The maximum number of old log files to keep. |
| **env.ssh.opts** | \(none\) | String | Additional command line options passed to SSH clients when starting or stopping JobManager, TaskManager, and Zookeeper services \(start-cluster.sh, stop-cluster.sh, start-zookeeper-quorum.sh, stop-zookeeper-quorum.sh\). |
| **env.yarn.conf.dir** | \(none\) | String | Path to yarn configuration directory. It is required to run flink on YARN. You can also set it via environment variable. |

## 转发环境变量

你可以配置环境变量，以在Yarn / Mesos上启动的Flink Master和TaskManager进程上进行设置。

* `containerized.master.env.`：前缀，用于将自定义环境变量传递给Flink的Master进程。例如，要将LD\_LIBRARY\_PATH作为env变量传递给Master服务器，请在flink-conf.yaml中设置containerized.master.env.LD\_LIBRARY\_PATH：“ / usr / lib / native”。
* `containerized.taskmanager.env.`：与上述类似，此配置前缀允许为工作程序（TaskManagers）设置自定义环境变量。

## 不推荐使用的配置项

这些选项与Flink中不再被积极开发的部分相关。这些选项可能会在将来的版本中删除。

 **DataSet API优化器**

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **compiler.delimited-informat.max-line-samples** | 10 | Integer | he maximum number of line samples taken by the compiler for delimited inputs. The samples are used to estimate the number of records. This value can be overridden for a specific input with the input format’s parameters. |
| **compiler.delimited-informat.max-sample-len** | 2097152 | Integer | The maximal length of a line sample that the compiler takes for delimited inputs. If the length of a single sample exceeds this value \(possible because of misconfiguration of the parser\), the sampling aborts. This value can be overridden for a specific input with the input format’s parameters. |
| **compiler.delimited-informat.min-line-samples** | 2 | Integer | The minimum number of line samples taken by the compiler for delimited inputs. The samples are used to estimate the number of records. This value can be overridden for a specific input with the input format’s parameters |

 **DataSet API运行时算法**

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **taskmanager.runtime.hashjoin-bloom-filters** | false | Boolean | Flag to activate/deactivate bloom filters in the hybrid hash join implementation. In cases where the hash join needs to spill to disk \(datasets larger than the reserved fraction of memory\), these bloom filters can greatly reduce the number of spilled records, at the cost some CPU cycles. |
| **taskmanager.runtime.max-fan** | 128 | Integer | The maximal fan-in for external merge joins and fan-out for spilling hash tables. Limits the number of file handles per operator, but may cause intermediate merging/partitioning, if set too small. |
| **taskmanager.runtime.sort-spilling-threshold** | 0.8 | Float | A sort operation starts spilling when this fraction of its memory budget is full. |

 **DataSet File Sinks**

| 配置项 | 默认值 | 类型 | 描述  |
| :--- | :--- | :--- | :--- |


| **fs.output.always-create-directory** | false | Boolean | File writers running with a parallelism larger than one create a directory for the output file path and put the different result files \(one per parallel writer task\) into that directory. If this option is set to "true", writers with a parallelism of 1 will also create a directory and place a single result file into it. If the option is set to "false", the writer will directly create the file directly at the output path, without creating a containing directory. |
| :--- | :--- | :--- | :--- |


| **fs.overwrite-files** | false | Boolean | Specifies whether file output writers should overwrite existing files by default. Set to "true" to overwrite by default,"false" otherwise. |
| :--- | :--- | :--- | :--- |


## 备份

**Execution**

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **execution.attached** | false | Boolean | Specifies if the pipeline is submitted in attached or detached mode. |
| **execution.shutdown-on-attached-exit** | false | Boolean | If the job is submitted in attached mode, perform a best-effort cluster shutdown when the CLI is terminated abruptly, e.g., in response to a user interrupt, such as typing Ctrl + C. |
| **execution.target** | \(none\) | String | The deployment target for the execution, e.g. "local" for local execution. |

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **execution.savepoint.ignore-unclaimed-state** | false | Boolean | Allow to skip savepoint state that cannot be restored. Allow this if you removed an operator from your pipeline after the savepoint was triggered. |
| **execution.savepoint.path** | \(none\) | String | Path to a savepoint to restore the job from \(for example hdfs:///flink/savepoint-1537\). |

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
      <td style="text-align:left"><b>execution.buffer-timeout</b>
      </td>
      <td style="text-align:left">100 ms</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">
        <p>The maximum time frequency (milliseconds) for the flushing of the output
          buffers. By default the output buffers flush frequently to provide low
          latency and to aid smooth developer experience. Setting the parameter can
          result in three logical modes:</p>
        <ul>
          <li>A positive value triggers flushing periodically by that interval</li>
          <li>0 triggers flushing after every record thus minimizing latency</li>
          <li>-1 ms triggers flushing only when the output buffer is full thus maximizing
            throughput</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>execution.checkpointing.snapshot-compression</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Tells if we should use compression for the state snapshot data or not</td>
    </tr>
  </tbody>
</table>

**Pipeline**

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
      <td style="text-align:left"><b>pipeline.auto-generate-uids</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">When auto-generated UIDs are disabled, users are forced to manually specify
        UIDs on DataStream applications.
        <br />
        <br />It is highly recommended that users specify UIDs before deploying to production
        since they are used to match state in savepoints to operators in a job.
        Because auto-generated ID&apos;s are likely to change when modifying a
        job, specifying custom IDs allow an application to evolve over time without
        discarding state.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.auto-type-registration</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Controls whether Flink is automatically registering all types in the user
        programs with Kryo.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.auto-watermark-interval</b>
      </td>
      <td style="text-align:left">0 ms</td>
      <td style="text-align:left">Duration</td>
      <td style="text-align:left">The interval of the automatic watermark emission. Watermarks are used
        throughout the streaming system to keep track of the progress of time.
        They are used, for example, for time based windowing.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.cached-files</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">Files to be registered at the distributed cache under the given name.
        The files will be accessible from any user-defined function in the (distributed)
        runtime under a local path. Files may be local files (which will be distributed
        via BlobServer), or files in a distributed file system. The runtime will
        copy the files temporarily to a local cache, if needed.
        <br />
        <br />Example:
        <br /><code>name:file1,path:</code>file:///tmp/file1<code>;name:file2,path:</code>hdfs:///tmp/file2``</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.classpaths</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">A semicolon-separated list of the classpaths to package with the job jars
        to be sent to the cluster. These have to be valid URLs.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.closure-cleaner-level</b>
      </td>
      <td style="text-align:left">RECURSIVE</td>
      <td style="text-align:left">EnumPossible values: [NONE, TOP_LEVEL, RECURSIVE]</td>
      <td style="text-align:left">
        <p>Configures the mode in which the closure cleaner works</p>
        <ul>
          <li><code>NONE</code> - disables the closure cleaner completely</li>
          <li><code>TOP_LEVEL</code> - cleans only the top-level class without recursing
            into fields</li>
          <li><code>RECURSIVE</code> - cleans all the fields recursively</li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.default-kryo-serializers</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">Semicolon separated list of pairs of class names and Kryo serializers
        class names to be used as Kryo default serializers
        <br />
        <br />Example:
        <br /><code>class:org.example.ExampleClass,serializer:org.example.ExampleSerializer1; class:org.example.ExampleClass2,serializer:org.example.ExampleSerializer2</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.force-avro</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Forces Flink to use the Apache Avro serializer for POJOs.
        <br />
        <br />Important: Make sure to include the <code>flink-avro</code> module.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.force-kryo</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">If enabled, forces TypeExtractor to use Kryo serializer for POJOS even
        though we could analyze as POJO. In some cases this might be preferable.
        For example, when using interfaces with subclasses that cannot be analyzed
        as POJO.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.generic-types</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">If the use of generic types is disabled, Flink will throw an <code>UnsupportedOperationException</code> whenever
        it encounters a data type that would go through Kryo for serialization.
        <br
        />
        <br />Disabling generic types can be helpful to eagerly find and eliminate the
        use of types that would go through Kryo serialization during runtime. Rather
        than checking types individually, using this option will throw exceptions
        eagerly in the places where generic types are used.
        <br />
        <br />We recommend to use this option only during development and pre-production
        phases, not during actual production use. The application program and/or
        the input data may be such that new, previously unseen, types occur at
        some point. In that case, setting this option would cause the program to
        fail.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.global-job-parameters</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">Map</td>
      <td style="text-align:left">Register a custom, serializable user configuration object. The configuration
        can be accessed in operators</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.jars</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">A semicolon-separated list of the jars to package with the job jars to
        be sent to the cluster. These have to be valid paths.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.max-parallelism</b>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">The program-wide maximum parallelism used for operators which haven&apos;t
        specified a maximum parallelism. The maximum parallelism specifies the
        upper limit for dynamic scaling and the number of key groups used for partitioned
        state.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.object-reuse</b>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">When enabled objects that Flink internally uses for deserialization and
        passing data to user-code functions will be reused. Keep in mind that this
        can lead to bugs when the user-code function of an operation is not aware
        of this behaviour.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.operator-chaining</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">Operator chaining allows non-shuffle operations to be co-located in the
        same thread fully avoiding serialization and de-serialization.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.registered-kryo-types</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">Semicolon separated list of types to be registered with the serialization
        stack. If the type is eventually serialized as a POJO, then the type is
        registered with the POJO serializer. If the type ends up being serialized
        with Kryo, then it will be registered at Kryo to make sure that only tags
        are written.</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>pipeline.registered-pojo-types</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">List&lt;String&gt;</td>
      <td style="text-align:left">Semicolon separated list of types to be registered with the serialization
        stack. If the type is eventually serialized as a POJO, then the type is
        registered with the POJO serializer. If the type ends up being serialized
        with Kryo, then it will be registered at Kryo to make sure that only tags
        are written.</td>
    </tr>
  </tbody>
</table>

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **pipeline.time-characteristic** | ProcessingTime | EnumPossible values: \[ProcessingTime, IngestionTime, EventTime\] | The time characteristic for all created streams, e.g., processingtime, event time, or ingestion time.  If you set the characteristic to IngestionTime or EventTime this will set a default watermark update interval of 200 ms. If this is not applicable for your application you should change it using `pipeline.auto-watermark-interval`. |

**Checkpointing**

| 配置项 | 默认值 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **execution.checkpointing.externalized-checkpoint-retention** | \(none\) | EnumPossible values: \[DELETE\_ON\_CANCELLATION, RETAIN\_ON\_CANCELLATION\] | Externalized checkpoints write their meta data out to persistent storage and are not automatically cleaned up when the owning job fails or is suspended \(terminating with job status `JobStatus#FAILED` or `JobStatus#SUSPENDED`. In this case, you have to manually clean up the checkpoint state, both the meta data and actual program state.  The mode defines how an externalized checkpoint should be cleaned up on job cancellation. If you choose to retain externalized checkpoints on cancellation you have to handle checkpoint clean up manually when you cancel the job as well \(terminating with job status `JobStatus#CANCELED`\).  The target directory for externalized checkpoints is configured via `state.checkpoints.dir`. |
| **execution.checkpointing.interval** | \(none\) | Duration | Gets the interval in which checkpoints are periodically scheduled.  This setting defines the base interval. Checkpoint triggering may be delayed by the settings `execution.checkpointing.max-concurrent-checkpoints` and `execution.checkpointing.min-pause` |
| **execution.checkpointing.max-concurrent-checkpoints** | 1 | Integer | The maximum number of checkpoint attempts that may be in progress at the same time. If this value is n, then no checkpoints will be triggered while n checkpoint attempts are currently in flight. For the next checkpoint to be triggered, one checkpoint attempt would need to finish or expire. |
| **execution.checkpointing.min-pause** | 0 ms | Duration | The minimal pause between checkpointing attempts. This setting defines how soon thecheckpoint coordinator may trigger another checkpoint after it becomes possible to triggeranother checkpoint with respect to the maximum number of concurrent checkpoints\(see `execution.checkpointing.max-concurrent-checkpoints`\).  If the maximum number of concurrent checkpoints is set to one, this setting makes effectively sure that a minimum amount of time passes where no checkpoint is in progress at all. |
| **execution.checkpointing.mode** | EXACTLY\_ONCE | EnumPossible values: \[EXACTLY\_ONCE, AT\_LEAST\_ONCE\] | The checkpointing mode \(exactly-once vs. at-least-once\). |
| **execution.checkpointing.prefer-checkpoint-for-recovery** | false | Boolean | If enabled, a job recovery should fallback to checkpoint when there is a more recent savepoint. |
| **execution.checkpointing.timeout** | 10 min | Duration | The maximum time that a checkpoint may take before being discarded. |
| **execution.checkpointing.tolerable-failed-checkpoints** | \(none\) | Integer | The tolerable checkpoint failure number. If set to 0, that meanswe do not tolerance any checkpoint failure. |

