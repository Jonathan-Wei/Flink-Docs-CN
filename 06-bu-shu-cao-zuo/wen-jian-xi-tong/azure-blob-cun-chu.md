# Azure Blob存储

 [Azure Blob存储](https://docs.microsoft.com/en-us/azure/storage/)是一项Microsoft托管的服务，可为各种使用案例提供云存储。可以将Azure Blob存储与Flink 一起使用，用于[流](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)[**状态后端**](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/state/state_backends.html)**读取**和**写入数据**

通过指定以下格式的路径，可以使用Azure Blob存储对象（如常规文件）：

```text
wasb://<your-container>@$<your-azure-account>.blob.core.windows.net/<object-path>

// SSL encrypted access
wasbs://<your-container>@$<your-azure-account>.blob.core.windows.net/<object-path>
```

有关如何在Flink作业中使用Azure Blob存储的信息，请参见下文：

```text
// Read from Azure Blob storage
env.readTextFile("wasb://<your-container>@$<your-azure-account>.blob.core.windows.net/<object-path>");

// Write to Azure Blob storage
stream.writeAsText("wasb://<your-container>@$<your-azure-account>.blob.core.windows.net/<object-path>")

// Use Azure Blob Storage as FsStatebackend
env.setStateBackend(new FsStateBackend("wasb://<your-container>@$<your-azure-account>.blob.core.windows.net/<object-path>"));
```

## **影射**Hadoop Azure Blob存储文件系统

要在启动Flink之前使用`flink-azure-fs-hadoop,`相应的JAR文件从`opt`目录复制到`plugins`Flink发行版的目录，例如

```text
mkdir ./plugins/azure-fs-hadoop
cp ./opt/flink-azure-fs-hadoop-1.10.0.jar ./plugins/azure-fs-hadoop/
```

`flink-azure-fs-hadoop`使用_`wasb://`_和_`wasbs://`_（SSL加密访问）方式注册URI的默认FileSystem包装器。

## 凭证配置

Hadoop的Azure文件系统支持通过[Hadoop Azure Blob存储文档中](https://hadoop.apache.org/docs/current/hadoop-azure/index.html#Configuring_Credentials)概述的Hadoop配置来配置凭据。为方便起见，Flink将所有键前缀为`fs`的Flink配置转换成`fs.azure`的Hadoop文件系统配置。因此，可以通过以下方式在`flink-conf.yaml`配置azure blob存储密钥：

```text
fs.azure.account.key.<account_name>.blob.core.windows.net: <azure_storage_key>
```

或者，可以将文件系统配置为从环境变量`AZURE_STORAGE_KEY`中读取Azure Blob存储键，方法是在`flink-conf.yaml`中设置以下配置键。

```text
fs.azure.account.keyprovider.<account_name>.blob.core.windows.net: org.apache.flink.fs.azurefs.EnvironmentVariableKeyProvider
```

