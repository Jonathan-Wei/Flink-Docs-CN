# Metrics

Flink公开了一个指标系统，允许收集和公开指标到外部系统。

## 注册指标

可以通过调用`getRuntimeContext().getMetricGroup()`从任何扩展[`RichFunction`](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/api_concepts.html#rich-functions)的用户函数访问度量系统。 此方法返回`MetricGroup`对象，可以在该对象上创建和注册新指标。

### 指标类型

Flink支持`Counters`，`Gauges`，`Histograms`和`Meters`。

#### **计数器**

计数器用于计算某些东西。 可以使用`inc()`/ `inc(long n)`或`dec()`/ `dec(long n)`来减小或减小当前值。 可以通过在MetricGroup上调用`counter(String name)`来创建和注册Counter。

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<String, String> {
  private transient Counter counter;

  @Override
  public void open(Configuration config) {
    this.counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCounter");
  }

  @Override
  public String map(String value) throws Exception {
    this.counter.inc();
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[String,String] {
  @transient private var counter: Counter = _

  override def open(parameters: Configuration): Unit = {
    counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCounter")
  }

  override def map(value: String): String = {
    counter.inc()
    value
  }
}
```
{% endtab %}
{% endtabs %}

或者，也可以使用自己的`Counter`实现：

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<String, String> {
  private transient Counter counter;

  @Override
  public void open(Configuration config) {
    this.counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCustomCounter", new CustomCounter());
  }

  @Override
  public String map(String value) throws Exception {
    this.counter.inc();
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[String,String] {
  @transient private var counter: Counter = _

  override def open(parameters: Configuration): Unit = {
    counter = getRuntimeContext()
      .getMetricGroup()
      .counter("myCustomCounter", new CustomCounter())
  }

  override def map(value: String): String = {
    counter.inc()
    value
  }
}
```
{% endtab %}
{% endtabs %}

#### **计量器\(**Gauge**\)**

`Gauge`根据需要提供任何类型的值。 要使用`Gauge`，必须首先创建一个实现`org.apache.flink.metrics.Gauge`接口的类。 返回值的类型没有限制。 可以通过在`MetricGroup`上调用`gauge(String name，Gauge gauge)`来注册仪表。

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<String, String> {
  private transient int valueToExpose = 0;

  @Override
  public void open(Configuration config) {
    getRuntimeContext()
      .getMetricGroup()
      .gauge("MyGauge", new Gauge<Integer>() {
        @Override
        public Integer getValue() {
          return valueToExpose;
        }
      });
  }

  @Override
  public String map(String value) throws Exception {
    valueToExpose++;
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
new class MyMapper extends RichMapFunction[String,String] {
  @transient private var valueToExpose = 0

  override def open(parameters: Configuration): Unit = {
    getRuntimeContext()
      .getMetricGroup()
      .gauge[Int, ScalaGauge[Int]]("MyGauge", ScalaGauge[Int]( () => valueToExpose ) )
  }

  override def map(value: String): String = {
    valueToExpose += 1
    value
  }
}
```
{% endtab %}
{% endtabs %}

请注意，高爆会将公开的对象转换为String，这意味着需要一个有意义的`toString()`实现。

#### **直方图**

直方图测量长值的分布。 可以通过在`MetricGroup`上调用`histogram(String name, Histogram histogram)`来注册。

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<Long, Long> {
  private transient Histogram histogram;

  @Override
  public void open(Configuration config) {
    this.histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new MyHistogram());
  }

  @Override
  public Long map(Long value) throws Exception {
    this.histogram.update(value);
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[Long,Long] {
  @transient private var histogram: Histogram = _

  override def open(parameters: Configuration): Unit = {
    histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new MyHistogram())
  }

  override def map(value: Long): Long = {
    histogram.update(value)
    value
  }
}
```
{% endtab %}
{% endtabs %}

Flink没有提供默认实现`Histogram`，但提供了一个允许使用Codahale / DropWizard直方图的[Wrapper](https://github.com/apache/flink/blob/master/flink-metrics/flink-metrics-dropwizard/src/main/java/org/apache/flink/dropwizard/metrics/DropwizardHistogramWrapper.java)。要使用此包装，请在以下内容中添加以下依赖项`pom.xml`：

```markup
<dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-metrics-dropwizard</artifactId>
      <version>1.7.1</version>
</dependency>
```

然后你可以像这样注册一个Codahale / DropWizard直方图：

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<Long, Long> {
  private transient Histogram histogram;

  @Override
  public void open(Configuration config) {
    com.codahale.metrics.Histogram dropwizardHistogram =
      new com.codahale.metrics.Histogram(new SlidingWindowReservoir(500));

    this.histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new DropwizardHistogramWrapper(dropwizardHistogram));
  }
  
  @Override
  public Long map(Long value) throws Exception {
    this.histogram.update(value);
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[Long, Long] {
  @transient private var histogram: Histogram = _

  override def open(config: Configuration): Unit = {
    com.codahale.metrics.Histogram dropwizardHistogram =
      new com.codahale.metrics.Histogram(new SlidingWindowReservoir(500))
        
    histogram = getRuntimeContext()
      .getMetricGroup()
      .histogram("myHistogram", new DropwizardHistogramWrapper(dropwizardHistogram))
  }
  
  override def map(value: Long): Long = {
    histogram.update(value)
    value
  }
}
```
{% endtab %}
{% endtabs %}

#### **仪表**

仪表测量平均吞吐量。 可以使用`markEvent()`方法注册事件的发生。 可以使用`markEvent(long n)`方法注册多个事件同时发生。 可以通过在`MetricGroup`上调用`meter(String name，Meter meter)`来注册仪表。

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<Long, Long> {
  private transient Meter meter;

  @Override
  public void open(Configuration config) {
    this.meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new MyMeter());
  }

  @Override
  public Long map(Long value) throws Exception {
    this.meter.markEvent();
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[Long,Long] {
  @transient private var meter: Meter = _

  override def open(config: Configuration): Unit = {
    meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new MyMeter())
  }

  override def map(value: Long): Long = {
    meter.markEvent()
    value
  }
}
```
{% endtab %}
{% endtabs %}

Flink提供了一个允许使用Codahale / DropWizard表的[包装器](https://github.com/apache/flink/blob/master/flink-metrics/flink-metrics-dropwizard/src/main/java/org/apache/flink/dropwizard/metrics/DropwizardMeterWrapper.java)。要使用此包装，请在以下内容中添加以下依赖项`pom.xml`：

```markup
<dependency>
      <groupId>org.apache.flink</groupId>
      <artifactId>flink-metrics-dropwizard</artifactId>
      <version>1.7.1</version>
</dependency>
```

然后你可以像这样注册一个Codahale / DropWizard仪表：

{% tabs %}
{% tab title="Java" %}
```java
public class MyMapper extends RichMapFunction<Long, Long> {
  private transient Meter meter;

  @Override
  public void open(Configuration config) {
    com.codahale.metrics.Meter dropwizardMeter = new com.codahale.metrics.Meter();

    this.meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new DropwizardMeterWrapper(dropwizardMeter));
  }

  @Override
  public Long map(Long value) throws Exception {
    this.meter.markEvent();
    return value;
  }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
class MyMapper extends RichMapFunction[Long,Long] {
  @transient private var meter: Meter = _

  override def open(config: Configuration): Unit = {
    com.codahale.metrics.Meter dropwizardMeter = new com.codahale.metrics.Meter()
  
    meter = getRuntimeContext()
      .getMetricGroup()
      .meter("myMeter", new DropwizardMeterWrapper(dropwizardMeter))
  }

  override def map(value: Long): Long = {
    meter.markEvent()
    value
  }
}
```
{% endtab %}
{% endtabs %}

## 范围

每个指标都被分配一个标识符和一组键值对，在这些键值对下将报告度量。

标识符基于3个组件：注册指标时的用户定义名称、可选的用户定义范围和系统提供的范围。例如，如果`a.b`是系统范围，`c.d`是用户范围，`e`是名称，那么指标的标识符将是`a.b.c.d.e`。

通过在`conf/flink-conf.yaml`中设置`metrics.scope.delimiter`键，可以配置要用于标识符的分隔符（默认为）。

### 用户范围

你可以通过调用`MetricGroup#addGroup(String name)`，`MetricGroup#addGroup(int name)`或`Metric#addGroup(String key，String value)`来定义用户范围。 这些方法会影响`MetricGroup#getMetricIdentifier`和`MetricGroup#getScopeComponents`返回的内容。

{% tabs %}
{% tab title="Java" %}
```java
counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetrics")
  .counter("myCounter");

counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetricsKey", "MyMetricsValue")
  .counter("myCounter");
```
{% endtab %}

{% tab title="Scala" %}
```scala
counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetrics")
  .counter("myCounter")

counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetricsKey", "MyMetricsValue")
  .counter("myCounter")
```
{% endtab %}
{% endtabs %}

### 系统范围

系统范围包含有关度量标准的上下文信息，例如，它在哪个任务中注册或该任务属于哪个作业。

可以通过在`conf / flink-conf.yaml`中设置以下属性来配置应包含哪些上下文信息。 这些属性中的每一个都期望格式字符串可以包含常量（例如“taskmanager”）和变量（例如“”），它们将在运行时被替换。

* `metrics.scope.jm`
  * 默认值：&lt;host&gt; .jobmanager
  * 应用于作用域作业管理器的所有指标。
* `metrics.scope.jm.job`
  * 默认值：&lt;host&gt; .jobmanager.&lt;job\_name&gt;
  * 应用于作用于作业管理器和作业的所有度量标准。
* `metrics.scope.tm`
  * 默认值：&lt;host&gt; .taskmanager.&lt;tm\_id&gt;
  * 应用于作用于任务管理器的所有度量标准。
* `metrics.scope.tm.job`
  * 默认值：&lt;host&gt; .taskmanager.&lt;tm\_id&gt;.&lt;job\_name&gt;
  * 应用于作用于任务管理器和作业的所有度量标准。
* `metrics.scope.task`
  * 默认值：&lt;host&gt; .taskmanager.&lt;tm\_id&gt;.&lt;job\_name&gt;.&lt;task\_name&gt;.&lt;subtask\_index&gt;
  * 应用于作用于任务的所有指标。
* `metrics.scope.operator`
  * 默认值：&lt;host&gt; .taskmanager.&lt;tm\_id&gt;.&lt;job\_name&gt;.&lt;operator\_name&gt;.&lt;subtask\_index&gt;
  * 应用于作用域的所有指标。

变量的数量或顺序没有限制。 变量区分大小写。

操作符符度量标准的默认范围将生成类似于`localhost.taskmanager.1234.MyJob.MyOperator.0.MyMetric`的标识符

如果您还想包含任务名称但省略任务管理器信息，则可以指定以下格式：

`metrics.scope.operator: <host>.<job_name>.<task_name>.<operator_name>.<subtask_index>`

这可以创建标识符`localhost.MyJob.MySource_->_MyOperator.MyOperator.0.MyMetric`

请注意，对于此格式字符串，如果同时多次运行同一作业，则可能发生标识符冲突，这可能导致度量标准数据不一致。 因此，建议使用通过包含ID（例如）或通过为作业和运算符分配唯一名称来提供一定程度的唯一性的格式字符串。

### 所有变量列表

* JobManager: &lt;host&gt;
* TaskManager: &lt;host&gt;, &lt;tm\_id&gt;
* Job: &lt;job\_id&gt;, &lt;job\_name&gt;
* Task: &lt;task\_id&gt;, &lt;task\_name&gt;, &lt;task\_attempt\_id&gt;, &lt;task\_attempt\_num&gt;, &lt;subtask\_index&gt;
* Operator: &lt;operator\_id&gt;,&lt;operator\_name&gt;, &lt;subtask\_index&gt;

**要点：**对于Batch API，&lt;operator\_id&gt;始终等于&lt;task\_id&gt;。

### 用户变量

可以通过调用`MetricGroup＃addGroup(String key,String value)`来定义用户变量。 此方法会影响`MetricGroup#getMetricIdentifier`，`MetricGroup#getScopeComponents`和`MetricGroup#getAllVariables()`返回的内容。

**重要提示**：用户变量不能用于范围格式。

{% tabs %}
{% tab title="Java" %}
```java
counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetricsKey", "MyMetricsValue")
  .counter("myCounter");

```
{% endtab %}

{% tab title="Scala" %}
```scala
counter = getRuntimeContext()
  .getMetricGroup()
  .addGroup("MyMetricsKey", "MyMetricsValue")
  .counter("myCounter")
```
{% endtab %}
{% endtabs %}

## 报告

通过在`conf/flink-conf.yaml`中配置一个或多个报告器，可以将指标公开给外部系统。 这些报告将在每个工作和任务管理器启动时进行实例化。

* `metrics.reporter.<name>.<config>`: 报告名称为`<name>`

   的通用设置`<config>` 

* `metrics.reporter.<name>.class`: 报告类用于名称为`<name>`的报告。
* `metrics.reporter.<name>.interval`: 报告名称为`<name>`的报告间隔时间
* `metrics.reporter.<name>.scope.delimiter`: 用于名称`<name>`的报告标识符（默认值使用`metrics.scope.delimiter`）的分隔符。
* `metrics.reporters`:\(可选）以逗号分隔的包含报告名称列表。默认情况下，将使用所有已配置的报告。

所有报告必须至少拥有类属性，有些允许指定报告间隔。 下面，我们将列出针对每位记者的更多设置。

示例报表配置，指定多个报告：

```text
metrics.reporters: my_jmx_reporter,my_other_reporter

metrics.reporter.my_jmx_reporter.class: org.apache.flink.metrics.jmx.JMXReporter
metrics.reporter.my_jmx_reporter.port: 9020-9040

metrics.reporter.my_other_reporter.class: org.apache.flink.metrics.graphite.GraphiteReporter
metrics.reporter.my_other_reporter.host: 192.168.1.1
metrics.reporter.my_other_reporter.port: 10000
```

**重要说明：**启动Flink时，通过将其放在/ lib文件夹中，可以访问包含报告者的jar。

您可以通过实现`org.apache.flink.metrics.reporter.MetricReporter`接口编写自己的`Reporter`。如果想Reporter定期发送报告，您还必须实现`Scheduled`接口。

以下部分列出了支持的报告类型：  


### JMX（org.apache.flink.metrics.jmx.JMXReporter）

不必包含其他依赖项，因为默认情况下JMX报告器可用但未激活。

参数：

* `port` - （可选）JMX侦听连接的端口。为了能够在一个主机上运行多个报告实例（例如，当一个TaskManager与JobManager共处时），建议使用类似的端口范围`9250-9260`。指定范围时，实际端口将显示在相关作业或任务管理器日志中。如果设置此设置，Flink将为给定的端口/范围启动额外的JMX连接器。指标始终在默认的本地JMX界面上可用。

配置示例：

```text
metrics.reporter.jmx.class: org.apache.flink.metrics.jmx.JMXReporter
metrics.reporter.jmx.port: 8789
```

通过JMX公开的指标由域和键属性列表标识，这些键属性一起形成对象名称。

域始终以`org.apache.flink`广义指标标识符开头。与通常的标识符相比，它不受范围格式的影响，不包含任何变量，并且在作业中保持不变。这种域的一个例子是`org.apache.flink.job.task.numBytesOut`。

键属性列表包含与给定指标关联的所有变量的值，无论配置的范围格式如何。这样一个列表的一个例子是`host=localhost,job_name=MyJob,task_name=MyTask`。

因此，域标识指标类，而关键属性列表标识该指标的一个（或多个）实例。

### Graphite（org.apache.flink.metrics.graphite.GraphiteReporter）

要使用此报告，必须复制`/opt/flink-metrics-graphite-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

参数：

* `host` - Graphite服务器主机
* `port` - Graphite服务器端口
* `protocol` - 使用协议（TCP / UDP）

配置示例：

```text
metrics.reporter.grph.class: org.apache.flink.metrics.graphite.GraphiteReporter
metrics.reporter.grph.host: localhost
metrics.reporter.grph.port: 2003
metrics.reporter.grph.protocol: TCP
```

### Prometheus \(org.apache.flink.metrics.prometheus.PrometheusReporter\)

要使用此报告，必须复制`/opt/flink-metrics-prometheus_2.11-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

参数：

* `port`- （可选）Prometheus导出器侦听的端口，默认为[9249](https://github.com/prometheus/prometheus/wiki/Default-port-allocations)。为了能够在一个主机上运行多个报告实例（例如，当一个TaskManager与JobManager共处时），建议使用类似的端口范围`9250-9260`。

配置示例：

```text
metrics.reporter.prom.class: org.apache.flink.metrics.prometheus.PrometheusReporter
```

Flink指标类型映射到Prometheus指标类型，如下所示：

| Flink | Prometheus | Note |
| :--- | :--- | :--- |
| Counter | Gauge | Prometheus计数器不能减少。 |
| Gauge | Gauge | 仅支持数字和布尔值。 |
| Histogram | Summary | 分位数 .5, .75, .95, .98, .99 and .999 |
| Meter | Gauge | 仪表输出仪表的速率。 |

所有Flink指标变量（请参阅[所有变量列表](https://ci.apache.org/projects/flink/flink-docs-release-1.7/monitoring/metrics.html#list-of-all-variables)）都将作为标签导出到Prometheus。

### PrometheusPushGateway \(org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter\)

要使用此报告，必须复制`/opt/flink-metrics-prometheus-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

参数：

| Key | Default | Description |
| :--- | :--- | :--- |
| **deleteOnShutdown** | true | 指定是否在关闭时从PushGateway中删除指标。 |
| **host** | \(none\) | PushGateway服务器主机。 |
| **jobName** | \(none\) | 将推送指标的作业名称 |
| **port** | -1 | PushGateway服务器端口。 |
| **randomJobNameSuffix** | true | 指定是否应将随机后缀附加到作业名称。 |

配置示例：

```text
metrics.reporter.promgateway.class: org.apache.flink.metrics.prometheus.PrometheusPushGatewayReporter
metrics.reporter.promgateway.host: localhost
metrics.reporter.promgateway.port: 9091
metrics.reporter.promgateway.jobName: myJob
metrics.reporter.promgateway.randomJobNameSuffix: true
metrics.reporter.promgateway.deleteOnShutdown: false
```

PrometheusPushGatewayReporter将指标推送到[Pushgateway](https://github.com/prometheus/pushgateway)，可由Prometheus [抓取](https://github.com/prometheus/pushgateway)。

有关用例，请参阅[Prometheus文档](https://prometheus.io/docs/practices/pushing/)。

### StatsD \(org.apache.flink.metrics.statsd.StatsDReporter\)

要使用此报告，必须复制`/opt/flink-metrics-statsd-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

参数：

* `host` - StatsD服务器主机
* `port` - StatsD服务器端口

配置示例：

```text
metrics.reporter.stsd.class: org.apache.flink.metrics.statsd.StatsDReporter
metrics.reporter.stsd.host: localhost
metrics.reporter.stsd.port: 8125
```

### Datadog \(org.apache.flink.metrics.datadog.DatadogHttpReporter\)

要使用此报告，必须复制`/opt/flink-metrics-datadog-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

注意Flink指标中的任何变量，如`<host>`，`<job_name>`，`<tm_id>`，`<subtask_index>`，`<task_name>`，和`<operator_name>`，都将作为标签发送到Datadog。标签看起来像`host:localhost`和`job_name:myjobname`。

参数：

* `apikey` - Datadog API密钥
* `tags` - （可选）发送到Datadog时将应用于指标的全局标记。标签应仅以逗号分隔

配置示例：

```text
metrics.reporter.dghttp.class: org.apache.flink.metrics.datadog.DatadogHttpReporter
metrics.reporter.dghttp.apikey: xxx
metrics.reporter.dghttp.tags: myflinkapp,prod
```

### Slf4j \(org.apache.flink.metrics.slf4j.Slf4jReporter\)

要使用此报告，您必须复制`/opt/flink-metrics-slf4j-1.7.2.jar`到`/lib`Flink发行版的文件夹中。

配置示例：

```text
metrics.reporter.slf4j.class: org.apache.flink.metrics.slf4j.Slf4jReporter
metrics.reporter.slf4j.interval: 60 SECONDS
```

## 系统指标

### CPU

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| Job-/TaskManager | Status.JVM.CPU | Load | JVM最近的CPU使用情况。 | Gauge |
|  |  | Time | JVM使用的CPU时间。 | Gauge |

### 内存

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| Job-/TaskManager | Status.JVM.Memory | Heap.Used | 当前使用的堆内存量（以字节为单位） | Gauge |
|  |  | Heap.Committed | 保证可供JVM使用的堆内存量（以字节为单位）。 | Gauge |
|  |  | Heap.Max | 可用于内存管理的最大堆内存量（以字节为单位）。 | Gauge |
|  |  | NonHeap.Used | 当前使用的非堆内存量（以字节为单位）。 | Gauge |
|  |  | NonHeap.Committed | 保证JVM可用的非堆内存量（以字节为单位）。 | Gauge |
|  |  | NonHeap.Max | 可用于内存管理的最大非堆内存量（以字节为单位）。 | Gauge |
|  |  | Direct.Count | 直接缓冲池中的缓冲区数。 | Gauge |
|  |  | Direct.MemoryUsed | JVM用于直接缓冲池的内存量（以字节为单位）。 | Gauge |
|  |  | Direct.TotalCapacity | 直接缓冲池中所有缓冲区的总容量（以字节为单位）。 | Gauge |
|  |  | Mapped.Count | 映射缓冲池中的缓冲区数。 | Gauge |
|  |  | Mapped.MemoryUsed | JVM用于映射缓冲池的内存量（以字节为单位）。 | Gauge |
|  |  | Mapped.TotalCapacity | 映射缓冲池中的缓冲区数（以字节为单位）。 | Gauge |

### 线程

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | Status.JVM.Threads | Count | 活动线程的总数。 | Gauge |

### 垃圾回收

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | Status.JVM.GarbageCollector | &lt;GarbageCollector&gt;.Count | 已发生的集合总数。 | Gauge |
|  |  | &lt;GarbageCollector&gt;.Time | 执行垃圾收集所花费的总时间 | Gauge |

### 类加载器

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | Status.JVM.ClassLoader | ClassesLoaded | 自JVM启动以来加载的类总数。 | Gauge |
|  |  | ClassesUnloaded | 自JVM启动以来卸载的类的总数。 | Gauge |

### 网络

| Scope | Infix | Metrics | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| **TaskManager** | Status.Network | AvailableMemorySegments | 未使用的内存片段数。 | Gauge |
|  |  | TotalMemorySegments | 已分配的内存片段数。 | Gauge |
| Task | buffers | inputQueueLength | 排队输入缓冲区的数量。 | Gauge |
|  |  | outputQueueLength | 排队输出缓冲区的数量。 | Gauge |
|  |  | inPoolUsage | 估计输入缓冲区的使用情况。 | Gauge |
|  |  | outPoolUsage | 估计输出缓冲区的使用情况。 | Gauge |
|  | Network.&lt;Input\|Output&gt;.&lt;gate&gt; **\(仅在设置了taskmanager.net.detailed-metrics配置选项时可用\)** | totalQueueLen | 所有输入/输出通道中排队缓冲区的总数。 | Gauge |
|  |  | minQueueLen | 所有输入/输出通道中的最小排队缓冲区数。 | Gauge |
|  |  | maxQueueLen | 所有输入/输出通道中的最大排队缓冲区数。 | Gauge |
|  |  | avgQueueLen | 所有输入/输出通道中的平均缓冲区数。 | Gauge |

### 集群

| Scope | Metrics | Description | Type |
| :--- | :--- | :--- | :--- |
| **JobManager** | numRegisteredTaskManagers | 注册任务管理员的数量。 | Gauge |
|  | numRunningJobs | 正在运行的作业数量。 | Gauge |
|  | taskSlotsAvailable | 可用任务槽的数量。 | Gauge |
|  | taskSlotsTotal | 任务槽的总数。 | Gauge |

### 可用性

<table>
  <thead>
    <tr>
      <th style="text-align:left">Scope</th>
      <th style="text-align:left">Metrics</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Job&#xFF08;&#x4EC5;&#x9002;&#x7528;&#x4E8E;JobManager&#xFF09;</b>
      </td>
      <td style="text-align:left">restartingTime</td>
      <td style="text-align:left">&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x4F5C;&#x4E1A;&#x6240;&#x82B1;&#x8D39;&#x7684;&#x65F6;&#x95F4;&#xFF0C;&#x6216;&#x5F53;&#x524D;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x7684;&#x6301;&#x7EED;&#x65F6;&#x95F4;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</td>
      <td
      style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">uptime</td>
      <td style="text-align:left">
        <p>&#x4F5C;&#x4E1A;&#x8FD0;&#x884C;&#x7684;&#x65F6;&#x95F4;&#x6CA1;&#x6709;&#x4E2D;&#x65AD;&#x3002;</p>
        <p>&#x5BF9;&#x4E8E;&#x5DF2;&#x5B8C;&#x6210;&#x7684;&#x4F5C;&#x4E1A;&#xFF0C;&#x8FD4;&#x56DE;-1&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</p>
      </td>
      <td style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">downtime</td>
      <td style="text-align:left">
        <p>&#x5BF9;&#x4E8E;&#x5F53;&#x524D;&#x5904;&#x4E8E;&#x6545;&#x969C;/&#x6062;&#x590D;&#x72B6;&#x6001;&#x7684;&#x4F5C;&#x4E1A;&#xFF0C;&#x5728;&#x6B64;&#x4E2D;&#x65AD;&#x671F;&#x95F4;&#x7ECF;&#x8FC7;&#x7684;&#x65F6;&#x95F4;&#x3002;</p>
        <p>&#x5BF9;&#x4E8E;&#x6B63;&#x5728;&#x8FD0;&#x884C;&#x7684;&#x4F5C;&#x4E1A;&#x8FD4;&#x56DE;0&#xFF0C;&#x5BF9;&#x4E8E;&#x5DF2;&#x5B8C;&#x6210;&#x7684;&#x4F5C;&#x4E1A;&#x8FD4;&#x56DE;-1&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</p>
      </td>
      <td style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">fullRestarts</td>
      <td style="text-align:left">&#x81EA;&#x63D0;&#x4EA4;&#x6B64;&#x4F5C;&#x4E1A;&#x4EE5;&#x6765;&#x5B8C;&#x5168;&#x91CD;&#x65B0;&#x542F;&#x52A8;&#x7684;&#x603B;&#x6B21;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">Gauge</td>
    </tr>
  </tbody>
</table>

### 检查点

| Scope | Metrics | Description | Type |
| :--- | :--- | :--- | :--- |
| **Job（仅适用于JobManager）** | lastCheckpointDuration | 完成最后一个检查点所花费的时间（以毫秒为单位）。 | Gauge |
|  | lastCheckpointSize | 最后一个检查点的总大小（以字节为单位）。 | Gauge |
|  | lastCheckpointExternalPath | 存储最后一个外部检查点的路径。 | Gauge |
|  | lastCheckpointRestoreTimestamp | 在协调器上恢复最后一个检查点的时间戳（以毫秒为单位）。 | Gauge |
|  | lastCheckpointAlignmentBuffered | 在最后一个检查点的所有子任务上进行对齐期间的缓冲字节数（以字节为单位）。 | Gauge |
|  | numberOfInProgressCheckpoints | 进行中检查点的数量。 | Gauge |
|  | numberOfCompletedCheckpoints | 成功完成检查点的数量。 | Gauge |
|  | numberOfFailedCheckpoints | 失败检查点的数量。 | Gauge |
|  | totalNumberOfCheckpoints | 总检查点的数量（正在进行，已完成，失败）。 | Gauge |
| Task | checkpointAlignmentTime | 最后一个障碍对齐完成所花费的时间（以纳秒为单位），或当前对齐到目前为止所用的时间（以纳秒为单位）。 | Gauge |

### RocksDB

某些RocksDB本机指标是可用的，但默认情况下已禁用，您可以[在此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html#rocksdb-native-metrics)找到完整文档

### IO

<table>
  <thead>
    <tr>
      <th style="text-align:left">Scope</th>
      <th style="text-align:left">Metrics</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Type</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Job (&#x4EC5;&#x5728; TaskManager&#x4E0A;&#x53EF;&#x7528;)</b>
      </td>
      <td style="text-align:left">&lt;SOURCE_ID&gt; &lt;source_subtask_index&gt; &lt;operator_id&gt; &lt;operator_subtask_index&gt;
        .latency</td>
      <td style="text-align:left">&#x4ECE;&#x7ED9;&#x5B9A;&#x6E90;&#x5B50;&#x4EFB;&#x52A1;&#x5230;&#x8FD0;&#x7B97;&#x7B26;&#x5B50;&#x4EFB;&#x52A1;&#x7684;&#x5EF6;&#x8FDF;&#x5206;&#x5E03;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</td>
      <td
      style="text-align:left">&#x76F4;&#x65B9;&#x56FE;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Task</b>
      </td>
      <td style="text-align:left">numBytesInLocal</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x4ECE;&#x672C;&#x5730;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x603B;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBytesInLocalPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x4ECE;&#x672C;&#x5730;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBytesInRemote</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x4ECE;&#x8FDC;&#x7A0B;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x603B;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBytesInRemotePerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x4ECE;&#x8FDC;&#x7A0B;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersInLocal</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x4ECE;&#x672C;&#x5730;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x603B;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersInLocalPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x4ECE;&#x672C;&#x5730;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersInRemote</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x4ECE;&#x8FDC;&#x7A0B;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x603B;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersInRemotePerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x4ECE;&#x8FDC;&#x7A0B;&#x6E90;&#x8BFB;&#x53D6;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBytesOut</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x5DF2;&#x53D1;&#x51FA;&#x7684;&#x603B;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBytesOutPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x53D1;&#x51FA;&#x7684;&#x5B57;&#x8282;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersOut</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x5DF2;&#x53D1;&#x51FA;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x603B;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numBuffersOutPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x53D1;&#x51FA;&#x7684;&#x7F51;&#x7EDC;&#x7F13;&#x51B2;&#x533A;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Task/Operator</b>
      </td>
      <td style="text-align:left">numRecordsIn</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x5DF2;&#x6536;&#x5230;&#x7684;&#x8BB0;&#x5F55;&#x603B;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numRecordsInPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x63A5;&#x6536;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numRecordsOut</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x5DF2;&#x53D1;&#x51FA;&#x7684;&#x8BB0;&#x5F55;&#x603B;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numRecordsOutPerSecond</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x6BCF;&#x79D2;&#x53D1;&#x9001;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x4EEA;&#x8868;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numLateRecordsDropped</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x56E0;&#x8FDF;&#x5230;&#x800C;&#x4E22;&#x5931;&#x7684;&#x8BB0;&#x5F55;&#x6570;&#x3002;</td>
      <td
      style="text-align:left">&#x8BA1;&#x6570;&#x5668;</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">currentInputWatermark</td>
      <td style="text-align:left">
        <p>&#x6B64;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#x6536;&#x5230;&#x7684;&#x200B;&#x200B;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x6C34;&#x5370;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x5177;&#x6709;2&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x64CD;&#x4F5C;&#x7B26;/&#x4EFB;&#x52A1;&#xFF0C;&#x8FD9;&#x662F;&#x6700;&#x540E;&#x6536;&#x5230;&#x7684;&#x6C34;&#x5370;&#x7684;&#x6700;&#x5C0F;&#x503C;&#x3002;</p>
      </td>
      <td style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Operator</b>
      </td>
      <td style="text-align:left">currentInput1Watermark</td>
      <td style="text-align:left">
        <p>&#x6B64;&#x64CD;&#x4F5C;&#x7B26;&#x5728;&#x5176;&#x7B2C;&#x4E00;&#x4E2A;&#x8F93;&#x5165;&#x4E2D;&#x63A5;&#x6536;&#x7684;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x6C34;&#x5370;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x5177;&#x6709;2&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x64CD;&#x4F5C;&#x7B26;&#x3002;</p>
      </td>
      <td style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">currentInput2Watermark</td>
      <td style="text-align:left">
        <p>&#x6B64;&#x64CD;&#x4F5C;&#x7B26;&#x5728;&#x5176;&#x7B2C;&#x4E8C;&#x4E2A;&#x8F93;&#x5165;&#x4E2D;&#x63A5;&#x6536;&#x7684;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x6C34;&#x5370;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x5177;&#x6709;2&#x4E2A;&#x8F93;&#x5165;&#x7684;&#x64CD;&#x4F5C;&#x7B26;&#x3002;</p>
      </td>
      <td style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">currentOutputWatermark</td>
      <td style="text-align:left">&#x6B64;&#x64CD;&#x4F5C;&#x7B26;&#x53D1;&#x51FA;&#x7684;&#x6700;&#x540E;&#x4E00;&#x4E2A;&#x6C34;&#x5370;&#xFF08;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#x3002;</td>
      <td
      style="text-align:left">Gauge</td>
    </tr>
    <tr>
      <td style="text-align:left"></td>
      <td style="text-align:left">numSplitsProcessed</td>
      <td style="text-align:left">&#x6B64;&#x6570;&#x636E;&#x6E90;&#x5DF2;&#x5904;&#x7406;&#x7684;InputSplits&#x603B;&#x6570;&#xFF08;&#x5982;&#x679C;&#x8FD0;&#x7B97;&#x7B26;&#x662F;&#x6570;&#x636E;&#x6E90;&#xFF09;&#x3002;</td>
      <td
      style="text-align:left">Gauge</td>
    </tr>
  </tbody>
</table>

### 连接器

#### **Kafka连接器**

| Scope | Metrics | User Variables | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| Operator | commitsSucceeded | n/a | 如果启用了偏移提交并且启用了检查点，则是成功向Kafka提交的偏移提交总数。 | 计数器 |
| Operator | commitsFailed | n/a | 如果启用了偏移提交并且启用了检查点，则是Kafka的偏移提交失败总数。请注意，将偏移量提交回Kafka只是暴露消费者进度的一种方法，因此提交失败不会影响Flink的检查点分区偏移的完整性。 | 计数器 |
| Operator | committedOffsets | topic, partition | 对于每个分区，最后成功提交到Kafka的偏移量。可以通过主题名称和分区ID指定特定分区的度量标准。 | Gauge |
| Operator | currentOffsets | topic, partition | 消费者对每个分区的当前读取偏移量。可以通过主题名称和分区ID指定特定分区的度量标准。 | Gauge |

#### **Kinesis连接器**

| Scope | Metrics | User Variables | Description | Type |
| :--- | :--- | :--- | :--- | :--- |
| Operator | millisBehindLatest | stream，shardId | 消费者在流的头部后面的毫秒数，表示每个Kinesis分片的消费者当前时间落后多少。 可以通过流名称和分片ID指定特定分片的指标。 值为0表示记录处理被捕获，此时没有要处理的新记录。 值-1表示该指标尚未报告。 | Gauge |
| Operator | sleepTimeMillis | stream，shardId | 消费者在从Kinesis获取记录之前花费的毫秒数。可以通过流名称和分片ID指定特定分片的**指标**。 | Gauge |
| Operator | maxNumberOfRecordsPerFetch | stream，shardId | 消费者在单个getRecords调用Kinesis时请求的最大记录数。如果ConsumerConfigConstants.SHARD\_USE\_ADAPTIVE\_READS设置为true，则自适应地计算此值以最大化Kinesis的2 Mbps读取限制。 | Gauge |
| Operator | numberOfAggregatedRecordsPerFetch | stream，shardId | 消费者在单个getRecords调用Kinesis时获取的聚合Kinesis记录数。 | Gauge |
| Operator | numberOfDeggregatedRecordsPerFetch | stream，shardId | 消费者在一次getRecords调用Kinesis时获取的分解Kinesis记录的数量。 | Gauge |
| Operator | averageRecordSizeBytes | stream，shardId | Kinesis记录的平均大小（以字节为单位），由消费者在单个getRecords调用中获取。 | Gauge |
| Operator | runLoopTimeNanos | stream，shardId | 消费者在运行循环中花费的实际时间（以纳秒为单位） | Gauge |
| Operator | loopFrequencyHz | stream，shardId | 一秒钟内调用getRecords的次数。 | Gauge |
| Operator | bytesRequestedPerFetch | stream，shardId | 在一次调用getRecords中请求的字节数（2 Mbps / loopFrequencyHz）。 | Gauge |

### 系统资源

默认情况下禁用系统资源报告。 启用`metrics.system-resource`时，下面列出的其他度量标准将在`Job-`和`TaskManager`上可用。 系统资源度量标准会定期更新，并显示已配置时间间隔的平均值（`metrics.system-resource-probing-interval`）。

系统资源报告要求在类路径上存在可选的依赖项（例如，放在Flink的`lib`目录中）：

* `com.github.oshi:oshi-core:3.4.0` （根据EPL 1.0许可证授权）

包括它的传递依赖：

* `net.java.dev.jna:jna-platform:jar:4.2.2`
* `net.java.dev.jna:jna:jar:4.2.2`

这方面的失败将被报告为启动期间`NoClassDefFoundError` 记录的警告消息`SystemResourcesMetricsInitializer`。

#### 系统CPU

| Scope | Infix | Metrics | Description |
| :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | System.CPU | Usage | CPU使用率的总体百分比。 |
|  |  | Idle | CPU空闲使用率的百分比。 |
|  |  | Sys | 系统CPU使用率的百分比。 |
|  |  | User | 用户CPU使用率的百分比 |
|  |  | IOWait | IOWait CPU使用率的百分比。 |
|  |  | Irq | Irq CPU使用率的百分比。 |
|  |  | SoftIrq | SoftIrq CPU使用率的百分比。 |
|  |  | Nice | 空闲使用百分比。 |
|  |  | Load1min | 平均CPU负载超过1分钟 |
|  |  | Load5min | 平均CPU负载超过5分钟 |
|  |  | Load15min | 平均CPU负载超过15分钟 |
|  |  | UsageCPU**\*** | 每个处理器的CPU使用率百分比 |

#### **系统内存**

| Scope | Infix | Metrics | Description |
| :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | System.Memory | Available | 可用内存以字节为单位 |
|  |  | Total | 总内存（字节） |
|  | System.Swap | Used | 使用的交换字节 |
|  |  | Total | 总交换字节数 |

#### **系统网络**

| Scope | Infix | Metrics | Description |
| :--- | :--- | :--- | :--- |
| **Job-/TaskManager** | System.Network.INTERFACE\_NAME | ReceiveRate | 平均接收速率，以每秒字节数为单位 |
|  |  | SendRate | 平均发送速率，以字节/秒为单位 |

## 延迟跟踪

Flink允许跟踪通过系统传输的记录的延迟。默认情况下禁用此功能。要启用延迟跟踪，必须在Flink配置或`ExecutionConfig`中将`latencyTrackingInterval`设置为正数。

在`latencyTrackingInterval`中，源将定期发出一个称为`LatencyMarker`的特殊记录。标记包含从源处发出记录的时间戳。延迟标记不能超过常规用户记录，因此如果记录在操作符面前排队，则会增加标记跟踪的延迟。

请注意，延迟标记不会考虑用户记录在绕过它们时在操作符中花费的时间。特别是标记不考虑记录在窗口缓冲区中花费的时间。只有当操作符无法接受新记录，致使他们排队时，使用标记测量的延迟才会反映出来。

`LatencyMarkers`用于导出拓扑源和每个下游操作符之间的延迟分布。这些分布报告为直方图度量。这些发行版的粒度可以在\[Flink配置\]中控制（// ci.apache.org/projects/flink/flink-docs-release-1.7/ops/config.html\#metrics-latency-interval。最高粒度子任务Flink将导出每个源子任务和每个下游子任务之间的延迟分布，这导致二次（在并行性方面）直方图的数量。

目前，Flink假定群集中所有计算机的时钟都是同步的。我们建议设置自动时钟同步服务（如NTP）以避免错误的延迟结果。

{% hint style="danger" %}
警告：启用延迟指标可能会显著影响群集的性能（特别是对于`subtask`粒度）。强烈建议仅将它们用于调试目的。
{% endhint %}

## REST API集成

可以通过[Monitoring REST API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/monitoring/rest_api.html)查询指标。

下面是可用端点列表，带有示例JSON响应。所有端点都是样本表单`http://hostname:8081/jobmanager/metrics`，下面我们只列出URL 的_路径_部分。

尖括号中的值是变量，例如`http://hostname:8081/jobs/<jobid>/metrics`必须请求例如`http://hostname:8081/jobs/7684be6004e4e955c2a558a9bc463f65/metrics`。

请求特定实体的指标：

* `/jobmanager/metrics`
* `/taskmanagers/<taskmanagerid>/metrics`
* `/jobs/<jobid>/metrics`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/<subtaskindex>`

请求在相应类型的所有实体之间聚合的指标：

* `/taskmanagers/metrics`
* `/jobs/metrics`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/metrics`

请求在相应类型的所有实体的子集上聚合的指标：

* `/taskmanagers/metrics?taskmanagers=A,B,C`
* `/jobs/metrics?jobs=D,E,F`
* `/jobs/<jobid>/vertices/<vertexid>/subtasks/metrics?subtask=1,2,3`

请求可用指标列表：

`GET /jobmanager/metrics`

```text
[
  {
    "id": "metric1"
  },
  {
    "id": "metric2"
  }
]
```

请求特定（未聚合）指标的值：

`GET taskmanagers/ABCDE/metrics?get=metric1,metric2`

```text
[
  {
    "id": "metric1",
    "value": "34"
  },
  {
    "id": "metric2",
    "value": "2"
  }
]
```

请求特定指标的汇总值：

`GET /taskmanagers/metrics?get=metric1,metric2`

```text
[
  {
    "id": "metric1",
    "min": 1,
    "max": 34,
    "avg": 15,
    "sum": 45
  },
  {
    "id": "metric2",
    "min": 2,
    "max": 14,
    "avg": 7,
    "sum": 16
  }
]
```

请求特定指标的特定聚合值：

`GET /taskmanagers/metrics?get=metric1,metric2&agg=min,max`

```text
[
  {
    "id": "metric1",
    "min": 1,
    "max": 34,
  },
  {
    "id": "metric2",
    "min": 2,
    "max": 14,
  }
]
```

## 仪表盘集成

为每个任务或操作符收集的指标也可以在仪表板中显示。在作业的主页面上，选择`Metrics`选项卡。选择顶部图表中的一个任务后，您可以使用`Add Metric`下拉菜单选择要显示的指标。

* 任务指标列为`<subtask_index>.<metric_name>`。
* 操作符指标列为`<subtask_index>.<operator_name>.<metric_name>`。

每个指标将可视化为单独的图形，x轴表示时间，y轴表示测量值。所有图表每10秒自动更新一次，并在导航到另一页时继续更新。

可视化指标的数量没有限制; 但是只能显示数字指标。

