# 概述

Apache Flink提供了两个关系型API——Table API和SQL——用于统一的流和批处理。Table API是面向Scala和Java的语言集成查询API，它允许以非常直观的方式组合来自关系操作符\(如选择、筛选和连接\)的查询。Flink的SQL支持基于[Apache Calcite](https://calcite.apache.org/)，它实现了SQL标准。无论输入是批处理输入\(数据集\)还是流输入\(DataStream\)，在这两个接口中指定的查询都具有相同的语义并指定相同的结果。

Table API和SQL接口以及Flink的DataStream和DataSet API紧密集成在一起。您可以轻松地在所有api和基于api的库之间切换。例如，可以使用CEP库从DataStream提取模式，然后使用Table API分析模式，或者在对预处理数据运行Gelly图形算法之前，可以使用SQL查询扫描、筛选和聚合批处理表。

请注意，Table API和SQL特性还不够完善，并且正在积极开发。并非所有操作都受\[Table API、SQL\]和\[stream、batch\]输入的每个组合的支持。

## 依赖结构

从Flink 1.9开始，Flink提供了两种不同的计划程序实现来评估Table＆SQL API程序：Blink计划程序和Flink 1.9之前可用的旧计划程序。计划人员负责将关系运算符转换为可执行的，优化的Flink作业。两位计划者都带有不同的优化规则和运行时类。它们在支持的功能方面也可能有所不同。

{% hint style="danger" %}
对于生产场景，建议使用Flink 1.9之前的旧计划器。
{% endhint %}

 所有Table API和SQL组件都捆绑在`flink-table`或`flink-table-blink`Maven依赖中。

以下依赖关系与大多数项目有关：

* `flink-table-common`：用于通过自定义功能，格式等扩展表生态系统的通用模块。
* `flink-table-api-java`：适用于使用Java编程语言的纯表程序的Table＆SQL API（处于开发初期，不建议使用！）。
* `flink-table-api-scala`：使用Scala编程语言的纯表程序的Table＆SQL API（处于开发初期，不建议使用！）。
* `flink-table-api-java-bridge`：使用Java编程语言支持带有DataStream / DataSet API的Table＆SQL API。
* `flink-table-api-scala-bridge`：使用Scala编程语言支持带有DataStream / DataSet API的Table＆SQL API。
* `flink-table-planner`：表程序计划程序和运行时。这是1.9版本之前Flink的唯一计划器。仍然是推荐的一种。
* `flink-table-planner-blink`：新的Blink计划器。
* `flink-table-runtime-blink`：新的Blink运行时。
* `flink-table-uber`：将上述API模块以及旧的计划器打包到大多数Table＆SQL API用例的分发中。默认情况下，超级JAR文件`flink-table-*.jar`位于`/lib`Flink版本的目录中。
* `flink-table-uber-blink`：将上述API模块以及特定于Blink的模块打包到大多数Table＆SQL API用例的分发中。默认情况下，超级JAR文件`flink-table-blink-*.jar`位于`/lib`Flink版本的目录中。

有关如何在表程序中的新旧Blink计划程序之间进行切换的更多信息，请参见[通用API](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html)页面。

### 表程序依赖性

根据目标编程语言，需要将Java或Scala API添加到项目中，以便使用Table API和SQL定义管道：

```markup
<!-- Either... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-java-bridge_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
<!-- or... -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

此外，如果你想在你的IDE中本地运行表API和SQL程序，你必须添加以下模块之一，这取决于你想使用的Planner:

```text
<!-- Either... (for the old planner that was available before Flink 1.9) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
<!-- or.. (for the new Blink planner) -->
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-planner-blink_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

在内部，表生态系统的一部分是由Scala实现的。因此，请确保为批处理和流应用程序都添加以下依赖项：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

### 扩展依赖

 如果要实现与Kafka或一组[用户定义函数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/systemFunctions.html)进行交互的[自定义格式](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/sourceSinks.html#define-a-tablefactory)，则以下依赖关系就足够了，并且可以用于SQL Client的JAR文件：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table-common</artifactId>
  <version>1.10.0</version>
  <scope>provided</scope>
</dependency>
```

当前，该模块包括以下扩展点：

* `SerializationSchemaFactory`
* `DeserializationSchemaFactory`
* `ScalarFunction`
* `TableFunction`
* `AggregateFunction`

