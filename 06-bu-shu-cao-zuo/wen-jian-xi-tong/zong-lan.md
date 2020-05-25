# 总览

Apache Flink使用文件系统来消耗和持久存储数据，以获取应用程序的结果以及容错和恢复。这些都是一些最流行的文件系统，包括_本地_，_Hadoop的兼容_，_亚马逊S3_，_MAPR FS_，_OpenStack的斯威夫特FS_，_阿里云OSS_和_Azure的Blob存储_。

用于特定文件的文件系统由其URI方式确定。例如，`file:///home/user/text.txt`是指本地文件系统中的文件，而`hdfs://namenode:50010/data/user/text.txt`指的是特定HDFS群集中的文件。

文件系统实例每个进程实例化一次，然后进行缓存/池化，以避免每次流创建时的配置开销，并强制执行某些约束，例如连接/流限制。

## 本地文件系统

 Flink内置了对本地计算机文件系统的支持，包括安装在该本地文件系统中的所有NFS或SAN驱动器。默认情况下可以使用它，而无需其他配置。本地文件使用_`file://`_`URI`方式引用。

## 可插拔文件系统

Apache Flink项目支持以下文件系统：

* Amazon S3对象存储由两种可选实现支持:`flink-s3-fs-presto`和`flink-s3-fs-hadoop`。这两个实现都是自包含的，没有依赖项占用。
* 在`maprfs:// URI`方案下的主Flink发行版中已经支持MapR FS文件系统适配器。必须在类路径中提供MapR库\(例如在lib目录中\)。
* OpenStack Swift FS受`flink-swift-fs-hadoop`支持，注册在`Swift:// URI`方案下。该实现基于Hadoop项目，但是是自包含的，没有依赖项。要在使用Flink作为库时使用它，请添加相应的maven依赖项\(`org.apache.flink: Flink -swift-fs-hadoop:1.10.0`\)。
* 阿里云对象存储服务受`flink-os -fs-hadoop`支持，注册在`oss:// URI`方案下。该实现基于Hadoop项目，但是是自包含的，没有依赖项。

除了**MapR FS**，可以并且应该使用它们中的任何一个作为 [插件](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/plugins.html)。

要使用可插入文件系统，请在启动Flink之前将相应的JAR文件从`opt`目录复制到Flink发行版目录下的`plugins`目录中，例如

```text
mkdir ./plugins/s3-fs-hadoop
cp ./opt/flink-s3-fs-hadoop-1.10.0.jar ./plugins/s3-fs-hadoop/
```

 **注意**：Flink 1.9版本引入了文件系统的插件机制，以支持每个插件使用专用的Java类加载器，并摆脱了类shading机制。通过旧的机制，仍然可以使用提供的文件系统\(或您自己的实现\)，方法是将相应的JAR文件复制到lib目录中。但是，**从1.10开始，s3插件必须通过插件机制加载**;旧的方法不再生效，因为这些插件不再是shaded的\(或者更具体地讲，自1.10版本以来，这些类不再重定位\)。

鼓励对支持该功能的文件系统使用基于[插件](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/plugins.html)的加载机制。 将来的Flink版本不支持从`lib`目录中加载文件系统组件。

## 添加新的可插拔文件系统实现

文件系统通过 `org.apache.flink.core.fs.FileSystem` 类，它捕获访问和修改该文件系统中的文件和对象的方法。

* 添加文件系统实现，它是的子类`org.apache.flink.core.fs.FileSystem`。
* 添加一个实例化文件系统并声明用于注册FileSystem方式的工厂。必须是

  `org.apache.flink.core.fs.FileSystemFactory`的子类。

* 添加服务条目。创建一个包含`META-INF/services/org.apache.flink.core.fs.FileSystemFactory`

  文件系统工厂类的类名的文件（有关更多详细信息，请参见[Java Service Loader文档](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)）。

在插件发现期间，文件系统工厂类将由专用的Java类加载器加载，以避免与其他插件和Flink组件的类冲突。在文件系统实例化和文件系统操作调用期间，应使用相同的类加载器。

{% hint style="warning" %}
警告： 实际上，这意味着您应该避免在实现中使用Thread.currentThread\(\). getcontextclassloader\(\)类加载器。
{% endhint %}

## Hadoop文件系统（HDFS）及其其他实现

对于所有Flink无法找到直接支持的文件系统的方案，将回到Hadoop。当`flink-runtime`和Hadoop库位于类路径上时，所有Hadoop文件系统都自动可用。请参见Hadoop集成。

通过这种方式， Flink无缝支持实现该`org.apache.hadoop.fs.FileSystem`接口的所有Hadoop文件系统以及所有Hadoop兼容文件系统（HCFS）。

* HDFS（已测试）
* [适用于Hadoop的Google Cloud Storage连接器](https://cloud.google.com/hadoop/google-cloud-storage-connector)（已测试）
* [Alluxio](http://alluxio.org/)（经过测试，请参阅下面的配置详细信息）
* [XtreemFS](http://www.xtreemfs.org/)（已测试）
* 通过[Hftp的](http://hadoop.apache.org/docs/r1.2.1/hftp.html) FTP （未经测试）
* HAR（未测试）
* …

Hadoop配置必须在`core-site.xml`文件中为所需的文件系统实现提供一个条目。参见Alluxio示例。

我们建议使用Flink的内置文件系统，除非另有要求。直接使用Hadoop文件系统可能是必需的，例如，当通过Hadoop的`core-site.xml`中的`fs.defaultFS`配置属性将该文件系统用于YARN的资源存储时。

### Alluxio

 对于Alluxio支持，将以下条目添加到`core-site.xml`文件中：

```text
<property>
  <name>fs.alluxio.impl</name>
  <value>alluxio.hadoop.FileSystem</value>
</property>
```

