# 调试类加载

## Flink中的类加载概述

运行Flink应用程序时，JVM将随着时间的推移加载各种类。这些类可以根据其来源分为三类：

*  **Java Classpath**：这是Java的公共类路径，它包括JDK库和Flink的/lib文件夹中的所有代码\(Apache Flink的类和一些依赖项\)。
*  **Flink Plugin Components：**插件代码在Flink的/plugins文件夹下的文件夹中。Flink的插件机制会在启动时动态加载一次。
*  **Dynamic User Code：**这些都是包含在动态提交作业的JAR文件中的类\(通过REST、CLI、web UI\)。它们是针对每个作业动态加载\(和卸载\)的。

通常，无论何时首先启动Flink进程，然后提交作业，作业的类都会动态加载。如果Flink进程与作业/应用程序一起启动，或者如果应用程序生成Flink组件\(JobManager、TaskManager等\)，那么所有作业的类都在Java类路径中。

插件组件中的代码由每个插件的专用类加载器动态加载一次。

以下是有关不同部署模式的更多详细信息：

**Standalone Session**

 当将Flink集群作为独立会话启动时，JobManager和TaskManager随Java类路径中的Flink框架类一起启动。_动态地_加载针对会话（通过REST / CLI）提交的所有作业/应用程序中的类。

 **Docker / Kubernetes Session**

Docker / Kubernetes设置首先启动一组JobManager / TaskManager，然后通过REST或CLI提交作业/应用程序，就像独立会话一样：Flink的代码在Java类路径中，插件组件在启动时动态加载，而作业的代码加载动态地。

**Yarn**

YARN类加载在单个作业部署和会话之间有所不同：

* 将Flink作业/应用程序直接提交到YARN（通过`bin/flink run -m yarn-cluster ...`）时，将启动该作业的专用TaskManager和JobManager。这些JVM在Java类路径中具有用户代码类。这意味着在这种情况下，该作业_不_涉及_动态类加载_。
* 启动YARN会话时，将使用类路径中的Flink框架类来启动JobManager和TaskManager。动态加载针对该会话提交的所有作业中的类。

**Mesos**

 目前，遵循[此文档的](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/mesos.html) Mesos设置的行为非常类似于YARN会话：TaskManager和JobManager进程以Java类路径中的Flink框架类启动，提交作业时动态加载作业类。

## 反向类加载和类加载器解析顺序

在涉及动态类加载的设置（插件组件，会话设置中的Flink作业）中，通常有两个ClassLoader层次结构：（1）Java _应用程序classloader_，它在类路径中包含所有类，以及（2）动态_插件/用户代码类加载器_。用于从插件或用户代码jar加载类。动态ClassLoader将应用程序类加载器作为其父级。

默认情况下，Flink会反转类加载顺序，这意味着它首先查看动态类加载器，并且仅在类不属于动态加载的代码时才查看父类（应用程序类加载器）。

## 避免为用户代码动态加载类

## 用户代码中的手动类加载

## X不能转换为X异常

## 用户代码中动态加载的类的卸载

## 使用maven-shade-plugin解决Flink的依赖冲突

