# 执行计划

根据各种参数（如数据大小或群集中的计算机数量），Flink的优化程序会自动为您的程序选择执行策略。在许多情况下，了解Flink将如何执行程序会很有用。

**计划可视化工具**

Flink附带了一个用于执行计划的可视化工具。包含可视化工具的HTML文档位于`tools/planVisualizer.html`。它采用JSON的方式表示作业执行计划，并将其可视化为具有执行策略的完整注释的图形。

以下代码展示了如何从程序中打印JSON形式的执行计划：

{% tabs %}
{% tab title="Java" %}
```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

...

System.out.println(env.getExecutionPlan());
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment

...

println(env.getExecutionPlan())
```
{% endtab %}
{% endtabs %}

要可视化执行计划，请执行以下操作：

1. 使用Web浏览器打开`planVisualizer.html`
2. 将JSON字符串**粘贴**到文本字段中
3. **按下**绘图按钮。

完成这些步骤后，将显示详细的执行计划。

![](../../.gitbook/assets/plan_visualizer.png)

**Web界面**

Flink提供了一个用于提交和执行作业的Web界面。该接口是用于监控的JobManager Web接口的一部分，默认情况下，它在端口`8081`上运行。通过此接口提交作业需要在`flink-conf.yaml`中设置`web.submit.enable:true`。

可以在执行作业之前指定程序参数。计划可视化使您能够在执行Flink作业之前显示执行计划

