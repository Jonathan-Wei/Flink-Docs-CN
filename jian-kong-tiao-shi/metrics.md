# Metrics

Flink公开了一个指标系统，允许收集和公开指标到外部系统。

## 注册指标

### 指标类型

Flink支持`Counters`，`Gauges`，`Histograms`和`Meters`。

#### **计数器**

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

#### **计量器**

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

#### **直方图**

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

### 用户范围

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

### 所有变量列表

### 用户变量

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
</table>### 检查点

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



### IO

### 连接器

### 系统资源

## 延迟跟踪

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

