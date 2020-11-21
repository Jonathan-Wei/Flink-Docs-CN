# 事件处理\(CEP\)



FlinkCEP是在Flink上层实现的复杂事件处理库。 它可以让你在无限事件流中检测出特定的事件模型，有机会掌握数据中重要的那部分。

本页讲述了Flink CEP中可用的API，我们首先讲述[模式API](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/libs/cep.html#%E6%A8%A1%E5%BC%8Fapi)，它可以让你指定想在数据流中检测的模式，然后讲述如何[检测匹配的事件序列并进行处理](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/libs/cep.html#%E6%A3%80%E6%B5%8B%E6%A8%A1%E5%BC%8F)。 再然后我们讲述Flink在按照事件时间[处理迟到事件](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/libs/cep.html#%E6%8C%89%E7%85%A7%E4%BA%8B%E4%BB%B6%E6%97%B6%E9%97%B4%E5%A4%84%E7%90%86%E8%BF%9F%E5%88%B0%E4%BA%8B%E4%BB%B6)时的假设， 以及如何从旧版本的Flink向1.3之后的版本[迁移作业](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/libs/cep.html#%E4%BB%8E%E6%97%A7%E7%89%88%E6%9C%AC%E8%BF%81%E7%A7%BB13%E4%B9%8B%E5%89%8D)。

## Getting Started

如果你想现在开始尝试，[创建一个Flink程序](https://ci.apache.org/projects/flink/flink-docs-release-1.11/zh/dev/project-configuration.html)， 添加FlinkCEP的依赖到项目的`pom.xml`文件中。

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

现在可以开始使用Pattern API写你的第一个CEP程序了。

{% hint style="danger" %}
注意：`DataStream`中的事件，如果你想在上面进行模式匹配的话，必须实现合适的 `equals()`和`hashCode()`方法， 因为FlinkCEP使用它们来比较和匹
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

模式可以是单例模式，也可以是循环模式。单例模式接受单个事件，而循环模式可以接受多个事件。在模式匹配符号中，模式“a b+ c?”d”\(或“a”，后面跟着一个或多个“b”，可选地后面跟着一个“c”，后面跟着一个“d”\)，a, c?， d为单例模式，b+为循环模式。默认情况下，模式是单例模式，您可以使用量词将其转换为循环模式。每个模式可以有一个或多个条件，基于这些条件接受事件。

#### 量词

在FlinkCEP中，可以使用以下方法指定循环模式:`pattern.oneOrMore()`对于期望一个或多个特定事件发生的模式（例如`b+`前面提到的）;   
`pattern.times(#ofTimes)`，对于期望特定类型事件的特定出现次数的模式，例如4 `a`;   
`pattern.times(#fromTimes, #toTimes)`，对于期望特定最小出现次数和给定类型事件的最大出现次数的模式，例如2-4s `a`。

可以使用`pattern.greed()`方法使循环模式变得贪婪，但是还不能使组模式变得贪婪。您可以使用pattern.optional\(\)方法使所有模式\(无论是否循环\)都是可选的。

对于名为start的模式，以下是有效的量词:

{% tabs %}
{% tab title="Java" %}
```java
 // expecting 4 occurrences
 start.times(4);

 // expecting 0 or 4 occurrences
 start.times(4).optional();

 // expecting 2, 3 or 4 occurrences
 start.times(2, 4);

 // expecting 2, 3 or 4 occurrences and repeating as many as possible
 start.times(2, 4).greedy();

 // expecting 0, 2, 3 or 4 occurrences
 start.times(2, 4).optional();

 // expecting 0, 2, 3 or 4 occurrences and repeating as many as possible
 start.times(2, 4).optional().greedy();

 // expecting 1 or more occurrences
 start.oneOrMore();

 // expecting 1 or more occurrences and repeating as many as possible
 start.oneOrMore().greedy();

 // expecting 0 or more occurrences
 start.oneOrMore().optional();

 // expecting 0 or more occurrences and repeating as many as possible
 start.oneOrMore().optional().greedy();

 // expecting 2 or more occurrences
 start.timesOrMore(2);

 // expecting 2 or more occurrences and repeating as many as possible
 start.timesOrMore(2).greedy();

 // expecting 0, 2 or more occurrences and repeating as many as possible
 start.timesOrMore(2).optional().greedy();
```
{% endtab %}

{% tab title="Scala" %}
```scala
 // expecting 4 occurrences
 start.times(4)

 // expecting 0 or 4 occurrences
 start.times(4).optional()

 // expecting 2, 3 or 4 occurrences
 start.times(2, 4)

 // expecting 2, 3 or 4 occurrences and repeating as many as possible
 start.times(2, 4).greedy()

 // expecting 0, 2, 3 or 4 occurrences
 start.times(2, 4).optional()

 // expecting 0, 2, 3 or 4 occurrences and repeating as many as possible
 start.times(2, 4).optional().greedy()

 // expecting 1 or more occurrences
 start.oneOrMore()

 // expecting 1 or more occurrences and repeating as many as possible
 start.oneOrMore().greedy()

 // expecting 0 or more occurrences
 start.oneOrMore().optional()

 // expecting 0 or more occurrences and repeating as many as possible
 start.oneOrMore().optional().greedy()

 // expecting 2 or more occurrences
 start.timesOrMore(2)

 // expecting 2 or more occurrences and repeating as many as possible
 start.timesOrMore(2).greedy()

 // expecting 0, 2 or more occurrences
 start.timesOrMore(2).optional()

 // expecting 0, 2 or more occurrences and repeating as many as possible
 start.timesOrMore(2).optional().greedy()
 
```
{% endtab %}
{% endtabs %}

#### 条件

对于每个模式，都可以指定传入事件必须满足的条件才能被“接受”到模式中，例如，它的值应该大于5，或者大于先前接受的事件的平均值。可以通过`pattern.where()`、`pattern.or()`或`pattern.until()`方法指定事件属性的条件。这些可以是`IterativeConditions`或`SimpleConditions`。

迭代条件\(**Iterative Conditions**\):这是最常见的条件类型。这就是如何指定一个条件，该条件基于先前接受的事件的属性或它们子集上的统计信息来接受后续事件。

下面是一个迭代条件的代码，如果一个名为“middle”的模式的名称以“foo”开头，并且如果该模式以前接受的事件的价格加上当前事件的价格的总和不超过5.0，则该迭代条件接受下一个事件。迭代条件可能很强大，特别是与循环模式结合使用时，例如`onOrmore()`。

{% tabs %}
{% tab title="Java" %}
```java
middle.oneOrMore()
    .subtype(SubEvent.class)
    .where(new IterativeCondition<SubEvent>() {
        @Override
        public boolean filter(SubEvent value, Context<SubEvent> ctx) throws Exception {
            if (!value.getName().startsWith("foo")) {
                return false;
            }
    
            double sum = value.getPrice();
            for (Event event : ctx.getEventsForPattern("middle")) {
                sum += event.getPrice();
            }
            return Double.compare(sum, 5.0) < 0;
        }
    });
```
{% endtab %}

{% tab title="Scala" %}
```scala
middle.oneOrMore()
    .subtype(classOf[SubEvent])
    .where(
        (value, ctx) => {
            lazy val sum = ctx.getEventsForPattern("middle").map(_.getPrice).sum
            value.getName.startsWith("foo") && sum + value.getPrice < 5.0
        }
    )
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
注意：调用`ctx.getEventsForPattern(...)`以查找给定潜在匹配的所有先前接受的事件。此操作的成本可能会有所不同，因此在实施你的条件时，请尽量减少其使用。
{% endhint %}

**简单条件：**此类条件扩展了上述`IterativeCondition`类，并_仅_根据事件本身的属性决定是否接受事件。

{% tabs %}
{% tab title="Java" %}
```java
start.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return value.getName().startsWith("foo");
    }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
start.where(event => event.getName.startsWith("foo"))
```
{% endtab %}
{% endtabs %}

最后，还可以`Event`通过`pattern.subtype(subClass)`方法将接受事件的类型限制为初始事件类型（此处）的子类型。

{% tabs %}
{% tab title="Java" %}
```java
start.subtype(SubEvent.class).where(new SimpleCondition<SubEvent>() {
    @Override
    public boolean filter(SubEvent value) {
        return ... // some condition
    }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
start.subtype(classOf[SubEvent]).where(subEvent => ... /* some condition */)
```
{% endtab %}
{% endtabs %}

**组合条件：**如上所示，可以将子类型条件与其他条件结合起来。这适用于所有的条件。最终结果将是各个条件的逻辑结果和结果。可以通过顺序调用`where()`任意组合条件。最终的结果将是符合逻辑的，各个条件**AND**的结果。要使用**OR**组合条件，可以使用`or()`方法，如下所示。

{% tabs %}
{% tab title="Java" %}
```java
pattern.where(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return ... // some condition
    }
}).or(new SimpleCondition<Event>() {
    @Override
    public boolean filter(Event value) {
        return ... // or condition
    }
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
pattern.where(event => ... /* some condition */).or(event => ... /* or condition */)
```
{% endtab %}
{% endtabs %}

**停止条件：**在循环模式（`oneOrMore()`和`oneOrMore().optional()`）的情况下，您还可以指定停止条件，例如，接受值大于5的事件，直到值的总和小于50。

为了更好地理解它，请看下面特定的示例。

* 模式`"(a+ until b)"`（一个或多个`"a"`直到`"b"`）
* 一系列传入事件 `"a1" "c" "a2" "b" "a3"`
* 该库将输出结果：`{a1 a2} {a1} {a2} {a3}`。

如你所见，由于停止条件，{a1 a2 a3}或{a2 a3}没有返回。

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>where(condition)</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E3A;&#x5F53;&#x524D;&#x6A21;&#x5F0F;&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x6761;&#x4EF6;&#x3002;&#x8981;&#x5339;&#x914D;&#x6A21;&#x5F0F;&#xFF0C;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x6EE1;&#x8DB3;&#x6761;&#x4EF6;&#x3002;&#x591A;&#x4E2A;&#x8FDE;&#x7EED;where()&#x5B50;&#x53E5;&#x5BFC;&#x81F4;&#x5B83;&#x4EEC;&#x7684;&#x6761;&#x4EF6;&#x88AB;&#x53E0;&#x52A0;:</p>
        <p></p>
        <p>pattern<b>.</b>where(<b>new</b> IterativeCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value, Context ctx) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b>  <b>...</b> // some condition</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>or(condition)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x4E0E;&#x73B0;&#x6709;&#x6761;&#x4EF6;&#x8FDB;&#x884C;&#x201C;&#x6216;&#x201D;&#x8FD0;&#x7B97;&#x7684;&#x65B0;&#x6761;&#x4EF6;&#x3002;&#x4EC5;&#x5F53;&#x4E8B;&#x4EF6;&#x81F3;&#x5C11;&#x901A;&#x8FC7;&#x5176;&#x4E2D;&#x4E00;&#x4E2A;&#x6761;&#x4EF6;&#x65F6;&#xFF0C;&#x624D;&#x80FD;&#x5339;&#x914D;&#x8BE5;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>pattern<b>.</b>where(<b>new</b> IterativeCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value, Context ctx) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b>  <b>...</b> // some condition</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>)<b>.</b>or(<b>new</b> IterativeCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value, Context ctx) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b>  <b>...</b> // alternative condition</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>until(condition)</b>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>&#x6307;&#x5B9A;&#x5FAA;&#x73AF;&#x6A21;&#x5F0F;&#x7684;&#x505C;&#x6B62;&#x6761;&#x4EF6;&#x3002;&#x610F;&#x5473;&#x7740;&#x5982;&#x679C;&#x5339;&#x914D;&#x7ED9;&#x5B9A;&#x6761;&#x4EF6;&#x7684;&#x4E8B;&#x4EF6;&#x53D1;&#x751F;&#xFF0C;&#x5219;&#x4E0D;&#x518D;&#x63A5;&#x53D7;&#x8BE5;&#x6A21;&#x5F0F;&#x4E2D;&#x7684;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x4EC5;&#x9002;&#x7528;&#x4E8E;<b> oneOrMore</b>()</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5B83;&#x5141;&#x8BB8;&#x5728;&#x57FA;&#x4E8E;&#x4E8B;&#x4EF6;&#x7684;&#x6761;&#x4EF6;&#x4E0B;&#x6E05;&#x9664;&#x76F8;&#x5E94;&#x6A21;&#x5F0F;&#x7684;&#x72B6;&#x6001;&#x3002;</p>
        <p>pattern<b>.</b>oneOrMore()<b>.</b>until(<b>new</b> IterativeCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value, Context ctx) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b>  <b>...</b> // alternative condition</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>subtype(subClass)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x5F53;&#x524D;&#x6A21;&#x5F0F;&#x7684;&#x5B50;&#x7C7B;&#x578B;&#x6761;&#x4EF6;&#x3002;&#x5982;&#x679C;&#x4E8B;&#x4EF6;&#x5C5E;&#x4E8E;&#x6B64;&#x5B50;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x4E8B;&#x4EF6;&#x53EA;&#x80FD;&#x5339;&#x914D;&#x8BE5;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>pattern<b>.</b>subtype(SubEvent<b>.</b>class);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>oneOrMore()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x81F3;&#x5C11;&#x9700;&#x8981;&#x51FA;&#x73B0;&#x4E00;&#x6B21;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>&#x6CE8;&#xFF1A;&#x8FD9;&#x662F;&#x5E94;&#x4E3A;&#x4F7F;&#x7528;until()&#x6216;within()&#x542F;&#x7528;&#x72B6;&#x6001;&#x7ED3;&#x7B97;</p>
        <p>pattern<b>.</b>oneOrMore();</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>timesOrMore(#times)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x81F3;&#x5C11;&#x9700;&#x8981;<b>#times</b>&#x51FA;&#x73B0;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>timesOrMore(2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>times(#ofTimes)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x9700;&#x8981;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;&#x786E;&#x5207;&#x51FA;&#x73B0;&#x6B21;&#x6570;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>times(2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>times(#fromTimes, #toTimes)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x671F;&#x671B;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;<b>#fromTimes</b> &#x548C;<b>#toTimes</b>&#x4E4B;&#x95F4;&#x51FA;&#x73B0;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>times(2, 4);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>optional()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x662F;&#x53EF;&#x9009;&#x7684;&#xFF0C;&#x5373;&#x6839;&#x672C;&#x4E0D;&#x4F1A;&#x53D1;&#x751F;&#x3002;&#x8FD9;&#x9002;&#x7528;&#x4E8E;&#x6240;&#x6709;&#x4E0A;&#x8FF0;&#x91CF;&#x8BCD;&#x3002;</p>
        <p>pattern<b>.</b>oneOrMore()<b>.</b>optional();</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>greedy()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x662F;&#x8D2A;&#x5A6A;&#x7684;&#xFF0C;&#x5373;&#x5B83;&#x5C06;&#x5C3D;&#x53EF;&#x80FD;&#x591A;&#x5730;&#x91CD;&#x590D;&#x3002;&#x8FD9;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x91CF;&#x8BCD;&#xFF0C;&#x76EE;&#x524D;&#x4E0D;&#x652F;&#x6301;&#x7EC4;&#x6A21;&#x5F0F;&#x3002;</p>
        <p>pattern<b>.</b>oneOrMore()<b>.</b>greedy();</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}


<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>where(condition)</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E3A;&#x5F53;&#x524D;&#x6A21;&#x5F0F;&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x6761;&#x4EF6;&#x3002;&#x8981;&#x5339;&#x914D;&#x6A21;&#x5F0F;&#xFF0C;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x6EE1;&#x8DB3;&#x6761;&#x4EF6;&#x3002;&#x591A;&#x4E2A;&#x8FDE;&#x7EED;where()&#x5B50;&#x53E5;&#x5BFC;&#x81F4;&#x5B83;&#x4EEC;&#x7684;&#x6761;&#x4EF6;&#x88AB;&#x53E0;&#x52A0;:</p>
        <p></p>
        <p>pattern.where(event =&gt; ... /<em> some condition </em>/)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>or(condition)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x4E0E;&#x73B0;&#x6709;&#x6761;&#x4EF6;&#x8FDB;&#x884C;&#x201C;&#x6216;&#x201D;&#x8FD0;&#x7B97;&#x7684;&#x65B0;&#x6761;&#x4EF6;&#x3002;&#x4EC5;&#x5F53;&#x4E8B;&#x4EF6;&#x81F3;&#x5C11;&#x901A;&#x8FC7;&#x5176;&#x4E2D;&#x4E00;&#x4E2A;&#x6761;&#x4EF6;&#x65F6;&#xFF0C;&#x624D;&#x80FD;&#x5339;&#x914D;&#x8BE5;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p></p>
        <p>pattern.where(event =&gt; ... /<em> some condition </em>/) .or(event =&gt;
          ... /<em> alternative condition </em>/)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>until(condition)</b>
      </td>
      <td style="text-align:left">
        <p></p>
        <p>&#x6307;&#x5B9A;&#x5FAA;&#x73AF;&#x6A21;&#x5F0F;&#x7684;&#x505C;&#x6B62;&#x6761;&#x4EF6;&#x3002;&#x610F;&#x5473;&#x7740;&#x5982;&#x679C;&#x5339;&#x914D;&#x7ED9;&#x5B9A;&#x6761;&#x4EF6;&#x7684;&#x4E8B;&#x4EF6;&#x53D1;&#x751F;&#xFF0C;&#x5219;&#x4E0D;&#x518D;&#x63A5;&#x53D7;&#x8BE5;&#x6A21;&#x5F0F;&#x4E2D;&#x7684;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x4EC5;&#x9002;&#x7528;&#x4E8E;<b> oneOrMore</b>()</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5B83;&#x5141;&#x8BB8;&#x5728;&#x57FA;&#x4E8E;&#x4E8B;&#x4EF6;&#x7684;&#x6761;&#x4EF6;&#x4E0B;&#x6E05;&#x9664;&#x76F8;&#x5E94;&#x6A21;&#x5F0F;&#x7684;&#x72B6;&#x6001;&#x3002;</p>
        <p>pattern.oneOrMore().until(event =&gt; ... /<em> some condition </em>/)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>subtype(subClass)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x5F53;&#x524D;&#x6A21;&#x5F0F;&#x7684;&#x5B50;&#x7C7B;&#x578B;&#x6761;&#x4EF6;&#x3002;&#x5982;&#x679C;&#x4E8B;&#x4EF6;&#x5C5E;&#x4E8E;&#x6B64;&#x5B50;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x4E8B;&#x4EF6;&#x53EA;&#x80FD;&#x5339;&#x914D;&#x8BE5;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>pattern.subtype(classOf[SubEvent])</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>oneOrMore()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x81F3;&#x5C11;&#x9700;&#x8981;&#x51FA;&#x73B0;&#x4E00;&#x6B21;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>&#x6CE8;&#xFF1A;&#x8FD9;&#x662F;&#x5E94;&#x4E3A;&#x4F7F;&#x7528;until()&#x6216;within()&#x542F;&#x7528;&#x72B6;&#x6001;&#x7ED3;&#x7B97;</p>
        <p>pattern<b>.</b>oneOrMore();</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>timesOrMore(#times)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x81F3;&#x5C11;&#x9700;&#x8981;<b>#times</b>&#x51FA;&#x73B0;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>timesOrMore(2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>times(#ofTimes)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x9700;&#x8981;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;&#x786E;&#x5207;&#x51FA;&#x73B0;&#x6B21;&#x6570;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>times(2);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>times(#fromTimes, #toTimes)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x671F;&#x671B;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;<b>#fromTimes</b> &#x548C;<b>#toTimes</b>&#x4E4B;&#x95F4;&#x51FA;&#x73B0;&#x3002;</p>
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4F7F;&#x7528;&#x5BBD;&#x677E;&#x7684;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5728;&#x540E;&#x7EED;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#xFF09;&#x3002;&#x6709;&#x5173;&#x5185;&#x90E8;&#x8FDE;&#x7EED;&#x6027;&#x7684;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/cep.html#consecutive_java">&#x8FDE;&#x7EED;</a>&#x3002;</p>
        <p>pattern<b>.</b>times(2, 4);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>optional()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x662F;&#x53EF;&#x9009;&#x7684;&#xFF0C;&#x5373;&#x6839;&#x672C;&#x4E0D;&#x4F1A;&#x53D1;&#x751F;&#x3002;&#x8FD9;&#x9002;&#x7528;&#x4E8E;&#x6240;&#x6709;&#x4E0A;&#x8FF0;&#x91CF;&#x8BCD;&#x3002;</p>
        <p>pattern<b>.</b>oneOrMore()<b>.</b>optional();</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>greedy()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6307;&#x5B9A;&#x6B64;&#x6A21;&#x5F0F;&#x662F;&#x8D2A;&#x5A6A;&#x7684;&#xFF0C;&#x5373;&#x5B83;&#x5C06;&#x5C3D;&#x53EF;&#x80FD;&#x591A;&#x5730;&#x91CD;&#x590D;&#x3002;&#x8FD9;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x91CF;&#x8BCD;&#xFF0C;&#x76EE;&#x524D;&#x4E0D;&#x652F;&#x6301;&#x7EC4;&#x6A21;&#x5F0F;&#x3002;</p>
        <p>pattern<b>.</b>oneOrMore()<b>.</b>greedy();</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 组合模式

现在你已经看到了个体模式的样子，是时候看看如何将它们组合成一个完整的模式序列。

模式序列必须以初始模式开始，如下所示：

{% tabs %}
{% tab title="Java" %}
```java
Pattern<Event, ?> start = Pattern.<Event>begin("start");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val start : Pattern[Event, _] = Pattern.begin("start")
```
{% endtab %}
{% endtabs %}

接下来，您可以通过指定它们之间所需的_连续条件_，为模式序列添加更多模式。FlinkCEP支持以下事件之间连续形式：

1. **严格连续性**：预期所有匹配事件一个接一个地出现，中间没有任何不匹配的事件。
2. **轻松连续性**：忽略匹配的事件之间出现的不匹配事件。
3. **非确定性**松弛连续性：进一步放宽连续性，允许忽略一些匹配事件的其他匹配。

要在连续模式之间应用它们，您可以使用：

1. `next()`，_严格来说_
2. `followedBy()`，_放松_
3. `followedByAny()`，对于_非确定性放松_连续性。

或

1. `notNext()`，如果不希望事件类型直接跟随另一个事件类型
2. `notFollowedBy()`，如果不希望事件类型在两个其他事件类型之间的任何位置。

{% hint style="danger" %}
注意：模式序列不能以notFollowedBy\(\)结尾。
{% endhint %}

{% hint style="danger" %}
注意：NOT模式之前不能有可选的模式。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
// strict contiguity
Pattern<Event, ?> strict = start.next("middle").where(...);

// relaxed contiguity
Pattern<Event, ?> relaxed = start.followedBy("middle").where(...);

// non-deterministic relaxed contiguity
Pattern<Event, ?> nonDetermin = start.followedByAny("middle").where(...);

// NOT pattern with strict contiguity
Pattern<Event, ?> strictNot = start.notNext("not").where(...);

// NOT pattern with relaxed contiguity
Pattern<Event, ?> relaxedNot = start.notFollowedBy("not").where(...);
```
{% endtab %}

{% tab title="Scala" %}
```scala
// strict contiguity
val strict: Pattern[Event, _] = start.next("middle").where(...)

// relaxed contiguity
val relaxed: Pattern[Event, _] = start.followedBy("middle").where(...)

// non-deterministic relaxed contiguity
val nonDetermin: Pattern[Event, _] = start.followedByAny("middle").where(...)

// NOT pattern with strict contiguity
val strictNot: Pattern[Event, _] = start.notNext("not").where(...)

// NOT pattern with relaxed contiguity
val relaxedNot: Pattern[Event, _] = start.notFollowedBy("not").where(...)
```
{% endtab %}
{% endtabs %}

宽松的连续性意味着仅匹配第一个匹配事件，而具有非确定性的松弛连续性，将为同一个开始发出多个匹配。 例如，给定事件序列“a”，“c”，“b1”，“b2”的模式“a b”将给出以下结果：

* “a”和“b”之间的严格连续性：{}（不匹配），“a”之后的“c”导致“a”被丢弃。
* “a”和“b”之间的宽松连续性：{a b1}，因为宽松的连续性被视为“跳过非匹配事件直到下一个匹配事件”。
* “a”和“b”之间的非确定性松弛连续性：{a b1}，{a b2}，因为这是最一般的形式。

{% tabs %}
{% tab title="Java" %}
```java
next.within(Time.seconds(10));
```
{% endtab %}

{% tab title="Scala" %}
```scala
next.within(Time.seconds(10))
```
{% endtab %}
{% endtabs %}

也可以为模式定义时间约束以使其有效。 例如，可以通过`pattern.within()`方法定义模式应在10秒内发生。 处理和事件时间都支持时间模式。

{% hint style="danger" %}
**注意：**模式序列只能有一个时间约束。 如果在不同的单独模式上定义了多个这样的约束，则应用最小的约束。
{% endhint %}

#### 循环模式中的连续性

可以在循环模式中应用与上一节中讨论的相同的连续条件。 连续性将应用于接受到这种模式的元素之间。 为了举例说明上述情况，模式序列`"a b+ c"` （“a”后跟一个或多个“b”的任何（非确定性松弛）序列，后跟“c”），输入`"a", "b1", "d1", "b2", "d2", "b3" "c"` 将产生以下结果：

* **严格连续性：**{a b3 c} - “b1”之后的“d1”导致“b1”被丢弃，“b2”因“d2”而发生同样的情况。
* **宽松的连续性：**{a b1 c}，{a b1 b2 c}，{a b1 b2 b3 c}，{a b2 c}，{a b2 b3 c}，{a b3 c} - “d”被忽略。
* **非确定性松弛连续性**：{a b1 c}，{a b1 b2 c}，{a b1 b3 c}，{a b1 b2 b3 c}，{a b2 c}，{a b2 b3 c}，{a b3 c} - 注意{a b1 b3 c}，这是放松“b”之间连续性的结果。

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>consecutive()</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>oneOrMore</b>()&#x548C;<b>times</b>()&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x5E76;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x5F3A;&#x52A0;&#x4E25;&#x683C;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF0C;&#x5373;&#x4EFB;&#x4F55;&#x4E0D;&#x5339;&#x914D;&#x7684;&#x5143;&#x7D20;&#x90FD;&#x4F1A;&#x4E2D;&#x65AD;&#x5339;&#x914D;&#xFF08;&#x5982;<b>next</b>()&#xFF09;&#x3002;</p>
        <p>&#x5982;&#x679C;&#x4E0D;&#x5E94;&#x7528;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x677E;&#x5F1B;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followBy</b>()&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#x3002; &#x50CF;&#x8FD9;&#x6837;&#x7684;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;c&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>)</p>
        <p><b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;a&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>)<b>.</b>oneOrMore()<b>.</b>consecutive()</p>
        <p><b>.</b>followedBy(&quot;end1&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;b&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
        <p>&#x5C06;&#x4E3A;&#x8F93;&#x5165;&#x5E8F;&#x5217;&#x751F;&#x6210;&#x4EE5;&#x4E0B;&#x5339;&#x914D;&#x9879;&#xFF1A;C
          D A1 A2 A3 D A4 B.</p>
        <p>&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#xFF1A;{C A1 B}&#xFF0C;{C A1 A2 B}&#xFF0C;{C
          A1 A2 A3 B}</p>
        <p>&#x6CA1;&#x6709;&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#xFF1A;{C A1 B}&#xFF0C;{C
          A1 A2 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>allowCombinations()</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>oneOrMore</b>()&#x548C;<b>times</b>()&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x5E76;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x5F3A;&#x52A0;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x7684;&#x8F7B;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followAyAny</b>()&#xFF09;&#x3002;</p>
        <p>&#x5982;&#x679C;&#x4E0D;&#x5E94;&#x7528;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x677E;&#x5F1B;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followBy</b>()&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#x3002; &#x50CF;&#x8FD9;&#x6837;&#x7684;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;c&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>)</p>
        <p><b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;a&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>)<b>.</b>oneOrMore()<b>.</b>allowCombinations()</p>
        <p><b>.</b>followedBy(&quot;end1&quot;)<b>.</b>where(<b>new</b> SimpleCondition<b>&lt;</b>Event<b>&gt;</b>() <b>{</b>
        </p>
        <p>@Override</p>
        <p> <b>public</b>  <b>boolean</b>  <b>filter</b>(Event value) <b>throws</b> Exception <b>{</b>
        </p>
        <p> <b>return</b> value<b>.</b>getName()<b>.</b>equals(&quot;b&quot;);</p>
        <p> <b>}</b>
        </p>
        <p><b>}</b>);</p>
        <p>&#x5C06;&#x4E3A;&#x8F93;&#x5165;&#x5E8F;&#x5217;&#x751F;&#x6210;&#x4EE5;&#x4E0B;&#x5339;&#x914D;&#x9879;&#xFF1A;C
          D A1 A2 A3 D A4 B.</p>
        <p>&#x542F;&#x7528;&#x7EC4;&#x5408;&#xFF1A;{C A1 B}&#xFF0C;{C A1 A2 B}&#xFF0C;{C
          A1 A3 B}&#xFF0C;{C A1 A4 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A4 B}&#xFF0C;{C
          A1 A3 A4 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
        <p>&#x672A;&#x542F;&#x7528;&#x7EC4;&#x5408;&#xFF1A;{C A1 B}&#xFF0C;{C A1
          A2 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>consecutive()</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>oneOrMore</b>()&#x548C;<b>times</b>()&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x5E76;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x5F3A;&#x52A0;&#x4E25;&#x683C;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF0C;&#x5373;&#x4EFB;&#x4F55;&#x4E0D;&#x5339;&#x914D;&#x7684;&#x5143;&#x7D20;&#x90FD;&#x4F1A;&#x4E2D;&#x65AD;&#x5339;&#x914D;&#xFF08;&#x5982;<b>next</b>()&#xFF09;&#x3002;</p>
        <p>&#x5982;&#x679C;&#x4E0D;&#x5E94;&#x7528;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x677E;&#x5F1B;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followBy</b>()&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#x3002; &#x50CF;&#x8FD9;&#x6837;&#x7684;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p><b>Pattern.</b>begin(&quot;start&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;c&quot;))</p>
        <p> <b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;a&quot;))</p>
        <p> <b>.</b>oneOrMore()<b>.</b>consecutive()</p>
        <p> <b>.</b>followedBy(&quot;end1&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;b&quot;))</p>
        <p>&#x5C06;&#x4E3A;&#x8F93;&#x5165;&#x5E8F;&#x5217;&#x751F;&#x6210;&#x4EE5;&#x4E0B;&#x5339;&#x914D;&#x9879;&#xFF1A;C
          D A1 A2 A3 D A4 B.</p>
        <p>&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#xFF1A;{C A1 B}&#xFF0C;{C A1 A2 B}&#xFF0C;{C
          A1 A2 A3 B}</p>
        <p>&#x6CA1;&#x6709;&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#xFF1A;{C A1 B}&#xFF0C;{C
          A1 A2 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>allowCombinations()</b>
      </td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>oneOrMore</b>()&#x548C;<b>times</b>()&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#xFF0C;&#x5E76;&#x5728;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x5F3A;&#x52A0;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x7684;&#x8F7B;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followAyAny</b>()&#xFF09;&#x3002;</p>
        <p>&#x5982;&#x679C;&#x4E0D;&#x5E94;&#x7528;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x677E;&#x5F1B;&#x7684;&#x8FDE;&#x7EED;&#x6027;&#xFF08;&#x5982;<b>followBy</b>()&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#x3002; &#x50CF;&#x8FD9;&#x6837;&#x7684;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p><b>Pattern.</b>begin(&quot;start&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;c&quot;))</p>
        <p> <b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;a&quot;))</p>
        <p> <b>.</b>oneOrMore()<b>.</b>allowCombinations()</p>
        <p> <b>.</b>followedBy(&quot;end1&quot;)<b>.</b>where(<b>_.</b>getName()<b>.</b>equals(&quot;b&quot;))</p>
        <p>&#x5C06;&#x4E3A;&#x8F93;&#x5165;&#x5E8F;&#x5217;&#x751F;&#x6210;&#x4EE5;&#x4E0B;&#x5339;&#x914D;&#x9879;&#xFF1A;C
          D A1 A2 A3 D A4 B.</p>
        <p>&#x542F;&#x7528;&#x7EC4;&#x5408;&#xFF1A;{C A1 B}&#xFF0C;{C A1 A2 B}&#xFF0C;{C
          A1 A3 B}&#xFF0C;{C A1 A4 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A4 B}&#xFF0C;{C
          A1 A3 A4 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
        <p>&#x672A;&#x542F;&#x7528;&#x7EC4;&#x5408;&#xFF1A;{C A1 B}&#xFF0C;{C A1
          A2 B}&#xFF0C;{C A1 A2 A3 B}&#xFF0C;{C A1 A2 A3 A4 B}</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 模式组

也可以将模式序列定义为`begin`，`followBy`，`followByAny`和`next`的条件。 模式序列将被逻辑地视为匹配条件，并且将返回`GroupPattern`并且可以应用`oneOrMore()`, `times(#ofTimes)`, `times(#fromTimes, #toTimes)`, `optional()`, `consecutive()`, `allowCombinations()`到`GroupPattern`。

{% tabs %}
{% tab title="Java" %}
```java
Pattern<Event, ?> start = Pattern.begin(
    Pattern.<Event>begin("start").where(...).followedBy("start_middle").where(...)
);

// strict contiguity
Pattern<Event, ?> strict = start.next(
    Pattern.<Event>begin("next_start").where(...).followedBy("next_middle").where(...)
).times(3);

// relaxed contiguity
Pattern<Event, ?> relaxed = start.followedBy(
    Pattern.<Event>begin("followedby_start").where(...).followedBy("followedby_middle").where(...)
).oneOrMore();

// non-deterministic relaxed contiguity
Pattern<Event, ?> nonDetermin = start.followedByAny(
    Pattern.<Event>begin("followedbyany_start").where(...).followedBy("followedbyany_middle").where(...)
).optional();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val start: Pattern[Event, _] = Pattern.begin(
    Pattern.begin[Event]("start").where(...).followedBy("start_middle").where(...)
)

// strict contiguity
val strict: Pattern[Event, _] = start.next(
    Pattern.begin[Event]("next_start").where(...).followedBy("next_middle").where(...)
).times(3)

// relaxed contiguity
val relaxed: Pattern[Event, _] = start.followedBy(
    Pattern.begin[Event]("followedby_start").where(...).followedBy("followedby_middle").where(...)
).oneOrMore()

// non-deterministic relaxed contiguity
val nonDetermin: Pattern[Event, _] = start.followedByAny(
    Pattern.begin[Event]("followedbyany_start").where(...).followedBy("followedbyany_middle").where(...)
).optional()
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>begin(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x8D77;&#x59CB;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> start <b>=</b> Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>begin(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x8D77;&#x59CB;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> start <b>=</b> Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>next(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x63A5;&#x66FF;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> next <b>=</b> start<b>.</b>next(&quot;middle&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>next(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x63A5;&#x66FF;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#xFF09;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> next <b>=</b> start<b>.</b>next(</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedBy(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#xFF09;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> followedBy <b>=</b> start<b>.</b>followedBy(&quot;middle&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedBy(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5728;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#xFF09;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> followedBy <b>=</b> start<b>.</b>followedBy(</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedByAny(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x5E76;&#x4E14;&#x5C06;&#x9488;&#x5BF9;&#x6BCF;&#x4E2A;&#x5907;&#x9009;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x653E;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x5448;&#x73B0;&#x66FF;&#x4EE3;&#x5339;&#x914D;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> followedByAny <b>=</b> start<b>.</b>followedByAny(&quot;middle&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedByAny(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5728;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x5E76;&#x4E14;&#x5C06;&#x9488;&#x5BF9;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;&#x6BCF;&#x4E2A;&#x66FF;&#x4EE3;&#x5E8F;&#x5217;&#xFF08;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x677E;&#x5F1B;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x5448;&#x73B0;&#x66FF;&#x4EE3;&#x5339;&#x914D;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> followedByAny <b>=</b> start<b>.</b>followedByAny(</p>
        <p>Pattern<b>.&lt;</b>Event<b>&gt;</b>begin(&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>notNext()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x7684;&#x8D1F;&#x9762;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#xFF08;&#x5426;&#x5B9A;&#xFF09;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x6210;&#x529F;&#x6267;&#x884C;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x624D;&#x80FD;&#x4E22;&#x5F03;&#x90E8;&#x5206;&#x5339;&#x914D;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> notNext <b>=</b> start<b>.</b>notNext(&quot;not&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>notFollowedBy()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x7684;&#x8D1F;&#x9762;&#x6A21;&#x5F0F;&#x3002;&#x5373;&#x4F7F;&#x5728;&#x5339;&#x914D;&#xFF08;&#x5426;&#x5B9A;&#xFF09;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x4E4B;&#x95F4;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x4E5F;&#x5C06;&#x4E22;&#x5F03;&#x90E8;&#x5206;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#xFF1A;</p>
        <p>Pattern<b>&lt;</b>Event, <b>?&gt;</b> notFollowedBy <b>=</b> start<b>.</b>notFollowedBy(&quot;not&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>within(time)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#x4E0E;&#x6A21;&#x5F0F;&#x5339;&#x914D;&#x7684;&#x6700;&#x5927;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;&#x5982;&#x679C;&#x672A;&#x5B8C;&#x6210;&#x7684;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#x8D85;&#x8FC7;&#x6B64;&#x65F6;&#x95F4;&#xFF0C;&#x5219;&#x5C06;&#x5176;&#x4E22;&#x5F03;&#xFF1A;</p>
        <p>pattern<b>.</b>within(Time<b>.</b>seconds(10));</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6A21;&#x5F0F;&#x64CD;&#x4F5C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>begin(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x8D77;&#x59CB;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p><b>val</b> start <b>=</b>  <b>Pattern.</b>begin[<b>Event</b>](&quot;start&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>begin(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x8D77;&#x59CB;&#x6A21;&#x5F0F;&#xFF1A;</p>
        <p><b>val</b> start <b>=</b>  <b>Pattern.</b>begin(</p>
        <p> <b>Pattern.</b>begin[<b>Event</b>](&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>next(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x63A5;&#x66FF;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#xFF1A;</p>
        <p><b>val next = start.next</b>(<b>&quot;middle&quot;</b>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>next(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x63A5;&#x66FF;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#xFF09;&#xFF1A;</p>
        <p><b>val</b> next <b>=</b> start<b>.</b>next(</p>
        <p> <b>Pattern.</b>begin[<b>Event</b>](&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedBy(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#xFF09;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF1A;</p>
        <p><b>val</b> followedBy <b>=</b> start<b>.</b>followedBy(&quot;middle&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedBy(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5728;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#xFF09;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF1A;</p>
        <p><b>val</b> followedBy <b>=</b> start<b>.</b>followedBy(</p>
        <p> <b>Pattern.</b>begin[<b>Event</b>](&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedByAny(#name)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x5E76;&#x4E14;&#x5C06;&#x9488;&#x5BF9;&#x6BCF;&#x4E2A;&#x5907;&#x9009;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x653E;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x5448;&#x73B0;&#x66FF;&#x4EE3;&#x5339;&#x914D;&#xFF1A;</p>
        <p><b>val</b> followedByAny <b>=</b> start<b>.</b>followedByAny(&quot;middle&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>followedByAny(#pattern_sequence)</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x6A21;&#x5F0F;&#x3002;&#x5728;&#x4E00;&#x7CFB;&#x5217;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x4E4B;&#x95F4;&#x53EF;&#x80FD;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x5E76;&#x4E14;&#x5C06;&#x9488;&#x5BF9;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x7684;&#x6BCF;&#x4E2A;&#x66FF;&#x4EE3;&#x5E8F;&#x5217;&#xFF08;&#x975E;&#x786E;&#x5B9A;&#x6027;&#x677E;&#x5F1B;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x5448;&#x73B0;&#x66FF;&#x4EE3;&#x5339;&#x914D;&#xFF1A;</p>
        <p><b>val</b> followedByAny <b>=</b> start<b>.</b>followedByAny(</p>
        <p> <b>Pattern.</b>begin[<b>Event</b>](&quot;start&quot;)<b>.</b>where(<b>...</b>)<b>.</b>followedBy(&quot;middle&quot;)<b>.</b>where(<b>...</b>)</p>
        <p>)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>notNext()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x7684;&#x8D1F;&#x9762;&#x6A21;&#x5F0F;&#x3002;&#x5339;&#x914D;&#xFF08;&#x5426;&#x5B9A;&#xFF09;&#x4E8B;&#x4EF6;&#x5FC5;&#x987B;&#x76F4;&#x63A5;&#x6210;&#x529F;&#x6267;&#x884C;&#x5148;&#x524D;&#x7684;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x4E25;&#x683C;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x624D;&#x80FD;&#x4E22;&#x5F03;&#x90E8;&#x5206;&#x5339;&#x914D;&#xFF1A;</p>
        <p><b>val</b> notNext <b>=</b> start<b>.</b>notNext(&quot;not&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>notFollowedBy()</b>
      </td>
      <td style="text-align:left">
        <p>&#x6DFB;&#x52A0;&#x65B0;&#x7684;&#x8D1F;&#x9762;&#x6A21;&#x5F0F;&#x3002;&#x5373;&#x4F7F;&#x5728;&#x5339;&#x914D;&#xFF08;&#x5426;&#x5B9A;&#xFF09;&#x4E8B;&#x4EF6;&#x548C;&#x5148;&#x524D;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#xFF08;&#x5BBD;&#x677E;&#x8FDE;&#x7EED;&#x6027;&#xFF09;&#x4E4B;&#x95F4;&#x53D1;&#x751F;&#x5176;&#x4ED6;&#x4E8B;&#x4EF6;&#xFF0C;&#x4E5F;&#x5C06;&#x4E22;&#x5F03;&#x90E8;&#x5206;&#x5339;&#x914D;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#xFF1A;</p>
        <p><b>val</b> notFollowedBy <b>=</b> start<b>.</b>notFollowedBy(&quot;not&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>within(time)</b>
      </td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#x4E0E;&#x6A21;&#x5F0F;&#x5339;&#x914D;&#x7684;&#x6700;&#x5927;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;&#x5982;&#x679C;&#x672A;&#x5B8C;&#x6210;&#x7684;&#x4E8B;&#x4EF6;&#x5E8F;&#x5217;&#x8D85;&#x8FC7;&#x6B64;&#x65F6;&#x95F4;&#xFF0C;&#x5219;&#x5C06;&#x5176;&#x4E22;&#x5F03;&#xFF1A;</p>
        <p>pattern<b>.</b>within(<b>Time.</b>seconds(10))</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

## 匹配后跳过策略

对于给定模式，可以将同一事件分配给多个成功匹配。 要控制将分配事件的匹配数，您需要指定名为`AfterMatchSkipStrategy`的跳过策略。 跳过策略有四种类型，如下所示：

* _**NO\_SKIP**_：将发出每个可能的匹配。
* _**SKIP\_PAST\_LAST\_EVENT**_：丢弃匹配开始后但结束前开始的每个部分匹配。
* _**SKIP\_TO\_FIRST**_：丢弃在匹配开始后但在 _PatternName_的第一个事件发生之前开始的每个部分匹配。
* _**SKIP\_TO\_LAST**_：丢弃在匹配开始后但在 _PatternName_的最后一个事件发生之前开始的每个部分匹配。

请注意，使用_`SKIP_TO_FIRST`_和_`SKIP_TO_LAST`_跳过策略时，还应指定有效的_`PatternName`_。

例如，对于给定模式`b+ c`和数据流`b1 b2 b3 c`，这四种跳过策略之间的差异如下：

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8DF3;&#x8FC7;&#x7B56;&#x7565;</th>
      <th style="text-align:left">&#x7ED3;&#x679C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>NO_SKIP</b>
      </td>
      <td style="text-align:left"><code>b1 b2 b3 c</code>
        <br /><code>b2 b3 c</code>
        <br /><code>b3 c</code>
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>b1 b2 b3 c</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x4E0D;&#x4F1A;&#x4E22;&#x5F03;&#x4EFB;</p>
        <p>&#x4F55;&#x7ED3;&#x679C;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_TO_NEXT</b>
      </td>
      <td style="text-align:left"><code>b1 b2 b3 c</code>
        <br /><code>b2 b3 c</code>
        <br /><code>b3 c</code>
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>b1 b2 b3 c</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x4E0D;&#x4F1A;&#x4E22;</p>
        <p>&#x5F03;&#x4EFB;&#x4F55;&#x7ED3;&#x679C;&#xFF0C;&#x56E0;&#x4E3A;</p>
        <p>&#x6CA1;&#x6709;&#x5176;&#x4ED6;&#x5339;&#x914D;&#x53EF;</p>
        <p>&#x4EE5;&#x4ECE;b1&#x5F00;&#x59CB;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_PAST_LAST_EVENT</b>
      </td>
      <td style="text-align:left"><code>b1 b2 b3 c</code>
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>b1 b2 b3 c</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x5C06;&#x4E22;&#x5F03;</p>
        <p>&#x6240;&#x6709;&#x5DF2;&#x5F00;&#x59CB;&#x7684;&#x90E8;&#x5206;</p>
        <p>&#x5339;&#x914D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_TO_FIRST</b> [ <code>b</code>]</td>
      <td style="text-align:left"><code>b1 b2 b3 c</code>
        <br /><code>b2 b3 c</code>
        <br /><code>b3 c</code>
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>b1 b2 b3 c</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x5C06;&#x5C1D;&#x8BD5;&#x4E22;</p>
        <p>&#x5F03;&#x4E4B;&#x524D;&#x5F00;&#x59CB;&#x7684;&#x6240;&#x6709;</p>
        <p>&#x90E8;&#x5206;&#x5339;&#x914D;<code>b1</code>&#xFF0C;</p>
        <p>&#x4F46;&#x6CA1;&#x6709;&#x8FD9;&#x6837;&#x7684;&#x5339;&#x914D;&#x3002;</p>
        <p>&#x56E0;&#x6B64;&#xFF0C;&#x4E0D;&#x4F1A;&#x4E22;&#x5F03;&#x4EFB;</p>
        <p>&#x4F55;&#x4E1C;&#x897F;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_TO_LAST</b> [ <code>b</code>]</td>
      <td style="text-align:left"><code>b1 b2 b3 c</code>
        <br /><code>b3 c</code>
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>b1 b2 b3 c</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x5C06;&#x5C1D;&#x8BD5;</p>
        <p>&#x4E22;&#x5F03;&#x4E4B;&#x524D;&#x5F00;&#x59CB;&#x7684;</p>
        <p>&#x6240;&#x6709;&#x90E8;&#x5206;&#x5339;&#x914D;<code>b3</code>&#x3002;</p>
        <p>&#x6709;&#x4E00;&#x4E2A;&#x8FD9;&#x6837;&#x7684;&#x6BD4;&#x8D5B;</p>
        <p><code>b2 b3 c</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

看看另一个例子，以便更好地看到`NO_SKIP`和`SKIP_TO_FIRST`之间的区别：模式：`(a | c) (b | c) c+.greedy d`和序列：`a b c1 c2 c3 d`然后结果将是：

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8DF3;&#x8FC7;&#x7B56;&#x7565;</th>
      <th style="text-align:left">&#x7ED3;&#x679C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>NO_SKIP</b>
      </td>
      <td style="text-align:left"><code>a b c1 c2 c3 d</code>
        <br /><code>b c1 c2 c3 d</code>
        <br /><code>c1 c2 c3 d</code>
        <br /><code>c2 c3 d</code>
        <br />
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>a b c1 c2 c3 d</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x4E0D;&#x4F1A;&#x4E22;&#x5F03;</p>
        <p>&#x4EFB;&#x4F55;&#x7ED3;&#x679C;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_TO_FIRST</b> [ <code>b*</code>]</td>
      <td style="text-align:left"><code>a b c1 c2 c3 d</code>
        <br /><code>c1 c2 c3 d</code>
        <br />
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>a b c1 c2 c3 d</code>&#xFF0C;</p>
        <p>&#x5339;&#x914D;&#x8FC7;&#x7A0B;&#x5C06;&#x4E22;&#x5F03;&#x4E4B;&#x524D;</p>
        <p>&#x5F00;&#x59CB;&#x7684;&#x6240;&#x6709;&#x90E8;&#x5206;&#x5339;&#x914D;</p>
        <p><code>c1</code>&#x3002;&#x6709;&#x4E00;&#x4E2A;&#x8FD9;&#x6837;&#x7684;</p>
        <p>&#x6BD4;&#x8D5B;<code>b c1 c2 c3 d</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

为了更好地理解`NO_SKIP`和`SKIP_TO_NEXT`之间的区别，请看下面的例子：模式：`a b+`和序列：`a b1 b2 b3`然后结果将是：

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8DF3;&#x8FC7;&#x7B56;&#x7565;</th>
      <th style="text-align:left">&#x7ED3;&#x679C;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>NO_SKIP</b>
      </td>
      <td style="text-align:left"><code>a b1</code>
        <br /><code>a b1 b2</code>
        <br /><code>a b1 b2 b3</code>
        <br />
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>a b1</code>&#xFF0C;&#x5339;</p>
        <p>&#x914D;&#x8FC7;&#x7A0B;&#x4E0D;&#x4F1A;</p>
        <p>&#x4E22;&#x5F03;&#x4EFB;&#x4F55;&#x7ED3;&#x679C;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>SKIP_TO_NEXT</b> [ <code>b*</code>]</td>
      <td style="text-align:left"><code>a b1</code>
        <br />
      </td>
      <td style="text-align:left">
        <p>&#x627E;&#x5230;&#x5339;&#x914D;&#x540E;</p>
        <p><code>a b1</code>&#xFF0C;&#x5339;&#x914D;</p>
        <p>&#x8FC7;&#x7A0B;&#x5C06;&#x4E22;&#x5F03;&#x6240;</p>
        <p>&#x6709;&#x5F00;&#x59CB;&#x7684;&#x90E8;&#x5206;</p>
        <p>&#x5339;&#x914D;<code>a</code>&#x3002;&#x8FD9;&#x610F;</p>
        <p>&#x5473;&#x7740;&#x65E2;<code>a b1 b2</code>
        </p>
        <p>&#x4E0D;&#x80FD;<code>a b1 b2 b3</code>
        </p>
        <p>&#x4E5F;&#x4E0D;&#x80FD;&#x751F;&#x6210;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>

要指定要使用的跳过策略，只需`AfterMatchSkipStrategy`通过调用创建：

| 功能 | 描述 |
| :--- | :--- |
| `AfterMatchSkipStrategy.noSkip()` | 创建**NO\_SKIP**跳过策略 |
| `AfterMatchSkipStrategy.skipToNext()` | 创建**SKIP\_TO\_NEXT**跳过策略 |
| `AfterMatchSkipStrategy.skipPastLastEvent()` | 创建**SKIP\_PAST\_LAST\_EVENT**跳过策略 |
| `AfterMatchSkipStrategy.skipToFirst(patternName)` | 使用引用的模式名称patternName创建**SKIP\_TO\_FIRST**跳过策略 |
| `AfterMatchSkipStrategy.skipToLast(patternName)` | 使用引用的模式名称patternName创建**SKIP\_TO\_LAST**跳过策略 |

然后通过调用将跳过策略应用于模式：

{% tabs %}
{% tab title="Java" %}
```java
AfterMatchSkipStrategy skipStrategy = ...
Pattern.begin("patternName", skipStrategy);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val skipStrategy = ...
Pattern.begin("patternName", skipStrategy)
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
**注意：**对于SKIP\_TO\_FIRST / LAST，当没有元素映射到指定变量时，如何处理大小写有两种选择。默认情况下，在这种情况下将使用NO\_SKIP策略。另一种选择是在这种情况下抛出异常。可以通过以下方式启用此选项：
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
AfterMatchSkipStrategy.skipToFirst(patternName).throwExceptionOnMiss()
```
{% endtab %}

{% tab title="Scala" %}
```scala
AfterMatchSkipStrategy.skipToFirst(patternName).throwExceptionOnMiss()
```
{% endtab %}
{% endtabs %}

## 检测模式

指定要查找的模式序列后，是时候将其应用于输入流以检测潜在匹配。 要针对模式序列运行事件流，必须创建`PatternStream`。 给定输入流输入，模式模式和可选的比较器比较器，用于在`EventTime`的情况下对具有相同时间戳的事件进行排序或在同一时刻到达，通过调用以下方法创建`PatternStream`：

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Event> input = ...
Pattern<Event, ?> pattern = ...
EventComparator<Event> comparator = ... // optional

PatternStream<Event> patternStream = CEP.pattern(input, pattern, comparator);
```
{% endtab %}

{% tab title="Scala" %}
```scala

val input : DataStream[Event] = ...
val pattern : Pattern[Event, _] = ...
var comparator : EventComparator[Event] = ... // optional

val patternStream: PatternStream[Event] = CEP.pattern(input, pattern, comparator)
```
{% endtab %}
{% endtabs %}

根据您的使用情况，输入流可以是键控的或非键控的。

{% hint style="danger" %}
**注意**:在非键控流上应用模式将导致并行度等于1的作业。
{% endhint %}

### 从模式中选择

一旦获得了PatternStream，就可以通过select或flatSelect方法从检测到的事件序列中进行选择。

{% tabs %}
{% tab title="Java" %}
`select()`方法需要`PatternSelectFunction`实现。 `PatternSelectFunction`具有为每个匹配事件序列调用的`select`方法。 它以`Map<String, List<IN>>` 的形式接收匹配，其中键是模式序列中每个模式的名称，值是该模式的所有已接受事件的列表（IN是你的类型） 输入元素）。 给定模式的事件按时间戳排序。 返回每个模式的接受事件列表的原因是当使用循环模式（例如，`oneToMany()`和`times()`）时，对于给定模式可以接受多个事件。 选择函数只返回一个结果。

```java
class MyPatternSelectFunction<IN, OUT> implements PatternSelectFunction<IN, OUT> {
    @Override
    public OUT select(Map<String, List<IN>> pattern) {
        IN startEvent = pattern.get("start").get(0);
        IN endEvent = pattern.get("end").get(0);
        return new OUT(startEvent, endEvent);
    }
}
```

`PatternFlatSelectFunction`类似于`PatternSelectFunction`，唯一的区别是它可以返回任意数量的结果。为此，`select`方法有一个附加`Collector`参数，用于将输出元素向下游转发。

```java
class MyPatternFlatSelectFunction<IN, OUT> implements PatternFlatSelectFunction<IN, OUT> {
    @Override
    public void flatSelect(Map<String, List<IN>> pattern, Collector<OUT> collector) {
        IN startEvent = pattern.get("start").get(0);
        IN endEvent = pattern.get("end").get(0);

        for (int i = 0; i < startEvent.getValue(); i++ ) {
            collector.collect(new OUT(startEvent, endEvent));
        }
    }
}
```
{% endtab %}

{% tab title="Scala" %}
`select()`方法将选择函数作为参数，为每个匹配的事件序列调用。 它以`Map [String，Iterable [IN]]`的形式接收匹配，其中键是模式序列中每个模式的名称，值是该模式的所有已接受事件的`Iterable`（IN是你的类型） 输入元素）。

给定模式的事件按时间戳排序。 为每个模式返回可迭代的可接受事件的原因是当使用循环模式（例如，`oneToMany()`和`times()`）时，对于给定模式可以接受多个事件。 选择函数每次调用只返回一个结果。

```scala
def selectFn(pattern : Map[String, Iterable[IN]]): OUT = {
    val startEvent = pattern.get("start").get.next
    val endEvent = pattern.get("end").get.next
    OUT(startEvent, endEvent)
}
```

`flatSelect`方法与`select`方法类似。 它们唯一的区别是传递给`flatSelect`方法的函数可以为每次调用返回任意数量的结果。 为此，`flatSelect`函数有一个额外的`Collector`参数，用于将输出元素向下游转发。

```scala
def flatSelectFn(pattern : Map[String, Iterable[IN]], collector : Collector[OUT]) = {
    val startEvent = pattern.get("start").get.next
    val endEvent = pattern.get("end").get.next
    for (i <- 0 to startEvent.getValue) {
        collector.collect(OUT(startEvent, endEvent))
    }
}
```
{% endtab %}
{% endtabs %}

### 处理超时部分模式

每当模式通过`within`关键字附加窗口长度时，部分事件序列可能因为超出窗口长度而被丢弃。为了对这些超时的部分匹配做出反应`select` ，`flatSelect`API调用允许指定超时处理程序。为每个超时的部分事件序列调用此超时处理程序。超时处理程序接收到目前为止由模式匹配的所有事件，以及检测到超时时的时间戳。

为了处理部分模式，`select`和`flatSelect`API调用提供了一个重载版本，它作为参数：

* `PatternTimeoutFunction`/`PatternFlatTimeoutFunction`
* 将返回超时匹配的侧输出的[OutputTag](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/stream/side_output.html)
* 和已知`PatternSelectFunction`/ `PatternFlatSelectFunction`

{% tabs %}
{% tab title="Java" %}
```java
PatternStream<Event> patternStream = CEP.pattern(input, pattern);

OutputTag<String> outputTag = new OutputTag<String>("side-output"){};

SingleOutputStreamOperator<ComplexEvent> result = patternStream.select(
    outputTag,
    new PatternTimeoutFunction<Event, TimeoutEvent>() {...},
    new PatternSelectFunction<Event, ComplexEvent>() {...}
);

DataStream<TimeoutEvent> timeoutResult = result.getSideOutput(outputTag);

SingleOutputStreamOperator<ComplexEvent> flatResult = patternStream.flatSelect(
    outputTag,
    new PatternFlatTimeoutFunction<Event, TimeoutEvent>() {...},
    new PatternFlatSelectFunction<Event, ComplexEvent>() {...}
);

DataStream<TimeoutEvent> timeoutFlatResult = flatResult.getSideOutput(outputTag);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val patternStream: PatternStream[Event] = CEP.pattern(input, pattern)

val outputTag = OutputTag[String]("side-output")

val result: SingleOutputStreamOperator[ComplexEvent] = patternStream.select(outputTag){
    (pattern: Map[String, Iterable[Event]], timestamp: Long) => TimeoutEvent()
} {
    pattern: Map[String, Iterable[Event]] => ComplexEvent()
}

val timeoutResult: DataStream<TimeoutEvent> = result.getSideOutput(outputTag)
```

`flatSelect` API调用提供了相同的重载版本，它以超时函数作为第一个参数，以选择函数作为第二个参数。与`select`函数不同，`flatSelect`函数由收集器调用。可以使用收集器发出任意数量的事件。

```scala
val patternStream: PatternStream[Event] = CEP.pattern(input, pattern)

val outputTag = OutputTag[String]("side-output")

val result: SingleOutputStreamOperator[ComplexEvent] = patternStream.flatSelect(outputTag){
    (pattern: Map[String, Iterable[Event]], timestamp: Long, out: Collector[TimeoutEvent]) =>
        out.collect(TimeoutEvent())
} {
    (pattern: mutable.Map[String, Iterable[Event]], out: Collector[ComplexEvent]) =>
        out.collect(ComplexEvent())
}

val timeoutResult: DataStream<TimeoutEvent> = result.getSideOutput(outputTag)
```
{% endtab %}
{% endtabs %}

## 处理EventTime延迟

在CEP中，元素被处理的顺序。为了保证在EventTime工作时元素以正确的顺序处理，传入元素最初放在缓冲区中，元素_按照时间戳按升序排序_，当水位线到达时，此缓冲区中的所有元素都包含在处理小于水位线的时间戳。这意味着水位线之间的元素按事件时间顺序处理。

{% hint style="danger" %}
注意：在事件时间工作时，Library假定水位线是正确的。
{% endhint %}

为了保证水位线中的元素按事件时间顺序处理，Flink的CEP库假定 水位线_的正确性_，并将其视为时间戳小于上次看到的水位线的_后期_元素。后期元素不会被进一步处理。此外，您可以指定sideOutput标记来收集最后看到的水位线之后的后期元素，可以像这样使用：

{% tabs %}
{% tab title="Java" %}
```java
PatternStream<Event> patternStream = CEP.pattern(input, pattern);

OutputTag<String> lateDataOutputTag = new OutputTag<String>("late-data"){};

SingleOutputStreamOperator<ComplexEvent> result = patternStream
    .sideOutputLateData(lateDataOutputTag)
    .select(
        new PatternSelectFunction<Event, ComplexEvent>() {...}
    );

DataStream<String> lateData = result.getSideOutput(lateDataOutputTag);
```
{% endtab %}

{% tab title="Scala" %}
```scala
val patternStream: PatternStream[Event] = CEP.pattern(input, pattern)

val lateDataOutputTag = OutputTag[String]("late-data")

val result: SingleOutputStreamOperator[ComplexEvent] = patternStream
      .sideOutputLateData(lateDataOutputTag)
      .select{
          pattern: Map[String, Iterable[ComplexEvent]] => ComplexEvent()
      }

val lateData: DataStream<String> = result.getSideOutput(lateDataOutputTag)
```
{% endtab %}
{% endtabs %}

## 例子

以下的示例检测事件的键控数据流上的模式start、middle\(name = "error"\) -&gt; end\(name = "critical"\)。事件由id作为key，并且有效模式必须在10秒内发生。整个处理是在事件时间完成的。

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = ...
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<Event> input = ...

DataStream<Event> partitionedInput = input.keyBy(new KeySelector<Event, Integer>() {
	@Override
	public Integer getKey(Event value) throws Exception {
		return value.getId();
	}
});

Pattern<Event, ?> pattern = Pattern.<Event>begin("start")
	.next("middle").where(new SimpleCondition<Event>() {
		@Override
		public boolean filter(Event value) throws Exception {
			return value.getName().equals("error");
		}
	}).followedBy("end").where(new SimpleCondition<Event>() {
		@Override
		public boolean filter(Event value) throws Exception {
			return value.getName().equals("critical");
		}
	}).within(Time.seconds(10));

PatternStream<Event> patternStream = CEP.pattern(partitionedInput, pattern);

DataStream<Alert> alerts = patternStream.select(new PatternSelectFunction<Event, Alert>() {
	@Override
	public Alert select(Map<String, List<Event>> pattern) throws Exception {
		return createAlert(pattern);
	}
});
```
{% endtab %}

{% tab title="Scala" %}
```scala
StreamExecutionEnvironment env = ...
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);

DataStream<Event> input = ...

DataStream<Event> partitionedInput = input.keyBy(new KeySelector<Event, Integer>() {
	@Override
	public Integer getKey(Event value) throws Exception {
		return value.getId();
	}
});

Pattern<Event, ?> pattern = Pattern.<Event>begin("start")
	.next("middle").where(new SimpleCondition<Event>() {
		@Override
		public boolean filter(Event value) throws Exception {
			return value.getName().equals("error");
		}
	}).followedBy("end").where(new SimpleCondition<Event>() {
		@Override
		public boolean filter(Event value) throws Exception {
			return value.getName().equals("critical");
		}
	}).within(Time.seconds(10));

PatternStream<Event> patternStream = CEP.pattern(partitionedInput, pattern);

DataStream<Alert> alerts = patternStream.select(new PatternSelectFunction<Event, Alert>() {
	@Override
	public Alert select(Map<String, List<Event>> pattern) throws Exception {
		return createAlert(pattern);
	}
});
```
{% endtab %}
{% endtabs %}

## 从较旧的Flink版本迁移\(1.3之前版本\)

### 迁移到1.4+

在Flink-1.4中放弃了CEP库与&lt;= Flink 1.2的向后兼容性。不幸的是，无法恢复曾经在1.2.x中运行的CEP作业

### 迁移到1.3.x

Flink-1.3中的CEP库附带了许多新功能，这些功能导致了API的一些变化。在这里，我们描述了您需要对旧CEP作业进行的更改，以便能够使用Flink-1.3运行它们。完成这些更改并重新编译作业后，您将能够从使用旧版本作业的保存点恢复执行，_即_无需重新处理过去的数据。

所需的更改是：

1. 更改条件（`where(...)`子句中的条件）以扩展`SimpleCondition`类而不是实现`FilterFunction`接口。
2. 将作为参数提供给select\(…\)和flatSelect\(…\)方法的函数更改为期望与每个模式关联的事件列表\(Java中的List，Scala中的Iterable\)。这是因为添加了循环模式后，多个输入事件可以匹配一个\(循环\)模式。
3. Fl.1.1和1.2中的followed.\(\)暗示了非确定性松弛邻接（参见这里）。在Flink 1.3中，这已经改变，followed.\(\)意味着松弛的连续性，而followedBy.\(\)在需要非确定性松弛的连续性时应该使用。

