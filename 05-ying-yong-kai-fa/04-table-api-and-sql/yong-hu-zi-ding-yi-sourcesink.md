# 用户自定义Source&Sink

`TableSource`提供对存储在外部系统（数据库，键值存储，消息队列）或文件中的数据的访问。[TableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource)在[TableEnvironment中注册后](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource)，可以通过[Table API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html)或[SQL](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html)查询访问它。

`TableSink` [将表发送](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#emit-a-table)到外部存储系统，例如数据库，键值存储，消息队列或文件系统（在不同的编码中，例如CSV，Parquet或ORC）。

TableFactory允许将到外部系统的连接声明与实际实现分离。TableFactory从规范化的、基于字符串的属性创建表源和接收的配置实例。可以使用描述符或通过[SQL客户端](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sqlClient.html)的YAML配置文件以编程方式生成属性。

有关如何[注册TableSource](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesource)以及如何[通过TableSink发出表的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#emit-a-table)详细信息，请查看[常见概念和API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html)页面。有关如何使用工厂的示例[，](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html)请参阅[内置源，接收器和格式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/connect.html)页面。

## 定义TableSource

`TableSource`是一个通用接口，它允许Table API和SQL查询访问存储在外部系统中的数据。它提供了表的模式以及映射到具有表模式的行的记录。根据`TableSource`是在流式查询还是批量查询中使用，记录将生成为`DataSet`或`DataStream`。

如果在流查询中使用了一个`TableSource`，它必须实现`StreamTableSource`源接口，如果在批处理查询中使用了它，它必须实现`BatchTableSource`源接口。`TableSource`还可以实现这两个接口，并用于流查询和批处理查询。

`StreamTableSource`和`BatchTableSource`扩展了基本接口`TableSource`，它定义了以下方法：

{% tabs %}
{% tab title="Java" %}
```java
TableSource<T> {

  public TableSchema getTableSchema();

  public TypeInformation<T> getReturnType();

  public String explainSource();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
TableSource[T] {

  def getTableSchema: TableSchema

  def getReturnType: TypeInformation[T]

  def explainSource: String

}
```
{% endtab %}
{% endtabs %}

* `getTableSchema()`：返回表的结构，即表的字段的名称和类型。字段类型使用Flink定义`TypeInformation`（请参阅[表API类型](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#data-types)和[SQL类型](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#data-types)）。
* `getReturnType()`：返回`DataStream`（`StreamTableSource`）或`DataSet`（`BatchTableSource`）的物理类型以及由此生成的记录`TableSource`。
* `explainSource()`：返回描述`TableSource`的字符串 。此方法是可选的，仅用于显示目的。

`TableSource`接口将逻辑表模式与返回的`DataStream`或`DataSet`的物理类型分隔开来。因此，表模式的所有字段\(`getTableSchema()`\)必须映射到具有相应的物理返回类型\(`getReturnType()`\)的字段。默认情况下，此映射是基于字段名完成的。例如，定义具有两个字段`[name: String, size: Integer]`的表模式的表源需要至少具有两个字段的类型信息，分别称为类型`String`和`Integer`的`name`和`size`。这可以是一个`PojoTypeInfo`或一个`RowTypeInfo`，其中有两个字段`name`和`size`，并具有匹配的类型。

但是，有些类型，例如`Tuple`或`CaseClass`类型，确实支持自定义字段名。如果`TableSource`返回具有固定字段名的类型的`DataStream`或`DataSet`，它可以实现`DefinedFieldMapping`接口，将字段名从表模式映射到物理返回类型的字段名。

### 定义BatchTableSource

`BatchTableSource`接口扩展了`TableSource`接口，并定义一个额外的方法:

{% tabs %}
{% tab title="Java" %}
```java
BatchTableSource<T> implements TableSource<T> {

  public DataSet<T> getDataSet(ExecutionEnvironment execEnv);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
BatchTableSource[T] extends TableSource[T] {

  def getDataSet(execEnv: ExecutionEnvironment): DataSet[T]
}
```
{% endtab %}
{% endtabs %}

`getDataSet(execEnv)`:返回包含表数据的`DataSet`。`DataSet`的类型必须与`TableSource.getReturnType()`方法定义的返回类型相同。`DataSet`可以通过使用`DataSet` API的常规[数据源](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html#data-sources)的创建。通常，BatchTableSource源是通过包装InputFormat或[批处理连接器实现](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/batch/connectors.html)的。

### 定义StreamTableSource

`StreamTableSource`接口扩展了`TableSource`接口，并定义一个额外的方法：

{% tabs %}
{% tab title="Java" %}
```java
StreamTableSource<T> implements TableSource<T> {

  public DataStream<T> getDataStream(StreamExecutionEnvironment execEnv);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
StreamTableSource[T] extends TableSource[T] {

  def getDataStream(execEnv: StreamExecutionEnvironment): DataStream[T]
}
```
{% endtab %}
{% endtabs %}

* `getDataStream(execEnv)`：返回带有表数据的DataStream。DataStream的类型必须与`TableSource.getReturnType()`方法定义的返回类型相同。可以使用DataStream API的常规[数据源](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/datastream_api.html#data-sources)创建DataStream。通常，`StreamTableSource`源是通过包装`SourceFunction`或[流连接器实现](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/connectors/)的。

### 使用时间属性定义TableSource

  
流[表API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#group-windows)和[SQL](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/sql.html#group-windows)查询的基于时间的操作（例如窗口化聚合或连接）需要明确指定的[时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)。

`TableSource`在其表模式中将time属性定义为`Types.SQL_TIMESTAMP`类型的字段。 与模式中的所有常规字段相比，时间属性不能与表源的返回类型中的物理字段匹配。 相反，`TableSource`通过实现某个接口来定义时间属性。

#### **定义处理时间属性**

[处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html#processing-time)通常用于流式查询。处理时间属性返回访问它的操作员的当前挂钟时间。`TableSource`通过实现`DefinedProctimeAttribute`接口来定义处理时间属性。界面如下：

{% tabs %}
{% tab title="Java" %}
```java
DefinedProctimeAttribute {

  public String getProctimeAttribute();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
DefinedProctimeAttribute {

  def getProctimeAttribute: String
}
```
{% endtab %}
{% endtabs %}

* `getProctimeAttribute()`：返回处理时间属性的名称。必须在表模式中定义指定的属性类型`Types.SQL_TIMESTAMP`，并且可以在基于时间的操作中使用。 DefinedProctimeAttribute表源可以通过返回null来定义无处理时间属性。

{% hint style="danger" %}
**注意：**`StreamTableSource`和`BatchTableSource`都可以实现`DefinedProctimeAttribute`并定义处理时间属性。 在`BatchTableSource`的情况下，在表扫描期间使用当前时间戳初始化处理时间字段。
{% endhint %}

#### **定义RowTime属性**

[Rowtime属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html#event-time)是TIMESTAMP类型的属性，在流和批处理查询中以统一的方式处理。

可以通过指定将`SQL_TIMESTAMP`类型的表模式字段声明为`RowTime`属性:

* 字段的名称，
* `TimestampExtractor`计算属性的实际值（通常来自一个或多个其他字段）
* `WatermarkStrategy`指定如何为`RowTime`属性生成水印。

`TableSource`通过实现`DefinedRowtimeAttributes`接口来定义`RowTime`属性。界面如下：

{% tabs %}
{% tab title="Java" %}
```java
DefinedRowtimeAttribute {

  public List<RowtimeAttributeDescriptor> getRowtimeAttributeDescriptors();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
DefinedRowtimeAttributes {

  def getRowtimeAttributeDescriptors: util.List[RowtimeAttributeDescriptor]
}
```
{% endtab %}
{% endtabs %}

* `getRowtimeAttributeDescriptors()`：返回`RowtimeAttributeDescriptor`的列表。 `RowtimeAttributeDescriptor`描述具有以下属性的`RowTime`属性：
  * `attributeName`: 表模式中的rowtime属性的名称。 必须使用Types.SQL\_TIMESTAMP类型定义该字段。
  * `timestampExtractor`: 时间戳提取器从具有返回类型的记录中提取时间戳。 例如，它可以将Long字段转换为时间戳或解析字符串编码的时间戳。 Flink附带了一组针对常见用例的内置`TimestampExtractor`实现。 还可以提供自定义实现。
  * `watermarkStrategy`：水印策略定义了如何为`RowTime`属性生成水印。 Flink附带了一组针对常见用例的内置`WatermarkStrategy`实现。 还可以提供自定义实现。

{% hint style="danger" %}
**注意：**虽然`getRowtimeAttributeDescriptors()`方法返回一个描述符列表，但目前只支持单个`Rowtime`属性。我们计划在将来取消这个限制，并支持具有多个`Rowtime`属性的表。
{% endhint %}

{% hint style="danger" %}
**注意：**`StreamTableSource`和`BatchTableSource`都可以实现`DefinedRowtimeAttributes`并定义`rowtime`属性。 在任何一种情况下，都使用`TimestampExtractor`提取`rowtime`字段。 因此，实现StreamTableSource和`BatchTableSource`并定义`rowtime`属性的`TableSource`为流式和批量查询提供完全相同的数据。
{% endhint %}

**提供时间戳提取器**

Flink提供`TimestampExtractor`常见用例的实现。

`TimestampExtractor`目前提供以下实现：

* `ExistingField(fieldName)`:从现有的LONG，SQL\_TIMESTAMP或时间戳格式的STRING字段中提取rowtime属性的值。 这种字符串的一个例子是'2018-05-28 12：34：56.000'。
* `StreamRecordTimestamp()`：从DataStream StreamRecord的时间戳中提取rowtime属性的值。 请注意，此TimestampExtractor不适用于批处理表源。

自定义时间戳提取器\(`TimestampExtractor`\)可以通过实现相应的接口来定义

**提供水印策略**

Flink提供`WatermarkStrategy`常见用例的实现。

`WatermarkStrategy`目前提供以下实现：

* `AscendingTimestamps`：用于提升时间戳的水印策略。带有时间戳的记录如果顺序错误，将被认为是延迟的。
* `BoundedOutOfOrderTimestamps(delay)`：时间戳的水印策略，其最多在指定的延迟之外是无序的。
* `PreserveWatermarks()`：一种表示水印应该从基础数据流中保留的策略。

自定义水印策略\(`WatermarkStrategy`\)可以通过实现相应的接口来定义。

### 使用Projection Push-Down定义TableSource

`TableSource`通过实现`ProjectableTableSource`接口支持投影下推。该接口定义了一个方法：

{% tabs %}
{% tab title="Java" %}
```java
ProjectableTableSource<T> {

  public TableSource<T> projectFields(int[] fields);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
ProjectableTableSource[T] {

  def projectFields(fields: Array[Int]): TableSource[T]
}
```
{% endtab %}
{% endtabs %}

* `projectFields(fields)`：返回具有调整后的物理返回类型的`TableSource`的副本。 `fields`参数提供`TableSource`必须提供的字段的索引。 索引与物理返回类型的`TypeInformation`有关，而与逻辑表模式无关。 复制的`TableSource`必须调整其返回类型和返回的`DataStream`或`DataSet`。 不得更改复制的`TableSource`的`TableSchema`，即它必须与原始`TableSource`相同。 如果`TableSource`实现`DefinedFieldMapping`接口，则必须将字段映射调整为新的返回类型。

`ProjectableTableSource`添加了对项目平面字段的支持。 如果`TableSource`定义了一个具有嵌套模式的表，它可以实现`NestedFieldsProjectableTableSource`以将投影扩展到嵌套字段。 `NestedFieldsProjectableTableSource`定义如下：

{% tabs %}
{% tab title="Java" %}
```java
NestedFieldsProjectableTableSource<T> {

  public TableSource<T> projectNestedFields(int[] fields, String[][] nestedFields);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
NestedFieldsProjectableTableSource[T] {

  def projectNestedFields(fields: Array[Int], nestedFields: Array[Array[String]]): TableSource[T]
}
```
{% endtab %}
{% endtabs %}

* `projectNestedField(fields, nestedFields)`：返回具有调整后的物理返回类型的`TableSource`的副本。 可以删除或重新排序物理返回类型的字段，但不得更改其类型。 此方法的契约与`ProjectableTableSource.projectFields()`方法的契约基本相同。 此外，nestedFields参数包含字段列表中的每个字段索引，该列表指向查询访问的所有嵌套字段的路径。 不需要在`TableSource`生成的记录中读取，解析和设置所有其他嵌套字段。 重要信息不得更改投影字段的类型，但可以将未使用的字段设置为空或默认值。

### 使用过滤器下推定义TableSource

`FilterableTableSource`接口将对过滤器下推的支持添加到`TableSource`。 扩展此接口的TableSource能够过滤记录，以便返回的DataStream或DataSet返回更少的记录。

接口定义如下：

{% tabs %}
{% tab title="Java" %}
```java
FilterableTableSource<T> {

  public TableSource<T> applyPredicate(List<Expression> predicates);

  public boolean isFilterPushedDown();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
FilterableTableSource[T] {

  def applyPredicate(predicates: java.util.List[Expression]): TableSource[T]

  def isFilterPushedDown: Boolean
}
```
{% endtab %}
{% endtabs %}

* `applyPredicate(predicates)`：返回带有添加谓词的`TableSource`的副本。 谓词参数是“提供”给TableSource的可变谓词的可变列表。 `TableSource`接受通过从列表中删除谓词来评估谓词。 列表中剩余的谓词将由后续过滤器运算符进行评估。
* `isFilterPushedDown()`：如果之前调用了`applyPredicate()`方法，则返回true。 因此，对于从`applyPredicate()`调用返回的所有`TableSource`实例，`isFilterPushedDown()`必须返回true。

## 定义TableSink

`TableSink`指定如何将表发送到外部系统或位置。 该接口是通用的，因此它可以支持不同的存储位置和格式。 批处理表和流表有不同的表接收器。

通用接口定义如下所示：

{% tabs %}
{% tab title="Java" %}
```java
TableSink<T> {

  public TypeInformation<T> getOutputType();

  public String[] getFieldNames();

  public TypeInformation[] getFieldTypes();

  public TableSink<T> configure(String[] fieldNames, TypeInformation[] fieldTypes);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
TableSink[T] {

  def getOutputType: TypeInformation<T>

  def getFieldNames: Array[String]

  def getFieldTypes: Array[TypeInformation]

  def configure(fieldNames: Array[String], fieldTypes: Array[TypeInformation]): TableSink[T]
}
```
{% endtab %}
{% endtabs %}

调用`TableSink#configure`方法以传递Table的模式（字段名称和类型）以发送到`TableSink`。 该方法必须返回`TableSink`的新实例，该实例配置为发出提供的表模式。

### BatchTableSink

定义拓展`TableSink`以发出批处理表。

接口定义如下：

{% tabs %}
{% tab title="Java" %}
```java
BatchTableSink<T> implements TableSink<T> {

  public void emitDataSet(DataSet<T> dataSet);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
BatchTableSink[T] extends TableSink[T] {

  def emitDataSet(dataSet: DataSet[T]): Unit
}
```
{% endtab %}
{% endtabs %}

### AppendStreamTableSink

定义拓展`TableSink`以发出仅包含插入更改的流表。

接口定义如下：

{% tabs %}
{% tab title="Java" %}
```java
AppendStreamTableSink<T> implements TableSink<T> {

  public void emitDataStream(DataStream<T> dataStream);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
AppendStreamTableSink[T] extends TableSink[T] {

  def emitDataStream(dataStream: DataStream<T>): Unit
}
```
{% endtab %}
{% endtabs %}

如果还通过更新或删除更改来修改表，则将抛出TableException。

### RetractStreamTableSink

定义拓展TableSink以发出包含插入，更新和删除更改的流表。

接口定义如下：

{% tabs %}
{% tab title="Java" %}
```java
RetractStreamTableSink<T> implements TableSink<Tuple2<Boolean, T>> {

  public TypeInformation<T> getRecordType();

  public void emitDataStream(DataStream<Tuple2<Boolean, T>> dataStream);
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
RetractStreamTableSink[T] extends TableSink[Tuple2[Boolean, T]] {

  def getRecordType: TypeInformation[T]

  def emitDataStream(dataStream: DataStream[Tuple2[Boolean, T]]): Unit
}
```
{% endtab %}
{% endtabs %}

该表将被转换为累积和撤销消息流，这些消息被编码为Java Tuple2。 第一个字段是一个布尔标志，用于指示消息类型（true表示插入，false表示删除）。 第二个字段保存所请求类型T的记录。

### UpsertStreamTableSink

定义拓展`TableSink`以发出包含插入，更新和删除更改的流表。

接口定义如下：

{% tabs %}
{% tab title="Java" %}
```java
UpsertStreamTableSink<T> implements TableSink<Tuple2<Boolean, T>> {

  public void setKeyFields(String[] keys);

  public void setIsAppendOnly(boolean isAppendOnly);

  public TypeInformation<T> getRecordType();

  public void emitDataStream(DataStream<Tuple2<Boolean, T>> dataStream);
}

```
{% endtab %}

{% tab title="Scala" %}
```scala
UpsertStreamTableSink[T] extends TableSink[Tuple2[Boolean, T]] {

  def setKeyFields(keys: Array[String]): Unit

  def setIsAppendOnly(isAppendOnly: Boolean): Unit

  def getRecordType: TypeInformation[T]

  def emitDataStream(dataStream: DataStream[Tuple2[Boolean, T]]): Unit
}
```
{% endtab %}
{% endtabs %}

该表必须具有唯一的键字段（原子或复合）或仅附加。 如果表没有唯一键且不是仅附加，则抛出`TableException`。 表的唯一键由`UpsertStreamTableSink＃setKeyFields()`方法配置。

该表将被转换为`upsert`和`delete`消息流，这些消息被编码为Java Tuple2。 第一个字段是一个布尔标志，用于指示消息类型。 第二个字段保存所请求类型T的记录。

具有`true`标记的布尔字段的消息是已配置密钥的`upsert`消息。 带有`false`标志的消息是已配置密钥的`delete`消息。 如果表是仅附加的，则所有消息都将具有`true`标志，并且必须解释为插入。

## 定义TableFactory

TableFactory允许从基于字符串的属性创建不同的表相关实例。 调用所有可用工厂以匹配给定的属性集和相应的工厂类。

工厂利用Java的[服务提供商接口（SPI）](https://docs.oracle.com/javase/tutorial/sound/SPI-intro.html)进行发现。 这意味着每个依赖项和JAR文件都应包含`META_INF/services`资源目录中的文件`org.apache.flink.table.factories.TableFactory`，该文件列出了它提供的所有可用表工厂。

每个表工厂都需要实现以下接口：

{% tabs %}
{% tab title="Java" %}
```java
package org.apache.flink.table.factories;

interface TableFactory {

  Map<String, String> requiredContext();

  List<String> supportedProperties();
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
package org.apache.flink.table.factories

trait TableFactory {

  def requiredContext(): util.Map[String, String]

  def supportedProperties(): util.List[String]
}
```
{% endtab %}
{% endtabs %}

* `requiredContext()`：指定已为此工厂实现的上下文。 如果满足指定的属性和值集，框架保证仅匹配此工厂。 典型属性可能是`connector.type`，`format.type`或`update-mode`。 诸如`connector.property-version`和`format.property-version`之类的属性键保留用于将来的向后兼容性情况。
* `supportedProperties`：此工厂可以处理的属性键列表。 此方法将用于验证。 如果传递了该工厂无法处理的属性，则会抛出异常。 该列表不得包含上下文指定的键。

为了创建一个特定的实例，工厂类可以实现org.apache.flink.table.factories中提供的一个或多个接口:

* `BatchTableSourceFactory`：创建批处理表源。
* `BatchTableSinkFactory`：创建批处理表接收器。
* `StreamTableSoureFactory`：创建流表源。
* `StreamTableSinkFactory`：创建流表接收器。
* `DeserializationSchemaFactory`：创建反序列化结构格式。
* `SerializationSchemaFactory`：创建序列化结构格式。

工厂的发现有多个阶段:

* 发现所有可用的工厂。
* 按工厂类别过滤（例如`StreamTableSourceFactory`）。
* 通过匹配上下文过滤。
* 按支持的属性过滤。
* 验证确切的一个工厂匹配，否则抛出一个`AmbiguousTableFactoryException`或`NoMatchingTableFactoryException`。

以下示例显示如何为参数化提供附加的connector.debug属性标志的自定义流式源。

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.table.sources.StreamTableSource;
import org.apache.flink.types.Row;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

class MySystemTableSourceFactory implements StreamTableSourceFactory<Row> {

  @Override
  public Map<String, String> requiredContext() {
    Map<String, String> context = new HashMap<>();
    context.put("update-mode", "append");
    context.put("connector.type", "my-system");
    return context;
  }

  @Override
  public List<String> supportedProperties() {
    List<String> list = new ArrayList<>();
    list.add("connector.debug");
    return list;
  }

  @Override
  public StreamTableSource<Row> createStreamTableSource(Map<String, String> properties) {
    boolean isDebug = Boolean.valueOf(properties.get("connector.debug"));

    # additional validation of the passed properties can also happen here

    return new MySystemAppendTableSource(isDebug);
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import java.util
import org.apache.flink.table.sources.StreamTableSource
import org.apache.flink.types.Row

class MySystemTableSourceFactory extends StreamTableSourceFactory[Row] {

  override def requiredContext(): util.Map[String, String] = {
    val context = new util.HashMap[String, String]()
    context.put("update-mode", "append")
    context.put("connector.type", "my-system")
    context
  }

  override def supportedProperties(): util.List[String] = {
    val properties = new util.ArrayList[String]()
    properties.add("connector.debug")
    properties
  }

  override def createStreamTableSource(properties: util.Map[String, String]): StreamTableSource[Row] = {
    val isDebug = java.lang.Boolean.valueOf(properties.get("connector.debug"))

    # additional validation of the passed properties can also happen here

    new MySystemAppendTableSource(isDebug)
  }
}
```
{% endtab %}
{% endtabs %}

### 在SQL客户端中使用TableFactory

在SQL客户端环境文件中，先前展示的工厂可以声明为：

```yaml
tables:
 - name: MySystemTable
   type: source
   update-mode: append
   connector:
     type: my-system
     debug: true
```

YAML文件被转换为扁平字符串属性，并使用描述与外部系统的连接的属性调用表工厂：

```text
update-mode=append
connector.type=my-system
connector.debug=true
```

{% hint style="danger" %}
**注意：**属性，例如`tables.#.name`或是`tables.#.type`SQL客户端细节，不会传递给任何工厂。type属性根据执行环境决定是否需要发现`BatchTableSourceFactory` / `StreamTableSourceFactory`（用于源），`BatchTableSinkFactory` / `StreamTableSinkFactory`（用于接收器）或两者（两者）。
{% endhint %}

### 在Table＆SQL API中使用TableFactory

对于具有解释性Scaladoc / Javadoc的类型安全的编程方法，Table＆SQL API在`org.apache.flink.table.descriptors`中提供了转换为基于字符串的属性的描述符。 请参阅源，接收器和格式的内置描述符作为参考。

在我们的示例中，`MySystem`的连接器可以扩展`ConnectorDescriptor`，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
import org.apache.flink.table.descriptors.ConnectorDescriptor;
import java.util.HashMap;
import java.util.Map;

/**
  * Connector to MySystem with debug mode.
  */
public class MySystemConnector extends ConnectorDescriptor {

  public final boolean isDebug;

  public MySystemConnector(boolean isDebug) {
    super("my-system", 1, false);
    this.isDebug = isDebug;
  }

  @Override
  protected Map<String, String> toConnectorProperties() {
    Map<String, String> properties = new HashMap<>();
    properties.put("connector.debug", Boolean.toString(isDebug));
    return properties;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.table.descriptors.ConnectorDescriptor
import java.util.HashMap
import java.util.Map

/**
  * Connector to MySystem with debug mode.
  */
class MySystemConnector(isDebug: Boolean) extends ConnectorDescriptor("my-system", 1, false) {
  
  override protected def toConnectorProperties(): Map[String, String] = {
    val properties = new HashMap[String, String]
    properties.put("connector.debug", isDebug.toString)
    properties
  }
}
```
{% endtab %}
{% endtabs %}

然后可以在API中使用描述符，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
StreamTableEnvironment tableEnv = // ...

tableEnv
  .connect(new MySystemConnector(true))
  .inAppendMode()
  .registerTableSource("MySystemTable");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val tableEnv: StreamTableEnvironment = // ...

tableEnv
  .connect(new MySystemConnector(isDebug = true))
  .inAppendMode()
  .registerTableSource("MySystemTable")
```
{% endtab %}
{% endtabs %}

