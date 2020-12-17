# 词汇表

## **Flink应用程序集群\(Flink Application Cluster\)**

A Flink Application Cluster is a dedicated [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) that only executes [Flink Jobs](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) from one [Flink Application](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-application). The lifetime of the [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) is bound to the lifetime of the Flink Application.

## **Flink工作集群\(Flink Job Cluster\)**

A Flink Job Cluster is a dedicated [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) that only executes a single [Flink Job](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job). The lifetime of the [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) is bound to the lifetime of the Flink Job.

## **Flink Cluster**

一般情况下，Flink 集群是由一个 [Flink JobManager](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-jobmanager) 和一个或多个 [Flink TaskManager](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-taskmanager) 进程组成的分布式系统。

## **Event**

Event 是对应用程序建模的域的状态更改的声明。它可以同时为流或批处理应用程序的 input 和 output，也可以单独是 input 或者 output 中的一种。Event 是特殊类型的 [Record](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#record)。

## **ExecutionGraph**

见 [Physical Graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#physical-graph)。

## **Function**

Function 是由用户实现的，并封装了 Flink 程序的应用程序逻辑。大多数 Function 都由相应的 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 封装。

## **Instance**

Instance 常用于描述运行时的特定类型\(通常是 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 或者 [Function](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#function)\)的一个具体实例。由于 Apache Flink 主要是用 Java 编写的，所以，这与 Java 中的 _Instance_ 或 _Object_ 的定义相对应。在 Apache Flink 的上下文中，_parallel instance_ 也常用于强调同一 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 或者 [Function](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#function) 的多个 instance 以并行的方式运行。

## **Flink Application**

A Flink application is a Java Application that submits one or multiple [Flink Jobs](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) from the `main()` method \(or by some other means\). Submitting jobs is usually done by calling `execute()` on an execution environment.

The jobs of an application can either be submitted to a long running [Flink Session Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-session-cluster), to a dedicated [Flink Application Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-application-cluster), or to a [Flink Job Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job-cluster).

## **Flink Job**

A Flink Job is the runtime representation of a [logical graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#logical-graph) \(also often called dataflow graph\) that is created and submitted by calling `execute()` in a [Flink Application](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-application).

## **JobGraph**

见 [Logical Graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#logical-graph)。

## **Flink JobManager**

Flink JobManager 是 [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) 的主节点。它包含三个不同的组件：Flink Resource Manager、Flink Dispatcher、运行每个 [Flink Job](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) 的 [Flink JobMaster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-jobmaster)。

## **Flink JobMaster**

JobMaster 是在 [Flink JobManager](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-jobmanager) 运行中的组件之一。JobManager 负责监督单个作业 [Task](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#task) 的执行。以前，整个 [Flink JobManager](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-jobmanager) 都叫做 JobManager。

## **Logical Graph**

A logical graph is a directed graph where the nodes are [Operators](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) and the edges define input/output-relationships of the operators and correspond to data streams or data sets. A logical graph is created by submitting jobs from a [Flink Application](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-application).

Logical graphs are also often referred to as _dataflow graphs_.

## **Managed State**

Managed State 描述了已在框架中注册的应用程序的托管状态。对于托管状态，Apache Flink 会负责持久化和重伸缩等事宜。

## **Operator**

[Logical Graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#logical-graph) 的节点。算子执行某种操作，该操作通常由 [Function](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#function) 执行。Source 和 Sink 是数据输入和数据输出的特殊算子。

## **Operator Chain**

算子链由两个或多个连续的 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 组成，两者之间没有任何的重新分区。同一算子链内的算子可以彼此直接传递 record，而无需通过序列化或 Flink 的网络栈。

## **Partition**

分区是整个数据流或数据集的独立子集。通过将每个 [Record](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#record) 分配给一个或多个分区，来把数据流或数据集划分为多个分区。在运行期间，[Task](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#task) 会消费数据流或数据集的分区。改变数据流或数据集分区方式的转换通常称为重分区。

## **Physical Graph**

Physical graph 是一个在分布式运行时，把 [Logical Graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#logical-graph) 转换为可执行的结果。节点是 [Task](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#task)，边表示数据流或数据集的输入/输出关系或 [partition](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#partition)。

## **Record**

Record 是数据集或数据流的组成元素。[Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 和 [Function](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#Function)接收 record 作为输入，并将 record 作为输出发出。

## **Flink Session Cluster**

长时间运行的 [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster)，它可以接受多个 [Flink Job](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) 的执行。此 [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) 的生命周期不受任何 [Flink Job](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) 生命周期的约束限制。以前，Flink Session Cluster 也称为 _session mode_ 的 [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster)，和 [Flink Application Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-application-cluster) 相对应。

## **State Backend**

对于流处理程序，[Flink Job](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-job) 的 State Backend 决定了其 [state](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#managed-state) 是如何存储在每个 TaskManager 上的（ TaskManager 的 Java 堆栈或嵌入式 RocksDB），以及它在 checkpoint 时的写入位置（ [Flink JobManager](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-jobmanager) 的 Java 堆或者 Filesystem）。

## **Sub-Task**

Sub-Task 是负责处理数据流 [Partition](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#partition) 的 [Task](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#task)。”Sub-Task”强调的是同一个 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 或者 [Operator Chain](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator-chain) 具有多个并行的 Task 。

## **Task**

Task 是 [Physical Graph](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#physical-graph) 的节点。它是基本的工作单元，由 Flink 的 runtime 来执行。Task 正好封装了一个 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 或者 [Operator Chain](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator-chain) 的 _parallel instance_。

## **Flink TaskManager**

TaskManager 是 [Flink Cluster](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#flink-cluster) 的工作进程。[Task](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#task) 被调度到 TaskManager 上执行。TaskManager 相互通信，只为在后续的 Task 之间交换数据。

## **Transformation**

Transformation 应用于一个或多个数据流或数据集，并产生一个或多个输出数据流或数据集。Transformation 可能会在每个记录的基础上更改数据流或数据集，但也可以只更改其分区或执行聚合。虽然 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 和 [Function](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#function) 是 Flink API 的“物理”部分，但 Transformation 只是一个 API 概念。具体来说，大多数（但不是全部）Transformation 是由某些 [Operator](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/concepts/glossary.html#operator) 实现的。

