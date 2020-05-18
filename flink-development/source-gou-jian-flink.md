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

## 构建PyFlink

 如果要构建可用于pip安装的PyFlink软件包，则需要首先构建Flink jar，如[Build Flink中所述](https://ci.apache.org/projects/flink/flink-docs-release-1.10/flinkDev/building.html#build-flink)。然后转到flink源代码的根目录，并运行以下命令来构建sdist包和wheel包：

```text
cd flink-python; python setup.py sdist bdist_wheel
```

{% hint style="info" %}
构建PyFlink需要Python版本（3.5、3.6或3.7）。
{% endhint %}

sdist和wheel package可以在./flink-python/dist/下找到。它们中的任何一个都可用于安装pip，例如:

```text
python -m pip install dist/*.tar.gz
```

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

 请参阅有关如何处理Hadoop类和版本的[Hadoop集成部分](https://ci.apache.org/projects/flink/flink-docs-release-1.10/ops/deployment/hadoop.html)。

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

