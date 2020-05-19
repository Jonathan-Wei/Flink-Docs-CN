# Yarn

## 快速启动\(QuickStart\)

### 在YARN上启动长时间运行的Flink集群

启动一个包含4个TaskManager的YarnSession\(每个具有4 GB Heapspace\)

```bash
# get the hadoop2 package from the Flink download page at
# http://flink.apache.org/downloads.html
curl -O <flink_hadoop2_download_url>
tar xvzf flink-1.8-SNAPSHOT-bin-hadoop2.tgz
cd flink-1.8-SNAPSHOT/
./bin/yarn-session.sh -n 4 -jm 1024m -tm 4096m
```

`-s`指定每个任务管理器的处理槽数。建议将插槽数设置为每台计算机的处理器数。

会话启动后，可以使用该`./bin/flink`工具将作业提交到群集运行。

### 在YARN上运行Flink作业

```bash
# get the hadoop2 package from the Flink download page at
# http://flink.apache.org/downloads.html
curl -O <flink_hadoop2_download_url>
tar xvzf flink-1.8-SNAPSHOT-bin-hadoop2.tgz
cd flink-1.8-SNAPSHOT/
./bin/flink run -m yarn-cluster -yn 4 -yjm 1024m -ytm 4096m ./examples/batch/WordCount.jar
```

## Flink YARN Session

Apache [Hadoop YARN](http://hadoop.apache.org/)是一个集群资源管理框架。它允许在群集之上运行各种分布式应用程序。如果已经有YARN设置，用户不必设置或安装任何东西。

**要求**

* 至少Apache Hadoop 2.2
* HDFS（Hadoop分布式文件系统）（或Hadoop支持的其他分布式文件系统）

如果您在使用Flink YARN客户端时遇到麻烦，请查看[FAQ部分](http://flink.apache.org/faq.html#yarn-deployment)。

### 启动Flink会话

请按照以下说明了解如何在YARN群集中启动Flink会话。

会话将启动所有必需的Flink服务（JobManager和TaskManagers），以便您可以将程序提交到群集。请注意，您可以在每个会话中运行多个程序。

#### **下载Flink**

从[下载页面下载](http://flink.apache.org/downloads.html) Hadoop&gt; = 2的Flink软件包。它包含所需的文件。

使用以下方法解压缩包：

```text
tar xvzf flink-1.8-SNAPSHOT-bin-hadoop2.tgz
cd flink-1.8-SNAPSHOT/
```

#### **开始一个会话**

使用以下命令启动会话

```text
./bin/yarn-session.sh
```

此命令将显示以下概述：

```text
Usage:
   Required
     -n,--container <arg>   Number of YARN container to allocate (=Number of Task Managers)
   Optional
     -D <arg>                        Dynamic properties
     -d,--detached                   Start detached
     -jm,--jobManagerMemory <arg>    Memory for JobManager Container with optional unit (default: MB)
     -nm,--name                      Set a custom name for the application on YARN
     -q,--query                      Display available YARN resources (memory, cores)
     -qu,--queue <arg>               Specify YARN queue.
     -s,--slots <arg>                Number of slots per TaskManager
     -tm,--taskManagerMemory <arg>   Memory per TaskManager Container with optional unit (default: MB)
     -z,--zookeeperNamespace <arg>   Namespace to create the Zookeeper sub-paths for HA mode
```

请注意，客户端要求`YARN_CONF_DIR`或`HADOOP_CONF_DIR`环境变量设置为读取Yarn和HDFS配置。

**示例：**执行以下命令分配10个任务管理器，每个管理器具有8 GB内存和32个处理插槽：

```text
./bin/yarn-session.sh -n 10 -tm 8192 -s 32
```

系统将使用`conf/flink-conf.yaml`中的配置。如果您想更改某些内容，请按照我们的[配置指南操作](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html)。

YARN上的Flink将覆盖以下配置参数`jobmanager.rpc.address`（因为JobManager总是在不同的机器上分配），`taskmanager.tmp.dirs`（我们使用YARN给出的tmp目录）以及`parallelism.default`是否已指定插槽数。

如果不想更改配置文件以设置配置参数，则可以选择通过`-D`标志传递动态属性。例如：`-Dfs.overwrite-files=true -Dtaskmanager.network.memory.min=536346624`。

示例调用启动了11个容器（即使只请求了10个容器），因为ApplicationMaster和Job Manager还有一个额外的容器。

在YARN群集中部署Flink后，它将显示作业管理器的连接详细信息。

通过停止unix进程（使用CTRL + C）或在客户端输入“stop”来停止YARN会话。

如果群集上有足够的资源，则YARN上的Flink将仅启动所有请求的容器。大多数YARN调度程序考虑了所请求的容器内存，也考虑了vcores的数量。默认情况下，vcores的数量等于处理slots（`-s`）参数。在[`yarn.containers.vcores`](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html#yarn-containers-vcores)允许覆盖vcores的数量与自定义值。为了使此参数起作用，您应该在群集中启用CPU调度。

#### **分离的YARN会话**

如果您不希望Flink YARN客户端始终保持运行，则还可以启动_分离的_ YARN会话。该参数称为`-d`或`--detached`。

在这种情况下，Flink YARN客户端将仅向群集提交Flink，然后自行关闭。请注意，在这种情况下，无法使用Flink停止YARN会话。

使用YARN实用程序（`yarn application -kill <appId>`）来停止YARN会话。

#### **附加到现有会话**

使用以下命令启动会话

```text
./bin/yarn-session.sh
```

此命令将显示以下概述：

```text
Usage:
   Required
     -id,--applicationId <yarnAppId> YARN application Id
```

如前所述，必须设置环境变量`YARN_CONF_DIR`或者`HADOOP_CONF_DIR`以读取YARN和HDFS配置。

**示例：**执行以下命令附加到正在运行的Flink YARN会话`application_1463870264508_0029`：

```text
./bin/yarn-session.sh -id application_1463870264508_0029
```

附加到正在运行的会话使用YARN ResourceManager来确定作业管理器RPC端口。

通过停止unix进程（使用CTRL + C）或在客户端输入“stop”来停止YARN会话。

### 向Flink提交工作

使用以下命令将Flink程序提交到YARN群集：

```text
./bin/flink
```

请参阅[命令行客户端](https://ci.apache.org/projects/flink/flink-docs-master/ops/cli.html)的文档。

该命令将显示如下的帮助菜单：

```text
[...]
Action "run" compiles and runs a program.

  Syntax: run [OPTIONS] <jar-file> <arguments>
  "run" action arguments:
     -c,--class <classname>           Class with the program entry point ("main"
                                      method or "getPlan()" method. Only needed
                                      if the JAR file does not specify the class
                                      in its manifest.
     -m,--jobmanager <host:port>      Address of the JobManager (master) to
                                      which to connect. Use this flag to connect
                                      to a different JobManager than the one
                                      specified in the configuration.
     -p,--parallelism <parallelism>   The parallelism with which to run the
                                      program. Optional flag to override the
                                      default value specified in the
                                      configuration
```

使用_run_操作将作业提交给YARN。客户端能够确定JobManager的地址。在极少数问题的情况下，还可以使用`-m`参数传递JobManager地址。JobManager地址在YARN控制台中可见。

**例**

```text
wget -O LICENSE-2.0.txt http://www.apache.org/licenses/LICENSE-2.0.txt
hadoop fs -copyFromLocal LICENSE-2.0.txt hdfs:/// ...
./bin/flink run ./examples/batch/WordCount.jar \
        hdfs:///..../LICENSE-2.0.txt hdfs:///.../wordcount-result.txt
```

如果出现以下错误，请确保所有TaskManagers都已启动：

```text
Exception in thread "main" org.apache.flink.compiler.CompilerException:
    Available instances could not be determined from job manager: Connection timed out.
```

您可以在JobManager Web界面中检查TaskManagers的数量。该接口的地址打印在YARN会话控制台中。

如果一分钟后TaskManagers没有显示，您应该使用日志文件调查问题。

## 在YARN上运行单个Flink作业

上面的文档描述了如何在Hadoop YARN环境中启动Flink集群。也可以仅在执行单个作业时在YARN中启动Flink。

请注意，客户端希望设置-yn值（任务管理器的数量）。

_**例：**_

```text
./bin/flink run -m yarn-cluster -yn 2 ./examples/batch/WordCount.jar
```

Yarn会话的命令行选项也可以用于./bin/flink。使用y或yarn\(对于长参数选项\)作为前缀。

{% hint style="info" %}
注意：通过设置环境变量FLINK\_CONF\_DIR，可以对每个作业使用不同的配置目录。要使用此命令，请从flink分发版复制conf目录，并修改每个作业的日志设置。
{% endhint %}

{% hint style="info" %}
注意:可以将-m yarn-cluster与分离的Yarn提交\(-yd\)结合使用，以“解除并忘记”对Yarn集群的Flink作业。在这种情况下，应用程序不会从ExecutionEnvironment.execute\(\)调用中获得任何累加器结果或异常!
{% endhint %}

### 用户jar和Classpath

默认情况下，Flink将在运行单个作业时将用户jar包含到系统类路径中。可以使用`yarn.per-job-cluster.include-user-jar`参数控制此行为。

Flink将此设置为`DISABLED`时，将在用户类路径中包含jar。

可以通过将参数设置为以下之一来控制类路径中的user-jar位置：

* `ORDER`:\(默认）根据字典顺序将jar添加到系统类路径。
* `FIRST`：将jar添加到系统类路径的开头。
* `LAST`：将jar添加到系统类路径的末尾。

## Flink在YARN上的恢复行为

Flink的纱线客户端具有以下配置参数，用于控制容器发生故障时的行为。这些参数可以从`conf/flink-conf`中设置。或在开始纱线会话时，使用`-D`参数。

* `yarn.reallocate-failed`：此参数控制Flink是否应重新分配失败的TaskManager容器。默认值：`true`
* `yarn.maximum-failed-containers`：ApplicationMaster在YARN会话失败之前接受的最大失败容器数。默认值：最初请求的TaskManagers（`-n`）的数量。
* `yarn.application-attempts`：ApplicationMaster（+其TaskManager容器）尝试的数量。如果此值设置为1（默认值），则当应用程序主服务器失败时，整个YARN会话将失败。较高的值指定YARN重新启动ApplicationMaster的次数。

## 调试失败的YARN会话

Flink YARN会话部署失败的原因有很多。配置错误的Hadoop设置（HDFS权限，YARN配置），版本不兼容（在Cloudera Hadoop上运行Flink与vanilla Hadoop依赖关系）或其他错误。

### 日志文件

如果Flink YARN会话在部署期间失败，则用户必须依赖Hadoop YARN的日志记录功能。最有用的功能是[YARN日志聚合](http://hortonworks.com/blog/simplifying-user-logs-management-and-access-in-yarn/)。要启用它，用户必须将`yarn-site.xml`文件中的`yarn.log-aggregation-enable`属性设置为`true`。启用后，用户可以使用以下命令检索（失败的）YARN会话的所有日志文件。

```text
yarn logs -applicationId <application ID>
```

{% hint style="info" %}
请注意，会话结束后需要几秒钟才会显示日志。
{% endhint %}

### YARN客户端控制台和Web界面

如果在运行期间发生错误，Flink YARN客户端还会在终端中输出错误消息（例如，如果TaskManager在一段时间后停止工作）。

除此之外，还有YARN Resource Manager Web界面（默认情况下在端口8088上）。资源管理器Web界面的端口由`yarn.resourcemanager.webapp.address`配置值确定。

它允许访问日志文件以运行YARN应用程序，并显示失败应用程序的诊断。

## 为特定的Hadoop版本构建YARN客户端

使用来自Hortonworks，Cloudera或MapR等公司的Hadoop发行版的用户可能必须针对其特定版本的Hadoop（HDFS）和YARN构建Flink。有关更多详细信息，请阅读[构建说明](https://ci.apache.org/projects/flink/flink-docs-master/flinkDev/building.html)。

## 在防火墙后面的YARN上运行Flink

某些YARN群集使用防火墙来控制群集与网络其余部分之间的网络流量。在这些设置中，Flink作业只能从群集网络内（防火墙后面）提交到YARN会话。如果这对于生产使用不可行，Flink允许为所有相关服务配置端口范围。通过配置这些范围，用户还可以通过防火墙向Flink提交作业。

目前，提交工作需要两项服务：

* JobManager（YARN中的ApplicationMaster）
* 在JobManager中运行的BlobServer。

向Flink提交作业时，BlobServer会将带有用户代码的jar分发给所有工作节点（TaskManagers）。JobManager自己接收作业并触发执行。

用于指定端口的两个配置参数如下：

* `yarn.application-master.port`
* `blob.server.port`

这两个配置选项接受单个端口\(例如：“50010”\)，范围\(“50000-50025”\)或两者的组合\(“50010,50011,50020-50025,50050-50075”\)。

\(Hadoop使用类似的机制，在这里调用配置参数`yarn.app.mapreduce.am.job.client.port-range`\)

## 背景/**内幕**

本节简要介绍Flink和YARN如何交互。

![](../../.gitbook/assets/image%20%2824%29.png)

YARN客户端需要访问Hadoop配置以连接到YARN资源管理器和HDFS。它使用以下策略确定Hadoop配置：

* 测试是否`YARN_CONF_DIR`，`HADOOP_CONF_DIR`或`HADOOP_CONF_PATH`设置\(按顺序\)。如果设置了其中一个变量，则用于读取配置。
* 如果上述策略失败\(在正确的YARN设置中不应该这样\)，则客户端正在使用`HADOOP_HOME`环境变量。如果已设置，则客户端尝试访问`$HADOOP_HOME/etc/hadoop`\(Hadoop 2\)和`$HADOOP_HOME/conf`\(Hadoop 1\)。

在启动新的Flink YARN会话时，客户端首先检查所请求的资源（容器和内存）是否可用。之后，它将包含Flink和配置的jar上传到HDFS\(步骤1\)。

客户端的下一步是请求\(步骤2\)YARN容器以启动_ApplicationMaster\(_步骤3\)。由于客户端将配置和jar文件注册为容器的资源，因此在该特定机器上运行的YARN的NodeManager将负责准备容器\(例如，下载文件\)。完成后，将启动_ApplicationMaster\(_AM\)。

_JobManager_和AM在同一容器中运行。一旦它们成功启动，AM就知道JobManager\(自己的主机\)的地址。它为TaskManagers生成一个新的Flink配置文件\(以便可以连接到JobManager\)。该文件也上传到HDFS。此外，_AM_容器还提供Flink的Web界面。YARN代码分配的所有端口都是_临时端口_。这允许用户并行执行多个Flink YARN会话。

之后，AM开始为Flink的TaskManagers分配容器，这将从HDFS下载jar文件和修改后的配置。完成这些步骤后，即可建立Flink并准备接受作业。

