# 数据类型

## 数据类型

由于历史原因，在Flink 1.9之前，Flink的Table和SQL API数据类型与Flink紧密耦合`TypeInformation`。`TypeInformation`在DataStream和DataSet API中使用，足以描述在分布式设置中序列化和反序列化基于JVM的对象所需的所有信息

 但是，`TypeInformation`并非旨在表示独立于实际JVM类的逻辑类型。过去，很难将SQL标准类型映射到此抽象。此外，某些类型不符合SQL，因此在引入时并没有大的眼光。

从Flink 1.9开始，Table＆SQL API将获得一个新的类型系统，作为API稳定性和标准合规性的长期解决方案。

重新设计类型系统是一项主要工作，它几乎涉及所有面向用户的界面。因此，它的介绍涵盖多个版本，社区旨在通过Flink 1.10完成这项工作。

由于同时为表程序添加了一个新的Planner\(请参阅FLINK-11439\)，因此并不支持计划表和数据类型的所有组合。此外，Planners可能不支持具有所需精度或参数的每种数据类型。

{% hint style="danger" %}
注意：在使用数据类型之前，请参阅规划器兼容性表和限制部分。
{% endhint %}

## 数据类型

 _数据类型_描述在表中的生态系统的值的逻辑类型。它可用于声明操作的输入和/或输出类型。

 Flink的数据类型类似于SQL标准的数据类型术语，但也包含关于值的可空性的信息，以便有效地处理标量表达式。

数据类型示例：

* `INT`
* `INT NOT NULL`
* `INTERVAL DAY TO SECOND(3)`
* `ROW<myField ARRAY<BOOLEAN>, myOtherField TIMESTAMP(3)>`

 可以在[下面](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/types.html#list-of-data-types)找到所有预定义数据类型的列表。

### Table API数据类型

 基于JVM的API的用户可以使用`org.apache.flink.table.types.DataType`Table API中的实例，也可以在定义连接器，目录或用户定义的函数时使用它们。

 一个`DataType`实例有两个职责：

* **逻辑类型的声明**，并不表示要进行传输或存储的具体物理表示形式，而是定义了基于JVM的语言和表生态系统之间的界限。
* _可选：_ **向计划者提供有关数据物理表示的提示，**这对于其他API的边缘很有用。

 对于基于JVM的语言，所有预定义的数据类型都在`org.apache.flink.table.api.DataTypes`中提供。

建议使用星号导入到表程序中以使用流畅的API：

{% tabs %}
{% tab title="Java" %}
```java
import static org.apache.flink.table.api.DataTypes.*;

DataType t = INTERVAL(DAY(), SECOND(3));
```
{% endtab %}

{% tab title="Scala" %}
```scala
import org.apache.flink.table.api.DataTypes._

val t: DataType = INTERVAL(DAY(), SECOND(3));
```
{% endtab %}
{% endtabs %}

**物理提示**

在基于SQL的类型系统结束且需要特定于编程的数据类型的表生态系统的边缘，需要物理提示。提示指示实现所需的数据格式。

例如，数据源可以表示`TIMESTAMP`使用一个`java.sql.Timestamp`类而不是使用`java.time.LocalDateTime`默认值来为逻辑生成值。有了这些信息，运行时就可以将产生的类转换为其内部数据格式。作为回报，数据接收器可以声明其从运行时使用的数据格式。

以下是一些如何声明桥接转换类的示例：

{% tabs %}
{% tab title="Java" %}
```java
// tell the runtime to not produce or consume java.time.LocalDateTime instances
// but java.sql.Timestamp
DataType t = DataTypes.TIMESTAMP(3).bridgedTo(java.sql.Timestamp.class);

// tell the runtime to not produce or consume boxed integer arrays
// but primitive int arrays
DataType t = DataTypes.ARRAY(DataTypes.INT().notNull()).bridgedTo(int[].class);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// tell the runtime to not produce or consume java.time.LocalDateTime instances
// but java.sql.Timestamp
val t: DataType = DataTypes.TIMESTAMP(3).bridgedTo(classOf[java.sql.Timestamp]);

// tell the runtime to not produce or consume boxed integer arrays
// but primitive int arrays
val t: DataType = DataTypes.ARRAY(DataTypes.INT().notNull()).bridgedTo(classOf[Array[Int]]);
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
请注意，通常只有在扩展API时才需要物理提示。预定义源/接收器/功能的用户无需定义此类提示。表程序（例如`field.cast(TIMESTAMP(3).bridgedTo(Timestamp.class))`）中的提示将被忽略。
{% endhint %}

## Planner兼容性

如引言中所述，对类型系统进行重做将跨越多个版本，每种数据类型的支持取决于所使用的Planner。本节旨在总结最重要的差异。

### 旧的Planner

Flink 1.9之前引入的Flink旧计划器主要支持类型信息。它仅对数据类型提供有限的支持。可以声明可以转换为类型信息的数据类型，以便旧计划者可以理解它们。

下表总结了数据类型和类型信息之间的区别。大多数简单类型以及行类型均保持不变。时间类型，数组类型和十进制类型需要特别注意。不允许使用其他提示。

 对于“ _类型信息”_列，该表省略了前缀`org.apache.flink.table.api.Types`。

 对于“ _数据类型表示形式”_列，该表省略了前缀`org.apache.flink.table.api.DataTypes`。

| 类型信息 | Java表达式字符串 | 数据类型表示 | 数据类型备注 |
| :--- | :--- | :--- | :--- |
| `STRING()` | `STRING` | `STRING()` |  |
| `BOOLEAN()` | `BOOLEAN` | `BOOLEAN()` |  |
| `BYTE()` | `BYTE` | `TINYINT()` |  |
| `SHORT()` | `SHORT` | `SMALLINT()` |  |
| `INT()` | `INT` | `INT()` |  |
| `LONG()` | `LONG` | `BIGINT()` |  |
| `FLOAT()` | `FLOAT` | `FLOAT()` |  |
| `DOUBLE()` | `DOUBLE` | `DOUBLE()` |  |
| `ROW(...)` | `ROW<...>` | `ROW(...)` |  |
| `BIG_DEC()` | `DECIMAL` | \[ `DECIMAL()`\] | 不是1：1的映射，因为会忽略精度和小数位数，而是使用Java的可变精度和小数位数。 |
| `SQL_DATE()` | `SQL_DATE` | `DATE()` `.bridgedTo(java.sql.Date.class)` |  |
| `SQL_TIME()` | `SQL_TIME` | `TIME(0)` `.bridgedTo(java.sql.Time.class)` |  |
| `SQL_TIMESTAMP()` | `SQL_TIMESTAMP` | `TIMESTAMP(3)` `.bridgedTo(java.sql.Timestamp.class)` |  |
| `INTERVAL_MONTHS()` | `INTERVAL_MONTHS` | `INTERVAL(MONTH())` `.bridgedTo(Integer.class)` |  |
| `INTERVAL_MILLIS()` | `INTERVAL_MILLIS` | `INTERVAL(DataTypes.SECOND(3))` `.bridgedTo(Long.class)` |  |
| `PRIMITIVE_ARRAY(...)` | `PRIMITIVE_ARRAY<...>` | `ARRAY(DATATYPE.notNull()` `.bridgedTo(PRIMITIVE.class))` | 适用于除之外的所有JVM基本类型`byte`。 |
| `PRIMITIVE_ARRAY(BYTE())` | `PRIMITIVE_ARRAY<BYTE>` | `BYTES()` |  |
| `OBJECT_ARRAY(...)` | `OBJECT_ARRAY<...>` | `ARRAY(` `DATATYPE.bridgedTo(OBJECT.class))` |  |
| `MULTISET(...)` |  | `MULTISET(...)` |  |
| `MAP(..., ...)` | `MAP<...,...>` | `MAP(...)` |  |
| 其他通用类型 |  | `RAW(...)` |  |

{% hint style="danger" %}
**注意：**如果新型系统有问题。用户可以随时根据org.apache.flink.table.api.Types的类型定义信息进行应变
{% endhint %}

### 新的Blink Planner

新的Blink Planner支持所有旧Planner类型。这尤其包括列出的Java表达式字符串和类型信息。

支持以下数据类型：

| 数据类型 | 数据类型备注 |
| :--- | :--- |
| `STRING` | `CHAR`并且`VARCHAR`尚不支持。 |
| `BOOLEAN` |  |
| `BYTES` | `BINARY`并且`VARBINARY`尚不支持。 |
| `DECIMAL` | 支持固定的精度和比例。 |
| `TINYINT` |  |
| `SMALLINT` |  |
| `INTEGER` |  |
| `BIGINT` |  |
| `FLOAT` |  |
| `DOUBLE` |  |
| `DATE` |  |
| `TIME` | 仅支持的精度`0`。 |
| `TIMESTAMP` | 仅支持的精度`3`。 |
| `TIMESTAMP WITH LOCAL TIME ZONE` | 仅支持的精度`3`。 |
| `INTERVAL` | 仅支持`MONTH`和的间隔`SECOND(3)`。 |
| `ARRAY` |  |
| `MULTISET` |  |
| `MAP` |  |
| `ROW` |  |
| `RAW` |  |

## 局限性

**Java表达式字符串**：Table API中的Java表达式字符串，例如`table.select("field.cast(STRING)")` 尚未更新为新类型的系统。使用在[旧计划器部分中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/types.html#old-planner)声明的字符串表示形式。

**连接器描述符和SQL客户端**：描述符字符串表示形式尚未更新为新类型的系统。使用在“ [连接到外部系统”部分中](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/connect.html#type-strings)声明的字符串表示形式

**用户定义的函数**：用户定义的函数尚不能声明数据类型。

## 数据类型列表

 本节列出了所有预定义的数据类型。对于基于JVM的Table API，这些类型在`org.apache.flink.table.api.DataTypes`中也可用。

### 字符串

**CHAR**

定长字符串的数据类型

声明

{% tabs %}
{% tab title="SQL" %}
```text
CHAR
CHAR(n)
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.CHAR(n)
```
{% endtab %}
{% endtabs %}

### 二进制字符串

### 精确数值

### 近似数值

### 日期和时间

### 结构化数据类型

### 其他数据类型

