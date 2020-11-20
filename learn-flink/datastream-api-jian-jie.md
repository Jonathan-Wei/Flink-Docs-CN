---
description: 此培训的重点是广泛覆盖DataStream API，以便您能够开始编写流媒体应用程序。
---

# DataStream API简介

## 哪些可以做流媒体? 

Flink为Java和Scala提供的DataStream api，允许用于Flink自己的序列化器流化任何它们可以序列化的内容。

* 基础类型, i.e., String, Long, Integer, Boolean, Array
* 复合类型: Tuples, POJOs, 以及 Scala case classes

在Flink中也可以使用其他序列化器。特别是Avro，它得到了很好的支持。

### Java tuples 和 POJOs

如果满足以下条件，则Flink将数据类型识别为POJO类型（并允许“按名称”字段引用）：

* 该类是公共的和独立的（没有非静态内部类）
* 该类具有公共的无参数构造函数
* 类（和所有超类）中的所有非静态，非瞬态字段都是公共的（并且不是最终的），或者具有公共的getter和setter方法，这些方法遵循针对getter和setter的Java bean命名约定。

例：

```java
public class Person {
    public String name;  
    public Integer age;  
    public Person() {};  
    public Person(String name, Integer age) {  
        . . .
    };  
}  

Person person = new Person("Fred Flintstone", 35);
```

Flink的序列化器[支持POJO类型的模式演变](https://ci.apache.org/projects/flink/flink-docs-release-1.11/dev/stream/state/schema_evolution.html#pojo-types)。

### Scala tuples 和 case classes

正如你所期望的那样。

## 一个完整的例子

这个示例将关于人员的记录流作为输入，并过滤只包括成年人。

```java
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.api.common.functions.FilterFunction;

public class Example {

    public static void main(String[] args) throws Exception {
        final StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Person> flintstones = env.fromElements(
                new Person("Fred", 35),
                new Person("Wilma", 35),
                new Person("Pebbles", 2));

        DataStream<Person> adults = flintstones.filter(new FilterFunction<Person>() {
            @Override
            public boolean filter(Person person) throws Exception {
                return person.age >= 18;
            }
        });

        adults.print();

        env.execute();
    }

    public static class Person {
        public String name;
        public Integer age;
        public Person() {};

        public Person(String name, Integer age) {
            this.name = name;
            this.age = age;
        };

        public String toString() {
            return this.name.toString() + ": age " + this.age.toString();
        };
    }
}
```

### 流执行环境

每个Flink应用程序都需要一个执行环境，即本例中的env。流应用程序需要使用`StreamExecutionEnvironment`。

在应用程序中进行的DataStream API调用将构建一个附加到StreamExecutionEnvironment的作业图（job graph）。当调用env.execute\(\)时，此图被打包并发送到JobManager，后者将作业并行化，并将作业的各个部分分发给任务管理器执行。作业的每个并行切片都将在一个任务槽中执行。

注意，如果不调用execute\(\)，应用程序将不会运行。

![](../.gitbook/assets/image%20%2846%29.png)

这个分布式运行时取决于您的应用程序是否可序列化。它还要求集群中的每个节点都可以使用所有依赖项。

### 基本流Source

上述的例子使用env.fromElements\(…\)构造了一个DataStream。这是在原型或测试中创建简单流的一种方便方法。StreamExecutionEnvironment上还有一个fromCollection\(Collection\)方法。所以，你可以这样做:

```text
List<Person> people = new ArrayList<Person>();

people.add(new Person("Fred", 35));
people.add(new Person("Wilma", 35));
people.add(new Person("Pebbles", 2));

DataStream<Person> flintstones = env.fromCollection(people);
```

在原型设计时，另一种方便的方法是使用scoket

```text
DataStream<String> lines = env.socketTextStream("localhost", 9999)
```

或一个文件

```text
DataStream<String> lines = env.readTextFile("file:///path");
```

在真实的应用程序中，最常用的数据源是那些支持低延迟、高吞吐量并行读并支持倒带和重放\(高性能和容错的先决条件\)的数据源，比如Apache Kafka、Kinesis和各种文件系统。REST api和数据库也经常用于充实流。

### 基本流Sink

上面的示例使用adult .print\(\)将结果打印到任务管理器日志\(当在IDE中运行时，它将出现在IDE的控制台\)。这将在流的每个元素上调用toString\(\)。

输出是这样的

```text
1> Fred: age 35
2> Wilma: age 35
```

其中1&gt;和2&gt;表示哪个子任务\(即线程\)生成了输出。

在生产中，常用的接收器包括StreamingFileSink、各种数据库和几个发布-子系统。

### 调试

在生产中，应用程序将在远程集群或一组容器中运行。如果失败，它将远程失败。JobManager和TaskManager日志对于调试此类故障非常有用，但是Flink支持在IDE中进行本地调试要容易得多。你可以设置断点，检查局部变量，并逐步执行代码。也可以进入Flink的代码，如果您想了解Flink的工作原理，这可能是了解其内部的一种好方法。

## 上手

至此，您已经足够了解如何开始编码和运行一个简单的DataStream应用程序。克隆[flink-training](https://github.com/apache/flink-training/tree/release-1.11) repo，然后按照README中的说明进行第一个练习： [过滤流（Ride Cleansing）](https://github.com/apache/flink-training/tree/release-1.11/ride-cleansing)。

