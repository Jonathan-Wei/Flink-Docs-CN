# 亚马逊S3

 [Amazon Simple Storage Service](http://aws.amazon.com/s3/)（Amazon S3）为各种使用场景提供​​云对象存储。可以将S3与Flink 一起使用，用于[流](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)[**状态后端**](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)**读取**和**写入数据**。

可以通过以下格式指定路径来使用S3对象（如常规文件）：

```text
s3://<your-bucket>/<endpoint>
```

端点可以是单个文件或目录，例如：

```text
// Read from S3 bucket
env.readTextFile("s3://<bucket>/<endpoint>");

// Write to S3 bucket
stream.writeAsText("s3://<bucket>/<endpoint>");

// Use S3 as FsStatebackend
env.setStateBackend(new FsStateBackend("s3://<your-bucket>/<endpoint>"));
```

请注意，这些示例_并不_详尽，也可以在其他地方使用S3，包括[高可用性设置](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/jobmanager_high_availability.html)或[RocksDBStateBackend](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html#the-rocksdbstatebackend)；Flink需要文件系统URI的任何地方。

对于大多数用例，可以使用我们的文件系统插件`flink-s3-fs-hadoop`和`flink-s3-fs-presto`S3文件系统插件之一，它们是自带的，易于设置。但是，在某些情况下，例如，使用S3作为YARN的资源存储目录时，可能有必要设置特定的Hadoop S3文件系统实现。

### Hadoop / Presto S3文件系统插件

{% hint style="info" %}
 **注意：**如果您正在[EMR](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-flink.html)上运行[Flink，](https://docs.aws.amazon.com/emr/latest/ReleaseGuide/emr-flink.html)则不必手动配置它。
{% endhint %}

Flink提供了两个方式来与Amazon S3进行通信： `flink-s3-fs-presto`以及`flink-s3-fs-hadoop`。这两个实现都是自带的，没有依赖项，因此不需要将Hadoop添加到类路径中来使用它们。

* `flink-s3-fs-presto`：基于[Presto项目](https://prestodb.io/)代码，以_`s3://`_和_`s3p://`_方式注册，。可以通过将配置添加到`flink-conf.yaml`的方式进行[配置](https://prestodb.io/docs/0.187/connector/hive.html#amazon-s3-configuration)，与[配置Presto文件系统](https://prestodb.io/docs/0.187/connector/hive.html#amazon-s3-configuration)的方式相同。推荐使用Presto文件系统来检查S3。
* `flink-s3-fs-hadoop`：基于[Hadoop项目](https://hadoop.apache.org/)代码，以_`s3://`_和_`s3a://`_方式注册。通过将配置添加到flink- confi .yaml中，文件系统可以像Hadoop的s3a一样进行配置。它是唯一支持[StreamingFileSink的](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/connectors/streamfile_sink.html) S3文件系统。

flink-s3-fs-hadoop和flink-s3-fs-presto都使用`s3://`模式为uri注册默认文件系统包装器，flink-s3-fs-hadoop也可以使用s3a://注册， flink-s3-fs-presto也可以使用s3p://注册，因此可以同时使用二者。例如，作业使用仅支持Hadoop的`StreamingFileSink`，但使用Presto进行检查点。在这种情况下，建议显式地使用`s3a://`作为接收器\(Hadoop\)的方案，使用`s3p://`作为检查点\(Presto\)的方案。

要使用`flink-s3-fs-hadoop`或`flink-s3-fs-presto`，请在启动Flink之前将相应的JAR文件从`opt`目录复制到Flink发行版的`plugins`目录下，例如

```text
mkdir ./plugins/s3-fs-presto
cp ./opt/flink-s3-fs-presto-1.10.0.jar ./plugins/s3-fs-presto/
```

#### **配置访问凭证**

设置S3 FileSystem包装器之后，需要确保允许Flink访问S3存储桶。

身份和访问管理\(IAM\)\(推荐\)

在AWS上设置凭证的推荐方法是通过[身份和访问管理（IAM）](http://docs.aws.amazon.com/IAM/latest/UserGuide/introduction.html)。您可以使用IAM功能安全地为Flink实例提供访问S3存储桶所需的凭据。有关如何执行此操作的详细信息超出了本文档的范围。请参阅AWS用户指南。您正在寻找的是[IAM角色](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html)。

如果设置正确，则可以在AWS中管理对S3的访问，而无需将任何访问密钥分配给Flink。

**访问密钥（已淘汰）**

可以通过**访问密钥对**来授予对S3的**访问权限**。请注意，自从[引入IAM角色](https://blogs.aws.amazon.com/security/post/Tx1XG3FX6VMU6O5/A-safer-way-to-distribute-AWS-credentials-to-EC2)之后，不建议这样做。

需要在Flink的`flink-conf.yaml`中同时配置`s3.access-key`和`s3.secret-key` ：

```text
s3.access-key: your-access-key
s3.secret-key: your-secret-key
```

## 配置非S3端点

S3文件系统还支持使用兼容S3的对象存储，例如[IBM的Cloud Object Storage](https://www.ibm.com/cloud/object-storage)和[Minio](https://min.io/)。为此，请在`flink-conf.yaml`中配置端点。

```text
s3.endpoint: your-endpoint-hostname
```

## 配置路径样式访问

默认情况下，某些符合S3的对象存储可能未启用虚拟主机样式寻址。在这种情况下，将必须提供属性以启用中的路径样式访问`flink-conf.yaml`。

```text
s3.path.style.access: true
```

## S3文件系统的熵注入**\(**Entropy injection**\)**

捆绑的S3文件系统（`flink-s3-fs-presto`和`flink-s3-fs-hadoop`）支持熵注入。熵注入是一种通过在密钥的开头附近添加一些随机字符来提高AWS S3存储桶的可伸缩性的技术。

如果激活了熵注入，则将路径中配置的子字符串替换为随机字符。例如，路径 `s3://my-bucket/checkpoints/_entropy_/dashboard-job/`将被替换为`s3://my-bucket/checkpoints/gf36ikvg/dashboard-job/`。 **这仅在文件创建通过注入熵的选项时发生！** 否则，文件路径会完全删除熵密钥子字符串。有关 详细信息[，](https://ci.apache.org/projects/flink/flink-docs-release-1.6/api/java/org/apache/flink/core/fs/FileSystem.html#create-org.apache.flink.core.fs.Path-org.apache.flink.core.fs.FileSystem.WriteOptions-)请参见[FileSystem.create（Path，WriteOption）](https://ci.apache.org/projects/flink/flink-docs-release-1.6/api/java/org/apache/flink/core/fs/FileSystem.html#create-org.apache.flink.core.fs.Path-org.apache.flink.core.fs.FileSystem.WriteOptions-)。

{% hint style="info" %}
**注意**:Flink运行时目前只向检查点数据文件注入熵的选项。所有其他文件，包括检查点元数据和外部URI，都不会注入熵来保持检查点URI的可预测性。
{% endhint %}

要启用熵注入，请配置_熵密钥_和_熵长度_参数。

```text
s3.entropy.key: _entropy_
s3.entropy.length: 4 (default)
```

s3.entropy.key定义路径中的字符串，该路径由随机字符替换。 不包含熵密钥的路径保持不变。如果文件系统操作未通过_“注入熵”_写选项，则仅删除熵密钥子字符串。 `s3.entropy.length`定义了用于熵的随机的字母数字字符的长度。

