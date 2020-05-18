---
description: 本页面提供了如何在静态(但可能是异构)集群上以完全分布式方式运行Flink的说明
---

# 独立集群

## 要求

### 软件要求

Flink可在所有_类UNIX环境中运行_，例如**Linux**，**Mac OS X**和**Cygwin**（适用于Windows），并期望群集由**一个主节点**和**一个或多个工作节点组成**。在开始设置系统之前，请确保**在每个节点上**安装了以下软件：

* **Java 1.8.x**或更高版本，
* **ssh**（必须运行sshd才能使用管理远程组件的Flink脚本）

如果您的群集不满足这些软件要求，则需要安装/升级它。

在所有群集节点上使用无**密码SSH**和 **相同的目录结构**将允许您使用我们的脚本来控制所有内容。

### JAVA\_HOME配置

Flink要求在`JAVA_HOME`主节点和所有工作节点上设置环境变量，并指向Java安装的目录。

您可以`conf/flink-conf.yaml`通过`env.java.home`键设置此变量。

## Flink配置

转到[下载页面](https://flink.apache.org/downloads.html)并获取准备运行的软件包。确保选择**与您的Hadoop版本匹配**的Flink软件包。如果您不打算使用Hadoop，请选择任何版本。

下载最新版本后，将存档复制到您的主节点并解压缩它：

```text
tar xzf flink-*.tgz
cd flink-*
```

### 配置Flink

在提取系统文件之后，您需要通过编辑`conf/Flink -conf.yaml`来为集群配置Flink。

 通过设置`jobmanager.rpc.address`指向主节点。通过设置`jobmanager.heap.size`和`taskmanager.memory.process.size`来定义允许Flink在每个节点上分配的最大主内存量。

 这些值以MB为单位。如果某些工作节点有更多主内存要分配给Flink系统，则可以通过在这些特定节点上设置`taskmanager.memory.process.size`或`taskmanager.memory.flink.size`在_`conf/flink-conf.yaml`中_覆盖默认值。

最后，必须提供集群中作为工作节点的所有节点的列表。因此，类似于HDFS配置，编辑文件conf/slaves并输入每个工作节点的IP/主机名。每个工作节点稍后将运行一个TaskManager。

 以下示例说明了具有三个节点（IP地址从_10.0.0.1_ 到_10.0.0.3_且主机名_master_，_worker1_，_worker2_）的设置，并显示了配置文件的内容（需要在所有计算机上的同一路径上进行访问） ）：

![](../../.gitbook/assets/image%20%2830%29.png)



/ path / to / **flink / conf /  
flink-conf.yaml**

```text
jobmanager.rpc。地址：10.0.0.1
```

/ path / to / **flink /  
conf / slaves**

```text
10.0.0.2
10.0.0.3
```

Flink目录必须在相同路径下的每个worker上可用。您可以使用共享的NFS目录，或者将整个Flink目录复制到每个工作节点。

 请参阅[配置页面](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html)以获取详细信息和其他配置选项。

特别是

* 每个JobManager的可用内存量（`jobmanager.heap.size`），
* 每个TaskManager的可用内存量（`taskmanager.memory.process.size`并查看[内存设置指南](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/memory/mem_tuning.html#configure-memory-for-standalone-deployment)），
* 每台计算机可用的CPU数（`taskmanager.numberOfTaskSlots`），
* 集群中的CPU总数（`parallelism.default`）和
* 临时目录（`io.tmp.dirs`）

是非常重要的配置值。

### 启动Flink

以下脚本在本地节点上启动JobManager，并通过SSH连接到_slaves_文件中列出的所有辅助节点，以在每个节点上启动TaskManager。现在，您的Flink系统已启动并正在运行。现在，在本地节点上运行的JobManager将在配置的RPC端口上接受作业。

假设您位于主节点上，并且位于Flink目录中：

```text
bin/start-cluster.sh
```

要停止Flink，还有一个`stop-cluster.sh`脚本。

### 添加JobManager/TaskManager实例到集群

您可以使用`bin/jobmanager.sh`和`bin/taskmanager.sh`脚本将JobManager和TaskManager实例都添加到正在运行的集群中。

**添加JobManager**

```text
bin/jobmanager.sh ((start|start-foreground) [host] [webui-port])|stop|stop-all
```

**添加任务管理器**

```text
bin/taskmanager.sh start|start-foreground|stop|stop-all
```

确保在要启动/停止相应实例的主机上调用这些脚本。

