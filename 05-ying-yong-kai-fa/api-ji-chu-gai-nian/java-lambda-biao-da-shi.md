# Java Lambda表达式

Java 8引入了几种新的语言功能，旨在实现更快，更清晰的编码。凭借最重要的功能，即所谓的“Lambda表达式”，它打开了函数式编程的大门。Lambda表达式允许以直接的方式实现和传递函数，而无需声明其他（匿名）类。

{% hint style="danger" %}
Flink支持对Java API的所有运算符使用lambda表达式，但是，每当lambda表达式使用Java泛型时，您需要_显式_声明类型信息。
{% endhint %}

本文档介绍如何使用lambda表达式并描述当前的限制。有关Flink API的一般介绍，请参阅[编程指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html)

## 示例和限制

下面的示例演示如何实现一个简单的内联`map()`函数，该函数使用lambda表达式对其输入进行平方。函数的输入`i`和输出参数的类型`map()`不需要声明，因为它们是由Java编译器推断的。

```text
env.fromElements(1, 2, 3)
// returns the squared i
.map(i -> i*i)
.print();
```

Flink可以自动从方法签名OUT map\(IN value\)的实现中提取结果类型信息，因为OUT不是泛型而是整数。

不幸的是，像flatMap\(\)这样带有签名void flatMap\(IN value, Collector&lt;OUT&gt; out\)的函数被Java编译器编译成void flatMap\(IN value, Collector out\)。这使得Flink无法自动推断输出类型的类型信息。

Flink很可能会抛出类似于以下内容的异常：

```text
org.apache.flink.api.common.functions.InvalidTypesException: The generic type parameters of 'Collector' are missing.
    In many cases lambda methods don't provide enough information for automatic type extraction when Java generics are involved.
    An easy workaround is to use an (anonymous) class instead that implements the 'org.apache.flink.api.common.functions.FlatMapFunction' interface.
    Otherwise the type has to be specified explicitly using type information.
```

在这种情况下，需要_显式指定_类型信息，否则输出将被视为`Object`导致无效序列化的类型。

```java
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.DataSet;
import org.apache.flink.util.Collector;

DataSet<Integer> input = env.fromElements(1, 2, 3);

// collector type must be declared
input.flatMap((Integer number, Collector<String> out) -> {
    StringBuilder builder = new StringBuilder();
    for(int i = 0; i < number; i++) {
        builder.append("a");
        out.collect(builder.toString());
    }
})
// provide type information explicitly
.returns(Types.STRING)
// prints "a", "a", "aa", "a", "aa", "aaa"
.print();
```

使用带有泛型返回类型的map\(\)函数时也会出现类似的问题。方法签名Tuple2 map\(整数值\)在下面的示例中被擦除变成Tuple2 map\(整数值\)。

```java
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.java.tuple.Tuple2;

env.fromElements(1, 2, 3)
    .map(i -> Tuple2.of(i, i))    // no information about fields of Tuple2
    .print();
```

一般来说，这些问题可以通过多种方式解决：

```java
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.api.java.tuple.Tuple2;

// use the explicit ".returns(...)"
env.fromElements(1, 2, 3)
    .map(i -> Tuple2.of(i, i))
    .returns(Types.TUPLE(Types.INT, Types.INT))
    .print();

// use a class instead
env.fromElements(1, 2, 3)
    .map(new MyTuple2Mapper())
    .print();

public static class MyTuple2Mapper extends MapFunction<Integer, Tuple2<Integer, Integer>> {
    @Override
    public Tuple2<Integer, Integer> map(Integer i) {
        return Tuple2.of(i, i);
    }
}

// use an anonymous class instead
env.fromElements(1, 2, 3)
    .map(new MapFunction<Integer, Tuple2<Integer, Integer>> {
        @Override
        public Tuple2<Integer, Integer> map(Integer i) {
            return Tuple2.of(i, i);
        }
    })
    .print();

// or in this example use a tuple subclass instead
env.fromElements(1, 2, 3)
    .map(i -> new DoubleTuple(i, i))
    .print();

public static class DoubleTuple extends Tuple2<Integer, Integer> {
    public DoubleTuple(int f0, int f1) {
        this.f0 = f0;
        this.f1 = f1;
    }
}
```

