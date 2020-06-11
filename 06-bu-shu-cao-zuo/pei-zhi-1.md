# 配置

类型p配置项描述描述配置项所有配置都在`conf/flink-conf.yaml`中完成，该配置应该是YAML键值对的平面集合，格式为`key:value`。

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

| 配置项 | 默认值 | 类型类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **classloader.parent-first-patterns.additional** | \(none\) | String | A \(semicolon-separated\) list of patterns that specifies which classes should always be resolved through the parent ClassLoader first. A pattern is a simple prefix that is checked against the fully qualified class name. These patterns are appended to "classloader.parent-first-patterns.default". |
| **classloader.parent-first-patterns.default** | "java.;scala.;org.apache.flink.;com.esotericsoftware.kryo;org.apache.hadoop.;javax.annotation.;org.slf4j;org.apache.log4j;org.apache.logging;org.apache.commons.logging;ch.qos.logback;org.xml;javax.xml;org.apache.xerces;org.w3c" | String | A \(semicolon-separated\) list of patterns that specifies which classes should always be resolved through the parent ClassLoader first. A pattern is a simple prefix that is checked against the fully qualified class name. This setting should generally not be modified. To add another pattern we recommend to use "classloader.parent-first-patterns.additional" instead. |
| **classloader.resolve-order** | "child-first" | String | Defines the class resolution strategy when loading classes from user code, meaning whether to first check the user code jar \("child-first"\) or the application classpath \("parent-first"\). The default settings indicate to load classes first from the user code jar, which means that user code jars can include and load different dependencies than Flink uses \(transitively\). |

## JVM和日志记录选项

## 转发环境变量

## 不推荐使用的配置项

## 备份



