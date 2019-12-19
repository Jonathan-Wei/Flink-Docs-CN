---
description: >-
  Apache Flink提供Table
  API作为统一的关系API，用于批处理和流处理，即，对无边界的实时流或有约束的批处理数据集以相同的语义执行查询，并产生相同的结果。Flink中的Table
  API通常用于简化数据分析，数据管道和ETL应用程序的定义。
---

# Table API

## 你将要构建什么？

在本教程中，您将学习如何建立一个连续的ETL管道，以随时间推移按帐户跟踪财务交易。您将首先将报告构建为夜间批处理作业，然后迁移到流式处理管道。

## 先决条件

本演练假定您对Java或Scala有所了解，但是即使您来自其他编程语言，也应该能够进行后续操作。它还假定您熟悉基本的关系概念，例如`SELECT`和`GROUP BY`子句。

## 救命，我被卡住了！

如果遇到困难，请查看[社区支持资源](https://flink.apache.org/community.html)。特别是，Apache Flink的[用户邮件列表](https://flink.apache.org/community.html#mailing-lists)一直被评为所有Apache项目中最活跃的[邮件列表](https://flink.apache.org/community.html#mailing-lists)之一，并且是快速获得帮助的好方法。

## 如何跟进

如果要继续学习，则需要一台具有以下功能的计算机：

* Java 8
* Maven

提供的Flink Maven原型将快速创建具有所有必需依赖项的框架项目：

{% tabs %}
{% tab title="Java" %}
```text
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-table-java \
    -DarchetypeCatalog=https://repository.apache.org/content/repositories/snapshots/ \
    -DarchetypeVersion=1.11-SNAPSHOT \
    -DgroupId=spend-report \
    -DartifactId=spend-report \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```
{% endtab %}

{% tab title="Scala" %}
```text
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-table-scala \
    -DarchetypeCatalog=https://repository.apache.org/content/repositories/snapshots/ \
    -DarchetypeVersion=1.11-SNAPSHOT \
    -DgroupId=spend-report \
    -DartifactId=spend-report \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**注意**：对于Maven 3.0或更高版本，不再可以通过命令行指定存储库（-DarchetypeCatalog）。如果要使用快照存储库，则需要将存储库条目添加到settings.xml中。有关此更改的详细信息，请参考[Maven的官方文档](http://maven.apache.org/archetype/maven-archetype-plugin/archetype-repository.html)
{% endhint %}

你可以根据你的喜好去修改`groupId`, `artifactId` 和`package`。使用上述参数，Maven将创建一个具有所有依赖项的项目以完成本教程。将项目导入编辑器后，您将看到一个包含以下代码的文件，可以直接在IDE中运行。

{% tabs %}
{% tab title="Java" %}
```text
ExecutionEnvironment env   = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment tEnv = BatchTableEnvironment.create(env);

tEnv.registerTableSource("transactions", new BoundedTransactionTableSource());
tEnv.registerTableSink("spend_report", new SpendReportTableSink());
tEnv.registerFunction("truncateDateToHour", new TruncateDateToHour());

tEnv
    .scan("transactions")
    .insertInto("spend_report");

env.execute("Spend Report");
```
{% endtab %}

{% tab title="Scala" %}
```text
val env  = ExecutionEnvironment.getExecutionEnvironment
val tEnv = BatchTableEnvironment.create(env)

tEnv.registerTableSource("transactions", new BoundedTransactionTableSource)
tEnv.registerTableSink("spend_report", new SpendReportTableSink)

val truncateDateToHour = new TruncateDateToHour

tEnv
    .scan("transactions")
    .insertInto("spend_report")

env.execute("Spend Report")
```
{% endtab %}
{% endtabs %}



