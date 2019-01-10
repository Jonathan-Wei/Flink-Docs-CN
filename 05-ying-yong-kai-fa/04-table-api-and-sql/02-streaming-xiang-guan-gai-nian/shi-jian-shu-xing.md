# 时间属性

## 时间属性介绍

## 处理时间\(Processing Time\)

有两种方法可以定义处理时间属性。

### 在DataStream到表转换期间

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

### 在DataStream到表转换期间

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

