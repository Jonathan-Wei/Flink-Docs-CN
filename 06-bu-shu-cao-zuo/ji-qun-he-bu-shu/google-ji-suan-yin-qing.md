# Google计算引擎

本文档提供了有关如何在[Google Compute Engine](https://cloud.google.com/compute/)群集上使用Hadoop 1或Hadoop 2完全自动设置Flink的说明。这是由Google的[bdutil实现的](https://cloud.google.com/hadoop/bdutil)，它启动了一个集群并使用Hadoop部署了Flink。要开始使用，请按照以下步骤操作。

## 先决条件 <a id="prerequisites"></a>

### 安装Google Cloud SDK <a id="install-google-cloud-sdk"></a>

请按照有关如何设置[Google Cloud SDK](https://cloud.google.com/sdk/)的说明进行操作。特别是，请务必使用以下命令通过Google Cloud进行身份验证：

```text
gcloud auth login
```

### 安装bdutil <a id="install-bdutil"></a>

目前还没有包含Flink扩展的bdutil发布。但是，您可以从[GitHub](https://github.com/GoogleCloudPlatform/bdutil)获得Flink支持的最新版本的bdutil ：

```text
git clone https://github.com/GoogleCloudPlatform/bdutil.git
```

下载源后，切换到新创建的`bdutil`目录并继续执行后续步骤。

## 在Google Compute Engine上部署Flink <a id="deploying-flink-on-google-compute-engine"></a>

### 设置一个桶\(Bucket\) <a id="set-up-a-bucket"></a>

如果尚未执行此操作，请为bdutil配置和暂存文件创建存储桶。可以使用gsutil创建一个新存储桶：

```text
gsutil mb gs://<bucket_name>
```

### 调整bdutil配置 <a id="adapt-the-bdutil-config"></a>

要使用bdutil部署Flink，请至少调整bdutil\_env.sh中的以下变量。

```text
CONFIGBUCKET="<bucket_name>"
PROJECT="<compute_engine_project_name>"
NUM_WORKERS=<number_of_workers>

# set this to 'n1-standard-2' if you're using the free trial
GCE_MACHINE_TYPE="<gce_machine_type>"

# for example: "europe-west1-d"
GCE_ZONE="<gce_zone>"
```

### 调整Flink配置 <a id="adapt-the-flink-config"></a>

bdutil的Flink扩展为你处理配置。你可以在`extensions/flink/flink_env.sh`另外调整配置变量。如果想进一步配置，请查看[配置Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html)。使用`bin/stop-cluster`和更改其配置后，必须重新启动Flink `bin/start-cluster`。

### 使用Flink创建一个集群 <a id="bring-up-a-cluster-with-flink"></a>

要在Google Compute Engine上调出Flink群集，请执行：

```text
./bdutil -e extensions/flink/flink_env.sh deploy
```

### 运行Flink示例作业： <a id="run-a-flink-example-job"></a>

```text
./bdutil shell
cd /home/hadoop/flink-install/bin
./flink run ../examples/batch/WordCount.jar gs://dataflow-samples/shakespeare/othello.txt gs://<bucket_name>/output
```

### 关闭群集 <a id="shut-down-your-cluster"></a>

关闭集群就像执行一样简单

```text
./bdutil -e extensions/flink/flink_env.sh delete
```



