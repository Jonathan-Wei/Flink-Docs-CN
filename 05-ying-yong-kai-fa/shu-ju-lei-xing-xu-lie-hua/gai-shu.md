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



## Flink的TypeInformation类

### **POJO类型的规则**

### **创建TypeInformation或TypeSerializer**

{% tabs %}
{% tab title="Java" %}
```java
TypeInformation<String> info = TypeInformation.of(String.class);
```

```java
TypeInformation<Tuple2<String, Double>> info = TypeInformation.of(new TypeHint<Tuple2<String, Double>>(){});
```
{% endtab %}

{% tab title="Scala" %}
```scala
// important: this import is needed to access the 'createTypeInformation' macro function
import org.apache.flink.streaming.api.scala._

val stringInfo: TypeInformation[String] = createTypeInformation[String]

val tupleInfo: TypeInformation[(String, Double)] = createTypeInformation[(String, Double)]
```
{% endtab %}
{% endtabs %}

## 在Scala API中输入信息



```scala
import org.apache.flink.api.scala._
```

```scala
def selectFirst[T](input: DataSet[(T, _)]) : DataSet[T] = {
  input.map { v => v._1 }
}

val data : DataSet[(String, Long) = ...

val result = selectFirst(data)
```

```java
def selectFirst[T : TypeInformation](input: DataSet[(T, _)]) : DataSet[T] = {
  input.map { v => v._1 }
}
```

## 在Java API中输入信息



```java
public class AppendOne<T> implements MapFunction<T, Tuple2<T, Long>> {

    public Tuple2<T, Long> map(T value) {
        return new Tuple2<T, Long>(value, 1L);
    }
}
```

```java
DataSet<SomeType> result = dataSet
    .map(new MyGenericNonInferrableFunction<Long, SomeType>())
        .returns(SomeType.class);
```

```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().enableForceAvro();
```

```java
final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
env.getConfig().enableForceKryo();
```

```java
env.getConfig().addDefaultKryoSerializer(Class<?> type, Class<? extends Serializer<?>> serializerClass)
```

## 禁用Kryo回退



```java
env.getConfig().disableGenericTypes();
```

## 使用工厂定义类型信息

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

