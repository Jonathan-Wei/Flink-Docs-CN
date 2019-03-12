# Hadoop兼容性

Flink与Apache Hadoop MapReduce接口兼容，因此允许重用为Hadoop MapReduce实现的代码。

您可以：

* 在Flink程序中使用Hadoop的`Writable` [数据类型](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html#data-types)。
* 使用任何Hadoop `InputFormat`作为[数据源](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html#data-sources)。
* 使用任何Hadoop `OutputFormat`作为[DataSink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html#data-sinks)。
* 使用Hadoop `Mapper`作为[FlatMapFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#flatmap)。
* 使用Hadoop `Reducer`作为[GroupReduceFunction](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#groupreduce-on-grouped-dataset)。

本文档展示了如何在Flink中使用现有的Hadoop MapReduce代码。有关从Hadoop支持的文件系统中读取的信息，请参阅“ [连接到其他系统”](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/connectors.html)指南。

## 项目配置

Hadoop输入/输出格式的支持是FLink Java和FLink Scala Maven模块的一部分，在编写FLink作业时总是需要这些模块。代码位于`org.apache.flink.api.java.hadoop`和 `org.apache.flink.api.scala.hadoop`在附加的子包的 `mapred`和`mapreduce`API。

`flink-hadoop-compatibility` Maven模块中包含对Hadoop Mappers和Reducers的支持。此代码驻留在`org.apache.flink.hadoopcompatibility` 包中。

如果要重用Mappers和Reducers，请将以下依赖项添加到`pom.xml`。

```markup
<dependency>
	<groupId>org.apache.flink</groupId>
	<artifactId>flink-hadoop-compatibility_2.11</artifactId>
	<version>1.7.1</version>
</dependency>
```

## 使用Hadoop数据类型

Flink支持开箱即用的所有Hadoop `Writable`和`WritableComparable`数据类型。如果您只想使用Hadoop数据类型，则不需要包含Hadoop兼容性依赖项。有关详细信息，请参阅 [编程指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/index.html#data-types)。

## 使用Hadoop InputFormats

要将Hadoop InputFormats与Flink一起使用，必须首先使用`HadoopInputs`实用程序类的`readHadoopFile`或`createHadoopInput`来包装格式。前者用于从`FileInputFormat`后者输出的输入格式，而后者必须用于通用输入格式。结果`InputFormat`可用于通过使用创建数据源`ExecutionEnvironmen#createInput`。

结果`DataSet`包含2元组，其中第一个字段是键，第二个字段是从Hadoop InputFormat检索的值。

以下示例展示了如何使用Hadoop `TextInputFormat`。

{% tabs %}
{% tab title="Java" %}
```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

DataSet<Tuple2<LongWritable, Text>> input =
    env.createInput(HadoopInputs.readHadoopFile(new TextInputFormat(),
                        LongWritable.class, Text.class, textPath));

// Do something with the data.
[...]
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = ExecutionEnvironment.getExecutionEnvironment

val input: DataSet[(LongWritable, Text)] =
  env.createInput(HadoopInputs.readHadoopFile(
                    new TextInputFormat, classOf[LongWritable], classOf[Text], textPath))

// Do something with the data.
[...]

```
{% endtab %}
{% endtabs %}

## 使用Hadoop OutputFormats

Flink为Hadoop提供了兼容性包装器`OutputFormats`。支持实现`org.apache.hadoop.mapred.OutputFormat`或扩展`org.apache.hadoop.mapreduce.OutputFormat`的任何类。OutputFormat包装器期望其输入数据是包含2元组键和值的DataSet。这些将由Hadoop OutputFormat处理。

以下示例展示了如何使用Hadoop `TextOutputFormat`。

{% tabs %}
{% tab title="Java" %}
```java
// Obtain the result we want to emit
DataSet<Tuple2<Text, IntWritable>> hadoopResult = [...]

// Set up the Hadoop TextOutputFormat.
HadoopOutputFormat<Text, IntWritable> hadoopOF =
  // create the Flink wrapper.
  new HadoopOutputFormat<Text, IntWritable>(
    // set the Hadoop OutputFormat and specify the job.
    new TextOutputFormat<Text, IntWritable>(), job
  );
hadoopOF.getConfiguration().set("mapreduce.output.textoutputformat.separator", " ");
TextOutputFormat.setOutputPath(job, new Path(outputPath));

// Emit data using the Hadoop TextOutputFormat.
hadoopResult.output(hadoopOF);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Obtain your result to emit.
val hadoopResult: DataSet[(Text, IntWritable)] = [...]

val hadoopOF = new HadoopOutputFormat[Text,IntWritable](
  new TextOutputFormat[Text, IntWritable],
  new JobConf)

hadoopOF.getJobConf.set("mapred.textoutputformat.separator", " ")
FileOutputFormat.setOutputPath(hadoopOF.getJobConf, new Path(resultPath))

hadoopResult.output(hadoopOF)
```
{% endtab %}
{% endtabs %}

## 使用Hadoop Mappers和Reducers

Hadoop Mappers在语义上等同于Flink的[FlatMapFunctions](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#flatmap)，Hadoop [Reducers](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#flatmap)等同于Flink的[GroupReduceFunctions](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#groupreduce-on-grouped-dataset)。Flink为Hadoop MapReduce `Mapper`和`Reducer`接口的实现提供包装器，即，您可以在常规Flink程序中重用您的Hadoop Mappers和Reducers。目前，只支持hadoop的mapred api（`org.apache.hadoop.mapred`）的`mapper`和`reduce`接口。

包装器将`DataSet<Tuple2<KEYIN,VALUEIN>>`作为输入，并生成`DataSet<Tuple2<KEYOUT,VALUEOUT>>` 作为输出，其中`KEYIN`和`KEYOUT`是键，`VALUEIN`和`VALUEOUT`是Hadoop键值对的值，由Hadoop函数处理。对于Reducers，Flink为带有（`HadooPreduceCombineFunction`）和不带有Combiner（`HadooPreduceFunction`）的`GroupReduceFunction`提供了一个包装。包装器接受可选的`JobConf`对象来配置Hadoop映射器或Reducer。

Flink的函数包装器是

* `org.apache.flink.hadoopcompatibility.mapred.HadoopMapFunction`
* `org.apache.flink.hadoopcompatibility.mapred.HadoopReduceFunction`
* `org.apache.flink.hadoopcompatibility.mapred.HadoopReduceCombineFunction`

并可用作常规Flink [FlatMapFunctions](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#flatmap)或[GroupReduceFunctions](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/dataset_transformations.html#groupreduce-on-grouped-dataset)。

以下示例显示了如何使用Hadoop `Mapper`和`Reducer`函数。

```java
// Obtain data to process somehow.
DataSet<Tuple2<Text, LongWritable>> text = [...]

DataSet<Tuple2<Text, LongWritable>> result = text
  // use Hadoop Mapper (Tokenizer) as MapFunction
  .flatMap(new HadoopMapFunction<LongWritable, Text, Text, LongWritable>(
    new Tokenizer()
  ))
  .groupBy(0)
  // use Hadoop Reducer (Counter) as Reduce- and CombineFunction
  .reduceGroup(new HadoopReduceCombineFunction<Text, LongWritable, Text, LongWritable>(
    new Counter(), new Counter()
  ));
```

## 完成Hadoop WordCount示例

以下示例展示了使用Hadoop数据类型，`Input-`和`OutputFormats`以及Mapper和Reducer实现的完整WordCount实现。

```java
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();

// Set up the Hadoop TextInputFormat.
Job job = Job.getInstance();
HadoopInputFormat<LongWritable, Text> hadoopIF =
  new HadoopInputFormat<LongWritable, Text>(
    new TextInputFormat(), LongWritable.class, Text.class, job
  );
TextInputFormat.addInputPath(job, new Path(inputPath));

// Read data using the Hadoop TextInputFormat.
DataSet<Tuple2<LongWritable, Text>> text = env.createInput(hadoopIF);

DataSet<Tuple2<Text, LongWritable>> result = text
  // use Hadoop Mapper (Tokenizer) as MapFunction
  .flatMap(new HadoopMapFunction<LongWritable, Text, Text, LongWritable>(
    new Tokenizer()
  ))
  .groupBy(0)
  // use Hadoop Reducer (Counter) as Reduce- and CombineFunction
  .reduceGroup(new HadoopReduceCombineFunction<Text, LongWritable, Text, LongWritable>(
    new Counter(), new Counter()
  ));

// Set up the Hadoop TextOutputFormat.
HadoopOutputFormat<Text, IntWritable> hadoopOF =
  new HadoopOutputFormat<Text, IntWritable>(
    new TextOutputFormat<Text, IntWritable>(), job
  );
hadoopOF.getConfiguration().set("mapreduce.output.textoutputformat.separator", " ");
TextOutputFormat.setOutputPath(job, new Path(outputPath));

// Emit data using the Hadoop TextOutputFormat.
result.output(hadoopOF);

// Execute Program
env.execute("Hadoop WordCount");
```

