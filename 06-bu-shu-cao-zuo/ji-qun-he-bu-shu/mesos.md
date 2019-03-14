# Mesos

## 背景

Mesos实现包含两个组件：Application Master和Worker。Worker是简单的TaskManagers，它们由Application Master设置的环境进行参数化。Mesos实现中最复杂的组件是Application Master。Application Master当前托管以下组件：

### Mesos调度程序

调度程序负责向Mesos框架注册，请求资源和启动工作节点。调度程序需要不断向Mesos报告以确保框架处于健康状态。为了验证集群的运行状况，调度程序监视生成的worker并将其标记为失败，并在必要时重新启动它们。

目前，Flink的Mesos调度程序本身不具备高可用性。但是，它会在Zookeeper中保留有关其状态（例如配置，工作者列表）的所有必要信息。在出现故障时，它依赖于外部系统来启动新的调度程序。然后，调度程序将再次向Mesos注册并完成调整阶段。在协调阶段，调度程序接收正在运行的工作程序节点的列表。它将这些与Zookeeper中恢复的信息进行匹配，并确保在发生故障之前将群集恢复到状态。

### 工件服务器\(Artifact Server\)

工件服务器负责向工作节点提供资源。资源可以是任何东西，从Flink二进制文件到共享机密或配置文件。例如，在非容器化环境中，工件服务器将提供Flink二进制文件。将提供哪些文件取决于使用的配置覆盖。

### Flink的调度程序和Web界面

调度程序和web接口提供用于监测，作业提交，以及与该集群的其他客户端的交互的中心点（参见[FLIP-6 ](https://cwiki.apache.org/confluence/pages/viewpage.action?pageId=65147077)）。

### 启动脚本和配置叠加层

启动脚本提供了一种配置和启动Application Master的方法。然后，Worker继承所有进一步的配置。这是使用配置叠加来实现的。配置叠加提供了一种从环境变量和配置文件推断配置的方法，这些配置文件将发送到工作节点。

## DC/OS

本节涉及[DC/OS](https://dcos.io/)，它是具有复杂应用程序管理层的Mesos分发。它预装了Marathon，这是一项监督应用程序并在发生故障时维持状态的服务。

如果您没有正在运行的DC/OS群集，请按照[如何在官方网站上安装DC/OS](https://dcos.io/install/)的 [说明进行操作](https://dcos.io/install/)。

拥有DC/OS群集后，您可以通过DC/OS Universe安装Flink。在搜索提示中，只需搜索Flink。或者，您可以使用DC/OS CLI：

```text
dcos package install flink
```

更多信息可以在 [DC/OS示例文档中找到](https://github.com/dcos/examples/tree/master/1.8/flink)。

## 没有DC / OS的Mesos

你也可以在没有DC / OS的情况下运行Mesos。

### 安装Mesos

请按照[官方网站上如何设置Mesos](http://mesos.apache.org/getting-started/)的[说明进行操作](http://mesos.apache.org/getting-started/)。

安装后，必须通过创建文件`MESOS_HOME/etc/mesos/masters`和配置主节点和代理节点集`MESOS_HOME/etc/mesos/slaves`。这些文件在每行中包含一个主机名，相应的组件将在该主机名上启动（假设SSH访问这些节点）。

接下来，必须创建`MESOS_HOME/etc/mesos/mesos-master-env.sh`或使用在同一目录中找到的模板。在此文件中，必须定义

```text
export MESOS_work_dir=WORK_DIRECTORY
```

并建议取消注册

```text
export MESOS_log_dir=LOGGING_DIRECTORY
```

要配置Mesos代理，必须创建`MESOS_HOME/etc/mesos/mesos-agent-env.sh`或使用在同一目录中找到的模板。必须配置

```text
export MESOS_master=MASTER_HOSTNAME:MASTER_PORT
```

并取消注释

```text
export MESOS_log_dir=LOGGING_DIRECTORY
export MESOS_work_dir=WORK_DIRECTORY
```

#### **Mesos库**

要使用Mesos运行Java应用程序，必须`MESOS_NATIVE_JAVA_LIBRARY=MESOS_HOME/lib/libmesos.so`在Linux 上导出。在Mac OS X下，必须导出`MESOS_NATIVE_JAVA_LIBRARY=MESOS_HOME/lib/libmesos.dylib`。

#### **部署Mesos**

要启动**M**esos群集，请使用部署脚本`MESOS_HOME/sbin/mesos-start-cluster.sh`。要停止Mesos群集，请使用部署脚本`MESOS_HOME/sbin/mesos-stop-cluster.sh`。可以在[此处](http://mesos.apache.org/documentation/latest/deploy-scripts/)找到有关部署脚本的更多信息。

### 安装Marathon

您还可以选择[安装Marathon](https://mesosphere.github.io/marathon/docs/)，它使您能够在[高可用性\(HA\)模式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/mesos.html#high-availability)下运行Flink。

### 预安装Flink与Docker / Mesos容器

可以在所有Mesos主节点和代理节点上安装Flink。还可以在部署期间从Flink网站提取二进制文件，并在启动应用程序主服务器之前应用自定义配置。更方便，更易于维护的方法是使用Docker容器来管理Flink二进制文件和配置。

这可以通过以下配置条目进行控制：

```text
mesos.resourcemanager.tasks.container.type: mesos _or_ docker
```

如果设置为“docker”，请指定图像名称：

```text
mesos.resourcemanager.tasks.container.image.name: image_name
```

### Standalone（独立）

在Flink发行版的`/bin`目录中，可以找到两个启动脚本来管理Mesos集群中的Flink进程：

1. `mesos-appmaster.sh` 用于启动Mesos Application Master，它将注册Mesos调度程序以及负责启动工作节点。
2. `mesos-taskmanager.sh` Mesos工作进程的入口点。不需要显式执行此脚本。该脚本由Mesos工作节点自动启动以启动新的TaskManager。

为了运行`mesos-appmaster.sh`脚本，必须在`flink-conf.yaml`或通过`-Dmesos.master=...`定义`mesos.master`在并将其传递给Java进程。

执行时`mesos-appmaster.sh`，将在你执行脚本的机器上创建一个TaskManager。与此相反，TaskManager将作为Mesos集群中的Mesos任务运行。  


**一般配置**

可以通过传递给Mesos Application Master的Java属性完全参数化Mesos应用程序。也允许指定一般的Flink配置参数。例如：

```bash
bin/mesos-appmaster.sh \
    -Dmesos.master=master.foobar.org:5050 \
    -Djobmanager.heap.mb=1024 \
    -Djobmanager.rpc.port=6123 \
    -Drest.port=8081 \
    -Dmesos.resourcemanager.tasks.mem=4096 \
    -Dtaskmanager.heap.mb=3500 \
    -Dtaskmanager.numberOfTaskSlots=2 \
    -Dparallelism.default=10
```

### 高可用性

你将需要运行Marathon或Apache Aurora之类的服务，该服务负责在节点或进程出现故障时重新启动Flink主进程。此外，还需要像[Flink文档](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/jobmanager_high_availability.html)的“ [高可用性”部分中所述](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/jobmanager_high_availability.html)配置Zookeeper 。

#### **Marathon**

需要设置Marathon才能启动`bin/mesos-appmaster.sh`脚本。特别是，它还应调整Flink群集的任何配置参数。

以下是Marathon的示例配置：

```text
{
    "id": "flink",
    "cmd": "$FLINK_HOME/bin/mesos-appmaster.sh -Djobmanager.heap.mb=1024 -Djobmanager.rpc.port=6123 -Drest.port=8081 -Dmesos.resourcemanager.tasks.mem=1024 -Dtaskmanager.heap.mb=1024 -Dtaskmanager.numberOfTaskSlots=2 -Dparallelism.default=2 -Dmesos.resourcemanager.tasks.cpus=1",
    "cpus": 1.0,
    "mem": 1024
}
```

使用Marathon运行Flink时，包含作业管理器的整个Flink集群将作为Mesos集群中的Mesos任务运行。

### 配置参数

有关Mesos特定配置的列表，请参阅 配置文档的[Mesos部分](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html#mesos)。

###  

