# Hadoop整合

## 引入Hadoop配置

您可以通过设置环境变量来引用Hadoop配置`HADOOP_CONF_DIR`。

```text
HADOOP_CONF_DIR=/path/to/etc/hadoop
```

不建议在[Flink配置中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/config.html#hdfs)引用HDFS配置。

提供Hadoop配置的另一种方法是将其放在Flink流程的类路径中，请参阅下面的更多详细信息。

## 提供Hadoop类

为了使用Hadoop功能（例如YARN，HDFS），必须为Flink提供所需的Hadoop类，因为默认情况下未捆绑这些类。

这可以通过以下步骤完成：

1）将Hadoop类路径添加到Flink 

2）将所需的jar文件放入Flink发行版的/ lib目录中

选项1）只需很少的工作，可以与现有的Hadoop设置很好地集成在一起，应该是首选方法。但是，Hadoop的依赖项占用量很大，这增加了发生依赖项冲突的风险。如果发生这种情况，请参阅选项2）。

以下小节将详细介绍这些方法。

### 添加Hadoop类路径

Flink将使用环境变量HADOOP\_CLASSPATH来扩充启动Flink组件（如Client，JobManager或TaskManager）时使用的类路径。大多数Hadoop发行版和云环境在默认情况下不会设置此变量，因此，如果Flink应该选择Hadoop类路径，则必须在运行Flink组件的所有计算机上导出环境变量。

在运行yarn时，这通常不是问题，因为在yarn中运行的组件将从Hadoop类路径开始，但在向yarn提交作业时，Hadoop依赖项可能必须位于类路径中。为此，通常只需

```text
export HADOOP_CLASSPATH=`hadoop classpath`
```

在shell中。请注意，`hadoop`是hadoop的二进制文件，`classpath`是一个参数，用于打印已配置的Hadoop类路径。

### 将Hadoop添加到/lib目录

Flink项目发布了特定版本的Hadoop发行版，该发行版重新定位或排除了多个依赖项，以减少依赖项冲突的风险。这些可以在下载页面的“ [其他组件”](https://flink.apache.org/downloads.html#additional-components)部分中找到。对于这些版本，下载相应的`Pre-bundled Hadoop`组件并将其放入`/lib`Flink分发目录中就足够了。

{% hint style="info" %}
如果要针对特定于供应商的Hadoop版本构建缺陷阴影，首先必须在本地maven设置中配置特定于供应商的maven存储库，如下所述。
{% endhint %}

如果下载页面上未列出使用的Hadoop版本（可能由于是特定于供应商的版本），则有必要针对该版本构建[flink依赖的](https://github.com/apache/flink-shaded)版本。您可以在下载页面的“ [其他组件”](https://flink.apache.org/downloads.html#additional-components)部分中找到该项目的源代码。

运行以下命令以`flink-shaded`针对所需的Hadoop版本（例如version `2.6.5-custom`）进行构建和安装：

```text
mvn clean install -Dhadoop.version=2.6.5-custom
```

完成此步骤后，将`flink-shaded-hadoop-2-uber`jar放入`/lib`Flink分发目录中。

## 在本地执行作业

要使用小型集群将作业作为一个JVM进程在本地运行，必须将必需的hadoop依赖项显式添加到已启动的JVM进程的类路径中。

要使用maven运行应用程序（也来自IDE作为maven项目），可以将所需的hadoop依赖项添加到pom.xml中，例如：

```text
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>2.8.3</version>
    <scope>provided</scope>
</dependency>
```

这样，它既可以在本地运行又可以在群集运行中工作，在本地运行和群集运行中，如上所述，将提供的依赖项添加到相应位置。

要在IntelliJ Idea中运行或调试应用程序，可以在“运行\|编辑配置”窗口的类路径中包含提供的依赖项。

