# 事件处理\(CEP\)

## Getting Started

如果想使用CEP，请[设置Flink程序](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking_with_flink.html)并将FlinkCEP依赖项添加到`pom.xml`项目中。

{% tabs %}
{% tab title="Java" %}
```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-cep_2.11</artifactId>
  <version>1.7.0</version>
</dependency>
```
{% endtab %}

{% tab title="Scala" %}
```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-cep-scala_2.11</artifactId>
  <version>1.7.0</version>
</dependency>
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
FlinkCEP不包含在二进制发布包中。[点此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking.html)了解如何与集群执行相关联。
{% endhint %}

现在，您可以使用Pattern API开始编写第一个CEP程序。

{% hint style="danger" %}
要应用模式匹配的DataStream中的Events必须实现适当的equals\(\)和hashCode\(\)方法，因为FlinkCEP使用它们来比较和匹配事件。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Event> input = ...

Pattern<Event, ?> pattern = Pattern.<Event>begin("start").where(
        new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getId() == 42;
            }
        }
    ).next("middle").subtype(SubEvent.class).where(
        new SimpleCondition<SubEvent>() {
            @Override
            public boolean filter(SubEvent subEvent) {
                return subEvent.getVolume() >= 10.0;
            }
        }
    ).followedBy("end").where(
         new SimpleCondition<Event>() {
            @Override
            public boolean filter(Event event) {
                return event.getName().equals("end");
            }
         }
    );

PatternStream<Event> patternStream = CEP.pattern(input, pattern);

DataStream<Alert> result = patternStream.select(
    new PatternSelectFunction<Event, Alert>() {
        @Override
        public Alert select(Map<String, List<Event>> pattern) throws Exception {
            return createAlertFrom(pattern);
        }
    }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
val input: DataStream[Event] = ...

val pattern = Pattern.begin[Event]("start").where(_.getId == 42)
  .next("middle").subtype(classOf[SubEvent]).where(_.getVolume >= 10.0)
  .followedBy("end").where(_.getName == "end")

val patternStream = CEP.pattern(input, pattern)

val result: DataStream[Alert] = patternStream.select(createAlert(_))
```
{% endtab %}
{% endtabs %}

## Pattern API

Pattern API允许你定义希望从输入流中提取的复杂模式序列。

每个复杂的Pattern序列由多个简单的Pattern组成，即寻找具有相同属性的单个事件的Pattern。从现在起，我们将调用这些简单Pattern，以及我们在流中搜索的最终复杂模式序列，即**模式序列**。您可以将模式序列视为此类模式的图形，其中根据用户指定的条件\(例如event.getName\(\).equals\(“end”\)\)从一个模式转换到下一个模式。匹配是一系列输入事件，它们通过一系列有效的模式转换访问复杂模式图的所有模式。

{% hint style="danger" %}
每个Pattern必须具有唯一的名称，稍后您可以使用该名称来标识匹配的事件。
{% endhint %}

{% hint style="danger" %}
Pattern名称**不能**包含该字符`":"`。
{% endhint %}

在本节的其余部分，我们将首先介绍如何定义[个体模式](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#individual-patterns)，然后如何将各个模式组合到[复杂模式中](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#combining-patterns)。

### 个体模式

#### 量词

#### 条件

### 组合模式

### 模式组

## 检测模式

## 处理EventTime延迟

## 例子

## 从较旧的Flink版本迁移\(1.3之前版本\)

