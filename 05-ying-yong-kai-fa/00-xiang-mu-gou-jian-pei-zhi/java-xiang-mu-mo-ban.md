# Java项目模版

## 前提

唯一的需求是Maven 3.0.4\(或更高版本\)和Java 8.x安装。

## 创建项目

使用以下命令之一来创建项目:

* Maven方式

  ```bash
  $ mvn archetype:generate                               \
     -DarchetypeGroupId=org.apache.flink              \
     -DarchetypeArtifactId=flink-quickstart-java      \
     -DarchetypeCatalog=https://repository.apache.org/content/repositories/snapshots/ \
     -DarchetypeVersion=1.8-SNAPSHOT
  ```

* 快速开始脚本

  ```bash
  $ curl https://flink.apache.org/q/quickstart-SNAPSHOT.sh | bash -s 1.8-SNAPSHOT
  ```

## 检查项目

这里将会在你的工作目录中创建一个新目录。如果您使用了curl方法，则该目录称为quickstart。否则，会以你的artifactId来创建:

```bash
$ tree quickstart/
quickstart/
├── pom.xml
└── src
    └── main
        ├── java
        │   └── org
        │       └── myorg
        │           └── quickstart
        │               ├── BatchJob.java
        │               └── StreamingJob.java
        └── resources
            └── log4j.properties
```

示例项目是一个Maven项目，它包含两个类:StreamingJob和BatchJob是DataStream和DataSet程序的基本框架程序。main\(\)方法是程序的入口点，用于ide测试/执行和适当的部署。

我们建议您将这个项目导入您的IDE中进行开发和测试。IntelliJ IDEA支持Maven项目开箱即用。如果使用Eclipse, m2e插件允许导入Maven项目。有些Eclipse包默认包含这个插件，有些则需要您手动安装。

给Mac OS X用户的提示:对于Flink来说，Java的默认JVM堆大小可能太小了。你必须手动增加。在Eclipse中，选择Run Configurations -&gt;Arguments 并写入VM参数框:- Xmx800m。在IntelliJ IDEA中建议的改变JVM选项的方法是从 Help\|Edit Custom VM Options菜单修改。有关详细信息，请参阅[此链接](https://intellij-support.jetbrains.com/hc/en-us/articles/206544869-Configuring-JVM-options-and-platform-properties)。

## 构建项目

如果您想build/package 项目，请转到项目目录并运行“mvn clean package”命令。您将找到一个JAR文件，其中包含您的应用程序，以及作为应用程序依赖项添加的连接器和库:target/-.jar。

注意:如果您使用与StreamingJob不同的类作为应用程序的主类/入口点，我们建议更改pom.xml配置文件中的mainClass设置。这样，Flink运行应用程序时则无需另外指定mainClass，直接运行jar包即可。

