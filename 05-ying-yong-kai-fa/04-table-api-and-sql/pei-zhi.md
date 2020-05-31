# 配置

默认情况下，Table和SQL API是预先配置的，用于生成具有可接受性能的精确结果。

根据表程序的要求，可能需要调整某些参数以进行优化。例如，无界流程序可能需要确保所需的状态大小是有上限的（请参阅[流概念](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/streaming/query_configuration.html)）。

## 总览

在每个表环境中，TableConfig提供配置当前会话的选项。

对于常见的或重要的配置选项，TableConfig提供了带有详细内联文档的getter和setter方法。

对于更高级的配置，用户可以直接访问底层键-值映射。下面列出了所有可用于调整Flink表和SQL API程序的配置项。

{% hint style="danger" %}
**注意**：由于执行操作时会在不同的时间点读取选项，因此建议在实例化表环境后尽早设置配置选项。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```java
// instantiate table environment
TableEnvironment tEnv = ...

// access flink configuration
Configuration configuration = tEnv.getConfig().getConfiguration();
// set low-level key-value options
configuration.setString("table.exec.mini-batch.enabled", "true");
configuration.setString("table.exec.mini-batch.allow-latency", "5 s");
configuration.setString("table.exec.mini-batch.size", "5000");
```
{% endtab %}

{% tab title="Scala" %}
```scala
// instantiate table environment
val tEnv: TableEnvironment = ...

// access flink configuration
val configuration = tEnv.getConfig().getConfiguration()
// set low-level key-value options
configuration.setString("table.exec.mini-batch.enabled", "true")
configuration.setString("table.exec.mini-batch.allow-latency", "5 s")
configuration.setString("table.exec.mini-batch.size", "5000")
```
{% endtab %}

{% tab title="Python" %}
```python
# instantiate table environment
t_env = ...

# access flink configuration
configuration = t_env.get_config().get_configuration();
# set low-level key-value options
configuration.set_string("table.exec.mini-batch.enabled", "true");
configuration.set_string("table.exec.mini-batch.allow-latency", "5 s");
configuration.set_string("table.exec.mini-batch.size", "5000");
```
{% endtab %}
{% endtabs %}

{% hint style="danger" %}
注意：当前，仅Blink Planner支持键值配置项。
{% endhint %}

## 执行配置项

以下选项可用于调整查询执行的性能。

## 优化器配置项

以下选项可用于调整查询优化器的行为，以获得更好的执行计划。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x952E;</th>
      <th style="text-align:left">&#x9ED8;&#x8BA4;</th>
      <th style="text-align:left">&#x7C7B;&#x578B;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.agg-phase-strategy</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">&#x201C;&#x6C7D;&#x8F66;&#x201D;</td>
      <td style="text-align:left">&#x4E32;</td>
      <td style="text-align:left">&#x6C47;&#x603B;&#x9636;&#x6BB5;&#x7684;&#x7B56;&#x7565;&#x3002;&#x53EA;&#x80FD;&#x8BBE;&#x7F6E;AUTO&#xFF0C;TWO_PHASE&#x6216;ONE_PHASE&#x3002;&#x81EA;&#x52A8;&#xFF1A;&#x805A;&#x5408;&#x9636;&#x6BB5;&#x6CA1;&#x6709;&#x7279;&#x6B8A;&#x7684;&#x6267;&#x884C;&#x5668;&#x3002;&#x9009;&#x62E9;&#x4E24;&#x9636;&#x6BB5;&#x6C47;&#x603B;&#x8FD8;&#x662F;&#x4E00;&#x9636;&#x6BB5;&#x6C47;&#x603B;&#x53D6;&#x51B3;&#x4E8E;&#x6210;&#x672C;&#x3002;TWO_PHASE&#xFF1A;&#x5F3A;&#x5236;&#x4F7F;&#x7528;&#x5177;&#x6709;localAggregate&#x548C;globalAggregate&#x7684;&#x4E24;&#x9636;&#x6BB5;&#x805A;&#x5408;&#x3002;&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x5982;&#x679C;&#x805A;&#x5408;&#x8C03;&#x7528;&#x4E0D;&#x652F;&#x6301;&#x5206;&#x4E3A;&#x4E24;&#x9636;&#x6BB5;&#x7684;&#x4F18;&#x5316;&#xFF0C;&#x6211;&#x4EEC;&#x4ECD;&#x5C06;&#x4F7F;&#x7528;&#x4E00;&#x7EA7;&#x805A;&#x5408;&#x3002;ONE_PHASE&#xFF1A;&#x5F3A;&#x5236;&#x4F7F;&#x7528;&#x4EC5;&#x5177;&#x6709;CompleteGlobalAggregate&#x7684;&#x4E00;&#x7EA7;&#x805A;&#x5408;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.distinct-agg.split.bucket-num</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left">1024</td>
      <td style="text-align:left">&#x6574;&#x6570;</td>
      <td style="text-align:left">&#x62C6;&#x5206;&#x72EC;&#x7ACB;&#x805A;&#x5408;&#x65F6;&#x914D;&#x7F6E;&#x5B58;&#x50A8;&#x6876;&#x6570;&#x3002;&#x8BE5;&#x6570;&#x5B57;&#x5728;&#x7B2C;&#x4E00;&#x7EA7;&#x805A;&#x5408;&#x4E2D;&#x7528;&#x4E8E;&#x8BA1;&#x7B97;&#x5B58;&#x50A8;&#x6876;&#x5BC6;&#x94A5;&#x201C;
        hash_code&#xFF08;distinct_key&#xFF09;&#xFF05;BUCKET_NUM&#x201D;&#xFF0C;&#x8BE5;&#x5B58;&#x50A8;&#x6876;&#x5BC6;&#x94A5;&#x5728;&#x62C6;&#x5206;&#x540E;&#x7528;&#x4F5C;&#x9644;&#x52A0;&#x7EC4;&#x5BC6;&#x94A5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.distinct-agg.split.enabled</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">&#x5E03;&#x5C14;&#x578B;</td>
      <td style="text-align:left">&#x544A;&#x8BC9;&#x4F18;&#x5316;&#x5668;&#x662F;&#x5426;&#x5C06;&#x4E0D;&#x540C;&#x7684;&#x805A;&#x5408;&#xFF08;&#x4F8B;&#x5982;COUNT&#xFF08;DISTINCT
        col&#xFF09;&#xFF0C;SUM&#xFF08;DISTINCT col&#xFF09;&#xFF09;&#x5206;&#x6210;&#x4E24;&#x4E2A;&#x7EA7;&#x522B;&#x3002;&#x7B2C;&#x4E00;&#x6B21;&#x805A;&#x5408;&#x88AB;&#x4E00;&#x4E2A;&#x9644;&#x52A0;&#x5BC6;&#x94A5;&#x6539;&#x7EC4;&#xFF0C;&#x8BE5;&#x9644;&#x52A0;&#x5BC6;&#x94A5;&#x4F7F;&#x7528;distinct_key&#x7684;&#x54C8;&#x5E0C;&#x7801;&#x548C;&#x5B58;&#x50A8;&#x6876;&#x6570;&#x8BA1;&#x7B97;&#x5F97;&#x51FA;&#x3002;&#x5F53;&#x4E0D;&#x540C;&#x7684;&#x805A;&#x5408;&#x4E2D;&#x5B58;&#x5728;&#x6570;&#x636E;&#x503E;&#x659C;&#x65F6;&#xFF0C;&#x6B64;&#x4F18;&#x5316;&#x975E;&#x5E38;&#x6709;&#x7528;&#xFF0C;&#x5E76;&#x4E14;&#x53EF;&#x4EE5;&#x6269;&#x5927;&#x5DE5;&#x4F5C;&#x91CF;&#x3002;&#x9ED8;&#x8BA4;&#x4E3A;false&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.join-reorder-enabled</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">&#x5047;</td>
      <td style="text-align:left">&#x5E03;&#x5C14;&#x578B;</td>
      <td style="text-align:left">&#x5728;&#x4F18;&#x5316;&#x5668;&#x4E2D;&#x542F;&#x7528;&#x8054;&#x63A5;&#x91CD;&#x65B0;&#x6392;&#x5E8F;&#x3002;&#x9ED8;&#x8BA4;&#x8BBE;&#x7F6E;&#x4E3A;&#x7981;&#x7528;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.join.broadcast-threshold</b>
        </p>
        <p>
          <br />Batch</p>
      </td>
      <td style="text-align:left">1048576</td>
      <td style="text-align:left">&#x957F;</td>
      <td style="text-align:left">&#x914D;&#x7F6E;&#x8868;&#x7684;&#x6700;&#x5927;&#x5927;&#x5C0F;&#xFF08;&#x4EE5;&#x5B57;&#x8282;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#xFF0C;&#x8BE5;&#x8868;&#x5728;&#x6267;&#x884C;&#x8054;&#x63A5;&#x65F6;&#x5C06;&#x5E7F;&#x64AD;&#x5230;&#x6240;&#x6709;&#x5DE5;&#x4F5C;&#x7A0B;&#x5E8F;&#x8282;&#x70B9;&#x3002;&#x901A;&#x8FC7;&#x5C06;&#x6B64;&#x503C;&#x8BBE;&#x7F6E;&#x4E3A;-1&#x4EE5;&#x7981;&#x7528;&#x5E7F;&#x64AD;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">&lt;b&gt;&lt;/b&gt;</td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;&#x4E3A;true&#xFF0C;&#x5219;&#x4F18;&#x5316;&#x5668;&#x5C06;&#x5C1D;&#x8BD5;&#x627E;&#x51FA;&#x91CD;&#x590D;&#x7684;&#x8868;&#x6E90;&#x5E76;&#x91CD;&#x65B0;&#x4F7F;&#x7528;&#x5B83;&#x4EEC;&#x3002;&#x4EC5;&#x5F53;&#x542F;&#x7528;table.optimizer.reuse-sub-plan&#x4E3A;true&#x65F6;&#xFF0C;&#x6B64;&#x65B9;&#x6CD5;&#x624D;&#x6709;&#x6548;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>&#x5DF2;&#x542F;&#x7528;table.optimizer.reuse-sub-plan</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF; &#x6D41;</p>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6B63;&#x786E;&#x65F6;&#xFF0C;&#x4F18;&#x5316;&#x5668;&#x5C06;&#x5C1D;&#x8BD5;&#x627E;&#x51FA;&#x91CD;&#x590D;&#x7684;&#x5B50;&#x8BA1;&#x5212;&#x5E76;&#x91CD;&#x7528;&#x5B83;&#x4EEC;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.source.predicate-pushdown-enabled</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF; &#x6D41;</p>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;&#x4E3A;true&#xFF0C;&#x5219;&#x4F18;&#x5316;&#x5668;&#x4F1A;&#x5C06;&#x8C13;&#x8BCD;&#x4E0B;&#x63A8;&#x5230;FilterableTableSource&#x4E2D;&#x3002;&#x9ED8;&#x8BA4;&#x503C;&#x4E3A;true&#x3002;</td>
    </tr>
  </tbody>
</table>

