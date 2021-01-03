# 项目配置

每个Flink应用程序都依赖于一组Flink库。至少，该应用程序取决于Flink API。此外，许多应用程序还依赖于某些连接器库（例如Kafka，Cassandra等）。当运行Flink应用程序时（在分布式部署中或在IDE中进行测试），Flink运行时库也必须可用。

## Flink核心和应用程序依赖性

与大多数运行用户定义应用程序的系统一样，Flink中有两大类依赖项和库：

* **Flink核心依赖项**：Flink本身包含运行系统所需的一组类和依赖项，例如，`coordination`, `networking`, `checkpoints`, `failover`, `APIs`, `operations` \(例如`windowing`\), `resource management`,等。这些类和依赖项构成Flink运行时的核心，并且在启动Flink应用程序时必须存在。

   这些核心类和依赖项打包在`flink-dist`jar中。它们是Flink`lib`文件夹的一部分，也是基本Flink容器映像的一部分。可以将这些依赖关系看作类似于Java的核心库\(rt.jar、charsets。jar等\)，其中包含String和List等类。

  Flink核心依赖项不包含任何连接器或库\(CEP、SQL、ML等\)，以避免在类路径中默认存在过多的依赖项和类。事实上，我们尽量保持核心依赖关系的精简，以保持默认的类路径较小，并避免依赖关系冲突。

* **用户应用程序依赖关系**是特定用户应用程序需要的所有connectors, formats, 或 libraries。

  用户应用程序通常打包到_应用程序jar中_，其中包含应用程序代码以及所需的connectors和libraries

  依赖项。

  用户应用程序依存关系明确不包括Flink DataStream API和运行时依存关系，因为它们已经是Flink核心依存关系的一部分

## 项目配置：基本依赖性

每个Flink应用程序都需要最少的API依赖来进行开发。

手动设置项目时，您需要为Java / Scala API添加以下依赖项（此处以Maven语法显示），但是相同的依赖项也适用于其他构建工具（Gradle，SBT等）。

{% tabs %}
{% tab title="Java" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-java_2.11</artifactId>
  <version>1.12.0</version>
  <scope>provided</scope>
</dependency>
```
{% endtab %}

{% tab title="Scala" %}
```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.12.0</version>
  <scope>provided</scope>
</dependency>
```
{% endtab %}
{% endtabs %}

 **重要提示：**请注意，所有这些依赖项的范围都设置为 _`provided`_。这意味着需要对其进行编译，但不应将它们打包到项目的结果应用程序jar文件中-这些依赖项是Flink Core依赖项，在任何设置中都已经可用。

强烈建议将依赖性保持在_`provided`_的范围内。如果没有将它们设置为_`provided`_，最好的情况是生成的JAR变得非常大，因为它也包含所有的Flink core依赖项。最坏的情况是添加到应用程序jar文件中的Flink核心依赖项与您自己的一些依赖项版本发生冲突\(这通常通过反向类加载来避免\)。

 **关于IntelliJ的注意事项：**为了让应用程序在IntelliJ IDEA中运行，有必要在运行配置中的“_`provided`_”范围框中勾选“`Include dependencies`”。如果这个选项不可用\(可能是由于使用旧的IntelliJ IDEA版本\)，那么一个简单的解决办法是创建一个调用应用程序`main()`方法的测试。

## 添加连接器和库依赖关系

大多数应用程序需要特定的连接器或库来运行，例如，连接到Kafka，Cassandra等的连接器。这些连接器不是Flink核心依赖项的一部分，必须作为依赖项添加到应用程序中。

下面是添加Kafka连接器作为依赖项的示例（Maven语法）：

```markup
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka_2.11</artifactId>
    <version>1.12.0</version>
</dependency>
```

我们建议将应用程序代码及其所需的所有依赖项打包到一个带有依赖项的jar中，我们将其称为应用程序jar。应用程序jar可以提交给已经运行的Flink集群，也可以添加到Flink应用程序容器映像中。

通过Java项目模板或Scala项目模板创建的项目被配置为在运行`mvn clean package`时自动将应用程序依赖项包含到应用程序jar中。对于那些不是通过这些模板建立的项目，我们建议添加Maven Shade插件\(如下面的附录中所列\)，以构建包含所有必需依赖项的应用程序jar。

**重要提示:**Maven\(和其他构建工具\)要想正确地将依赖项打包到应用程序jar中，必须在范围编译中指定这些应用程序依赖项\(与核心依赖项不同，核心依赖项必须在提供的范围中指定\)。

## Scala版本

Scala版本（2.11、2.12等）不是二进制兼容的。因此，Scala 2.11的Flink不能与使用Scala 2.12的应用程序一起使用。

例如，所有（短暂地）依赖于Scala的Flink依赖项都带有为其构建的Scala版本的后缀`flink-streaming-scala_2.11`。

仅使用Java的开发人员可以选择任何Scala版本，Scala开发人员需要选择与其应用程序的Scala版本匹配的Scala版本。

请参阅[构建指南](https://ci.apache.org/projects/flink/flink-docs-release-1.12/flinkDev/building.html#scala-versions) 以获取有关如何为特定Scala版本构建Flink的详细信息。

## Hadoop依赖关系

 **一般规则：永远不必将Hadoop依赖项直接添加到您的应用程序。** _（唯一的例外是将现有的Hadoop输入/输出格式与Flink的Hadoop兼容性包装器一起使用时）_

```text
export HADOOP_CLASSPATH=`hadoop classpath`
```

这种设计有两个主要原因:

* 某些Hadoop交互发生在Flink的核心中，可能在启动用户应用程序之前发生，例如，

  设置检查点为HDFS，通过Hadoop的Kerberos令牌进行身份验证或在YARN上进行部署。

* Flink的反向类加载方法从核心依赖中隐藏了许多传递依赖。这不仅适用于Flink自己的核心依赖关系，还适用于设置中存在的Hadoop依赖关系。这样，应用程序可以使用相同依赖性的不同版本，而不会遇到依赖性冲突（并且请相信我们，这很重要，因为Hadoop依赖性树很大。）

如果在IDE内进行测试或开发时（例如，用于HDFS访问）需要Hadoop依赖项，请配置这些依赖项，类似于要_测试_或_提供_的依赖项的范围。

## Maven快速入门

### **要求**

唯一的要求是需要**Maven 3.0.4**（或更高版本）和**Java8.x**。

### 创建项目

 使用以下命令之一**创建项目**：

{% tabs %}
{% tab title="使用maven原型" %}
```bash
$ mvn archetype:generate                               \
  -DarchetypeGroupId=org.apache.flink              \
  -DarchetypeArtifactId=flink-quickstart-java      \
  -DarchetypeVersion=1.11.2
```
{% endtab %}

{% tab title="运行quickstart脚本" %}
```bash
$ curl https://flink.apache.org/q/quickstart.sh | bash -s 1.11.2
```
{% endtab %}
{% endtabs %}

### 构建项目

## Gradle

### 要求

 唯一的要求是可以使用**Gradle 3.x**（或更高版本）和**Java 8.x**安装。

### 创建项目

 使用以下命令之一**创建项目**：

{% tabs %}
{% tab title="Gradle示例" %}
```text
### build.gradle

buildscript {
    repositories {
        jcenter() // this applies only to the Gradle 'Shadow' plugin
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:2.0.4'
    }
}

plugins {
    id 'java'
    id 'application'
    // shadow plugin to produce fat JARs
    id 'com.github.johnrengelman.shadow' version '2.0.4'
}


// artifact properties
group = 'org.myorg.quickstart'
version = '0.1-SNAPSHOT'
mainClassName = 'org.myorg.quickstart.StreamingJob'
description = """Flink Quickstart Job"""

ext {
    javaVersion = '1.8'
    flinkVersion = '1.11.2'
    scalaBinaryVersion = '2.11'
    slf4jVersion = '1.7.15'
    log4jVersion = '2.12.1'
}


sourceCompatibility = javaVersion
targetCompatibility = javaVersion
tasks.withType(JavaCompile) {
	options.encoding = 'UTF-8'
}

applicationDefaultJvmArgs = ["-Dlog4j.configurationFile=log4j2.properties"]

task wrapper(type: Wrapper) {
    gradleVersion = '3.1'
}

// declare where to find the dependencies of your project
repositories {
    mavenCentral()
    maven { url "https://repository.apache.org/content/repositories/snapshots/" }
}

// NOTE: We cannot use "compileOnly" or "shadow" configurations since then we could not run code
// in the IDE or with "gradle run". We also cannot exclude transitive dependencies from the
// shadowJar yet (see https://github.com/johnrengelman/shadow/issues/159).
// -> Explicitly define the // libraries we want to be included in the "flinkShadowJar" configuration!
configurations {
    flinkShadowJar // dependencies which go into the shadowJar

    // always exclude these (also from transitive dependencies) since they are provided by Flink
    flinkShadowJar.exclude group: 'org.apache.flink', module: 'force-shading'
    flinkShadowJar.exclude group: 'com.google.code.findbugs', module: 'jsr305'
    flinkShadowJar.exclude group: 'org.slf4j'
    flinkShadowJar.exclude group: 'org.apache.logging.log4j'
}

// declare the dependencies for your production and test code
dependencies {
    // --------------------------------------------------------------
    // Compile-time dependencies that should NOT be part of the
    // shadow jar and are provided in the lib folder of Flink
    // --------------------------------------------------------------
    compile "org.apache.flink:flink-streaming-java_${scalaBinaryVersion}:${flinkVersion}"

    // --------------------------------------------------------------
    // Dependencies that should be part of the shadow jar, e.g.
    // connectors. These must be in the flinkShadowJar configuration!
    // --------------------------------------------------------------
    //flinkShadowJar "org.apache.flink:flink-connector-kafka-0.11_${scalaBinaryVersion}:${flinkVersion}"

    compile "org.apache.logging.log4j:log4j-api:${log4jVersion}"
    compile "org.apache.logging.log4j:log4j-core:${log4jVersion}"
    compile "org.apache.logging.log4j:log4j-slf4j-impl:${log4jVersion}"
    compile "org.slf4j:slf4j-log4j12:${slf4jVersion}"

    // Add test dependencies here.
    // testCompile "junit:junit:4.12"
}

// make compileOnly dependencies available for tests:
sourceSets {
    main.compileClasspath += configurations.flinkShadowJar
    main.runtimeClasspath += configurations.flinkShadowJar

    test.compileClasspath += configurations.flinkShadowJar
    test.runtimeClasspath += configurations.flinkShadowJar

    javadoc.classpath += configurations.flinkShadowJar
}

run.classpath = sourceSets.main.runtimeClasspath

jar {
    manifest {
        attributes 'Built-By': System.getProperty('user.name'),
                'Build-Jdk': System.getProperty('java.version')
    }
}

shadowJar {
    configurations = [project.configurations.flinkShadowJar]
}
```

```text
### settings.gradle

rootProject.name = 'quickstart'
```
{% endtab %}

{% tab title="运行quickstart脚本" %}

{% endtab %}
{% endtabs %}

## SBT

### 项目创建

可以通过以下两种方法之一来创建新项目：

{% tabs %}
{% tab title="使用SBT模板" %}
```text
$ sbt new tillrohrmann/flink-project.g8
```

这里会提示你输入几个参数（项目名称，flink版本...），然后从[flink-project模板](https://github.com/tillrohrmann/flink-project.g8)创建一个Flink项目。需要sbt&gt; = 0.13.13才能执行此命令。如有必要，可以按照本[安装指南](http://www.scala-sbt.org/download.html)进行获取。
{% endtab %}

{% tab title="使用quickstart脚本" %}
```text
$ bash <(curl https://flink.apache.org/q/sbt-quickstart.sh)
```

 这将在**指定的**项目目录中创建一个Flink项目。
{% endtab %}
{% endtabs %}

### 构建项目

 构建项目，需要执行`sbt clean assembly`命令。这将在**target / scala\_your-major-scala-version /**目录中创建fat-jar **your-project-name-assembly-0.1-SNAPSHOT.jar**。

### **运行项目**

为了运行你的项目，必须执行`sbt run`命令。

默认情况下，这将在sbt运行的相同JVM中运行您的作业。。为了在不同的JVM中运行您的作业，将以下行添加到`build.sbt`

```text
fork in run := true
```

### **IntelliJ**

官方建议使用[IntelliJ](https://www.jetbrains.com/idea/) 来进行Flink作业开发。首先，必须将新创建的项目导入IntelliJ。你可以通过`File -> New -> Project from Existing Sources...`选择项目目录来执行此操作。然后，IntelliJ将自动检测`build.sbt`文件并进行所有设置。

为了运行Flink作业，建议选择`mainRunner`模块作为**Run / Debug Configuration**的类路径。这将确保设置为_提供的_所有依赖项在执行时均可用。你可以通过`Run -> Edit Configurations...`来配置**运行/调试配置**，然后然后从module dropbox的使用类路径中选择mainRunner。。

### **Eclipse**

 为了将新创建的项目导入[Eclipse中](https://eclipse.org/)，首先必须为其创建Eclipse项目文件。这些项目文件可以通过[sbteclipse](https://github.com/typesafehub/sbteclipse)插件创建。将以下行添加到你的`PROJECT_DIR/project/plugins.sbt`文件中：

```text
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")
```

 在`sbt`使用以下命令创建Eclipse项目文件

```text
> eclipse
```

 现在，可以通过将项目导入Eclipse `File -> Import... -> Existing Projects into Workspace`，然后选择项目目录。

## 附录：用于构建具有依赖关系的Jar的模板

要构建包含声明的连接器和库所需的所有依赖关系的应用程序JAR，可以使用以下maven插件定义：

```markup
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <artifactSet>
                            <excludes>
                                <exclude>com.google.code.findbugs:jsr305</exclude>
                                <exclude>org.slf4j:*</exclude>
                                <exclude>log4j:*</exclude>
                            </excludes>
                        </artifactSet>
                        <filters>
                            <filter>
                                <!-- Do not copy the signatures in the META-INF folder.
                                Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>my.programs.main.clazz</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

