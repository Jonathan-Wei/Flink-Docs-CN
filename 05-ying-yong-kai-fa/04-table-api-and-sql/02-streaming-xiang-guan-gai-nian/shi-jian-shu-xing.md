# 时间属性

Flink能够根据不同的_时间_概念处理流数据。

* _处理时间\(Processing Time\)：_指执行相应操作的机器的系统时间\(也称为“挂钟时间”\)。
* _事件时间\(Event Time\)：_指基于附加到每行上的时间戳处理流数据的过程。时间戳可以在事件发生时进行编码。
* _摄取时间\(Ingestion Time_ _\)：_事件进入Flink的时间;在内部，它的处理类似于事件时间。

有关Flink中时间处理的更多信息，请参阅有关[事件时间和水印的简介](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/event_time.html)。

本页介绍了如何在Flink的Table API和SQL中为基于时间的操作定义时间属性。

## 时间属性介绍

[Table API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#group-windows)和[SQL中](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#group-windows)基于时间的操作（如窗口）需要有关时间概念及其来源的信息。因此，表可以提供_逻辑时间属性，_用于指定时间和访问表程序中的相应时间戳。

时间属性可以是每个表模式的一部分。它们是在从DataStream创建表时定义的，或者是在使用表源时预定义的。一旦开始定义了时间属性，就可以将其引用为字段，并在基于时间的操作中使用。

只要时间属性没有被修改，并且只是从查询的一个部分转发到另一个部分，它就仍然是一个有效的时间属性。时间属性的行为类似于常规的时间戳，可用于计算。如果在计算中使用时间属性，它将被具体化并成为一个常规的时间戳。常规的时间戳与Flink的时间和水印系统不兼容，因此不能再用于基于时间的操作。

表程序要求为流环境指定相应的时间特征:

{% tabs %}
{% tab title="Java" %}
```java
final StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime); // default

// alternatively:
// env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime);
// env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env = StreamExecutionEnvironment.getExecutionEnvironment

env.setStreamTimeCharacteristic(TimeCharacteristic.ProcessingTime) // default

// alternatively:
// env.setStreamTimeCharacteristic(TimeCharacteristic.IngestionTime)
// env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
```
{% endtab %}
{% endtabs %}

## 处理时间\(Processing Time\)

处理时间允许表程序根据本地机器的时间生成结果。它是时间最简单的概念，但不提供决定论。它既不需要提取时间戳，也不需要生成水印。

有两种方法可以定义处理时间属性。

### 在DataStream到表转换期间

处理时间属性是在模式定义期间使用`.proctime`属性定义的。time属性只能通过附加的逻辑字段扩展物理模式。因此，它只能在模式定义的末尾定义。

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Tuple2<String, String>> stream = ...;

// declare an additional logical field as a processing time attribute
Table table = tEnv.fromDataStream(stream, "Username, Data, UserActionTime.proctime");

WindowedTable windowedTable = table.window(Tumble.over("10.minutes").on("UserActionTime").as("userActionWindow"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
val stream: DataStream[(String, String)] = ...

// declare an additional logical field as a processing time attribute
val table = tEnv.fromDataStream(stream, 'UserActionTimestamp, 'Username, 'Data, 'UserActionTime.proctime)

val windowedTable = table.window(Tumble over 10.minutes on 'UserActionTime as 'userActionWindow)
```
{% endtab %}
{% endtabs %}

### 使用TableSource

处理时间属性由实现`DefinedProctimeAttribute`接口的表源定义。逻辑时间属性被附加到由表源的返回类型定义的物理模式中。

{% tabs %}
{% tab title="Java" %}
```java
// define a table source with a processing attribute
public class UserActionSource implements StreamTableSource<Row>, DefinedProctimeAttribute {

	@Override
	public TypeInformation<Row> getReturnType() {
		String[] names = new String[] {"Username" , "Data"};
		TypeInformation[] types = new TypeInformation[] {Types.STRING(), Types.STRING()};
		return Types.ROW(names, types);
	}

	@Override
	public DataStream<Row> getDataStream(StreamExecutionEnvironment execEnv) {
		// create stream
		DataStream<Row> stream = ...;
		return stream;
	}

	@Override
	public String getProctimeAttribute() {
		// field with this name will be appended as a third field
		return "UserActionTime";
	}
}

// register table source
tEnv.registerTableSource("UserActions", new UserActionSource());

WindowedTable windowedTable = tEnv
	.scan("UserActions")
	.window(Tumble.over("10.minutes").on("UserActionTime").as("userActionWindow"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// define a table source with a processing attribute
class UserActionSource extends StreamTableSource[Row] with DefinedProctimeAttribute {

	override def getReturnType = {
		val names = Array[String]("Username" , "Data")
		val types = Array[TypeInformation[_]](Types.STRING, Types.STRING)
		Types.ROW(names, types)
	}

	override def getDataStream(execEnv: StreamExecutionEnvironment): DataStream[Row] = {
		// create stream
		val stream = ...
		stream
	}

	override def getProctimeAttribute = {
		// field with this name will be appended as a third field
		"UserActionTime"
	}
}

// register table source
tEnv.registerTableSource("UserActions", new UserActionSource)

val windowedTable = tEnv
	.scan("UserActions")
	.window(Tumble over 10.minutes on 'UserActionTime as 'userActionWindow)
```
{% endtab %}
{% endtabs %}

## 事件时间\(Event Time\)

事件时间允许表程序根据每个记录中包含的时间生成结果。即使在无序事件或延迟事件的情况下，这也允许一致的结果。它还确保在从持久存储中读取记录时，表程序的结果是可重放的。

此外，事件时间允许在批处理和流环境中对表程序使用统一的语法。流环境中的时间属性可以是批处理环境中的记录的常规字段。

为了处理无序事件和区分流中的准时事件和延迟事件，Flink需要从事件中提取时间戳并在时间上取得某种进展\(即所谓的[水印](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/event_time.html)\)。

事件时间属性既可以在`Datastream`到`Table`的转换过程中定义，也可以通过使用`TableSource`定义。

### 在DataStream到表转换期间

事件时间属性是在模式定义期间使用`.rowtime`属性定义的。[时间戳和水印](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/event_time.html)必须在转换的数据流中分配。

在将数据流转换为表时，有两种定义时间属性的方法。取决于指定的`.rowtime`字段名是否存在于`DataStream`模式中，`timestamp`字段要么是：

* 作为新字段添加到模式
* 替换现有字段。

无论哪种情况，事件时间戳字段都将保存DataStream事件时间戳的值。

{% tabs %}
{% tab title="Java" %}
```java
// Option 1:

// extract timestamp and assign watermarks based on knowledge of the stream
DataStream<Tuple2<String, String>> stream = inputStream.assignTimestampsAndWatermarks(...);

// declare an additional logical field as an event time attribute
Table table = tEnv.fromDataStream(stream, "Username, Data, UserActionTime.rowtime");


// Option 2:

// extract timestamp from first field, and assign watermarks based on knowledge of the stream
DataStream<Tuple3<Long, String, String>> stream = inputStream.assignTimestampsAndWatermarks(...);

// the first field has been used for timestamp extraction, and is no longer necessary
// replace first field with a logical event time attribute
Table table = tEnv.fromDataStream(stream, "UserActionTime.rowtime, Username, Data");

// Usage:

WindowedTable windowedTable = table.window(Tumble.over("10.minutes").on("UserActionTime").as("userActionWindow"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Option 1:

// extract timestamp and assign watermarks based on knowledge of the stream
val stream: DataStream[(String, String)] = inputStream.assignTimestampsAndWatermarks(...)

// declare an additional logical field as an event time attribute
val table = tEnv.fromDataStream(stream, 'Username, 'Data, 'UserActionTime.rowtime)


// Option 2:

// extract timestamp from first field, and assign watermarks based on knowledge of the stream
val stream: DataStream[(Long, String, String)] = inputStream.assignTimestampsAndWatermarks(...)

// the first field has been used for timestamp extraction, and is no longer necessary
// replace first field with a logical event time attribute
val table = tEnv.fromDataStream(stream, 'UserActionTime.rowtime, 'Username, 'Data)

// Usage:

val windowedTable = table.window(Tumble over 10.minutes on 'UserActionTime as 'userActionWindow)
```
{% endtab %}
{% endtabs %}

### 使用TableSource

事件时间属性由实现`DefinedRowtimeAttributes`接口的TableSource定义。`getRowtimeAttributeDescriptors()`方法返回一个`RowtimeAttributeDescriptor`列表，用于描述时间属性的最终名称、用于派生属性值的时间戳提取器和与属性关联的水印策略。

请确保`getDataStream()`方法返回的`DataStream`与定义的`time`属性对齐。`DataStream`的时间戳\(由`TimestampAssigner`分配的时间戳\)只在定义了`StreamRecordTimestamp`时间戳提取器时才会考虑。只有定义了保留水印策略，数据流的水印才会被保留。否则，只有`TableSource`的`rowtime`属性的值是相关的。

{% tabs %}
{% tab title="Java" %}
```java
// define a table source with a rowtime attribute
public class UserActionSource implements StreamTableSource<Row>, DefinedRowtimeAttribute {

	@Override
	public TypeInformation<Row> getReturnType() {
		String[] names = new String[] {"Username", "Data", "UserActionTime"};
		TypeInformation[] types =
		    new TypeInformation[] {Types.STRING(), Types.STRING(), Types.LONG()};
		return Types.ROW(names, types);
	}

	@Override
	public DataStream<Row> getDataStream(StreamExecutionEnvironment execEnv) {
		// create stream
		// ...
		// assign watermarks based on the "UserActionTime" attribute
		DataStream<Row> stream = inputStream.assignTimestampsAndWatermarks(...);
		return stream;
	}

	@Override
	public String getRowtimeAttribute() {
		// Mark the "UserActionTime" attribute as event-time attribute.
		return "UserActionTime";
	}
}

// register the table source
tEnv.registerTableSource("UserActions", new UserActionSource());

WindowedTable windowedTable = tEnv
	.scan("UserActions")
	.window(Tumble.over("10.minutes").on("UserActionTime").as("userActionWindow"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// define a table source with a rowtime attribute
class UserActionSource extends StreamTableSource[Row] with DefinedRowtimeAttribute {

	override def getReturnType = {
		val names = Array[String]("Username" , "Data", "UserActionTime")
		val types = Array[TypeInformation[_]](Types.STRING, Types.STRING, Types.LONG)
		Types.ROW(names, types)
	}

	override def getDataStream(execEnv: StreamExecutionEnvironment): DataStream[Row] = {
		// create stream
		// ...
		// assign watermarks based on the "UserActionTime" attribute
		val stream = inputStream.assignTimestampsAndWatermarks(...)
		stream
	}

	override def getRowtimeAttribute = {
		// Mark the "UserActionTime" attribute as event-time attribute.
		"UserActionTime"
	}
}

// register the table source
tEnv.registerTableSource("UserActions", new UserActionSource)

val windowedTable = tEnv
	.scan("UserActions")
	.window(Tumble over 10.minutes on 'UserActionTime as 'userActionWindow)
```
{% endtab %}
{% endtabs %}

