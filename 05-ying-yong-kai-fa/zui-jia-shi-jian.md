# 最佳实践

## 解析命令行参数并在Flink应用程序中传递

几乎所有Flink应用程序（批处理和流式处理）都依赖于外部配置参数。它们用于指定输入和输出源（如路径或地址），系统参数（并行性，运行时配置）和特定于应用程序的参数（通常在用户功能中使用）。

Flink提供了一个简单的实用程序，`ParameterTool`用于提供一些解决这些问题的基本工具。请注意，您不必使用`ParameterTool`此处描述的内容。其他框架（如[Commons CLI](https://commons.apache.org/proper/commons-cli/)和 [argparse4j）](http://argparse4j.sourceforge.net/)也适用于Flink。

### `ParameterTool`获取配置值 

`ParameterTool`提供了一组用于读取配置的预定义静态方法。该工具在内部期望 `Map<String, String>`，因此很容易将其与您自己的配置风格集成。

#### **来自.properties文件**

以下方法将读取[属性](https://docs.oracle.com/javase/tutorial/essential/environment/properties.html)文件并提供键/值对：

```java
String propertiesFilePath = "/home/sam/flink/myjob.properties";
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFilePath);

File propertiesFile = new File(propertiesFilePath);
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFile);

InputStream propertiesFileInputStream = new FileInputStream(file);
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFileInputStream);
```

#### **命令行参数**

允许从命令行获取参数`--input hdfs:///mydata --elements 42`。

```java
public static void main(String[] args) {
    ParameterTool parameter = ParameterTool.fromArgs(args);
    // .. regular code ..
```

#### **来自系统属性**

启动JVM时，可以将系统属性传递给它：`-Dinput=hdfs:///mydata`。还可以`ParameterTool`从这些系统属性初始化：

```java
ParameterTool parameter = ParameterTool.fromSystemProperties();
```

### 使用Flink程序中的参数

现在我们已经从某个地方获得了参数（见上文），我们可以通过各种方式使用它们。

**直接来自 `ParameterTool`**

`ParameterTool`本身有访问值的方法。

```java
ParameterTool parameters = // ...
parameter.getRequired("input");
parameter.get("output", "myDefaultValue");
parameter.getLong("expectedCount", -1L);
parameter.getNumberOfParameters()
// .. there are more methods available.
```

可以直接在提交应用程序的客户端的`main()`方法中使用这些方法的返回值。例如，可以设置运算符的并行度，如下所示：

```java
ParameterTool parameters = ParameterTool.fromArgs(args);
int parallelism = parameters.get("mapParallelism", 2);
DataSet<Tuple2<String, Integer>> counts = text.flatMap(new Tokenizer()).setParallelism(parallelism);
```

由于`ParameterTool`可序列化，可以将其传递给函数本身：

```java
ParameterTool parameters = ParameterTool.fromArgs(args);
DataSet<Tuple2<String, Integer>> counts = text.flatMap(new Tokenizer(parameters));
```

然后在函数内部使用它从命令行获取值。

#### **全局注册参数**

在ExecutionConfig中注册为全局作业参数的参数可以作为配置值从JobManager web接口和用户定义的所有函数中访问。

全局注册参数：

```java
ParameterTool parameters = ParameterTool.fromArgs(args);

// set up the execution environment
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().setGlobalJobParameters(parameters);
```

在任何丰富的用户函数中访问它们：

```java
public static final class Tokenizer extends RichFlatMapFunction<String, Tuple2<String, Integer>> {

    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
	ParameterTool parameters = (ParameterTool)
	    getRuntimeContext().getExecutionConfig().getGlobalJobParameters();
	parameters.getRequired("input");
	// .. do more ..
```

## 命名大型TupleX类型

对于具有许多字段的数据类型，建议使用pojo\(普通的旧Java对象\)而不是TupleX。此外，pojo还可以用于为大型元类型提供名称。

**例**

而不是使用：

```java
Tuple11<String, String, ..., String> var = new ...;
```

创建从大型元组类型扩展的自定义类型要容易得多。

```java
CustomType var = new ...;

public static class CustomType extends Tuple11<String, String, ..., String> {
    // constructor matching super
}
```

## 使用Logback而不是Log4j

**注意：本教程适用于从Flink 0.10开始**

Apache Flink使用[slf4j](http://www.slf4j.org/)作为代码中的日志记录抽象。建议用户在其用户功能中使用sfl4j。

Sfl4j是一个编译时日志记录接口，可以在运行时使用不同的日志记录实现，例如[log4j](http://logging.apache.org/log4j/2.x/)或[Logback](http://logback.qos.ch/)。

Flink默认依赖于Log4j。本页介绍如何使用Flink with Logback。用户报告说他们也可以使用本教程使用Graylog设置集中式日志记录。

要在代码中获取记录器实例，请使用以下代码：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyClass implements MapFunction {
    private static final Logger LOG = LoggerFactory.getLogger(MyClass.class);
    // ...
```

### 使用IDE从Java应用程序运行FLink时使用Logback

在所有情况下，类都是由依赖管理器（如Maven）创建的类路径执行的，Flink会将log4j拉入类路径。

因此，您需要从Flink的依赖项中排除log4j。以下描述将假设从[Flink快速入门](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/projectsetup/java_api_quickstart.html)创建的Maven项目。

像这样更改项目文件`pom.xml`：

```markup
<dependencies>
	<!-- Add the two required logback dependencies -->
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-core</artifactId>
		<version>1.1.3</version>
	</dependency>
	<dependency>
		<groupId>ch.qos.logback</groupId>
		<artifactId>logback-classic</artifactId>
		<version>1.1.3</version>
	</dependency>

	<!-- Add the log4j -> sfl4j (-> logback) bridge into the classpath
	 Hadoop is logging to log4j! -->
	<dependency>
		<groupId>org.slf4j</groupId>
		<artifactId>log4j-over-slf4j</artifactId>
		<version>1.7.7</version>
	</dependency>

	<dependency>
		<groupId>org.apache.flink</groupId>
		<artifactId>flink-java</artifactId>
		<version>1.7.1</version>
		<exclusions>
			<exclusion>
				<groupId>log4j</groupId>
				<artifactId>*</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.flink</groupId>
		<artifactId>flink-streaming-java_2.11</artifactId>
		<version>1.7.1</version>
		<exclusions>
			<exclusion>
				<groupId>log4j</groupId>
				<artifactId>*</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>org.apache.flink</groupId>
		<artifactId>flink-clients_2.11</artifactId>
		<version>1.7.1</version>
		<exclusions>
			<exclusion>
				<groupId>log4j</groupId>
				<artifactId>*</artifactId>
			</exclusion>
			<exclusion>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
</dependencies>
```

该`<dependencies>`部分进行了以下更改：

* 从所有Flink依赖项中排除所有log4j依赖项:这会导致Maven忽略Flink对log4j的传递依赖项。 
* 从Flink的依赖项中排除`slf4j-log4j12`工件:因为我们要使用slf4j来进行回滚绑定，所以我们必须删除slf4j到log4j的绑定。
* 添加Logback依赖项：`logback-core`和`logback-classic`
* 添加依赖项`log4j-over-slf4j`。`log4j-over-slf4j`是一种工具，它允许直接使用Log4j API的遗留应用程序使用Slf4j接口。Flink依赖于Hadoop，它直接使用Log4j进行日志记录。因此，我们需要将所有记录器调用从Log4j重定向到Slf4j，而Slf4j又记录到Logback。

请注意，需要手动将排除项添加到要添加到POM文件的所有新Flink依赖项中。

可能还需要检查其他（非Flink）依赖项是否正在引入log4j绑定。可以使用分析项目的依赖关系`mvn dependency:tree`。

### 在群集上运行Flink时使用Logback

本教程适用于在YARN上运行Flink或作为独立群集。

要使用Logback而不是使用Flink的Log4j，您需要从目录中删除`log4j-1.2.xx.jar`和。`sfl4j-log4j12-xxx.jarlib/`

接下来，您需要将以下jar文件放入该`lib/`文件夹：

* `logback-classic.jar`
* `logback-core.jar`
* `log4j-over-slf4j.jar`：此桥接器需要存在于类路径中，以便将日志记录调用从Hadoop（使用Log4j）重定向到Slf4j。

注意，在使用每个作业Yarn集群时，需要显式地设置`lib/`目录。

使用自定义记录器在YARN上提交Flink的命令是： `./bin/flink run -yt $FLINK_HOME/lib <... remaining arguments ...>`

