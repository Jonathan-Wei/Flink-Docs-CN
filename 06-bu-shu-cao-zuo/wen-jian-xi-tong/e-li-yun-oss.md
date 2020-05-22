# 阿里云OSS

### OSS：对象存储服务 <a id="oss-object-storage-service"></a>

 [阿里云对象存储服务](https://www.aliyun.com/product/oss)（Aliyun OSS）被广泛使用，在中国的云用户中尤为流行，它为各种用例提供​​云对象存储。可以使用带有Flink的OSS来读取和写入数据，也可以与[流](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)[**状态后端**](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)一起使用

可以通过指定以下格式的路径来使用OSS对象（如常规文件）：

```text
oss://<your-bucket>/<object-name>
```

下面展示了如何在Flink作业中使用OSS：

```text
// Read from OSS bucket
env.readTextFile("oss://<your-bucket>/<object-name>");

// Write to OSS bucket
stream.writeAsText("oss://<your-bucket>/<object-name>")

// Use OSS as FsStatebackend
env.setStateBackend(new FsStateBackend("oss://<your-bucket>/<object-name>"));
```

#### 影射Hadoop OSS文件系统 <a id="shaded-hadoop-oss-file-system"></a>

要在启动Flink之前使用`flink-oss-fs-hadoop,`相应的JAR文件从`opt`目录复制到`plugins`Flink发行版目录中的目录，例如

```text
mkdir ./plugins/oss-fs-hadoop
cp ./opt/flink-oss-fs-hadoop-1.10.0.jar ./plugins/oss-fs-hadoop/
```

`flink-oss-fs-hadoop`使用_oss://方式_为URI注册默认的FileSystem包装器。

**配置设置**

在设置OSS文件系统包装器之后，您需要添加一些配置，以确保Flink被允许访问您的OSS存储段。

为了便于采用，可以在flink-conf.yaml中使用与Hadoop的core-site.xml相同的配置键。

可以在[Hadoop OSS文档中](http://hadoop.apache.org/docs/current/hadoop-aliyun/tools/hadoop-aliyun/index.html)看到配置密钥。

需要在`flink-conf.yaml`添加一些必需的配置（**Hadoop OSS文档中定义的其他配置是性能调整所使用的高级配置**）：

```text
fs.oss.endpoint: Aliyun OSS endpoint to connect to
fs.oss.accessKeyId: Aliyun access key ID
fs.oss.accessKeySecret: Aliyun access key secret
```

还可以在flink-conf.yaml中配置另一个CredentialsProvider。

```text
# Read Credentials from OSS_ACCESS_KEY_ID and OSS_ACCESS_KEY_SECRET
fs.oss.credentials.provider: com.aliyun.oss.common.auth.EnvironmentVariableCredentialsProvider
```

其他凭证提供者可以在https://github.com/aliyun/aliyun-oss-java-sdk/tree/master/src/main/java/com/aliyun/oss/common/auth下找到。

