# 流式聚合

SQL是用于数据分析的最广泛使用的语言。Flink的Table API和SQL使用户能够以更少的时间和精力定义高效的流分析应用程序。而且，Flink Table API和SQL得到了有效的优化，它集成了许多查询优化和优化的运算符实现。但是并非默认情况下会启用所有优化，因此对于某些工作负载，可以通过打开某些选项来提高性能。

在此页面中，我们将介绍一些有用的优化选项以及流聚合的内部原理，这将在某些情况下带来很大的改进。

{% hint style="danger" %}
注意：当前，仅Blink计划程序支持此页面中提到的优化选项。
{% endhint %}

{% hint style="danger" %}
注意：当前，仅对无边界聚合支持流聚合优化。将来将支持窗口聚合的优化。
{% endhint %}

默认情况下，无界聚合运算符一个一个地处理输入记录，即（1）从状态读取累加器，（2）将记录累加/放回到累加器，（3）将累加器写回到状态，（4）下一条记录将从（1）重新进行处理。此处理模式可能会增加StateBackend的开销（尤其是对于RocksDB StateBackend）。此外，生产中非常常见的数据偏斜会使问题恶化，并使工作容易承受背压情况。

## 小批量聚合

微批处理聚合的核心思想是将一组输入缓存在聚合运算符内部的缓冲区中。当触发输入束以进行处理时，每个键只需一个操作即可访问状态。这样可以大大减少状态开销并获得更好的吞吐量。但是，这可能会增加一些延迟，因为它会缓冲一些记录而不是立即处理它们。这是吞吐量和延迟之间的权衡。

下图说明了小批量聚合如何减少状态操作。

![](../../../.gitbook/assets/image%20%287%29.png)

MiniBatch优化默认情况下处于禁用状态。为了使这种优化，您应该设置选项`table.exec.mini-batch.enabled`，`table.exec.mini-batch.allow-latency`和`table.exec.mini-batch.size`。请参阅[配置](https://ci.apache.org/projects/flink/flink-docs-release-1.9/dev/table/config.html#execution-options)页面以获取更多详细信息。

以下示例展示如何启用这些选项。

{% tabs %}
{% tab title="Java" %}
```java
// instantiate table environment
TableEnvironment tEnv = ...

tEnv.getConfig()        // access high-level configuration
  .getConfiguration()   // set low-level key-value options
  .setString("table.exec.mini-batch.enabled", "true")  // enable mini-batch optimization
  .setString("table.exec.mini-batch.allow-latency", "5 s") // use 5 seconds to buffer input records
  .setString("table.exec.mini-batch.size", "5000"); // the maximum number of records can be buffered by each aggregate operator task
```
{% endtab %}

{% tab title="Scala" %}
```scala
// instantiate table environment
val tEnv: TableEnvironment = ...

tEnv.getConfig         // access high-level configuration
  .getConfiguration    // set low-level key-value options
  .setString("table.exec.mini-batch.enabled", "true") // enable mini-batch optimization
  .setString("table.exec.mini-batch.allow-latency", "5 s") // use 5 seconds to buffer input records
  .setString("table.exec.mini-batch.size", "5000") // the maximum number of records can be buffered by each aggregate operator task
```
{% endtab %}

{% tab title="Python" %}
```python
# instantiate table environment
t_env = ...

t_env.get_config()        # access high-level configuration
  .get_configuration()    # set low-level key-value options
  .set_string("table.exec.mini-batch.enabled", "true") # enable mini-batch optimization
  .set_string("table.exec.mini-batch.allow-latency", "5 s") # use 5 seconds to buffer input records
  .set_string("table.exec.mini-batch.size", "5000"); # the maximum number of records can be buffered by each aggregate operator task
```
{% endtab %}
{% endtabs %}

## 本地全局聚合

提出使用本地全局来解决数据倾斜问题，方法是将组聚合分为两个阶段，首先在上游进行本地聚合，然后在下游进行全局聚合，这类似于MapReduce中的Combine + Reduce模式。例如以下SQL：

```text
SELECT color, sum(id)
FROM T
GROUP BY color
```

数据流中的记录可能会倾斜，因此聚合运算符的某些实例必须比其他实例处理更多的记录，这会导致热点。本地聚合可以帮助将具有相同密钥的一定数量的输入累积到单个累加器中。全局汇总将仅接收减少的累加器，而不是大量的原始输入。这可以大大减少网络改组和状态访问的成本。每次本地聚合累积的输入数量基于最小批处理间隔。这意味着本地-全局聚合依赖于小批量优化的启用。

下图显示了本地全局聚合如何提高性能。

![](../../../.gitbook/assets/image%20%288%29.png)

以下示例展示了如何启用本地全局聚合。

{% tabs %}
{% tab title="Java" %}
```java
// instantiate table environment
TableEnvironment tEnv = ...

tEnv.getConfig()        // access high-level configuration
  .getConfiguration()   // set low-level key-value options
  .setString("table.exec.mini-batch.enabled", "true")  // local-global aggregation depends on mini-batch is enabled
  .setString("table.exec.mini-batch.allow-latency", "5 s")
  .setString("table.exec.mini-batch.size", "5000")
  .setString("table.optimizer.agg-phase-strategy", "TWO_PHASE"); // enable two-phase, i.e. local-global aggregation
```
{% endtab %}

{% tab title="Scala" %}
```scala
// instantiate table environment
val tEnv: TableEnvironment = ...

tEnv.getConfig         // access high-level configuration
  .getConfiguration    // set low-level key-value options
  .setString("table.exec.mini-batch.enabled", "true") // local-global aggregation depends on mini-batch is enabled
  .setString("table.exec.mini-batch.allow-latency", "5 s")
  .setString("table.exec.mini-batch.size", "5000")
  .setString("table.optimizer.agg-phase-strategy", "TWO_PHASE") // enable two-phase, i.e. local-global aggregation
```
{% endtab %}

{% tab title="Python" %}
```python
# instantiate table environment
t_env = ...

t_env.get_config()        # access high-level configuration
  .get_configuration()    # set low-level key-value options
  .set_string("table.exec.mini-batch.enabled", "true") # local-global aggregation depends on mini-batch is enabled
  .set_string("table.exec.mini-batch.allow-latency", "5 s")
  .set_string("table.exec.mini-batch.size", "5000")
  .set_string("table.optimizer.agg-phase-strategy", "TWO_PHASE"); # enable two-phase, i.e. local-global aggregation
```
{% endtab %}
{% endtabs %}

## 拆分去重聚合

本地全局优化可有效消除常规聚合的数据偏斜，例如SUM，COUNT，MAX，MIN，AVG。但是，在处理不同的聚合时，其性能并不令人满意。

例如，如果我们要分析今天有多少唯一用户登录。我们可能有以下查询：

```text
SELECT day, COUNT(DISTINCT user_id)
FROM T
GROUP BY day
```

如果distinct key（即，user\_id）的值稀疏，则COUNT DISTINCT不适合减少记录。即使启用了局部全局优化，它也无济于事。因为累加器仍然包含几乎所有原始记录，并且全局聚合将成为瓶颈（大多数繁重的累加器由一项任务处理，即在同一天）。

这种优化的思想是将不同的聚合\(例如`COUNT(distinct col)`\)分成两个级别。第一个聚合由组键和一个附加的bucket键打乱。bucket键是使用`HASH_CODE(distinct t_key) % BUCKET_NUM`计算的。`BUCKET_NUM`默认为1024，可以通过`table.optimizer.distinct-agg.split.bucket-num`选项配置。第二个聚合由原始的组键打乱，并使用`SUM`来聚合来自不同bucket的不同值。因为相同的不同键只会在同一个bucket中计算，所以转换是等价的。bucket键充当附加的组键，以分担组键中hotspot的负担。bucket键使作业具有可伸缩性，可以解决不同聚合中的数据倾斜/热点问题。

拆分去重聚合后，上述查询将自动重写为以下查询：

```text
SELECT day, SUM(cnt)
FROM (
    SELECT day, COUNT(DISTINCT user_id) as cnt
    FROM T
    GROUP BY day, MOD(HASH_CODE(user_id), 1024)
)
GROUP BY day
```

下图显示了拆分去重聚合如何提高性能（假设颜色代表天，字母代表user\_id）。

![](../../../.gitbook/assets/image%20%2823%29.png)

注意:上面是最简单的例子，可以从这种优化中获益。除此之外，Flink还支持分割更复杂的聚合查询，例如，多个具有不同键的不同聚合\(如`COUNT(distinct a)、SUM(distinct b)`\)，以及其他非不同的聚合\(如`SUM`、`MAX`、`MIN`、`COUNT`\)。

{% hint style="danger" %}
注意：但是，当前，拆分优化不支持包含用户定义的AggregateFunction的聚合。
{% endhint %}

以下示例显示了如何启用拆分非重复聚合优化。

{% tabs %}
{% tab title="Java" %}
```java
// instantiate table environment
TableEnvironment tEnv = ...

tEnv.getConfig()        // access high-level configuration
  .getConfiguration()   // set low-level key-value options
  .setString("table.optimizer.distinct-agg.split.enabled", "true");  // enable distinct agg split
```
{% endtab %}

{% tab title="Scala" %}
```scala
// instantiate table environment
val tEnv: TableEnvironment = ...

tEnv.getConfig         // access high-level configuration
  .getConfiguration    // set low-level key-value options
  .setString("table.optimizer.distinct-agg.split.enabled", "true")  // enable distinct agg split
```
{% endtab %}

{% tab title="Python" %}
```python
# instantiate table environment
t_env = ...

t_env.get_config()        # access high-level configuration
  .get_configuration()    # set low-level key-value options
  .set_string("table.optimizer.distinct-agg.split.enabled", "true"); # enable distinct agg split
```
{% endtab %}
{% endtabs %}

## 在Distinct聚合上使用FILTER修饰符

在某些情况下，用户可能需要从不同维度计算UV（唯一访客）的数量，例如Android的UV，iPhone的UV，Web的UV和总UV。许多用户将选择`CASE WHEN`支持此功能，例如：

```text
SELECT
 day,
 COUNT(DISTINCT user_id) AS total_uv,
 COUNT(DISTINCT CASE WHEN flag IN ('android', 'iphone') THEN user_id ELSE NULL END) AS app_uv,
 COUNT(DISTINCT CASE WHEN flag IN ('wap', 'other') THEN user_id ELSE NULL END) AS web_uv
FROM T
GROUP BY day
```

但是，在这种情况下，建议使用`FILTER`语法而不是CASE WHEN。因为`FILTER`它更符合SQL标准，并且将获得更多的性能改进。 `FILTER`是在聚合函数上使用的修饰符，用于限制聚合中使用的值。将上面的示例替换为`FILTER`修饰符，如下所示：

```text
SELECT
 day,
 COUNT(DISTINCT user_id) AS total_uv,
 COUNT(DISTINCT user_id) FILTER (WHERE flag IN ('android', 'iphone')) AS app_uv,
 COUNT(DISTINCT user_id) FILTER (WHERE flag IN ('wap', 'other')) AS web_uv
FROM T
GROUP BY day
```

Flink SQL优化器可以识别同一唯一键上的不同过滤器参数。例如，在上面的示例中，所有三个COUNT DISTINCT都在`user_id`列上。然后Flink可以只使用一个共享状态实例，而不是三个状态实例，以减少状态访问和状态大小。在某些工作负载中，这可以显着提高性能。

