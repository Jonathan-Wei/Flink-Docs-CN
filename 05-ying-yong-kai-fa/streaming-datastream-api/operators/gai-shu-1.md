# 概述

操作符将一个或多个数据流转换为新的数据流。程序可以将多个转换组合成复杂的数据流拓扑。

本节描述了基本转换、应用这些转换后的有效物理分区以及对Flink的操作符链接的理解。

## DataStream转换

{% tabs %}
{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Map</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;&#x4E00;&#x4E2A;map&#x51FD;&#x6570;&#xFF0C;&#x4F7F;&#x8F93;&#x5165;&#x6D41;&#x7684;&#x503C;&#x52A0;&#x500D;:</p>
        <p><code>DataStream dataStream = //... </code>
        </p>
        <p><code>dataStream.map(new MapFunction() { </code>
        </p>
        <p><code>    @Override </code>
        </p>
        <p><code>    public Integer map(Integer value) throws Exception {</code>
        </p>
        <p><code>         return 2 * value; </code>
        </p>
        <p><code>     } </code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>FlatMap</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;0&#x4E2A;&#x3001;1&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;&#x4E00;&#x4E2A;flatMap&#x51FD;&#x6570;&#xFF0C;&#x5C06;&#x53E5;&#x5B50;&#x5206;&#x5272;&#x6210;&#x5355;&#x8BCD;:</p>
        <p><code>dataStream.flatMap(new FlatMapFunction() { </code>
        </p>
        <p><code>    @Override public void flatMap(String value, Collector out) throws Exception { </code>
        </p>
        <p><code>        for(String word: value.split(&quot; &quot;)){ </code>
        </p>
        <p><code>            out.collect(word); </code>
        </p>
        <p><code>        } </code>
        </p>
        <p><code>    } </code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Filter</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4E3A;&#x6BCF;&#x4E2A;&#x5143;&#x7D20;&#x8BA1;&#x7B97;&#x5E03;&#x5C14;&#x51FD;&#x6570;&#xFF0C;&#x5E76;&#x4FDD;&#x7559;&#x8BE5;&#x51FD;&#x6570;&#x8FD4;&#x56DE;true&#x7684;&#x5143;&#x7D20;&#x3002;&#x8FC7;&#x6EE4;&#x6389;&#x96F6;&#x503C;&#x7684;&#x8FC7;&#x6EE4;&#x5668;:</p>
        <p><code>dataStream.filter(new FilterFunction() { </code>
        </p>
        <p><code>    @Override public boolean filter(Integer value) throws Exception { </code>
        </p>
        <p><code>        return value != 0; </code>
        </p>
        <p><code>    } </code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>KeyBy</b>
        <br />DataStream &#x2192; KeyedStream</td>
      <td style="text-align:left">
        <p>&#x903B;&#x8F91;&#x4E0A;&#x5C06;&#x6D41;&#x5206;&#x533A;&#x4E3A;&#x4E0D;&#x76F8;&#x4EA4;&#x7684;&#x5206;&#x533A;&#x3002;&#x5177;&#x6709;&#x76F8;&#x540C;&#x5BC6;&#x94A5;&#x7684;&#x6240;&#x6709;&#x8BB0;&#x5F55;&#x90FD;&#x5206;&#x914D;&#x7ED9;&#x540C;&#x4E00;&#x5206;&#x533A;&#x3002;&#x5728;&#x5185;&#x90E8;&#xFF0C;<em>keyBy&#xFF08;&#xFF09;</em>&#x662F;&#x4F7F;&#x7528;&#x6563;&#x5217;&#x5206;&#x533A;&#x5B9E;&#x73B0;&#x7684;&#x3002;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/api_concepts.html#specifying-keys">&#x6307;&#x5B9A;&#x952E;</a>&#x6709;&#x591A;&#x79CD;&#x65B9;&#x6CD5;&#x3002;</p>
        <p>&#x6B64;&#x8F6C;&#x6362;&#x8FD4;&#x56DE;<em>KeyedStream</em>&#xFF0C;&#x5176;&#x4E2D;&#x5305;&#x62EC;&#x4F7F;&#x7528;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html#keyed-state">&#x952E;&#x63A7;&#x72B6;&#x6001;</a>&#x6240;&#x9700;&#x7684;<em>KeyedStream</em>&#x3002;</p>
        <p><code>dataStream.keyBy(&quot;someKey&quot;) // Key by field &quot;someKey&quot; </code>
        </p>
        <p><code>dataStream.keyBy(0) // Key by the first element of a Tuple</code>
        </p>
        <p>&#x6CE8;&#x610F;&#xFF1A;</p>
        <p>&#x5982;&#x679C;&#x51FA;&#x73B0;&#x4EE5;&#x4E0B;&#x60C5;&#x51B5;&#xFF0C;&#x5219;&#x7C7B;&#x578B;<b>&#x4E0D;&#x80FD;&#x6210;&#x4E3A;Key</b>&#xFF1A;</p>
        <ol>
          <li>POJO&#x7C7B;&#x578B;&#xFF0C;&#x4F46;&#x4E0D;&#x8986;&#x76D6;<em>hashCode()</em>&#x65B9;&#x6CD5;&#x5E76;&#x4F9D;&#x8D56;&#x4E8E;<em>Object.hashCode()</em>&#x5B9E;&#x73B0;&#x3002;</li>
          <li>&#x4EFB;&#x4F55;&#x7C7B;&#x578B;&#x7684;&#x6570;&#x7EC4;&#x3002;</li>
        </ol>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Reduce</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x952E;&#x63A7;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x201C;&#x6EDA;&#x52A8;&#x201D;&#x51CF;&#x5C11;&#x3002;&#x5C06;&#x5F53;&#x524D;&#x5143;&#x7D20;&#x4E0E;&#x6700;&#x540E;&#x4E00;&#x4E2A;Reduce&#x7684;&#x503C;&#x7EC4;&#x5408;&#x5E76;&#x4EA7;&#x751F;&#x65B0;&#x503C;&#x3002;</p>
        <p></p>
        <p>reduce&#x51FD;&#x6570;&#xFF0C;&#x7528;&#x4E8E;&#x521B;&#x5EFA;&#x5C40;&#x90E8;&#x548C;&#x7684;&#x6D41;&#xFF1A;
          <br
          />
        </p>
        <p><code>keyedStream.reduce(new ReduceFunction&lt;Integer&gt;() {</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public Integer reduce(Integer value1, Integer value2)</code>
        </p>
        <p><code>    throws Exception {</code>
        </p>
        <p><code>        return value1 + value2;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Fold</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5177;&#x6709;&#x521D;&#x59CB;&#x503C;&#x7684;&#x952E;&#x63A7;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x201C;&#x6EDA;&#x52A8;&#x201D;&#x6298;&#x53E0;&#x3002;&#x5C06;&#x5F53;&#x524D;&#x5143;&#x7D20;&#x4E0E;&#x6700;&#x540E;&#x6298;&#x53E0;&#x7684;&#x503C;&#x7EC4;&#x5408;&#x5E76;&#x751F;&#x6210;&#x65B0;&#x503C;&#x3002;
          <br
          />
          <br />
        </p>
        <p>&#x6298;&#x53E0;&#x51FD;&#x6570;&#xFF0C;&#x5F53;&#x5E94;&#x7528;&#x4E8E;&#x5E8F;&#x5217;&#xFF08;1,2,3,4,5&#xFF09;&#x65F6;&#xFF0C;&#x53D1;&#x51FA;&#x5E8F;&#x5217;&#x201C;start-1&#x201D;&#xFF0C;&#x201C;start-1-2&#x201D;&#xFF0C;&#x201C;start-1-2-3&#x201D;,.
          ..</p>
        <p></p>
        <p><code>DataStream&lt;String&gt; result =</code>
        </p>
        <p><code>  keyedStream.fold(&quot;start&quot;, new FoldFunction&lt;Integer, String&gt;() {</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public String fold(String current, Integer value) {</code>
        </p>
        <p><code>        return current + &quot;-&quot; + value;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>  });</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregations</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x6EDA;&#x52A8;&#x805A;&#x5408;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x805A;&#x5408;&#x3002;min&#x548C;minBy&#x4E4B;&#x95F4;&#x7684;&#x5DEE;&#x5F02;&#x662F;min&#x8FD4;&#x56DE;&#x6700;&#x5C0F;&#x503C;&#xFF0C;&#x800C;minBy&#x8FD4;&#x56DE;&#x8BE5;&#x5B57;&#x6BB5;&#x4E2D;&#x5177;&#x6709;&#x6700;&#x5C0F;&#x503C;&#x7684;&#x5143;&#x7D20;&#xFF08;max&#x548C;maxBy&#x76F8;&#x540C;&#xFF09;&#x3002;</p>
        <p></p>
        <p><code>keyedStream.sum(0);</code>
        </p>
        <p><code>keyedStream.sum(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.min(0);</code>
        </p>
        <p><code>keyedStream.min(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.max(0);</code>
        </p>
        <p><code>keyedStream.max(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.minBy(0);</code>
        </p>
        <p><code>keyedStream.minBy(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.maxBy(0);</code>
        </p>
        <p><code>keyedStream.maxBy(&quot;key&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window</b>
        <br />KeyedStream &#x2192; WindowedStream</td>
      <td style="text-align:left">
        <p>&#x53EF;&#x4EE5;&#x5728;&#x5DF2;&#x7ECF;&#x5206;&#x533A;&#x7684;KeyedStream&#x4E0A;&#x5B9A;&#x4E49;Windows&#x3002;Windows&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x7279;&#x5F81;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x5728;&#x6700;&#x540E;5&#x79D2;&#x5185;&#x5230;&#x8FBE;&#x7684;&#x6570;&#x636E;&#xFF09;&#x5BF9;&#x6BCF;&#x4E2A;&#x5BC6;&#x94A5;&#x4E2D;&#x7684;&#x6570;&#x636E;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x3002;&#x6709;&#x5173;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html">&#x7A97;&#x53E3;</a>&#x7684;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;windows&#x3002;</p>
        <p></p>
        <p><code>dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>WindowAll</b>
        <br />DataStream &#x2192; AllWindowedStream</td>
      <td style="text-align:left">
        <p>Windows&#x53EF;&#x4EE5;&#x5728;&#x5E38;&#x89C4;DataStream&#x4E0A;&#x5B9A;&#x4E49;&#x3002;Windows&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x7279;&#x5F81;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x5728;&#x6700;&#x540E;5&#x79D2;&#x5185;&#x5230;&#x8FBE;&#x7684;&#x6570;&#x636E;&#xFF09;&#x5BF9;&#x6240;&#x6709;&#x6D41;&#x4E8B;&#x4EF6;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x3002;&#x6709;&#x5173;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html">&#x7A97;&#x53E3;</a>&#x7684;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;windows&#x3002;</p>
        <p><b>&#x8B66;&#x544A;&#xFF1A;</b>&#x5728;&#x8BB8;&#x591A;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x8FD9;<b>&#x662F;&#x975E;&#x5E76;&#x884C;</b>&#x8F6C;&#x6362;&#x3002;&#x6240;&#x6709;&#x8BB0;&#x5F55;&#x5C06;&#x6536;&#x96C6;&#x5728;windowAll&#x64CD;&#x4F5C;&#x7B26;&#x7684;&#x4E00;&#x4E2A;&#x4EFB;&#x52A1;&#x4E2D;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.windowAll(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Apply</b>
        <br />WindowedStream &#x2192; DataStream
        <br />AllWindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x822C;&#x51FD;&#x6570;&#x5E94;&#x7528;&#x4E8E;&#x6574;&#x4E2A;&#x7A97;&#x53E3;&#x3002;&#x4E0B;&#x9762;&#x662F;&#x4E00;&#x4E2A;&#x624B;&#x52A8;&#x6C42;&#x548C;&#x7A97;&#x53E3;&#x5143;&#x7D20;&#x7684;&#x51FD;&#x6570;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5982;&#x679C;&#x60A8;&#x6B63;&#x5728;&#x4F7F;&#x7528;windowAll&#x8F6C;&#x6362;&#xFF0C;&#x5219;&#x9700;&#x8981;&#x4F7F;&#x7528;AllWindowFunction&#x3002;
          <br
          />
        </p>
        <p><code>windowedStream.apply (new WindowFunction&lt;Tuple2&lt;String,Integer&gt;, Integer, Tuple, Window&gt;() {</code>
        </p>
        <p><code>    public void apply (Tuple tuple,</code>
        </p>
        <p><code>            Window window,</code>
        </p>
        <p><code>            Iterable&lt;Tuple2&lt;String, Integer&gt;&gt; values,</code>
        </p>
        <p><code>            Collector&lt;Integer&gt; out) throws Exception {</code>
        </p>
        <p><code>        int sum = 0;</code>
        </p>
        <p><code>        for (value t: values) {</code>
        </p>
        <p><code>            sum += t.f1;</code>
        </p>
        <p><code>        }</code>
        </p>
        <p><code>        out.collect (new Integer(sum));</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
        <p><code>// applying an AllWindowFunction on non-keyed window stream</code>
        </p>
        <p><code>allWindowedStream.apply (new AllWindowFunction&lt;Tuple2&lt;String,Integer&gt;, Integer, Window&gt;() {</code>
        </p>
        <p><code>    public void apply (Window window,</code>
        </p>
        <p><code>            Iterable&lt;Tuple2&lt;String, Integer&gt;&gt; values,</code>
        </p>
        <p><code>            Collector&lt;Integer&gt; out) throws Exception {</code>
        </p>
        <p><code>        int sum = 0;</code>
        </p>
        <p><code>        for (value t: values) {</code>
        </p>
        <p><code>            sum += t.f1;</code>
        </p>
        <p><code>        }</code>
        </p>
        <p><code>        out.collect (new Integer(sum));</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Reduce</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;ReduceFunction&#x5E94;&#x7528;&#x4E8E;&#x7A97;&#x53E3;&#x5E76;&#x8FD4;&#x56DE;Reduce&#x7684;&#x503C;&#x3002;
          <br
          />
        </p>
        <p><code>windowedStream.reduce (new ReduceFunction&lt;Tuple2&lt;String,Integer&gt;&gt;() {</code>
        </p>
        <p><code>    public Tuple2&lt;String, Integer&gt; reduce(Tuple2&lt;String, Integer&gt; value1, Tuple2&lt;String, Integer&gt; value2) throws Exception {</code>
        </p>
        <p><code>        return new Tuple2&lt;String,Integer&gt;(value1.f0, value1.f1 + value2.f1);</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Fold</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;FoldFunction&#x5E94;&#x7528;&#x4E8E;&#x7A97;&#x53E3;&#x5E76;&#x8FD4;&#x56DE;&#x6298;&#x53E0;&#x503C;&#x3002;&#x793A;&#x4F8B;&#x51FD;&#x6570;&#x5E94;&#x7528;&#x4E8E;&#x5E8F;&#x5217;&#xFF08;1,2,3,4,5&#xFF09;&#x65F6;&#xFF0C;&#x5C06;&#x5E8F;&#x5217;&#x6298;&#x53E0;&#x4E3A;&#x5B57;&#x7B26;&#x4E32;&#x201C;start-1-2-3-4-5&#x201D;&#xFF1A;
          <br
          />
        </p>
        <p><code>windowedStream.fold(&quot;start&quot;, new FoldFunction&lt;Integer, String&gt;() {</code>
        </p>
        <p><code>    public String fold(String current, Integer value) {</code>
        </p>
        <p><code>        return current + &quot;-&quot; + value;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregations on windows</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x805A;&#x5408;&#x7A97;&#x53E3;&#x7684;&#x5185;&#x5BB9;&#x3002;min&#x548C;minBy&#x4E4B;&#x95F4;&#x7684;&#x5DEE;&#x5F02;&#x662F;min&#x8FD4;&#x56DE;&#x6700;&#x5C0F;&#x503C;&#xFF0C;&#x800C;minBy&#x8FD4;&#x56DE;&#x8BE5;&#x5B57;&#x6BB5;&#x4E2D;&#x5177;&#x6709;&#x6700;&#x5C0F;&#x503C;&#x7684;&#x5143;&#x7D20;&#xFF08;max&#x548C;maxBy&#x76F8;&#x540C;&#xFF09;&#x3002;
          <br
          />
        </p>
        <p><code>windowedStream.sum(0);</code>
        </p>
        <p><code>windowedStream.sum(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.min(0);</code>
        </p>
        <p><code>windowedStream.min(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.max(0);</code>
        </p>
        <p><code>windowedStream.max(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.minBy(0);</code>
        </p>
        <p><code>windowedStream.minBy(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.maxBy(0);</code>
        </p>
        <p><code>windowedStream.maxBy(&quot;key&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />DataStream* &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4E24;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x7684;&#x8054;&#x5408;&#xFF0C;&#x521B;&#x5EFA;&#x5305;&#x542B;&#x6765;&#x81EA;&#x6240;&#x6709;&#x6D41;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x7684;&#x65B0;&#x6D41;&#x3002;&#x6CE8;&#x610F;&#xFF1A;&#x5982;&#x679C;&#x5C06;&#x6570;&#x636E;&#x6D41;&#x4E0E;&#x5176;&#x81EA;&#x8EAB;&#x8054;&#x5408;&#xFF0C;&#x5219;&#x4F1A;&#x5728;&#x7ED3;&#x679C;&#x6D41;&#x4E2D;&#x83B7;&#x53D6;&#x4E24;&#x6B21;&#x5143;&#x7D20;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.union(otherStream1, otherStream2, ...);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Join</b>
        <br />DataStream,DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x7ED9;&#x5B9A;&#x5BC6;&#x94A5;&#x548C;&#x516C;&#x5171;&#x7A97;&#x53E3;&#x4E0A;&#x5173;&#x8054;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.join(otherStream)</code>
        </p>
        <p><code>    .where(&lt;key selector&gt;).equalTo(&lt;key selector&gt;)</code>
        </p>
        <p><code>    .window(TumblingEventTimeWindows.of(Time.seconds(3)))</code>
        </p>
        <p><code>    .apply (new JoinFunction () {...});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Interval Join</b>
        <br />KeyedStream,KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5728;&#x7ED9;&#x5B9A;&#x7684;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x5185;&#x4F7F;&#x7528;&#x516C;&#x5171;&#x5BC6;&#x94A5;&#x8FDE;&#x63A5;&#x4E24;&#x4E2A;&#x952E;&#x63A7;&#x6D41;&#x7684;&#x4E24;&#x4E2A;&#x5143;&#x7D20;e1&#x548C;e2&#xFF0C;&#x4ECE;&#x800C;&#x5F97;&#x5230;e1.timestamp
          + lowerBound &lt;= e2.timestamp &lt;= e1.timestamp + upperBound</p>
        <p></p>
        <p><code>// this will join the two streams so that</code>
        </p>
        <p><code>// key1 == key2 &amp;&amp; leftTs - 2 &lt; rightTs &lt; leftTs + 2</code>
        </p>
        <p><code>keyedStream.intervalJoin(otherKeyedStream)</code>
        </p>
        <p><code>    .between(Time.milliseconds(-2), Time.milliseconds(2)) // lower and upper bound</code>
        </p>
        <p><code>    .upperBoundExclusive(true) // optional</code>
        </p>
        <p><code>    .lowerBoundExclusive(true) // optional</code>
        </p>
        <p><code>    .process(new IntervalJoinFunction() {...});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window CoGroup</b>
        <br />DataStream,DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5728;&#x7ED9;&#x5B9A;Key&#x548C;&#x516C;&#x5171;&#x7A97;&#x53E3;&#x4E0A;&#x5BF9;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x8FDB;&#x884C;Cogroup&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.coGroup(otherStream)</code>
        </p>
        <p><code>    .where(0).equalTo(1)</code>
        </p>
        <p><code>    .window(TumblingEventTimeWindows.of(Time.seconds(3)))</code>
        </p>
        <p><code>    .apply (new CoGroupFunction () {...});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Connect</b>
        <br />DataStream,DataStream &#x2192; ConnectedStreams</td>
      <td style="text-align:left">
        <p>&#x201C;&#x8FDE;&#x63A5;&#x201D;&#x4E24;&#x4E2A;&#x4FDD;&#x7559;&#x7C7B;&#x578B;&#x7684;&#x6570;&#x636E;&#x6D41;&#x3002;&#x8FDE;&#x63A5;&#x5141;&#x8BB8;&#x5728;&#x4E24;&#x4E2A;&#x6D41;&#x4E4B;&#x95F4;&#x5171;&#x4EAB;&#x72B6;&#x6001;&#x3002;
          <br
          />
        </p>
        <p><code>DataStream&lt;Integer&gt; someStream = //...</code>
        </p>
        <p><code>DataStream&lt;String&gt; otherStream = //...</code>
        </p>
        <p><code>ConnectedStreams&lt;Integer, String&gt; connectedStreams = someStream.connect(otherStream);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>CoMap, CoFlatMap</b>
        <br />ConnectedStreams &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;&#x8FDE;&#x63A5;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;map&#x548C;flatMap
          <br
          />
        </p>
        <p><code>connectedStreams.map(new CoMapFunction&lt;Integer, String, Boolean&gt;() {</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public Boolean map1(Integer value) {</code>
        </p>
        <p><code>        return true;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public Boolean map2(String value) {</code>
        </p>
        <p><code>        return false;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
        <p><code>connectedStreams.flatMap(new CoFlatMapFunction&lt;Integer, String, String&gt;() {</code>
        </p>
        <p><code>   @Override</code>
        </p>
        <p><code>   public void flatMap1(Integer value, Collector&lt;String&gt; out) {</code>
        </p>
        <p><code>       out.collect(value.toString());</code>
        </p>
        <p><code>   }</code>
        </p>
        <p><code>   @Override</code>
        </p>
        <p><code>   public void flatMap2(String value, Collector&lt;String&gt; out) {</code>
        </p>
        <p><code>       for (String word: value.split(&quot; &quot;)) {</code>
        </p>
        <p><code>         out.collect(word);</code>
        </p>
        <p><code>       }</code>
        </p>
        <p><code>   }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Split</b>
        <br />DataStream &#x2192; SplitStream</td>
      <td style="text-align:left">
        <p>&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x6807;&#x51C6;&#x5C06;&#x6D41;&#x62C6;&#x5206;&#x4E3A;&#x4E24;&#x4E2A;&#x6216;&#x66F4;&#x591A;&#x4E2A;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>SplitStream&lt;Integer&gt; split = someDataStream.split(new OutputSelector&lt;Integer&gt;() {</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public Iterable&lt;String&gt; select(Integer value) {</code>
        </p>
        <p><code>        List&lt;String&gt; output = new ArrayList&lt;String&gt;();</code>
        </p>
        <p><code>        if (value % 2 == 0) {</code>
        </p>
        <p><code>            output.add(&quot;even&quot;);</code>
        </p>
        <p><code>        }</code>
        </p>
        <p><code>        else {</code>
        </p>
        <p><code>            output.add(&quot;odd&quot;);</code>
        </p>
        <p><code>        }</code>
        </p>
        <p><code>        return output;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Select</b>
        <br />SplitStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x62C6;&#x5206;&#x6D41;&#x4E2D;&#x9009;&#x62E9;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>SplitStream&lt;Integer&gt; split;</code>
        </p>
        <p><code>DataStream&lt;Integer&gt; even = split.select(&quot;even&quot;);</code>
        </p>
        <p><code>DataStream&lt;Integer&gt; odd = split.select(&quot;odd&quot;);</code>
        </p>
        <p><code>DataStream&lt;Integer&gt; all = split.select(&quot;even&quot;,&quot;odd&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Iterate</b>
        <br />DataStream &#x2192; IterativeStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x5C06;&#x4E00;&#x4E2A;&#x64CD;&#x4F5C;&#x7B26;&#x7684;&#x8F93;&#x51FA;&#x91CD;&#x5B9A;&#x5411;&#x5230;&#x67D0;&#x4E2A;&#x5148;&#x524D;&#x7684;&#x64CD;&#x4F5C;&#x7B26;&#xFF0C;&#x5728;&#x6D41;&#x4E2D;&#x521B;&#x5EFA;&#x201C;&#x53CD;&#x9988;&#x201D;&#x5FAA;&#x73AF;&#x3002;&#x8FD9;&#x5BF9;&#x4E8E;&#x5B9A;&#x4E49;&#x4E0D;&#x65AD;&#x66F4;&#x65B0;&#x6A21;&#x578B;&#x7684;&#x7B97;&#x6CD5;&#x7279;&#x522B;&#x6709;&#x7528;&#x3002;&#x4EE5;&#x4E0B;&#x4EE3;&#x7801;&#x4EE5;&#x6D41;&#x5F00;&#x5934;&#x5E76;&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#x8FED;&#x4EE3;&#x4F53;&#x3002;&#x5927;&#x4E8E;0&#x7684;&#x5143;&#x7D20;&#x5C06;&#x88AB;&#x53D1;&#x9001;&#x56DE;&#x53CD;&#x9988;&#x901A;&#x9053;&#xFF0C;&#x5176;&#x4F59;&#x5143;&#x7D20;&#x5C06;&#x5411;&#x4E0B;&#x6E38;&#x8F6C;&#x53D1;&#x3002;&#x6709;&#x5173;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/#iterations">&#x8FED;&#x4EE3;</a>&#x3002;
            <br />
        </p>
        <p><code>IterativeStream&lt;Long&gt; iteration = initialStream.iterate();</code>
        </p>
        <p><code>DataStream&lt;Long&gt; iterationBody = iteration.map (/*do something*/);</code>
        </p>
        <p><code>DataStream&lt;Long&gt; feedback = iterationBody.filter(new FilterFunction&lt;Long&gt;(){</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public boolean filter(Long value) throws Exception {</code>
        </p>
        <p><code>        return value &gt; 0;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
        <p><code>iteration.closeWith(feedback);</code>
        </p>
        <p><code>DataStream&lt;Long&gt; output = iterationBody.filter(new FilterFunction&lt;Long&gt;(){</code>
        </p>
        <p><code>    @Override</code>
        </p>
        <p><code>    public boolean filter(Long value) throws Exception {</code>
        </p>
        <p><code>        return value &lt;= 0;</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>});</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Extract Timestamps</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x8BB0;&#x5F55;&#x4E2D;&#x63D0;&#x53D6;&#x65F6;&#x95F4;&#x6233;&#xFF0C;&#x4EE5;&#x4FBF;&#x5728;&#x4F7F;&#x7528;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#x8BED;&#x4E49;&#x7684;&#x7A97;&#x53E3;&#x65F6;&#x4F7F;&#x7528;&#x3002;&#x67E5;&#x770B;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html">&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;</a>&#x3002;
            <br />
        </p>
        <p><code>stream.assignTimestamps (new TimeStampExtractor() {...});</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Map</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x3002;&#x4E00;&#x4E2A;map&#x51FD;&#x6570;&#xFF0C;&#x4F7F;&#x8F93;&#x5165;&#x6D41;&#x7684;&#x503C;&#x52A0;&#x500D;:</p>
        <p></p>
        <p><code>dataStream.map { x =&gt; x * 2 }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>FlatMap</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x53D6;&#x4E00;&#x4E2A;&#x5143;&#x7D20;&#x5E76;&#x751F;&#x6210;0&#x4E2A;&#x3001;1&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x5143;&#x7D20;&#x3002;&#x4E00;&#x4E2A;flatMap&#x51FD;&#x6570;&#xFF0C;&#x5C06;&#x53E5;&#x5B50;&#x5206;&#x5272;&#x6210;&#x5355;&#x8BCD;:</p>
        <p></p>
        <p><code>dataStream.flatMap { str =&gt; str.split(&quot; &quot;) }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Filter</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4E3A;&#x6BCF;&#x4E2A;&#x5143;&#x7D20;&#x8BA1;&#x7B97;&#x5E03;&#x5C14;&#x51FD;&#x6570;&#xFF0C;&#x5E76;&#x4FDD;&#x7559;&#x8BE5;&#x51FD;&#x6570;&#x8FD4;&#x56DE;true&#x7684;&#x5143;&#x7D20;&#x3002;&#x8FC7;&#x6EE4;&#x6389;&#x96F6;&#x503C;&#x7684;&#x8FC7;&#x6EE4;&#x5668;:
          <br
          />
        </p>
        <p><code>dataStream.filter { _ != 0 }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>KeyBy</b>
        <br />DataStream &#x2192; KeyedStream</td>
      <td style="text-align:left">
        <p>&#x903B;&#x8F91;&#x4E0A;&#x5C06;&#x6D41;&#x5206;&#x533A;&#x4E3A;&#x4E0D;&#x76F8;&#x4EA4;&#x7684;&#x5206;&#x533A;&#x3002;&#x5177;&#x6709;&#x76F8;&#x540C;&#x5BC6;&#x94A5;&#x7684;&#x6240;&#x6709;&#x8BB0;&#x5F55;&#x90FD;&#x5206;&#x914D;&#x7ED9;&#x540C;&#x4E00;&#x5206;&#x533A;&#x3002;&#x5728;&#x5185;&#x90E8;&#xFF0C;<em>keyBy&#xFF08;&#xFF09;</em>&#x662F;&#x4F7F;&#x7528;&#x6563;&#x5217;&#x5206;&#x533A;&#x5B9E;&#x73B0;&#x7684;&#x3002;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/api_concepts.html#specifying-keys">&#x6307;&#x5B9A;&#x952E;</a>&#x6709;&#x591A;&#x79CD;&#x65B9;&#x6CD5;&#x3002;</p>
        <p>&#x6B64;&#x8F6C;&#x6362;&#x8FD4;&#x56DE;<em>KeyedStream</em>&#xFF0C;&#x5176;&#x4E2D;&#x5305;&#x62EC;&#x4F7F;&#x7528;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/state.html#keyed-state">&#x952E;&#x63A7;&#x72B6;&#x6001;</a>&#x6240;&#x9700;&#x7684;<em>KeyedStream</em>&#x3002;</p>
        <p><code>dataStream.keyBy(&quot;someKey&quot;) // Key by field &quot;someKey&quot; </code>
        </p>
        <p><code>dataStream.keyBy(0) // Key by the first element of a Tuple</code>
        </p>
        <p>&#x6CE8;&#x610F;&#xFF1A;</p>
        <p>&#x5982;&#x679C;&#x51FA;&#x73B0;&#x4EE5;&#x4E0B;&#x60C5;&#x51B5;&#xFF0C;&#x5219;&#x7C7B;&#x578B;<b>&#x4E0D;&#x80FD;&#x6210;&#x4E3A;Key</b>&#xFF1A;</p>
        <ol>
          <li>POJO&#x7C7B;&#x578B;&#xFF0C;&#x4F46;&#x4E0D;&#x8986;&#x76D6;<em>hashCode()</em>&#x65B9;&#x6CD5;&#x5E76;&#x4F9D;&#x8D56;&#x4E8E;<em>Object.hashCode()</em>&#x5B9E;&#x73B0;&#x3002;</li>
          <li>&#x4EFB;&#x4F55;&#x7C7B;&#x578B;&#x7684;&#x6570;&#x7EC4;&#x3002;</li>
        </ol>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Reduce</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x952E;&#x63A7;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x201C;&#x6EDA;&#x52A8;&#x201D;&#x51CF;&#x5C11;&#x3002;&#x5C06;&#x5F53;&#x524D;&#x5143;&#x7D20;&#x4E0E;&#x6700;&#x540E;&#x4E00;&#x4E2A;Reduce&#x7684;&#x503C;&#x7EC4;&#x5408;&#x5E76;&#x4EA7;&#x751F;&#x65B0;&#x503C;&#x3002;</p>
        <p></p>
        <p>reduce&#x51FD;&#x6570;&#xFF0C;&#x7528;&#x4E8E;&#x521B;&#x5EFA;&#x5C40;&#x90E8;&#x548C;&#x7684;&#x6D41;&#xFF1A;
          <br
          />
        </p>
        <p><code>keyedStream.reduce { _ + _ }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Fold</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5177;&#x6709;&#x521D;&#x59CB;&#x503C;&#x7684;&#x952E;&#x63A7;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x201C;&#x6EDA;&#x52A8;&#x201D;&#x6298;&#x53E0;&#x3002;&#x5C06;&#x5F53;&#x524D;&#x5143;&#x7D20;&#x4E0E;&#x6700;&#x540E;&#x6298;&#x53E0;&#x7684;&#x503C;&#x7EC4;&#x5408;&#x5E76;&#x751F;&#x6210;&#x65B0;&#x503C;&#x3002;
          <br
          />
          <br />
        </p>
        <p>&#x6298;&#x53E0;&#x51FD;&#x6570;&#xFF0C;&#x5F53;&#x5E94;&#x7528;&#x4E8E;&#x5E8F;&#x5217;&#xFF08;1,2,3,4,5&#xFF09;&#x65F6;&#xFF0C;&#x53D1;&#x51FA;&#x5E8F;&#x5217;&#x201C;start-1&#x201D;&#xFF0C;&#x201C;start-1-2&#x201D;&#xFF0C;&#x201C;start-1-2-3&#x201D;,.
          ..</p>
        <p></p>
        <p><code>val result: DataStream[String] =</code>
        </p>
        <p><code>    keyedStream.fold(&quot;start&quot;)((str, i) =&gt; { str + &quot;-&quot; + i })</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregations</b>
        <br />KeyedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x6EDA;&#x52A8;&#x805A;&#x5408;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;&#x805A;&#x5408;&#x3002;min&#x548C;minBy&#x4E4B;&#x95F4;&#x7684;&#x5DEE;&#x5F02;&#x662F;min&#x8FD4;&#x56DE;&#x6700;&#x5C0F;&#x503C;&#xFF0C;&#x800C;minBy&#x8FD4;&#x56DE;&#x8BE5;&#x5B57;&#x6BB5;&#x4E2D;&#x5177;&#x6709;&#x6700;&#x5C0F;&#x503C;&#x7684;&#x5143;&#x7D20;&#xFF08;max&#x548C;maxBy&#x76F8;&#x540C;&#xFF09;&#x3002;</p>
        <p></p>
        <p><code>keyedStream.sum(0);</code>
        </p>
        <p><code>keyedStream.sum(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.min(0);</code>
        </p>
        <p><code>keyedStream.min(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.max(0);</code>
        </p>
        <p><code>keyedStream.max(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.minBy(0);</code>
        </p>
        <p><code>keyedStream.minBy(&quot;key&quot;);</code>
        </p>
        <p><code>keyedStream.maxBy(0);</code>
        </p>
        <p><code>keyedStream.maxBy(&quot;key&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window</b>
        <br />KeyedStream &#x2192; WindowedStream</td>
      <td style="text-align:left">
        <p>&#x53EF;&#x4EE5;&#x5728;&#x5DF2;&#x7ECF;&#x5206;&#x533A;&#x7684;KeyedStream&#x4E0A;&#x5B9A;&#x4E49;Windows&#x3002;Windows&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x7279;&#x5F81;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x5728;&#x6700;&#x540E;5&#x79D2;&#x5185;&#x5230;&#x8FBE;&#x7684;&#x6570;&#x636E;&#xFF09;&#x5BF9;&#x6BCF;&#x4E2A;&#x5BC6;&#x94A5;&#x4E2D;&#x7684;&#x6570;&#x636E;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x3002;&#x6709;&#x5173;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html">&#x7A97;&#x53E3;</a>&#x7684;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;windows&#x3002;</p>
        <p></p>
        <p><code>dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>WindowAll</b>
        <br />DataStream &#x2192; AllWindowedStream</td>
      <td style="text-align:left">
        <p>Windows&#x53EF;&#x4EE5;&#x5728;&#x5E38;&#x89C4;DataStream&#x4E0A;&#x5B9A;&#x4E49;&#x3002;Windows&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x7279;&#x5F81;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x5728;&#x6700;&#x540E;5&#x79D2;&#x5185;&#x5230;&#x8FBE;&#x7684;&#x6570;&#x636E;&#xFF09;&#x5BF9;&#x6240;&#x6709;&#x6D41;&#x4E8B;&#x4EF6;&#x8FDB;&#x884C;&#x5206;&#x7EC4;&#x3002;&#x6709;&#x5173;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/windows.html">&#x7A97;&#x53E3;</a>&#x7684;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x89C1;windows&#x3002;</p>
        <p><b>&#x8B66;&#x544A;&#xFF1A;</b>&#x5728;&#x8BB8;&#x591A;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x8FD9;<b>&#x662F;&#x975E;&#x5E76;&#x884C;</b>&#x8F6C;&#x6362;&#x3002;&#x6240;&#x6709;&#x8BB0;&#x5F55;&#x5C06;&#x6536;&#x96C6;&#x5728;windowAll&#x64CD;&#x4F5C;&#x7B26;&#x7684;&#x4E00;&#x4E2A;&#x4EFB;&#x52A1;&#x4E2D;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.windowAll(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Apply</b>
        <br />WindowedStream &#x2192; DataStream
        <br />AllWindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;&#x4E00;&#x822C;&#x51FD;&#x6570;&#x5E94;&#x7528;&#x4E8E;&#x6574;&#x4E2A;&#x7A97;&#x53E3;&#x3002;&#x4E0B;&#x9762;&#x662F;&#x4E00;&#x4E2A;&#x624B;&#x52A8;&#x6C42;&#x548C;&#x7A97;&#x53E3;&#x5143;&#x7D20;&#x7684;&#x51FD;&#x6570;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5982;&#x679C;&#x60A8;&#x6B63;&#x5728;&#x4F7F;&#x7528;windowAll&#x8F6C;&#x6362;&#xFF0C;&#x5219;&#x9700;&#x8981;&#x4F7F;&#x7528;AllWindowFunction&#x3002;
          <br
          />
        </p>
        <p><code>windowedStream.apply { WindowFunction }</code>
        </p>
        <p><code>// applying an AllWindowFunction on non-keyed window stream</code>
        </p>
        <p><code>allWindowedStream.apply { AllWindowFunction }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Reduce</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;ReduceFunction&#x5E94;&#x7528;&#x4E8E;&#x7A97;&#x53E3;&#x5E76;&#x8FD4;&#x56DE;Reduce&#x7684;&#x503C;&#x3002;</p>
        <p></p>
        <p><code>windowedStream.reduce { _ + _ }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Fold</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5C06;FoldFunction&#x5E94;&#x7528;&#x4E8E;&#x7A97;&#x53E3;&#x5E76;&#x8FD4;&#x56DE;&#x6298;&#x53E0;&#x503C;&#x3002;&#x793A;&#x4F8B;&#x51FD;&#x6570;&#x5E94;&#x7528;&#x4E8E;&#x5E8F;&#x5217;&#xFF08;1,2,3,4,5&#xFF09;&#x65F6;&#xFF0C;&#x5C06;&#x5E8F;&#x5217;&#x6298;&#x53E0;&#x4E3A;&#x5B57;&#x7B26;&#x4E32;&#x201C;start-1-2-3-4-5&#x201D;&#xFF1A;
          <br
          />
        </p>
        <p><code>val result: DataStream[String] =</code>
        </p>
        <p><code>    windowedStream.fold(&quot;start&quot;, (str, i) =&gt; { str + &quot;-&quot; + i })</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Aggregations on windows</b>
        <br />WindowedStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x805A;&#x5408;&#x7A97;&#x53E3;&#x7684;&#x5185;&#x5BB9;&#x3002;min&#x548C;minBy&#x4E4B;&#x95F4;&#x7684;&#x5DEE;&#x5F02;&#x662F;min&#x8FD4;&#x56DE;&#x6700;&#x5C0F;&#x503C;&#xFF0C;&#x800C;minBy&#x8FD4;&#x56DE;&#x8BE5;&#x5B57;&#x6BB5;&#x4E2D;&#x5177;&#x6709;&#x6700;&#x5C0F;&#x503C;&#x7684;&#x5143;&#x7D20;&#xFF08;max&#x548C;maxBy&#x76F8;&#x540C;&#xFF09;&#x3002;
          <br
          />
        </p>
        <p><code>windowedStream.sum(0);</code>
        </p>
        <p><code>windowedStream.sum(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.min(0);</code>
        </p>
        <p><code>windowedStream.min(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.max(0);</code>
        </p>
        <p><code>windowedStream.max(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.minBy(0);</code>
        </p>
        <p><code>windowedStream.minBy(&quot;key&quot;);</code>
        </p>
        <p><code>windowedStream.maxBy(0);</code>
        </p>
        <p><code>windowedStream.maxBy(&quot;key&quot;);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Union</b>
        <br />DataStream* &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4E24;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x7684;&#x8054;&#x5408;&#xFF0C;&#x521B;&#x5EFA;&#x5305;&#x542B;&#x6765;&#x81EA;&#x6240;&#x6709;&#x6D41;&#x7684;&#x6240;&#x6709;&#x5143;&#x7D20;&#x7684;&#x65B0;&#x6D41;&#x3002;&#x6CE8;&#x610F;&#xFF1A;&#x5982;&#x679C;&#x5C06;&#x6570;&#x636E;&#x6D41;&#x4E0E;&#x5176;&#x81EA;&#x8EAB;&#x8054;&#x5408;&#xFF0C;&#x5219;&#x4F1A;&#x5728;&#x7ED3;&#x679C;&#x6D41;&#x4E2D;&#x83B7;&#x53D6;&#x4E24;&#x6B21;&#x5143;&#x7D20;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.union(otherStream1, otherStream2, ...);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window Join</b>
        <br />DataStream,DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x7ED9;&#x5B9A;&#x5BC6;&#x94A5;&#x548C;&#x516C;&#x5171;&#x7A97;&#x53E3;&#x4E0A;&#x5173;&#x8054;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.join(otherStream)</code>
        </p>
        <p><code>    .where(&lt;key selector&gt;).equalTo(&lt;key selector&gt;)</code>
        </p>
        <p><code>    .window(TumblingEventTimeWindows.of(Time.seconds(3)))</code>
        </p>
        <p><code>    .apply { ... }</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Window CoGroup</b>
        <br />DataStream,DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x5728;&#x7ED9;&#x5B9A;Key&#x548C;&#x516C;&#x5171;&#x7A97;&#x53E3;&#x4E0A;&#x5BF9;&#x4E24;&#x4E2A;&#x6570;&#x636E;&#x6D41;&#x8FDB;&#x884C;Cogroup&#x3002;
          <br
          />
        </p>
        <p><code>dataStream.coGroup(otherStream)</code>
        </p>
        <p><code>    .where(0).equalTo(1)</code>
        </p>
        <p><code>    .window(TumblingEventTimeWindows.of(Time.seconds(3)))</code>
        </p>
        <p><code>    .apply {}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Connect</b>
        <br />DataStream,DataStream &#x2192; ConnectedStreams</td>
      <td style="text-align:left">
        <p>&#x201C;&#x8FDE;&#x63A5;&#x201D;&#x4E24;&#x4E2A;&#x4FDD;&#x7559;&#x7C7B;&#x578B;&#x7684;&#x6570;&#x636E;&#x6D41;&#x3002;&#x8FDE;&#x63A5;&#x5141;&#x8BB8;&#x5728;&#x4E24;&#x4E2A;&#x6D41;&#x4E4B;&#x95F4;&#x5171;&#x4EAB;&#x72B6;&#x6001;&#x3002;
          <br
          />
        </p>
        <p><code>someStream : DataStream[Int] = ...</code>
        </p>
        <p><code>otherStream : DataStream[String] = ...</code>
        </p>
        <p><code>val connectedStreams = someStream.connect(otherStream)</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>CoMap, CoFlatMap</b>
        <br />ConnectedStreams &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x7C7B;&#x4F3C;&#x4E8E;&#x8FDE;&#x63A5;&#x6570;&#x636E;&#x6D41;&#x4E0A;&#x7684;map&#x548C;flatMap
          <br
          />
        </p>
        <p><code>connectedStreams.map(</code>
        </p>
        <p><code>    (_ : Int) =&gt; true,</code>
        </p>
        <p><code>    (_ : String) =&gt; false</code>
        </p>
        <p><code>)</code>
        </p>
        <p><code>connectedStreams.flatMap(</code>
        </p>
        <p><code>    (_ : Int) =&gt; true,</code>
        </p>
        <p><code>    (_ : String) =&gt; false</code>
        </p>
        <p><code>)</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Split</b>
        <br />DataStream &#x2192; SplitStream</td>
      <td style="text-align:left">
        <p>&#x6839;&#x636E;&#x67D0;&#x4E9B;&#x6807;&#x51C6;&#x5C06;&#x6D41;&#x62C6;&#x5206;&#x4E3A;&#x4E24;&#x4E2A;&#x6216;&#x66F4;&#x591A;&#x4E2A;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>val split = someDataStream.split(</code>
        </p>
        <p><code>  (num: Int) =&gt;</code>
        </p>
        <p><code>    (num % 2) match {</code>
        </p>
        <p><code>      case 0 =&gt; List(&quot;even&quot;)</code>
        </p>
        <p><code>      case 1 =&gt; List(&quot;odd&quot;)</code>
        </p>
        <p><code>    }</code>
        </p>
        <p><code>)</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Select</b>
        <br />SplitStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x62C6;&#x5206;&#x6D41;&#x4E2D;&#x9009;&#x62E9;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x6D41;&#x3002;
          <br
          />
        </p>
        <p><code>val even = split select &quot;even&quot;</code>
        </p>
        <p><code>val odd = split select &quot;odd&quot;</code>
        </p>
        <p><code>val all = split.select(&quot;even&quot;,&quot;odd&quot;)</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Iterate</b>
        <br />DataStream &#x2192; IterativeStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x901A;&#x8FC7;&#x5C06;&#x4E00;&#x4E2A;&#x64CD;&#x4F5C;&#x7B26;&#x7684;&#x8F93;&#x51FA;&#x91CD;&#x5B9A;&#x5411;&#x5230;&#x67D0;&#x4E2A;&#x5148;&#x524D;&#x7684;&#x64CD;&#x4F5C;&#x7B26;&#xFF0C;&#x5728;&#x6D41;&#x4E2D;&#x521B;&#x5EFA;&#x201C;&#x53CD;&#x9988;&#x201D;&#x5FAA;&#x73AF;&#x3002;&#x8FD9;&#x5BF9;&#x4E8E;&#x5B9A;&#x4E49;&#x4E0D;&#x65AD;&#x66F4;&#x65B0;&#x6A21;&#x578B;&#x7684;&#x7B97;&#x6CD5;&#x7279;&#x522B;&#x6709;&#x7528;&#x3002;&#x4EE5;&#x4E0B;&#x4EE3;&#x7801;&#x4EE5;&#x6D41;&#x5F00;&#x5934;&#x5E76;&#x8FDE;&#x7EED;&#x5E94;&#x7528;&#x8FED;&#x4EE3;&#x4F53;&#x3002;&#x5927;&#x4E8E;0&#x7684;&#x5143;&#x7D20;&#x5C06;&#x88AB;&#x53D1;&#x9001;&#x56DE;&#x53CD;&#x9988;&#x901A;&#x9053;&#xFF0C;&#x5176;&#x4F59;&#x5143;&#x7D20;&#x5C06;&#x5411;&#x4E0B;&#x6E38;&#x8F6C;&#x53D1;&#x3002;&#x6709;&#x5173;&#x5B8C;&#x6574;&#x8BF4;&#x660E;&#xFF0C;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/operators/#iterations">&#x8FED;&#x4EE3;</a>&#x3002;
            <br />
        </p>
        <p><code>initialStream.iterate {</code>
        </p>
        <p><code>  iteration =&gt; {</code>
        </p>
        <p><code>    val iterationBody = iteration.map {/*do something*/}</code>
        </p>
        <p><code>    (iterationBody.filter(_ &gt; 0), iterationBody.filter(_ &lt;= 0))</code>
        </p>
        <p><code>  }</code>
        </p>
        <p><code>}</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Extract Timestamps</b>
        <br />DataStream &#x2192; DataStream</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x8BB0;&#x5F55;&#x4E2D;&#x63D0;&#x53D6;&#x65F6;&#x95F4;&#x6233;&#xFF0C;&#x4EE5;&#x4FBF;&#x5728;&#x4F7F;&#x7528;&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;&#x8BED;&#x4E49;&#x7684;&#x7A97;&#x53E3;&#x65F6;&#x4F7F;&#x7528;&#x3002;&#x67E5;&#x770B;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-master/dev/event_time.html">&#x4E8B;&#x4EF6;&#x65F6;&#x95F4;</a>&#x3002;
            <br />
        </p>
        <p><code>stream.assignTimestamps { timestampExtractor }</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

通过匿名模式匹配从元组、case类和集合中提取，如下所示:

```scala
val data: DataStream[(Int, String, Double)] = // [...]
data.map {
  case (id, name, temperature) => // [...]
}
```

API不支持开箱即用。要使用此功能，您应该使用[Scala API扩展](https://ci.apache.org/projects/flink/flink-docs-master/dev/scala_api_extensions.html)。
{% endtab %}
{% endtabs %}

以下转换可用于元组\(Tuple\)的数据流：

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Project</b>
        <br />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x5143;&#x7EC4;&#x4E2D;&#x9009;&#x62E9;&#x5B57;&#x6BB5;&#x7684;&#x5B50;&#x96C6;</p>
        <p><code>DataStream&gt; in = // [...] </code>
        </p>
        <p><code>DataStream&gt; out = in.project(2,0);</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

## 物理分区

Flink还通过以下函数对转换后的精确流分区进行低级别控制（如果需要）。

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>&#x81EA;&#x5B9A;&#x4E49;&#x5206;&#x533A;</b>
        <br />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">&#x4F7F;&#x7528;&#x7528;&#x6237;&#x5B9A;&#x4E49;&#x7684;&#x5206;&#x533A;&#x7A0B;&#x5E8F;&#x4E3A;&#x6BCF;&#x4E2A;&#x5143;&#x7D20;&#x9009;&#x62E9;&#x76EE;&#x6807;&#x4EFB;&#x52A1;&#x3002;
        <br
        /><code>dataStream.partitionCustom(partitioner, &quot;someKey&quot;); dataStream.partitionCustom(partitioner, 0);</code>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x968F;&#x673A;&#x5206;&#x533A;</b>
        <br />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">
        <p>&#x6839;&#x636E;&#x5747;&#x5300;&#x5206;&#x5E03;&#x968F;&#x673A;&#x5206;&#x914D;&#x5143;&#x7D20;&#x3002;</p>
        <p><code>dataStream.shuffle();</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x91CD;&#x65B0;&#x5E73;&#x8861;&#xFF08;&#x5FAA;&#x73AF;&#x5206;&#x533A;&#xFF09;</b>
        <br
        />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">
        <p>&#x5FAA;&#x73AF;&#x5206;&#x533A;&#x5143;&#x7D20;&#xFF0C;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x521B;&#x5EFA;&#x76F8;&#x7B49;&#x7684;&#x8D1F;&#x8F7D;&#x3002;&#x5728;&#x5B58;&#x5728;&#x6570;&#x636E;&#x504F;&#x659C;&#x65F6;&#x7528;&#x4E8E;&#x6027;&#x80FD;&#x4F18;&#x5316;&#x3002;</p>
        <p><code>dataStream.rebalance();</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x91CD;&#x65B0;&#x8C03;&#x6574;</b>
        <br />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">
        <p>&#x5206;&#x533A;&#x5143;&#x7D20;&#xFF0C;&#x5FAA;&#x73AF;&#xFF0C;&#x5230;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x7684;&#x5B50;&#x96C6;&#x3002;&#x5982;&#x679C;&#x60A8;&#x5E0C;&#x671B;&#x62E5;&#x6709;&#x7BA1;&#x9053;&#xFF0C;&#x4F8B;&#x5982;&#xFF0C;&#x4ECE;&#x6E90;&#x7684;&#x6BCF;&#x4E2A;&#x5E76;&#x884C;&#x5B9E;&#x4F8B;&#x5411;&#x591A;&#x4E2A;&#x6620;&#x5C04;&#x5668;&#x7684;&#x5B50;&#x96C6;&#x5C55;&#x5F00;&#x4EE5;&#x5206;&#x914D;&#x8D1F;&#x8F7D;&#xFF0C;&#x4F46;&#x53C8;&#x4E0D;&#x5E0C;&#x671B;&#x4F7F;&#x7528;rebalance()&#x5E26;&#x6765;&#x7684;&#x5B8C;&#x5168;&#x91CD;&#x65B0;&#x5E73;&#x8861;&#xFF0C;&#x90A3;&#x4E48;&#x8FD9;&#x975E;&#x5E38;&#x6709;&#x7528;&#x3002;&#x8FD9;&#x5C06;&#x53EA;&#x9700;&#x8981;&#x672C;&#x5730;&#x6570;&#x636E;&#x4F20;&#x8F93;&#x800C;&#x4E0D;&#x662F;&#x901A;&#x8FC7;&#x7F51;&#x7EDC;&#x4F20;&#x8F93;&#x6570;&#x636E;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x5176;&#x4ED6;&#x914D;&#x7F6E;&#x503C;&#xFF0C;&#x4F8B;&#x5982;TaskManagers&#x7684;&#x63D2;&#x69FD;&#x6570;&#x3002;</p>
        <p>&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x53D1;&#x9001;&#x5143;&#x7D20;&#x7684;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x7684;&#x5B50;&#x96C6;&#x53D6;&#x51B3;&#x4E8E;&#x4E0A;&#x6E38;&#x548C;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x7684;&#x5E76;&#x884C;&#x5EA6;&#x3002;&#x4F8B;&#x5982;&#xFF0C;&#x5982;&#x679C;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5177;&#x6709;&#x5E76;&#x884C;&#x6027;2&#x5E76;&#x4E14;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x5177;&#x6709;&#x5E76;&#x884C;&#x6027;6&#xFF0C;&#x5219;&#x4E00;&#x4E2A;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5C06;&#x5206;&#x914D;&#x5143;&#x7D20;&#x5230;&#x4E09;&#x4E2A;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#xFF0C;&#x800C;&#x53E6;&#x4E00;&#x4E2A;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5C06;&#x5206;&#x914D;&#x5230;&#x5176;&#x4ED6;&#x4E09;&#x4E2A;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x3002;&#x53E6;&#x4E00;&#x65B9;&#x9762;&#xFF0C;&#x5982;&#x679C;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x5177;&#x6709;&#x5E76;&#x884C;&#x6027;2&#x800C;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5177;&#x6709;&#x5E76;&#x884C;&#x6027;6&#xFF0C;&#x5219;&#x4E09;&#x4E2A;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5C06;&#x5206;&#x914D;&#x5230;&#x4E00;&#x4E2A;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#xFF0C;&#x800C;&#x5176;&#x4ED6;&#x4E09;&#x4E2A;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x5C06;&#x5206;&#x914D;&#x5230;&#x53E6;&#x4E00;&#x4E2A;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x3002;</p>
        <p>&#x5728;&#x4E0D;&#x540C;&#x5E76;&#x884C;&#x5EA6;&#x4E0D;&#x662F;&#x5F7C;&#x6B64;&#x7684;&#x500D;&#x6570;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x4E00;&#x4E2A;&#x6216;&#x591A;&#x4E2A;&#x4E0B;&#x6E38;&#x64CD;&#x4F5C;&#x5C06;&#x5177;&#x6709;&#x6765;&#x81EA;&#x4E0A;&#x6E38;&#x64CD;&#x4F5C;&#x7684;&#x4E0D;&#x540C;&#x6570;&#x91CF;&#x7684;&#x8F93;&#x5165;&#x3002;</p>
        <p></p>
        <p><code>dataStream.rescale();</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left"><b>&#x5E7F;&#x64AD;</b>
        <br />DataStream&#x2192;DataStream</td>
      <td style="text-align:left">
        <p>&#x5411;&#x6BCF;&#x4E2A;&#x5206;&#x533A;&#x5E7F;&#x64AD;&#x5143;&#x7D20;&#x3002;</p>
        <p><code>dataStream.broadcast();</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>

![&#x91CD;&#x65B0;&#x8C03;&#x6574;&#x7684;&#x793A;&#x4F8B;&#x53EF;&#x89C6;&#x5316;&#x56FE;&#x5F62;](../../../.gitbook/assets/image%20%2817%29.png)

## 任务链和资源组 <a id="task-chaining-and-resource-groups"></a>

资源组是Flink中的一个插槽\(Slot\)，请参阅 [插槽](https://ci.apache.org/projects/flink/flink-docs-master/ops/config.html#configuring-taskmanager-processing-slots)。如果需要，您可以在单独的插槽中手动隔离操作符

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x8F6C;&#x6362;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">&#x5F00;&#x59CB;&#x65B0;&#x7684;Chain</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x8FD9;&#x4E2A;&#x64CD;&#x4F5C;&#x7B26;&#x5F00;&#x59CB;&#xFF0C;&#x5F00;&#x542F;&#x4E00;&#x4E2A;&#x65B0;&#x7684;Chain&#x3002;&#x4E24;&#x4E2A;&#x6620;&#x5C04;&#x5668;&#x5C06;&#x88AB;&#x94FE;&#x63A5;&#xFF0C;</p>
        <p>&#x8FC7;&#x6EE4;&#x5668;&#x5C06;&#x4E0D;&#x4F1A;&#x94FE;&#x63A5;&#x5230;&#x7B2C;&#x4E00;&#x4E2A;&#x6620;&#x5C04;&#x5668;&#x3002;</p>
        <p><code>someStream.filter(...).map(...).startNewChain().map(...);</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x7981;&#x7528;Chain</td>
      <td style="text-align:left">
        <p>&#x4E0D;&#x94FE;&#x63A5;map&#x64CD;&#x4F5C;&#x7B26;</p>
        <p><code>someStream.map(...).disableChaining();</code>
        </p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">&#x8BBE;&#x7F6E;&#x63D2;&#x69FD;(Slot)&#x5171;&#x4EAB;&#x7EC4;</td>
      <td
      style="text-align:left">
        <p>&#x8BBE;&#x7F6E;&#x64CD;&#x4F5C;&#x7684;&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x3002;Flink&#x5C06;&#x628A;&#x5177;&#x6709;&#x76F8;&#x540C;&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x7684;&#x64CD;&#x4F5C;&#x653E;&#x5165;&#x540C;&#x4E00;&#x4E2A;&#x63D2;&#x69FD;&#xFF0C;</p>
        <p>&#x540C;&#x65F6;&#x4FDD;&#x7559;&#x5176;&#x4ED6;&#x63D2;&#x69FD;&#x4E2D;&#x6CA1;&#x6709;&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x7684;&#x64CD;&#x4F5C;&#x3002;&#x8FD9;&#x53EF;&#x7528;&#x4E8E;&#x9694;&#x79BB;&#x63D2;&#x69FD;&#x3002;&#x5982;&#x679C;&#x6240;&#x6709;</p>
        <p>&#x8F93;&#x5165;&#x64CD;&#x4F5C;&#x90FD;&#x5728;&#x540C;&#x4E00;&#x4E2A;&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x4E2D;&#xFF0C;&#x5219;&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x5C06;&#x7EE7;&#x627F;&#x8F93;&#x5165;&#x64CD;&#x4F5C;&#x3002;&#x9ED8;&#x8BA4;</p>
        <p>&#x63D2;&#x69FD;&#x5171;&#x4EAB;&#x7EC4;&#x7684;&#x540D;&#x79F0;&#x4E3A;&#x201C;default&#x201D;&#xFF0C;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;&#x8C03;&#x7528;slotSharingGroup&#xFF08;&#x201C;default&#x201D;&#xFF09;&#x5C06;&#x64CD;&#x4F5C;&#x660E;&#x786E;&#x5730;&#x653E;&#x5165;&#x6B64;&#x7EC4;&#x4E2D;&#x3002;</p>
        <p><code>someStream.filter(...).slotSharingGroup(&quot;name&quot;);</code>
        </p>
        </td>
    </tr>
  </tbody>
</table>

