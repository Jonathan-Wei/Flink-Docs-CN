# 高可用\(HA\)

JobManager协调每个Flink部署。它负责_调度_和_资源管理_。

默认情况下，每个Flink群集都有一个JobManager实例。这会产生_单点故障_（SPOF）：如果JobManager崩溃，则无法提交新程序并且运行程序失败。

使用JobManager High Availability，可以从JobManager故障中恢复，从而消除_SPOF_。可以为**独立**和**YARN群集**配置高可用性。

## Standalone集群高可用

针对独立集群的JobManager高可用性的一般思想是，在任何时候都有一个**领导的JobManager**，并且在领导者失败时有多个**备用JobManager**来接管领导权。这保证了没有单一的故障点，程序可以在备用JobManager接管后立即继续执行。备用实例和主JobManager实例之间没有明显的区别。每个JobManager都可以充当master或standby角色。

例如，考虑以下三个JobManager实例的设置:

![](../.gitbook/assets/jobmanager_ha_overview.png)

### 配置

要启用JobManager高可用性，您必须将**高可用性模式设置**为_zookeeper_，配置**ZooKeeper仲裁**并设置包含所有JobManagers主机及其Web UI端口的**主服务器文件**。

Flink利用[**ZooKeeper**](http://zookeeper.apache.org/)在所有正在运行的JobManager实例之间进行_分布式协调_。ZooKeeper是Flink的独立服务，通过领导者选举和轻量级一致状态存储提供高度可靠的分布式协调。有关ZooKeeper的更多信息，请查看[ZooKeeper的入门指南](http://zookeeper.apache.org/doc/trunk/zookeeperStarted.html)。Flink包含用于[引导简单ZooKeeper](https://ci.apache.org/projects/flink/flink-docs-master/ops/jobmanager_high_availability.html#bootstrap-zookeeper)安装的脚本。

#### Masters文件

为了启动HA-cluster，在`conf/master`文件中配置`master`:

* **masters文件**：_masters文件_包含启动JobManagers的所有主机以及Web用户界面绑定的端口。

```text
jobManagerAddress1：webUIPort1
[...]
jobManagerAddressX：webUIPortX
```

默认情况下，作业管理器将为进程间通信选择一个_随机端口_。您可以通过**`high-availability.jobmanager.port`**更改此设置。配置接受单个端口（例如`50010`），范围（`50000-50025`）或两者的组合（`50010,50011,50020-50025,50050-50075`）。  


#### **配置文件（flink-conf.yaml）**

要启动HA群集，请在`conf/flink-conf.yaml`添加以下配置：

* **高可用性模式**（必需）：必须在`conf/flink-conf.yaml`中将高可用模式设置为zookeeper才能启用高可用模式。 或者，此选项可以设置为Flink应该用于创建`HighAvailabilityServices`实例的工厂类的FQN。

```text
high-availability: zookeeper
```

* **ZooKeeper仲裁**（必需）：_ZooKeeper仲裁_是ZooKeeper服务器的复制组，它提供分布式协调服务。

```text
high-availability.zookeeper.quorum: address1:2181[,...],addressX:2181
```

每个_addressX：port_指的是一个ZooKeeper服务器，Flink可以在给定的地址和端口访问它。

* **Zookeeper root**（推荐）：_ZooKeeper root节点_，在该_节点_下放置所有集群节点。

```text
high-availability.zookeeper.path.root: /flink
```

* **Zookeeper Cluster-id**（推荐）：_cluster-id ZooKeeper节点_，在该_节点_下放置集群的所有必需的协调数据。

```
high-availability.cluster-id: /default_ns # important: customize per cluster
```

* **存储目录**（必需）：JobManager元数据保存在文件系统storageDir中，只有一个指向该状态的指针存储在ZooKeeper中。

```text
high-availability.storageDir: hdfs:///flink/recovery
```

        storageDir存储JobManager故障恢复所需的所有元数据。

配置主服务器和ZooKeeper仲裁后，您可以像往常一样使用提供的集群启动脚本。他们将启动HA群集。请记住，调用脚本时**必须运行ZooKeeper仲裁**，并确保为要**启动的**每个HA群集**配置单独的ZooKeeper根路径**。

**示例: 2个JobManager的Standalone集群**

1.在`conf/flink-conf.yaml`中配置高可用模式和ZooKeeper quorum:

```text
high-availability: zookeeper
high-availability.zookeeper.quorum: localhost:2181
high-availability.zookeeper.path.root: /flink
high-availability.cluster-id: /cluster_one # important: customize per cluster
high-availability.storageDir: hdfs:///flink/recovery
```

2.在`conf/masters`中**配置masters**

```text
localhost:8081
localhost:8082
```

3.在`conf/zoo.cfg`中**配置zookeeper servers**\(目前每台机器只能运行一台ZooKeeper服务器\):

```text
server.0=localhost:2888:3888
```

4.**启动ZooKeeper quorum**:

```text
$ bin/start-zookeeper-quorum.sh
Starting zookeeper daemon on host localhost.
```

5.**启动一个高可用集群**

```text
$ bin/start-cluster.sh
Starting HA cluster with 2 masters and 1 peers in ZooKeeper quorum.
Starting jobmanager daemon on host localhost.
Starting jobmanager daemon on host localhost.
Starting taskmanager daemon on host localhost.
```

6.**停止ZooKeeper quorum和集群：**

```bash
$ bin/stop-cluster.sh
Stopping taskmanager daemon (pid: 7647) on localhost.
Stopping jobmanager daemon (pid: 7495) on host localhost.
Stopping jobmanager daemon (pid: 7349) on host localhost.
$ bin/stop-zookeeper-quorum.sh
Stopping zookeeper daemon (pid: 7101) on host localhost.
```

## Yarn集群高可用

在运行高可用性YARN群集时，**我们不会运行多个JobManager（ApplicationMaster）实例**，而只会运行一个，由YARN在失败时重新启动。确切的行为取决于您使用的特定YARN版本。

### 配置

#### **最大应用程序主要尝试次数（yarn-site.xml）**

您必须在yarn-site.xml中为您的yarn设置配置Application Master的最大尝试次数：

```markup
<property>
  <name>yarn.resourcemanager.am.max-attempts</name>
  <value>4</value>
  <description>
    The maximum number of application master execution attempts.
  </description>
</property>
```

当前YARN版本的默认值为2（表示允许单个JobManager失败）。

#### 申请重试\(**flink-conf.yaml**\)

除HA配置（[参见上文](https://ci.apache.org/projects/flink/flink-docs-master/ops/jobmanager_high_availability.html#configuration)）外，您还必须配置最大尝试次数`conf/flink-conf.yaml`：

```text
yarn.application-attempts: 10
```

这意味着在Yarn应用程序失败之前，应用程序可以重新启动9次\(9次重试+ 1次初始尝试\)。如果Yarn需要，Yarn可以执行额外的重启操作:抢占、节点硬件故障或重新引导，或NodeManager重新同步。这些重启不计入`yarn.application-attempts`，请参阅[Jian Fang的博客文章](http://johnjianfang.blogspot.de/2015/04/the-number-of-maximum-attempts-of-yarn.html)。需要注意的是，`yarn.resourcemanager.am.max-attempts`是应用程序重新启动的上限。因此，Flink中设置的应用程序尝试次数不能超过开始使用Yarn的Yarn集群设置。

#### **容器关闭行为**

* **YARN 2.3.0 &lt;版本&lt;2.4.0**。如果应用程序主机失败，则重新启动所有容器。
* **YARN 2.4.0 &lt;版本&lt;2.6.0**。TaskManager容器在应用程序主故障期间​​保持活动状态。具有以下优点：启动时间更快并且用户不必再等待再次获得容器资源。
* **YARN 2.6.0 &lt;= version**：将尝试失败有效性间隔设置为Flinks的Akka超时值。尝试失败有效性间隔表示只有在系统在一个间隔期间看到最大应用程序尝试次数后才会终止应用程序。这避免了持久的工作会耗尽它的应用程序尝试。

{% hint style="danger" %}
注意：Hadoop YARN 2.4.0有一个主要的bug\(修复在2.5.0中\)，它阻止容器从重新启动的Application Master/JobManager容器重新启动。有关详细信息，请参阅[FLINK-4142](https://issues.apache.org/jira/browse/FLINK-4142)。我们建议至少在YARN上使用Hadoop 2.5.0进行高可用性设置。
{% endhint %}

#### **示例: 高可用YARN Session**

1.在`conf/flink-conf.yaml`中**配置高可用模式和ZooKeeper quorum**:

```text
high-availability: zookeeper
high-availability.zookeeper.quorum: localhost:2181
high-availability.storageDir: hdfs:///flink/recovery
high-availability.zookeeper.path.root: /flink
yarn.application-attempts: 10
```

2.在`conf/zoo.cfg`中**配置zookeeper servers**\(目前每台机器只能运行一台ZooKeeper服务器\):

```text
server.0=localhost:2888:3888
```

3.**启动ZooKeeper quorum**:

```text
$ bin/start-zookeeper-quorum.sh
Starting zookeeper daemon on host localhost.
```

4.**启动一个高可用集群**

```text
$ bin/yarn-session.sh -n 2
```

## Zookeeper安全配置

如果ZooKeeper使用Kerberos以安全模式运行，则可以`flink-conf.yaml`根据需要覆盖以下配置：

```text
zookeeper.sasl.service-name: zookeeper     # default is "zookeeper". If the ZooKeeper quorum is configured
                                           # with a different service name then it can be supplied here.
zookeeper.sasl.login-context-name: Client  # default is "Client". The value needs to match one of the values
                                           # configured in "security.kerberos.login.contexts".
```

有关Kerberos安全性的Flink配置的更多信息，请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html)。您还可以[在此处](https://ci.apache.org/projects/flink/flink-docs-master/ops/security-kerberos.html)找到有关Flink内部如何设置基于Kerberos的安全性的更多详细信息。

## 引导Zookeeper

如果您没有独立的已运行的Zookeeper，则可以使用Flink附带的帮助程序脚本。

在`conf/zoo.cfg`中有一个ZooKeeper配置模板。您可以将主机配置为使用`server.X`条目运行ZooKeeper ，其中X是每个服务器的唯一ID：

```text
server.X=addressX:peerPort:leaderPort
[...]
server.Y=addressY:peerPort:leaderPort
```

`bin/start-zookeeper-quorum.sh`脚本将在每个配置的主机上启动ZooKeeper服务器。启动的进程通过Flink包装器启动ZooKeeper服务器，该包装器从中读取配置`conf/zoo.cfg`并确保为方便起见设置一些必需的配置值。在生产设置中，建议构建单独的zookeeper集群管理。

