# Kerberos

本文档简要描述了Flink安全性在各种部署机制（独立，YARN或Mesos），文件系统，连接器和状态后端的上下文中如何工作。

## 目的

Flink Kerberos安全基础结构的主要目标是：

1. 通过连接器（例如Kafka）为群集内的作业启用安全数据访问
2. 对ZooKeeper进行身份验证（如果配置为使用SASL）
3. 对Hadoop组件（例如HDFS，HBase）进行身份验证

在生产部署方案中，流作业应理解为可以长时间运行（几天/几周/几月），并且能够在整个工作周期内进行身份验证以保护数据源。Kerberos密钥表不会在该时间范围内到期，这与Hadoop委派令牌或票证缓存配置不同。

当前的实现支持使用配置的keytab凭据或Hadoop委托令牌运行Flink集群\(JobManager / TaskManager / jobs\)。请记住，所有作业都共享为给定集群配置的凭据。要为某个作业使用不同的keytab，只需使用不同的配置启动一个单独的Flink集群。多个Flink集群可以在纱线或Mesos环境中并排运行。

## Flink Security如何工作

在概念上，Flink程序可以使用第一或第三方连接器\(Kafka、HDFS、Cassandra、Flume、Kinesis等\)，需要使用任意的身份验证方法\(Kerberos、SSL/TLS、用户名/密码等\)。虽然满足所有连接器的安全需求是一项持续的工作，但是Flink只提供了对Kerberos身份验证的一流支持。Kerberos身份验证支持以下服务和连接器:

* Kafka \(0.9+\)
* HDFS
* HBase
* ZooKeeper

请注意，可以为每个服务或连接器单独启用Kerberos。例如，用户可以启用Hadoop安全性，而不必为ZooKeeper使用Kerberos，反之亦然。共享元素是Kerberos凭据的配置，然后由每个组件显式使用。

内部体系结构基于`org.apache.flink.runtime.security.modules.SecurityModule`在启动时安装的安全模块（实现）。以下各节介绍每个安全模块。

### Hadoop安全模块

该模块使用Hadoop UserGroupInformation \(UGI\)类来建立一个进程范围的登录用户上下文。然后，登录用户用于与Hadoop的所有交互，包括HDFS、HBase和YARN。

如果启用了Hadoop安全性\(在`core-site.xml`中\)，那么登录用户将拥有所配置的任何Kerberos凭证。否则，登录用户只传递启动集群的OS帐户的用户标识。

### JAAS安全模块

该模块为集群提供了动态的JAAS配置，使配置的Kerberos凭据可供ZooKeeper，Kafka和其他依赖JAAS的组件使用。

请注意，用户还可以使用[Java SE文档中](http://docs.oracle.com/javase/7/docs/technotes/guides/security/jgss/tutorials/LoginConfigFile.html)描述的机制来提供静态JAAS配置文件。静态配置会覆盖此模块提供的任何动态配置。

### ZooKeeper安全模块

 此模块配置某些与流程范围相关的ZooKeeper与安全性相关的设置，即ZooKeeper服务名称（默认值：`zookeeper`）和JAAS登录上下文名称（默认值：）`Client`。

## 部署方式

以下是一些特定于每种部署模式的信息。

### 独立模式

在独立/群集模式下运行安全Flink群集的步骤：

1. 将安全性相关的配置选项添加到Flink配置文件（在所有群集节点上）（请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#kerberos-based-security)）。
2. 确保keytab文件存在于所有群集节点上指定的`security.kerberos.login.keytab`路径上。
3. 正常部署Flink群集。

### YARN / Mesos模式

在YARN / Mesos模式下运行安全Flink群集的步骤：

1. 将安全性相关的配置选项添加到客户端上的Flink配置文件中（请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#kerberos-based-security)）。
2. 确保密钥表文件位于客户端节点上指定的`security.kerberos.login.keytab`路径上。
3. 正常部署Flink群集。

在YARN / Mesos模式下，密钥表会自动从客户端复制到Flink容器。

有关更多信息，请参见[YARN安全](https://github.com/apache/hadoop/blob/trunk/hadoop-yarn-project/hadoop-yarn/hadoop-yarn-site/src/site/markdown/YarnApplicationSecurity.md)文档。

#### **使用kinit（仅YARN）**

在YARN模式下，可以仅使用票证缓存（由管理`kinit`）来部署没有密钥表的安全Flink集群。这避免了生成密钥表的复杂性，并避免了将密钥表委托给集群管理器。在这种情况下，Flink CLI获取Hadoop委托令牌（用于HDFS和HBase）。主要缺点是，由于生成的委派令牌将过期（通常在一周内），因此群集的寿命必然很短。

使用以下步骤运行安全的Flink群集的步骤`kinit`：

1. 将安全性相关的配置选项添加到客户端上的Flink配置文件中（请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#kerberos-based-security)）。
2. 使用`kinit`命令登录。
3. 正常部署Flink群集。

## 更多详情

### Ticket **续签**

每个使用Kerberos的组件都独立负责更新Kerberos ticket-granting-ticket（TGT）。当提供密钥表时，Hadoop，ZooKeeper和Kafka都会自动更新TGT。在委派令牌场景中，YARN本身会更新令牌（直至其最大使用寿命）。

