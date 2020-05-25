# SSL设置

本页提供有关如何启用TLS / SSL身份验证和加密以与Flink进程之间进行网络通信的说明。

## 内部和外部连接

通过身份验证和加密保护机器进程之间的网络连接时，Apache Flink区分_内部_和_外部_连接。 _内部连接_是指Flink进程之间建立的所有连接。这些连接运行Flink自定义协议。用户永远不会直接连接到内部连接端点。 _外部/REST连接_端点是指从外部到Flink进程的所有连接。这包括用于启动和控制正在运行的Flink作业/应用程序的Web UI和REST命令，包括Flink CLI与JobManager / Dispatcher的通信。

为了获得更大的灵活性，可以分别启用和配置内部和外部连接的安全性。

![](../.gitbook/assets/image%20%2844%29.png)

### 内部连接

内部连接包括：

* 控制消息：JobManager / TaskManager / Dispatcher / ResourceManager之间的RPC
* 数据平面：TaskManager之间的连接，用于在随机播放，广播，重新分发等期间交换数据。
* Blob服务（库和其他工件的分发）。

 所有内部连接均经过SSL身份验证和加密。这些连接使用**相互身份验证**，这意味着每个连接的服务器端和客户端都需要相互提供证书。当使用专用CA对内部证书进行独占签名时，证书可以有效地充当共享机密。内部通信的证书不需要任何其他方与Flink进行交互，可以将其简单地添加到容器映像中，或附加到YARN部署中。



### 外部/REST连接

### 可查询状态

## 配置SSL

### 密钥库和信任库

#### **内部连接**

**REST端点（外部连接）**

### 密码套件

### SSL选项的完整列表

## 创建和部署密钥库和信任库

## 设置独立版和Kubernetes ****SSL示例

## YARN / Mesos部署建议

对于YARN和Mesos，可以使用Yarn和Mesos的工具来帮忙：

* 配置内部通信的安全性与上面的示例完全相同。
* 为了保护REST端点，需要颁发REST端点的证书，以使其对于Flink主服务器可能部署到的所有主机均有效。可以使用通配符DNS名称或添加多个DNS名称来完成。
*  部署密钥库和信任库的最简单方法是使用YARN客户端的选项（`-yt`）。将密钥库和信任库文件复制到本地目录（例如`deploy-keys/`），并按如下所示启动YARN会话：`flink run -m yarn-cluster -yt deploy-keys/ flinkapp.jar`
* 使用YARN部署时，可通过YARN代理的跟踪URL访问Flink的Web仪表板。为了确保YARN代理能够访问Flink的HTTPS URL，需要配置YARN代理以接受Flink的SSL证书。为此，请将自定义CA证书添加到YARN代理节点上的Java的默认信任库中。



