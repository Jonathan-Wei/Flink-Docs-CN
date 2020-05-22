# 升级应用程序和Flink版本

## 

| 操作符 | 内部操作符状态的数据类型 |
| :--- | :--- |
| ReduceFunction\[IOT\] | IOT \(Input and output type\) \[, KEY\] |
| FoldFunction\[IT, OT\] | OT \(Output type\) \[, KEY\] |
| WindowFunction\[IT, OT, KEY, WINDOW\] | IT \(Input type\), KEY |
| AllWindowFunction\[IT, OT, WINDOW\] | IT \(Input type\) |
| JoinFunction\[IT1, IT2, OT\] | IT1, IT2 \(Type of 1. and 2. input\), KEY |
| CoGroupFunction\[IT1, IT2, OT\] | IT1, IT2 \(Type of 1. and 2. input\), KEY |
| Built-in Aggregations \(sum, min, max, minBy, maxBy\) | Input Type \[, KEY\] |

## 兼容性表格

| 创建于\还原于 | 1.1.x | 1.2.x | 1.3.x | 1.4.x | 1.5.x | 1.6.x | 1.7.x | 1.8.x | 1.9.x | 1.10.x | 局限性 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **1.1.x** | Ø | Ø | Ø |  |  |  |  |  |  |  | 从Flink 1.1.x迁移到1.2.x +的作业的最大并行度目前固定为该作业的并行度。这意味着迁移后不能增加并行度。此限制可能会在将来的错误修正版本中删除。 |
| **1.2.x** |  | Ø | Ø | Ø | Ø | Ø | Ø | Ø | Ø | Ø | 从Flink 1.2.x迁移到Flink 1.3.x +时，不支持同时更改并行性。在迁移到Flink 1.3.x +之后，用户必须首先获取一个保存点，然后更改并行性。  为CEP应用程序创建的保存点无法在1.4.x +中还原。  Flink 1.2中包含Scala TraversableSerializer的保存点不再与Flink 1.8兼容，因为该序列化程序已进行了更新。通过首先升级到Flink 1.3和Flink 1.7之间的版本，然后再更新到Flink 1.8，可以解决此限制。 |
| **1.3.x** |  |  | Ø | Ø | Ø | Ø | Ø | Ø | Ø | Ø | 如果保存点包含Scala案例类，则从Flink 1.3.0迁移到Flink 1.4。\[0,1\]将失败。用户必须直接迁移到1.4.2+。 |
| **1.4.x** |  |  |  | Ø | Ø | Ø | Ø | Ø | Ø | Ø |  |
| **1.5.x** |  |  |  |  | Ø | Ø | Ø | Ø | Ø | Ø | 在版本1.6.x到1.6.2以及1.7.0中使用1.5.x创建的广播状态恢复时存在一个已知问题：[FLINK-11087](https://issues.apache.org/jira/browse/FLINK-11087)。升级到1.6.x或1.7.x系列的用户需要分别直接迁移到高于1.6.2和1.7.0的次要版本。 |
| **1.6.x** |  |  |  |  |  | Ø | Ø | Ø | Ø | Ø |  |
| **1.7.x** |  |  |  |  |  |  | Ø | Ø | Ø | Ø |  |
| **1.8.x** |  |  |  |  |  |  |  | Ø | Ø | Ø |  |
| **1.9.x** |  |  |  |  |  |  |  |  | Ø | Ø |  |
| **1.10.x** |  |  |  |  |  |  |  |  |  | Ø |  |

