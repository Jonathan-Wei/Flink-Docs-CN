# 概述

Apache Flink以独特的方式处理数据类型和序列化，包含自己的类型描述符，泛型类型提取和类型序列化框架。本文档描述了它们背后的概念和基本原理。

## Flink中的类型处理

Flink尝试推断有关在分布式计算期间交换和存储的数据类型的大量信息。把它想象成一个推断表格模式的数据库。在大多数情况下，Flink自己无缝地推断所有必要的信息。拥有类型信息允许Flink做一些很酷的事情：

* 使用POJO类型并通过引用字段名称（如`dataSet.keyBy("username")`）来进行分组/关联/聚合。类型信息允许Flink提前检查（用于拼写错误和类型兼容性），而不是稍后在运行时失败。
* Flink对数据类型的了解越多，序列化和数据布局方案就越好。这对于Flink中的内存使用范例非常重要（尽可能地处理堆内部/外部的序列化数据并使序列化非常便宜）。
* 最后，它还使大多数情况下的用户免于担心序列化框架和必须注册类型。

一般来说，在需要有关数据类型的信息_飞行前阶段_ -也就是，当程序的调用`DataStream` 和`DataSet`制成，任何调用之前`execute()`，`print()`，`count()`，或`collect()`。

## 最常见的问题

用户需要与Flink的数据类型处理进行交互的最常见问题是：

* **注册子类型：**如果函数签名仅描述超类型，但实际上它们在执行期间使用了这些类型的子类型，那么使Flink了解这些子类型

  则可能会大大提高性能。为此，请在`StreamExecutionEnvironment`或`ExecutionEnvironment`上为每个子类型调用`.registerType（clazz）`。

* **注册自定义序列化程序：** Flink会回退到[Kryo](https://github.com/EsotericSoftware/kryo)，因为它本身无法透明地处理这些类型。并非所有类型都由Kryo（以及Flink）无缝处理。例如，默认情况下，许多Google Guava集合类型无法正常工作。解决方案是为引起问题的类型注册额外的序列化器。调用`StreamExecutionEnvironment`或`ExecutionEnvironment`上的`.getConfig().addDefaultKryoSerializer(clazz, serializer)`。许多库中都提供了额外的Kryo序列化器。有关使用[自定义序列化](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/custom_serializers.html)程序的详细信息，请参阅[自定义序列](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/custom_serializers.html)化程
* **添加类型提示：**有时，尽管有所有技巧，但Flink无法推断泛型类型时，用户必须传递_类型提示_。这通常只在Java API中是必需的。该[类型提示章节](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/types_serialization.html#type-hints-in-the-java-api)描述了更多的细节。
* **手动创建`TypeInformation`：**这对于某些API调用可能是必要的，因为由于Java的泛型类型擦除，Flink无法推断数据类型。有关 详细信息，请参阅[创建TypeInformation或TypeSerializer](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/types_serialization.html#creating-a-typeinformation-or-typeserializer)。

## Flink的TypeInformation类

类[TypeInformation](https://github.com/apache/flink/blob/master//flink-core/src/main/java/org/apache/flink/api/common/typeinfo/TypeInformation.java) 是所有类型描述符的基类。它揭示了该类型的一些基本属性，并且可以生成序列化器，并且在专业化中，可以生成类型的比较器。（_请注意，Flink中的比较器不仅仅是定义一个顺序 - 它们基本上是处理键的实用程序_）

在内部，Flink在类型之间做出以下区分：

* 基本类型：所有的Java原语及其装箱形式，加`void`，`String`，`Date`，`BigDecimal`，和`BigInteger`。
* 基元数组和对象数组
* 复合类型
  * Flink Java Tuples（Flink Java API的一部分）：最多25个字段，不支持空字段
  * Scala Case_类_（包括Scala元组）：最多22个字段，不支持空字段
  * Row：具有任意数量字段的元组并支持空字段
  * POJO：遵循某种类似bean的模式的类
* 辅助类型（Option, Either, Lists, Maps, …）
* 通用类型：这些不是由Flink本身序列化的，而是由Kryo序列化的。

POJO特别令人感兴趣，因为其支持复杂类型的创建以及在键的定义中使用字段名称：`dataSet.join(another).where("name").equalTo("personName")`。其对运行时也是透明的，并且可以由Flink非常有效地处理。

### **POJO类型的规则**

如果满足以下条件，Flink会将数据类型识别为POJO类型（并允许“按名称”字段引用）：

* 该类是公共的和独立的（没有非静态内部类）
* 该类有一个公共的无参数构造函数
* 类（以及所有超类）中的所有非静态非瞬态字段都是公共的（和非最终的），或者具有公共getter和setter方法，该方法遵循getter和setter的Java bean命名约定。

请注意，当用户定义的数据类型无法识别为POJO类型时，必须将其作为GenericType处理并使用Kryo进行序列化。

### **创建TypeInformation或TypeSerializer**

要创建类型的类型信息对象，请使用语言特定的方法:

{% tabs %}
{% tab title="Java" %}
  
因为Java通常会擦除泛型类型信息，所以需要将类型传递给TypeInformation构造：

对于非泛型类型，可以传递类：

```java
TypeInformation<String> info = TypeInformation.of(String.class);
```

对于泛型类型，需要通过以下方式“捕获”泛型类型信息`TypeHint`：

```java
TypeInformation<Tuple2<String, Double>> info = TypeInformation.of(new TypeHint<Tuple2<String, Double>>(){});
```

在内部，这将创建TypeHint的匿名子类，该类捕获泛型信息，并将其保存到运行时。
{% endtab %}

{% tab title="Scala" %}
在Scala中，Flink使用在编译时运行的_宏_，并在仍然可用时捕获所有泛型类型信息。

```scala
// important: this import is needed to access the 'createTypeInformation' macro function
import org.apache.flink.streaming.api.scala._

val stringInfo: TypeInformation[String] = createTypeInformation[String]

val tupleInfo: TypeInformation[(String, Double)] = createTypeInformation[(String, Double)]
```

你仍然可以使用与Java相同的方法作为后备。
{% endtab %}
{% endtabs %}

要创建一个`TypeSerializer`，只需调用`TypeInformation`对象的`typeInfo.createSerializer(config)`即可

配置参数的类型为`ExecutionConfig`，它保存有关程序已注册的自定义序列化器的信息。尽可能地传递程序正确的`ExecutionConfig`。通常可以通过调用`getExecutionConfig()`从`DataStream`或`DataSet`获得它。在函数内部\(如`MapFunction`\)，您可以通过使函数成为一个[富函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/types_serialization.html)\([Rich Function](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/types_serialization.html) \)并调用`getRuntimeContext(). getexecutionconfig()`来获得它。

## 在Scala API中输入信息

Scala通过类型清单和类标记为运行时类型信息提供了非常详细的概念。一般来说，类型和方法可以访问它们的泛型参数的类型——因此，Scala程序不会像Java程序那样遭受类型擦除的痛苦。

此外，Scala允许通过Scala宏在Scala编译器中运行自定义代码 - 这意味着无论何时编译针对Flink的Scala API编写的Scala程序，都会执行一些Flink代码。

在编译过程中，我们使用宏查看所有用户函数的参数类型和返回类型——这是所有类型信息都完全可用的时间点。在宏中，我们为函数的返回类型\(或参数类型\)创建一个_`TypeInformation`_，并将其作为操作的一部分。

### **证据参数误差无隐含值**

在无法创建TypeInformation的情况下，程序无法编译，并显示_“无法找到TypeInformation类型的证据参数的隐式值”_的错误。

如果尚未导入生成TypeInformation的代码的常见原因。确保导入整个flink.api.scala包。

```scala
import org.apache.flink.api.scala._
```

另一个常见原因是通用方法，可以按照以下部分所述进行修复。

### **通用方法**

请考虑以下情况：

```scala
def selectFirst[T](input: DataSet[(T, _)]) : DataSet[T] = {
  input.map { v => v._1 }
}

val data : DataSet[(String, Long) = ...

val result = selectFirst(data)
```

对于这样的通用方法，函数参数和返回类型的数据类型对于每个调用可能不相同，并且在定义方法的站点处不知道。上面的代码将导致错误，即没有足够的隐式证据可用。

在这种情况下，必须在调用站点生成类型信息并将其传递给方法。Scala 为此提供_隐式参数_。

以下代码告诉Scala将_T_的类型信息带入函数。然后，将在调用方法的站点生成类型信息，而不是在定义方法的位置生成类型信息。

```scala
def selectFirst[T : TypeInformation](input: DataSet[(T, _)]) : DataSet[T] = {
  input.map { v => v._1 }
}
```

## 在Java API中输入信息

在一般情况下，Java删除泛型类型信息。Flink试图通过反射尽可能多地重构类型信息，使用Java保留的少量信息\(主要是函数签名和子类信息\)。该逻辑还包含一些简单的类型推断，用于函数的返回类型取决于其输入类型的情况:

```java
public class AppendOne<T> implements MapFunction<T, Tuple2<T, Long>> {

    public Tuple2<T, Long> map(T value) {
        return new Tuple2<T, Long>(value, 1L);
    }
}
```

在某些情况下，Flink无法重建所有通用类型信息。在这种情况下，用户必须通过_类型提示_帮助。

### **在Java API中输入提示**

在Flink无法重建已擦除的泛型类型信息的情况下，Java API提供所谓的_类型提示_。类型提示告诉系统函数生成的数据流或数据集的类型：

```java
DataSet<SomeType> result = dataSet
    .map(new MyGenericNonInferrableFunction<Long, SomeType>())
        .returns(SomeType.class);
```

returns语句指定生成的类型，在本例中是通过类。 提示支持通过类型定义**：**

* 类，非参数化类型（无泛型）
* `TypeHints`的形式`returns(new TypeHint<Tuple2<Integer, SomeType>>(){})`。`TypeHint`类可以捕获泛型类型信息并将其保存到运行时\(通过一个匿名子类\)。

### **Java 8 lambdas的类型提取**

Java 8 lambdas的类型提取与非lambda的工作方式不同，因为lambdas与扩展函数接口的实现类没有关联。

目前，Flink试图找出实现lambda的方法，并使用Java的通用签名来确定参数类型和返回类型。但是，并非所有编译器都为lambdas生成这些签名（至于Eclipse JDT编译器从4.5开始就可靠地编写本文档）。

### **POJO类型的序列化**

`PojoTypeInformation`正在为POJO中的所有字段创建序列化器。诸如`int`、`long`、`String`等标准类型由我们附带Flink的序列化器处理。对于所有其他类型，我们回到`Kryo`。

如果`Kryo`不能处理该类型，可以请求`PojoTypeInfo`使用`Avro`序列化`POJO`。要做到这一点，你必须调用

```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().enableForceAvro();
```

请注意，Flink会使用`Avro`序列化程序自动序列化`Avro`生成的`POJO`。

如果希望`Kryo`序列化程序处理**整个** `POJO`类型，请进行如下设置

```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().enableForceKryo();
```

如果Kryo无法序列化你的POJO，你可以使用自定义序列化程序添加到Kryo

```java
env.getConfig().addDefaultKryoSerializer(Class<?> type, Class<? extends Serializer<?>> serializerClass)
```

这些方法有不同的变体可供选择。

## 禁用Kryo后备

在某些情况下，程序可能希望明确避免使用Kryo作为泛型类型的后备。最常见的是希望确保通过Flink自己的序列化程序或通过用户定义的自定义序列化程序有效地序列化所有类型。

下面的设置将在遇到需要通过Kryo的数据类型时引发异常:

```java
env.getConfig().disableGenericTypes();
```

## 使用工厂定义类型信息

类型信息工厂允许将用户定义的类型信息插入到Flink类型系统中。您必须实现`org.apache.flink.api.common.typeinfo.TypeInfoFactory`来返回自定义类型信息。如果使用`@org.apache.flink.api.common.typeinfo.TypeInfo`注释了相应的类型，则在类型提取阶段调用工厂。

类型信息工厂可以在Java和Scala API中使用。

在类型层次结构中，向上遍历时将选择最近的工厂，但是，内置工厂具有最高的优先级。工厂的优先级也高于Flink的内置类型，因此您应该知道自己在做什么。

下面的示例展示了如何使用Java中的工厂来注释自定义类型`MyTuple`并为其提供自定义类型信息。

带注释的自定义类型：

```java
@TypeInfo(MyTupleTypeInfoFactory.class)
public class MyTuple<T0, T1> {
  public T0 myfield0;
  public T1 myfield1;
}
```

工厂提供自定义类型信息：

```java
public class MyTupleTypeInfoFactory extends TypeInfoFactory<MyTuple> {

  @Override
  public TypeInformation<MyTuple> createTypeInfo(Type t, Map<String, TypeInformation<?>> genericParameters) {
    return new MyTupleTypeInfo(genericParameters.get("T0"), genericParameters.get("T1"));
  }
}
```

该方法`createTypeInfo(Type, Map<String, TypeInformation<?>>)`为工厂的目标类型创建类型信息。参数提供有关类型本身的其他信息以及类型的泛型类型参数（如果可用）。

如果你的类型包含可能需要从Flink函数的输入类型派生的泛型参数，请确保还实现`org.apache.flink.api.common.typeinfo.TypeInformation#getGenericParameters`泛型参数到类型信息的双向映射。

