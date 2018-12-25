---
description: 本页面提供了如何在静态(但可能是异构)集群上以完全分布式方式运行Flink的说明
---

# 独立集群

## 要求

### 软件要求

Flink可在所有_类UNIX环境中运行_，例如**Linux**，**Mac OS X**和**Cygwin**（适用于Windows），并期望群集由**一个主节点**和**一个或多个工作节点组成**。在开始设置系统之前，请确保**在每个节点上**安装了以下软件：

* **Java 1.8.x**或更高版本，
* **ssh**（必须运行sshd才能使用管理远程组件的Flink脚本）

如果您的群集不满足这些软件要求，则需要安装/升级它。

在所有群集节点上使用无**密码SSH**和 **相同的目录结构**将允许您使用我们的脚本来控制所有内容。

## JAVA\_HOME配置

Flink要求在`JAVA_HOME`主节点和所有工作节点上设置环境变量，并指向Java安装的目录。

您可以`conf/flink-conf.yaml`通过`env.java.home`键设置此变量。

## Flink配置

```text
tar xzf flink-*.tgz
cd flink-*
```

### 配置Flink

