# 概述

Apache Flink提供了两个关系型API——Table API和SQL——用于统一的流和批处理。Table API是面向Scala和Java的语言集成查询API，它允许以非常直观的方式组合来自关系操作符\(如选择、筛选和连接\)的查询。Flink的SQL支持基于[Apache Calcite](https://calcite.apache.org/)，它实现了SQL标准。无论输入是批处理输入\(数据集\)还是流输入\(DataStream\)，在这两个接口中指定的查询都具有相同的语义并指定相同的结果。

Table API和SQL接口以及Flink的DataStream和DataSet API紧密集成在一起。您可以轻松地在所有api和基于api的库之间切换。例如，可以使用CEP库从DataStream提取模式，然后使用Table API分析模式，或者在对预处理数据运行Gelly图形算法之前，可以使用SQL查询扫描、筛选和聚合批处理表。

请注意，Table API和SQL特性还不够完善，并且正在积极开发。并非所有操作都受\[Table API、SQL\]和\[stream、batch\]输入的每个组合的支持。

## 配置

Table API和SQL绑定在flink-table Maven artifact中。为了使用表API和SQL，您的项目必须添加以下依赖项:

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-table_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

此外，您需要为Flink的Scala批处理或流API添加一个依赖项。对于批处理查询，您需要添加:

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-scala_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

对于流式查询，您需要添加:

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
</dependency>
```

注意:由于Apache Calcite中的一个问题，它阻止用户类加载器被垃圾收集，我们不建议构建包含Flink-table依赖项的fat-jar。相反，我们建议将Flink配置为在系统类加载器中包含Flink-table依赖项。这可以通过复制flink-table来实现。从./opt文件夹到./lib文件夹的jar文件。有关更多细节，请参见[这些](https://github.com/Jonathan-Wei/Flink-Docs-CN/blob/master/05%20应用开发/00%20项目构建配置/配置Dependencies%2C%20Connectors%2C%20Libraries.md)说明。

