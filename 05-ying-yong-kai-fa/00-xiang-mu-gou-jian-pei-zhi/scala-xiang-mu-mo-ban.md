# Scala项目模版

## 构建工具

## SBT

### 创建项目

使用以下命令之一来创建项目:

{% tabs %}
{% tab title="sbt模版" %}
```text
$ sbt new tillrohrmann/flink-project.g8
```

这里将提示您输入几个参数\(项目名称、flink版本…\)，然后从flink-project模板中创建flink项目。执行此命令需要sbt &gt;= 0.13.13。如果有必要，您可以遵循这个安装指南来获得它。
{% endtab %}

{% tab title="快速启动脚本" %}
```text
$ bash <(curl https://flink.apache.org/q/sbt-quickstart.sh)
```

这将会在您指定的项目目录中创建一个Flink项目。
{% endtab %}
{% endtabs %}

### 构建项目

如果要构建项目，您只需发出sbt clean assembly命令即可。这将在目录target/scala\_your-major-scala-version/中 创建your-project-name-assembly-0.1-SNAPSHOT.jar文件

### 运行项目

要运行项目，需要通过sbt run命令运行。

默认情况下，这将在与sbt运行的相同JVM中运行您的作业。为了在不同的JVM中运行作业，向build.sbt添加以下代码

```text
fork in run := true
```

#### IntelliJ

我们建议您在Flink job开发中使用[IntelliJ](https://www.jetbrains.com/idea/)。为了开始，您必须将新创建的项目导入到IntelliJ中。您可以通过现有来源的File -&gt; New -&gt; Project来实现这一点…然后选择项目的目录。然后IntelliJ会自动检测这个构建。sbt文件和设置一切。

为了运行Flink作业，建议选择mainRunner模块作为 Run/Debug 配置的类路径。这将确保所有设置为provide的依赖项在执行时可用。您可以通过Run -&gt; Edit配置来配置Run/Debug配置…然后从模块dropbox的classpath中选择mainRunner。

#### Eclipse

为了将新创建的项目导入Eclipse，首先必须为其创建[Eclipse](https://eclipse.org/)项目文件。这些项目文件可以通过[sbteclipse](https://github.com/typesafehub/sbteclipse)插件创建。向您的PROJECT\_DIR/project/plugins.sbt文件添加以下行:

```text
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")
```

在sbt中，使用以下命令创建Eclipse项目文件

```text
> eclipse
```

现在您可以通过File -&gt; Import... -&gt; Existing Projects，然后选择项目目录将现有项目导入到Eclipse中。

## Maven

### 前提

唯一的需求是Maven 3.0.4\(或更高版本\)和Java 8.x安装。

### 创建项目

使用以下命令之一来创建项目:

{% tabs %}
{% tab title="Maven方式" %}
```text
$ mvn archetype:generate                               \
   -DarchetypeGroupId=org.apache.flink              \
   -DarchetypeArtifactId=flink-quickstart-scala     \
   -DarchetypeCatalog=https://repository.apache.org/content/repositories/snapshots/ \
   -DarchetypeVersion=1.8-SNAPSHOT
```

这允许您命名新创建的项目。它将以交互式的方式询问您的groupId、artifactId和包名。
{% endtab %}

{% tab title="快速启动脚本" %}
```text
$ curl https://flink.apache.org/q/quickstart-scala-SNAPSHOT.sh | bash -s 1.8-SNAPSHOT
```
{% endtab %}
{% endtabs %}

### 检查项目

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

我们建议您将这个项目导入您的IDE中进行开发和测试。 IntelliJ IDEA支持Maven项目开箱即用。从我们的经验来看，IntelliJ提供了开发Flink应用程序的最佳经验。

对于Eclipse，您需要以下插件，您可以从提供的Eclipse更新站点安装这些插件:

* Eclipse 4.x
  * [Scala IDE](http://download.scala-ide.org/sdk/lithium/e44/scala211/stable/site)
  * [m2eclipse-scala](http://alchim31.free.fr/m2e-scala/update-site)
  * [Build Helper Maven Plugin](https://repo1.maven.org/maven2/.m2e/connectors/m2eclipse-buildhelper/0.15.0/N/0.15.0.201207090124/)
* Eclipse 3.8
  * [Scala IDE for Scala 2.11](http://download.scala-ide.org/sdk/helium/e38/scala211/stable/site) or [Scala IDE for Scala 2.10](http://download.scala-ide.org/sdk/helium/e38/scala210/stable/site)
  * [m2eclipse-scala](http://alchim31.free.fr/m2e-scala/update-site)
  * [Build Helper Maven Plugin](https://repository.sonatype.org/content/repositories/forge-sites/m2e-extras/0.14.0/N/0.14.0.201109282148/)

### 构建项目

如果您想build/package 项目，请转到项目目录并运行“mvn clean package”命令。您将找到一个JAR文件，其中包含您的应用程序，以及作为应用程序依赖项添加的连接器和库:target/-.jar。

注意:如果您使用与StreamingJob不同的类作为应用程序的主类/入口点，我们建议更改pom.xml配置文件中的mainClass设置。这样，Flink运行应用程序时则无需另外指定mainClass，直接运行jar包即可。

