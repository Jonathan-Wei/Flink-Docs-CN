# 数据流编程模型

## 抽象层次

Flink提供不同级别的抽象来开发流/批处理应用程序。

![](../.gitbook/assets/levels_of_abstraction.svg)

## 程序和数据流

![](../.gitbook/assets/program_dataflow.svg)

## 并行数据流

![](../.gitbook/assets/parallel_dataflow.svg)

## 窗口

![](../.gitbook/assets/windows.svg)

## Time

当在流程序中引用时间（例如定义窗口）时，可以参考不同的时间概念：

![](../.gitbook/assets/event_ingestion_processing_time.svg)

有关如何处理时间的更多详细信息，请参阅[事件时间文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html)。

## 有状态操作

虽然数据流中的许多操作只是一次查看一个单独的_事件_（例如事件解析器），但某些操作会记住多个事件（例如窗口操作符）的信息。这些操作称为**有状态**。

![](../.gitbook/assets/state_partitioning.svg)

有关更多信息，请参阅有关[状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/index.html)的文档。

## 容错检查点

## 批处理流媒体

