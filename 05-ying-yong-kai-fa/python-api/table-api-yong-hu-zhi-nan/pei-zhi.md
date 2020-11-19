# 配置

 根据Python Table API程序的要求，可能需要调整某些参数以进行优化。Java / Scala Table API程序可用的所有配置选项也可以在Python Table API程序中使用。您可以参考[Table API配置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/config.html)以获取有关Java / Scala Table API程序的所有可用配置选项的更多详细信息。它还提供了有关如何在Table API程序中设置配置选项的示例。

### Python选项 <a id="python-options"></a>

| 键 | 默认 | 类型 | 描述 |
| :--- | :--- | :--- | :--- |
| **python.fn-execution.buffer.memory.size** | “ 15mb” | String | Python辅助程序的输入缓冲区和输出缓冲区分配的内存量。如果分配给运算符的实际内存不小于Python worker的总内存，则该内存将被视为托管内存。否则，此配置无效。 |
| **python.fn-execution.bundle.size** | 1000 | Integer | 捆绑包中用于Python用户定义函数执行的最大元素数。元素被异步处理。在处理下一个元素束之前，先处理一个元素束。较大的值可以提高吞吐量，但以更多的内存使用量和更高的延迟为代价。 |
| **python.fn-execution.bundle.time** | 1000 | Long | 设置在处理要用于Python用户定义函数执行的包之前的等待超时（以毫秒为单位）。超时定义捆绑包中的元素在被处理之前将被缓冲多长时间。较低的超时会导致较低的尾部等待时间，但可能会影响吞吐量。 |
| **python.fn-execution.framework.memory.size** | “ 64mb” | String | Python框架要分配的内存量。此配置的值和“ python.fn-execution.buffer.memory.size”的总和表示Python工作线程的总内存。如果分配给运算符的实际内存不小于Python worker的总内存，则该内存将被视为托管内存。否则，此配置无效。 |

