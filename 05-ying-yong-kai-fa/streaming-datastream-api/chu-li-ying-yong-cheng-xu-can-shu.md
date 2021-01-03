# 处理应用程序参数

## 处理应用程序参数

几乎所有的Flink应用程序（批处理和流式处理）都依赖于外部配置参数。它们用于指定输入和输出源（如路径或地址），系统参数（并行性，运行时配置）和特定于应用程序的参数（通常在用户功能内使用）。

Flink提供了一个名为`ParameterTool`的简单实用程序，它为解决这些问题提供了一些基本工具。请注意，您不必使用这里描述的参数工具。 其他框架（例如[Commons CLI](https://commons.apache.org/proper/commons-cli/)和 [argparse4j）](http://argparse4j.sourceforge.net/)也可以与Flink一起使用。

### 将配置值放入 `ParameterTool`

ParameterTool提供了一组用于读取配置的预定义静态方法。这个工具在内部需要一个`Map<String，Str>`，因此很容易将其与您自己的配置样式集成。

#### **通过.properties文件**

下面的方法将读取一个 [Properties](https://docs.oracle.com/javase/tutorial/essential/environment/properties.html)文件并提供键/值对:

```java
String propertiesFilePath = "/home/sam/flink/myjob.properties";
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFilePath);

File propertiesFile = new File(propertiesFilePath);
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFile);

InputStream propertiesFileInputStream = new FileInputStream(file);
ParameterTool parameter = ParameterTool.fromPropertiesFile(propertiesFileInputStream);
```

#### 通过命令行参数

 可以这样从命令行获取参数`--input hdfs:///mydata --elements 42`

```java
public static void main(String[] args) {
    ParameterTool parameter = ParameterTool.fromArgs(args);
    // .. regular code ..
```

#### 通过系统属性

当启动一个JVM时，你可以把系统属性传递给它，你也可以从这些系统属性初始化参数工具:

```java
ParameterTool parameter = ParameterTool.fromSystemProperties();
```

### 使用Flink程序中的参数

现在我们已经从某个地方获得了参数\(见上文\)，我们可以以各种方式使用它们。

**直接通过ParameterTool**

ParameterTool本身有访问这些值的方法。

```java
ParameterTool parameters = // ...
parameter.getRequired("input");
parameter.get("output", "myDefaultValue");
parameter.getLong("expectedCount", -1L);
parameter.getNumberOfParameters()
// .. there are more methods available.
```

你可以在提交应用程序的客户机的`main()`方法中直接使用这些方法的返回值。例如，你可以这样设置一个操作符的并行度:

```java
ParameterTool parameters = ParameterTool.fromArgs(args);
int parallelism = parameters.get("mapParallelism", 2);
DataSet<Tuple2<String, Integer>> counts = text.flatMap(new Tokenizer()).setParallelism(parallelism);
```

因为ParameterTool是可序列化的，你可以把它传递给函数本身:

```java
ParameterTool parameters = ParameterTool.fromArgs(args);
DataSet<Tuple2<String, Integer>> counts = text.flatMap(new Tokenizer(parameters));
```

然后在函数中使用它从命令行获取值。

#### 全局注册参数

在`ExecutionConfig`中注册为全局作业参数的参数可以作为配置值从`JobManager` web界面和用户定义的所有函数中访问。

全局注册参数:

```java
ParameterTool parameters = ParameterTool.fromArgs(args);

// set up the execution environment
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().setGlobalJobParameters(parameters);
```

在任何Rich用户函数中访问参数值:

```java
public static final class Tokenizer extends RichFlatMapFunction<String, Tuple2<String, Integer>> {

    @Override
    public void flatMap(String value, Collector<Tuple2<String, Integer>> out) {
	ParameterTool parameters = (ParameterTool)
	    getRuntimeContext().getExecutionConfig().getGlobalJobParameters();
	parameters.getRequired("input");
	// .. do more ..
```

