# Flink 操作游乐园

我们有多种方法可以在各种环境中部署和操作Apache Flink。不管这种多样性如何，Flink群集的基本构建块都相同，并且适用类似的操作原理。

在这个游乐园中，您将学习如何管理和运行Flink Jobs。您将看到如何部署和监视应用程序，体验Flink如何从作业失败中恢复，以及执行日常操作任务，例如升级和缩放。

## 这个游乐场的解剖

 这个游乐场是由一个生命周期较长的 [Flink会话群集](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/glossary.html#flink-session-cluster)和一个Kafka群集组成。

 Flink群集通常由 [Flin](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/glossary.html#flink-master)Master和一个或多个 [Flink TaskManager组成](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/glossary.html#flink-taskmanager)。 Flink Master负责处理[作业](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/glossary.html#flink-job)提交，作业的监督以及资源管理。Flink TaskManager是工作进程，负责执行构成Flink作业的实际 [任务](https://ci.apache.org/projects/flink/flink-docs-release-1.10/concepts/glossary.html#task)。 在这个游乐园中，您将从单个TaskManager开始，但以后可以扩展到更多的TaskManager。此外，该游乐场还带有一个专用的_客户端_容器，我们可以使用该容器初始化提交Flink作业，并在之后执行各种操作任务。该_客户端_ Flink群集本身不需要容器，而是仅包含容器以方便使用。

Kafka群集由Zookeeper服务器和Kafka代理组成。

![](../.gitbook/assets/image%20%2810%29.png)

 当游乐场启动时，名为_Flink Event Count_的Flink作业将被提交给Flink主机。此外，还将创建两个Kafka Topic_输入_和_输出_。

![](../.gitbook/assets/image%20%2838%29.png)

 Job `ClickEvent`从_输入Topic_中消费数据 ，每个主题都有一个`timestamp`和一个`page`。然后由15秒[窗口](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/stream/operators/windows.html)键入事件并进行计数 `page`。结果将写入 _输出_Topic。

有六个不同的页面，我们每页面15秒生成1000次点击事件。因此，Flink作业的输出应在每个页面和窗口显示1000个视图。

## 开启游乐场

只需几个步骤即可设置游乐场环境。我们将引导您完成必要的命令，并演示如何验证一切是否正常运行。

我们假设您在计算机上安装了[Docker](https://docs.docker.com/)（1.12+）和docker [-compose](https://docs.docker.com/compose/)（2.1+）。

 所需的配置文件在[flink-playgrounds](https://github.com/apache/flink-playgrounds)存储库中可用 。检查一下并优化环境：

```bash
git clone --branch release-1.10 https://github.com/apache/flink-playgrounds.git
cd flink-playgrounds/operations-playground
docker-compose build
docker-compose up -d
```

之后，您可以使用以下命令检查正在运行的Docker容器：

```text
docker-compose ps

                    Name                                  Command               State                   Ports                
-----------------------------------------------------------------------------------------------------------------------------
operations-playground_clickevent-generator_1   /docker-entrypoint.sh java ...   Up       6123/tcp, 8081/tcp                  
operations-playground_client_1                 /docker-entrypoint.sh flin ...   Exit 0                                       
operations-playground_jobmanager_1             /docker-entrypoint.sh jobm ...   Up       6123/tcp, 0.0.0.0:8081->8081/tcp    
operations-playground_kafka_1                  start-kafka.sh                   Up       0.0.0.0:9094->9094/tcp              
operations-playground_taskmanager_1            /docker-entrypoint.sh task ...   Up       6123/tcp, 8081/tcp                  
operations-playground_zookeeper_1              /bin/sh -c /usr/sbin/sshd  ...   Up       2181/tcp, 22/tcp, 2888/tcp, 3888/tcp
```

这表明客户端容器已成功提交Flink Job（`Exit 0`），并且所有群集组件以及数据生成器都在运行（`Up`）。

您可以通过以下方式停止游乐场环境：

```text
docker-compose down -v
```

## 进入游乐场

您可以在这个游乐场尝试多种尝试。在以下两个部分中，我们将向您展示如何与Flink群集进行交互，并演示Flink的一些关键功能。

### Flink WebUI

 观察Flink群集的最自然的起点是在[http//localhost:8081](http://localhost:8081/)下公开的WebUI 。如果一切顺利，您将看到集群最初由一个TaskManager组成，并执行一个名为_Click Event Count_的Job 。

![](../.gitbook/assets/image%20%2822%29.png)

Flink WebUI包含有关Flink群集及其作业的许多有用和有趣的信息（JobGraph，指标，检查点统计信息，TaskManager状态等等）。

### 日志

#### JobManager

 JobManager日志可以通过`docker-compose`添加.

```text
docker-compose logs -f jobmanager
```

初始启动后，您应该主要看到每个检查点完成的日志消息。

#### TaskManager

TaskManager日志可以用相同的方式尾注

```text
docker-compose logs -f taskmanager
```

初始启动后，您应该主要看到每个检查点完成的日志消息。

### Flink CLI

 所述[**Flink** CLI](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/cli.html)可以从客户端容器内使用。例如，要打印`help`Flink CLI 的消息，您可以运行

```text
docker-compose run --no-deps client flink --help
```

### Flink REST API

 该[Flink REST API](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/rest_api.html#api)通过 `localhost:8081`主机或通过`jobmanager:8081`从客户端容器上暴露，例如以列出当前所有正在运行的作业，你可以运行：

```text
curl localhost:8081/jobs
```

### Kafka Topic

您可以通过运行以下命令查看写入“ Kafka Topic”的记录

```text
//input topic (1000 records/s)
docker-compose exec kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 --topic input

//output topic (24 records/min)
docker-compose exec kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 --topic output
```

## 玩的时间到了！

 既然您已经了解了如何与Flink和Docker容器进行交互，那么让我们看一些可以在我们的操场上尝试的常见操作任务。所有这些任务彼此独立，即，您可以按任何顺序执行它们。大多数任务可以通过[CLI](https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/getting-started/docker-playgrounds/flink-operations-playground.html#flink-cli)和[REST API执行](https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/getting-started/docker-playgrounds/flink-operations-playground.html#flink-rest-api)。

### 列出正在运行的作业

{% tabs %}
{% tab title="命令行界面" %}
 **命令**

```text
docker-compose run --no-deps client flink list
```

 **预期显示**

```text
Waiting for response...
------------------ Running/Restarting Jobs -------------------
16.07.2019 16:37:55 : <job-id> : Click Event Count (RUNNING)
--------------------------------------------------------------
No scheduled jobs.
```
{% endtab %}

{% tab title="REST API" %}
 **请求**

```text
curl localhost:8081/jobs
```

 **预期响应（打印精美）**

```text
{
  "jobs": [
    {
      "id": "<job-id>",
      "status": "RUNNING"
    }
  ]
}
```
{% endtab %}
{% endtabs %}

JobID在提交时已分配给作业，并且需要它通过CLI或REST API对作业执行操作。

### 观察故障与恢复

Flink在（部分）故障下提供一次精确的处理保证。在这个游乐园中，您可以观察并（在某种程度上）验证此行为。

#### **步骤1：观察输出**

 如上所述，这个游乐园产生事件，使得每个窗口只包含1000个记录。因此，为了验证Flink是否已成功从TaskManager故障中恢复而没有数据丢失或重复，您可以跟踪输出主题并检查-恢复后-所有窗口均存在并且计数是否正确。

为此，请从_输出Topic_开始阅读，并使该命令运行直到恢复为止（步骤3）。

```text
docker-compose exec kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 --topic output
```

**步骤2：引入故障**

为了模拟部分故障，您可以杀死TaskManager。在生产设置中，这可能对应于TaskManager进程的丢失，TaskManager计算机的丢失或仅是从框架或用户代码引发的临时异常（例如，由于外部资源的暂时不可用）。

```text
docker-compose kill taskmanager
```

 几秒钟后，Flink Master将注意到TaskManager丢失，取消受影响的作业，然后立即重新提交以进行恢复。作业重新启动后，其任务保持`SCHEDULED`状态，由紫色正方形指示（请参见下面的屏幕截图）。

![](../.gitbook/assets/image%20%2839%29.png)

{% hint style="info" %}
**注意：**即使作业的任务处于SCHEDULED状态且尚未处于RUNNING状态，作业的整体状态仍显示为RUNNING。
{% endhint %}

 此时，作业的任务无法从`SCHEDULED`状态转移到`RUNNING`，因为没有资源（任务管理器提供的TaskSlot）来运行任务。在新的TaskManager可用之前，作业将经历一个取消和重新提交的循环。

 同时，数据生成器不断将`ClickEvents` 推入_输入_Topic。这类似于真实的生产设置，在该生产设置中，当要处理的Job停机时会生成数据。

#### **步骤3：复原**

重新启动TaskManager后，它将重新连接到Master。

```text
docker-compose up -d taskmanager
```

 当主服务器收到有关新TaskManager的通知时，它会将恢复作业的任务安排到新可用的TaskSlot。重新启动后，任务将从失败之前执行的最后一个成功[检查点](https://ci.apache.org/projects/flink/flink-docs-release-1.10/internals/stream_checkpointing.html)恢复其状态，并切换到该`RUNNING`状态。

 Job将快速处理来自Kafka的输入事件（在中断期间累积）的完整积压，并以更高的速率（&gt; 24个记录/分钟）产生输出，直到到达流的开头。在_输出中，_您将看到所有时间窗口都存在所有`page`键，并且每个计数正好是一千。由于我们 在“至少一次”模式下使用 [FlinkKafkaProducer](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/connectors/kafka.html#kafka-producers-and-fault-tolerance)，因此有机会看到一些重复的输出记录。

{% hint style="info" %}
**注意：**大多数生产设置都依赖于资源管理器（Kubernetes，Yarn，Mesos）来自动重启失败的进程。
{% endhint %}

### 升级和缩放作业

 升级Flink作业始终涉及两个步骤：首先，使用[Savepoint](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/savepoints.html)正常停止Flink作业 。SavePoint是在定义良好的全局一致时间点（类似于检查点）的完整应用程序状态的一致快照。其次，从保存点启动升级的Flink作业。在这种情况下，“升级”可能意味着不同的含义，包括：

* 升级配置（包括作业的并行性）
* 作业拓扑的升级（已添加/已删除的操作员）
* 作业的用户定义功能的升级

 在开始升级之前，您可能想要开始跟踪_输出_Topic，以观察到在升级过程中没有数据丢失或损坏。

```text
docker-compose exec kafka kafka-console-consumer.sh \
  --bootstrap-server localhost:9092 --topic output
```

**步骤1：停止作业**

 要正常停止作业，您需要使用CLI或REST API的“**stop**”命令。为此，您需要Job的JobID，您可以通过[列出所有正在运行](https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/getting-started/docker-playgrounds/flink-operations-playground.html#listing-running-jobs)的Job 或从WebUI 获得该ID 。使用JobID，您可以继续停止作业：

{% tabs %}
{% tab title="命令行" %}
 **命令**

```text
docker-compose run --no-deps client flink stop <job-id>
```

 **预期显示**

```text
Suspending job "<job-id>" with a savepoint.
Suspended job "<job-id>" with a savepoint.
```

 保存点已存储到_flink-conf.yaml中_的`state.savepoint.dir`配置中，该文件安装在本地计算机上的_/tmp/flink-savepoints-directory /下_。下一步将需要此保存点的路径。如果是REST API，则此路径已经是响应的一部分，您将需要直接查看文件系统。

**命令**

```text
ls -lia /tmp/flink-savepoints-directory
```

**预期显示**

```text
total 0
  17 drwxr-xr-x   3 root root   60 17 jul 17:05 .
   2 drwxrwxrwt 135 root root 3420 17 jul 17:09 ..
1002 drwxr-xr-x   2 root root  140 17 jul 17:05 savepoint-<short-job-id>-<uuid>
```
{% endtab %}

{% tab title="REST API" %}
**请求**

```text
# triggering stop
curl -X POST localhost:8081/jobs/<job-id>/stop -d '{"drain": false}'
```

**预期响应（打印精美）**

```text
{
  "request-id": "<trigger-id>"
}
```

**请求**

```text
# check status of stop action and retrieve savepoint path
 curl localhost:8081/jobs/<job-id>/savepoints/<trigger-id>
```

**预期响应（打印精美）**

```text
{
  "status": {
    "id": "COMPLETED"
  },
  "operation": {
    "location": "<savepoint-path>"
  }
```
{% endtab %}
{% endtabs %}

**步骤2a：无需更改即可重新启动作业**

您现在可以从此保存点重新启动升级的作业。为简单起见，您可以先重新启动它，而无需进行任何更改。

{% tabs %}
{% tab title="命令行" %}
**命令**

```text
docker-compose run --no-deps client flink run -s <savepoint-path> \
  -d /opt/ClickCountJob.jar \
  --bootstrap.servers kafka:9092 --checkpointing --event-time
```

**预期显示**

```text
Starting execution of program
Job has been submitted with JobID <job-id>
```
{% endtab %}

{% tab title="REST API" %}
**请求**

```text
# Uploading the JAR from the Client container
docker-compose run --no-deps client curl -X POST -H "Expect:" \
  -F "jarfile=@/opt/ClickCountJob.jar" http://jobmanager:8081/jars/upload
```

**预期响应（打印精美）**

```text
{
  "filename": "/tmp/flink-web-<uuid>/flink-web-upload/<jar-id>",
  "status": "success"
}
```

**请求**

```text
# Submitting the Job
curl -X POST http://localhost:8081/jars/<jar-id>/run \
  -d '{"programArgs": "--bootstrap.servers kafka:9092 --checkpointing --event-time", "savepointPath": "<savepoint-path>"}'
```

**预期响应（打印精美）**

```text
{
  "jobid": "<job-id>"
}
```
{% endtab %}
{% endtabs %}

 再次执行作业`RUNNING`后，您将在_输出_Topic中看到，在作业处理中断期间累积的积压工作时，记录的生成速度更高。此外，您会看到在升级过程中没有数据丢失：所有窗口都以一千个的数量存在。

#### **步骤2b：以不同的并行度重新启动作业（重新缩放）**

或者，您也可以通过在重新提交期间传递不同的并行度来从此保存点重新调整作业。

{% tabs %}
{% tab title="命令行" %}
**命令**

```text
docker-compose run --no-deps client flink run -p 3 -s <savepoint-path> \
  -d /opt/ClickCountJob.jar \
  --bootstrap.servers kafka:9092 --checkpointing --event-time
```

**预期显示**

```text
Starting execution of program
Job has been submitted with JobID <job-id>
```

现在，作业已被重新提交，但是由于没有足够的TaskSlots来执行并行性增强的任务槽（2个可用，需要3个），因此无法启动。用

```text
docker-compose scale taskmanager=2
```
{% endtab %}

{% tab title="REST API" %}
**请求**

```text
# Uploading the JAR from the Client container
docker-compose run --no-deps client curl -X POST -H "Expect:" \
  -F "jarfile=@/opt/ClickCountJob.jar" http://jobmanager:8081/jars/upload
```

**预期响应（打印精美）**

```text
{
  "filename": "/tmp/flink-web-<uuid>/flink-web-upload/<jar-id>",
  "status": "success"
}
```

**请求**

```text
# Submitting the Job
curl -X POST http://localhost:8081/jars/<jar-id>/run \
  -d '{"parallelism": 3, "programArgs": "--bootstrap.servers kafka:9092 --checkpointing --event-time", "savepointPath": "<savepoint-path>"}'
```

**预期响应（打印精美）**

```text
{
  "jobid": "<job-id>"
}
```

现在，作业已被重新提交，但是由于没有足够的TaskSlots来执行并行性增强的任务槽（2个可用，需要3个），因此无法启动。用

```text
docker-compose scale taskmanager=2
```
{% endtab %}
{% endtabs %}

您可以将另一个带有两个TaskSlot的TaskManager添加到Flink群集，该群集将自动向Flink Master注册。添加TaskManager后不久，作业应重新开始运行。

一旦作业再次处于“运行中”状态，您将在_输出_主题中看到，现在在重新缩放过程中丢失了数据：所有窗口的数量都恰好为一千。  


### 查询作业指标

 Flink Master 通过其REST API 公开系统和用户[指标](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/metrics.html)。

 端点取决于这些指标的范围。可以通过列出工作范围的指标 `jobs/<job-id>/metrics`。可以通过`get`查询参数查询度量标准的实际值。

**请求**

```text
curl "localhost:8081/jobs/<jod-id>/metrics?get=lastCheckpointSize"
```

**预期响应（打印精美；无占位符）**

```text
[
  {
    "id": "lastCheckpointSize",
    "value": "9378"
  }
]
```

REST API不仅可以用于查询指标，而且还可以检索有关正在运行的作业状态的详细信息。

**请求**

```text
# find the vertex-id of the vertex of interest
curl localhost:8081/jobs/<jod-id>
```

**预期响应（打印精美）**

```text
{
  "jid": "<job-id>",
  "name": "Click Event Count",
  "isStoppable": false,
  "state": "RUNNING",
  "start-time": 1564467066026,
  "end-time": -1,
  "duration": 374793,
  "now": 1564467440819,
  "timestamps": {
    "CREATED": 1564467066026,
    "FINISHED": 0,
    "SUSPENDED": 0,
    "FAILING": 0,
    "CANCELLING": 0,
    "CANCELED": 0,
    "RECONCILING": 0,
    "RUNNING": 1564467066126,
    "FAILED": 0,
    "RESTARTING": 0
  },
  "vertices": [
    {
      "id": "<vertex-id>",
      "name": "ClickEvent Source",
      "parallelism": 2,
      "status": "RUNNING",
      "start-time": 1564467066423,
      "end-time": -1,
      "duration": 374396,
      "tasks": {
        "CREATED": 0,
        "FINISHED": 0,
        "DEPLOYING": 0,
        "RUNNING": 2,
        "CANCELING": 0,
        "FAILED": 0,
        "CANCELED": 0,
        "RECONCILING": 0,
        "SCHEDULED": 0
      },
      "metrics": {
        "read-bytes": 0,
        "read-bytes-complete": true,
        "write-bytes": 5033461,
        "write-bytes-complete": true,
        "read-records": 0,
        "read-records-complete": true,
        "write-records": 166351,
        "write-records-complete": true
      }
    },
    {
      "id": "<vertex-id>",
      "name": "Timestamps/Watermarks",
      "parallelism": 2,
      "status": "RUNNING",
      "start-time": 1564467066441,
      "end-time": -1,
      "duration": 374378,
      "tasks": {
        "CREATED": 0,
        "FINISHED": 0,
        "DEPLOYING": 0,
        "RUNNING": 2,
        "CANCELING": 0,
        "FAILED": 0,
        "CANCELED": 0,
        "RECONCILING": 0,
        "SCHEDULED": 0
      },
      "metrics": {
        "read-bytes": 5066280,
        "read-bytes-complete": true,
        "write-bytes": 5033496,
        "write-bytes-complete": true,
        "read-records": 166349,
        "read-records-complete": true,
        "write-records": 166349,
        "write-records-complete": true
      }
    },
    {
      "id": "<vertex-id>",
      "name": "ClickEvent Counter",
      "parallelism": 2,
      "status": "RUNNING",
      "start-time": 1564467066469,
      "end-time": -1,
      "duration": 374350,
      "tasks": {
        "CREATED": 0,
        "FINISHED": 0,
        "DEPLOYING": 0,
        "RUNNING": 2,
        "CANCELING": 0,
        "FAILED": 0,
        "CANCELED": 0,
        "RECONCILING": 0,
        "SCHEDULED": 0
      },
      "metrics": {
        "read-bytes": 5085332,
        "read-bytes-complete": true,
        "write-bytes": 316,
        "write-bytes-complete": true,
        "read-records": 166305,
        "read-records-complete": true,
        "write-records": 6,
        "write-records-complete": true
      }
    },
    {
      "id": "<vertex-id>",
      "name": "ClickEventStatistics Sink",
      "parallelism": 2,
      "status": "RUNNING",
      "start-time": 1564467066476,
      "end-time": -1,
      "duration": 374343,
      "tasks": {
        "CREATED": 0,
        "FINISHED": 0,
        "DEPLOYING": 0,
        "RUNNING": 2,
        "CANCELING": 0,
        "FAILED": 0,
        "CANCELED": 0,
        "RECONCILING": 0,
        "SCHEDULED": 0
      },
      "metrics": {
        "read-bytes": 20668,
        "read-bytes-complete": true,
        "write-bytes": 0,
        "write-bytes-complete": true,
        "read-records": 6,
        "read-records-complete": true,
        "write-records": 0,
        "write-records-complete": true
      }
    }
  ],
  "status-counts": {
    "CREATED": 0,
    "FINISHED": 0,
    "DEPLOYING": 0,
    "RUNNING": 4,
    "CANCELING": 0,
    "FAILED": 0,
    "CANCELED": 0,
    "RECONCILING": 0,
    "SCHEDULED": 0
  },
  "plan": {
    "jid": "<job-id>",
    "name": "Click Event Count",
    "nodes": [
      {
        "id": "<vertex-id>",
        "parallelism": 2,
        "operator": "",
        "operator_strategy": "",
        "description": "ClickEventStatistics Sink",
        "inputs": [
          {
            "num": 0,
            "id": "<vertex-id>",
            "ship_strategy": "FORWARD",
            "exchange": "pipelined_bounded"
          }
        ],
        "optimizer_properties": {}
      },
      {
        "id": "<vertex-id>",
        "parallelism": 2,
        "operator": "",
        "operator_strategy": "",
        "description": "ClickEvent Counter",
        "inputs": [
          {
            "num": 0,
            "id": "<vertex-id>",
            "ship_strategy": "HASH",
            "exchange": "pipelined_bounded"
          }
        ],
        "optimizer_properties": {}
      },
      {
        "id": "<vertex-id>",
        "parallelism": 2,
        "operator": "",
        "operator_strategy": "",
        "description": "Timestamps/Watermarks",
        "inputs": [
          {
            "num": 0,
            "id": "<vertex-id>",
            "ship_strategy": "FORWARD",
            "exchange": "pipelined_bounded"
          }
        ],
        "optimizer_properties": {}
      },
      {
        "id": "<vertex-id>",
        "parallelism": 2,
        "operator": "",
        "operator_strategy": "",
        "description": "ClickEvent Source",
        "optimizer_properties": {}
      }
    ]
  }
}
```

请查阅[REST API参考](https://ci.apache.org/projects/flink/flink-docs-release-1.10/monitoring/rest_api.html#api) 以获得可能查询的完整列表，包括如何查询不同范围的指标（例如TaskManager指标）；

## **变体**

您可能已经注意到，单击事件计数应用程序始终以--checkpointing和--event-time程序参数启动。通过在docker-compose.yaml中的客户端容器命令中省略这些内容，可以更改Job的行为

* `--checkpointing`启用[checkpoint](https://ci.apache.org/projects/flink/flink-docs-release-1.10/internals/stream_checkpointing.html)，这是Flink的容错机制。如果没有它就运行，并且经历了 [故障和恢复](https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/getting-started/docker-playgrounds/flink-operations-playground.html#observing-failure--recovery)，您应该会看到数据实际上已经丢失。
* `--event-time`为您的工作启用[事件时间语义](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/event_time.html)。禁用后，作业将根据壁钟时间而不是的时间戳将事件分配给窗口`ClickEvent`。因此，每个窗口的事件数将不再精确到一千。

Click Event Count应用程序还具有另一个选项（默认情况下处于关闭状态），您可以启用该选项来探索此工作在背压下的行为。您可以在docker-compose.yaml中的客户端容器的命令中添加此选项。

* `--backpressure`在作业中间增加了一个额外的运算符，该操作符会在偶数分钟内（例如10:12期间，但10:13期间没有）导致严重的背压。可以通过检查各种网络指标（例如`outputQueueLength`和`outPoolUsage`）和/或使用WebUI中可用的背压监视来观察到这一点。



