# 本地安装

按照以下几个步骤下载最新的稳定版本并开始使用。

### Step 1: Download <a id="step-1-download"></a>

要运行Flink，唯一的要求是有一个工作的Java 8或11安装。你可以通过执行以下命令来检查Java的安装是否正确:

```text
java -version
```

[下载](https://flink.apache.org/downloads.html)1.11.0的安装包并解压

```text
$ tar -xzf flink-1.11.0-bin-scala_2.11.tgz
$ cd flink-1.11.0-bin-scala_2.11
```

### Step 2: Start a Cluster <a id="step-2-start-a-cluster"></a>

使用Flink附带一个bash脚本来启动本地集群。

```text
$ ./bin/start-cluster.sh
Starting cluster.
Starting standalonesession daemon on host.
Starting taskexecutor daemon on host.
```

### Step 3: Submit a Job <a id="step-3-submit-a-job"></a>

Flink的版本附带了许多示例作业。您可以快速地将其中一个应用程序部署到正在运行的集群中。

```text
$ ./bin/flink run examples/streaming/WordCount.jar
$ tail log/flink-*-taskexecutor-*.out
  (to,1)
  (be,1)
  (or,1)
  (not,1)
  (to,2)
  (be,2)
```

此外，可以检查Flink的[Web UI](http://localhost:8080/)来监视集群和正在运行的作业的状态。

### Step 4: Stop the Cluster <a id="step-4-stop-the-cluster"></a>

完成后，可以快速停止集群和所有正在运行的组件。

```text
$ ./bin/stop-cluster.sh
```

