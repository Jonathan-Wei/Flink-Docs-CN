# 概述

## 初步了解Flink

**Docker游乐园\(Playgrounds\)** 提供了Flink沙盒环境，几分钟内就可以设置好，让你可以探索和使用Flink环境

*  [**Operations Playground**](https://ci.apache.org/projects/flink/flink-docs-master/getting-started/docker-playgrounds/flink-operations-playground.html)展示了如何用弗林克工作流应用。您可以体验Flink如何从故障中恢复应用程序，如何上下升级和扩展流应用程序以及查询应用程序指标。

## 使用Flink API的第一步

代码走查是最佳的入门方式，并且可以一步步的为你介绍API。走查提供了使用代码框架引导小型Flink项目的说明，并展示了如何将其扩展到简单的应用程序。

* [**DataStream API**](https://ci.apache.org/projects/flink/flink-docs-master/getting-started/walkthroughs/datastream_api.html) 代码走查展示了如何实现一个简单的DataStream应用程序，以及如何把它扩大到有状态和使用定时器。DataStream API是Flink的主要抽象，用于以Java或Scala实现具有复杂时间语义的状态流应用程序
* [**Table API**](https://ci.apache.org/projects/flink/flink-docs-master/getting-started/walkthroughs/table_api.html) 代码走查展示了如何实现批来源以及如何将它演变成一个流源连续查询一个简单的表API查询。Table API Flink的语言嵌入式关系API用于在Java或Scala中编写类似于SQL的查询，这些查询会自动优化，类似于SQL查询。可以使用相同的语法和语义对批处理或流数据执行表API查询。

