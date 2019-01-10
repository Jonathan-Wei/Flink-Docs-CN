# 调试Windows和Event Time

## 监控当前的事件时间\(Event Time\)

Flink的[事件时间](https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html)和水印支持是处理无序事件的强大特性。然而，由于时间的进度是在系统中跟踪的，因此很难准确地理解发生了什么。

每个任务的低水印可以通过Flink web接口或[Metrics系统](https://ci.apache.org/projects/flink/flink-docs-master/monitoring/metrics.html)访问。

Flink中的每个任务都公开一个名为`CurrentInputWatermark`的`Metrics`，该`Metrics`表示该任务接收到的最低水印。这个Long值表示“当前事件时间”。该值是通过取上游操作符接收到的所有水印的最小值来计算的。这意味着使用水印跟踪的事件时间总是由最后面的源控制。

可以**使用Web界面**，通过在“度量标准”选项卡中选择任务并选择`<taskNr>.currentInputWatermark`Metrics 标准来访问低水印度量标准。在新框中，就可以看到任务的当前低水印。

获取Metrics的另一种方法是使用一个**Metrics报告程序**，如[Metrics系统](https://ci.apache.org/projects/flink/flink-docs-master/monitoring/metrics.html)文档中所述。对于本地设置，我们建议使用JMX度量报告器和[VisualVM](https://visualvm.github.io/)之类的工具。

## 处理事件时间\(Event Time\)延迟

* 方法1：水印停留较晚（表示完整性），窗户提前开火 
* 方法2：具有最大延迟的水印启发式，窗口接受延迟数据

