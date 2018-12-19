# 配置Dependencies, Connectors, Libraries

## 配置依赖项、连接器和库

每个Flink应用程序都依赖于一组Flink库。至少，应用程序依赖于Flink api。许多应用程序还依赖于某些连接器库\(如Kafka、Cassandra等\)。在运行Flink应用程序时\(无论是在分布式部署中，还是在用于测试的IDE中\)，Flink运行时库也必须可用。

### Flink core以及应用程序依赖关系

与大多数运行用户定义应用程序的系统一样，Flink中有两大类依赖项和库:

* Flink核心依赖关系:Flink本身由一组类和运行系统所需的依赖关系,例如协调、网络、检查点,故障转移、api、操作\(如窗口\)、资源管理等。所有这些类和依赖项的集合形式Flink的核心运行时和Flink应用程序启动时必须存在。

  这些核心类和依赖项打包在flink-dist jar中。它们是Flink的lib文件夹和基本Flink容器映像的一部分。可以将这些依赖关系看作类似于Java的核心库\(rt.jar、字符集\)。，其中包含字符串和列表等类。

  Flink核心依赖项不包含任何连接器或库\(CEP、SQL、ML等\)，以避免默认情况下类路径中存在过多的依赖项和类。事实上，我们试图保持核心依赖项尽可能小，以使默认类路径尽可能小，并避免依赖项冲突。

* 用户应用程序依赖关系是特定用户应用程序需要的所有连接器、格式或库.

  用户应用程序通常打包到应用程序jar中，其中包含应用程序代码、所需的连接器和库依赖项。

  用户应用程序依赖项显式地不包括Flink DataSet/DataStream API和运行时依赖项，因为它们已经是Flink核心依赖项的一部分。

### 建立项目:基本依赖

每个Flink应用程序都需要最少的API依赖项来进行开发。对于Maven，您可以使用Java项目模板或Scala项目模板来创建带有这些初始依赖项的程序框架。

在手动设置项目时，您需要为Java/Scala API添加以下依赖项\(这里以Maven语法表示，但是相同的依赖项也适用于其他构建工具\(Gradle、SBT等\)。

Java

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-java</artifactId>
  <version>1.8-SNAPSHOT</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-java_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
  <scope>provided</scope>
</dependency>
```

Scala

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-scala_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
  <scope>provided</scope>
</dependency>
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-streaming-scala_2.11</artifactId>
  <version>1.8-SNAPSHOT</version>
  <scope>provided</scope>
</dependency>
```

重要提示:请注意，所有这些依赖项的范围都设置为已`provided`。这意味着需要根据它们进行编译，但是不应该将它们打包到项目的最终应用程序jar文件中——这些依赖关系是Flink核心依赖关系，在任何设置中都可以使用。

强烈建议在提供的范围内保持依赖关系。如果没有设置为`provided`，最好的情况是，得到的JAR会变得过大，因为它还包含所有Flink核心依赖项。最糟糕的情况是，添加到应用程序jar文件中的Flink核心依赖项与您自己的一些依赖项版本冲突\(通常通过反向类加载来避免\)。

关于IntelliJ的注意事项:要使应用程序在IntelliJ IDEA中运行，Flink依赖关系需要在作用域compile中声明，而不是提供。否则IntelliJ将不会将它们添加到类路径中，ide内的执行将会失败。为了避免宣布编译依赖范围\(不推荐,见上图\),上述有关Java - Scala项目模板使用技巧:他们添加一个配置文件,选择性地激活当应用程序运行在IntelliJ,才促进了依赖编译范围,而不影响包装的JAR文件。

## 添加Connector以及Library依赖

大多数应用程序需要特定的连接器或库来运行，例如到Kafka、Cassandra等的连接器。这些连接器不是Flink核心依赖项的一部分，因此必须作为对应用程序的依赖项添加

下面是一个为Kafka 0.10添加连接器作为依赖项的示例\(Maven语法\):

```markup
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-connector-kafka-0.10_2.11</artifactId>
    <version>1.8-SNAPSHOT</version>
</dependency>
```

我们建议将应用程序代码及其所有必需的依赖项打包到一个jar-with-dependencies中，我们将其称为应用程序jar。应用程序jar可以提交到已经运行的Flink集群，或者添加到Flink应用程序容器映像。

从Java项目模板或Scala项目模板创建的项目被配置为在运行mvn clean包时自动将应用程序依赖项包含到应用程序jar中。对于没有从这些模板中设置的项目，我们建议添加Maven Shade插件\(如下面附录中列出的\)来构建具有所有必需依赖项的应用程序jar。

重要提示:对于Maven\(和其他构建工具\)来说，要正确地将依赖项打包到应用程序jar中，必须在作用域编译中指定这些应用程序依赖项\(与核心依赖项不同，核心依赖项必须在提供的作用域中指定\)。

### Scala 版本

Scala版本\(2.10、2.11、2.12等等\)不是二进制兼容的。因此，Scala 2.11的Flink不能与使用Scala 2.12的应用程序一起使用。

所有\(暂时性地\)依赖Scala的Flink依赖项都添加了它们所针对的Scala版本的后缀，例如Flink -streaming-scala\_2.11。

只使用Java的开发人员可以选择任何Scala版本，Scala开发人员需要选择与他们的应用程序的Scala版本匹配的Scala版本。

有关如何为特定Scala版本构建Flink的详细信息，请参阅构建指南。

注意:由于Scala 2.12中有重大的突破性变化，Flink 1.5目前只针对Scala 2.11构建。我们的目标是在下一个版本中添加对Scala 2.12的支持。

### Hadoop依赖关系

一般规则:永远没有必要将Hadoop依赖项直接添加到应用程序中。\(唯一的例外是使用现有Hadoop输入/输出格式时使用Flink的Hadoop兼容性包装器\)

如果您想在Hadoop中使用Flink，您需要一个包含Hadoop依赖项的Flink设置，而不是将Hadoop作为应用程序依赖项添加。详情请参阅Hadoop安装指南。

这种设计有两个主要原因:

* 一些Hadoop交互发生在Flink的核心中，可能是在用户应用程序启动之前，例如为检查点设置HDFS、通过Hadoop的Kerberos令牌进行身份验证或在纱线上部署。
* Flink的反向类加载方法从核心依赖项中隐藏了许多可传递依赖项。这不仅适用于Flink自己的核心依赖项，也适用于设置中出现的Hadoop依赖项。这样，应用程序可以使用相同依赖项的不同版本，而不会遇到依赖项冲突\(相信我们，这很重要，因为hadoop依赖项树很大\)。

如果在IDE内部测试或开发期间需要Hadoop依赖项\(例如HDFS访问\)，请将这些依赖项配置为类似于要测试或提供的依赖项的范围。

### 附录:用依赖关系构建Jar的模板

要构建包含声明的连接器和库所需的所有依赖项的应用程序JAR，您可以使用以下的shade插件定义:

```text
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.0.0</version>
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

