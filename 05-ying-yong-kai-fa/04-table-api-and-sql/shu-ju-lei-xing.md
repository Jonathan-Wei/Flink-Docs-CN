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

#### **CHAR**

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

 可以使用`CHAR(n)`来声明类型，`n`是类型长度。`n`的值必须介于`1` 和`2,147,483,647`（包括两者之间）之间。如果未指定长度，`n`则等于`1`。

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.String` | X | X | _默认_ |
| `byte[]` | X | X | 假设使用UTF-8编码。 |

#### **VARCHAR / STRING**

可变长度字符串的数据类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
VARCHAR
VARCHAR(n)

STRING
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.VARCHAR(n)

DataTypes.STRING()
```
{% endtab %}
{% endtabs %}

可以使用`VARCHAR(n)`来声明类型， `n`是最大类型长度。`n`的值必须介于`1`和`2,147,483,647`（包括两者之间）之间。如果未指定长度，`n`则等于`1`。

`STRING`等同于`VARCHAR(2147483647)`。

 **桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.String` | X | X | _默认_ |
| `byte[]` | X | X | 假设使用UTF-8编码。 |

### 二进制字符串

#### **BINARY**

固定长度的二进制字符串（=字节序列）的数据类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
BINARY
BINARY(n)
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.BINARY(n)
```
{% endtab %}
{% endtabs %}

 可以使用`BINARY(n)`来声明类型，`n`是字节数量。`n`的值必须介于`1`和`2,147,483,647`（包括两者之间）之间。如果未指定长度，`n`则等于`1`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `byte[]` | X | X | _默认_ |

**VARBINARY / BYTES**

可变长度二进制字符串（=字节序列）的数据类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
VARBINARY
VARBINARY(n)

BYTES
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```scala
DataTypes.VARBINARY(n)

DataTypes.BYTES()
```
{% endtab %}
{% endtabs %}

可以使用`VARBINARY(n)`来声明类型，`n`是最大字节数。`n`的值必须介于`1`和`2,147,483,647`（包括两者之间）之间。如果未指定长度，`n`则等于`1`。

`BYTES`等同于`VARBINARY(2147483647)`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `byte[]` | X | X | _默认_ |

### 精确数值

**DECIMAL**

具有固定精度和小数位数的十进制数字的数据类型。

可以使用`DECIMAL(p, s)`声明类型，其中`p`（_precision_）是数字中的位数，`s`（_scale_）是数字中小数点右边的位数。`p`的值必须介于`1`和`38`（包括两者之间）之间。`s` 的值必须介于`0`和`p`（包括两者之间）之间。`p`默认值是10，`s`默认值是 `0`。

`NUMERIC(p, s)`等同于`DEC(p, s)`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.math.BigDecimal` | X | X | _默认_ |

**TINYINT**

 1个字节有符号整数的数据类型，其值从`-128`to到`127`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
TINYINT
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.TINYINT()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Byte` | X | X | _默认_ |
| `byte` | X | （X） | 仅当类型不可为空时才输出。 |

**SMALLINT**

2个字节有符号整数的数据类型，其值从`-32,768`到`32,767`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
SMALLINT
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.SMALLINT()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Short` | X | X | _默认_ |
| `short` | X | （X） | 仅当类型不可为空时才输出。 |

**INT**

一个4字节有符号整数的数据类型，其值从`-2,147,483,648`to到`2,147,483,647`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
INT

INTEGER
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.INT()
```
{% endtab %}
{% endtabs %}

`INTEGER` 等于与`INT`

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Integer` | X | X | _默认_ |
| `int` | X | （X） | 仅当类型不可为空时才输出。 |

**BIGINT**

一个8字节有符号整数的数据类型，其值从`-9,223,372,036,854,775,808`to到 `9,223,372,036,854,775,807`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
BIGINT
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.BIGINT()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Long` | X | X | _默认_ |
| `long` | X | （X） | 仅当类型不可为空时才输出。 |

### 近似数值

**FLOAT**

4字节单精度浮点数的数据类型。

与SQL标准相比，该类型不带参数。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
FLOAT
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.FLOAT()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Float` | X | X | _默认_ |
| `float` | X | （X） | 仅当类型不可为空时才输出。 |

**DOUBLE**

8字节双精度浮点数的数据类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
DOUBLE

DOUBLE PRECISION
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```
DataTypes.DOUBLE()
```
{% endtab %}
{% endtabs %}

`DOUBLE PRECISION` 是此类型的同义词。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Double` | X | X | _默认_ |
| `double` | X | （X） | 仅当类型不可为空时才输出。 |

### 日期和时间

**DATE**

日期的数据类型，包含`year-month-day`范围从`0000-01-01` 到的值`9999-12-31`。

与SQL标准相比，范围从year开始`0000`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
DATE
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.DATE()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.LocalDate` | X | X | _默认_ |
| `java.sql.Date` | X | X |  |
| `java.lang.Integer` | X | X | 描述自纪元以来的天数。 |
| `int` | X | （X） | 描述自纪元以来的天数。 仅当类型不可为空时才输出。 |

**TIME**

一种不包括时区的时间数据类型，其精度高达纳秒`hour:minute:second[.fractional]`，范围从`00:00:00.000000000`到 `23:59:59.999999999`。

与SQL标准相比，不支持闰秒\(`23:59:60`和`23:59:61`\)，因为语义更接近`java.time.LocalTime`。不提供具有时区的时间。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
TIME
TIME(p)
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.TIME(p)
```
{% endtab %}
{% endtabs %}

 可以使用`TIME(p)`来声明类型，`p`（_precision_）是小数秒的位数。`p`的值必须介于`0`和`9`（包括两者之间）之间。如果未指定精度，`p`则等于`0`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.LocalTime` | X | X | _默认_ |
| `java.sql.Time` | X | X |  |
| `java.lang.Integer` | X | X | 描述一天中的毫秒数。 |
| `int` | X | （X） | 描述一天中的毫秒数。 仅当类型不可为空时才输出。 |
| `java.lang.Long` | X | X | 描述一天中的纳秒数。 |
| `long` | X | （X） | 描述一天中的纳秒数。 仅当类型不可为空时才输出。 |

**TIMESTAMP**

没有时区的时间戳的数据类型，包括`year-month-day hour:minute:second[.fractional]`，精度高达纳秒，值范围为`00:00 -01-01 00:00:00 0000000`到`9999-12-31 23:59:59.999999999`。

与SQL标准相比，不支持闰秒\(`23:59:60`和`23:59:61`\)，因为语义更接近`java.time.LocalDateTime`。

 不支持从`BIGINT`（与JVM `long`类型）之间的转换，因为这意味着时区。但是，此类型没有时区。对于更`java.time.Instant`类似的语义，请使用 `TIMESTAMP WITH LOCAL TIME ZONE`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
TIMESTAMP
TIMESTAMP(p)

TIMESTAMP WITHOUT TIME ZONE
TIMESTAMP(p) WITHOUT TIME ZONE
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.TIMESTAMP(p)
```
{% endtab %}
{% endtabs %}

可以使用`TIMESTAMP(p)`来声明类型，`p`（_precision_）是小数秒的位数。`p`的值必须介于`0`和`9`（包括两者之间）之间。如果未指定精度，`p`则等于`6`。

`TIMESTAMP(p) WITHOUT TIME ZONE` 是此类型的同义词。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.LocalDateTime` | X | X | _默认_ |
| `java.sql.Timestamp` | X | X |  |

**TIMESTAMP WITH TIME ZONE**

时间戳数据类型，包括`year-month-day hour:minute:second[.fractional] zone`，精度可达纳秒，值范围为`00:00 -01-01 00:00:00 0000000 +14:59`到`9999-12-31 23:59:59 -14:59`。

与SQL标准相比，不支持闰秒\(`23:59:60`和`23:59:61`\)，因为语义更接近 `java.time.OffsetDateTime`。

与`TIMESTAMP WITH LOCAL TIME ZONE`相比，时区偏移信息物理存储在每个数据中。它单独用于每次计算、可视化或与外部系统的通信。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
TIMESTAMP WITH TIME ZONE
TIMESTAMP(p) WITH TIME ZONE
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.TIMESTAMP_WITH_TIME_ZONE(p)
```
{% endtab %}
{% endtabs %}

 可以使用`TIMESTAMP(p) WITH TIME ZONE`来声明类型**，**`p`（_precision_）是小数秒的位数。`p`的值必须介于`0`和`9`（包括两者之间）之间。如果未指定精度，`p`则等于`6`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.OffsetDateTime` | X | X | _默认_ |
| `java.time.ZonedDateTime` | X |  | 忽略区域ID。 |

**TIMESTAMP WITH LOCAL TIME ZONE**

本地时区时间戳的数据类型，包括`year-month-day hour:minute:second[.fractional]`，精度高达纳秒，值范围为`00:00 -01-01 00:00:00 0000000 +14:59`到`9999-12-31 23:59:59 -14:59`。

不支持闰秒\(`23:59:60`和`23:59:61`\)，因为语义更接近`java.time.OffsetDateTime`。

与带时区的时间戳相比，时区偏移信息并没有物理地存储在每个数据中。相反，该类型`java.time.Instant`。UTC时区表生态系统边缘的即时语义。每个数据都在当前会话中配置的本地时区中进行解释，以便进行计算和可视化。

此类型允许根据配置的会话时区解释UTC时间戳，从而填补了时区自由和时区强制时间戳类型之间的空白。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
TIMESTAMP WITH LOCAL TIME ZONE
TIMESTAMP(p) WITH LOCAL TIME ZONE
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.TIMESTAMP_WITH_LOCAL_TIME_ZONE(p)
```
{% endtab %}
{% endtabs %}

可以使用  `TIMESTAMP(p) WITH LOCAL TIME ZONE`来声明类型**，**`p`（_precision_）是小数秒的位数。`p`的值必须介于`0`和`9`（包括两者之间）之间。如果未指定精度，`p`则等于`6`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.Instant` | X | X | _默认_ |
| `java.lang.Integer` | X | X | 描述自纪元以来的秒数。 |
| `int` | X | （X） | 描述自纪元以来的秒数。 仅当类型不可为空时才输出。 |
| `java.lang.Long` | X | X | 描述自纪元以来的毫秒数。 |
| `long` | X | （X） | 描述自纪元以来的毫秒数。 仅当类型不可为空时才输出。 |

**INTERVAL YEAR TO MONTH**

一组`year-month`间隔类型的数据类型。

类型必须参数化为以下任一分辨率:

* 年间隔
* 几年到几个月的间隔
* 或几个月的间隔。

年-月的间隔由`+years-months`组成，其值从`-9999-11`到`+9999-11`。

值表示对于所有类型的分辨率都是相同的。例如，月的间隔50总是以年到月的间隔格式表示\(具有默认的年份精度\):`+04-02`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
INTERVAL YEAR
INTERVAL YEAR(p)
INTERVAL YEAR(p) TO MONTH
INTERVAL MONTH
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.INTERVAL(DataTypes.YEAR())
DataTypes.INTERVAL(DataTypes.YEAR(p))
DataTypes.INTERVAL(DataTypes.YEAR(p), DataTypes.MONTH())
DataTypes.INTERVAL(DataTypes.MONTH())
```
{% endtab %}
{% endtabs %}

可以使用上面的组合声明类型，其中p是年份的位数\(年份精度\)。p的值必须在1和4之间\(包括1和4\)。如果没有指定年份精度，p等于2。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.Period` | X | X | 忽略`days`零件。_默认_ |
| `java.lang.Integer` | X | X | 描述月份数。 |
| `int` | X | （X） | 描述月份数。 仅当类型不可为空时才输出。 |

**INTERVAL DAY TO MONTH**

一组以天为间隔类型的数据类型。

类型必须参数化为以下分辨率之一，精度可达纳秒:

* 天间隔
* 几天到几小时的间隔
* 天到分钟的间隔
* 天到秒的间隔
* 小时间隔
* 小时到几分钟的间隔，
* 几小时到几秒钟的间隔
* 分钟的间隔
* 分钟到秒的间隔，
* 或秒间隔。

 一天中的时间间隔包含`+days hours:months:seconds.fractional`，范围从 `-999999 23:59:59.999999999`到`+999999 23:59:59.999999999`。所有类型的分辨率的值表示均相同。例如，70秒的间隔总是以天到秒的间隔格式表示\(具有默认精度\):`+00 00:01:10.000000`。

**声明**

{% tabs %}
{% tab title="SQL" %}
```sql
INTERVAL DAY
INTERVAL DAY(p1)
INTERVAL DAY(p1) TO HOUR
INTERVAL DAY(p1) TO MINUTE
INTERVAL DAY(p1) TO SECOND(p2)
INTERVAL HOUR
INTERVAL HOUR TO MINUTE
INTERVAL HOUR TO SECOND(p2)
INTERVAL MINUTE
INTERVAL MINUTE TO SECOND(p2)
INTERVAL SECOND
INTERVAL SECOND(p2)
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.INTERVAL(DataTypes.DAY())
DataTypes.INTERVAL(DataTypes.DAY(p1))
DataTypes.INTERVAL(DataTypes.DAY(p1), DataTypes.HOUR())
DataTypes.INTERVAL(DataTypes.DAY(p1), DataTypes.MINUTE())
DataTypes.INTERVAL(DataTypes.DAY(p1), DataTypes.SECOND(p2))
DataTypes.INTERVAL(DataTypes.HOUR())
DataTypes.INTERVAL(DataTypes.HOUR(), DataTypes.MINUTE())
DataTypes.INTERVAL(DataTypes.HOUR(), DataTypes.SECOND(p2))
DataTypes.INTERVAL(DataTypes.MINUTE())
DataTypes.INTERVAL(DataTypes.MINUTE(), DataTypes.SECOND(p2))
DataTypes.INTERVAL(DataTypes.SECOND())
DataTypes.INTERVAL(DataTypes.SECOND(p2))
```
{% endtab %}
{% endtabs %}

 可以使用上述组合声明类型，其中`p1`是天的位数（_日精度_）和`p2`是小数秒的位数（_分数精度_）。 `p1`的值必须介于`1`和`6`之间（包括两者之间）。`p2`的值必须介于`0` 和`9`之间（包括两者之间）。如果未`p1`指定，`2`默认情况下等于。如果未`p2`指定，`6`默认情况下等于。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.time.Duration` | X | X | _默认_ |
| `java.lang.Long` | X | X | 描述毫秒数。 |
| `long` | X | （X） | 描述毫秒数。 仅当类型不可为空时才输出。 |

### 结构化数据类型

**ARRAY**

具有相同子类型的元素数组的数据类型。

与SQL标准相比，无法指定数组的最大基数，但固定为`2,147,483,647`。另外，任何有效类型都支持作为子类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
ARRAY<t>
t ARRAY
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.ARRAY(t)
```
{% endtab %}
{% endtabs %}

可以使用`ARRAY<t>`来声明类型，`t`是所包含元素的数据类型。

`t ARRAY`是接近SQL标准的同义词。例如，`INT ARRAY`等效于`ARRAY<INT>`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| _Ť_`[]` | （X） | （X） | 取决于子类型。_默认_ |

**MAP**

关联数组的数据类型，该数组将键\(包括NULL\)映射到值\(包括NULL\)。MAP不能包含重复的键;每个键最多可以映射到一个值。

没有对元素类型的限制;确保唯一性是用户的责任。

MAP类型是SQL标准的扩展。

{% tabs %}
{% tab title="SQL" %}
```sql
MAP<kt, vt>
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.MAP(kt, vt)
```
{% endtab %}
{% endtabs %}

 可以使用`MAP<kt, vt>`来声明类型，其中`kt`是关键元素的数据类型，`vt`是值元素的数据类型。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.util.Map<kt, vt>` | X | X | _默认_ |
| `java.util.Map<kt, vt>的子类` | X |  |  |

**MULTISET**

多集\(=bag\)的数据类型。与set不同，它允许具有公共子类型的每个元素有多个实例。每个惟一值\(包括NULL\)都映射到某个多重性。

没有对元素类型的限制;确保唯一性是用户的责任。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
MULTISET<t>
t MULTISET
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.MULTISET(t)
```
{% endtab %}
{% endtabs %}

可以使用`MULTISET<t>`来声明类型，其中 `t`是所包含元素的数据类型。

`t MULTISET`是接近SQL标准的同义词。例如，`INT MULTISET`等效于`MULTISET<INT>`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.util.Map<t, java.lang.Integer>` | X | X | 将每个值分配给整数倍数。_默认_ |
| `java.util.Map<kt, java.lang.Integer>的子类` | X |  | 将每个值分配给整数倍数。 |

**ROW**

字段序列的数据类型。

字段由字段名、字段类型和可选描述组成。表的行中最特定的类型是行类型。在本例中，行中的每一列对应于具有与列相同序号位置的行类型的字段。

与SQL标准相比，可选字段描述简化了复杂结构的处理。

行类型类似于其他非标准框架中已知的 `STRUCT` 类型。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
ROW<n0 t0, n1 t1, ...>
ROW<n0 t0 'd0', n1 t1 'd1', ...>

ROW(n0 t0, n1 t1, ...>
ROW(n0 t0 'd0', n1 t1 'd1', ...)
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```java
DataTypes.ROW(DataTypes.FIELD(n0, t0), DataTypes.FIELD(n1, t1), ...)
DataTypes.ROW(DataTypes.FIELD(n0, t0, d0), DataTypes.FIELD(n1, t1, d1), ...)
```
{% endtab %}
{% endtabs %}

可以使用`ROW<n0 t0 'd0', n1 t1 'd1', ...>`来声明类型，其中`n`是字段的唯一名称，`t`是字段的逻辑类型，`d`是字段的描述。

`ROW(...)`是接近SQL标准的同义词。例如，`ROW(myField INT, myOtherField BOOLEAN)`等效于`ROW<myField INT, myOtherField BOOLEAN>`。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `org.apache.flink.types.Row` | X | X | _默认_ |

### 其他数据类型

**BOOLEAN**

一个布尔值的数据类型，它的三值逻辑可能为 `TRUE`，`FALSE`和`UNKNOWN`。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
BOOLEAN
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.BOOLEAN()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Boolean` | X | X | _默认_ |
| `boolean` | X | （X） | 仅当类型不可为空时才输出。 |

**NULL**

表示无类型空值的数据类型。

null类型是SQL标准的扩展。null类型除了null之外没有其他值，因此，可以将其转换为类似于JVM语义的任何可为空的类型。

这种类型有助于在使用NULL文字的API调用中表示未知类型，也有助于桥接到定义这种类型的格式\(如JSON或Avro\)。

这种类型在实践中不是很有用，这里只是为了完整性而提到它。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
NULL
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.NULL()
```
{% endtab %}
{% endtabs %}

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| `java.lang.Object` | X | X | _默认_ |
| 任意类 |  | （X） | 任何非原始类型。 |

**RAW**

任意序列化类型的数据类型。这种类型是表生态系统中的一个黑盒子，只在边缘进行反序列化。

RAW类型是SQL标准的扩展。

声明

{% tabs %}
{% tab title="SQL" %}
```sql
RAW('class', 'snapshot')
```
{% endtab %}

{% tab title="JAVA/Scala" %}
```text
DataTypes.RAW(class, serializer)

DataTypes.RAW(typeInfo)
```
{% endtab %}
{% endtabs %}

可以使用`RAW('class', 'snapshot')`来声明类型，`class`是原始类，而`snapshot`是Base64编码的序列化类型 `TypeSerializerSnapshot` 。通常，类型字符串不是直接声明的，而是在持久化类型时生成的。

在API中，`RAW`可以通过直接提供`Class`+ `TypeSerializer`或通过传递`TypeInformation`并让框架提取`Class`+ `TypeSerializer`来声明类型。

**桥接到JVM类型**

| Java类型 | 输入项 | 输出量 | 备注 |
| :--- | :--- | :--- | :--- |
| _类_ | X | X | 原始类或子类（用于输入）或超类（用于输出）。_默认_ |
| `byte[]` |  | X |  |

