# Table API

Table API是用于流和批处理的统一关系API。 表API查询可以在批量或流式输入上运行而无需修改。 Table API是SQL语言的超级集合，专门用于Apache Flink。 Table API是Scala和Java的语言集成API。 Table API查询不是像SQL中常见的那样将查询指定为String值，而是在Java或Scala中以嵌入语言的样式定义，具有IDE支持，如自动完成和语法验证。

Table API与Flink的SQL集成共享其API的许多概念和部分。查看[Common Concepts＆API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html)以了解如何注册表或创建`Table`对象。该[流概念](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming)的网页讨论流如动态表和时间属性，具体的概念。

以下示例假设一个名为Orders的已注册表，其中包含属性（a，b，c，rowtime）。 rowtime字段是流式传输中的逻辑[时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)或批处理中的常规时间戳字段。

## 概述和示例

Table API可用于Scala和Java。Scala Table API利用Scala表达式，Java Table API基于字符串，这些字符串被解析并转换为等效表达式。

以下示例显示了Scala和Java Table API之间的差异。 表程序在批处理环境中执行。 它按字段a扫描Orders表，分组，并计算每组的结果行数。 表程序的结果转换为Row类型的DataSet并打印。

{% tabs %}
{% tab title="Java" %}
通过导入`org.apache.flink.table.api.java.*`来启用Java Table API。 以下示例显示如何构造Java Table API程序以及如何将表达式指定为字符串。

```java
// environment configuration
ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
BatchTableEnvironment tEnv = TableEnvironment.getTableEnvironment(env);

// register Orders table in table environment
// ...

// specify table program
Table orders = tEnv.scan("Orders"); // schema (a, b, c, rowtime)

Table counts = orders
        .groupBy("a")
        .select("a, b.count as cnt");

// conversion to DataSet
DataSet<Row> result = tEnv.toDataSet(counts, Row.class);
result.print();
```
{% endtab %}

{% tab title="Scala" %}
  
Scala Table API通过导入`org.apache.flink.api.scala._`和`org.apache.flink.table.api.scala._`启用。

以下示例显示了如何构造Scala Table API程序。使用[Scala符号](http://scala-lang.org/files/archive/spec/2.12/01-lexical-syntax.html#symbol-literals)引用表属性，该[符号](http://scala-lang.org/files/archive/spec/2.12/01-lexical-syntax.html#symbol-literals)以撇号字符（`'`）开头。

```scala
import org.apache.flink.api.scala._
import org.apache.flink.table.api.scala._

// environment configuration
val env = ExecutionEnvironment.getExecutionEnvironment
val tEnv = TableEnvironment.getTableEnvironment(env)

// register Orders table in table environment
// ...

// specify table program
val orders = tEnv.scan("Orders") // schema (a, b, c, rowtime)

val result = orders
               .groupBy('a)
               .select('a, 'b.count as 'cnt)
               .toDataSet[Row] // conversion to DataSet
               .print()
```
{% endtab %}

{% tab title="Python" %}
通过`from pyflink.table import *`引入Python Table API。

以下示例展示了如何构建Python Table API程序以及如何将表达式指定为字符串。

```python
from pyflink.table import *
from pyflink.dataset import *

# environment configuration
env = ExecutionEnvironment.get_execution_environment()
t_env = TableEnvironment.create(env, TableConfig())

# register Orders table and Result table sink in table environment
# ...

# specify table program
orders = t_env.from_path("Orders")  # schema (a, b, c, rowtime)

orders.group_by("a").select("a, b.count as cnt").insert_into("result")

t_env.execute("python_job")
```
{% endtab %}
{% endtabs %}

下一个示例显示了一个更复杂的Table API程序。 程序再次扫描Orders表。 它过滤空值，规范化String类型的字段a，并计算每小时和产品的平均计费金额b。

{% tabs %}
{% tab title="Java" %}
```java
// environment configuration
// ...

// specify table program
Table orders = tEnv.scan("Orders"); // schema (a, b, c, rowtime)

Table result = orders
        .filter("a.isNotNull && b.isNotNull && c.isNotNull")
        .select("a.lowerCase() as a, b, rowtime")
        .window(Tumble.over("1.hour").on("rowtime").as("hourlyWindow"))
        .groupBy("hourlyWindow, a")
        .select("a, hourlyWindow.end as hour, b.avg as avgBillingAmount");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// environment configuration
// ...

// specify table program
val orders: Table = tEnv.scan("Orders") // schema (a, b, c, rowtime)

val result: Table = orders
        .filter('a.isNotNull && 'b.isNotNull && 'c.isNotNull)
        .select('a.lowerCase() as 'a, 'b, 'rowtime)
        .window(Tumble over 1.hour on 'rowtime as 'hourlyWindow)
        .groupBy('hourlyWindow, 'a)
        .select('a, 'hourlyWindow.end as 'hour, 'b.avg as 'avgBillingAmount)
```
{% endtab %}

{% tab title="" %}
```python
# environment configuration
# ...

# specify table program
orders = t_env.from_path("Orders")  # schema (a, b, c, rowtime)

result = orders.filter("a.isNotNull && b.isNotNull && c.isNotNull") \
               .select("a.lowerCase() as a, b, rowtime") \
               .window(Tumble.over("1.hour").on("rowtime").alias("hourlyWindow")) \
               .group_by("hourlyWindow, a") \
               .select("a, hourlyWindow.end as hour, b.avg as avgBillingAmount")
```
{% endtab %}
{% endtabs %}

由于Table API是批量和流数据的统一API，因此两个示例程序都可以在批处理和流式输入上执行，而无需对表程序本身进行任何修改。在这两种情况下，程序产生相同的结果，因为流记录不延迟（有关详细信息，请参阅[流式概念](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming)）。

## 操作

Table API支持以下操作。请注意，并非所有操作都可用于批处理和流式处理; 他们被相应地标记。

### 扫描，投影和过滤\(Scan, Projection, and Filter\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Scan</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;FROM&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x6267;&#x884C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x626B;&#x63CF;&#x3002;</p>
        <p>Table orders = tableEnv.scan(&quot;Orders&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Select</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL SELECT&#x8BED;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x6267;&#x884C;&#x9009;&#x62E9;&#x64CD;&#x4F5C;&#x3002;
          <br
          />Table orders = tableEnv.scan(&quot;Orders&quot;);
          <br />Table result = orders.select(&quot;a, c as d&quot;);</p>
        <p>&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x661F;&#x53F7;(*)&#x4F5C;&#x4E3A;&#x901A;&#x914D;&#x7B26;&#xFF0C;&#x9009;&#x62E9;&#x8868;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x5217;&#x3002;</p>
        <p>Table result = orders.select(&quot;*&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>As</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x91CD;&#x547D;&#x540D;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>Table orders = tableEnv.scan(&quot;Orders&quot;);
          <br />Table result = orders.as(&quot;x, y, z, t&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Where / Filter</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL WHERE&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002; &#x8FC7;&#x6EE4;&#x6389;&#x672A;&#x901A;&#x8FC7;&#x8FC7;&#x6EE4;&#x8C13;&#x8BCD;&#x7684;&#x884C;&#x3002;</p>
        <p>Table orders = tableEnv.scan(&quot;Orders&quot;);
          <br />Table result = orders.where(&quot;b === &apos;red&apos;&quot;);</p>
        <p>&#x6216;</p>
        <p>Table orders = tableEnv.scan(&quot;Orders&quot;);
          <br />Table result = orders.filter(&quot;a % 2 === 0&quot;);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Scan</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;FROM&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x6267;&#x884C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x626B;&#x63CF;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Select</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL SELECT&#x8BED;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x6267;&#x884C;&#x9009;&#x62E9;&#x64CD;&#x4F5C;&#x3002;
          <br
          />val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.select(&apos;a, &apos;c as &apos;d)</p>
        <p>&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x661F;&#x53F7;(*)&#x4F5C;&#x4E3A;&#x901A;&#x914D;&#x7B26;&#xFF0C;&#x9009;&#x62E9;&#x8868;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x5217;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.select(&apos;*)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>As</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x91CD;&#x547D;&#x540D;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;).as(&apos;x, &apos;y,
          &apos;z, &apos;t)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Where / Filter</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL WHERE&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002; &#x8FC7;&#x6EE4;&#x6389;&#x672A;&#x901A;&#x8FC7;&#x8FC7;&#x6EE4;&#x8C13;&#x8BCD;&#x7684;&#x884C;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.filter(&apos;a % 2 === 0)</p>
        <p>&#x6216;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.where(&apos;b === &quot;red&quot;)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Scan</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;FROM&#x5B50;&#x53E5;&#x3002;&#x626B;&#x63CF;&#x5DF2;&#x6CE8;&#x518C;&#x7684;&#x8868;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Select</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL SELECT&#x8BED;&#x53E5;&#x3002;&#x6267;&#x884C;&#x9009;&#x62E9;&#x64CD;&#x4F5C;&#x3002;
          <br
          />orders = table_env.from_path(&quot;Orders&quot;) result = orders.select(&quot;a,
          c as d&quot;)</p>
        <p>&#x60A8;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x661F;&#x53F7;&#xFF08;<code>*</code>&#xFF09;&#x5145;&#x5F53;&#x901A;&#x914D;&#x7B26;&#xFF0C;&#x9009;&#x62E9;&#x8868;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x5217;&#x3002;</p>
        <p>result = orders.select(&quot;*&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>As</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x91CD;&#x547D;&#x540D;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.alias(&quot;x,
          y, z, t&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Where / Filter</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL WHERE&#x5B50;&#x53E5;&#x3002;&#x8FC7;&#x6EE4;&#x6389;&#x672A;&#x901A;&#x8FC7;&#x8FC7;&#x6EE4;&#x8C13;&#x8BCD;&#x7684;&#x884C;&#x3002;
          <br
          />orders = table_env.from_path(&quot;Orders&quot;) result = orders.where(&quot;b
          === &apos;red&apos;&quot;)</p>
        <p>&#x8981;&#x4E48;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.filter(&quot;a
          % 2 === 0&quot;)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 列操作

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>AddColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5DF2;&#x7ECF;&#x5B58;&#x5728;&#xFF0C;&#x5B83;&#x5C06;&#x5F15;&#x53D1;&#x5F02;&#x5E38;&#x3002;</p>
        <p>Table orders = tableEnv.from(&quot;Orders&quot;); Table result = orders.addColumns(&quot;concat(c,
          &apos;sunny&apos;)&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>AddOrReplaceColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x5217;&#x540D;&#x79F0;&#x4E0E;&#x73B0;&#x6709;&#x5217;&#x540D;&#x79F0;&#x76F8;&#x540C;&#xFF0C;&#x5219;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x5C06;&#x88AB;&#x66FF;&#x6362;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5177;&#x6709;&#x91CD;&#x590D;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x3002;
        <br
        />Table orders = tableEnv.from(&quot;Orders&quot;); Table result = orders.addOrReplaceColumns(&quot;concat(c,
        &apos;sunny&apos;) as desc&quot;);</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>DropColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x5220;&#x9664;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x5B57;&#x6BB5;&#x5F15;&#x7528;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x5220;&#x9664;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>Table orders = tableEnv.from(&quot;Orders&quot;); Table result = orders.dropColumns(&quot;b,
          c&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>RenameColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x91CD;&#x547D;&#x540D;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x522B;&#x540D;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x91CD;&#x547D;&#x540D;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>Table orders = tableEnv.from(&quot;Orders&quot;); Table result = orders.renameColumns(&quot;b
          as b2, c as c2&quot;);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>AddColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5DF2;&#x7ECF;&#x5B58;&#x5728;&#xFF0C;&#x5B83;&#x5C06;&#x5F15;&#x53D1;&#x5F02;&#x5E38;&#x3002;</p>
        <p>val orders = tableEnv.from(&quot;Orders&quot;); val result = orders.addColumns(concat(&apos;c,
          &quot;Sunny&quot;))</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>AddOrReplaceColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x5217;&#x540D;&#x79F0;&#x4E0E;&#x73B0;&#x6709;&#x5217;&#x540D;&#x79F0;&#x76F8;&#x540C;&#xFF0C;&#x5219;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x5C06;&#x88AB;&#x66FF;&#x6362;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5177;&#x6709;&#x91CD;&#x590D;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x3002;</p>
        <p>val orders = tableEnv.from(&quot;Orders&quot;); val result = orders.addOrReplaceColumns(concat(&apos;c,
          &quot;Sunny&quot;) as &apos;desc)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>DropColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x5220;&#x9664;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x5B57;&#x6BB5;&#x5F15;&#x7528;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x5220;&#x9664;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>val orders = tableEnv.from(&quot;Orders&quot;); val result = orders.dropColumns(&apos;b,
          &apos;c)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>RenameColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x91CD;&#x547D;&#x540D;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x522B;&#x540D;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x91CD;&#x547D;&#x540D;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>val orders = tableEnv.from(&quot;Orders&quot;); val result = orders.renameColumns(&apos;b
          as &apos;b2, &apos;c as &apos;c2)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>AddColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5DF2;&#x7ECF;&#x5B58;&#x5728;&#xFF0C;&#x5B83;&#x5C06;&#x5F15;&#x53D1;&#x5F02;&#x5E38;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.add_columns(&quot;concat(c,
          &apos;sunny&apos;)&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>AddOrReplaceColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x6DFB;&#x52A0;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x5217;&#x540D;&#x79F0;&#x4E0E;&#x73B0;&#x6709;&#x5217;&#x540D;&#x79F0;&#x76F8;&#x540C;&#xFF0C;&#x5219;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x5C06;&#x88AB;&#x66FF;&#x6362;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5982;&#x679C;&#x6DFB;&#x52A0;&#x7684;&#x5B57;&#x6BB5;&#x5177;&#x6709;&#x91CD;&#x590D;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5219;&#x4F7F;&#x7528;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.add_or_replace_columns(&quot;concat(c,
          &apos;sunny&apos;) as desc&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>DropColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x5220;&#x9664;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x5B57;&#x6BB5;&#x5F15;&#x7528;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x5220;&#x9664;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.drop_columns(&quot;b,
          c&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>RenameColumns</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x6267;&#x884C;&#x5B57;&#x6BB5;&#x91CD;&#x547D;&#x540D;&#x64CD;&#x4F5C;&#x3002;&#x5B57;&#x6BB5;&#x8868;&#x8FBE;&#x5F0F;&#x5E94;&#x8BE5;&#x662F;&#x522B;&#x540D;&#x8868;&#x8FBE;&#x5F0F;&#xFF0C;&#x5E76;&#x4E14;&#x53EA;&#x80FD;&#x91CD;&#x547D;&#x540D;&#x73B0;&#x6709;&#x5B57;&#x6BB5;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;) result = orders.rename_columns(&quot;b
          as b2, c as c2&quot;)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 聚合\(Aggregations\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>GroupBy Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL GROUP BY&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x4F7F;&#x7528;&#x4EE5;&#x4E0B;&#x8FD0;&#x884C;&#x7684;&#x805A;&#x5408;&#x8FD0;&#x7B97;&#x7B26;&#x5BF9;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x7684;&#x884C;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#xFF0C;&#x4EE5;&#x6309;&#x7EC4;&#x805A;&#x5408;&#x884C;&#x3002;
          <br
          />Table orders = tableEnv.scan(&quot;Orders&quot;);
          <br />Table result = orders.groupBy(&quot;a&quot;).select(&quot;a, b.sum as
          d&quot;);</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x805A;&#x5408;&#x7C7B;&#x578B;&#x548C;&#x4E0D;&#x540C;&#x5206;&#x7EC4;&#x952E;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>GroupBy Window Aggregation</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7EC4;&#x7A97;&#x53E3;&#x4E0A;&#x7684;&#x8868;&#x4EE5;&#x53CA;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x5408;&#x3002;</p>
        <p>Table orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>Table result <b>=</b> orders</p>
        <p> <b>.</b>window(Tumble<b>.</b>over(&quot;5.minutes&quot;)<b>.</b>on(&quot;rowtime&quot;)<b>.</b>as(&quot;w&quot;))
          // define window</p>
        <p> <b>.</b>groupBy(&quot;a, w&quot;) // group by key and window</p>
        <p> <b>.</b>select(&quot;a, w.start, w.end, w.rowtime, b.sum as d&quot;);
          // access window properties and aggregate</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Over Window Aggregation</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL OVER&#x5B50;&#x53E5;&#x3002;&#x57FA;&#x4E8E;&#x524D;&#x4E00;&#x884C;&#x548C;&#x540E;&#x4E00;&#x884C;&#x7684;&#x7A97;&#x53E3;&#xFF08;&#x8303;&#x56F4;&#xFF09;&#x8BA1;&#x7B97;&#x6BCF;&#x884C;&#x7684;&#x7A97;&#x53E3;&#x805A;&#x5408;&#x3002;&#x6709;&#x5173;&#x66F4;&#x591A;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#over-windows">over windows&#x90E8;&#x5206;</a>&#x3002;</p>
        <p>Table orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>Table result <b>=</b> orders</p>
        <p>// define window</p>
        <p> <b>.</b>window(Over</p>
        <p> <b>.</b>partitionBy(&quot;a&quot;)</p>
        <p> <b>.</b>orderBy(&quot;rowtime&quot;)</p>
        <p> <b>.</b>preceding(&quot;UNBOUNDED_RANGE&quot;)</p>
        <p> <b>.</b>following(&quot;CURRENT_RANGE&quot;)</p>
        <p> <b>.</b>as(&quot;w&quot;))</p>
        <p> <b>.</b>select(&quot;a, b.avg over w, b.max over w, b.min over w&quot;);
          // sliding aggregate</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5FC5;&#x987B;&#x5728;&#x540C;&#x4E00;&#x7A97;&#x53E3;&#x4E2D;&#x5B9A;&#x4E49;&#x6240;&#x6709;&#x805A;&#x5408;&#xFF0C;&#x5373;&#x76F8;&#x540C;&#x7684;&#x5206;&#x533A;&#xFF0C;&#x6392;&#x5E8F;&#x548C;&#x8303;&#x56F4;&#x3002;&#x76EE;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x5177;&#x6709;PRREDING&#xFF08;UNBOUNDED&#x548C;&#x6709;&#x754C;&#xFF09;&#x5230;CURRENT
          ROW&#x8303;&#x56F4;&#x7684;&#x7A97;&#x53E3;&#x3002;&#x5C1A;&#x4E0D;&#x652F;&#x6301;&#x4F7F;&#x7528;FOLLOWING&#x7684;&#x8303;&#x56F4;&#x3002;&#x5FC5;&#x987B;&#x5728;&#x5355;&#x4E2A;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html">&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#x4E0A;&#x6307;&#x5B9A;ORDER BY &#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL DISTINCT</b>&#x805A;&#x5408;&#x5B50;&#x53E5;&#xFF0C;&#x4F8B;&#x5982;<b>COUNT</b>&#xFF08;<b>DISTINCT a</b>&#xFF09;&#x3002;&#x4E0D;&#x540C;&#x805A;&#x5408;&#x58F0;&#x660E;&#x805A;&#x5408;&#x51FD;&#x6570;&#xFF08;&#x5185;&#x7F6E;&#x6216;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#xFF09;&#x4EC5;&#x5E94;&#x7528;&#x4E8E;&#x4E0D;&#x540C;&#x7684;&#x8F93;&#x5165;&#x503C;&#x3002;<b>Distinct</b>&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;<b>GroupBy</b>&#x805A;&#x5408;&#xFF0C;<b>GroupBy</b>&#x7A97;&#x53E3;&#x805A;&#x5408;&#x548C;<b>Over Window Aggregation</b>&#x3002;</p>
        <p>Table orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>// Distinct aggregation on group by</p>
        <p>Table groupByDistinctResult <b>=</b> orders</p>
        <p> <b>.</b>groupBy(&quot;a&quot;)</p>
        <p> <b>.</b>select(&quot;a, b.sum.distinct as d&quot;);</p>
        <p>// Distinct aggregation on time window group by</p>
        <p>Table groupByWindowDistinctResult <b>=</b> orders</p>
        <p> <b>.</b>window(Tumble<b>.</b>over(&quot;5.minutes&quot;)<b>.</b>on(&quot;rowtime&quot;)<b>.</b>as(&quot;w&quot;))<b>.</b>groupBy(&quot;a,
          w&quot;)</p>
        <p> <b>.</b>select(&quot;a, b.sum.distinct as d&quot;);</p>
        <p>// Distinct aggregation on over window</p>
        <p>Table result <b>=</b> orders</p>
        <p> <b>.</b>window(Over</p>
        <p> <b>.</b>partitionBy(&quot;a&quot;)</p>
        <p> <b>.</b>orderBy(&quot;rowtime&quot;)</p>
        <p> <b>.</b>preceding(&quot;UNBOUNDED_RANGE&quot;)</p>
        <p> <b>.</b>as(&quot;w&quot;))</p>
        <p> <b>.</b>select(&quot;a, b.avg.distinct over w, b.max over w, b.min over
          w&quot;);</p>
        <p>&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x805A;&#x5408;&#x51FD;&#x6570;&#x4E5F;&#x53EF;&#x4EE5;&#x4E0E;<b>DISTINCT</b>&#x4FEE;&#x9970;&#x7B26;&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#x3002;&#x8981;&#x4EC5;&#x4E3A;&#x4E0D;&#x540C;&#x7684;&#x503C;&#x8BA1;&#x7B97;&#x805A;&#x5408;&#x7ED3;&#x679C;&#xFF0C;&#x53EA;&#x9700;&#x5C06;<b>distinct</b>&#x4FEE;&#x9970;&#x7B26;&#x6DFB;&#x52A0;&#x5230;&#x805A;&#x5408;&#x51FD;&#x6570;&#x5373;&#x53EF;&#x3002;</p>
        <p>Table orders <b>=</b> tEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>// Use distinct aggregation for user-defined aggregate functions</p>
        <p>tEnv<b>.</b>registerFunction(&quot;myUdagg&quot;, <b>new</b> MyUdagg());</p>
        <p>orders<b>.</b>groupBy(&quot;users&quot;)<b>.</b>select(&quot;users, myUdagg.distinct(points)
          as myDistinctResult&quot;);</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL DISTINCT&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8FD4;&#x56DE;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x503C;&#x7EC4;&#x5408;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p>Table orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>Table result <b>=</b> orders<b>.</b>distinct();</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>GroupBy Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL GROUP BY&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x4F7F;&#x7528;&#x4EE5;&#x4E0B;&#x8FD0;&#x884C;&#x7684;&#x805A;&#x5408;&#x8FD0;&#x7B97;&#x7B26;&#x5BF9;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x7684;&#x884C;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#xFF0C;&#x4EE5;&#x6309;&#x7EC4;&#x805A;&#x5408;&#x884C;&#x3002;
          <br
          />val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.groupBy(&apos;a).select(&apos;a, &apos;b.sum as &apos;d)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x805A;&#x5408;&#x7C7B;&#x578B;&#x548C;&#x4E0D;&#x540C;&#x5206;&#x7EC4;&#x952E;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>GroupBy Window Aggregation</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x7EC4;&#x7A97;&#x53E3;&#x4E0A;&#x7684;&#x8868;&#x4EE5;&#x53CA;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x5408;&#x3002;</p>
        <p><b>val</b> orders<b>:</b>  <b>Table</b>  <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;)</p>
        <p><b>val</b> result<b>:</b>  <b>Table</b>  <b>=</b> orders</p>
        <p> <b>.</b>window(<b>Tumble</b> over 5.minutes on &apos;rowtime as &apos;w)
          // define window</p>
        <p> <b>.</b>groupBy(&apos;a, &apos;w) // group by key and window</p>
        <p> <b>.</b>select(&apos;a, w<b>.</b>start, &apos;w.end, &apos;w.rowtime,
          &apos;b.sum as &apos;d) <b>//</b> access window properties and aggregate</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Over Window Aggregation</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL OVER&#x5B50;&#x53E5;&#x3002;&#x57FA;&#x4E8E;&#x524D;&#x4E00;&#x884C;&#x548C;&#x540E;&#x4E00;&#x884C;&#x7684;&#x7A97;&#x53E3;&#xFF08;&#x8303;&#x56F4;&#xFF09;&#x8BA1;&#x7B97;&#x6BCF;&#x884C;&#x7684;&#x7A97;&#x53E3;&#x805A;&#x5408;&#x3002;&#x6709;&#x5173;&#x66F4;&#x591A;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#over-windows">over windows&#x90E8;&#x5206;</a>&#x3002;</p>
        <p><b>val</b> orders<b>:</b>  <b>Table</b>  <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;)</p>
        <p><b>val</b> result<b>:</b>  <b>Table</b>  <b>=</b> orders</p>
        <p>// define window</p>
        <p> <b>.</b>window(<b>Over</b> 
        </p>
        <p>partitionBy &apos;a</p>
        <p>orderBy &apos;rowtime</p>
        <p>preceding <b>UNBOUNDED_RANGE</b>
        </p>
        <p>following <b>CURRENT_RANGE</b>
        </p>
        <p>as &apos;w)</p>
        <p> <b>.</b>select(&apos;a, &apos;b.avg over &apos;w, &apos;b.max over &apos;w,
          &apos;b.min over &apos;w) <b>//</b> sliding aggregate</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5FC5;&#x987B;&#x5728;&#x540C;&#x4E00;&#x7A97;&#x53E3;&#x4E2D;&#x5B9A;&#x4E49;&#x6240;&#x6709;&#x805A;&#x5408;&#xFF0C;&#x5373;&#x76F8;&#x540C;&#x7684;&#x5206;&#x533A;&#xFF0C;&#x6392;&#x5E8F;&#x548C;&#x8303;&#x56F4;&#x3002;&#x76EE;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x5177;&#x6709;PRREDING&#xFF08;UNBOUNDED&#x548C;&#x6709;&#x754C;&#xFF09;&#x5230;CURRENT
          ROW&#x8303;&#x56F4;&#x7684;&#x7A97;&#x53E3;&#x3002;&#x5C1A;&#x4E0D;&#x652F;&#x6301;&#x4F7F;&#x7528;FOLLOWING&#x7684;&#x8303;&#x56F4;&#x3002;&#x5FC5;&#x987B;&#x5728;&#x5355;&#x4E2A;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html">&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#x4E0A;&#x6307;&#x5B9A;ORDER BY &#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL DISTINCT</b>&#x805A;&#x5408;&#x5B50;&#x53E5;&#xFF0C;&#x4F8B;&#x5982;<b>COUNT</b>&#xFF08;<b>DISTINCT a</b>&#xFF09;&#x3002;&#x4E0D;&#x540C;&#x805A;&#x5408;&#x58F0;&#x660E;&#x805A;&#x5408;&#x51FD;&#x6570;&#xFF08;&#x5185;&#x7F6E;&#x6216;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#xFF09;&#x4EC5;&#x5E94;&#x7528;&#x4E8E;&#x4E0D;&#x540C;&#x7684;&#x8F93;&#x5165;&#x503C;&#x3002;<b>Distinct</b>&#x53EF;&#x4EE5;&#x5E94;&#x7528;&#x4E8E;<b>GroupBy</b>&#x805A;&#x5408;&#xFF0C;<b>GroupBy</b>&#x7A97;&#x53E3;&#x805A;&#x5408;&#x548C;<b>Over Window Aggregation</b>&#x3002;</p>
        <p><b>val</b> orders<b>:</b>  <b>Table</b>  <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>// Distinct aggregation on group by</p>
        <p><b>val</b> groupByDistinctResult <b>=</b> orders</p>
        <p> <b>.</b>groupBy(&apos;a)</p>
        <p> <b>.</b>select(&apos;a, &apos;b.sum<b>.</b>distinct as &apos;d)</p>
        <p>// Distinct aggregation on time window group by</p>
        <p><b>val</b> groupByWindowDistinctResult <b>=</b> orders</p>
        <p> <b>.</b>window(<b>Tumble</b> over 5.minutes on &apos;rowtime as &apos;w)<b>.</b>groupBy(&apos;a,
          &apos;w)</p>
        <p> <b>.</b>select(&apos;a, &apos;b.sum<b>.</b>distinct as &apos;d)</p>
        <p>// Distinct aggregation on over window</p>
        <p><b>val</b> result <b>=</b> orders</p>
        <p> <b>.</b>window(<b>Over</b>
        </p>
        <p>partitionBy &apos;a</p>
        <p>orderBy &apos;rowtime</p>
        <p>preceding <b>UNBOUNDED_RANGE</b>
        </p>
        <p>as &apos;w)</p>
        <p> <b>.</b>select(&apos;a, &apos;b.avg<b>.</b>distinct over &apos;w, &apos;b.max
          over &apos;w, &apos;b.min over &apos;w)</p>
        <p>&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x805A;&#x5408;&#x51FD;&#x6570;&#x4E5F;&#x53EF;&#x4EE5;&#x4E0E;<b>DISTINCT</b>&#x4FEE;&#x9970;&#x7B26;&#x4E00;&#x8D77;&#x4F7F;&#x7528;&#x3002;&#x8981;&#x4EC5;&#x4E3A;&#x4E0D;&#x540C;&#x7684;&#x503C;&#x8BA1;&#x7B97;&#x805A;&#x5408;&#x7ED3;&#x679C;&#xFF0C;&#x53EA;&#x9700;&#x5C06;<b>distinct</b>&#x4FEE;&#x9970;&#x7B26;&#x6DFB;&#x52A0;&#x5230;&#x805A;&#x5408;&#x51FD;&#x6570;&#x5373;&#x53EF;&#x3002;</p>
        <p><b>val</b> orders<b>:</b>  <b>Table</b>  <b>=</b> tEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>// Use distinct aggregation for user-defined aggregate functions</p>
        <p><b>val</b> myUdagg <b>=</b>  <b>new</b>  <b>MyUdagg</b>();</p>
        <p>orders<b>.</b>groupBy(&apos;users)<b>.</b>select(&apos;users, myUdagg<b>.</b>distinct(&apos;points)
          as &apos;myDistinctResult);</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL DISTINCT&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8FD4;&#x56DE;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x503C;&#x7EC4;&#x5408;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;)
          <br />val result = orders.distinct()</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>GroupBy Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL GROUP BY&#x5B50;&#x53E5;&#x3002;&#x4F7F;&#x7528;&#x4EE5;&#x4E0B;&#x6B63;&#x5728;&#x8FD0;&#x884C;&#x7684;&#x805A;&#x5408;&#x8FD0;&#x7B97;&#x7B26;&#x5C06;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x7684;&#x884C;&#x5206;&#x7EC4;&#xFF0C;&#x4EE5;&#x9010;&#x884C;&#x805A;&#x5408;&#x884C;&#x3002;
          <br
          />orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.group_by(&quot;a&quot;).select(&quot;a, b.sum as d&quot;)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x805A;&#x5408;&#x7684;&#x7C7B;&#x578B;&#x548C;&#x4E0D;&#x540C;&#x7684;&#x5206;&#x7EC4;&#x952E;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>GroupBy Window Aggregation</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x5728;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#group-windows">&#x7EC4;&#x7A97;&#x53E3;</a>&#x548C;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x5BF9;&#x8868;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x96C6;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.window(Tumble.over(&quot;5.minutes&quot;).on(&quot;rowtime&quot;).alias(&quot;w&quot;))
          .group_by(&quot;a, w&quot;) .select(&quot;a, w.start, w.end, w.rowtime,
          b.sum as d&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Over Window Aggregation</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL OVER&#x5B50;&#x53E5;&#x3002;&#x57FA;&#x4E8E;&#x524D;&#x4E00;&#x884C;&#x548C;&#x540E;&#x4E00;&#x884C;&#x7684;&#x7A97;&#x53E3;&#xFF08;&#x8303;&#x56F4;&#xFF09;&#xFF0C;&#x4E3A;&#x6BCF;&#x4E00;&#x884C;&#x8BA1;&#x7B97;&#x7A97;&#x53E3;&#x805A;&#x5408;&#x3002;&#x6709;&#x5173;&#x66F4;&#x591A;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#over-windows">Windows&#x90E8;&#x5206;</a>&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.over_window(Over.partition_by(&quot;a&quot;).order_by(&quot;rowtime&quot;)
          .preceding(&quot;UNBOUNDED_RANGE&quot;).following(&quot;CURRENT_RANGE&quot;)
          .alias(&quot;w&quot;)) .select(&quot;a, b.avg over w, b.max over w, b.min
          over w&quot;)<b><br />&#x6CE8;&#x610F;&#xFF1A;</b>&#x5FC5;&#x987B;&#x5728;&#x540C;&#x4E00;&#x7A97;&#x53E3;&#x4E0A;&#x5B9A;&#x4E49;&#x6240;&#x6709;&#x805A;&#x5408;&#xFF0C;&#x5373;&#xFF0C;&#x76F8;&#x540C;&#x7684;&#x5206;&#x533A;&#xFF0C;&#x6392;&#x5E8F;&#x548C;&#x8303;&#x56F4;&#x3002;&#x5F53;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;PRECEDING&#xFF08;&#x65E0;&#x8FB9;&#x754C;&#x548C;&#x6709;&#x754C;&#xFF09;&#x5230;CURRENT
          ROW&#x8303;&#x56F4;&#x7684;&#x7A97;&#x53E3;&#x3002;&#x76EE;&#x524D;&#x5C1A;&#x4E0D;&#x652F;&#x6301;&#x5E26;&#x6709;FOLLOWING&#x7684;&#x8303;&#x56F4;&#x3002;&#x5FC5;&#x987B;&#x5728;&#x5355;&#x4E2A;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/time_attributes.html">&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#x4E0A;&#x6307;&#x5B9A;ORDER BY &#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct Aggregation</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL DISTINCT&#x805A;&#x5408;&#x5B50;&#x53E5;&#xFF0C;&#x4F8B;&#x5982;COUNT&#xFF08;DISTINCT
          a&#xFF09;&#x3002;&#x4E0D;&#x540C;&#x7684;&#x805A;&#x5408;&#x58F0;&#x660E;&#x805A;&#x5408;&#x51FD;&#x6570;&#xFF08;&#x5185;&#x7F6E;&#x6216;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#xFF09;&#x4EC5;&#x5E94;&#x7528;&#x4E8E;&#x4E0D;&#x540C;&#x7684;&#x8F93;&#x5165;&#x503C;&#x3002;&#x53EF;&#x4EE5;&#x5C06;&#x4E0D;&#x540C;&#x5E94;&#x7528;&#x4E8E;<b>GroupBy&#x805A;&#x5408;</b>&#xFF0C;<b>GroupBy&#x7A97;&#x53E3;&#x805A;&#x5408;</b>&#x548C;<b>Over Window&#x805A;&#x5408;</b>&#x3002;
          <br
          />orders = table_env.from_path(&quot;Orders&quot;)</p>
        <p>group_by_distinct_result = orders.group_by(&quot;a&quot;) .select(&quot;a,
          b.sum.distinct as d&quot;)</p>
        <p>group_by_window_distinct_result = orders.window( Tumble.over(&quot;5.minutes&quot;).on(&quot;rowtime&quot;).alias(&quot;w&quot;)).groupBy(&quot;a,
          w&quot;) .select(&quot;a, b.sum.distinct as d&quot;)</p>
        <p>result = orders.over_window(Over .partition_by(&quot;a&quot;) .order_by(&quot;rowtime&quot;)
          .preceding(&quot;UNBOUNDED_RANGE&quot;) .alias(&quot;w&quot;)) .select(
          &quot;a, b.avg.distinct over w, b.max over w, b.min over w&quot;)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Distinct</b>
        <br />Batch Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL DISTINCT&#x5B50;&#x53E5;&#x3002;&#x8FD4;&#x56DE;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x503C;&#x7EC4;&#x5408;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.distinct()</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x5B57;&#x6BB5;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 关联\(Joins\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Inner Join</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL JOIN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5173;&#x8054;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x5FC5;&#x987B;&#x901A;&#x8FC7;&#x5173;&#x8054;&#x64CD;&#x4F5C;&#x7B26;&#x6216;&#x4F7F;&#x7528;<b>where</b>&#x6216;<b>filter</b>&#x64CD;&#x4F5C;&#x7B26;&#x5B9A;&#x4E49;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x3002;</p>
        <p><code>Table left = tableEnv.fromDataSet(ds1, &quot;a, b, c&quot;);</code>
        </p>
        <p><code>Table right = tableEnv.fromDataSet(ds2, &quot;d, e, f&quot;);</code>
        </p>
        <p><code>Table result = left.join(right).where(&quot;a = d&quot;).select(&quot;a, b, e&quot;);</code>
        </p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Outer Join</b>
        <br />Batch StreamingResult Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL LEFT / RIGHT / FULL OUTER JOIN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5173;&#x8054;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x5FC5;&#x987B;&#x81F3;&#x5C11;&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x3002;</p>
        <p><code>Table left = tableEnv.fromDataSet(ds1, &quot;a, b, c&quot;);</code>
        </p>
        <p><code>Table right = tableEnv.fromDataSet(ds2, &quot;d, e, f&quot;);</code>
        </p>
        <p><code>Table leftOuterResult = left.leftOuterJoin(right, &quot;a = d&quot;).select(&quot;a, b, e&quot;);</code>
        </p>
        <p><code>Table rightOuterResult = left.rightOuterJoin(right, &quot;a = d&quot;).select(&quot;a, b, e&quot;);</code>
        </p>
        <p><code>Table fullOuterResult = left.fullOuterJoin(right, &quot;a = d&quot;).select(&quot;a, b, e&quot;);</code>
        </p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Time-windowed Join<br />Batch Streaming</b>
      </td>
      <td style="text-align:left">
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x662F;&#x53EF;&#x4EE5;&#x4EE5;&#x6D41;&#x65B9;&#x5F0F;&#x5904;&#x7406;&#x7684;&#x5E38;&#x89C4;&#x5173;&#x8054;&#x7684;&#x5B50;&#x96C6;&#x3002;</p>
        <p>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x9700;&#x8981;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x548C;&#x4E00;&#x4E2A;&#x9650;&#x5236;&#x53CC;&#x65B9;&#x65F6;&#x95F4;&#x7684;&#x5173;&#x8054;&#x6761;&#x4EF6;&#x3002;
          &#x8FD9;&#x6837;&#x7684;&#x6761;&#x4EF6;&#x53EF;&#x4EE5;&#x7531;&#x4E24;&#x4E2A;&#x9002;&#x5F53;&#x7684;&#x8303;&#x56F4;&#x8C13;&#x8BCD;&#xFF08;<b>&lt;</b>&#xFF0C;<b>&lt;=</b>&#xFF0C;<b>&gt; =</b>&#xFF0C;<b>&gt;</b>&#xFF09;&#x6216;&#x5355;&#x4E2A;&#x7B49;&#x5F0F;&#x8C13;&#x8BCD;&#x6765;&#x5B9A;&#x4E49;&#xFF0C;&#x8BE5;&#x5355;&#x4E2A;&#x7B49;&#x5F0F;&#x8C13;&#x8BCD;&#x6BD4;&#x8F83;&#x4E24;&#x4E2A;&#x8F93;&#x5165;&#x8868;&#x7684;&#x76F8;&#x540C;&#x7C7B;&#x578B;&#x7684;&#x65F6;&#x95F4;&#x5C5E;&#x6027;&#xFF08;&#x5373;&#xFF0C;&#x5904;&#x7406;&#x65F6;&#x95F4;&#x6216;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x4EE5;&#x4E0B;&#x8C13;&#x8BCD;&#x662F;&#x6709;&#x6548;&#x7684;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x6761;&#x4EF6;&#xFF1A;</p>
        <ul>
          <li>&apos;ltime === &apos;rtime</li>
          <li>&apos;ltime &gt;= &apos;rtime &amp;&amp; &apos;ltime &lt; &apos;rtime
            + 10.minutes</li>
        </ul>
        <p><code>Table left = tableEnv.fromDataSet(ds1, &quot;a, b, c, ltime.rowtime&quot;);</code>
        </p>
        <p><code>Table right = tableEnv.fromDataSet(ds2, &quot;d, e, f, rtime.rowtime&quot;);</code>
        </p>
        <p><code>Table result = left.join(right)</code>
        </p>
        <p><code>  .where(&quot;a = d &amp;&amp; ltime &gt;= rtime - 5.minutes &amp;&amp; ltime &lt; rtime + 10.minutes&quot;)</code>
        </p>
        <p><code>  .select(&quot;a, b, e, ltime&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Inner Join with Table Function</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x8FDE;&#x63A5;&#x8868;&#x3002;
          &#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x8FDE;&#x63A5;&#x3002;
          &#x5982;&#x679C;&#x5176;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x5220;&#x9664;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x4E00;&#x884C;&#x3002;</p>
        <p><code>// register User-Defined Table Function</code>
        </p>
        <p><code>TableFunction&lt;String&gt; split = new MySplitUDTF();</code>
        </p>
        <p><code>tableEnv.registerFunction(&quot;split&quot;, split);</code>
        </p>
        <p><code>// join</code>
        </p>
        <p><code>Table orders = tableEnv.scan(&quot;Orders&quot;);</code>
        </p>
        <p><code>Table result = orders</code>
        </p>
        <p><code>    .join(new Table(tableEnv, &quot;split(c)&quot;).as(&quot;s&quot;, &quot;t&quot;, &quot;v&quot;))</code>
        </p>
        <p><code>    .select(&quot;a, b, s, t, v&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Left Outer Join with Table Function</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x5173;&#x8054;&#x8868;&#x3002;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x5173;&#x8054;&#x3002;&#x5982;&#x679C;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x4FDD;&#x7559;&#x76F8;&#x5E94;&#x7684;&#x5916;&#x90E8;&#x884C;&#xFF0C;&#x5E76;&#x4F7F;&#x7528;&#x7A7A;&#x503C;&#x586B;&#x5145;&#x7ED3;&#x679C;&#x3002;</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x76EE;&#x524D;&#xFF0C;&#x5DE6;&#x5916;&#x5173;&#x8054;&#x7684;&#x8868;&#x51FD;&#x6570;&#x7684;&#x8C13;&#x8BCD;&#x53EA;&#x80FD;&#x662F;&#x7A7A;&#x7684;&#x6216;&#x6587;&#x5B57;&#x7684;<b>true</b>&#x3002;</p>
        <p><code>// register User-Defined Table Function</code>
        </p>
        <p><code>TableFunction&lt;String&gt; split = new MySplitUDTF();</code>
        </p>
        <p><code>tableEnv.registerFunction(&quot;split&quot;, split);</code>
        </p>
        <p><code>// join</code>
        </p>
        <p><code>Table orders = tableEnv.scan(&quot;Orders&quot;);</code>
        </p>
        <p><code>Table result = orders</code>
        </p>
        <p><code>    .leftOuterJoin(new Table(tableEnv, &quot;split(c)&quot;).as(&quot;s&quot;, &quot;t&quot;, &quot;v&quot;))</code>
        </p>
        <p><code>    .select(&quot;a, b, s, t, v&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Join with Temporal Table</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html">&#x65F6;&#x6001;&#x8868;</a>&#x662F;&#x8DDF;&#x8E2A;&#x5176;&#x968F;&#x65F6;&#x95F4;&#x53D8;&#x5316;&#x7684;&#x8868;&#x3002;</p>
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html#temporal-table-functions">&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;</a>&#x63D0;&#x4F9B;&#x5BF9;&#x7279;&#x5B9A;&#x65F6;&#x95F4;&#x70B9;&#x7684;&#x65F6;&#x6001;&#x8868;&#x7684;&#x72B6;&#x6001;&#x7684;&#x8BBF;&#x95EE;&#x3002;&#x4F7F;&#x7528;&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;&#x5173;&#x8054;&#x8868;&#x7684;&#x8BED;&#x6CD5;&#x4E0E;&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x5185;&#x90E8;&#x5173;&#x8054;&#x76F8;&#x540C;&#x3002;</p>
        <p>&#x76EE;&#x524D;&#x4EC5;&#x652F;&#x6301;&#x5177;&#x6709;&#x65F6;&#x6001;&#x8868;&#x7684;&#x5185;&#x90E8;&#x5173;&#x8054;&#x3002;</p>
        <p><code>Table ratesHistory = tableEnv.scan(&quot;RatesHistory&quot;);</code>
        </p>
        <p><code>// register temporal table function with a time attribute and primary key</code>
        </p>
        <p><code>TemporalTableFunction rates = ratesHistory.createTemporalTableFunction(</code>
        </p>
        <p><code>    &quot;r_proctime&quot;,</code>
        </p>
        <p><code>    &quot;r_currency&quot;);</code>
        </p>
        <p><code>tableEnv.registerFunction(&quot;rates&quot;, rates);</code>
        </p>
        <p><code>// join with &quot;Orders&quot; based on the time attribute and key</code>
        </p>
        <p><code>Table orders = tableEnv.scan(&quot;Orders&quot;);</code>
        </p>
        <p><code>Table result = orders</code>
        </p>
        <p><code>    .join(new Table(tEnv, &quot;rates(o_proctime)&quot;), &quot;o_currency = r_currency&quot;)</code>
        </p>
        <p>&#x6709;&#x5173;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x67E5;&#x770B;&#x66F4;&#x8BE6;&#x7EC6;&#x7684;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html">&#x65F6;&#x6001;&#x8868;&#x6982;&#x5FF5;&#x63CF;&#x8FF0;</a>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Inner Join</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL JOIN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5173;&#x8054;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x5FC5;&#x987B;&#x901A;&#x8FC7;&#x5173;&#x8054;&#x64CD;&#x4F5C;&#x7B26;&#x6216;&#x4F7F;&#x7528;<b>where</b>&#x6216;<b>filter</b>&#x64CD;&#x4F5C;&#x7B26;&#x5B9A;&#x4E49;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;d, &apos;e,
          &apos;f)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>join(right)<b>.</b>where(&apos;a <b>===</b> &apos;d)<b>.</b>select(&apos;a,
          &apos;b, &apos;e)</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Outer Join</b>
        <br />Batch StreamingResult Updating</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL LEFT / RIGHT / FULL OUTER JOIN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5173;&#x8054;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x5FC5;&#x987B;&#x81F3;&#x5C11;&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x3002;</p>
        <p><b>val</b> left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &apos;a, &apos;b,
          &apos;c)</p>
        <p><b>val</b> right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &apos;d, &apos;e,
          &apos;f)</p>
        <p><b>val</b> leftOuterResult <b>=</b> left<b>.</b>leftOuterJoin(right, &apos;a <b>===</b> &apos;d)<b>.</b>select(&apos;a,
          &apos;b, &apos;e)</p>
        <p><b>val</b> rightOuterResult <b>=</b> left<b>.</b>rightOuterJoin(right, &apos;a <b>===</b> &apos;d)<b>.</b>select(&apos;a,
          &apos;b, &apos;e)</p>
        <p><b>val</b> fullOuterResult <b>=</b> left<b>.</b>fullOuterJoin(right, &apos;a <b>===</b> &apos;d)<b>.</b>select(&apos;a,
          &apos;b, &apos;e)</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Time-windowed Join<br />Batch Streaming</b>
      </td>
      <td style="text-align:left">
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x662F;&#x53EF;&#x4EE5;&#x4EE5;&#x6D41;&#x65B9;&#x5F0F;&#x5904;&#x7406;&#x7684;&#x5E38;&#x89C4;&#x5173;&#x8054;&#x7684;&#x5B50;&#x96C6;&#x3002;</p>
        <p>&#x65F6;&#x95F4;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x9700;&#x8981;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x548C;&#x4E00;&#x4E2A;&#x9650;&#x5236;&#x53CC;&#x65B9;&#x65F6;&#x95F4;&#x7684;&#x5173;&#x8054;&#x6761;&#x4EF6;&#x3002;
          &#x8FD9;&#x6837;&#x7684;&#x6761;&#x4EF6;&#x53EF;&#x4EE5;&#x7531;&#x4E24;&#x4E2A;&#x9002;&#x5F53;&#x7684;&#x8303;&#x56F4;&#x8C13;&#x8BCD;&#xFF08;<b>&lt;</b>&#xFF0C;<b>&lt;=</b>&#xFF0C;<b>&gt; =</b>&#xFF0C;<b>&gt;</b>&#xFF09;&#x6216;&#x5355;&#x4E2A;&#x7B49;&#x5F0F;&#x8C13;&#x8BCD;&#x6765;&#x5B9A;&#x4E49;&#xFF0C;&#x8BE5;&#x5355;&#x4E2A;&#x7B49;&#x5F0F;&#x8C13;&#x8BCD;&#x6BD4;&#x8F83;&#x4E24;&#x4E2A;&#x8F93;&#x5165;&#x8868;&#x7684;&#x76F8;&#x540C;&#x7C7B;&#x578B;&#x7684;&#x65F6;&#x95F4;&#x5C5E;&#x6027;&#xFF08;&#x5373;&#xFF0C;&#x5904;&#x7406;&#x65F6;&#x95F4;&#x6216;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#xFF09;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x4EE5;&#x4E0B;&#x8C13;&#x8BCD;&#x662F;&#x6709;&#x6548;&#x7684;&#x7A97;&#x53E3;&#x5173;&#x8054;&#x6761;&#x4EF6;&#xFF1A;</p>
        <ul>
          <li>&apos;ltime === &apos;rtime</li>
          <li>&apos;ltime &gt;= &apos;rtime &amp;&amp; &apos;ltime &lt; &apos;rtime
            + 10.minutes</li>
        </ul>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c,
          &apos;ltime.rowtime)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;d, &apos;e,
          &apos;f, &apos;rtime.rowtime)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>join(right)</p>
        <p> <b>.</b>where(&apos;a <b>===</b> &apos;d <b>&amp;&amp;</b> &apos;ltime <b>&gt;=</b> &apos;rtime <b>-</b> 5.minutes <b>&amp;&amp;</b> &apos;ltime <b>&lt;</b> &apos;rtime <b>+</b> 10.minutes)</p>
        <p> <b>.</b>select(&apos;a, &apos;b, &apos;e, &apos;ltime)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Inner Join with Table Function</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x8FDE;&#x63A5;&#x8868;&#x3002;
          &#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x8FDE;&#x63A5;&#x3002;
          &#x5982;&#x679C;&#x5176;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x5220;&#x9664;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x4E00;&#x884C;&#x3002;</p>
        <p>// instantiate User-Defined Table Function</p>
        <p><b>val</b> split<b>:</b>  <b>TableFunction</b>[<b>_</b>] <b>=</b>  <b>new</b>  <b>MySplitUDTF</b>()</p>
        <p>// join</p>
        <p><b>val</b> result<b>:</b>  <b>Table</b>  <b>=</b> table</p>
        <p> <b>.</b>join(split(&apos;c) as (&apos;s, &apos;t, &apos;v))</p>
        <p> <b>.</b>select(&apos;a, &apos;b, &apos;s, &apos;t, &apos;v)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Left Outer Join with Table Function</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x5173;&#x8054;&#x8868;&#x3002;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x5173;&#x8054;&#x3002;&#x5982;&#x679C;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x4FDD;&#x7559;&#x76F8;&#x5E94;&#x7684;&#x5916;&#x90E8;&#x884C;&#xFF0C;&#x5E76;&#x4F7F;&#x7528;&#x7A7A;&#x503C;&#x586B;&#x5145;&#x7ED3;&#x679C;&#x3002;</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x76EE;&#x524D;&#xFF0C;&#x5DE6;&#x5916;&#x5173;&#x8054;&#x7684;&#x8868;&#x51FD;&#x6570;&#x7684;&#x8C13;&#x8BCD;&#x53EA;&#x80FD;&#x662F;&#x7A7A;&#x7684;&#x6216;&#x6587;&#x5B57;&#x7684;<b>true</b>&#x3002;</p>
        <p>// instantiate User-Defined Table Function</p>
        <p><b>val</b> split<b>:</b>  <b>TableFunction</b>[<b>_</b>] <b>=</b>  <b>new</b>  <b>MySplitUDTF</b>()</p>
        <p>// join</p>
        <p><b>val</b> result<b>:</b>  <b>Table</b>  <b>=</b> table</p>
        <p> <b>.</b>leftOuterJoin(split(&apos;c) as (&apos;s, &apos;t, &apos;v))</p>
        <p> <b>.</b>select(&apos;a, &apos;b, &apos;s, &apos;t, &apos;v)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Join with Temporal Table</b>
        <br />Streaming</td>
      <td style="text-align:left">
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html">&#x65F6;&#x6001;&#x8868;</a>&#x662F;&#x8DDF;&#x8E2A;&#x5176;&#x968F;&#x65F6;&#x95F4;&#x53D8;&#x5316;&#x7684;&#x8868;&#x3002;</p>
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html#temporal-table-functions">&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;</a>&#x63D0;&#x4F9B;&#x5BF9;&#x7279;&#x5B9A;&#x65F6;&#x95F4;&#x70B9;&#x7684;&#x65F6;&#x6001;&#x8868;&#x7684;&#x72B6;&#x6001;&#x7684;&#x8BBF;&#x95EE;&#x3002;&#x4F7F;&#x7528;&#x65F6;&#x6001;&#x8868;&#x51FD;&#x6570;&#x5173;&#x8054;&#x8868;&#x7684;&#x8BED;&#x6CD5;&#x4E0E;&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x5185;&#x90E8;&#x5173;&#x8054;&#x76F8;&#x540C;&#x3002;</p>
        <p>&#x76EE;&#x524D;&#x4EC5;&#x652F;&#x6301;&#x5177;&#x6709;&#x65F6;&#x6001;&#x8868;&#x7684;&#x5185;&#x90E8;&#x5173;&#x8054;&#x3002;</p>
        <p><b>val</b> ratesHistory <b>=</b> tableEnv<b>.</b>scan(&quot;RatesHistory&quot;)</p>
        <p>// register temporal table function with a time attribute and primary
          key</p>
        <p><b>val</b> rates <b>=</b> ratesHistory<b>.</b>createTemporalTableFunction(&apos;r_proctime,
          &apos;r_currency)</p>
        <p>// join with &quot;Orders&quot; based on the time attribute and key</p>
        <p><b>val</b> orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;)</p>
        <p><b>val</b> result <b>=</b> orders</p>
        <p> <b>.</b>join(rates(&apos;o_rowtime), &apos;r_currency <b>===</b> &apos;o_currency)</p>
        <p>&#x6709;&#x5173;&#x66F4;&#x591A;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x67E5;&#x770B;&#x66F4;&#x8BE6;&#x7EC6;&#x7684;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/temporal_tables.html">&#x65F6;&#x6001;&#x8868;&#x6982;&#x5FF5;&#x63CF;&#x8FF0;</a>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Inner Join</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL JOIN&#x5B50;&#x53E5;&#x3002;&#x8FDE;&#x63A5;&#x4E24;&#x4E2A;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x81F3;&#x5C11;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x7684;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x5FC5;&#x987B;&#x901A;&#x8FC7;&#x5173;&#x8054;&#x8FD0;&#x7B97;&#x7B26;&#x6216;&#x4F7F;&#x7528;where&#x6216;filter&#x8FD0;&#x7B97;&#x7B26;&#x8FDB;&#x884C;&#x5B9A;&#x4E49;&#x3002;
          <br
          />left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;d, e, f&quot;)
          result = left.join(right).where(&quot;a = d&quot;).select(&quot;a, b, e&quot;)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Outer Join</b>
        <br />Batch StreamingResult Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL LEFT / RIGHT / FULL OUTER JOIN&#x5B50;&#x53E5;&#x3002;&#x5173;&#x8054;&#x4E24;&#x4E2A;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x4E0D;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x540D;&#x79F0;&#xFF0C;&#x5E76;&#x4E14;&#x5FC5;&#x987B;&#x81F3;&#x5C11;&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x76F8;&#x7B49;&#x5173;&#x8054;&#x8C13;&#x8BCD;&#x3002;
          <br
          />left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;d, e, f&quot;)
          <br
          />
        </p>
        <p>left_outer_result = left.left_outer_join(right, &quot;a = d&quot;).select(&quot;a,
          b, e&quot;)
          <br />right_outer_result = left.right_outer_join(right, &quot;a = d&quot;).select(&quot;a,
          b, e&quot;)
          <br />full_outer_result = left.full_outer_join(right, &quot;a = d&quot;).select(&quot;a,
          b, e&quot;)</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Time-windowed Join<br />Batch Streaming</b>
      </td>
      <td style="text-align:left">Python API&#x5F53;&#x524D;&#x4E0D;&#x652F;&#x6301;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Inner Join with Table Function (UDTF)</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x5173;&#x8054;&#x8868;&#x3002;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x90FD;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x8FDE;&#x63A5;&#x5728;&#x4E00;&#x8D77;&#x3002;&#x5982;&#x679C;&#x5DE6;&#x8868;&#xFF08;&#x5916;&#x90E8;&#xFF09;&#x7684;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x8BE5;&#x884C;&#x5C06;&#x88AB;&#x5220;&#x9664;&#x3002;</p>
        <p>table_env.register_java_function(&quot;split&quot;, &quot;com.my.udf.MySplitUDTF&quot;)</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.join_lateral(&quot;split(c).as(s, t, v)&quot;).select(&quot;a,
          b, s, t, v&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Left Outer Join with Table Function (UDTF)</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7528;&#x8868;&#x51FD;&#x6570;&#x7684;&#x7ED3;&#x679C;&#x5173;&#x8054;&#x8868;&#x3002;&#x5DE6;&#xFF08;&#x5916;&#xFF09;&#x8868;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#x90FD;&#x4E0E;&#x8868;&#x51FD;&#x6570;&#x7684;&#x76F8;&#x5E94;&#x8C03;&#x7528;&#x4EA7;&#x751F;&#x7684;&#x6240;&#x6709;&#x884C;&#x8FDE;&#x63A5;&#x5728;&#x4E00;&#x8D77;&#x3002;&#x5982;&#x679C;&#x8868;&#x51FD;&#x6570;&#x8C03;&#x7528;&#x8FD4;&#x56DE;&#x7A7A;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x5C06;&#x4FDD;&#x7559;&#x5BF9;&#x5E94;&#x7684;&#x5916;&#x90E8;&#x884C;&#xFF0C;&#x5E76;&#x7528;&#x7A7A;&#x503C;&#x586B;&#x5145;&#x7ED3;&#x679C;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5F53;&#x524D;&#xFF0C;&#x5DE6;&#x5916;&#x90E8;&#x5173;&#x8054;&#x7684;&#x8868;&#x51FD;&#x6570;&#x7684;&#x8C13;&#x8BCD;&#x53EA;&#x80FD;&#x4E3A;&#x7A7A;&#x6216;&#x6587;&#x5B57;<code>true</code>&#x3002;</p>
        <p>table_env.register_java_function(&quot;split&quot;, &quot;com.my.udf.MySplitUDTF&quot;)</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;)
          <br />result = orders.left_outer_join_lateral(&quot;split(c).as(s, t, v)&quot;).select(&quot;a,
          b, s, t, v&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Join with Temporal Table</b>
        <br />Streaming</td>
      <td style="text-align:left">Python API&#x5F53;&#x524D;&#x4E0D;&#x652F;&#x6301;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### Set操作\(Set Operations\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL UNION</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8054;&#x5408;&#x4E24;&#x4E2A;&#x8868;&#x5220;&#x9664;&#x4E86;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;a, b, c&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>union(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>UnionAll</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL UNION ALL</b>&#x5B50;&#x53E5;&#x3002;&#x8054;&#x5408;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;a, b, c&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>unionAll(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Intersect</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT</b>&#x5B50;&#x53E5;&#x3002;<b>Intersect</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E00;&#x4E2A;&#x6216;&#x4E24;&#x4E2A;&#x8868;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#x8BB0;&#x5F55;&#xFF0C;&#x5219;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x7ED3;&#x679C;&#x8868;&#x6CA1;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;d, e, f&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>intersect(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>IntersectAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>IntersectAll</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x7684;&#x503C;&#x4E0E;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E00;&#x6837;&#x591A;&#xFF0C;&#x5373;&#x751F;&#x6210;&#x7684;&#x8868;&#x53EF;&#x80FD;&#x5177;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;d, e, f&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>intersectAll(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Minus</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL EXCEPT</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;<b>Minus</b>&#x8FD4;&#x56DE;&#x5DE6;&#x8868;&#x4E2D;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5DE6;&#x8868;&#x4E2D;&#x7684;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x5220;&#x9664;&#x91CD;&#x590D;&#x9879;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;a, b, c&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>minus(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MinusAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL EXCEPT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>MinusAll</b>&#x8FD4;&#x56DE;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5728;&#x5DE6;&#x8868;&#x4E2D;&#x51FA;&#x73B0;n&#x6B21;&#x5E76;&#x5728;&#x53F3;&#x8868;&#x4E2D;&#x51FA;&#x73B0;m&#x6B21;&#x7684;&#x8BB0;&#x5F55;&#x8FD4;&#x56DE;&#xFF08;n-m&#xFF09;&#x6B21;&#xFF0C;&#x5373;&#xFF0C;&#x5220;&#x9664;&#x53F3;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x91CD;&#x590D;&#x6B21;&#x6570;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> tableEnv<b>.</b>fromDataSet(ds1, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> tableEnv<b>.</b>fromDataSet(ds2, &quot;a, b, c&quot;);</p>
        <p>Table result <b>=</b> left<b>.</b>minusAll(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>In</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL IN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5982;&#x679C;&#x8868;&#x8FBE;&#x5F0F;&#x5B58;&#x5728;&#x4E8E;&#x7ED9;&#x5B9A;&#x7684;&#x8868;&#x5B50;&#x67E5;&#x8BE2;&#x4E2D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;true&#x3002;&#x5B50;&#x67E5;&#x8BE2;&#x8868;&#x5FC5;&#x987B;&#x5305;&#x542B;&#x4E00;&#x5217;&#x3002;&#x6B64;&#x5217;&#x5FC5;&#x987B;&#x4E0E;&#x8868;&#x8FBE;&#x5F0F;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x6570;&#x636E;&#x7C7B;&#x578B;&#x3002;</p>
        <p>Table left <b>=</b> ds1<b>.</b>toTable(tableEnv, &quot;a, b, c&quot;);</p>
        <p>Table right <b>=</b> ds2<b>.</b>toTable(tableEnv, &quot;a&quot;);</p>
        <p>// using implicit registration</p>
        <p>Table result <b>=</b> left<b>.</b>select(&quot;a, b, c&quot;)<b>.</b>where(&quot;a.in(&quot; <b>+</b> right <b>+</b> &quot;)&quot;);</p>
        <p>// using explicit registration</p>
        <p>tableEnv<b>.</b>registerTable(&quot;RightTable&quot;, right);</p>
        <p>Table result <b>=</b> left<b>.</b>select(&quot;a, b, c&quot;)<b>.</b>where(&quot;a.in(RightTable)&quot;);</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x8FDE;&#x63A5;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL UNION</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8054;&#x5408;&#x4E24;&#x4E2A;&#x8868;&#x5220;&#x9664;&#x4E86;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;a, &apos;b,
          &apos;c)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>union(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>UnionAll</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL UNION ALL</b>&#x5B50;&#x53E5;&#x3002;&#x8054;&#x5408;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;a, &apos;b,
          &apos;c)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>unionAll(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Intersect</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT</b>&#x5B50;&#x53E5;&#x3002;<b>Intersect</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E00;&#x4E2A;&#x6216;&#x4E24;&#x4E2A;&#x8868;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#x8BB0;&#x5F55;&#xFF0C;&#x5219;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x7ED3;&#x679C;&#x8868;&#x6CA1;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;e, &apos;f,
          &apos;g)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>intersect(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>IntersectAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>IntersectAll</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x7684;&#x503C;&#x4E0E;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E00;&#x6837;&#x591A;&#xFF0C;&#x5373;&#x751F;&#x6210;&#x7684;&#x8868;&#x53EF;&#x80FD;&#x5177;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;e, &apos;f,
          &apos;g)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>intersectAll(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Minus</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL EXCEPT</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;<b>Minus</b>&#x8FD4;&#x56DE;&#x5DE6;&#x8868;&#x4E2D;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5DE6;&#x8868;&#x4E2D;&#x7684;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x5220;&#x9664;&#x91CD;&#x590D;&#x9879;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;a, &apos;b,
          &apos;c)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>minus(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MinusAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL EXCEPT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>MinusAll</b>&#x8FD4;&#x56DE;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5728;&#x5DE6;&#x8868;&#x4E2D;&#x51FA;&#x73B0;n&#x6B21;&#x5E76;&#x5728;&#x53F3;&#x8868;&#x4E2D;&#x51FA;&#x73B0;m&#x6B21;&#x7684;&#x8BB0;&#x5F55;&#x8FD4;&#x56DE;&#xFF08;n-m&#xFF09;&#x6B21;&#xFF0C;&#x5373;&#xFF0C;&#x5220;&#x9664;&#x53F3;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x91CD;&#x590D;&#x6B21;&#x6570;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;a, &apos;b,
          &apos;c)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>minusAll(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>In</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL IN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5982;&#x679C;&#x8868;&#x8FBE;&#x5F0F;&#x5B58;&#x5728;&#x4E8E;&#x7ED9;&#x5B9A;&#x7684;&#x8868;&#x5B50;&#x67E5;&#x8BE2;&#x4E2D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;true&#x3002;&#x5B50;&#x67E5;&#x8BE2;&#x8868;&#x5FC5;&#x987B;&#x5305;&#x542B;&#x4E00;&#x5217;&#x3002;&#x6B64;&#x5217;&#x5FC5;&#x987B;&#x4E0E;&#x8868;&#x8FBE;&#x5F0F;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x6570;&#x636E;&#x7C7B;&#x578B;&#x3002;</p>
        <p><b>val</b> left <b>=</b> ds1<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> right <b>=</b> ds2<b>.</b>toTable(tableEnv, &apos;a)</p>
        <p><b>val</b> result <b>=</b> left<b>.</b>select(&apos;a, &apos;b, &apos;c)<b>.</b>where(&apos;a.in(right))</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x8FDE;&#x63A5;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL UNION</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8054;&#x5408;&#x4E24;&#x4E2A;&#x8868;&#x5220;&#x9664;&#x4E86;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.union(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>UnionAll</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL UNION ALL</b>&#x5B50;&#x53E5;&#x3002;&#x8054;&#x5408;&#x4E24;&#x5F20;&#x8868;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.union_all(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Intersect</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT</b>&#x5B50;&#x53E5;&#x3002;<b>Intersect</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E00;&#x4E2A;&#x6216;&#x4E24;&#x4E2A;&#x8868;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#x8BB0;&#x5F55;&#xFF0C;&#x5219;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x7ED3;&#x679C;&#x8868;&#x6CA1;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.intersect(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>IntersectAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL INTERSECT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>IntersectAll</b>&#x8FD4;&#x56DE;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E0D;&#x6B62;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x7684;&#x503C;&#x4E0E;&#x4E24;&#x4E2A;&#x8868;&#x4E2D;&#x7684;&#x8BB0;&#x5F55;&#x4E00;&#x6837;&#x591A;&#xFF0C;&#x5373;&#x751F;&#x6210;&#x7684;&#x8868;&#x53EF;&#x80FD;&#x5177;&#x6709;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.intersect_all(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Minus</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL EXCEPT</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;<b>Minus</b>&#x8FD4;&#x56DE;&#x5DE6;&#x8868;&#x4E2D;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5DE6;&#x8868;&#x4E2D;&#x7684;&#x91CD;&#x590D;&#x8BB0;&#x5F55;&#x53EA;&#x8FD4;&#x56DE;&#x4E00;&#x6B21;&#xFF0C;&#x5373;&#x5220;&#x9664;&#x91CD;&#x590D;&#x9879;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.minus(right);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>MinusAll</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>SQL EXCEPT ALL</b>&#x5B50;&#x53E5;&#x3002;<b>MinusAll</b>&#x8FD4;&#x56DE;&#x53F3;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;&#x7684;&#x8BB0;&#x5F55;&#x3002;&#x5728;&#x5DE6;&#x8868;&#x4E2D;&#x51FA;&#x73B0;n&#x6B21;&#x5E76;&#x5728;&#x53F3;&#x8868;&#x4E2D;&#x51FA;&#x73B0;m&#x6B21;&#x7684;&#x8BB0;&#x5F55;&#x8FD4;&#x56DE;&#xFF08;n-m&#xFF09;&#x6B21;&#xFF0C;&#x5373;&#xFF0C;&#x5220;&#x9664;&#x53F3;&#x8868;&#x4E2D;&#x5B58;&#x5728;&#x7684;&#x91CD;&#x590D;&#x6B21;&#x6570;&#x3002;&#x4E24;&#x4E2A;&#x8868;&#x5FC5;&#x987B;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x5B57;&#x6BB5;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = left.minus_all(right)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>In</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL IN</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x5982;&#x679C;&#x8868;&#x8FBE;&#x5F0F;&#x5B58;&#x5728;&#x4E8E;&#x7ED9;&#x5B9A;&#x7684;&#x8868;&#x5B50;&#x67E5;&#x8BE2;&#x4E2D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;true&#x3002;&#x5B50;&#x67E5;&#x8BE2;&#x8868;&#x5FC5;&#x987B;&#x5305;&#x542B;&#x4E00;&#x5217;&#x3002;&#x6B64;&#x5217;&#x5FC5;&#x987B;&#x4E0E;&#x8868;&#x8FBE;&#x5F0F;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x6570;&#x636E;&#x7C7B;&#x578B;&#x3002;</p>
        <p>left = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />right = table_env.from_path(&quot;Source2&quot;).select(&quot;a&quot;)</p>
        <p>
          <br />#&#x4F7F;&#x7528;&#x9690;&#x5F0F;&#x6CE8;&#x518C;</p>
        <p>result = left.select(&quot;a, b, c&quot;).where(&quot;a.in(%s)&quot; %
          right)</p>
        <p></p>
        <p>#&#x4F7F;&#x7528;&#x663E;&#x5F0F;&#x6CE8;&#x518C;</p>
        <p>table_env.register_table(&quot;RightTable&quot;, right)
          <br />result = left.select(&quot;a, b, c&quot;).where(&quot;a.in(RightTable)&quot;)</p>
        <p>&#x6CE8;&#x610F;&#xFF1A;&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x8FDE;&#x63A5;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 排序，偏移&获取\(OrderBy，Offset＆Fetch\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Order By</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL ORDER BY</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8FD4;&#x56DE;&#x8DE8;&#x6240;&#x6709;&#x5E76;&#x884C;&#x5206;&#x533A;&#x5168;&#x5C40;&#x6392;&#x5E8F;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p>Table in <b>=</b> tableEnv<b>.</b>fromDataSet(ds, &quot;a, b, c&quot;);</p>
        <p>Table result <b>=</b> in<b>.</b>orderBy(&quot;a.asc&quot;);</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Offset &amp; Fetch</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL OFFSET&#x548C;FETCH&#x5B50;&#x53E5;&#x3002;&#x504F;&#x79FB;&#x548C;&#x63D0;&#x53D6;&#x9650;&#x5236;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x8FD4;&#x56DE;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;Offset&#x548C;Fetch&#x5728;&#x6280;&#x672F;&#x4E0A;&#x662F;Order
          By&#x8FD0;&#x7B97;&#x7B26;&#x7684;&#x4E00;&#x90E8;&#x5206;&#xFF0C;&#x56E0;&#x6B64;&#x5FC5;&#x987B;&#x5728;&#x5B83;&#x4E4B;&#x524D;&#x3002;</p>
        <p>Table in <b>=</b> tableEnv<b>.</b>fromDataSet(ds, &quot;a, b, c&quot;);</p>
        <p>// &#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x7684;&#x524D;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p>Table result1 <b>=</b> in<b>.</b>orderBy(&quot;a.asc&quot;)<b>.</b>fetch(5);</p>
        <p>// &#x8DF3;&#x8FC7;&#x524D;3&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x8BB0;&#x5F55;</p>
        <p>Table result2 <b>=</b> in<b>.</b>orderBy(&quot;a.asc&quot;)<b>.</b>offset(3);</p>
        <p>// &#x8DF3;&#x8FC7;&#x524D;10&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x8FD4;&#x56DE;&#x540E;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p>Table result3 <b>=</b> in<b>.</b>orderBy(&quot;a.asc&quot;)<b>.</b>offset(10)<b>.</b>fetch(5);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Order By</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;<b>SQL ORDER BY</b>&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x8FD4;&#x56DE;&#x8DE8;&#x6240;&#x6709;&#x5E76;&#x884C;&#x5206;&#x533A;&#x5168;&#x5C40;&#x6392;&#x5E8F;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p><b>val</b> in <b>=</b> ds<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p><b>val</b> result <b>=</b> in<b>.</b>orderBy(&apos;a.asc)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Offset &amp; Fetch</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL OFFSET&#x548C;FETCH&#x5B50;&#x53E5;&#x3002;&#x504F;&#x79FB;&#x548C;&#x63D0;&#x53D6;&#x9650;&#x5236;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x8FD4;&#x56DE;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;Offset&#x548C;Fetch&#x5728;&#x6280;&#x672F;&#x4E0A;&#x662F;Order
          By&#x8FD0;&#x7B97;&#x7B26;&#x7684;&#x4E00;&#x90E8;&#x5206;&#xFF0C;&#x56E0;&#x6B64;&#x5FC5;&#x987B;&#x5728;&#x5B83;&#x4E4B;&#x524D;&#x3002;</p>
        <p><b>val</b> in <b>=</b> ds<b>.</b>toTable(tableEnv, &apos;a, &apos;b, &apos;c)</p>
        <p>// &#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x7684;&#x524D;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p><b>val</b> result1<b>:</b>  <b>Table</b>  <b>=</b> in<b>.</b>orderBy(&apos;a.asc)<b>.</b>fetch(5)</p>
        <p>// &#x8DF3;&#x8FC7;&#x524D;3&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x8BB0;&#x5F55;</p>
        <p><b>val</b> result2<b>:</b>  <b>Table</b>  <b>=</b> in<b>.</b>orderBy(&apos;a.asc)<b>.</b>offset(3)</p>
        <p>//&#x8DF3;&#x8FC7;&#x524D;10&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x8FD4;&#x56DE;&#x540E;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p><b>val</b> result3<b>:</b>  <b>Table</b>  <b>=</b> in<b>.</b>orderBy(&apos;a.asc)<b>.</b>offset(10)<b>.</b>fetch(5)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Order By</b>
        <br />Batch
        <br />
      </td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL ORDER BY&#x5B50;&#x53E5;&#x3002;&#x8FD4;&#x56DE;&#x5728;&#x6240;&#x6709;&#x5E76;&#x884C;&#x5206;&#x533A;&#x4E0A;&#x5168;&#x5C40;&#x6392;&#x5E8F;&#x7684;&#x8BB0;&#x5F55;&#x3002;</p>
        <p>in = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />result = in.order_by(&quot;a.asc&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Offset &amp; Fetch</b>
        <br />Batch</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL OFFSET&#x548C;FETCH&#x5B50;&#x53E5;&#x7C7B;&#x4F3C;&#x3002;&#x504F;&#x79FB;&#x91CF;&#x548C;&#x63D0;&#x53D6;&#x9650;&#x5236;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x8FD4;&#x56DE;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;&#x504F;&#x79FB;&#x548C;&#x63D0;&#x53D6;&#x5728;&#x6280;&#x672F;&#x4E0A;&#x662F;Order
          By&#x8FD0;&#x7B97;&#x7B26;&#x7684;&#x4E00;&#x90E8;&#x5206;&#xFF0C;&#x56E0;&#x6B64;&#x5FC5;&#x987B;&#x5728;&#x5176;&#x4E4B;&#x524D;&#x3002;</p>
        <p>in = table_env.from_path(&quot;Source1&quot;).select(&quot;a, b, c&quot;)
          <br
          />
          <br />#&#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x7684;&#x524D;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p>result1 = in.order_by(&quot;a.asc&quot;).fetch(5)
          <br />
        </p>
        <p>#&#x8DF3;&#x8FC7;&#x524D;3&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x8FD4;&#x56DE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x7684;&#x6240;&#x6709;&#x8BB0;&#x5F55;</p>
        <p>result2 = in.order_by(&quot;a.asc&quot;).offset(3)
          <br />
          <br />#&#x8DF3;&#x8FC7;&#x524D;10&#x6761;&#x8BB0;&#x5F55;&#x5E76;&#x4ECE;&#x6392;&#x5E8F;&#x7ED3;&#x679C;&#x4E2D;&#x8FD4;&#x56DE;&#x540E;5&#x6761;&#x8BB0;&#x5F55;</p>
        <p>result3 = in.order_by(&quot;a.asc&quot;).offset(10).fetch(5)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 插入\(Insert\)

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Insert Into</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;INSERT INTO&#x5B50;&#x53E5;&#x3002;&#x6267;&#x884C;&#x63D2;&#x5165;&#x5DF2;&#x6CE8;&#x518C;&#x7684;&#x8F93;&#x51FA;&#x8868;&#x3002;</p>
        <p>&#x8F93;&#x51FA;&#x8868;&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;</a>&#xFF08;&#x8BF7;&#x53C2;&#x9605;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;TableSink</a>&#xFF09;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x6A21;&#x5F0F;&#x5FC5;&#x987B;&#x4E0E;&#x67E5;&#x8BE2;&#x7684;&#x6A21;&#x5F0F;&#x5339;&#x914D;&#x3002;</p>
        <p>Table orders <b>=</b> tableEnv<b>.</b>scan(&quot;Orders&quot;);</p>
        <p>orders<b>.</b>insertInto(&quot;OutOrders&quot;);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Insert Into</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;INSERT INTO&#x5B50;&#x53E5;&#x3002;&#x6267;&#x884C;&#x63D2;&#x5165;&#x5DF2;&#x6CE8;&#x518C;&#x7684;&#x8F93;&#x51FA;&#x8868;&#x3002;</p>
        <p>&#x8F93;&#x51FA;&#x8868;&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;</a>&#xFF08;&#x8BF7;&#x53C2;&#x9605;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;TableSink</a>&#xFF09;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x6A21;&#x5F0F;&#x5FC5;&#x987B;&#x4E0E;&#x67E5;&#x8BE2;&#x7684;&#x6A21;&#x5F0F;&#x5339;&#x914D;&#x3002;</p>
        <p>val orders: Table = tableEnv.scan(&quot;Orders&quot;) orders.insertInto(&quot;OutOrders&quot;)</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Insert Into</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4E0E;SQL&#x67E5;&#x8BE2;&#x4E2D;&#x7684;INSERT INTO&#x5B50;&#x53E5;&#x76F8;&#x4F3C;&#x3002;&#x5728;&#x5DF2;&#x63D2;&#x5165;&#x7684;&#x8F93;&#x51FA;&#x8868;&#x4E2D;&#x6267;&#x884C;&#x63D2;&#x5165;&#x3002;</p>
        <p>&#x8F93;&#x51FA;&#x8868;&#x5FC5;&#x987B;&#x5728;TableEnvironment&#x4E2D;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;</a>&#xFF08;&#x8BF7;&#x53C2;&#x9605;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/common.html#register-a-tablesink">&#x6CE8;&#x518C;TableSink</a>&#xFF09;&#x3002;&#x6B64;&#x5916;&#xFF0C;&#x5DF2;&#x6CE8;&#x518C;&#x8868;&#x7684;&#x67B6;&#x6784;&#x5FC5;&#x987B;&#x4E0E;&#x67E5;&#x8BE2;&#x7684;&#x67B6;&#x6784;&#x5339;&#x914D;&#x3002;</p>
        <p>orders = table_env.from_path(&quot;Orders&quot;); orders.insert_into(&quot;OutOrders&quot;);</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 分组窗口\(Group Windows\)

分组窗口根据时间或行计数间隔将行组聚合为有限组，并按组评估聚合函数。 对于批处理表，窗口是按时间间隔对记录进行分组的便捷快捷方式。

`Windows`是使用`window(w：Window)`子句定义的，需要使用as子句指定的别名。 为了按窗口对表进行分组，必须在`groupBy(...)`子句中引用窗口别名，就像常规分组属性一样。 以下示例展示如何在表上定义窗口聚合。

{% tabs %}
{% tab title="Java" %}
```java
Table table = input
  .window([Window w].as("w"))  // define window with alias w
  .groupBy("w")  // group the table by window w
  .select("b.sum");  // aggregate
```
{% endtab %}

{% tab title="Scala" %}
```scala
val table = input
  .window([w: Window] as 'w)  // define window with alias w
  .groupBy('w)   // group the table by window w
  .select('b.sum)  // aggregate
```
{% endtab %}

{% tab title="Python" %}
```python
# define window with alias w, group the table by window w, then aggregate
table = input.window([GroupWindow w].alias("w")) \
             .group_by("w").select("b.sum")
```
{% endtab %}
{% endtabs %}

在流式传输环境中，如果窗口聚合除了窗口之外还在一个或多个属性上进行分组，则它们只能并行计算，即，`groupBy(...)`子句引用窗口别名和至少一个附加属性。 仅引用窗口别名的`groupBy(...)`子句（例如上面的示例中）只能由单个非并行任务来评估。 以下示例显示如何使用其他分组属性定义窗口聚合。

{% tabs %}
{% tab title="Java" %}
```java
Table table = input
  .window([Window w].as("w"))  // define window with alias w
  .groupBy("w, a")  // group the table by attribute a and window w 
  .select("a, b.sum");  // aggregate
```
{% endtab %}

{% tab title="Scala" %}
```scala
val table = input
  .window([w: Window] as 'w) // define window with alias w
  .groupBy('w, 'a)  // group the table by attribute a and window w 
  .select('a, 'b.sum)  // aggregate
```
{% endtab %}

{% tab title="Python" %}
```python
# define window with alias w, group the table by attribute a and window w,
# then aggregate
table = input.window([GroupWindow w].alias("w")) \
             .group_by("w, a").select("b.sum")
```
{% endtab %}
{% endtabs %}

窗口属性（如时间窗口的开始，结束或行时间戳）可以在`select`语句中添加为窗口别名的属性，分别为`w.start`，`w.end`和`w.rowtime`。 窗口开始和行时间戳是包含的下窗口和上窗口边界。 相反，窗口结束时间戳是独占的上窗口边界。 例如，从下午2点开始的30分钟的翻滚窗口将以`14:00:00.000`作为开始时间戳，`14:29:59.999`作为行时间戳，并且`14:30:00.000`作为结束时间戳。

{% tabs %}
{% tab title="Java" %}
```java
Table table = input
  .window([Window w].as("w"))  // define window with alias w
  .groupBy("w, a")  // group the table by attribute a and window w 
  .select("a, w.start, w.end, w.rowtime, b.count"); // aggregate and add window start, end, and rowtime timestamps
```
{% endtab %}

{% tab title="Scala" %}
```scala
val table = input
  .window([w: Window] as 'w)  // define window with alias w
  .groupBy('w, 'a)  // group the table by attribute a and window w 
  .select('a, 'w.start, 'w.end, 'w.rowtime, 'b.count) // aggregate and add window start, end, and rowtime timestamps

```
{% endtab %}

{% tab title="Python" %}
```python
# define window with alias w, group the table by attribute a and window w,
# then aggregate and add window start, end, and rowtime timestamps
table = input.window([GroupWindow w].alias("w")) \
             .group_by("w, a") \
             .select("a, w.start, w.end, w.rowtime, b.count")
```
{% endtab %}
{% endtabs %}

Window参数定义行如何映射到窗口。 `Window`不是用户可以实现的接口。 相反，Table API提供了一组具有特定语义的预定义`Window`类，这些类被转换为基础`DataStream`或`DataSet`操作。 支持的窗口定义如下所示。

#### **滚动（滚动窗）**

**滚动**窗口将行分配给固定长度的非重叠连续窗口。例如，5分钟的滚动窗口以5分钟为间隔对行进行分组。可以在事件时间，处理时间或行数上定义滚动窗口。

使用`Tumble`类定义翻滚窗口如下：

| 方法 | 描述 |
| :--- | :--- |
| `over` | 定义窗口的长度，可以是时间间隔也可以是行数间隔。 |
| `on` | 时间属性为组（时间间隔）或排序（行计数）。对于批处理查询，这可能是任何Long或Timestamp属性。对于流式查询，这必须是[声明的事件时间或处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)。 |
| `as` | 为窗口指定别名。别名用于引用以下`groupBy()`子句中的窗口，并可选择在子句中选择窗口属性，如窗口开始，结束或行时间戳`select()`。 |

{% tabs %}
{% tab title="Java" %}
```java
// Tumbling Event-time Window
.window(Tumble.over("10.minutes").on("rowtime").as("w"));

// Tumbling Processing-time Window (assuming a processing-time attribute "proctime")
.window(Tumble.over("10.minutes").on("proctime").as("w"));

// Tumbling Row-count Window (assuming a processing-time attribute "proctime")
.window(Tumble.over("10.rows").on("proctime").as("w"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Tumbling Event-time Window
.window(Tumble over 10.minutes on 'rowtime as 'w)

// Tumbling Processing-time Window (assuming a processing-time attribute "proctime")
.window(Tumble over 10.minutes on 'proctime as 'w)

// Tumbling Row-count Window (assuming a processing-time attribute "proctime")
.window(Tumble over 10.rows on 'proctime as 'w)
```
{% endtab %}

{% tab title="Python" %}
```python
# Tumbling Event-time Window
.window(Tumble.over("10.minutes").on("rowtime").alias("w"))

# Tumbling Processing-time Window (assuming a processing-time attribute "proctime")
.window(Tumble.over("10.minutes").on("proctime").alias("w"))

# Tumbling Row-count Window (assuming a processing-time attribute "proctime")
.window(Tumble.over("10.rows").on("proctime").alias("w"));
```
{% endtab %}
{% endtabs %}

#### **滑动（滑动窗口）**

滑动窗口具有固定大小，并按指定的滑动间隔滑动。如果滑动间隔小于窗口大小，则滑动窗口重叠。因此，可以将行分配给多个窗口。例如，15分钟大小和5分钟滑动间隔的滑动窗口将每行分配给3个不同的15分钟大小的窗口，这些窗口以5分钟的间隔进行评估。可以在事件时间，处理时间或行数上定义滑动窗口。

使用`Slide`类定义滑动窗口如下：

| 方法 | 描述 |
| :--- | :--- |
| `over` | 定义窗口的长度，可以是时间或行计数间隔。 |
| `every` | 定义滑动间隔，可以是时间间隔也可以是行数。滑动间隔必须与大小间隔的类型相同。 |
| `on` | 时间属性为组（时间间隔）或排序（行计数）。对于批处理查询，这可能是任何Long或Timestamp属性。对于流式查询，这必须是[声明的事件时间或处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)。 |
| `as` | 为窗口指定别名。别名用于引用以下`groupBy()`子句中的窗口，并可选择在子句中选择窗口属性，如窗口开始，结束或行时间戳`select()`。 |

{% tabs %}
{% tab title="Java" %}
```java
// Sliding Event-time Window
.window(Slide.over("10.minutes").every("5.minutes").on("rowtime").as("w"));

// Sliding Processing-time window (assuming a processing-time attribute "proctime")
.window(Slide.over("10.minutes").every("5.minutes").on("proctime").as("w"));

// Sliding Row-count window (assuming a processing-time attribute "proctime")
.window(Slide.over("10.rows").every("5.rows").on("proctime").as("w"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Sliding Event-time Window
.window(Slide over 10.minutes every 5.minutes on 'rowtime as 'w)

// Sliding Processing-time window (assuming a processing-time attribute "proctime")
.window(Slide over 10.minutes every 5.minutes on 'proctime as 'w)

// Sliding Row-count window (assuming a processing-time attribute "proctime")
.window(Slide over 10.rows every 5.rows on 'proctime as 'w)
```
{% endtab %}

{% tab title="Python" %}
```python
# Sliding Event-time Window
.window(Slide.over("10.minutes").every("5.minutes").on("rowtime").alias("w"))

# Sliding Processing-time window (assuming a processing-time attribute "proctime")
.window(Slide.over("10.minutes").every("5.minutes").on("proctime").alias("w"))

# Sliding Row-count window (assuming a processing-time attribute "proctime")
.window(Slide.over("10.rows").every("5.rows").on("proctime").alias("w"))
```
{% endtab %}
{% endtabs %}

#### **会话（会话窗口）**

会话窗口没有固定的大小，但它们的边界由不活动的间隔定义，即如果在定义的间隙期间没有出现事件，则会话窗口关闭。例如，在30分钟不活动后观察到一行时，会有一个30分钟间隙的会话窗口（否则该行将被添加到现有窗口中），如果在30分钟内未添加任何行，则会关闭。会话窗口可以在事件时间或处理时间上工作。

使用`Session`类定义会话窗口，如下所示：

| 方法 | 描述 |
| :--- | :--- |
| `withGap` | 将两个窗口之间的间隔定义为时间间隔。 |
| `on` | 时间属性为组（时间间隔）或排序（行计数）。对于批处理查询，这可能是任何Long或Timestamp属性。对于流式查询，这必须是[声明的事件时间或处理时间属性](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html)。 |
| `as` | 为窗口指定别名。别名用于引用以下`groupBy()`子句中的窗口，并可选择在子句中选择窗口属性，如窗口开始，结束或行时间戳`select()`。 |

{% tabs %}
{% tab title="Java" %}
```java
// Session Event-time Window
.window(Session.withGap("10.minutes").on("rowtime").as("w"));

// Session Processing-time Window (assuming a processing-time attribute "proctime")
.window(Session.withGap("10.minutes").on("proctime").as("w"));
 回到顶部


```
{% endtab %}

{% tab title="Scala" %}
```scala
.window(Session withGap 10.minutes on 'rowtime as 'w)

// Session Processing-time Window (assuming a processing-time attribute "proctime")
.window(Session withGap 10.minutes on 'proctime as 'w)
```
{% endtab %}

{% tab title="Python" %}
```python
# Session Event-time Window
.window(Session.with_gap("10.minutes").on("rowtime").alias("w"))

# Session Processing-time Window (assuming a processing-time attribute "proctime")
.window(Session.with_gap("10.minutes").on("proctime").alias("w"))
```
{% endtab %}
{% endtabs %}

### Over Windows

{% tabs %}
{% tab title="Java" %}
```java
Table table = input
  .window([OverWindow w].as("w"))           // define over window with alias w
  .select("a, b.sum over w, c.min over w"); // aggregate over the over window w
```
{% endtab %}

{% tab title="Scala" %}
```scala
val table = input
  .window([w: OverWindow] as 'w)              // define over window with alias w
  .select('a, 'b.sum over 'w, 'c.min over 'w) // aggregate over the over window w
```
{% endtab %}

{% tab title="Python" %}
```python
# define over window with alias w and aggregate over the over window w
table = input.over_window([OverWindow w].alias("w")) \
    .select("a, b.sum over w, c.min over w")
```
{% endtab %}
{% endtabs %}

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x65B9;&#x6CD5;</th>
      <th style="text-align:left">&#x9700;&#x8981;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>partitionBy</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;&#x7684;</td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5C5E;&#x6027;&#x4E0A;&#x7684;&#x8F93;&#x5165;&#x5206;&#x533A;&#x3002;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x90FD;&#x662F;&#x5355;&#x72EC;&#x6392;&#x5E8F;&#x7684;&#xFF0C;&#x805A;&#x5408;&#x51FD;&#x6570;&#x5206;&#x522B;&#x5E94;&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5728;&#x6D41;&#x5F0F;&#x73AF;&#x5883;&#x4E2D;&#xFF0C;&#x5982;&#x679C;&#x7A97;&#x53E3;&#x5305;&#x542B;partition
          by&#x5B50;&#x53E5;&#xFF0C;&#x5219;&#x53EA;&#x80FD;&#x5E76;&#x884C;&#x8BA1;&#x7B97;&#x7A97;&#x53E3;&#x805A;&#x5408;&#x3002;&#x6CA1;&#x6709;<code>partitionBy(...)</code>&#x6D41;&#x7531;&#x5355;&#x4E2A;&#x975E;&#x5E76;&#x884C;&#x4EFB;&#x52A1;&#x5904;&#x7406;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>orderBy</code>
      </td>
      <td style="text-align:left">&#x9700;&#x8981;</td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x4E2D;&#x884C;&#x7684;&#x987A;&#x5E8F;&#xFF0C;&#x4ECE;&#x800C;&#x5B9A;&#x4E49;&#x805A;&#x5408;&#x51FD;&#x6570;&#x5E94;&#x7528;&#x4E8E;&#x884C;&#x7684;&#x987A;&#x5E8F;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x8FD9;&#x5FC5;&#x987B;&#x662F;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/time_attributes.html">&#x58F0;&#x660E;&#x7684;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#x6216;&#x5904;&#x7406;&#x65F6;&#x95F4;&#x5C5E;&#x6027;</a>&#x3002;&#x76EE;&#x524D;&#xFF0C;&#x4EC5;&#x652F;&#x6301;&#x5355;&#x4E2A;&#x6392;&#x5E8F;&#x5C5E;&#x6027;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>preceding</code>
      </td>
      <td style="text-align:left">&#x9700;&#x8981;</td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x7A97;&#x53E3;&#x4E2D;&#x5305;&#x542B;&#x7684;&#x884C;&#x7684;&#x95F4;&#x9694;&#xFF0C;&#x5E76;&#x5728;&#x5F53;&#x524D;&#x884C;&#x4E4B;&#x524D;&#x3002;&#x95F4;&#x9694;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E3A;&#x65F6;&#x95F4;&#x6216;&#x884C;&#x8BA1;&#x6570;&#x95F4;&#x9694;&#x3002;</p>
        <p><a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#bounded-over-windows">&#x5728;&#x7A97;&#x53E3;</a>&#x4E0A;&#x9650;&#x5B9A;&#x5177;&#x6709;&#x95F4;&#x9694;&#x7684;&#x5927;&#x5C0F;&#xFF0C;&#x4F8B;&#x5982;&#xFF0C;<code>10.minutes</code>&#x5BF9;&#x4E8E;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x6216;<code>10.rows</code>&#x884C;&#x8BA1;&#x6570;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F7F;&#x7528;&#x5E38;&#x91CF;&#x6307;&#x5B9A;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#unbounded-over-windows">&#x5728;&#x7A97;&#x53E3;</a>&#x4E0A;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/tableApi.html#unbounded-over-windows">&#x65E0;&#x754C;&#x9650;</a>&#xFF0C;&#x5373;&#xFF0C;<code>UNBOUNDED_RANGE</code>&#x5BF9;&#x4E8E;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x6216;<code>UNBOUNDED_ROW</code>&#x884C;&#x8BA1;&#x6570;&#x95F4;&#x9694;&#x3002;&#x5728;Windows&#x4E0A;&#x65E0;&#x9650;&#x5236;&#x5730;&#x4ECE;&#x5206;&#x533A;&#x7684;&#x7B2C;&#x4E00;&#x884C;&#x5F00;&#x59CB;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>following</code>
      </td>
      <td style="text-align:left">&#x53EF;&#x9009;&#x7684;</td>
      <td style="text-align:left">
        <p>&#x5B9A;&#x4E49;&#x7A97;&#x53E3;&#x4E2D;&#x5305;&#x542B;&#x7684;&#x884C;&#x7684;&#x7A97;&#x53E3;&#x95F4;&#x9694;&#xFF0C;&#x5E76;&#x8DDF;&#x968F;&#x5F53;&#x524D;&#x884C;&#x3002;&#x5FC5;&#x987B;&#x5728;&#x4E0E;&#x524D;&#x4E00;&#x4E2A;&#x95F4;&#x9694;&#xFF08;&#x65F6;&#x95F4;&#x6216;&#x884C;&#x8BA1;&#x6570;&#xFF09;&#x76F8;&#x540C;&#x7684;&#x5355;&#x4F4D;&#x4E2D;&#x6307;&#x5B9A;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x76EE;&#x524D;&#xFF0C;&#x4E0D;&#x652F;&#x6301;&#x5728;&#x5F53;&#x524D;&#x884C;&#x4E4B;&#x540E;&#x5305;&#x542B;&#x884C;&#x7684;&#x7A97;&#x53E3;&#x3002;&#x76F8;&#x53CD;&#xFF0C;&#x60A8;&#x53EF;&#x4EE5;&#x6307;&#x5B9A;&#x4E24;&#x4E2A;&#x5E38;&#x91CF;&#x4E4B;&#x4E00;&#xFF1A;</p>
        <ul>
          <li><code>CURRENT_ROW</code> &#x5C06;&#x7A97;&#x53E3;&#x7684;&#x4E0A;&#x9650;&#x8BBE;&#x7F6E;&#x4E3A;&#x5F53;&#x524D;&#x884C;&#x3002;</li>
          <li><code>CURRENT_RANGE</code> &#x8BBE;&#x7F6E;&#x7A97;&#x53E3;&#x7684;&#x4E0A;&#x9650;&#x4EE5;&#x5BF9;&#x5F53;&#x524D;&#x884C;&#x7684;&#x6392;&#x5E8F;&#x952E;&#x8FDB;&#x884C;&#x6392;&#x5E8F;&#xFF0C;&#x5373;&#x7A97;&#x53E3;&#x4E2D;&#x5305;&#x542B;&#x4E0E;&#x5F53;&#x524D;&#x884C;&#x5177;&#x6709;&#x76F8;&#x540C;&#x6392;&#x5E8F;&#x952E;&#x7684;&#x6240;&#x6709;&#x884C;&#x3002;</li>
        </ul>
        <p>&#x5982;&#x679C;<code>following</code>&#x7701;&#x7565;&#x8BE5;&#x5B50;&#x53E5;&#xFF0C;&#x5219;&#x5C06;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x7A97;&#x53E3;<code>CURRENT_RANGE</code>&#x7684;&#x4E0A;&#x9650;&#x5B9A;&#x4E49;&#x4E3A;&#xFF0C;&#x5E76;&#x5C06;&#x884C;&#x8BA1;&#x6570;&#x95F4;&#x9694;&#x7A97;&#x53E3;&#x7684;&#x4E0A;&#x9650;&#x5B9A;&#x4E49;&#x4E3A;<code>CURRENT_ROW</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><code>as</code>
      </td>
      <td style="text-align:left">&#x9700;&#x8981;</td>
      <td style="text-align:left">&#x4E3A;&#x8986;&#x76D6;&#x7A97;&#x53E3;&#x6307;&#x5B9A;&#x522B;&#x540D;&#x3002;&#x522B;&#x540D;&#x7528;&#x4E8E;&#x5F15;&#x7528;&#x4EE5;&#x4E0B;<code>select()</code>&#x5B50;&#x53E5;&#x4E2D;&#x7684;over
        window &#x3002;</td>
    </tr>
  </tbody>
</table>**注意：**当前，同一select（）调用中的所有聚合函数都必须使用同一over window计算。

#### **无界\(Unbounded\) Over Windows**

{% tabs %}
{% tab title="Java" %}
```java
// Unbounded Event-time over window (assuming an event-time attribute "rowtime")
.window(Over.partitionBy("a").orderBy("rowtime").preceding("unbounded_range").as("w"));

// Unbounded Processing-time over window (assuming a processing-time attribute "proctime")
.window(Over.partitionBy("a").orderBy("proctime").preceding("unbounded_range").as("w"));

// Unbounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.window(Over.partitionBy("a").orderBy("rowtime").preceding("unbounded_row").as("w"));
 
// Unbounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.window(Over.partitionBy("a").orderBy("proctime").preceding("unbounded_row").as("w"));
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Unbounded Event-time over window (assuming an event-time attribute "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding UNBOUNDED_RANGE as 'w)

// Unbounded Processing-time over window (assuming a processing-time attribute "proctime")
.window(Over partitionBy 'a orderBy 'proctime preceding UNBOUNDED_RANGE as 'w)

// Unbounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding UNBOUNDED_ROW as 'w)
 
// Unbounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.window(Over partitionBy 'a orderBy 'proctime preceding UNBOUNDED_ROW as 'w)
```
{% endtab %}

{% tab title="Python" %}
```python
# Unbounded Event-time over window (assuming an event-time attribute "rowtime")
.over_window(Over.partition_by("a").order_by("rowtime").preceding("unbounded_range").alias("w"))

# Unbounded Processing-time over window (assuming a processing-time attribute "proctime")
.over_window(Over.partition_by("a").order_by("proctime").preceding("unbounded_range").alias("w"))

# Unbounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.over_window(Over.partition_by("a").order_by("rowtime").preceding("unbounded_row").alias("w"))
 
# Unbounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.over_window(Over.partition_by("a").order_by("proctime").preceding("unbounded_row").alias("w"))
```
{% endtab %}
{% endtabs %}

#### **有界\(Bounded\) Over Windows**

{% tabs %}
{% tab title="Java" %}
```java
// Bounded Event-time over window (assuming an event-time attribute "rowtime")
.window(Over.partitionBy("a").orderBy("rowtime").preceding("1.minutes").as("w"))

// Bounded Processing-time over window (assuming a processing-time attribute "proctime")
.window(Over.partitionBy("a").orderBy("proctime").preceding("1.minutes").as("w"))

// Bounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.window(Over.partitionBy("a").orderBy("rowtime").preceding("10.rows").as("w"))
 
// Bounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.window(Over.partitionBy("a").orderBy("proctime").preceding("10.rows").as("w"))
```
{% endtab %}

{% tab title="Scala" %}
```scala
// Bounded Event-time over window (assuming an event-time attribute "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding 1.minutes as 'w)

// Bounded Processing-time over window (assuming a processing-time attribute "proctime")
.window(Over partitionBy 'a orderBy 'proctime preceding 1.minutes as 'w)

// Bounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.window(Over partitionBy 'a orderBy 'rowtime preceding 10.rows as 'w)
  
// Bounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.window(Over partitionBy 'a orderBy 'proctime preceding 10.rows as 'w)
```
{% endtab %}

{% tab title="Python" %}
```python
# Bounded Event-time over window (assuming an event-time attribute "rowtime")
.over_window(Over.partition_by("a").order_by("rowtime").preceding("1.minutes").alias("w"))

# Bounded Processing-time over window (assuming a processing-time attribute "proctime")
.over_window(Over.partition_by("a").order_by("proctime").preceding("1.minutes").alias("w"))

# Bounded Event-time Row-count over window (assuming an event-time attribute "rowtime")
.over_window(Over.partition_by("a").order_by("rowtime").preceding("10.rows").alias("w"))
 
# Bounded Processing-time Row-count over window (assuming a processing-time attribute "proctime")
.over_window(Over.partition_by("a").order_by("proctime").preceding("10.rows").alias("w"))
```
{% endtab %}
{% endtabs %}

### 基于行的操作

基于行的操作生成具有多列的输出。

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"> <b>Map</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x6807;&#x91CF;&#x51FD;&#x6570;&#x6216;&#x5185;&#x7F6E;&#x6807;&#x91CF;&#x51FD;&#x6570;&#x6267;&#x884C;&#x6620;&#x5C04;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x8F93;&#x51FA;&#x7C7B;&#x578B;&#x662F;&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x8F93;&#x51FA;&#x5C06;&#x88AB;&#x5C55;&#x5E73;&#x3002;</p>
        <p>public class MyMapFunction extends ScalarFunction {</p>
        <p>public Row eval(String a) {</p>
        <p>return Row.of(a, &quot;pre-&quot; + a);</p>
        <p>}</p>
        <p>@Override</p>
        <p>public TypeInformation&lt;?&gt;getResultType(Class&lt;?&gt;[] signature)
          {</p>
        <p>return Types.ROW(Types.STRING(), Types.STRING());</p>
        <p>}</p>
        <p>}</p>
        <p>ScalarFunction func = new MyMapFunction(); tableEnv.registerFunction(&quot;func&quot;,
          func);</p>
        <p>Table table = input .map(&quot;func(c)&quot;).as(&quot;a, b&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>FlatMap</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x6267;&#x884C;flatMap&#x64CD;&#x4F5C;&#x3002;</p>
        <p>public class MyFlatMapFunction extends TableFunction {</p>
        <p>public void eval(String str) {</p>
        <p>if (str.contains(&quot;#&quot;)) {</p>
        <p>String[] array = str.split(&quot;#&quot;);</p>
        <p>for (int i = 0; i &lt; array.length; ++i) {</p>
        <p>collect(Row.of(array[i], array[i].length()));</p>
        <p>}</p>
        <p>}</p>
        <p>}</p>
        <p>@Override</p>
        <p>public TypeInformation getResultType() {</p>
        <p>return Types.ROW(Types.STRING(), Types.INT());</p>
        <p>}</p>
        <p>}</p>
        <p>TableFunction func = new MyFlatMapFunction(); tableEnv.registerFunction(&quot;func&quot;,
          func);</p>
        <p>Table table = input .flatMap(&quot;func(c)&quot;).as(&quot;a, b&quot;)</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Aggregate</b>
        <br />Batch Streaming Result Updating</td>
      <td style="text-align:left">&#x4F7F;&#x7528;&#x805A;&#x5408;&#x51FD;&#x6570;&#x6267;&#x884C;&#x805A;&#x5408;&#x64CD;&#x4F5C;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;&#x805A;&#x5408;&#x201D;&#xFF0C;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x529F;&#x80FD;&#x3002;&#x5982;&#x679C;&#x8F93;&#x51FA;&#x7C7B;&#x578B;&#x662F;&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x805A;&#x5408;&#x7684;&#x8F93;&#x51FA;&#x5C06;&#x88AB;&#x5C55;&#x5E73;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Group Window Aggregate</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">&#x5728;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#group-windows">&#x7EC4;&#x7A97;&#x53E3;</a>&#x548C;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x5BF9;&#x8868;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x96C6;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;&#x805A;&#x5408;&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x201C;
        *&#x201D;&#x6216;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>FlatAggregate</b>
        <br />Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>GroupBy&#x805A;&#x5408;</b>&#x3002;&#x4F7F;&#x7528;&#x4EE5;&#x4E0B;&#x8FD0;&#x884C;&#x8868;&#x805A;&#x5408;&#x8FD0;&#x7B97;&#x7B26;&#x5C06;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x7684;&#x884C;&#x5206;&#x7EC4;&#xFF0C;&#x4EE5;&#x9010;&#x884C;&#x805A;&#x5408;&#x884C;&#x3002;&#x4E0E;AggregateFunction&#x7684;&#x533A;&#x522B;&#x5728;&#x4E8E;TableAggregateFunction&#x53EF;&#x4EE5;&#x4E3A;&#x4E00;&#x4E2A;&#x7EC4;&#x8FD4;&#x56DE;0&#x4E2A;&#x6216;&#x66F4;&#x591A;&#x8BB0;&#x5F55;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;
          flatAggregate&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</p>
        <p>&#x9664;&#x4E86;&#x4F7F;&#x7528;<code>emitValue</code>&#x8F93;&#x51FA;&#x7ED3;&#x679C;&#xFF0C;&#x8FD8;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;<code>emitUpdateWithRetract</code>&#x65B9;&#x6CD5;&#x3002;&#x4E0E;&#x4E0D;&#x540C;<code>emitValue</code>&#xFF0C;<code>emitUpdateWithRetract</code>&#x7528;&#x4E8E;&#x53D1;&#x51FA;&#x5DF2;&#x66F4;&#x65B0;&#x7684;&#x503C;&#x3002;&#x6B64;&#x65B9;&#x6CD5;&#x4EE5;&#x7F29;&#x56DE;&#x6A21;&#x5F0F;&#x589E;&#x91CF;&#x8F93;&#x51FA;&#x6570;&#x636E;&#xFF0C;&#x5373;&#xFF0C;&#x4E00;&#x65E6;&#x6709;&#x66F4;&#x65B0;&#xFF0C;&#x6211;&#x4EEC;&#x5FC5;&#x987B;&#x5148;&#x7F29;&#x56DE;&#x65E7;&#x8BB0;&#x5F55;&#xFF0C;&#x7136;&#x540E;&#x518D;&#x53D1;&#x9001;&#x65B0;&#x7684;&#x66F4;&#x65B0;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x5728;&#x8868;&#x805A;&#x5408;&#x51FD;&#x6570;&#x4E2D;&#x5B9A;&#x4E49;&#x4E86;&#x8FD9;&#x4E24;&#x79CD;&#x65B9;&#x6CD5;&#xFF0C;&#x5219;&#x8BE5;<code>emitUpdateWithRetract</code>&#x65B9;&#x6CD5;&#x5C06;&#x4F18;&#x5148;&#x4E8E;&#x8BE5;<code>emitValue</code>&#x65B9;&#x6CD5;&#x4F7F;&#x7528;&#xFF0C;&#x56E0;&#x4E3A;&#x8FD9;&#x88AB;&#x8BA4;&#x4E3A;&#x6BD4;&#x8BE5;&#x65B9;&#x6CD5;&#x66F4;&#x6709;&#x6548;&#xFF0C;<code>emitValue</code>&#x56E0;&#x4E3A;&#x5B83;&#x53EF;&#x4EE5;&#x9012;&#x589E;&#x5730;&#x8F93;&#x51FA;&#x503C;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/udfs.html#table-aggregation-functions">&#x53C2;&#x89C1;&#x8868;&#x805A;&#x5408;&#x51FD;&#x6570;</a>&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x805A;&#x5408;&#x7C7B;&#x578B;&#x548C;&#x4E0D;&#x540C;&#x5206;&#x7EC4;&#x5173;&#x952E;&#x5B57;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Group Window FlatAggregate</b>
        <br />Streaming
        <br />
      </td>
      <td style="text-align:left">&#x5728;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#group-windows">&#x7EC4;&#x7A97;&#x53E3;</a>&#x548C;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x5BF9;&#x8868;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x96C6;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;
        flatAggregate&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x64CD;&#x4F5C;&#x7B26;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"> <b>Map</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">&#x4F7F;&#x7528;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x6807;&#x91CF;&#x51FD;&#x6570;&#x6216;&#x5185;&#x7F6E;&#x6807;&#x91CF;&#x51FD;&#x6570;&#x6267;&#x884C;&#x6620;&#x5C04;&#x64CD;&#x4F5C;&#x3002;&#x5982;&#x679C;&#x8F93;&#x51FA;&#x7C7B;&#x578B;&#x662F;&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x8F93;&#x51FA;&#x5C06;&#x88AB;&#x5C55;&#x5E73;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>FlatMap</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">&#x4F7F;&#x7528;&#x8868;&#x51FD;&#x6570;&#x6267;&#x884C;flatMap&#x64CD;&#x4F5C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Aggregate</b>
        <br />Batch Streaming Result Updating</td>
      <td style="text-align:left">&#x4F7F;&#x7528;&#x805A;&#x5408;&#x51FD;&#x6570;&#x6267;&#x884C;&#x805A;&#x5408;&#x64CD;&#x4F5C;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;&#x805A;&#x5408;&#x201D;&#xFF0C;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x529F;&#x80FD;&#x3002;&#x5982;&#x679C;&#x8F93;&#x51FA;&#x7C7B;&#x578B;&#x662F;&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF0C;&#x5219;&#x805A;&#x5408;&#x7684;&#x8F93;&#x51FA;&#x5C06;&#x88AB;&#x5C55;&#x5E73;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Group Window Aggregate</b>
        <br />Batch Streaming</td>
      <td style="text-align:left">&#x5728;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#group-windows">&#x7EC4;&#x7A97;&#x53E3;</a>&#x548C;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x5BF9;&#x8868;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x96C6;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;&#x805A;&#x5408;&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x201C;
        *&#x201D;&#x6216;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>FlatAggregate</b>
        <br />Streaming
        <br />Result Updating</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;<b>GroupBy&#x805A;&#x5408;</b>&#x3002;&#x4F7F;&#x7528;&#x4EE5;&#x4E0B;&#x8FD0;&#x884C;&#x8868;&#x805A;&#x5408;&#x8FD0;&#x7B97;&#x7B26;&#x5C06;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x7684;&#x884C;&#x5206;&#x7EC4;&#xFF0C;&#x4EE5;&#x9010;&#x884C;&#x805A;&#x5408;&#x884C;&#x3002;&#x4E0E;AggregateFunction&#x7684;&#x533A;&#x522B;&#x5728;&#x4E8E;TableAggregateFunction&#x53EF;&#x4EE5;&#x4E3A;&#x4E00;&#x4E2A;&#x7EC4;&#x8FD4;&#x56DE;0&#x4E2A;&#x6216;&#x66F4;&#x591A;&#x8BB0;&#x5F55;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;
          flatAggregate&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</p>
        <p>&#x9664;&#x4E86;&#x4F7F;&#x7528;<code>emitValue</code>&#x8F93;&#x51FA;&#x7ED3;&#x679C;&#xFF0C;&#x8FD8;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;<code>emitUpdateWithRetract</code>&#x65B9;&#x6CD5;&#x3002;&#x4E0E;&#x4E0D;&#x540C;<code>emitValue</code>&#xFF0C;<code>emitUpdateWithRetract</code>&#x7528;&#x4E8E;&#x53D1;&#x51FA;&#x5DF2;&#x66F4;&#x65B0;&#x7684;&#x503C;&#x3002;&#x6B64;&#x65B9;&#x6CD5;&#x4EE5;&#x7F29;&#x56DE;&#x6A21;&#x5F0F;&#x589E;&#x91CF;&#x8F93;&#x51FA;&#x6570;&#x636E;&#xFF0C;&#x5373;&#xFF0C;&#x4E00;&#x65E6;&#x6709;&#x66F4;&#x65B0;&#xFF0C;&#x6211;&#x4EEC;&#x5FC5;&#x987B;&#x5148;&#x7F29;&#x56DE;&#x65E7;&#x8BB0;&#x5F55;&#xFF0C;&#x7136;&#x540E;&#x518D;&#x53D1;&#x9001;&#x65B0;&#x7684;&#x66F4;&#x65B0;&#x8BB0;&#x5F55;&#x3002;&#x5982;&#x679C;&#x5728;&#x8868;&#x805A;&#x5408;&#x51FD;&#x6570;&#x4E2D;&#x5B9A;&#x4E49;&#x4E86;&#x8FD9;&#x4E24;&#x79CD;&#x65B9;&#x6CD5;&#xFF0C;&#x5219;&#x8BE5;<code>emitUpdateWithRetract</code>&#x65B9;&#x6CD5;&#x5C06;&#x4F18;&#x5148;&#x4E8E;&#x8BE5;<code>emitValue</code>&#x65B9;&#x6CD5;&#x4F7F;&#x7528;&#xFF0C;&#x56E0;&#x4E3A;&#x8FD9;&#x88AB;&#x8BA4;&#x4E3A;&#x6BD4;&#x8BE5;&#x65B9;&#x6CD5;&#x66F4;&#x6709;&#x6548;&#xFF0C;<code>emitValue</code>&#x56E0;&#x4E3A;&#x5B83;&#x53EF;&#x4EE5;&#x9012;&#x589E;&#x5730;&#x8F93;&#x51FA;&#x503C;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/udfs.html#table-aggregation-functions">&#x53C2;&#x89C1;&#x8868;&#x805A;&#x5408;&#x51FD;&#x6570;</a>&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x6839;&#x636E;&#x805A;&#x5408;&#x7C7B;&#x578B;&#x548C;&#x4E0D;&#x540C;&#x5206;&#x7EC4;&#x5173;&#x952E;&#x5B57;&#x7684;&#x6570;&#x91CF;&#xFF0C;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x51FA;&#x73B0;&#x8FC7;&#x591A;&#x7684;&#x72B6;&#x6001;&#x3002;&#x6709;&#x5173;&#x8BE6;&#x7EC6;&#x4FE1;&#x606F;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"> <b>Group Window FlatAggregate</b>
        <br />Streaming
        <br />
      </td>
      <td style="text-align:left">&#x5728;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/tableApi.html#group-windows">&#x7EC4;&#x7A97;&#x53E3;</a>&#x548C;&#x53EF;&#x80FD;&#x7684;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5206;&#x7EC4;&#x952E;&#x4E0A;&#x5BF9;&#x8868;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x548C;&#x805A;&#x96C6;&#x3002;&#x60A8;&#x5FC5;&#x987B;&#x4F7F;&#x7528;select&#x8BED;&#x53E5;&#x5173;&#x95ED;&#x201C;
        flatAggregate&#x201D;&#x3002;&#x5E76;&#x4E14;select&#x8BED;&#x53E5;&#x4E0D;&#x652F;&#x6301;&#x805A;&#x5408;&#x51FD;&#x6570;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

## 数据类型

请参阅有关[数据类型](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/types.html)的专用页面。

泛型类型和（嵌套的）复合类型（例如POJO，元组，行，Scala案例类）也可以是一行的字段。

可以使用[值访问函数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/systemFunctions.html#value-access-functions)访问具有任意嵌套的复合类型的字段。

泛型类型被视为黑盒，可以通过[用户定义的函数](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/functions/udfs.html)传递或处理。

## 表达式语法

前面部分中的一些操作符需要一个或多个表达式。可以使用嵌入式Scala DSL或字符串指定表达式。请参阅上面的示例以了解如何指定表达式。

这是表达式的EBNF语法：

```text
expressionList = expression , { "," , expression } ;

expression = overConstant | alias ;

alias = logic | ( logic , "as" , fieldReference ) | ( logic , "as" , "(" , fieldReference , { "," , fieldReference } , ")" ) ;

logic = comparison , [ ( "&&" | "||" ) , comparison ] ;

comparison = term , [ ( "=" | "==" | "===" | "!=" | "!==" | ">" | ">=" | "<" | "<=" ) , term ] ;

term = product , [ ( "+" | "-" ) , product ] ;

product = unary , [ ( "*" | "/" | "%") , unary ] ;

unary = [ "!" | "-" | "+" ] , composite ;

composite = over | suffixed | nullLiteral | prefixed | atom ;

suffixed = interval | suffixAs | suffixCast | suffixIf | suffixDistinct | suffixFunctionCall | timeIndicator ;

prefixed = prefixAs | prefixCast | prefixIf | prefixDistinct | prefixFunctionCall ;

interval = timeInterval | rowInterval ;

timeInterval = composite , "." , ("year" | "years" | "quarter" | "quarters" | "month" | "months" | "week" | "weeks" | "day" | "days" | "hour" | "hours" | "minute" | "minutes" | "second" | "seconds" | "milli" | "millis") ;

rowInterval = composite , "." , "rows" ;

suffixCast = composite , ".cast(" , dataType , ")" ;

prefixCast = "cast(" , expression , dataType , ")" ;

dataType = "BYTE" | "SHORT" | "INT" | "LONG" | "FLOAT" | "DOUBLE" | "BOOLEAN" | "STRING" | "DECIMAL" | "SQL_DATE" | "SQL_TIME" | "SQL_TIMESTAMP" | "INTERVAL_MONTHS" | "INTERVAL_MILLIS" | ( "MAP" , "(" , dataType , "," , dataType , ")" ) | ( "PRIMITIVE_ARRAY" , "(" , dataType , ")" ) | ( "OBJECT_ARRAY" , "(" , dataType , ")" ) ;

suffixAs = composite , ".as(" , fieldReference , ")" ;

prefixAs = "as(" , expression, fieldReference , ")" ;

suffixIf = composite , ".?(" , expression , "," , expression , ")" ;

prefixIf = "?(" , expression , "," , expression , "," , expression , ")" ;

suffixDistinct = composite , "distinct.()" ;

prefixDistinct = functionIdentifier , ".distinct" , [ "(" , [ expression , { "," , expression } ] , ")" ] ;

suffixFunctionCall = composite , "." , functionIdentifier , [ "(" , [ expression , { "," , expression } ] , ")" ] ;

prefixFunctionCall = functionIdentifier , [ "(" , [ expression , { "," , expression } ] , ")" ] ;

atom = ( "(" , expression , ")" ) | literal | fieldReference ;

fieldReference = "*" | identifier ;

nullLiteral = "nullOf(" , dataType , ")" ;

timeIntervalUnit = "YEAR" | "YEAR_TO_MONTH" | "MONTH" | "QUARTER" | "WEEK" | "DAY" | "DAY_TO_HOUR" | "DAY_TO_MINUTE" | "DAY_TO_SECOND" | "HOUR" | "HOUR_TO_MINUTE" | "HOUR_TO_SECOND" | "MINUTE" | "MINUTE_TO_SECOND" | "SECOND" ;

timePointUnit = "YEAR" | "MONTH" | "DAY" | "HOUR" | "MINUTE" | "SECOND" | "QUARTER" | "WEEK" | "MILLISECOND" | "MICROSECOND" ;

over = composite , "over" , fieldReference ;

overConstant = "current_row" | "current_range" | "unbounded_row" | "unbounded_row" ;

timeIndicator = fieldReference , "." , ( "proctime" | "rowtime" ) ;
```

**文字**:这里，文字是一个有效的Java文字。字符串字面值可以使用单引号或双引号指定。复制转义引用\(例如“这我。或者“我”“喜欢”“狗”\)。

**空文字**:空文字必须有一个附加的类型。使用`nullOf(type)`\(例如`nullOf(INT)`\)创建空值。

**字段引用**:`fieldReference`指定数据中的一列\(如果使用\*则指定所有列\)，`functionIdentifier`指定受支持的标量函数。列名和函数名遵循Java标识符语法。

**函数调用**:指定为字符串的表达式也可以使用前缀符号而不是后缀符号来调用操作符和函数。

**小数**:如果需要处理精确的数值或大的小数，那么表API还支持Java的BigDecimal类型。在Scala表中，API小数可以用`BigDecimal(“123456”)`来定义，而在Java中，可以在后面加一个“p”来精确定义，例如123456p。

**时间表示**:为了处理时态值，表API支持Java SQL的`Date`，`Time`和`Timestamp`。在Scala表中，可以使用`java.sql.Date.valueOf("2016-06-27")`、`java.sql. valueof("10:10:42")`或`java.sql.Timestamp`来定义字面量。返回对象的值\(“2016-06-27 10:10:42.123”\)。Java和Scala表API还支持调用`“2016-06-27”. todate()`、`“10:10:42”. totime()`和`“2016-06-27 10:10:42.123”. totimestamp()`来将字符串转换为时态类型。注意:由于Java的时态SQL类型依赖于时区，请确保Flink客户端和所有TaskManager使用相同的时区。

**时间间隔**:时间间隔可以表示为 `months (Types.INTERVAL_MONTHS)`或 `milliseconds (Types.INTERVAL_MILLIS)`。可以添加或减去相同类型的间隔\(例如1\)。小时+ 10.分钟\)。可以将毫秒间隔添加到时间点\(例如，“2016-08-10”。迄今为止,+ 5.天\)。

**Scala表达式**:Scala表达式使用隐式转换。因此，确保添加通配符import `org.apache.flink.table.api.scala._`你的程序。如果文字没有被当作表达式， ，请使用`.toExpr`,例如`3.toExpr`来强制转换文字。

