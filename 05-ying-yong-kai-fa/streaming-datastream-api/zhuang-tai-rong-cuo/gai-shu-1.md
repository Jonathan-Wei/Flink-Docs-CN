# 概述

有状态函数和运算符在各个元素/事件的处理中存储数据，使状态成为任何类型的更复杂操作的关键构建块。

例如:

* 当应用程序搜索特定的事件模式时，状态将存储到目前为止遇到的事件序列。 
* 当按每分钟/小时/每天聚合事件时，状态保留为处理的聚合。 
* 当在数据点流上训练机器学习模型时，状态保存模型参数的当前版本。 
* 当需要管理历史数据时，状态允许有效地访问过去发生的事件。

Flink需要了解状态，以便使用[检查点](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/checkpointing.html)使状态容错，并允许流应用程序的[保存点](https://ci.apache.org/projects/flink/flink-docs-master/ops/state/savepoints.html)。

有关状态的知识还允许重新调整Flink应用程序，这意味着Flink负责跨并行实例重新分配状态。

Flink 的[可查询状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/queryable_state.html)功能允许您在运行时从Flink外部访问状态。

在使用state时，阅读[Flink的状态后端](https://ci.apache.org/projects/flink/flink-docs-master/ops/state/state_backends.html)可能也很有用。Flink提供了不同的状态后端，用于指定状态的存储方式和位置。State可以位于Java的堆上或堆外。根据您的状态后端，Flink还可以_管理_应用程序的状态，这意味着Flink处理内存管理（如果需要可能会溢出到磁盘）以允许应用程序保持非常大的状态。可以在不更改应用程序逻辑的情况下配置状态后端。

