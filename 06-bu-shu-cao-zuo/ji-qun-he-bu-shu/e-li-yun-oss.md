# 阿里云OSS

## OSS：对象存储服务

[阿里云对象存储服务](https://www.aliyun.com/product/oss)（Aliyun OSS）在中国的云用户中得到了广泛的应用，它为各种用例提供​​了云对象存储。

自版本2.9.1起，[Hadoop文件系统](http://hadoop.apache.org/docs/current/hadoop-aliyun/tools/hadoop-aliyun/index.html)支持OSS。现在，你还可以使用带有Fink的OSS来**读取**和**写入数据**。

你可以像这样访问OSS对象：

```text
oss://<your-bucket>/<object-name>
```

下面显示了如何在Flink中使用OSS：

```text
// Read from OSS bucket
env.readTextFile("oss://<your-bucket>/<object-name>");

// Write to OSS bucket
dataSet.writeAsText("oss://<your-bucket>/<object-name>")
```

有两种方法可以在Flink中使用OSS，我们的Shaded `flink-oss-fs-hadoop`将涵盖大多数场景。但是，如果要将OSS用作YARN的资源存储目录（[此修补程序](https://issues.apache.org/jira/browse/HADOOP-15919)使YARN能够使用OSS），则可能需要设置特定的Hadoop OSS文件系统实现。两种方式如下所述。

### Shaded Hadoop OSS文件系统（推荐）

要使用`flink-oss-fs-hadoop`，请在启动Flink之前将相应的JAR文件从opt目录复制到Flink发行版的lib目录，例如

```text
cp ./opt/flink-oss-fs-hadoop-1.9-SNAPSHOT.jar ./lib/
```

`flink-oss-fs-hadoop` 使用oss// 方案为URI注册默认的FileSystem包装器。

**配置设置**

设置OSS FileSystem包装器后，需要添加一些配置以确保允许Flink访问您的OSS存储桶。

为了更轻松地将OSS与Flink一起使用，可以在`flink-conf.yaml`中使用与Hadoop的`core-site.xml`中相同的配置键。

可以在[Hadoop OSS文档中](http://hadoop.apache.org/docs/current/hadoop-aliyun/tools/hadoop-aliyun/index.html)查看配置参数。

必须添加一些必需的配置`flink-conf.yaml`（**Hadoop OSS文档中定义的其他配置是性能调整使用的高级配置**）：

```text
fs.oss.endpoint: Aliyun OSS endpoint to connect to
fs.oss.accessKeyId: Aliyun access key ID
fs.oss.accessKeySecret: Aliyun access key secret
```

### Hadoop提供的OSS文件系统 - 手动设置

这个设置稍微复杂一点，我们建议使用我们的Shaded Hadoop文件系统（见上文），除非另有要求，例如通过Hadoop的core-site.xml中的fs.defaultFS配置属性将OSS用作YARN的资源存储目录。

#### **设置OSS文件系统**

需要将Flink指向一个有效的Hadoop配置，该配置在core-site.xml中包含以下属性：

```text
<configuration>

<property>
    <name>fs.oss.impl</name>
    <value>org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystem</value>
  </property>

  <property>
    <name>fs.oss.endpoint</name>
    <value>Aliyun OSS endpoint to connect to</value>
    <description>Aliyun OSS endpoint to connect to. An up-to-date list is provided in the Aliyun OSS Documentation.</description>
  </property>

  <property>
    <name>fs.oss.accessKeyId</name>
    <description>Aliyun access key ID</description>
  </property>

  <property>
    <name>fs.oss.accessKeySecret</name>
    <description>Aliyun access key secret</description>
  </property>

  <property>
    <name>fs.oss.buffer.dir</name>
    <value>/tmp/oss</value>
  </property>

</property>

</configuration>
```

#### **Hadoop配置**

你可以通过各种方式指定[Hadoop配置](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html#hdfs)，将Flink指向Hadoop配置目录的路径

* 通过设置环境变量`HADOOP_CONF_DIR`
* 通过`fs.hdfs.hadoopconf`在`flink-conf.yaml`以下位置设置配置选项：

```text
fs.hdfs.hadoopconf: /path/to/etc/hadoop
```

在Flink中注册Hadoop的配置目录`/path/to/etc/hadoop`。Flink将查找指定目录中的`core-site.xml`和`hdfs-site.xml`文件。

#### **提供OSS文件系统依赖性**

你可以找到Hadoop OSS FileSystem打包在hadoop-aliyun工件中。需要将此JAR及其所有依赖项添加到Flink的类路径中，即Job和TaskManagers的类路径。

有多种方法可以将JAR添加到Flink的类路径中，最简单的方法就是将JAR放到Flink的lib文件夹中。你需要复制hadoop-aliyun JAR及其所有依赖项（你可以在hadoop-3/share/hadoop/tools/lib中找到这些作为Hadoop二进制文件的一部分）。还可以在所有计算机上将包含这些JAR的目录导出为HADOOP\_CLASSPATH环境变量的一部分。

## 例子

下面是一个示例展示我们的设置结果（数据由TPC-DS工具生成）

```bash
// Read from OSS bucket
scala> val dataSet = benv.readTextFile("oss://<your-bucket>/50/call_center/data-m-00049")
dataSet: org.apache.flink.api.scala.DataSet[String] = org.apache.flink.api.scala.DataSet@31940704

scala> dataSet.print()
1|AAAAAAAABAAAAAAA|1998-01-01|||2450952|NY Metro|large|2935|1670015|8AM-4PM|Bob Belcher|6|More than other authori|Shared others could not count fully dollars. New members ca|Julius Tran|3|pri|6|cally|730|Ash Hill|Boulevard|Suite 0|Oak Grove|Williamson County|TN|38370|United States|-5|0.11|
2|AAAAAAAACAAAAAAA|1998-01-01|2000-12-31||2450806|Mid Atlantic|medium|1574|594972|8AM-8AM|Felipe Perkins|2|A bit narrow forms matter animals. Consist|Largely blank years put substantially deaf, new others. Question|Julius Durham|5|anti|1|ought|984|Center Hill|Way|Suite 70|Midway|Williamson County|TN|31904|United States|-5|0.12|
3|AAAAAAAACAAAAAAA|2001-01-01|||2450806|Mid Atlantic|medium|1574|1084486|8AM-4PM|Mark Hightower|2|Wrong troops shall work sometimes in a opti|Largely blank years put substantially deaf, new others. Question|Julius Durham|1|ought|2|able|984|Center Hill|Way|Suite 70|Midway|Williamson County|TN|31904|United States|-5|0.01|
4|AAAAAAAAEAAAAAAA|1998-01-01|2000-01-01||2451063|North Midwest|medium|10137|6578913|8AM-4PM|Larry Mccray|2|Dealers make most historical, direct students|Rich groups catch longer other fears; future,|Matthew Clifton|4|ese|3|pri|463|Pine Ridge|RD|Suite U|Five Points|Ziebach County|SD|56098|United States|-6|0.05|
5|AAAAAAAAEAAAAAAA|2000-01-02|2001-12-31||2451063|North Midwest|small|17398|4610470|8AM-8AM|Larry Mccray|2|Dealers make most historical, direct students|Blue, due beds come. Politicians would not make far thoughts. Specifically new horses partic|Gary Colburn|4|ese|3|pri|463|Pine Ridge|RD|Suite U|Five Points|Ziebach County|SD|56098|United States|-6|0.12|
6|AAAAAAAAEAAAAAAA|2002-01-01|||2451063|North Midwest|medium|13118|6585236|8AM-4PM|Larry Mccray|5|Silly particles could pro|Blue, due beds come. Politicians would not make far thoughts. Specifically new horses partic|Gary Colburn|5|anti|3|pri|463|Pine Ridge|RD|Suite U|Five Points|Ziebach County|SD|56098|United States|-6|0.11|
7|AAAAAAAAHAAAAAAA|1998-01-01|||2451024|Pacific Northwest|small|6280|1739560|8AM-4PM|Alden Snyder|6|Major, formal states can suppor|Reduced, subsequent bases could not lik|Frederick Weaver|5|anti|4|ese|415|Jefferson Tenth|Court|Suite 180|Riverside|Walker County|AL|39231|United States|-6|0.00|
8|AAAAAAAAIAAAAAAA|1998-01-01|2000-12-31||2450808|California|small|4766|2459256|8AM-12AM|Wayne Ray|6|Here possible notions arrive only. Ar|Common, free creditors should exper|Daniel Weller|5|anti|2|able|550|Cedar Elm|Ct.|Suite I|Fairview|Williamson County|TN|35709|United States|-5|0.06|

scala> dataSet.count()
res0: Long = 8

// Write to OSS bucket
scala> dataSet.writeAsText("oss://<your-bucket>/50/call_center/data-m-00049.1")

scala> benv.execute("My batch program")
res1: org.apache.flink.api.common.JobExecutionResult = org.apache.flink.api.common.JobExecutionResult@77476fcf

scala> val newDataSet = benv.readTextFile("oss://<your-bucket>/50/call_center/data-m-00049.1")
newDataSet: org.apache.flink.api.scala.DataSet[String] = org.apache.flink.api.scala.DataSet@40b70f31

scala> newDataSet.count()
res2: Long = 8
```

## 常见问题

### 找不到OSS文件系统

如果您的作业提交失败并显示如下所示的异常消息，请检查我们的Shaded jar（flink-oss-fs-hadoop-1.9-SNAPSHOT.jar）是否在lib目录中。

```text
Caused by: org.apache.flink.runtime.client.JobExecutionException: Could not set up JobManager
	at org.apache.flink.runtime.jobmaster.JobManagerRunner.<init>(JobManagerRunner.java:176)
	at org.apache.flink.runtime.dispatcher.Dispatcher$DefaultJobManagerRunnerFactory.createJobManagerRunner(Dispatcher.java:1058)
	at org.apache.flink.runtime.dispatcher.Dispatcher.lambda$createJobManagerRunner$5(Dispatcher.java:308)
	at org.apache.flink.util.function.CheckedSupplier.lambda$unchecked$0(CheckedSupplier.java:34)
	... 7 more
Caused by: org.apache.flink.runtime.JobException: Creating the input splits caused an error: Could not find a file system implementation for scheme 'oss'. The scheme is not directly supported by Flink and no Hadoop file system to support this scheme could be loaded.
	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.<init>(ExecutionJobVertex.java:273)
	at org.apache.flink.runtime.executiongraph.ExecutionGraph.attachJobGraph(ExecutionGraph.java:827)
	at org.apache.flink.runtime.executiongraph.ExecutionGraphBuilder.buildGraph(ExecutionGraphBuilder.java:232)
	at org.apache.flink.runtime.executiongraph.ExecutionGraphBuilder.buildGraph(ExecutionGraphBuilder.java:100)
	at org.apache.flink.runtime.jobmaster.JobMaster.createExecutionGraph(JobMaster.java:1151)
	at org.apache.flink.runtime.jobmaster.JobMaster.createAndRestoreExecutionGraph(JobMaster.java:1131)
	at org.apache.flink.runtime.jobmaster.JobMaster.<init>(JobMaster.java:294)
	at org.apache.flink.runtime.jobmaster.JobManagerRunner.<init>(JobManagerRunner.java:157)
	... 10 more
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Could not find a file system implementation for scheme 'oss'. The scheme is not directly supported by Flink and no Hadoop file system to support this scheme could be loaded.
	at org.apache.flink.core.fs.FileSystem.getUnguardedFileSystem(FileSystem.java:403)
	at org.apache.flink.core.fs.FileSystem.get(FileSystem.java:318)
	at org.apache.flink.core.fs.Path.getFileSystem(Path.java:298)
	at org.apache.flink.api.common.io.FileInputFormat.createInputSplits(FileInputFormat.java:587)
	at org.apache.flink.api.common.io.FileInputFormat.createInputSplits(FileInputFormat.java:62)
	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.<init>(ExecutionJobVertex.java:259)
	... 17 more
Caused by: org.apache.flink.core.fs.UnsupportedFileSystemSchemeException: Hadoop is not in the classpath/dependencies.
	at org.apache.flink.core.fs.UnsupportedSchemeFactory.create(UnsupportedSchemeFactory.java:64)
	at org.apache.flink.core.fs.FileSystem.getUnguardedFileSystem(FileSystem.java:399)
	... 22 more
```

### 缺少配置

如果你的作业提交失败，并显示如下所示的异常消息，请检查是否存在相应的配置 `flink-conf.yaml` 

```text
Caused by: org.apache.flink.runtime.JobException: Creating the input splits caused an error: Aliyun OSS endpoint should not be null or empty. Please set proper endpoint with 'fs.oss.endpoint'.
	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.<init>(ExecutionJobVertex.java:273)
	at org.apache.flink.runtime.executiongraph.ExecutionGraph.attachJobGraph(ExecutionGraph.java:827)
	at org.apache.flink.runtime.executiongraph.ExecutionGraphBuilder.buildGraph(ExecutionGraphBuilder.java:232)
	at org.apache.flink.runtime.executiongraph.ExecutionGraphBuilder.buildGraph(ExecutionGraphBuilder.java:100)
	at org.apache.flink.runtime.jobmaster.JobMaster.createExecutionGraph(JobMaster.java:1151)
	at org.apache.flink.runtime.jobmaster.JobMaster.createAndRestoreExecutionGraph(JobMaster.java:1131)
	at org.apache.flink.runtime.jobmaster.JobMaster.<init>(JobMaster.java:294)
	at org.apache.flink.runtime.jobmaster.JobManagerRunner.<init>(JobManagerRunner.java:157)
	... 10 more
Caused by: java.lang.IllegalArgumentException: Aliyun OSS endpoint should not be null or empty. Please set proper endpoint with 'fs.oss.endpoint'.
	at org.apache.flink.fs.shaded.hadoop3.org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystemStore.initialize(AliyunOSSFileSystemStore.java:145)
	at org.apache.flink.fs.shaded.hadoop3.org.apache.hadoop.fs.aliyun.oss.AliyunOSSFileSystem.initialize(AliyunOSSFileSystem.java:323)
	at org.apache.flink.fs.osshadoop.OSSFileSystemFactory.create(OSSFileSystemFactory.java:87)
	at org.apache.flink.core.fs.FileSystem.getUnguardedFileSystem(FileSystem.java:395)
	at org.apache.flink.core.fs.FileSystem.get(FileSystem.java:318)
	at org.apache.flink.core.fs.Path.getFileSystem(Path.java:298)
	at org.apache.flink.api.common.io.FileInputFormat.createInputSplits(FileInputFormat.java:587)
	at org.apache.flink.api.common.io.FileInputFormat.createInputSplits(FileInputFormat.java:62)
	at org.apache.flink.runtime.executiongraph.ExecutionJobVertex.<init>(ExecutionJobVertex.java:259)
	... 17 more
```

