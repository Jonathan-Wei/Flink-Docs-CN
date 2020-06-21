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
        <p><b>table.exec.async-lookup.buffer-capacity</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">100</td>
      <td style="text-align:left">Integer</td>
      <td style="text-align:left">&#x5F02;&#x6B65;&#x67E5;&#x627E;&#x5173;&#x8054;&#x53EF;&#x4EE5;&#x89E6;&#x53D1;&#x7684;&#x6700;&#x5927;&#x5F02;&#x6B65;I/O&#x64CD;&#x4F5C;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.async-lookup.timeout</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">&quot;3 min&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x5F02;&#x6B65;&#x64CD;&#x4F5C;&#x5B8C;&#x6210;&#x7684;&#x5F02;&#x6B65;&#x8D85;&#x65F6;&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.disabled-operators</b>
        </p>
        <p>
          <br />Batch</p>
      </td>
      <td style="text-align:left">(none)</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x4E3B;&#x8981;&#x7528;&#x4E8E;&#x6D4B;&#x8BD5;&#x3002;&#x4EE5;&#x9017;&#x53F7;&#x5206;&#x9694;&#x7684;&#x8FD0;&#x7B97;&#x7B26;&#x540D;&#x79F0;&#x5217;&#x8868;&#xFF0C;&#x6BCF;&#x4E2A;&#x540D;&#x79F0;&#x4EE3;&#x8868;&#x4E00;&#x79CD;&#x7981;&#x7528;&#x7684;&#x8FD0;&#x7B97;&#x7B26;&#x3002;&#x53EF;&#x4EE5;&#x7981;&#x7528;&#x7684;&#x8FD0;&#x7B97;&#x7B26;&#x5305;&#x62EC;&quot;NestedLoopJoin&quot;,
        &quot;ShuffleHashJoin&quot;, &quot;BroadcastHashJoin&quot;, &quot;SortMergeJoin&quot;,
        &quot;HashAgg&quot;, &quot;SortAgg&quot;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x672A;&#x7981;&#x7528;&#x4EFB;&#x4F55;&#x8FD0;&#x7B97;&#x7B26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.mini-batch.allow-latency</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left">&quot;-1 ms&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x6700;&#x5927;&#x5EF6;&#x8FDF;&#x53EF;&#x7528;&#x4E8E;MiniBatch&#x6765;&#x7F13;&#x51B2;&#x8F93;&#x5165;&#x8BB0;&#x5F55;&#x3002;MiniBatch&#x662F;&#x4E00;&#x79CD;&#x7528;&#x6765;&#x7F13;&#x51B2;&#x8F93;&#x5165;&#x8BB0;&#x5F55;&#x4EE5;&#x51CF;&#x5C11;&#x72B6;&#x6001;&#x8BBF;&#x95EE;&#x7684;&#x4F18;&#x5316;&#x65B9;&#x6CD5;&#x3002;&#x5728;&#x5141;&#x8BB8;&#x7684;&#x5EF6;&#x8FDF;&#x95F4;&#x9694;&#x548C;&#x8FBE;&#x5230;&#x6700;&#x5927;&#x7F13;&#x51B2;&#x8BB0;&#x5F55;&#x6570;&#x65F6;&#x89E6;&#x53D1;MiniBatch&#x3002;&#x6CE8;&#x610F;:&#x5982;&#x679C;table.exec.mini-batch&#x3002;enabled&#x8BBE;&#x7F6E;&#x4E3A;true&#xFF0C;&#x5176;&#x503C;&#x5FC5;&#x987B;&#x5927;&#x4E8E;&#x96F6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.mini-batch.enabled</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x6307;&#x5B9A;&#x662F;&#x5426;&#x542F;&#x7528;MiniBatch&#x4F18;&#x5316;&#x3002;MiniBatch&#x662F;&#x7528;&#x4E8E;&#x7F13;&#x51B2;&#x8F93;&#x5165;&#x8BB0;&#x5F55;&#x4EE5;&#x51CF;&#x5C11;&#x72B6;&#x6001;&#x8BBF;&#x95EE;&#x7684;&#x4F18;&#x5316;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x7981;&#x7528;&#x6B64;&#x529F;&#x80FD;&#x3002;&#x8981;&#x542F;&#x7528;&#x6B64;&#x529F;&#x80FD;&#xFF0C;&#x7528;&#x6237;&#x5E94;&#x5C06;&#x6B64;&#x914D;&#x7F6E;&#x8BBE;&#x7F6E;&#x4E3A;true&#x3002;&#x6CE8;&#x610F;&#xFF1A;&#x5982;&#x679C;&#x542F;&#x7528;&#x4E86;mini-batch&#xFF0C;&#x5219;&#x5FC5;&#x987B;&#x8BBE;&#x7F6E;&apos;table.exec.mini-batch.allow-latency&apos;&#x548C;&apos;table.exec.mini-batch.size&apos;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.mini-batch.size</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">&#x53EF;&#x4EE5;&#x4E3A;MiniBatch&#x7F13;&#x51B2;&#x6700;&#x5927;&#x8F93;&#x5165;&#x8BB0;&#x5F55;&#x6570;&#x3002;MiniBatch&#x662F;&#x7528;&#x4E8E;&#x7F13;&#x51B2;&#x8F93;&#x5165;&#x8BB0;&#x5F55;&#x4EE5;&#x51CF;&#x5C11;&#x72B6;&#x6001;&#x8BBF;&#x95EE;&#x7684;&#x4F18;&#x5316;&#x3002;MiniBatch&#x4EE5;&#x5141;&#x8BB8;&#x7684;&#x7B49;&#x5F85;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x4EE5;&#x53CA;&#x8FBE;&#x5230;&#x6700;&#x5927;&#x7F13;&#x51B2;&#x8BB0;&#x5F55;&#x6570;&#x89E6;&#x53D1;&#x3002;&#x6CE8;&#x610F;&#xFF1A;MiniBatch&#x5F53;&#x524D;&#x4EC5;&#x9002;&#x7528;&#x4E8E;&#x975E;&#x7A97;&#x53E3;&#x805A;&#x5408;&#x3002;&#x5982;&#x679C;table.exec.mini-batch.enabled&#x8BBE;&#x7F6E;&#x4E3A;true&#xFF0C;&#x5219;&#x5176;&#x503C;&#x5FC5;&#x987B;&#x4E3A;&#x6B63;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.resource.default-parallelism</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF; &#x6D41;</p>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">&#x6574;&#x6570;</td>
      <td style="text-align:left">&#x4E3A;&#x6240;&#x6709;&#x8FD0;&#x7B97;&#x7B26;&#xFF08;&#x4F8B;&#x5982;&#x805A;&#x5408;&#xFF0C;&#x8054;&#x63A5;&#xFF0C;&#x8FC7;&#x6EE4;&#x5668;&#xFF09;&#x8BBE;&#x7F6E;&#x9ED8;&#x8BA4;&#x5E76;&#x884C;&#x5EA6;&#x4EE5;&#x4E0E;&#x5E76;&#x884C;&#x5B9E;&#x4F8B;&#x4E00;&#x8D77;&#x8FD0;&#x884C;&#x3002;&#x6B64;&#x914D;&#x7F6E;&#x6BD4;StreamExecutionEnvironment&#x7684;&#x5E76;&#x884C;&#x6027;&#x5177;&#x6709;&#x66F4;&#x9AD8;&#x7684;&#x4F18;&#x5148;&#x7EA7;&#xFF08;&#x5B9E;&#x9645;&#x4E0A;&#xFF0C;&#x6B64;&#x914D;&#x7F6E;&#x4F18;&#x5148;&#x4E8E;StreamExecutionEnvironment&#x7684;&#x5E76;&#x884C;&#x6027;&#xFF09;&#x3002;&#x503C;-1&#x8868;&#x793A;&#x672A;&#x8BBE;&#x7F6E;&#x9ED8;&#x8BA4;&#x7684;&#x5E76;&#x884C;&#x6027;&#xFF0C;&#x7136;&#x540E;&#x4F7F;&#x7528;StreamExecutionEnvironment&#x7684;&#x5E76;&#x884C;&#x6027;&#x5C06;&#x56DE;&#x9000;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.shuffle-mode</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">&#x201C;&#x6279;&#x91CF;&#x201D;</td>
      <td style="text-align:left">&#x4E32;</td>
      <td style="text-align:left">&#x8BBE;&#x7F6E;&#x6267;&#x884C;&#x968F;&#x673A;&#x64AD;&#x653E;&#x6A21;&#x5F0F;&#x3002;&#x53EA;&#x80FD;&#x8BBE;&#x7F6E;&#x6279;&#x5904;&#x7406;&#x6216;&#x6D41;&#x6C34;&#x7EBF;&#x3002;&#x6279;&#x5904;&#x7406;&#xFF1A;&#x5DE5;&#x4F5C;&#x5C06;&#x9010;&#x6B65;&#x8FDB;&#x884C;&#x3002;&#x6D41;&#x6C34;&#x7EBF;&#xFF1A;&#x4F5C;&#x4E1A;&#x5C06;&#x4EE5;&#x6D41;&#x6A21;&#x5F0F;&#x8FD0;&#x884C;&#xFF0C;&#x4F46;&#x662F;&#x5F53;&#x53D1;&#x9001;&#x65B9;&#x6301;&#x6709;&#x8D44;&#x6E90;&#x7B49;&#x5F85;&#x5411;&#x63A5;&#x6536;&#x65B9;&#x53D1;&#x9001;&#x6570;&#x636E;&#x65F6;&#xFF0C;&#x63A5;&#x6536;&#x65B9;&#x7B49;&#x5F85;&#x8D44;&#x6E90;&#x542F;&#x52A8;&#x53EF;&#x80FD;&#x4F1A;&#x5BFC;&#x81F4;&#x8D44;&#x6E90;&#x6B7B;&#x9501;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.sort.async-merge-enabled</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">&#x771F;&#x6B63;</td>
      <td style="text-align:left">&#x5E03;&#x5C14;&#x578B;</td>
      <td style="text-align:left">&#x662F;&#x5426;&#x5F02;&#x6B65;&#x5408;&#x5E76;&#x6392;&#x5E8F;&#x7684;&#x6EA2;&#x51FA;&#x6587;&#x4EF6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.sort.default-limit</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">-1</td>
      <td style="text-align:left">&#x6574;&#x6570;</td>
      <td style="text-align:left">&#x7528;&#x6237;&#x8BA2;&#x8D2D;&#x540E;&#x672A;&#x8BBE;&#x7F6E;&#x9650;&#x5236;&#x65F6;&#x7684;&#x9ED8;&#x8BA4;&#x9650;&#x5236;&#x3002;-1&#x8868;&#x793A;&#x6B64;&#x914D;&#x7F6E;&#x88AB;&#x5FFD;&#x7565;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.sort.max-num-file-handles</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">128</td>
      <td style="text-align:left">&#x6574;&#x6570;</td>
      <td style="text-align:left">&#x5916;&#x90E8;&#x5408;&#x5E76;&#x6392;&#x5E8F;&#x7684;&#x6700;&#x5927;&#x6247;&#x5165;&#x3002;&#x5B83;&#x9650;&#x5236;&#x4E86;&#x6BCF;&#x4E2A;&#x8FD0;&#x7B97;&#x7B26;&#x7684;&#x6587;&#x4EF6;&#x53E5;&#x67C4;&#x6570;&#x3002;&#x5982;&#x679C;&#x592A;&#x5C0F;&#xFF0C;&#x53EF;&#x80FD;&#x4F1A;&#x5BFC;&#x81F4;&#x4E2D;&#x95F4;&#x5408;&#x5E76;&#x3002;&#x4F46;&#x662F;&#xFF0C;&#x5982;&#x679C;&#x5B83;&#x592A;&#x5927;&#xFF0C;&#x5C06;&#x5BFC;&#x81F4;&#x540C;&#x65F6;&#x6253;&#x5F00;&#x592A;&#x591A;&#x6587;&#x4EF6;&#xFF0C;&#x5360;&#x7528;&#x5185;&#x5B58;&#x5E76;&#x5BFC;&#x81F4;&#x968F;&#x673A;&#x8BFB;&#x53D6;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.source.idle-timeout</b>
        </p>
        <p>
          <br />&#x6D41;&#x5A92;&#x4F53;</p>
      </td>
      <td style="text-align:left">&#x201C; -1&#x6BEB;&#x79D2;&#x201D;</td>
      <td style="text-align:left">&#x4E32;</td>
      <td style="text-align:left">&#x5F53;&#x6E90;&#x5728;&#x8D85;&#x65F6;&#x65F6;&#x95F4;&#x5185;&#x672A;&#x6536;&#x5230;&#x4EFB;&#x4F55;&#x5143;&#x7D20;&#x65F6;&#xFF0C;&#x5B83;&#x5C06;&#x88AB;&#x6807;&#x8BB0;&#x4E3A;&#x4E34;&#x65F6;&#x7A7A;&#x95F2;&#x3002;&#x8FD9;&#x4F7F;&#x5F97;&#x4E0B;&#x6E38;&#x4EFB;&#x52A1;&#x53EF;&#x4EE5;&#x524D;&#x8FDB;&#x5176;&#x6C34;&#x5370;&#xFF0C;&#x800C;&#x65E0;&#x9700;&#x5728;&#x7A7A;&#x95F2;&#x65F6;&#x7B49;&#x5F85;&#x6765;&#x81EA;&#x8BE5;&#x6E90;&#x7684;&#x6C34;&#x5370;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.spill-compression.block-size</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">&#x201C; 64 kb&#x201D;</td>
      <td style="text-align:left">&#x4E32;</td>
      <td style="text-align:left">&#x6EA2;&#x51FA;&#x6570;&#x636E;&#x65F6;&#x7528;&#x4E8E;&#x538B;&#x7F29;&#x7684;&#x5185;&#x5B58;&#x5927;&#x5C0F;&#x3002;&#x5185;&#x5B58;&#x8D8A;&#x5927;&#xFF0C;&#x538B;&#x7F29;&#x7387;&#x8D8A;&#x9AD8;&#xFF0C;&#x4F46;&#x662F;&#x4F5C;&#x4E1A;&#x5C06;&#x6D88;&#x8017;&#x66F4;&#x591A;&#x7684;&#x5185;&#x5B58;&#x8D44;&#x6E90;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.spill-compression.enabled</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">&#x771F;&#x6B63;</td>
      <td style="text-align:left">&#x5E03;&#x5C14;&#x578B;</td>
      <td style="text-align:left">&#x662F;&#x5426;&#x538B;&#x7F29;&#x6EA2;&#x51FA;&#x7684;&#x6570;&#x636E;&#x3002;&#x76EE;&#x524D;&#xFF0C;&#x6211;&#x4EEC;&#x4EC5;&#x652F;&#x6301;&#x5BF9;sort&#x548C;hash-agg&#x548C;hash-join&#x8FD0;&#x7B97;&#x7B26;&#x8FDB;&#x884C;&#x538B;&#x7F29;&#x6EA2;&#x51FA;&#x6570;&#x636E;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.exec.window-agg.buffer-size-limit</b>
        </p>
        <p>
          <br />&#x6279;&#x91CF;</p>
      </td>
      <td style="text-align:left">100000</td>
      <td style="text-align:left">&#x6574;&#x6570;</td>
      <td style="text-align:left">&#x8BBE;&#x7F6E;&#x7EC4;&#x7A97;&#x53E3;agg&#x8FD0;&#x7B97;&#x7B26;&#x4E2D;&#x4F7F;&#x7528;&#x7684;&#x7A97;&#x53E3;&#x5143;&#x7D20;&#x7F13;&#x51B2;&#x533A;&#x5927;&#x5C0F;&#x9650;&#x5236;&#x3002;</td>
    </tr>
  </tbody>
</table>

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
      <td style="text-align:left">&quot;AUTO&quot;</td>
      <td style="text-align:left">String</td>
      <td style="text-align:left">&#x6C47;&#x603B;&#x9636;&#x6BB5;&#x7684;&#x7B56;&#x7565;&#x3002;&#x53EA;&#x80FD;&#x8BBE;&#x7F6E;AUTO&#xFF0C;TWO_PHASE&#x6216;ONE_PHASE&#x3002;AUTO&#xFF1A;&#x805A;&#x5408;&#x9636;&#x6BB5;&#x6CA1;&#x6709;&#x7279;&#x6B8A;&#x7684;&#x6267;&#x884C;&#x5668;&#x3002;&#x9009;&#x62E9;&#x4E24;&#x9636;&#x6BB5;&#x6C47;&#x603B;&#x8FD8;&#x662F;&#x4E00;&#x9636;&#x6BB5;&#x6C47;&#x603B;&#x53D6;&#x51B3;&#x4E8E;&#x6210;&#x672C;&#x3002;TWO_PHASE&#xFF1A;&#x5F3A;&#x5236;&#x4F7F;&#x7528;&#x5177;&#x6709;<code>localAggregate</code>&#x548C;<code>globalAggregate</code>&#x7684;&#x4E24;&#x9636;&#x6BB5;&#x805A;&#x5408;&#x3002;&#x8BF7;&#x6CE8;&#x610F;&#xFF0C;&#x5982;&#x679C;&#x805A;&#x5408;&#x8C03;&#x7528;&#x4E0D;&#x652F;&#x6301;&#x5206;&#x4E3A;&#x4E24;&#x9636;&#x6BB5;&#x7684;&#x4F18;&#x5316;&#xFF0C;&#x6211;&#x4EEC;&#x4ECD;&#x5C06;&#x4F7F;&#x7528;&#x4E00;&#x7EA7;&#x805A;&#x5408;&#x3002;ONE_PHASE&#xFF1A;&#x5F3A;&#x5236;&#x4F7F;&#x7528;&#x4EC5;&#x5177;&#x6709;CompleteGlobalAggregate&#x7684;&#x4E00;&#x7EA7;&#x805A;&#x5408;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.distinct-agg.split.bucket-num</b>
        </p>
        <p>
          <br />Streaming</p>
      </td>
      <td style="text-align:left">1024</td>
      <td style="text-align:left">Integer</td>
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
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x544A;&#x8BC9;&#x4F18;&#x5316;&#x5668;&#x662F;&#x5426;&#x5C06;&#x4E0D;&#x540C;&#x7684;&#x805A;&#x5408;(&#x4F8B;&#x5982;COUNT(distinct
        col)&#x3001;SUM(distinct col))&#x62C6;&#x5206;&#x4E3A;&#x4E24;&#x4E2A;&#x7EA7;&#x522B;&#x3002;&#x7B2C;&#x4E00;&#x4E2A;&#x805A;&#x5408;&#x7531;&#x4E00;&#x4E2A;&#x9644;&#x52A0;&#x7684;&#x952E;&#x6253;&#x4E71;&#xFF0C;&#x8FD9;&#x4E2A;&#x952E;&#x662F;&#x4F7F;&#x7528;distinct_key&#x7684;hashcode&#x548C;&#x6876;&#x6570;&#x8BA1;&#x7B97;&#x51FA;&#x6765;&#x7684;&#x3002;&#x5F53;&#x4E0D;&#x540C;&#x7684;&#x805A;&#x5408;&#x4E2D;&#x5B58;&#x5728;&#x6570;&#x636E;&#x503E;&#x659C;&#x65F6;&#xFF0C;&#x8FD9;&#x79CD;&#x4F18;&#x5316;&#x975E;&#x5E38;&#x6709;&#x7528;&#xFF0C;&#x5E76;&#x63D0;&#x4F9B;&#x4E86;&#x6269;&#x5C55;&#x4F5C;&#x4E1A;&#x7684;&#x80FD;&#x529B;&#x3002;&#x9ED8;&#x8BA4;&#x662F;false&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.join-reorder-enabled</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">false</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5728;&#x4F18;&#x5316;&#x5668;&#x4E2D;&#x542F;&#x7528;&#x5173;&#x8054;&#x91CD;&#x65B0;&#x6392;&#x5E8F;&#x3002;&#x9ED8;&#x8BA4;&#x8BBE;&#x7F6E;&#x4E3A;&#x7981;&#x7528;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.join.broadcast-threshold</b>
        </p>
        <p>
          <br />Batch</p>
      </td>
      <td style="text-align:left">1048576</td>
      <td style="text-align:left">Long</td>
      <td style="text-align:left">&#x914D;&#x7F6E;&#x8868;&#x7684;&#x6700;&#x5927;&#x5927;&#x5C0F;&#xFF08;&#x4EE5;&#x5B57;&#x8282;&#x4E3A;&#x5355;&#x4F4D;&#xFF09;&#xFF0C;&#x8BE5;&#x8868;&#x5728;&#x6267;&#x884C;&#x5173;&#x8054;&#x65F6;&#x5C06;&#x5E7F;&#x64AD;&#x5230;&#x6240;&#x6709;&#x5DE5;&#x4F5C;&#x7A0B;&#x5E8F;&#x8282;&#x70B9;&#x3002;&#x901A;&#x8FC7;&#x5C06;&#x6B64;&#x503C;&#x8BBE;&#x7F6E;&#x4E3A;-1&#x4EE5;&#x7981;&#x7528;&#x5E7F;&#x64AD;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.reuse-source-enabled</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;&#x4E3A;true&#xFF0C;&#x5219;&#x4F18;&#x5316;&#x5668;&#x5C06;&#x5C1D;&#x8BD5;&#x627E;&#x51FA;&#x91CD;&#x590D;&#x7684;&#x8868;&#x6E90;&#x5E76;&#x91CD;&#x65B0;&#x4F7F;&#x7528;&#x5B83;&#x4EEC;&#x3002;&#x4EC5;&#x5F53;&#x542F;&#x7528;table.optimizer.reuse-sub-plan&#x4E3A;true&#x65F6;&#xFF0C;&#x6B64;&#x65B9;&#x6CD5;&#x624D;&#x6709;&#x6548;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.reuse-sub-plan-enabled</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;&#x4E3A;true&#xFF0C;&#x4F18;&#x5316;&#x5668;&#x5C06;&#x5C1D;&#x8BD5;&#x627E;&#x51FA;&#x91CD;&#x590D;&#x7684;&#x5B50;&#x8BA1;&#x5212;&#x5E76;&#x91CD;&#x7528;&#x5B83;&#x4EEC;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>table.optimizer.source.predicate-pushdown-enabled</b>
        </p>
        <p>
          <br />Batch Streaming</p>
      </td>
      <td style="text-align:left">true</td>
      <td style="text-align:left">Boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;&#x4E3A;true&#xFF0C;&#x5219;&#x4F18;&#x5316;&#x5668;&#x4F1A;&#x5C06;&#x8C13;&#x8BCD;&#x4E0B;&#x63A8;&#x5230;FilterableTableSource&#x4E2D;&#x3002;&#x9ED8;&#x8BA4;&#x503C;&#x4E3A;true&#x3002;</td>
    </tr>
  </tbody>
</table>

