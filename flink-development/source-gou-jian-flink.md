# Source构建Flink

## 构建Flink

为了构建Flink，需要源代码。无论是[下载一个Release源](http://flink.apache.org/downloads.html)或[克隆Git仓库](https://github.com/apache/flink)。

此外，还需要**Maven 3**和**JDK**（Java Development Kit）。Flink **至少**需要**Java 8**才能构建。

_注意：Maven 3.3.x可以构建Flink，但不会正确地遮蔽某些依赖项。Maven 3.2.5正确创建了库。要构建单元测试，请使用Java 8u51或更高版本来防止使用PowerMock运行程序的单元测试失败。_

要从git克隆，请输入：

```text
git clone https://github.com/apache/flink
```

构建Flink的最简单方法是运行：

```text
mvn clean install -DskipTests
```

这指示[Maven](http://maven.apache.org/)（`mvn`）首先删除所有现有的构建（`clean`），然后创建一个新的Flink二进制文件（`install`）。

要加快构建速度，您可以跳过测试，QA插件和JavaDocs：

```text
mvn clean install -DskipTests -Dfast
```

默认构建为Hadoop 2添加了特定于Flink的JAR，以允许将Flink与HDFS和YARN一起使用。

## 依赖遮蔽

Flink[遮挡](https://maven.apache.org/plugins/maven-shade-plugin/)一些使用的库中，以避免与使用不同版本的这些库的用户程序版本冲突。遮蔽的库包括_Google Guava_，_Asm_，_Apache Curator_，_Apache HTTP Components_，_Netty_等。

最近在Maven中更改了依赖关系遮蔽机制，并要求用户根据Maven版本略微不同地构建Flink：

**Maven 3.0.x，3.1.x和3.2.x** 只需调用`mvn clean install -DskipTests`Flink代码库的根目录即可。

**Maven 3.3.x** 构建必须分两步完成：首先在基本目录中，然后在分发项目中：

```text
mvn clean install -DskipTests
cd flink-dist
mvn clean install
```

_注意：_要检查Maven版本，请运行`mvn --version`。

## Hadoop版本

{% hint style="info" %}
大多数用户不需要手动执行此操作。该[下载页面](http://flink.apache.org/downloads.html)包含了常见的Hadoop版本的二进制软件包。
{% endhint %}

Flink依赖于HDFS和YARN，它们都是来自[Apache Hadoop的](http://hadoop.apache.org/)依赖项。存在许多不同版本的Hadoop（来自上游项目和不同的Hadoop发行版）。如果使用错误的版本组合，则可能发生异常。

Hadoop仅从2.4.0版本开始支持。您还可以指定要构建的特定Hadoop版本：

```text
mvn clean install -DskipTests -Dhadoop.version=2.6.1
```

### 供应商特定版本

要针对特定​​于供应商的Hadoop版本构建Flink，请执行以下命令：

```text
mvn clean install -DskipTests -Pvendor-repos -Dhadoop.version=2.6.1-cdh5.0.0
```

在`-Pvendor-repos`激活一个Maven [构建Profile](http://maven.apache.org/guides/introduction/introduction-to-profiles.html)，其包括流行的Hadoop厂商如Cloudera的，Hortonworks，或MAPR的存储库。

## Scala版本

{% hint style="info" %}
纯粹使用Java API和库的用户可以_忽略_此部分。
{% endhint %}

Flink具有用[Scala](http://scala-lang.org/)编写的API，库和运行时模块。Scala API和库的用户可能必须将Flink的Scala版本与其项目的Scala版本匹配（因为Scala不是严格向后兼容的）。

从版本1.7开始，Flink使用Scala版本2.11和2.12构建。

## 加密文件系统

如果您的主目录已加密，则可能会遇到`java.io.IOException: File name too long`异常。某些加密文件系统（如Ubuntu使用的encfs）不允许长文件名，这是导致此错误的原因。

解决方法是添加：

```text
<args>
    <arg>-Xmax-classfile-name</arg>
    <arg>128</arg>
</args>
```

在`pom.xml`导致错误的模块文件的编译器配置中。例如，如果`flink-yarn`模块中出现错误，则应在`<configuration>`标记下添加上述代码`scala-maven-plugin`。有关更多信息，请参阅[此问题](https://issues.apache.org/jira/browse/FLINK-2003)。

