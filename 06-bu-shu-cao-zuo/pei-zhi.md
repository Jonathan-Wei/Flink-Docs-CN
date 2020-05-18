# 配置

**对于单节点设置，Flink已准备好开箱即用，您无需更改默认配置即可开始使用。**

开箱即用的配置将使用默认Java安装。可以手动设置环境变量`JAVA_HOME`或配置项`env.java.home`中`conf/flink-conf.yaml`，如果想手动覆盖Java运行时使用。

此页面列出了设置执行良好（分布式）安装通常所需的最常用选项。此外，此处还列出了所有可用配置参数的完整列表。

所有配置都已完成`conf/flink-conf.yaml`，预计将是具有格式的[YAML键值对](http://www.yaml.org/spec/1.2/spec.html)的扁平集合`key: value`。

系统和运行脚本在启动时解析配置。对配置文件的更改需要重新启动Flink JobManager和TaskManagers。

TaskManagers的配置文件可能不同，Flink不假设集群中有统一的机器。

## 常见选项

| Key | Default | Description |
| :--- | :--- | :--- |
| **jobmanager.heap.size** | "1024m" | JobManager的JVM堆大小。 |
| **taskmanager.heap.size** | "1024m" | TaskManagers的JVM堆大小，它是系统的并行工作者。在YARN设置中，此值自动配置为TaskManager的YARN容器的大小，减去一定的容差值。 |
| **parallelism.default** | 1 |  |
| **taskmanager.numberOfTaskSlots** | 1 | 单个TaskManager可以运行的并行操作符或用户函数实例的数量。 如果此值大于1，则单个TaskManager将获取函数或操作符的多个实例。 这样，TaskManager可以使用多个CPU内核，但同时，可用内存在不同的操作员或功能实例之间划分。 该值通常与TaskManager的机器具有的物理CPU核心数成比例（例如，等于核的数量，或核的数量的一半）。 |
| **state.backend** | \(none\) | 状态后端用于存储和检查点状态。 |
| **state.checkpoints.dir** | \(none\) | 用于在Flink支持的文件系统中存储检查点的数据文件和元数据的默认目录。 必须可以从所有参与的进程/节点（即所有TaskManagers和JobManagers）访问存储路径。 |
| **state.savepoints.dir** | \(none\) | 保存点的默认目录。 由将后端写入文件系统的状态后端（MemoryStateBackend，FsStateBackend，RocksDBStateBackend）使用。 |
| **high-availability** | "NONE" | 定义用于群集执行的高可用性模式。 要启用高可用性，请将此模式设置为“ZOOKEEPER”或指定工厂类的FQN。 |
| **high-availability.storageDir** | \(none\) | 文件系统路径（URI）Flink在高可用性设置中保留元数据。 |
| **security.ssl.internal.enabled** | false | 打开SSL以进行内部网络通信。 可选地，特定组件可以通过它们自己的设置（rpc，数据传输，REST等）覆盖它。 |
| **security.ssl.rest.enabled** | false | 通过REST端点打开SSL以进行外部通信。 |

## 完整参考

### HDFS

{% hint style="warning" %}
**注意：**不推荐使用这些参数，建议使用环境变量HADOOP\_CONF\_DIR配置Hadoop路径。
{% endhint %}

* `fs.hdfs.hadoopconf`：Hadoop文件系统（HDFS）配置**目录**的绝对路径（可选值）。指定此值允许程序使用短URI引用HDFS文件（`hdfs:///path/to/files`不包括文件URI中NameNode的地址和端口）。如果没有此选项，可以访问HDFS文件，但需要完全限定的URI `hdfs://address:port/path/to/files`。此选项还会导致文件编写者获取HDFS的块大小和复制因子的默认值。Flink将在指定目录中查找“core-site.xml”和“hdfs-site.xml”文件。
* `fs.hdfs.hdfsdefault`：Hadoop自己的配置文件“hdfs-default.xml”的绝对路径（DEFAULT：null）。
* `fs.hdfs.hdfssite`：Hadoop自己的配置文件“hdfs-site.xml”的绝对路径（DEFAULT：null）。

### Core



<table>
  <thead>
    <tr>
      <th style="text-align:left">Key</th>
      <th style="text-align:left">Default</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>classloader.parent-first-patterns.additional</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">&#x4E00;&#x4E2A;&#xFF08;&#x4EE5;&#x5206;&#x53F7;&#x5206;&#x9694;&#x7684;&#xFF09;&#x6A21;&#x5F0F;&#x5217;&#x8868;&#xFF0C;&#x6307;&#x5B9A;&#x5E94;&#x59CB;&#x7EC8;&#x9996;&#x5148;&#x901A;&#x8FC7;&#x7236;ClassLoader&#x89E3;&#x6790;&#x54EA;&#x4E9B;&#x7C7B;&#x3002;
        &#x6A21;&#x5F0F;&#x662F;&#x4E00;&#x4E2A;&#x7B80;&#x5355;&#x7684;&#x524D;&#x7F00;&#xFF0C;&#x5B83;&#x6839;&#x636E;&#x5B8C;&#x5168;&#x9650;&#x5B9A;&#x7684;&#x7C7B;&#x540D;&#x8FDB;&#x884C;&#x68C0;&#x67E5;&#x3002;
        &#x8FD9;&#x4E9B;&#x6A21;&#x5F0F;&#x9644;&#x52A0;&#x5230;&#x201C;classloader.parent-first-patterns.default&#x201D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>classloader.parent-first-patterns.default</b>
      </td>
      <td style="text-align:left">
        <p>&quot;java.;scala.;</p>
        <p>org.apache.flink.;</p>
        <p>com.esotericsoftware.kryo;</p>
        <p>org.apache.hadoop.;</p>
        <p>javax.annotation.;</p>
        <p>org.slf4j;</p>
        <p>org.apache.log4j;</p>
        <p>org.apache.logging;</p>
        <p>org.apache.commons.logging;</p>
        <p>ch.qos.logback&quot;</p>
      </td>
      <td style="text-align:left">&#x4E00;&#x4E2A;&#xFF08;&#x4EE5;&#x5206;&#x53F7;&#x5206;&#x9694;&#x7684;&#xFF09;&#x6A21;&#x5F0F;&#x5217;&#x8868;&#xFF0C;&#x6307;&#x5B9A;&#x5E94;&#x59CB;&#x7EC8;&#x9996;&#x5148;&#x901A;&#x8FC7;&#x7236;ClassLoader&#x89E3;&#x6790;&#x54EA;&#x4E9B;&#x7C7B;&#x3002;
        &#x6A21;&#x5F0F;&#x662F;&#x4E00;&#x4E2A;&#x7B80;&#x5355;&#x7684;&#x524D;&#x7F00;&#xFF0C;&#x5B83;&#x6839;&#x636E;&#x5B8C;&#x5168;&#x9650;&#x5B9A;&#x7684;&#x7C7B;&#x540D;&#x8FDB;&#x884C;&#x68C0;&#x67E5;&#x3002;
        &#x901A;&#x5E38;&#x4E0D;&#x5E94;&#x4FEE;&#x6539;&#x6B64;&#x8BBE;&#x7F6E;&#x3002;
        &#x8981;&#x6DFB;&#x52A0;&#x5176;&#x4ED6;&#x6A21;&#x5F0F;&#xFF0C;&#x6211;&#x4EEC;&#x5EFA;&#x8BAE;&#x4F7F;&#x7528;&#x201C;classloader.parent-first-patterns.additional&#x201D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>classloader.resolve-order</b>
      </td>
      <td style="text-align:left">&quot;child-first&quot;</td>
      <td style="text-align:left">&#x4ECE;&#x7528;&#x6237;&#x4EE3;&#x7801;&#x52A0;&#x8F7D;&#x7C7B;&#x65F6;&#x5B9A;&#x4E49;&#x7C7B;&#x89E3;&#x6790;&#x7B56;&#x7565;&#xFF0C;&#x8FD9;&#x610F;&#x5473;&#x7740;&#x662F;&#x9996;&#x5148;&#x68C0;&#x67E5;&#x7528;&#x6237;&#x4EE3;&#x7801;jar&#xFF08;&#x201C;child-first&#x201D;&#xFF09;&#x8FD8;&#x662F;&#x5E94;&#x7528;&#x7A0B;&#x5E8F;&#x7C7B;&#x8DEF;&#x5F84;&#xFF08;&#x201C;parent-first&#x201D;&#xFF09;&#x3002;
        &#x9ED8;&#x8BA4;&#x8BBE;&#x7F6E;&#x6307;&#x793A;&#x9996;&#x5148;&#x4ECE;&#x7528;&#x6237;&#x4EE3;&#x7801;jar&#x52A0;&#x8F7D;&#x7C7B;&#xFF0C;&#x8FD9;&#x610F;&#x5473;&#x7740;&#x7528;&#x6237;&#x4EE3;&#x7801;jar&#x53EF;&#x4EE5;&#x5305;&#x542B;&#x548C;&#x52A0;&#x8F7D;&#x4E0D;&#x540C;&#x4E8E;Flink&#x4F7F;&#x7528;&#x7684;&#xFF08;&#x4F9D;&#x8D56;&#xFF09;&#x4F9D;&#x8D56;&#x9879;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>io.tmp.dirs</b>
      </td>
      <td style="text-align:left">&apos;LOCAL_DIRS&apos; on Yarn. &apos;_FLINK_TMP_DIR&apos; on Mesos. System.getProperty(&quot;java.io.tmpdir&quot;)
        in standalone.</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left"><b>parallelism.default</b>
      </td>
      <td style="text-align:left">1</td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>### JobManager

| Key | Default | Description |
| :--- | :--- | :--- |
| **jobmanager.archive.fs.dir** | \(none\) |  |
| **jobmanager.execution.attempts-history-size** | 16 | 历史上保存的先前执行尝试的最大数量。 |
| **jobmanager.heap.size** | "1024m" | JobManager的JVM堆大小。 |
| **jobmanager.resourcemanager.reconnect-interval** | 2000 | 如果与资源管理器的连接已丢失，此选项指定触发资源管理器重连接的间隔。此选项仅供内部使用。 |
| **jobmanager.rpc.address** | \(none\) | 定义要连接到的网络地址的配置参数，以便与作业管理器通信。此值仅在具有静态名称或地址的单个作业管理器存在的设置中解释\(简单的独立设置或具有动态服务名称解析的容器设置\)。在许多高可用性设置中，当使用leader-election service\(如ZooKeeper\)从多个备用作业管理器中选择和发现作业管理器的leader时，并不使用它。 |
| **jobmanager.rpc.port** | 6123 | 定义要连接到的网络端口的配置参数，以便与作业管理器通信。像jobmanager.rpc。address，此值仅在具有静态名称/地址和端口的单个作业管理器存在的设置中解释\(简单的独立设置，或具有动态服务名称解析的容器设置\)。在许多高可用性设置中不使用此配置选项，当使用leader-election服务\(如ZooKeeper\)从潜在的多个备用作业管理器中选择和发现作业管理器的leader时。 |
| **jobstore.cache-size** | 52428800 | 作业存储缓存大小\(以字节为单位\)，用于将已完成的作业保存在内存中。 |
| **jobstore.expiration-time** | 3600 | 完成的作业过期并从作业存储区中清除的时间，以秒为单位。 |
| **slot.idle.timeout** | 50000 | 槽池中空闲槽的超时\(以毫秒为单位\)。 |
| **slot.request.timeout** | 300000 | 从插槽池请求插槽的超时\(以毫秒为单位\)。 |

### TaskManager

| Key | Default | Description |
| :--- | :--- | :--- |
| **task.cancellation.interval** | 30000 | 连续两次任务取消尝试之间的时间间隔\(以毫秒为单位\)。 |
| **task.cancellation.timeout** | 180000 | 超时\(以毫秒为单位\)，任务取消超时后将导致致命的TaskManager错误。值0使看门狗失效。 |
| **task.cancellation.timers.timeout** | 7500 |  |
| **task.checkpoint.alignment.max-size** | -1 | 检查点对齐可能缓冲的最大字节数。如果检查点对齐缓冲的数据量超过配置的数据量，检查点将被中止\(跳过\)。值-1表示没有限制。 |
| **taskmanager.data.port** | 0 | 用于数据交换操作的任务管理器端口。 |
| **taskmanager.data.ssl.enabled** | true | 为taskmanager数据传输启用SSL支持。 仅当内部SSL的全局标志（security.ssl.internal.enabled）设置为true时，此选项才适用 |
| **taskmanager.debug.memory.log** | false | 指示是否启动线程的标志，该线程重复记录JVM的内存使用情况。 |
| **taskmanager.debug.memory.log-interval** | 5000 | 日志线程记录当前内存使用情况的间隔时间（以毫秒为单位）。 |
| **taskmanager.exit-on-fatal-akka-error** | false | 是否应启动任务管理器的隔离监视器。 隔离监视器如果检测到它已隔离另一个actor系统或已被另一个actor系统隔离，则会关闭actor系统。 |
| **taskmanager.heap.size** | "1024m" | TaskManagers的JVM堆大小，它是系统的并行工作者。 在YARN设置中，此值自动配置为TaskManager的YARN容器的大小，减去一定的容差值。 |
| **taskmanager.host** | \(none\) | TaskManager绑定到的网络接口的主机名。 默认情况下，TaskManager搜索可以连接到JobManager和其他TaskManagers的网络接口。 如果该策略由于某种原因失败，则此选项可用于定义主机名。 由于不同的TaskManagers需要此选项的不同值，因此通常在其他非共享的特定于TaskManager的配置文件中指定。 |
| **taskmanager.jvm-exit-on-oom** | false | 是否在任务线程抛出OutOfMemoryError时终止TaskManager。 |
| **taskmanager.network.detailed-metrics** | false | 布尔标志，用于启用/禁用有关入站/出站网络队列长度的更详细指标。 |
| **taskmanager.network.memory.buffers-per-channel** | 2 | 每个传出/传入通道（子分区/输入通道）使用的最大网络缓冲区数。在基于信用的流量控制模式下，这表示每个输入通道中有多少信用。它应配置至少2以获得良好的性能。1个缓冲区用于接收子分区中的飞行中数据，1个缓冲区用于并行序列化。 |
| **taskmanager.network.memory.floating-buffers-per-gate** | 8 | 每个输出/输入门（结果分区/输入门）使用的额外网络缓冲区数。 在基于信用的流量控制模式中，这表示在所有输入通道之间共享多少浮动信用。 浮动缓冲区基于积压（子分区中的实时输出缓冲区）反馈来分布，并且可以帮助减轻由子分区之间的不平衡数据分布引起的背压。 如果节点之间的往返时间较长和/或群集中的机器数量较多，则应增加此值。 |
| **taskmanager.network.memory.fraction** | 0.1 | 用于网络缓冲区的JVM内存的分数。 这决定了TaskManager可以同时拥有多少流数据交换通道以及通道缓冲的程度。 如果作业被拒绝或者您收到系统没有足够缓冲区的警告，请增加此值或下面的最小/最大值。 另请注意，“taskmanager.network.memory.min”和“taskmanager.network.memory.max”可能会覆盖此分数。 |
| **taskmanager.network.memory.max** | "1gb" | 网络缓冲区的最大内存大小。 |
| **taskmanager.network.memory.min** | "64mb" | 网络缓冲区的最小内存大小。 |
| **taskmanager.network.request-backoff.initial** | 100 | 输入通道的分区请求的最小回退时间（以毫秒为单位）。 |
| **taskmanager.network.request-backoff.max** | 10000 | 输入通道的分区请求的最大回退时间（以毫秒为单位）。 |
| **taskmanager.numberOfTaskSlots** | 1 | 单个TaskManager可以运行的并行操作员或用户功能实例的数量。 如果此值大于1，则单个TaskManager将获取函数或运算符的多个实例。 这样，TaskManager可以使用多个CPU内核，但同时，可用内存在不同的操作员或功能实例之间划分。 该值通常与TaskManager的机器具有的物理CPU核心数成比例（例如，等于核的数量，或核的数量的一半）。 |
| **taskmanager.registration.initial-backoff** | "500 ms" | 初始注册在两次连续注册尝试之间后退时间。每次新的注册尝试的回退次数将增加一倍，直到达到最大的注册回退次数为止。 |
| **taskmanager.registration.max-backoff** | "30 s" | 两次连续注册尝试之间的最大注册回退时间。最大注册回退时间需要一个时间单位说明符\(ms/s/min/h/d\)。 |
| **taskmanager.registration.refused-backoff** | "10 s" | 在重新尝试连接之前，作业管理器拒绝了注册后的回退时间。 |
| **taskmanager.registration.timeout** | "5 min" | 定义TaskManager注册的超时时间。如果在未成功注册的情况下超过持续时间，则TaskManager将终止。 |
| **taskmanager.rpc.port** | "0" | 任务管理器的IPC端口。接受端口列表（“50100,50101”），范围（“50100-50200”）或两者的组合。建议在同一台计算机上运行多个TaskManager时，设置一系列端口以避免冲突。 |

用于批处理作业\(或者如果`taskmanager.memoy.preallocate` 启用\)Flink只分配0.7部分空闲内存\(通过`taskmanager.heap.mb`配置的总内存减去用于网络缓冲区的内存\)用于其托管内存。托管内存帮助Flink有效地运行批处理操作符。它可以防止OutOfMemoryExceptions，因为Flink知道可以使用多少内存来执行操作。如果Flink耗尽托管内存，它将利用磁盘空间。使用托管内存，可以直接对原始数据执行一些操作，而不必反序列化数据以将其转换为Java对象。总之，托管内存提高了系统的健壮性和速度。

托管内存的默认分数可以使用`taskmanager.memory.fraction`参数进行调整。可以使用`taskmanager.memory.size`设置绝对值\(覆盖分数参数\)。如果需要，可以在JVM堆之外分配托管内存。这可能在内存较大的设置中提高性能。

| Key | Default | Description |
| :--- | :--- | :--- |
| **taskmanager.memory.fraction** | 0.7 | 任务管理器为排序，哈希表和中间结果的缓存预留的相对内存量（减去网络缓冲区使用的内存量之后）。例如，值`0.8`表示任务管理器为内部数据缓冲区保留80％的内存（堆上或堆外，具体取决于`taskmanager.memory.off-heap`），为任务管理器留下20％的可用内存用户定义函数创建的对象的堆。如果未设置`taskmanager.memory.size`，则仅评估此参数。 |
| **taskmanager.memory.off-heap** | false | 内存分配方法（JVM堆或堆外），用于TaskManager的托管内存。对于具有更大存储量的设置，这可以提高对存储器执行的操作的效率。 设置为true时，建议将`taskmanager.memory.preallocate`其设置为true。 |
| **taskmanager.memory.preallocate** | false | 在TaskManager启动时是否应预先分配TaskManager托管内存。如果`taskmanager.memory.off-heap`设置为true，则建议此配置也设置为true。如果此配置设置为false，则仅在通过触发完整GC达到配置的JVM参数MaxDirectMemorySize时才会清除已分配的堆外内存。对于流设置，强烈建议将此值设置为false，因为核心状态后端当前不使用托管内存。 |
| **taskmanager.memory.segment-size** | "32kb" | 网络堆栈和内存管理器使用的内存缓冲区的大小。 |
| **taskmanager.memory.size** | "0" | 任务管理器在堆上或堆外（取决于taskmanager.memory.off-heap）保留用于排序，哈希表和中间结果缓存的内存量（以兆字节为单位）。如果未指定，则内存管理器将根据taskmanager.memory.fraction指定的任务管理器JVM的大小采用固定比率。 |

### 分布式协调（通过Akka）

| Key | Default | Description |
| :--- | :--- | :--- |
| **akka.ask.timeout** | "10 s" | 超时用于所有期货和阻塞Akka调用。如果Flink由于超时而失败，那么应该尝试增加这个值。超时可能是由速度较慢的机器或拥挤的网络造成的。超时值需要一个时间单位说明符\(ms/s/min/h/d\)。 |
| **akka.client-socket-worker-pool.pool-size-factor** | 1.0 | 池大小因子用于使用以下公式确定线程池大小：ceil（可用处理器\*因子）。 然后，结果大小由pool-size-min和pool-size-max值限制。 |
| **akka.client-socket-worker-pool.pool-size-max** | 2 | 基于因子的线程数的最大上限。 |
| **akka.client-socket-worker-pool.pool-size-min** | 1 | 基于因子的线程数的最小上限。 |
| **akka.client.timeout** | "60 s" | 客户端所有阻塞调用的超时时间。 |
| **akka.fork-join-executor.parallelism-factor** | 2.0 | 并行因子用于使用以下公式确定线程池大小：ceil（可用处理器\*因子）。 然后，得到的大小受parallelism-min和parallelism-max值的限制。 |
| **akka.fork-join-executor.parallelism-max** | 64 | 基于因子的并行度的最大线程数。 |
| **akka.fork-join-executor.parallelism-min** | 8 | 基于因子的并行度的最小线程数。 |
| **akka.framesize** | "10485760b" | 作业管理器和任务管理器之间发送的消息的最大大小。如果Flink失败是因为消息超过了这个限制，那么应该增加它。消息大小需要一个大小-单位说明符。 |
| **akka.jvm-exit-on-fatal-error** | true | 在出现致命Akka错误时退出JVM。 |
| **akka.log.lifecycle.events** | false | 打开Akka对事件的远程日志记录。在调试时将此值设置为'true'。 |
| **akka.lookup.timeout** | "10 s" | 用于查找作业管理器的超时。超时值必须包含一个时间单位说明符\(ms/s/min/h/d\)。 |
| **akka.retry-gate-closed-for** | 50 | 断开远程连接后，应关闭门的毫秒数。 |
| **akka.server-socket-worker-pool.pool-size-factor** | 1.0 | 池大小因子用于使用以下公式确定线程池大小:ceil\(可用处理器\*因子\)。然后，结果大小由池大小-min和池大小-max值限定。 |
| **akka.server-socket-worker-pool.pool-size-max** | 2 | 将基于因子的线程数限制到的最大线程数。 |
| **akka.server-socket-worker-pool.pool-size-min** | 1 | 将基于因子的数量限制为的最小线程数。 |
| **akka.ssl.enabled** | true | 为Akka的远程通信打开SSL。仅当全局ssl标志security.ssl.enabled设置为true时适用。 |
| **akka.startup-timeout** | \(none\) | 超时之后，远程组件的启动被视为失败。 |
| **akka.tcp.timeout** | "20 s" | 所有出站连接超时。如果您在连接到任务管理器时遇到问题\(由于网络速度较慢\)，应该增加此值。 |
| **akka.throughput** | 15 | 在将线程返回到池之前在批处理中处理的消息的数量。低值表示公平调度，而高值可以以不公平为代价提高性能。 |
| **akka.transport.heartbeat.interval** | "1000 s" | Akka传输故障检测器的心跳间隔。由于Flink使用TCP，所以不需要检测器。因此，通过将间隔设置为非常高的值来禁用检测器。如果您需要传输故障检测器，请将间隔设置为某个合理的值。间隔值需要时间单位指定符\(ms/s/min/h/d\). |
| **akka.transport.heartbeat.pause** | "6000 s" | Akka的传输故障检测器可接受的心跳暂停。由于Flink使用TCP，所以不需要检测器。因此，通过将暂停设置为非常高的值来禁用检测器。如果您需要传输故障检测器，请将暂停设置为某个合理的值。暂停值需要时间单位指定符\(ms/s/min/h/d\). |
| **akka.transport.threshold** | 300.0 | 传输故障检测器的阈值。由于Flink使用TCP，所以检测器不是必需的，因此阈值被设置为高值。 |
| **akka.watch.heartbeat.interval** | "10 s" | Akka的DeathWatch机制检测死亡TaskManagers的心跳间隔。如果由于心跳消息丢失或延迟而导致TaskManagers被错误标记为死亡，那么您应该减小此值或增加akka.watch.heartbeat.pause。可在[此处](http://doc.akka.io/docs/akka/snapshot/scala/remoting.html#failure-detector)找到Akka's DeathWatch的详细说明 |
| **akka.watch.heartbeat.pause** | "60 s" | Akka的DeathWatch机制可接受的心跳暂停。 较低的值不允许不规则心跳。 如果由于心跳消息丢失或延迟而导致TaskManagers被错误地标记为死亡，那么应该增加此值或减少akka.watch.heartbeat.interval。 较高的值会增加检测死的TaskManager的时间。 可在[此处](http://doc.akka.io/docs/akka/snapshot/scala/remoting.html#failure-detector)找到Akka's DeathWatch的详细说明 |
| **akka.watch.threshold** | 12 | DeathWatch故障检测器的阈值。较低的值容易出现误报，而较高的值会增加检测死的TaskManager的时间。可在[此处](http://doc.akka.io/docs/akka/snapshot/scala/remoting.html#failure-detector)找到Akka's DeathWatch的详细说明 |

### REST

| Key | Default | Description |
| :--- | :--- | :--- |
| **rest.address** | \(none\) | 用于客户端连接到服务器的地址 |
| **rest.await-leader-timeout** | 30000 | 客户端等待领导者地址的时间（以毫秒为单位），例如Dispatcher或WebMonitorEndpoint |
| **rest.bind-address** | \(none\) | 服务器绑定自身的地址。 |
| **rest.client.max-content-length** | 104857600 | 客户端将处理的最大内容长度（以字节为单位）。 |
| **rest.connection-timeout** | 15000 | 客户端建立TCP连接的最长时间（以毫秒为单位）。 |
| **rest.idleness-timeout** | 300000 | 失败前连接保持空闲的最长时间（以毫秒为单位）。 |
| **rest.port** | 8081 | 服务器侦听的端口/客户端连接到的端口。 |
| **rest.retry.delay** | 3000 | 客户端在重试等待的时间（以ms为单位）（另请参阅“rest.retry.max-attempts”）。 |
| **rest.retry.max-attempts** | 20 | 如果可重试操作失败，客户机将尝试重试的次数。 |
| **rest.server.max-content-length** | 104857600 | 服务器将处理的最大内容长度（以字节为单位）。 |
| **rest.server.numThreads** | 4 | 异步处理请求的线程数。 |
| **rest.server.thread-priority** | 5 | REST服务器执行程序的线程优先级，用于处理异步请求。降低线程优先级将为Flink的主要组件提供更多的CPU时间，而增加将为REST服务器的处理分配更多时间。 |

### Blob Server

| Key | Default | Description |
| :--- | :--- | :--- |
| **blob.fetch.backlog** | 1000 | 配置参数定义JobManager上BLOB提取的backlog。 |
| **blob.fetch.num-concurrent** | 50 | 配置参数定义JobManager服务的最大并发BLOB提取数。 |
| **blob.fetch.retries** | 5 | 配置参数定义失败的BLOB提取的退出次数。 |
| **blob.offload.minsize** | 1048576 | 要卸载到BlobServer的消息的最小大小。 |
| **blob.server.port** | "0" | 配置参数定义blob服务的服务器端口。 |
| **blob.service.cleanup.interval** | 3600 | 任务管理器中blob缓存的清理间隔（以秒为单位）。 |
| **blob.service.ssl.enabled** | true | 标记以覆盖对blob服务传输的ssl支持。 |
| **blob.storage.directory** | \(none\) | 配置参数定义blob服务器使用的存 |

### 心跳管理

| Key | Default | Description |
| :--- | :--- | :--- |
| **heartbeat.interval** | 10000 | 从发送方请求心跳的时间间隔。 |
| **heartbeat.timeout** | 50000 | 发送方和接收方都请求和接收心跳超时。 |

### SSL设置

| Key | Default | Description |
| :--- | :--- | :--- |
| **security.ssl.algorithms** | "TLS\_RSA\_WITH\_AES\_128\_CBC\_SHA" | 要支持的标准SSL算法的逗号分隔列表。[在这里](http://docs.oracle.com/javase/8/docs/technotes/guides/security/StandardNames.html#ciphersuites)阅读更多 |
| **security.ssl.internal.close-notify-flush-timeout** | -1 | 刷新由关闭通道触发的“close\_notify”的超时\(在ms中\)。如果“close\_notify”没有在给定超时内刷新，通道将被强制关闭。\(-1 =使用系统默认值\) |
| **security.ssl.internal.enabled** | false | 打开SSL以进行内部网络通信。可选地，特定组件可以通过它们自己的设置（rpc，数据传输，REST等）**进行**覆盖。 |
| **security.ssl.internal.handshake-timeout** | -1 | SSL握手期间的超时\(以毫秒为单位\)。\(-1 =使用系统默认值\) |
| **security.ssl.internal.key-password** | \(none\) | 解密Flink内部端点\(rpc、数据传输、blob服务器\)密钥存储库中的密钥的秘密。 |
| **security.ssl.internal.keystore** | \(none\) | 带有SSL密钥和证书的Java密钥存储文件，用于Flink的内部端点\(rpc、数据传输、blob服务器\)。 |
| **security.ssl.internal.keystore-password** | \(none\) | 为Flink的内部端点\(rpc、数据传输、blob服务器\)解密密钥存储文件的秘密。 |
| **security.ssl.internal.session-cache-size** | -1 | 用于存储SSL会话对象的缓存的大小。根据https://github.com/netty/netty/issues/832，应该始终将此设置为适当的数字，以避免在垃圾收集期间遇到阻塞IO线程的bug。\(-1 =使用系统默认值\)。 |
| **security.ssl.internal.session-timeout** | -1 | 缓存的SSL会话对象的超时时间（以毫秒为单位）。（-1 =使用系统默认值） |
| **security.ssl.internal.truststore** | \(none\) | 包含用于验证Flink内部端点\(rpc、数据传输、blob服务器\)对等点的公共CA证书的信任存储文件。 |
| **security.ssl.internal.truststore-password** | \(none\) | 用于解密Flink内部端点\(rpc、数据传输、blob服务器\)信任存储的密码。 |
| **security.ssl.key-password** | \(none\) | 解密密钥库中的服务器密钥的秘密。 |
| **security.ssl.keystore** | \(none\) | flink端点用于其SSL密钥和证书的Java密钥库文件。 |
| **security.ssl.keystore-password** | \(none\) | 解密密钥存储库文件的秘密。 |
| **security.ssl.protocol** | "TLSv1.2" | SSL传输支持的SSL协议版本。注意，它不支持逗号分隔的列表。 |
| **security.ssl.rest.authentication-enabled** | false | 通过REST端点打开外部通信的相互SSL身份验证。 |
| **security.ssl.rest.enabled** | false | 通过REST端点打开SSL以进行外部通信。 |
| **security.ssl.rest.key-password** | \(none\) | 解密Flink外部REST端点的密钥库中密钥的秘密。 |
| **security.ssl.rest.keystore** | \(none\) | 带有SSL密钥和证书的Java密钥库文件，用于Flink的外部REST端点。 |
| **security.ssl.rest.keystore-password** | \(none\) | 为Flink的外部REST端点解密Flink的密钥库文件的秘密。 |
| **security.ssl.rest.truststore** | \(none\) | 包含公共CA证书的信任库文件，用于验证Flink的外部REST端点的对等方。 |
| **security.ssl.rest.truststore-password** | \(none\) | 用于解密Flink外部REST端点的信任库的密码。 |
| **security.ssl.truststore** | \(none\) | 信任库文件，包含要由flink端点用于验证对等方证书的公共CA证书。 |
| **security.ssl.truststore-password** | \(none\) | 解密信任库的秘诀。 |
| **security.ssl.verify-hostname** | true | 标记以在ssl握手期间启用对等方的主机名验证。 |

### 网络通讯（通过Netty）

这些参数允许高级调整。在大型集群上运行并发的高吞吐量作业时，默认值就足够了。

| Key | Default | Description |
| :--- | :--- | :--- |
| **taskmanager.network.netty.client.connectTimeoutSec** | 120 | Netty客户端连接超时时间。 |
| **taskmanager.network.netty.client.numThreads** | -1 | Netty客户端线程的数量。 |
| **taskmanager.network.netty.num-arenas** | -1 | Netty arena的数量 |
| **taskmanager.network.netty.sendReceiveBufferSize** | 0 | Netty发送和接收缓冲区大小。这默认为系统缓冲区大小（cat / proc / sys / net / ipv4 / tcp\_ \[rw\] mem），在现代Linux中为4 MiB。 |
| **taskmanager.network.netty.server.backlog** | 0 | netty服务器连接backlog. |
| **taskmanager.network.netty.server.numThreads** | -1 | Netty服务器线程数。 |
| **taskmanager.network.netty.transport** | "nio" | Netty传输类型，“nio”或“epoll” |

### Web前端

| Key | Default | Description |
| :--- | :--- | :--- |
| **web.access-control-allow-origin** | "\*" |  |
| **web.address** | \(none\) |  |
| **web.backpressure.cleanup-interval** | 600000 |  |
| **web.backpressure.delay-between-samples** | 50 |  |
| **web.backpressure.num-samples** | 100 |  |
| **web.backpressure.refresh-interval** | 60000 |  |
| **web.checkpoints.history** | 10 |  |
| **web.history** | 5 |  |
| **web.log.path** | \(none\) |  |
| **web.refresh-interval** | 3000 |  |
| **web.ssl.enabled** | true |  |
| **web.submit.enable** | true |  |
| **web.timeout** | 10000 |  |
| **web.tmpdir** | System.getProperty\("java.io.tmpdir"\) |  |
| **web.upload.dir** | \(none\) |  |

### 文件系统

| Key | Default | Description |
| :--- | :--- | :--- |
| **fs.default-scheme** | \(none\) | 默认文件系统方案，用于不显式声明方案的路径。可能包含一个权限，例如，在HDFS NameNode的情况下，host:port。 |
| **fs.output.always-create-directory** | false | 并行度大于1的文件编写器为输出文件路径创建一个目录，并将不同的结果文件\(每个并行编写器任务一个\)放入该目录。如果将该选项设置为“true”，并行度为1的编写器还将创建一个目录，并将一个结果文件放入其中。如果将该选项设置为“false”，则编写器将直接在输出路径上直接创建文件，而不创建包含目录。 |
| **fs.overwrite-files** | false | 指定默认情况下文件输出写入器是否应覆盖现有文件。默认设置为“true”覆盖，否则为“false”。 |

### 编译/优化

| Key | Default | Description |
| :--- | :--- | :--- |
| **compiler.delimited-informat.max-line-samples** | 10 | 编译器为分隔输入获取的最大行样本数。这些样本用于估计记录的数量。可以使用输入格式的参数为特定的输入重写此值。 |
| **compiler.delimited-informat.max-sample-len** | 2097152 | 编译器用于分隔输入的行样本的最大长度。如果单个样本的长度超过这个值\(可能是由于解析器配置错误\)，则采样将中止。可以使用输入格式的参数为特定的输入重写此值。 |
| **compiler.delimited-informat.min-line-samples** | 2 | 编译器为分隔输入所取的最小行样本数。这些样本用于估计记录的数量。可以使用输入格式的参数为特定的输入重写此值 |

### 运行时算法

| Key | Default | Description |
| :--- | :--- | :--- |
| **taskmanager.runtime.hashjoin-bloom-filters** | false | 标志在混合散列连接实现中激活/禁用bloom过滤器。在散列连接需要溢出到磁盘\(数据集大于内存的保留部分\)的情况下，这些bloom过滤器可以极大地减少溢出记录的数量，代价是牺牲一些CPU周期。 |
| **taskmanager.runtime.max-fan** | 128 | 外部合并连接的最大扇入和溢出哈希表的最大扇出。限制每个操作符的文件句柄数量，但是如果设置得太小，可能会导致中间合并/分区。 |
| **taskmanager.runtime.sort-spilling-threshold** | 0.8 | 当这部分内存预算满时，排序操作开始溢出。 |

### 资源管理

| Key | Default | Description |
| :--- | :--- | :--- |
| **containerized.heap-cutoff-min** | 600 | 容器中要删除的堆内存的最小数量，作为安全余量。 |
| **containerized.heap-cutoff-ratio** | 0.25 | 要从容器中删除的堆空间百分比（YARN / Mesos），以补偿其他JVM内存使用情况。 |
| **local.number-resourcemanager** | 1 |  |
| **resourcemanager.job.timeout** | "5 minutes" |  |
| **resourcemanager.rpc.port** | 0 | 定义要连接到的网络端口，以便与资源管理器通信。默认情况下，是JobManager的端口，因为使用了相同的ActorSystem。不可能使用这个配置键来定义端口范围。 |
| **resourcemanager.taskmanager-timeout** | 30000 | 释放空闲任务管理器的超时时间 |

### Yarn

| Key | Default | Description |
| :--- | :--- | :--- |
| **yarn.application-attempts** | \(none\) | 重新启动ApplicationMaster的次数。注意，整个Flink集群将重新启动，Yarn客户端将断开连接。此外，JobManager地址将会更改，需要手动设置JM host:port。建议将此选项保留为1。 |
| **yarn.application-master.port** | "0" | 使用此配置选项，用户可以为应用程序主RPC端口\(和JobManager\)指定端口、端口范围或端口列表。默认情况下，我们建议使用默认值\(0\)让操作系统选择适当的端口。特别是当多个AMs在同一物理主机上运行时，固定的端口分配会阻止AM启动。例如，当在具有限制性防火墙的环境中对Yarn运行Flink时，此选项允许指定允许端口的范围。 |
| **yarn.appmaster.rpc.address** | \(none\) | 应用程序主RPC系统正在侦听的主机名或地址。 |
| **yarn.appmaster.rpc.port** | -1 | 应用程序主RPC系统正在侦听的端口。 |
| **yarn.containers.vcores** | -1 | 每个纱筒的虚拟核心数。默认情况下，vcore的数量被设置为每个TaskManager的槽数\(如果设置为1\)，否则设置为1。为了使用这个参数，集群必须启用CPU调度。可以通过设置org.apache.hadoop.yarn.server.resourcemanager.scheduler.fair.FairScheduler来实现这一点. |
| **yarn.heartbeat-delay** | 5 | 心跳与心跳之间的间隔时间以秒为单位。 |
| **yarn.maximum-failed-containers** | \(none\) | 如果发生故障，系统将重新分配容器的最大数量。 |
| **yarn.per-job-cluster.include-user-jar** | "ORDER" | 定义每个作业集群的系统类路径中是否包含用户jar，以及它们在路径中的位置。它们可以放在开头\(“FIRST”\)、末尾\(“LAST”\)，或者根据它们的名称\(“ORDER”\)进行定位。将此参数设置为“DISABLED”将导致jar被包含在用户类路径中。 |
| **yarn.properties-file.location** | \(none\) | 当一个Flink作业被提交给YARN时，JobManager的主机和可用处理槽的数量被写入一个属性文件，这样Flink客户端就可以获取这些细节。此配置参数允许更改该文件的默认位置\(例如，对于在用户之间共享Flink安装的环境\)。 |
| **yarn.tags** | \(none\) | 用于Flink Yarn应用程序的逗号分隔的标记列表。 |

### Mesos

<table>
  <thead>
    <tr>
      <th style="text-align:left">Key</th>
      <th style="text-align:left">Default</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>mesos.failover-timeout</b>
      </td>
      <td style="text-align:left">604800</td>
      <td style="text-align:left">Mesos&#x8C03;&#x5EA6;&#x7A0B;&#x5E8F;&#x7684;&#x6545;&#x969C;&#x8F6C;&#x79FB;&#x8D85;&#x65F6;&#xFF08;&#x4EE5;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x4E4B;&#x540E;&#x5C06;&#x81EA;&#x52A8;&#x5173;&#x95ED;&#x6B63;&#x5728;&#x8FD0;&#x884C;&#x7684;&#x4EFB;&#x52A1;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.master</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">
        <p>Mesos master URL&#x3002;&#x8BE5;&#x503C;&#x5E94;&#x91C7;&#x7528;&#x4EE5;&#x4E0B;&#x5F62;&#x5F0F;&#x4E4B;&#x4E00;&#xFF1A;</p>
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
      <td style="text-align:left">&#x5B9A;&#x4E49;&#x8981;&#x4F7F;&#x7528;&#x7684;Mesos&#x5DE5;&#x4EF6;&#x670D;&#x52A1;&#x5668;&#x7AEF;&#x53E3;&#x7684;&#x914D;&#x7F6E;&#x53C2;&#x6570;&#x3002;&#x5C06;&#x7AEF;&#x53E3;&#x8BBE;&#x7F6E;&#x4E3A;0&#x5C06;&#x5141;&#x8BB8;&#x64CD;&#x4F5C;&#x7CFB;&#x7EDF;&#x9009;&#x62E9;&#x4E00;&#x4E2A;&#x53EF;&#x7528;&#x7684;&#x7AEF;&#x53E3;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.artifactserver.ssl.enabled</b>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">&#x4E3A;Flink&#x5DE5;&#x4EF6;&#x670D;&#x52A1;&#x5668;&#x542F;&#x7528;SSL&#x3002;&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;security.ssl.enabled&#x4E5F;&#x9700;&#x8981;&#x8BBE;&#x7F6E;&#x4E3A;true&#x52A0;&#x5BC6;&#x624D;&#x80FD;&#x542F;&#x7528;&#x52A0;&#x5BC6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.name</b>
      </td>
      <td style="text-align:left">&quot;Flink&quot;</td>
      <td style="text-align:left">Mesos&#x6846;&#x67B6;&#x540D;&#x79F0;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.principal</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">Mesos&#x6846;&#x67B6;&#x4E3B;&#x4F53;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.role</b>
      </td>
      <td style="text-align:left">&quot;*&quot;</td>
      <td style="text-align:left">Mesos&#x6846;&#x67B6;&#x89D2;&#x8272;&#x5B9A;&#x4E49;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.secret</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">Mesos&#x6846;&#x67B6; <b>secret</b>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.framework.user</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">Mesos&#x6846;&#x67B6;&#x7528;&#x6237;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>mesos.resourcemanager.tasks.port-assignments</b>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">&#x8868;&#x793A;&#x53EF;&#x914D;&#x7F6E;&#x7AEF;&#x53E3;&#x7684;&#x7528;&#x9017;&#x53F7;&#x5206;&#x9694;&#x7684;&#x914D;&#x7F6E;&#x952E;&#x5217;&#x8868;&#x3002;&#x6240;&#x6709;&#x7AEF;&#x53E3;&#x952E;&#x90FD;&#x5C06;&#x52A8;&#x6001;&#x5730;&#x83B7;&#x5F97;&#x901A;&#x8FC7;Mesos&#x5206;&#x914D;&#x7684;&#x7AEF;&#x53E3;&#x3002;</td>
    </tr>
  </tbody>
</table>#### **Mesos TaskManager**

| Key | Default | Description |
| :--- | :--- | :--- |
| **mesos.constraints.hard.hostattribute** | \(none\) | 基于代理属性在Mesos上放置任务的约束。采用以逗号分隔的key：value对列表，这些键对应于目标mesos代理公开的属性。示例：az:eu-west-1a,series:t2 |
| **mesos.resourcemanager.tasks.bootstrap-cmd** | \(none\) | 在TaskManager启动之前执行的命令。. |
| **mesos.resourcemanager.tasks.container.docker.force-pull-image** | false | 指示docker containerizer强制拉出图像，而不是重用缓存版本。 |
| **mesos.resourcemanager.tasks.container.docker.parameters** | \(none\) | 使用docker容器时，要传递到docker run命令的自定义参数。以逗号分隔的“key = value”对列表。“值”可能包含“=”。 |
| **mesos.resourcemanager.tasks.container.image.name** | \(none\) | 用于容器的映像名称。 |
| **mesos.resourcemanager.tasks.container.type** | "mesos" | 使用的集装箱类型：“mesos”或“docker”。 |
| **mesos.resourcemanager.tasks.container.volumes** | \(none\) | 逗号分隔的\[host\_path：\] container\_path \[：RO \| RW\]列表。这允许将额外的卷安装到容器中。 |
| **mesos.resourcemanager.tasks.cpus** | 0.0 | 要分配给Mesos工作者的CPU。 |
| **mesos.resourcemanager.tasks.gpus** | 0 | 要分配给Mesos工作者的GPU。 |
| **mesos.resourcemanager.tasks.hostname** | \(none\) | 用于定义TaskManager主机名的可选值。模式_TASK_被Mesos任务的实际id替换。这可用于配置TaskManager以使用Mesos DNS（例如_TASK_.flink-service.mesos）进行名称查找。 |
| **mesos.resourcemanager.tasks.mem** | 1024 | 以MB为单位分配给Mesos worker的内存。 |
| **mesos.resourcemanager.tasks.taskmanager-cmd** | "$FLINK\_HOME/bin/mesos-taskmanager.sh" |  |
| **mesos.resourcemanager.tasks.uris** | \(none\) | 以逗号分隔的自定义工件URI的列表，这些URI将下载到Mesos工作者的沙箱中。 |
| **taskmanager.numberOfTaskSlots** | 1 | 单个任务管理器可以运行的并行操作符或用户函数实例的数量。如果该值大于1，则单个TaskManager接受一个函数或操作符的多个实例。这样，TaskManager就可以利用多个CPU内核，但同时，可用内存被分配给不同的操作符或函数实例。这个值通常与TaskManager机器的物理CPU内核的数量成正比\(例如，等于内核的数量，或内核数量的一半\)。 |

### 高可用性（HA）

| Key | Default | Description |
| :--- | :--- | :--- |
| **high-availability** | "NONE" | 定义用于群集执行的高可用性模式。要启用高可用性，请将此模式设置为“ZOOKEEPER”或指定工厂类的FQN。 |
| **high-availability.cluster-id** | "/default" | Flink集群的ID，用于将多个Flink集群彼此分开。需要为独立群集设置，但在YARN和Mesos中自动推断。 |
| **high-availability.job.delay** | \(none\) | 故障转移后JobManager之前的时间恢复当前作业。 |
| **high-availability.jobmanager.port** | "0" | 作业管理器在高可用性模式下使用的可选端口（范围）。 |
| **high-availability.storageDir** | \(none\) | Flink在高可用性设置中保存元数据的文件系统路径（URL）。 |

#### **基于ZooKeeper的HA模式**

| Key | Default | Description |
| :--- | :--- | :--- |
| **high-availability.zookeeper.client.acl** | "open" | 定义要在ZK节点上配置的ACL（open \| creator）。 如果ZooKeeper服务器配置将“authProvider”属性映射为使用SASLAuthenticationProvider并且群集配置为以安全模式（Kerberos）运行，则可以将配置值设置为“creator”。 |
| **high-availability.zookeeper.client.connection-timeout** | 15000 | 定义ZooKeeper的连接超时时间（以毫秒为单位）。 |
| **high-availability.zookeeper.client.max-retry-attempts** | 3 | 定义客户端放弃之前的连接重试次数。 |
| **high-availability.zookeeper.client.retry-wait** | 5000 | 以ms为单位定义连续重试之间的暂停时间。 |
| **high-availability.zookeeper.client.session-timeout** | 60000 | 以ms为单位定义ZooKeeper会话的会话超时时间。 |
| **high-availability.zookeeper.path.checkpoint-counter** | "/checkpoint-counter" | ZooKeeper根路径（ZNode）用于检查点计数器。 |
| **high-availability.zookeeper.path.checkpoints** | "/checkpoints" | 已完成检查点的ZooKeeper根路径（ZNode） |
| **high-availability.zookeeper.path.jobgraphs** | "/jobgraphs" | 作业图的ZooKeeper根路径（ZNode） |
| **high-availability.zookeeper.path.latch** | "/leaderlatch" | 定义用于选择领导者的领导者锁存器的znode。 |
| **high-availability.zookeeper.path.leader** | "/leader" | 定义leader的znode，其中包含leader的URL和当前leader会话ID。 |
| **high-availability.zookeeper.path.mesos-workers** | "/mesos-workers" | 用于保存Mesos worker信息的ZooKeeper根路径。 |
| **high-availability.zookeeper.path.root** | "/flink" | Flink在ZooKeeper中存储其条目的根路径。 |
| **high-availability.zookeeper.path.running-registry** | "/running\_job\_registry/" |  |
| **high-availability.zookeeper.quorum** | \(none\) | 使用ZooKeeper在高可用性模式下运行Flink时要使用的ZooKeeper **quorum**。 |

### ZooKeeper安全

| Key | Default | Description |
| :--- | :--- | :--- |
| **zookeeper.sasl.disable** | false |  |
| **zookeeper.sasl.login-context-name** | "Client" |  |
| **zookeeper.sasl.service-name** | "zookeeper" |  |

### 基于Kerberos的安全性

| Key | Default | Description |
| :--- | :--- | :--- |
| **security.kerberos.login.contexts** | \(none\) | 以逗号分隔的登录上下文列表，用于提供Kerberos凭据（例如，`Client，KafkaClient`使用凭据进行ZooKeeper身份验证和Kafka身份验证） |
| **security.kerberos.login.keytab** | \(none\) | 包含用户凭据的Kerberos密钥表文件的绝对路径。 |
| **security.kerberos.login.principal** | \(none\) | 与keytab关联的Kerberos主体名称。 |
| **security.kerberos.login.use-ticket-cache** | true | 指示是否从Kerberos票证缓存中读取。 |

### 环境

| Key | Default | Description |
| :--- | :--- | :--- |
| **env.hadoop.conf.dir** | \(none\) | hadoop配置目录的路径。需要读取HDFS和/或YARN配置。也可以通过环境变量进行设置。 |
| **env.java.opts** | \(none\) | 用于启动所有Flink进程的JVM的Java选项。 |
| **env.java.opts.historyserver** | \(none\) | 用于启动HistoryServer的JVM的Java选项。 |
| **env.java.opts.jobmanager** | \(none\) | 用于启动JobManager的JVM的Java选项。 |
| **env.java.opts.taskmanager** | \(none\) | 用于启动TaskManager的JVM的Java选项。 |
| **env.log.dir** | \(none\) | 定义保存Flink日志的目录。它必须是一条绝对的道路。（默认为Flink主页下的日志目录） |
| **env.log.max** | 5 | 要保留的最大旧日志文件数。 |
| **env.ssh.opts** | \(none\) | 启动或停止JobManager，TaskManager和Zookeeper服务时，其他命令行选项传递给SSH客户端\(start-cluster.sh，stop-cluster.sh，start-zookeeper-quorum.sh，stop-zookeeper-quorum.sh\) |
| **env.yarn.conf.dir** | \(none\) | YARN配置目录的路径。它需要在YARN上运行flink。也可以通过环境变量进行设置。 |

### 检查点

| Key | Default | Description |
| :--- | :--- | :--- |
| **state.backend** | \(none\) | 状态后端用于存储和检查点状态。 |
| **state.backend.async** | true | 选择状态后端是否应在可能和可配置的情况下使用异步快照方法。 某些状态后端可能不支持异步快照，或仅支持异步快照，并忽略此选项。 |
| **state.backend.fs.memory-threshold** | 1024 | 状态数据文件的最小大小。小于该值的所有状态块都内联存储在根检查点元数据文件中。 |
| **state.backend.incremental** | false | 选项状态后端是否应创建增量检查点（如果可能）。对于增量检查点，只存储与前一个检查点的差异，而不存储完整检查点状态。某些状态后端可能不支持增量检查点并忽略此选项。 |
| **state.backend.local-recovery** | false | 此选项配置此状态后端的本地恢复。默认情况下，本地恢复被停用。本地恢复目前只覆盖键控状态后端。目前，memoryStateBackend不支持本地恢复并忽略此选项。 |
| **state.checkpoints.dir** | \(none\) | 默认目录，用于在Flink支持的文件系统中存储检查点的数据文件和元数据。存储路径必须可从所有参与的流程/节点（即所有任务管理器和作业管理器）访问。 |
| **state.checkpoints.num-retained** | 1 | 要保留的已完成检查点的最大数量。 |
| **state.savepoints.dir** | \(none\) | 保存点的默认目录。由将后端写入文件系统的状态后端（MemoryStateBackend，FsStateBackend，RocksDBStateBackend）使用。 |
| **taskmanager.state.local.root-dirs** | \(none\) | 定义根目录的配置参数，用于存储用于本地恢复的基于文件的状态。本地恢复目前只覆盖键控状态后端。目前，MemoryStateBackend不支持本地恢复并忽略此选项 |

### RocksDB**状态**后端

| Key | Default | Description |
| :--- | :--- | :--- |
| **state.backend.rocksdb.localdir** | \(none\) | RocksDB放置文件的本地目录（在TaskManager上）。 |
| **state.backend.rocksdb.timer-service.factory** | "HEAP" | 这确定了工厂的计时器服务状态实现。 选项可以是基于RocksDB的HEAP（基于堆，默认）或ROCKSDB。 |

### 可查询状态

| Key | Default | Description |
| :--- | :--- | :--- |
| **query.client.network-threads** | 0 | 可查询状态客户端的网络线程数\(Netty的事件循环\)。 |
| **query.proxy.network-threads** | 0 | 可查询状态代理的网络线程数\(Netty的事件循环\)。 |
| **query.proxy.ports** | "9069" | 可查询状态代理的端口范围。指定的范围可以是单个端口：“9123”，一系列端口：“50100-50200”，或范围和端口列表：“50100-50200,50300-50400,51234”。 |
| **query.proxy.query-threads** | 0 | 可查询状态代理的查询线程数。如果设置为0，则使用插槽数。 |
| **query.server.network-threads** | 0 | 可查询状态服务器的网络线程数\(Netty的事件循环\)。 |
| **query.server.ports** | "9067" | 可查询状态服务器的端口范围。指定的范围可以是单个端口：“9123”，一系列端口：“50100-50200”，或范围和端口列表：“50100-50200,50300-50400,51234”。 |
| **query.server.query-threads** | 0 | 可查询状态服务器的查询线程数。如果设置为0，则使用插槽数。 |

### 度量



### RocksDB Native 度量标准

某些RocksDB本机指标可能会转发给Flink的指标报告者。 所有本机度量标准都限定为运算符，然后按列族进一步细分; 值报告为unsigned longs。

{% hint style="warning" %}
**注意：**启用本机度量标准可能会导致性能下降，应谨慎设置。
{% endhint %}

| Key | Default | Description |
| :--- | :--- | :--- |
| **state.backend.rocksdb.metrics.actual-delayed-write-rate** | false | 监控当前实际的延迟写入速率。0表示没有延迟。 |
| **state.backend.rocksdb.metrics.background-errors** | false | 监控RocksDB中的后台错误数。 |
| **state.backend.rocksdb.metrics.compaction-pending** | false | 跟踪RocksDB中的待定压缩。如果压缩处于挂起状态，则返回1，否则返回0。 |
| **state.backend.rocksdb.metrics.cur-size-active-mem-table** | false | 以字节为单位监视活跃memtable的大致大小。 |
| **state.backend.rocksdb.metrics.cur-size-all-mem-tables** | false | 以字节为单位监视活动和未刷新的不可变memtables的大致大小。 |
| **state.backend.rocksdb.metrics.estimate-live-data-size** | false | 以字节为单位估计实时数据量。 |
| **state.backend.rocksdb.metrics.estimate-num-keys** | false | 估计RocksDB中的Key数量。 |
| **state.backend.rocksdb.metrics.estimate-pending-compaction-bytes** | false | 压缩需要重写的估计总字节数，以使所有级别降至目标大小以下。对于基于级别的其他压缩无效。 |
| **state.backend.rocksdb.metrics.estimate-table-readers-mem** | false | 估计用于读取SST表的内存，不包括块缓存中使用的内存（例如，过滤器和索引块），以字节为单位。 |
| **state.backend.rocksdb.metrics.mem-table-flush-pending** | false | 监控RocksDB中待处理的memtable刷新次数。 |
| **state.backend.rocksdb.metrics.num-deletes-active-mem-table** | false | 监视活跃memtable中的删除条目总数。 |
| **state.backend.rocksdb.metrics.num-deletes-imm-mem-tables** | false | 监视未刷新的不可变memtables中删除条目的总数。 |
| **state.backend.rocksdb.metrics.num-entries-active-mem-table** | false | 监视活跃memtable中的条目总数。 |
| **state.backend.rocksdb.metrics.num-entries-imm-mem-tables** | false | 监视未刷新的不可变memtables中的条目总数。 |
| **state.backend.rocksdb.metrics.num-immutable-mem-table** | false | 监控RocksDB中不可变memtable的数量。 |
| **state.backend.rocksdb.metrics.num-live-versions** | false | 监控实时版本的数量。 版本是内部数据结构。 有关详细信息，请参阅RocksDB文件version\_set.h。 更多实时版本通常意味着更多SST文件被删除，迭代器或未完成的压缩。 |
| **state.backend.rocksdb.metrics.num-running-compactions** | false | 监控当前运行的压缩的数量。 |
| **state.backend.rocksdb.metrics.num-running-flushes** | false | 监控当前正在运行的刷新次数。 |
| **state.backend.rocksdb.metrics.num-snapshots** | false | 监视数据库未发布的快照数。 |
| **state.backend.rocksdb.metrics.size-all-mem-tables** | false | 以字节为单位监视活动，未刷新的不可变和固定不可变memtables的大致大小。 |
| **state.backend.rocksdb.metrics.total-sst-files-size** | false | 监视所有SST文件的总大小（字节）。警告：如果文件太多，可能会减慢在线查询的速度。 |

### HistoryServer

如果要通过HistoryServer的Web前端展示它们，则必须配置jobmanager.archive.fs.dir以存档已终止的作业并通过historyserver.archive.fs.dir将其添加到受监视目录列表中。

* `jobmanager.archive.fs.dir`：将有关已终止作业的信息上载到的目录。你必须将此目录添加到历史服务器的受监视目录列表中`historyserver.archive.fs.dir`。

| Key | Default | Description |
| :--- | :--- | :--- |
| **historyserver.archive.fs.dir** | \(none\) | 以逗号分隔的目录列表，用于从中获取已归档的作业。历史服务器将监视这些目录以获取已存档的作业。您可以将JobManager配置为通过`jobmanager.archive.fs.dir`将作业存档到目录。 |
| **historyserver.archive.fs.refresh-interval** | 10000 | 刷新已归档作业目录的时间间隔（以毫秒为单位）。 |
| **historyserver.web.address** | \(none\) | HistoryServer的Web界面的地址。 |
| **historyserver.web.port** | 8082 | HistoryServers的Web界面的端口。 |
| **historyserver.web.refresh-interval** | 10000 |  |
| **historyserver.web.ssl.enabled** | false | 启用对HistoryServer Web前端的HTTP访问。仅当全局SSL标志security.ssl.enabled设置为true时，此选项才适用。d |
| **historyserver.web.tmpdir** | \(none\) | 此配置参数允许定义历史服务器Web界面使用的Flink Web目录。Web界面将其静态文件复制到目录中。 |

## 遗留\(Legacy\)

* `mode`：Flink的执行模式。可能的值是`legacy`和`new`。要启动旧模式，您必须指定`legacy`（DEFAULT：`new`）。

## 背景

### 配置网络缓冲区

如果看到异常`java.io.IOException: Insufficient number of network buffers`，则需要调整用于网络缓冲区的内存量，以便程序在任务管理器上运行。

网络缓冲区是通信层的关键资源。用于在通过网络传输之前缓冲记录，并在将传入数据解析为记录并将其传递给应用程序之前缓冲传入数据。足够数量的网络缓冲区对于实现良好的吞吐量至关重要。

{% hint style="info" %}
从Flink 1.3开始，你可以遵循“越多越好”的成语而不会对延迟造成任何影响（我们通过限制每个通道使用的实际缓冲区数量来防止每个传出和传入通道中的过度缓冲，即_缓冲膨胀_）。
{% endhint %}

通常，将任务管理器配置为具有足够的缓冲区，使您希望同时打开的每个逻辑网络连接都具有专用缓冲区。 对于网络上的每个点对点数据交换存在逻辑网络连接，这通常发生在重新分区或广播步骤（混洗阶段）。 在这些任务中，TaskManager中的每个并行任务必须能够与所有其他并行任务进行通信。

{% hint style="warning" %}
**注意：**从Flink 1.5开始，无论taskmanager.memory.off-heap的值多少，网络缓冲区将始终在堆外分配，即在JVM堆之外。 这样，我们可以将这些缓冲区直接传递给底层网络堆栈层。
{% endhint %}

#### **设置内存部分**

以前，手动设置网络缓冲区的数量，这成为一个非常容易出错的任务（见下文）。 从Flink 1.3开始，可以使用以下配置参数定义用于网络缓冲区的部分内存：

* `taskmanager.network.memory.fraction`：用于网络缓冲区的JVM内存的分数（DEFAULT：0.1），
* `taskmanager.network.memory.min`：网络缓冲区的最小内存大小（默认值：64MB），
* `taskmanager.network.memory.max`：网络缓冲区的最大内存大小（默认值：1GB），和
* `taskmanager.memory.segment-size`：内存管理器和网络堆栈使用的内存缓冲区大小（以字节为单位）（默认值：32KB）。

#### **直接设置网络缓冲区的数量**

{% hint style="warning" %}
**注意：不建议使用**这种配置网络缓冲区使用的内存量的方法。请考虑使用上述方法，定义要使用的内存部分。
{% endhint %}

任务管理器上所需的缓冲区数量是完全并行度（目标数量）_节点内并行性（一个任务管理器中的源数量）_ n n是一个常量，用于定义重新分区的数量 - /你希望同时处于活动状态的广播步骤。 由于节点内并行性通常是核心数量，并且超过4个重新分区或广播频道很少并行活动，因此它通常归结为**：**

```text
#slots-per-TM^2 * #TMs * 4
```

其中`#slots per TM`是[每个任务管理器插槽数量](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html#configuring-taskmanager-processing-slots)，`＃TM`是任务管理器的总数。

例如，为了支持20个8插槽机器的集群，你应该使用大约5000个网络缓冲区来获得最佳吞吐量。

默认情况下，每个网络缓冲区的大小为32 KiBytes。在上面的示例中，系统因此将为网络缓冲区分配大约300 MiBytes。

可以使用以下参数配置网络缓冲区的数量和大小：

* `taskmanager.network.numberOfBuffers`
* `taskmanager.memory.segment-size`。

### 配置临时I / O目录

虽然Flink的目标是尽可能多地处理主内存中的数据，但是需要处理的内存比内存更多的数据并不少见。Flink的运行时用于将临时数据写入磁盘以处理这些情况。

`taskmanager.tmp.dirs`参数指定Flink写入临时文件的目录列表。目录的路径需要用**':'**（冒号字符）分隔。Flink将同时向（从）每个配置的目录写入（或读取）一个临时文件。这样，临时I/O可以均匀地分布在多个独立的I/O设备（如硬盘）上，以提高性能。要利用快速I/O设备（例如，SSD，RAID，NAS），可以多次指定目录。

如果`taskmanager.tmp.dirs`未显式指定参数，Flink会将临时数据写入操作系统的临时目录，例如Linux系统中的_`/tmp`_。

### 配置TaskManager处理槽

Flink通过将程序拆分为子任务并将这些子任务调度到处理槽来并行执行程序。

每个Flink TaskManager都在集群中提供处理插槽。插槽数通常与**每个** TaskManager 的可用CPU核心数成比例。作为一般建议，可用的CPU核心数量是一个很好的默认值`taskmanager.numberOfTaskSlots`。

启动Flink应用程序时，用户可以提供用于该作业的默认插槽数。因此调用命令行值`-p`（用于并行）。此外，可以为整个应用程序和各个操作员[设置编程API中的插槽数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/parallel.html)。

![](../.gitbook/assets/image%20%2819%29.png)

![](../.gitbook/assets/image%20%2817%29.png)

![](../.gitbook/assets/image%20%286%29.png)

![](../.gitbook/assets/image%20%2824%29.png)

