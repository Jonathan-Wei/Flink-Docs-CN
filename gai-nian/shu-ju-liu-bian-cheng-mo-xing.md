# 数据流编程模型

## 抽象层次

Flink提供不同级别的抽象来开发流/批处理应用程序。

![](../.gitbook/assets/levels_of_abstraction.svg)

* 最低层次的抽象只提供有状态流。它通过流程函数嵌入到DataStream API中。它允许用户自由地处理来自一个或多个流的事件，并使用一致的容错状态。此外，用户可以注册事件时间和处理时间回调，让程序实现复杂的计算。
* 在实践中，大多数应用程序不需要上述低层抽象，而是根据核心API\(如_DataStream API_\(有界/无界流\)和_DataSet API_\(有界数据集\)进行编程。这些连贯的api为数据处理提供了常见的构建块，比如各种形式的用户指定转换、连接、聚合、窗口、状态等。在这些api中处理的数据类型表示为各自编程语言中的类。

  低级流程功能与_DataStream API_集成，使得仅对某些操作进行低级抽象成为可能。数据集API在有界数据集上提供了额外的原语，比如循环/迭代。

* _Table API_是以表为中心的声明性DSL，表可以是动态更改的表\(在表示流时\)。_Table API_遵循\(扩展\)关系模型:表有一个附加模式\(类似于关系数据库表\)和API提供了类似的操作,如_select, project, join, group-by, aggregate_等。表API程序以声明的方式定义逻辑操作应该做什么，而不是指定操作的代码看起来如何。尽管表API可由各种类型的用户定义函数扩展，但它的表达能力不如_Core API_，但使用起来更简洁\(编写的代码更少\)。此外，_Table API_程序还会通过优化程序，在执行之前应用优化规则。

  Table和_DataStream/DataSet_之间可以无缝转换，允许程序混合_Table API_和_DataStream_和_DataSet API_。

* Flink提供的最高级抽象是**SQL**。这种抽象在语义和表达方面类似于_Table API_，但是将程序表示为SQL查询表达式。在[SQL](https://ci.apache.org/projects/flink/flink-docs-master/dev/table_api.html#sql)抽象与表API紧密地相互作用，SQL查询可以通过定义表来执行_Table API_。

## 程序和数据流

Flink程序的基本构建块是**streams**和**transformations**。（请注意，Flink的DataSet API中使用的DataSet也是内部流 - 稍后会详细介绍。）从概念上讲，_流_是（可能永无止境的）数据记录流，而_转换_是将一个或多个流作为一个或多个流的操作。输入，并产生一个或多个输出流。

执行时，Flink程序映射到**streaming dataflows**，包括**流**和转换**运算符**。每个数据流都以一个或多个**sources**开头，并以一个或多个**sinks**结束。数据流类似于任意有**向无环图** _（DAG）_。虽然通过迭代构造允许特殊形式的循环，但为了简单起见，我们将在大多数情况下对此进行解释。

![](../.gitbook/assets/program_dataflow.svg)

通常，程序中的转换与数据流中的运算符之间存在一对一的对应关系。但是，有时一个转换可能包含多个转换运算符。

Source和Sink记录在 [streaming connectors](https://ci.apache.org/projects/flink/flink-docs-master/dev/connectors/index.html)和 [batch connectors](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/connectors.html)文档中。[转换](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/dataset_transformations.html)记录在[DataStream运算符](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/index.html)和[DataSet转换](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/dataset_transformations.html)中。

## 并行数据流

Flink中的程序本质上是并行和分布式的。在执行期间，_流_具有一个或多个**流分区**，并且每个_运算符_具有一个或多个**运算符子任务**。运算符子任务彼此独立，并且可以在不同的线程中执行，甚至可能在不同的机器或容器上执行。

运算符子任务的数量就是该运算符的并行度。流的并行度始终是它的产生算子的并行度。同一程序的不同操作符可能具有不同的并行级别。

![](../.gitbook/assets/parallel_dataflow.svg)

流可以在两个操作符之间以一对一\(或转发\)模式传输数据，也可以以重分发模式传输数据:

* 一对一流\(例如上图中Source和map\(\)操作符之间的流\)保存元素的分区和顺序。这意味着map\(\)操作符的子任务\[1\]将以与Source操作符的子任务\[1\]生成的元素相同的顺序看到相同的元素。
* 重新分布流\(如上面的map\(\)和keyBy/window之间，以及keyBy/window和Sink之间\)更改流的分区。每个操作子任务根据选择的转换将数据发送到不同的目标子任务。例如keyBy\(\)\(通过散列键重新分区\)、broadcast\(\)或_rebalance_\(\)\(随机重新分区\)。在重分发交换中，元素之间的顺序仅保存在每一对发送和接收子任务中\(例如map\(\)的子任务\[1\]和keyBy/window的子任务\[2\]\)。因此，在本例中，保留了每个键内的顺序，但是并行度确实引入了关于不同键的聚合结果到达sink的顺序的不确定性。

有关配置和控制并行度的详细信息，请参阅[并行执行](https://ci.apache.org/projects/flink/flink-docs-master/dev/parallel.html)的文档。

## 窗口

聚合事件（例如，计数，总和）在流上的工作方式与批处理方式不同。例如，不可能计算流中的所有元素，因为流通常是无限的（无界）。相反，流上的聚合（计数，总和等）由**窗口**限定，例如_“在最后5分钟内计数”_或_“最后100个元素的总和”_。

Windows可以是_时间驱动的_（例如：每30秒）或_数据驱动_（例如：每100个元素）。一个典型地区分不同类型的窗口，例如_翻滚窗口_（没有重叠）， _滑动窗口_（具有重叠）和_会话窗口_（由不活动的间隙打断）。

![](../.gitbook/assets/windows.svg)

更多窗口示例可以在此[博客文章中](https://flink.apache.org/news/2015/12/04/Introducing-windows.html)找到。更多详细信息在[窗口文档中](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html)。

## Time

当在流程序中引用时间（例如定义窗口）时，可以参考不同的时间概念：

* **Event Time：**创建**事件的时间**。它通常由事件中的时间戳描述，例如由生产传感器或生产服务附加。Flink通过[时间戳分配器](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_timestamps_watermarks.html)访问事件时间戳。

  **Ingestion Time:**事件在源操作员处输入Flink数据流的时间。

  **Processing Time:** 执行基于时间的操作的每个操作员的本地时间。

![](../.gitbook/assets/event_ingestion_processing_time.svg)

有关如何处理时间的更多详细信息，请参阅[事件时间文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html)。

## 有状态操作

虽然数据流中的许多操作只是一次查看一个单独的_事件_（例如事件解析器），但某些操作会记住多个事件（例如窗口操作符）的信息。这些操作称为**有状态**。

有状态操作的状态是在可以看作是嵌入式键/值存储区中维护的。状态与由有状态操作符读取的流一起严格地划分和分布。因此，只有在keyBy\(\)函数之后的键控流上才能访问键/值状态，并且只能访问与当前事件的键关联的值。对流和状态的键进行对齐可以确保所有状态更新都是本地操作，从而保证一致性，而不会产生事务开销。这种对齐还允许Flink重新分配状态并透明地调整流分区。

![](../.gitbook/assets/state_partitioning.svg)

有关更多信息，请参阅有关[状态](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/index.html)的文档。

## 容错检查点

Flink使用**流重放\(stream replay\)**和**检查点\(checkpointing\)**的组合实现容错。检查点与每个输入流中的特定点以及每个操作符的对应状态相关。通过恢复操作符的状态并从检查点的点重放事件，可以从检查点恢复流数据流，同时保持一致性_（exactly-once处理语义）_。

检查点间隔是在执行期间用恢复时间（需要重放的事件的数量）来折衷容错开销的手段。

[容错内部](https://ci.apache.org/projects/flink/flink-docs-master/internals/stream_checkpointing.html)的描述提供了有关Flink如何管理检查点和相关主题的更多信息。有关启用和配置检查点的详细信息，请参阅[检查点API文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/checkpointing.html)。

## 批处理流媒体

Flink将[批处理程序](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/index.html)作为流程序的一种特殊情况执行，其中流是有界的\(元素数量有限\)。数据集在内部被视为数据流。因此，上述概念同样适用于批处理程序，也适用于流媒体程序，但有少数例外:

* [批处理程序的容错](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/fault_tolerance.html)不使用检查点。恢复通过完全重播流来实现。这是可能的，因为输入是有界的。这将使成本更接近于恢复，但会降低常规处理的成本，因为它可以避免检查点。
* DataSet API中的有状态操作使用简化的内存/内核外数据结构，而不是键/值索引。
* DataSet API引入了特殊的同步\(superstep-based\)迭代，这只在有界流上是可能的。有关详细信息，请查看[迭代文档](https://ci.apache.org/projects/flink/flink-docs-master/dev/batch/iterations.html)。

