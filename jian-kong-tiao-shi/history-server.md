# History Server

Flink有一个历史记录服务器，可用于在关闭相应的Flink群集后查询已完成作业的统计信息。

此外，它公开了一个REST API，它接受HTTP请求并使用JSON数据进行响应。

## 概述

HistoryServer允许您查询已由JobManager归档的已完成作业的状态和统计信息。

配置HistoryServer _和_ JobManager后，可以通过相应的启动脚本启动和停止HistoryServer：

```text
# Start or stop the HistoryServer
bin/historyserver.sh (start|start-foreground|stop)
```

默认情况下，此服务器绑定`localhost`并侦听端口`8082`。

目前，您只能将其作为独立进程运行。

## 配置

配置钥匙的jobmanager.archive.fs.dir和historyserver.archive.fs.refresh-interval需要调整为显示存档的存档工作。

**JobManager**

```text
# Directory to upload completed job information
jobmanager.archive.fs.dir: hdfs:///completed-jobs
```

**HistoryServer**

```text
# Monitor the following directories for completed jobs
historyserver.archive.fs.dir: hdfs:///completed-jobs

# Refresh every 10 seconds
historyserver.archive.fs.refresh-interval: 10000
```

包含的存档将下载并缓存在本地文件系统中。通过`historyserver.web.tmpdir`配置本地目录。

查看配置页面以获取[完整的配置选项列表](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html#history-server)

## 可用请求

以下是可用请求列表，其中包含示例JSON响应。所有请求都是样本表单`http://hostname:8082/jobs`，下面我们仅列出URL 的_路径_部分。

尖括号中的值是变量，例如 `http://hostname:port/jobs/<jobid>/exceptions` 在实际请求中是这样的：`http://hostname:port/jobs/7684be6004e4e955c2a558a9bc463f65/exceptions`

* `/config`
* `/jobs/overview`
* `/jobs/<jobid>`
* `/jobs/<jobid>/vertices`
* `/jobs/<jobid>/config`
* `/jobs/<jobid>/exceptions`
* `/jobs/<jobid>/accumulators`
* `/jobs/<jobid>/vertices/<vertexid>`
* `/jobs/<jobid>/vertices/<vertexid>/subtasktimes`
* `/jobs/<jobid>/vertices/<vertexid>/taskmanagers`
* `/jobs/<jobid>/vertices/<vertexid>/accumulators`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/accumulators`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>/attempts/<attempt>`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtasknum>/attempts/<attempt>/accumulators`
* `/jobs/<jobid>/plan`

