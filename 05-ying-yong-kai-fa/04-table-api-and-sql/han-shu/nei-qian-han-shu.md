# 内置函数

Flink Table API和SQL为用户提供了一组用于数据转换的内置函数。本页简要概述了它们。如果尚不支持您需要的函数，则可以实现[用户定义的](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/udfs.html)函数。如果您认为自定义的函数足够通用，请[打开一个Jira问题](https://issues.apache.org/jira/secure/CreateIssue!default.jspa)，并附上详细说明。

## 标量函数

标量函数将零，一个或多个值作为输入，并返回单个值作为结果。

### 比较函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6BD4;&#x8F83;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">value1 = value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x7B49;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 &lt;&gt; value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x4E0D;&#x7B49;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 &gt; value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x5927;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 &gt;= value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 &lt; value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x5C0F;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 &lt;= value2</td>
      <td style="text-align:left">&#x5982;&#x679C;value1&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;value2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;value1&#x6216;value2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value IS NULL</td>
      <td style="text-align:left">&#x5982;&#x679C;value&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value IS NOT NULL</td>
      <td style="text-align:left">&#x5982;&#x679C;value&#x4E0D;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">value1 IS DISTINCT FROM value2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x4E24;&#x4E2A;&#x503C;&#x4E0D;&#x76F8;&#x7B49;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;NULL&#x503C;&#x5728;&#x6B64;&#x5904;&#x89C6;&#x4E3A;&#x76F8;&#x540C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>1 IS DISTINCT FROM NULL</code>&#x8FD4;&#x56DE;TRUE; <code>NULL IS DISTINCT FROM NULL</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value1 IS NOT DISTINCT FROM value2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x4E24;&#x4E2A;&#x503C;&#x76F8;&#x7B49;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;NULL&#x503C;&#x5728;&#x6B64;&#x5904;&#x89C6;&#x4E3A;&#x76F8;&#x540C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>1 IS NOT DISTINCT FROM NULL</code>&#x8FD4;&#x56DE;FALSE; <code>NULL IS NOT DISTINCT FROM NULL</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value1 BETWEEN [ ASYMMETRIC | SYMMETRIC ] value2 AND value3</td>
      <td style="text-align:left">
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF08;&#x6216;&#x4F7F;&#x7528;ASYMMETRIC&#x5173;&#x952E;&#x5B57;&#xFF09;&#xFF0C;&#x5982;&#x679C;value1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;value2&#x4E14;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;value3&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x4F7F;&#x7528;SYMMETRIC&#x5173;&#x952E;&#x5B57;&#xFF0C;&#x5982;&#x679C;value1&#x5305;&#x542B;&#x5728;value2&#x548C;value3&#x4E4B;&#x95F4;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;value2&#x6216;value3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;FALSE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12 BETWEEN 15 AND 12</code>&#x8FD4;&#x56DE;FALSE; <code>12 BETWEEN SYMMETRIC 15 AND 12</code>&#x8FD4;&#x56DE;TRUE; <code>12 BETWEEN 10 AND NULL</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12 BETWEEN NULL AND 10</code>&#x8FD4;&#x56DE;FALSE; <code>12 BETWEEN SYMMETRIC NULL AND 12</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value1 NOT BETWEEN [ ASYMMETRIC | SYMMETRIC ] value2 AND value3</td>
      <td
      style="text-align:left">
        <p>&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF08;&#x6216;&#x4F7F;&#x7528;ASYMMETRIC&#x5173;&#x952E;&#x5B57;&#xFF09;&#xFF0C;&#x5982;&#x679C;value1&#x5C0F;&#x4E8E;value2&#x6216;&#x5927;&#x4E8E;value3&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x4F7F;&#x7528;SYMMETRIC&#x5173;&#x952E;&#x5B57;&#xFF0C;&#x5982;&#x679C;value1&#x4E0D;&#x5728;value2&#x548C;value3&#x4E4B;&#x95F4;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;value2&#x6216;value3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;TRUE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12 NOT BETWEEN 15 AND 12</code>&#x8FD4;&#x56DE;TRUE; <code>12 NOT BETWEEN SYMMETRIC 15 AND 12</code>&#x8FD4;&#x56DE;FALSE; <code>12 NOT BETWEEN NULL AND 15</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12 NOT BETWEEN 15 AND NULL</code>&#x8FD4;&#x56DE;TRUE; <code>12 NOT BETWEEN SYMMETRIC 12 AND NULL</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">string1 LIKE string2 [ ESCAPE char ]</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;string1&#x5339;&#x914D;&#x6A21;&#x5F0F;string2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;string1&#x6216;string2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;&#x5982;&#x6709;&#x5FC5;&#x8981;&#xFF0C;&#x53EF;&#x4EE5;&#x5B9A;&#x4E49;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5C1A;&#x672A;&#x652F;&#x6301;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">string1 NOT LIKE string2 [ ESCAPE char ]</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;string1&#x4E0E;&#x6A21;&#x5F0F;string2&#x4E0D;&#x5339;&#x914D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;string1&#x6216;string2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;&#x5982;&#x6709;&#x5FC5;&#x8981;&#xFF0C;&#x53EF;&#x4EE5;&#x5B9A;&#x4E49;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5C1A;&#x672A;&#x652F;&#x6301;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">string1 SIMILAR TO string2 [ ESCAPE char ]</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;string1&#x5339;&#x914D;SQL&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;string1&#x6216;string2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;&#x5982;&#x6709;&#x5FC5;&#x8981;&#xFF0C;&#x53EF;&#x4EE5;&#x5B9A;&#x4E49;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5C1A;&#x672A;&#x652F;&#x6301;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">string1 NOT SIMILAR TO string2 [ ESCAPE char ]</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;string1&#x4E0E;SQL&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x4E0D;&#x5339;&#x914D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5982;&#x679C;string1&#x6216;string2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;&#x5982;&#x6709;&#x5FC5;&#x8981;&#xFF0C;&#x53EF;&#x4EE5;&#x5B9A;&#x4E49;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5C1A;&#x672A;&#x652F;&#x6301;&#x8F6C;&#x4E49;&#x5B57;&#x7B26;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value1 IN (value2 [, value3]* )</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x5217;&#x8868;&#x4E2D;&#x5B58;&#x5728;value1
          &#xFF08;value2&#xFF0C;value3&#xFF0C;...&#xFF09;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;&#xFF08;value2&#xFF0C;value3&#xFF0C;...&#xFF09;&#x3002;&#x5305;&#x542B;NULL&#xFF0C;&#x5982;&#x679C;&#x53EF;&#x4EE5;&#x627E;&#x5230;&#x8BE5;&#x5143;&#x7D20;&#x5219;&#x8FD4;&#x56DE;TRUE&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;&#x5982;&#x679C;value1&#x4E3A;NULL&#xFF0C;&#x5219;&#x59CB;&#x7EC8;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4 IN (1, 2, 3)</code>&#x8FD4;&#x56DE;FALSE; <code>1 IN (1, 2, NULL)</code>&#x8FD4;&#x56DE;TRUE; <code>4 IN (1, 2, NULL)</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value1 NOT IN (value2 [, value3]* )</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x5217;&#x8868;&#x4E2D;&#x4E0D;&#x5B58;&#x5728;value1
          &#xFF08;value2&#xFF0C;value3&#xFF0C;...&#xFF09;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;&#xFF08;value2&#xFF0C;value3&#xFF0C;...&#xFF09;&#x3002;&#x5305;&#x542B;NULL&#xFF0C;&#x5982;&#x679C;&#x53EF;&#x4EE5;&#x627E;&#x5230;value1&#x5219;&#x8FD4;&#x56DE;FALSE
          &#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;&#x5982;&#x679C;value1&#x4E3A;NULL&#xFF0C;&#x5219;&#x59CB;&#x7EC8;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4 NOT IN (1, 2, 3)</code>&#x8FD4;&#x56DE;TRUE; <code>1 NOT IN (1, 2, NULL)</code>&#x8FD4;&#x56DE;FALSE;<code>4 NOT IN (1, 2, NULL)</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">EXISTS (sub-query)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x5B50;&#x67E5;&#x8BE2;&#x8FD4;&#x56DE;&#x81F3;&#x5C11;&#x4E00;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x4EC5;&#x5728;&#x53EF;&#x4EE5;&#x5728;&#x8FDE;&#x63A5;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x64CD;&#x4F5C;&#x65F6;&#x624D;&#x652F;&#x6301;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value IN (sub-query)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;value&#x7B49;&#x4E8E;&#x5B50;&#x67E5;&#x8BE2;&#x8FD4;&#x56DE;&#x7684;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">value NOT IN (sub-query)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;value&#x4E0D;&#x7B49;&#x4E8E;&#x5B50;&#x67E5;&#x8BE2;&#x8FD4;&#x56DE;&#x7684;&#x6BCF;&#x4E00;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6BD4;&#x8F83;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY1 === ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 !== ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C; ANY1&#x4E0D;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE;
        &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNotNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E0D;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.like(STRING2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x5339;&#x914D;&#x6A21;&#x5F0F;STRING2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;JoKn&quot;.like(&quot;Jo_n%&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.similar(STRING)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x4E0E;SQL&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;STRING2&#x5339;&#x914D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;A&quot;.similar(&quot;A+&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.in(ANY2, ANY3, ...)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x5217;&#x8868;&#x4E2D;&#x5B58;&#x5728;ANY1
          &#xFF08;ANY2&#xFF0C;ANY3&#xFF0C;...&#xFF09;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;&#xFF08;ANY2&#xFF0C;&#x4EFB;&#x4F55;3&#xFF0C;...&#xFF09;
          &#x3002;&#x5305;&#x542B;NULL&#xFF0C;&#x5982;&#x679C;&#x53EF;&#x4EE5;&#x627E;&#x5230;&#x8BE5;&#x5143;&#x7D20;&#x5219;&#x8FD4;&#x56DE;TRUE&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;&#x5982;&#x679C;ANY1&#x4E3A;NULL&#xFF0C;&#x5219;&#x59CB;&#x7EC8;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.in(1, 2, 3)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.in(TABLE)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY&#x7B49;&#x4E8E;&#x5B50;&#x67E5;&#x8BE2;TABLE&#x8FD4;&#x56DE;&#x7684;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.between(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x4E14;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;FALSE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.between(15, 12)</code>&#x8FD4;&#x56DE;FALSE; <code>12.between(10, Null(INT))</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12.between(Null(INT), 10)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.notBetween(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2&#x6216;&#x5927;&#x4E8E;ANY3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;TRUE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.notBetween(15, 12)</code>&#x8FD4;&#x56DE;TRUE; <code>12.notBetween(Null(INT), 15)</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12.notBetween(15, Null(INT))</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6BD4;&#x8F83;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY1 === ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 !== ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C; ANY1&#x4E0D;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE;
        &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNotNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E0D;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.like(STRING2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x5339;&#x914D;&#x6A21;&#x5F0F;STRING2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;JoKn&quot;.like(&quot;Jo_n%&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.similar(STRING)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x4E0E;SQL&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;STRING2&#x5339;&#x914D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;A&quot;.similar(&quot;A+&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.in(ANY2, ANY3, ...)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x5217;&#x8868;&#x4E2D;&#x5B58;&#x5728;ANY1
          &#xFF08;ANY2&#xFF0C;ANY3&#xFF0C;...&#xFF09;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;&#xFF08;ANY2&#xFF0C;&#x4EFB;&#x4F55;3&#xFF0C;...&#xFF09;
          &#x3002;&#x5305;&#x542B;NULL&#xFF0C;&#x5982;&#x679C;&#x53EF;&#x4EE5;&#x627E;&#x5230;&#x8BE5;&#x5143;&#x7D20;&#x5219;&#x8FD4;&#x56DE;TRUE&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;&#x5982;&#x679C;ANY1&#x4E3A;NULL&#xFF0C;&#x5219;&#x59CB;&#x7EC8;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.in(1, 2, 3)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.in(TABLE)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY&#x7B49;&#x4E8E;&#x5B50;&#x67E5;&#x8BE2;TABLE&#x8FD4;&#x56DE;&#x7684;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.between(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x4E14;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;FALSE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.between(15, 12)</code>&#x8FD4;&#x56DE;FALSE; <code>12.between(10, Null(INT))</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12.between(Null(INT), 10)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.notBetween(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2&#x6216;&#x5927;&#x4E8E;ANY3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;TRUE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.notBetween(15, 12)</code>&#x8FD4;&#x56DE;TRUE; <code>12.notBetween(Null(INT), 15)</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12.notBetween(15, Null(INT))</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6BD4;&#x8F83;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY1 === ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 !== ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x4E0D;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE ;
        &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &gt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2 &#x8FD4;&#x56DE;TRUE;
        &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt; ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1 &lt;= ANY2</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;ANY1&#x6216;ANY2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.isNotNull</td>
      <td style="text-align:left">&#x5982;&#x679C;ANY&#x4E0D;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.like(STRING2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x5339;&#x914D;&#x6A21;&#x5F0F;STRING2&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;JoKn&quot;.like(&quot;Jo_n%&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.similar(STRING2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;STRING1&#x4E0E;SQL&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;STRING2&#x5339;&#x914D;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          ; &#x5982;&#x679C;STRING1&#x6216;STRING2&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&quot;A&quot;.similar(&quot;A+&quot;)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.in(ANY2, ANY3, ...)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;&#x7ED9;&#x5B9A;&#x5217;&#x8868;&#x4E2D;&#x5B58;&#x5728;ANY1
          &#xFF08;ANY2&#xFF0C;ANY3&#xFF0C;...&#xFF09;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x5F53;&#xFF08;ANY2&#xFF0C;&#x4EFB;&#x4F55;3&#xFF0C;...&#xFF09;
          &#x3002;&#x5305;&#x542B;NULL&#xFF0C;&#x5982;&#x679C;&#x53EF;&#x4EE5;&#x627E;&#x5230;&#x8BE5;&#x5143;&#x7D20;&#x5219;&#x8FD4;&#x56DE;TRUE&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;&#x5982;&#x679C;ANY1&#x4E3A;NULL&#xFF0C;&#x5219;&#x59CB;&#x7EC8;&#x8FD4;&#x56DE;UNKNOWN
          &#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.in(1, 2, 3)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.in(TABLE)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY&#x7B49;&#x4E8E;&#x5B50;&#x67E5;&#x8BE2;TABLE&#x8FD4;&#x56DE;&#x7684;&#x884C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x5BF9;&#x4E8E;&#x6D41;&#x5F0F;&#x67E5;&#x8BE2;&#xFF0C;&#x64CD;&#x4F5C;&#x5C06;&#x5728;&#x5173;&#x8054;&#x548C;&#x7EC4;&#x64CD;&#x4F5C;&#x4E2D;&#x91CD;&#x5199;&#x3002;&#x8BA1;&#x7B97;&#x67E5;&#x8BE2;&#x7ED3;&#x679C;&#x6240;&#x9700;&#x7684;&#x72B6;&#x6001;&#x53EF;&#x80FD;&#x4F1A;&#x65E0;&#x9650;&#x589E;&#x957F;&#xFF0C;&#x5177;&#x4F53;&#x53D6;&#x51B3;&#x4E8E;&#x4E0D;&#x540C;&#x8F93;&#x5165;&#x884C;&#x7684;&#x6570;&#x91CF;&#x3002;&#x8BF7;&#x63D0;&#x4F9B;&#x5177;&#x6709;&#x6709;&#x6548;&#x4FDD;&#x7559;&#x95F4;&#x9694;&#x7684;&#x67E5;&#x8BE2;&#x914D;&#x7F6E;&#xFF0C;&#x4EE5;&#x9632;&#x6B62;&#x72B6;&#x6001;&#x8FC7;&#x5927;&#x3002;&#x8BF7;&#x53C2;&#x9605;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/streaming/query_configuration.html">&#x67E5;&#x8BE2;&#x914D;&#x7F6E;</a>&#x4E86;&#x89E3;&#x8BE6;&#x7EC6;&#x4FE1;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.between(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANY2&#x4E14;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;ANT3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;FALSE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.between(15, 12)</code>&#x8FD4;&#x56DE;FALSE; <code>12.between(10, Null(Types.INT))</code>&#x8FD4;&#x56DE;UNKNOWN; <code>12.between(Null(Types.INT), 10)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY1.notBetween(ANY2, ANY3)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;ANY1&#x5C0F;&#x4E8E;ANY2&#x6216;&#x5927;&#x4E8E;&#x4EFB;&#x4F55;3&#x8FD4;&#x56DE;TRUE&#x3002;&#x5F53;ANY2&#x6216;ANY3&#x4E3A;NULL&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;TRUE&#x6216;UNKNOWN&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>12.notBetween(15, 12)</code>&#x8FD4;&#x56DE;TRUE;<code>12.notBetween(Null(Types.INT), 15)</code>&#x8FD4;&#x56DE;UNKNOWN;<code>12.notBetween(15, Null(Types.INT))</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 逻辑函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x903B;&#x8F91;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">boolean1 OR boolean2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;boolean1&#x4E3A;TRUE&#x6216;boolean2&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>TRUE OR UNKNOWN</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">boolean1 AND boolean2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;boolean1&#x548C;boolean2&#x90FD;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>TRUE AND UNKNOWN</code>&#x8FD4;&#x56DE;UNKNOWN</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NOT boolean</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;boolean&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        ; &#x5982;&#x679C;&#x5E03;&#x5C14;&#x503C;&#x4E3A;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS FALSE</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5982;&#x679C;boolean&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS NOT FALSE</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;boolean&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS TRUE</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;
        &#x5982;&#x679C;boolean&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS NOT TRUE</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;boolean&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS UNKNOWN</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x662F;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;boolean&#x4E3A;TRUE&#x6216;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">boolean IS NOT UNKNOWN</td>
      <td style="text-align:left">&#x5982;&#x679C;boolean&#x4E3A;TRUE&#x6216;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;boolean&#x4E3A;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x903B;&#x8F91;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN1 || BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x4E3A;TRUE&#x6216;BOOLEAN2&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true || Null(BOOLEAN)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN1 &amp;&amp; BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x548C;BOOLEAN2&#x90FD;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true &amp;&amp; Null(BOOLEAN)</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">!BOOLEAN</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        ; &#x5982;&#x679C;BOOLEAN&#x672A;&#x77E5;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;
        &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x903B;&#x8F91;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN1 || BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x4E3A;TRUE&#x6216;BOOLEAN2&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true || Null(Types.BOOLEAN)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN1 &amp;&amp; BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x548C;BOOLEAN2&#x90FD;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true &amp;&amp; Null(Types.BOOLEAN)</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">!BOOLEAN</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        ; &#x5982;&#x679C;BOOLEAN&#x672A;&#x77E5;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;
        &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x903B;&#x8F91;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN1 || BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x4E3A;TRUE&#x6216;BOOLEAN2&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true || Null(Types.BOOLEAN)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN1 &amp;&amp; BOOLEAN2</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN1&#x548C;BOOLEAN2&#x90FD;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x652F;&#x6301;&#x4E09;&#x503C;&#x903B;&#x8F91;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>true &amp;&amp; Null(Types.BOOLEAN)</code>&#x8FD4;&#x56DE;UNKNOWN&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">!BOOLEAN</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        ; &#x5982;&#x679C;BOOLEAN&#x672A;&#x77E5;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;UNKNOWN&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;
        &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
        ; &#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotTrue</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BOOLEAN.isNotFalse</td>
      <td style="text-align:left">&#x5982;&#x679C;BOOLEAN&#x4E3A;TRUE&#x6216;UNKNOWN&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE;&#x5426;&#x5219;&#x8FD4;&#x56DE;TRUE
        &#x3002;&#x5982;&#x679C;BOOLEAN&#x4E3A;FALSE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;FALSE&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 算数函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B97;&#x6570;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">+ numeric</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">- numeric</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x8D1F;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1 + numeric2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x52A0;&#x4E0A;numeric2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1 - numeric2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x51CF;&#x53BB;numeric2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1 * numeric2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x4E58;&#x4EE5;numeric2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1 / numeric2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x9664;&#x4EE5;numeric2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">POWER(numeric1, numeric2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x7684;numeric2&#x7684;&#x5E42;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ABS(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric&#x7684;&#x7EDD;&#x5BF9;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">MOD(numeric1, numeric2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x9664;&#x4EE5;numeric2&#x7684;&#x4F59;&#x6570;(&#x6A21;&#x6570;)&#x3002;&#x4EC5;&#x5F53;numeric1&#x4E3A;&#x8D1F;&#x6570;&#x65F6;&#xFF0C;&#x7ED3;&#x679C;&#x624D;&#x4E3A;&#x8D1F;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">SQRT(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x5B57;&#x7684;&#x5E73;&#x65B9;&#x6839;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#xFF08;&#x4EE5;e&#x4E3A;&#x5E95;&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">LOG10(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x4EE5;10&#x4E3A;&#x5E95;&#x7684;&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LOG2(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x4EE5;2&#x4E3A;&#x5E95;&#x7684;&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>LOG(numeric2)</p>
        <p>LOG(numeric1, numeric2)</p>
      </td>
      <td style="text-align:left">
        <p>&#x4F7F;&#x7528;&#x4E00;&#x4E2A;&#x53C2;&#x6570;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;numeric2&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#x3002;&#x5F53;&#x4F7F;&#x7528;&#x4E24;&#x4E2A;&#x53C2;&#x6570;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x6B64;&#x51FD;&#x6570;&#x5C06;numeric2&#x7684;&#x5BF9;&#x6570;&#x8FD4;&#x56DE;&#x5230;&#x57FA;&#x6570;numeric1&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x76EE;&#x524D;&#xFF0C;numeric2&#x5FC5;&#x987B;&#x5927;&#x4E8E;0&#x4E14;numeric1&#x5FC5;&#x987B;&#x5927;&#x4E8E;1&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">EXP(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;e&#x7684;numeric&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>CEIL(numeric)</p>
        <p>CEILING(numeric)</p>
      </td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5C0F;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">FLOOR(numeric)</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5927;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">SIN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">SINH(numeric)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">COS(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TAN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TANH(numeric)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5207;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">COT(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x4F59;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ASIN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CD;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ACOS(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CD;&#x4F59;&#x5F26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ATAN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CD;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ATAN2(numeric1, numeric2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5750;&#x6807;(numeric1, numeric2)&#x7684;&#x53CD;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">COSH(numeric)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x53CC;&#x66F2;&#x4F59;&#x5F26;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x503C;&#x7C7B;&#x578B;&#x4E3A;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DEGREES(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5F27;&#x5EA6;&#x6570;&#x503C;&#x7684;&#x5EA6;&#x6570;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">RADIANS(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5EA6;&#x6570;&#x503C;&#x7684;&#x5F27;&#x5EA6;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">SIGN(numeric)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x503C;&#x7684;&#x7B26;&#x53F7;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">ROUND(numeric, integer)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6574;&#x6570;&#x5C0F;&#x6570;&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">PI</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;pi&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">E()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;e&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">RAND()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4ECB;&#x4E8E;0.0(&#x5305;&#x62EC;)&#x548C;1.0(&#x4E0D;&#x5305;&#x62EC;)&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">RAND(integer)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x4ECB;&#x4E8E;0.0(&#x5305;&#x542B;)&#x548C;1.0(&#x4E0D;&#x5305;&#x542B;)&#x4E4B;&#x95F4;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x6574;&#x6570;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;RAND&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#xFF0C;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">RAND_INTEGER(integer)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4ECB;&#x4E8E;0(&#x5305;&#x62EC;)&#x548C;integer(&#x6392;&#x9664;)&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x6574;&#x6570;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">RAND_INTEGER(integer1, integer2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4ECB;&#x4E8E;0(&#x5305;&#x542B;)&#x548C;&#x6307;&#x5B9A;&#x503C;(&#x6392;&#x9664;)&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x6574;&#x6570;&#x503C;&#xFF0C;&#x5E76;&#x5E26;&#x6709;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;RAND_INTEGER&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x548C;&#x8FB9;&#x754C;&#xFF0C;&#x90A3;&#x4E48;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">UUID()</td>
      <td style="text-align:left">&#x6839;&#x636E;RFC 4122 type 4(&#x4F2A;&#x968F;&#x673A;&#x751F;&#x6210;)UUID&#x8FD4;&#x56DE;UUID
        (universal Unique Identifier)&#x5B57;&#x7B26;&#x4E32;(&#x4F8B;&#x5982;&#xFF0C;&#x201C;3d3c68f7-f608-473f-b60c-b0c44ad4cc4e&#x201D;)&#x3002;UUID&#x662F;&#x4F7F;&#x7528;&#x52A0;&#x5BC6;&#x7684;&#x5F3A;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x751F;&#x6210;&#x5668;&#x751F;&#x6210;&#x7684;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">BIN(integer)</td>
      <td style="text-align:left">
        <p>&#x4EE5;&#x4E8C;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x8FD4;&#x56DE;&#x6574;&#x6570;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;&#x5982;&#x679C;integer&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>BIN(4)</code>&#x8FD4;&#x56DE;&apos;100&apos;&#x5E76;<code>BIN(12)</code>&#x8FD4;&#x56DE;&apos;1100&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>HEX(numeric)</p>
        <p>HEX(string)</p>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6574;&#x6570;&#x6570;&#x503C;&#x6216;&#x5341;&#x516D;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;&#x5982;&#x679C;&#x53C2;&#x6570;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x6570;&#x5B57;20&#x5BFC;&#x81F4;&#x201C;14&#x201D;&#xFF0C;&#x6570;&#x5B57;100&#x5BFC;&#x81F4;&#x201C;64&#x201D;&#xFF0C;&#x5B57;&#x7B26;&#x4E32;&#x201C;hello&#xFF0C;world&#x201D;&#x5BFC;&#x81F4;&#x201C;68656C6C6F2C776F726C64&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TRUNCATE(numeric1, integer2)</td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">PI()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x3C0;&#xFF08;pi&#xFF09;&#x7684;&#x503C;&#x3002;</p>
        <p>&#x4EC5;&#x5728;Blink Planner&#x4E2D;&#x652F;&#x6301;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B97;&#x6570;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">+ NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">- NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x8D1F;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 + NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x52A0;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 - NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x51CF;&#x53BB;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 * NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x4E58;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 / NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x9664;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1.power(NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x7684;numeric2&#x7684;&#x5E42;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.abs()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7EDD;&#x5BF9;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 % NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x9664;&#x4EE5;numeric2&#x7684;&#x4F59;&#x6570;(&#x6A21;&#x6570;)&#x3002;&#x4EC5;&#x5F53;numeric1&#x4E3A;&#x8D1F;&#x6570;&#x65F6;&#xFF0C;&#x7ED3;&#x679C;&#x624D;&#x4E3A;&#x8D1F;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sqrt()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x5B57;&#x7684;&#x5E73;&#x65B9;&#x6839;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ln()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#xFF08;&#x57FA;&#x6570;e&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log10()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;10&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log2()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;2&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC1.log()</p>
        <p>NUMERIC1.log(NUMERIC2)</p>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x4E0D;&#x5E26;&#x53C2;&#x6570;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;NUMERIC1&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#x3002;&#x4F7F;&#x7528;&#x53C2;&#x6570;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x5C06;NUMERIC1&#x7684;&#x5BF9;&#x6570;&#x8FD4;&#x56DE;&#x5230;&#x57FA;&#x6570;NUMERIC2&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x76EE;&#x524D;&#xFF0C;NUMERIC1&#x5FC5;&#x987B;&#x5927;&#x4E8E;0&#x4E14;NUMERIC2&#x5FC5;&#x987B;&#x5927;&#x4E8E;1&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.exp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;e&#x7684;&#x6570;&#x503C;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ceil()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5C0F;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.floor()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5927;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sinh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5207;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tanh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5207;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cot()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.asin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.acos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.atan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">atan2(NUMERIC1, NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5750;&#x6807;&#x7684;&#x53CD;&#x6B63;&#x5207;&#xFF08;NUMERIC1&#xFF0C;NUMERIC2&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cosh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x4F59;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x503C;&#x7C7B;&#x578B;&#x4E3A;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.degrees()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5F27;&#x5EA6;NUMERIC&#x7684;&#x5EA6;&#x6570;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.radians()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x5EA6;&#x7684;&#x5F27;&#x5EA6;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sign()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7B26;&#x53F7;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.round(INT)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6574;&#x6570;&#x5C0F;&#x6570;&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">pi()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;pi&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">e()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;e&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4ECB;&#x4E8E;0.0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;1.0&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x4ECB;&#x4E8E;0.0(&#x5305;&#x542B;)&#x548C;1.0(&#x4E0D;&#x5305;&#x542B;)&#x4E4B;&#x95F4;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x6574;&#x6570;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;RAND&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#xFF0C;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER1, INTEGER2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER2&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x4E3A;INTEGER1&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;randInteger&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x548C;&#x7ED1;&#x5B9A;&#xFF0C;&#x5219;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">uuid()</td>
      <td style="text-align:left">&#x6839;&#x636E;RFC 4122&#x7C7B;&#x578B;4&#xFF08;&#x4F2A;&#x968F;&#x673A;&#x751F;&#x6210;&#x7684;&#xFF09;UUID&#x8FD4;&#x56DE;UUID&#xFF08;&#x901A;&#x7528;&#x552F;&#x4E00;&#x6807;&#x8BC6;&#x7B26;&#xFF09;&#x5B57;&#x7B26;&#x4E32;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x201C;3d3c68f7-f608-473f-b60c-b0c44ad4cc4e&#x201D;&#xFF09;&#x3002;&#x4F7F;&#x7528;&#x52A0;&#x5BC6;&#x5F3A;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x751F;&#x6210;&#x5668;&#x751F;&#x6210;UUID&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">INTEGER.bin()</td>
      <td style="text-align:left">
        <p>&#x4EE5;&#x4E8C;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x8FD4;&#x56DE;INTEGER&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;&#x5982;&#x679C;INTEGER&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.bin()</code>&#x8FD4;&#x56DE;&#x201C;100&#x201D;&#x5E76;<code>12.bin()</code>&#x8FD4;&#x56DE;&#x201C;1100&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.hex()</p>
        <p>STRING.hex()</p>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6574;&#x6570;NUMERIC&#x503C;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x6216;&#x5341;&#x516D;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x7684;STRING&#x3002;&#x5982;&#x679C;&#x53C2;&#x6570;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x6570;&#x5B57;20&#x5BFC;&#x81F4;&#x201C;14&#x201D;&#xFF0C;&#x6570;&#x5B57;100&#x5BFC;&#x81F4;&#x201C;64&#x201D;&#xFF0C;&#x5B57;&#x7B26;&#x4E32;&#x201C;hello&#xFF0C;world&#x201D;&#x5BFC;&#x81F4;&#x201C;68656C6C6F2C776F726C64&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1.truncate(INTEGER2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x622A;&#x65AD;&#x4E3A;&#x5C0F;&#x6570;&#x70B9;&#x540E;2&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;&#x5982;&#x679C;numeric1&#x6216;integer2&#x4E3A;&#x7A7A;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x7A7A;&#x3002;&#x5982;&#x679C;&#x6574;&#x6570;2&#x4E3A;0&#xFF0C;&#x7ED3;&#x679C;&#x6CA1;&#x6709;&#x5C0F;&#x6570;&#x70B9;&#x6216;&#x5C0F;&#x6570;&#x70B9;&#x90E8;&#x5206;&#x6574;&#x6570;2&#x53EF;&#x4EE5;&#x662F;&#x8D1F;&#x6570;&#xFF0C;&#x4F7F;&#x503C;&#x7684;&#x5C0F;&#x6570;&#x70B9;&#x5DE6;&#x8FB9;&#x7684;&#x6574;&#x6570;2&#x4F4D;&#x53D8;&#x4E3A;&#x8D1F;&#x6570;&#x96F6;&#x3002;&#x8FD9;&#x4E2A;&#x51FD;&#x6570;&#x4E5F;&#x53EA;&#x80FD;&#x4F20;&#x5165;&#x4E00;&#x4E2A;numeric1&#x53C2;&#x6570;&#xFF0C;&#x800C;&#x4E0D;&#x80FD;&#x5C06;Integer2&#x8BBE;&#x7F6E;&#x4E3A;&#x4F7F;&#x7528;&#x3002;&#x5982;&#x679C;&#x672A;&#x8BBE;&#x7F6E;&#x6574;&#x6570;2&#xFF0C;&#x51FD;&#x6570;&#x4F1A;&#x622A;&#x65AD;&#xFF0C;&#x5C31;&#x50CF;Integer2&#x662F;0&#x4E00;&#x6837;&#x3002;</p>
        <p>&#x4F8B;&#x5982;42.324.truncate(2)&#x4E3A;42.34&#x3002;&#x4EE5;&#x53CA;42.324.truncate()&#x4E3A;42.0&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B97;&#x6570;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">+ NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">- NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x8D1F;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 + NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x52A0;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 - NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x51CF;&#x53BB;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 * NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x4E58;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 / NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x9664;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1.power(NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x7684;numeric2&#x7684;&#x5E42;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.abs()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7EDD;&#x5BF9;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 % NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x9664;&#x4EE5;numeric2&#x7684;&#x4F59;&#x6570;(&#x6A21;&#x6570;)&#x3002;&#x4EC5;&#x5F53;numeric1&#x4E3A;&#x8D1F;&#x6570;&#x65F6;&#xFF0C;&#x7ED3;&#x679C;&#x624D;&#x4E3A;&#x8D1F;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sqrt()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x5B57;&#x7684;&#x5E73;&#x65B9;&#x6839;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ln()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#xFF08;&#x57FA;&#x6570;e&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log10()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;10&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log2()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;2&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC1.log()</p>
        <p>NUMERIC1.log(NUMERIC2)</p>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x4E0D;&#x5E26;&#x53C2;&#x6570;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;NUMERIC1&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#x3002;&#x4F7F;&#x7528;&#x53C2;&#x6570;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x5C06;NUMERIC1&#x7684;&#x5BF9;&#x6570;&#x8FD4;&#x56DE;&#x5230;&#x57FA;&#x6570;NUMERIC2&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x76EE;&#x524D;&#xFF0C;NUMERIC1&#x5FC5;&#x987B;&#x5927;&#x4E8E;0&#x4E14;NUMERIC2&#x5FC5;&#x987B;&#x5927;&#x4E8E;1&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.exp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;e&#x7684;&#x6570;&#x503C;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ceil()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5C0F;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.floor()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5927;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sinh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5207;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tanh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5207;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cot()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.asin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.acos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.atan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">atan2(NUMERIC1, NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5750;&#x6807;&#x7684;&#x53CD;&#x6B63;&#x5207;&#xFF08;NUMERIC1&#xFF0C;NUMERIC2&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cosh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x4F59;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x503C;&#x7C7B;&#x578B;&#x4E3A;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.degrees()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5F27;&#x5EA6;NUMERIC&#x7684;&#x5EA6;&#x6570;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.radians()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x5EA6;&#x7684;&#x5F27;&#x5EA6;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sign()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7B26;&#x53F7;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.round(INT)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6574;&#x6570;&#x5C0F;&#x6570;&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">pi()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;pi&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">e()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;e&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4ECB;&#x4E8E;0.0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;1.0&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x4ECB;&#x4E8E;0.0(&#x5305;&#x542B;)&#x548C;1.0(&#x4E0D;&#x5305;&#x542B;)&#x4E4B;&#x95F4;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x6574;&#x6570;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;RAND&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#xFF0C;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER1, INTEGER2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER2&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x4E3A;INTEGER1&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;randInteger&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x548C;&#x7ED1;&#x5B9A;&#xFF0C;&#x5219;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">uuid()</td>
      <td style="text-align:left">&#x6839;&#x636E;RFC 4122&#x7C7B;&#x578B;4&#xFF08;&#x4F2A;&#x968F;&#x673A;&#x751F;&#x6210;&#x7684;&#xFF09;UUID&#x8FD4;&#x56DE;UUID&#xFF08;&#x901A;&#x7528;&#x552F;&#x4E00;&#x6807;&#x8BC6;&#x7B26;&#xFF09;&#x5B57;&#x7B26;&#x4E32;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x201C;3d3c68f7-f608-473f-b60c-b0c44ad4cc4e&#x201D;&#xFF09;&#x3002;&#x4F7F;&#x7528;&#x52A0;&#x5BC6;&#x5F3A;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x751F;&#x6210;&#x5668;&#x751F;&#x6210;UUID&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">INTEGER.bin()</td>
      <td style="text-align:left">
        <p>&#x4EE5;&#x4E8C;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x8FD4;&#x56DE;INTEGER&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;&#x5982;&#x679C;INTEGER&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.bin()</code>&#x8FD4;&#x56DE;&#x201C;100&#x201D;&#x5E76;<code>12.bin()</code>&#x8FD4;&#x56DE;&#x201C;1100&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.hex()</p>
        <p>STRING.hex()</p>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6574;&#x6570;NUMERIC&#x503C;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x6216;&#x5341;&#x516D;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x7684;STRING&#x3002;&#x5982;&#x679C;&#x53C2;&#x6570;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x6570;&#x5B57;20&#x5BFC;&#x81F4;&#x201C;14&#x201D;&#xFF0C;&#x6570;&#x5B57;100&#x5BFC;&#x81F4;&#x201C;64&#x201D;&#xFF0C;&#x5B57;&#x7B26;&#x4E32;&#x201C;hello&#xFF0C;world&#x201D;&#x5BFC;&#x81F4;&#x201C;68656C6C6F2C776F726C64&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">numeric1.truncate(INTEGER2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x622A;&#x65AD;&#x4E3A;&#x5C0F;&#x6570;&#x70B9;&#x540E;2&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;&#x5982;&#x679C;numeric1&#x6216;integer2&#x4E3A;&#x7A7A;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x7A7A;&#x3002;&#x5982;&#x679C;&#x6574;&#x6570;2&#x4E3A;0&#xFF0C;&#x7ED3;&#x679C;&#x6CA1;&#x6709;&#x5C0F;&#x6570;&#x70B9;&#x6216;&#x5C0F;&#x6570;&#x70B9;&#x90E8;&#x5206;&#x6574;&#x6570;2&#x53EF;&#x4EE5;&#x662F;&#x8D1F;&#x6570;&#xFF0C;&#x4F7F;&#x503C;&#x7684;&#x5C0F;&#x6570;&#x70B9;&#x5DE6;&#x8FB9;&#x7684;&#x6574;&#x6570;2&#x4F4D;&#x53D8;&#x4E3A;&#x8D1F;&#x6570;&#x96F6;&#x3002;&#x8FD9;&#x4E2A;&#x51FD;&#x6570;&#x4E5F;&#x53EA;&#x80FD;&#x4F20;&#x5165;&#x4E00;&#x4E2A;numeric1&#x53C2;&#x6570;&#xFF0C;&#x800C;&#x4E0D;&#x80FD;&#x5C06;Integer2&#x8BBE;&#x7F6E;&#x4E3A;&#x4F7F;&#x7528;&#x3002;&#x5982;&#x679C;&#x672A;&#x8BBE;&#x7F6E;&#x6574;&#x6570;2&#xFF0C;&#x51FD;&#x6570;&#x4F1A;&#x622A;&#x65AD;&#xFF0C;&#x5C31;&#x50CF;Integer2&#x662F;0&#x4E00;&#x6837;&#x3002;</p>
        <p>&#x4F8B;&#x5982;42.324.truncate(2)&#x4E3A;42.34&#x3002;&#x4EE5;&#x53CA;42.324.truncate()&#x4E3A;42.0&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7B97;&#x6570;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">+ NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">- NUMERIC</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x8D1F;NUMERIC&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 + NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x52A0;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 - NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x51CF;&#x53BB;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 * NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x4E58;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 / NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC1&#x9664;&#x4EE5;NUMERIC2&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1.power(NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x7684;numeric2&#x7684;&#x5E42;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.abs()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7EDD;&#x5BF9;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC1 % NUMERIC2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;numeric1&#x9664;&#x4EE5;numeric2&#x7684;&#x4F59;&#x6570;(&#x6A21;&#x6570;)&#x3002;&#x4EC5;&#x5F53;numeric1&#x4E3A;&#x8D1F;&#x6570;&#x65F6;&#xFF0C;&#x7ED3;&#x679C;&#x624D;&#x4E3A;&#x8D1F;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sqrt()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x6570;&#x5B57;&#x7684;&#x5E73;&#x65B9;&#x6839;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ln()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#xFF08;&#x57FA;&#x6570;e&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log10()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;10&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.log2()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x57FA;&#x6570;2&#x5BF9;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC1.log()</p>
        <p>NUMERIC1.log(NUMERIC2)</p>
      </td>
      <td style="text-align:left">
        <p>&#x5728;&#x4E0D;&#x5E26;&#x53C2;&#x6570;&#x7684;&#x60C5;&#x51B5;&#x4E0B;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x8FD4;&#x56DE;NUMERIC1&#x7684;&#x81EA;&#x7136;&#x5BF9;&#x6570;&#x3002;&#x4F7F;&#x7528;&#x53C2;&#x6570;&#x8C03;&#x7528;&#x65F6;&#xFF0C;&#x5C06;NUMERIC1&#x7684;&#x5BF9;&#x6570;&#x8FD4;&#x56DE;&#x5230;&#x57FA;&#x6570;NUMERIC2&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x76EE;&#x524D;&#xFF0C;NUMERIC1&#x5FC5;&#x987B;&#x5927;&#x4E8E;0&#x4E14;NUMERIC2&#x5FC5;&#x987B;&#x5927;&#x4E8E;1&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.exp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;e&#x7684;&#x6570;&#x503C;&#x6B21;&#x65B9;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.ceil()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5927;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5C0F;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.floor()</td>
      <td style="text-align:left">&#x5C06;&#x6570;&#x503C;&#x56DB;&#x820D;&#x4E94;&#x5165;&#xFF0C;&#x5E76;&#x8FD4;&#x56DE;&#x5C0F;&#x4E8E;&#x6216;&#x7B49;&#x4E8E;&#x6570;&#x503C;&#x7684;&#x6700;&#x5927;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sinh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x6B63;&#x5207;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.tanh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x6B63;&#x5207;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x662F;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cot()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x4F59;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.asin()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.acos()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x4F59;&#x5F26;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.atan()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CD;&#x6B63;&#x5207;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">atan2(NUMERIC1, NUMERIC2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5750;&#x6807;&#x7684;&#x53CD;&#x6B63;&#x5207;&#xFF08;NUMERIC1&#xFF0C;NUMERIC2&#xFF09;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.cosh()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x53CC;&#x66F2;&#x4F59;&#x5F26;&#x503C;&#x3002;</p>
        <p>&#x8FD4;&#x56DE;&#x503C;&#x7C7B;&#x578B;&#x4E3A;DOUBLE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.degrees()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x5F27;&#x5EA6;NUMERIC&#x7684;&#x5EA6;&#x6570;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.radians()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x5EA6;&#x7684;&#x5F27;&#x5EA6;&#x8868;&#x793A;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.sign()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;NUMERIC&#x7684;&#x7B26;&#x53F7;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.round(INT)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6574;&#x6570;&#x5C0F;&#x6570;&#x4F4D;&#x7684;&#x6570;&#x5B57;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">pi()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;pi&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">e()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6BD4;&#x4EFB;&#x4F55;&#x5176;&#x4ED6;&#x503C;&#x66F4;&#x63A5;&#x8FD1;e&#x7684;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4ECB;&#x4E8E;0.0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;1.0&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">rand(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x4F2A;&#x968F;&#x673A;&#x53CC;&#x7CBE;&#x5EA6;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x4ECB;&#x4E8E;0.0(&#x5305;&#x542B;)&#x548C;1.0(&#x4E0D;&#x5305;&#x542B;)&#x4E4B;&#x95F4;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x6574;&#x6570;&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;RAND&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#xFF0C;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">randInteger(INTEGER1, INTEGER2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;0&#xFF08;&#x5305;&#x62EC;&#xFF09;&#x548C;INTEGER2&#xFF08;&#x4E0D;&#x5305;&#x62EC;&#xFF09;&#x4E4B;&#x95F4;&#x7684;&#x4F2A;&#x968F;&#x673A;&#x6574;&#x6570;&#x503C;&#xFF0C;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x4E3A;INTEGER1&#x3002;&#x5982;&#x679C;&#x4E24;&#x4E2A;randInteger&#x51FD;&#x6570;&#x5177;&#x6709;&#x76F8;&#x540C;&#x7684;&#x521D;&#x59CB;&#x79CD;&#x5B50;&#x548C;&#x7ED1;&#x5B9A;&#xFF0C;&#x5219;&#x5B83;&#x4EEC;&#x5C06;&#x8FD4;&#x56DE;&#x76F8;&#x540C;&#x7684;&#x6570;&#x5B57;&#x5E8F;&#x5217;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">uuid()</td>
      <td style="text-align:left">&#x6839;&#x636E;RFC 4122&#x7C7B;&#x578B;4&#xFF08;&#x4F2A;&#x968F;&#x673A;&#x751F;&#x6210;&#x7684;&#xFF09;UUID&#x8FD4;&#x56DE;UUID&#xFF08;&#x901A;&#x7528;&#x552F;&#x4E00;&#x6807;&#x8BC6;&#x7B26;&#xFF09;&#x5B57;&#x7B26;&#x4E32;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;&#x201C;3d3c68f7-f608-473f-b60c-b0c44ad4cc4e&#x201D;&#xFF09;&#x3002;&#x4F7F;&#x7528;&#x52A0;&#x5BC6;&#x5F3A;&#x4F2A;&#x968F;&#x673A;&#x6570;&#x751F;&#x6210;&#x5668;&#x751F;&#x6210;UUID&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">INTEGER.bin()</td>
      <td style="text-align:left">
        <p>&#x4EE5;&#x4E8C;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x8FD4;&#x56DE;INTEGER&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x3002;&#x5982;&#x679C;INTEGER&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>4.bin()</code>&#x8FD4;&#x56DE;&#x201C;100&#x201D;&#x5E76;<code>12.bin()</code>&#x8FD4;&#x56DE;&#x201C;1100&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.hex()</p>
        <p>STRING.hex()zif</p>
      </td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x6574;&#x6570;NUMERIC&#x503C;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x8868;&#x793A;&#x5F62;&#x5F0F;&#x6216;&#x5341;&#x516D;&#x8FDB;&#x5236;&#x683C;&#x5F0F;&#x7684;STRING&#x3002;&#x5982;&#x679C;&#x53C2;&#x6570;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x6570;&#x5B57;20&#x5BFC;&#x81F4;&#x201C;14&#x201D;&#xFF0C;&#x6570;&#x5B57;100&#x5BFC;&#x81F4;&#x201C;64&#x201D;&#xFF0C;&#x5B57;&#x7B26;&#x4E32;&#x201C;hello&#xFF0C;world&#x201D;&#x5BFC;&#x81F4;&#x201C;68656C6C6F2C776F726C64&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 字符串函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5B57;&#x7B26;&#x4E32;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">string1 || string2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string1&#x548C;string2&#x7684;&#x4E32;&#x8054;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>CHAR_LENGTH(string)</p>
        <p>CHARACTER_LENGTH(string)</p>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string&#x4E2D;&#x7684;&#x5B57;&#x7B26;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">UPPER(string)</td>
      <td style="text-align:left">&#x4EE5;&#x5927;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x5B57;&#x7B26;&#x4E32;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LOWER(string)</td>
      <td style="text-align:left">&#x4EE5;&#x5C0F;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x5B57;&#x7B26;&#x4E32;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">POSITION(string1 IN string2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string1&#x5728;string2&#x7684;&#x7B2C;&#x4E00;&#x6B21;&#x51FA;&#x73B0;&#x7684;&#x4F4D;&#x7F6E;&#xFF08;&#x4ECE;1&#x5F00;&#x59CB;&#xFF09;;
        &#x5982;&#x679C;&#x5728;string2&#x4E2D;&#x627E;&#x4E0D;&#x5230;string1&#xFF0C;&#x5219;&#x8FD4;&#x56DE;0
        &#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TRIM([ BOTH | LEADING | TRAILING ] string1 FROM string2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;string2&#x4E2D;&#x5220;&#x9664;&#x5B57;&#x7B26;&#x4E32;string1&#x7684;&#x5F00;&#x5934;&#x548C;/&#x6216;&#x7ED3;&#x5C3E;&#x5B57;&#x7B26;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5220;&#x9664;&#x4E24;&#x8FB9;&#x7684;&#x7A7A;&#x767D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LTRIM(string)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;&#x53BB;&#x9664;&#x5DE6;&#x7A7A;&#x683C;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>LTRIM(&apos; This is a test String.&apos;)</code>&#x8FD4;&#x56DE;&#x201C;This
          is a test String&#x3002;&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">RTRIM(string)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;&#x53BB;&#x9664;&#x53F3;&#x65B9;&#x7684;&#x7A7A;&#x683C;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>RTRIM(&apos;This is a test String. &apos;)</code>&#x8FD4;&#x56DE;&#x201C;This
          is a test String&#x3002;&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">REPEAT(string, integer)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x91CD;&#x590D;&#x57FA;&#x672C;&#x5B57;&#x7B26;&#x4E32;
          &#x6574;&#x6570;&#x500D;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>REPEAT(&apos;This is a test String.&apos;, 2)</code>&#x8FD4;&#x56DE;&quot;This
          is a test String.This is a test String.&quot;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">REGEXP_REPLACE(string1, string2, string3)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x5176;&#x4E2D;&#x6240;&#x6709;&#x4E0E;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x5339;&#x914D;&#x7684;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x7EED;&#x88AB;string3&#x66FF;&#x6362;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>REGEXP_REPLACE(&apos;foobar&apos;, &apos;oo|ar&apos;, &apos;&apos;)</code>&#x8FD4;&#x56DE;&#x201C;fb&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">OVERLAY(string1 PLACING string2 FROM integer1 [ FOR integer2 ])</td>
      <td
      style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;integer1&#x4F4D;&#x7F6E;&#x7528;string2&#x66FF;&#x6362;string1&#x7684;integer2(&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x662F;string2&#x7684;&#x957F;&#x5EA6;)&#x5B57;&#x7B26;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF1A;<code>OVERLAY(&apos;This is an old string&apos; PLACING &apos; new&apos; FROM 10 FOR 5)</code> &#x8FD4;&#x56DE;
          &quot;This is a new string&quot;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">SUBSTRING(string FROM integer1 [ FOR integer2 ])</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4ECE;&#x4F4D;&#x7F6E;integer1&#x5F00;&#x59CB;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x5176;&#x957F;&#x5EA6;&#x4E3A;integer2&#xFF08;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x4E3A;&#x7ED3;&#x5C3E;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">REPLACE(string1, string2, string3)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x7684;string2</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;REPLACE(&quot;hello world&quot;&#xFF0C; &quot;world&quot;&#xFF0C;
          &quot;flink&quot;)&#x8FD4;&#x56DE;&quot;hello flink&quot;;REPLACE(&quot;ababab&quot;&#xFF0C;
          &quot;abab&quot;&#xFF0C; &quot;z&quot;)&#x8FD4;&#x56DE;&quot;zab&quot;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">REGEXP_EXTRACT(string1, string2[, integer])</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;string1&#x4E2D;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4F7F;&#x7528;&#x6307;&#x5B9A;&#x7684;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x548C;regex&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x6574;&#x6570;&#x63D0;&#x53D6;&#x3002;</p>
        <p><b>&#x6CE8;&#x610F;:</b>&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4ECE;1&#x5F00;&#x59CB;&#xFF0C;0&#x8868;&#x793A;&#x5339;&#x914D;&#x6574;&#x4E2A;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x6B64;&#x5916;&#xFF0C;regex&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4E0D;&#x5E94;&#x8D85;&#x8FC7;&#x5B9A;&#x4E49;&#x7684;&#x7EC4;&#x7684;&#x6570;&#x91CF;&#x3002;</p>
        <p>&#x4F8B;&#x5982;REGEXP_EXTRACT(&#x201C;foothebar&#x201D;&#x3001;&#x201C;foo(.
          * ?)(bar),2)&#x201C;&#x8FD4;&#x56DE;&#x201D;bar&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">INITCAP(string)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x79CD;&#x65B0;&#x5F62;&#x5F0F;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x6BCF;&#x4E2A;&#x5355;&#x8BCD;&#x7684;&#x7B2C;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5927;&#x5199;&#xFF0C;&#x5176;&#x4F59;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5C0F;&#x5199;&#x3002;&#x8FD9;&#x91CC;&#x7684;&#x5355;&#x8BCD;&#x8868;&#x793A;&#x4E00;&#x7CFB;&#x5217;&#x5B57;&#x6BCD;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">CONCAT(string1, string2,...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x8FDE;&#x63A5;string1&#xFF0C;string2&#xFF0C;...&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;&#x5982;&#x679C;&#x4EFB;&#x4F55;&#x53C2;&#x6570;&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>CONCAT(&apos;AA&apos;, &apos;BB&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AABBCC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">CONCAT_WS(string1, string2, string3,...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string2,
          string3&#xFF0C;&#x2026;&#x4F7F;&#x7528;&#x5206;&#x9694;&#x7B26;string1&#x3002;&#x5206;&#x9694;&#x7B26;&#x6DFB;&#x52A0;&#x5728;&#x8981;&#x8FDE;&#x63A5;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x4E4B;&#x95F4;&#x3002;&#x5982;&#x679C;string1&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;&#x4E0E;CONCAT()&#x76F8;&#x6BD4;&#xFF0C;CONCAT_WS()&#x81EA;&#x52A8;&#x8DF3;&#x8FC7;&#x7A7A;&#x53C2;&#x6570;&#x3002;</p>
        <p>&#x4F8B;&#x5982;,CONCAT_WS(&#x201C;~&#x201D;,&#x201C;AA&#x201D;,NULL,&#x201C;BB&#x201D;,&#x201D;,&#x201C;CC&#x201D;)&#x8FD4;&#x56DE;&#x201C;AA
          ~ BB ~ ~ CC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">LPAD(string1, integer, string2)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x5DE6;&#x586B;&#x5145;string2&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x957F;&#x5EA6;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF1A;<code>LPAD(&apos;hi&apos;,4,&apos;??&apos;)</code> &#x8FD4;&#x56DE;
          &quot;??hi&quot;; <code>LPAD(&apos;hi&apos;,1,&apos;??&apos;)</code> &#x8FD4;&#x56DE;
          &quot;h&quot;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">RPAD(string1, integer, string2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;string1&#x53F3;&#x586B;&#x5145;string2&#x5230;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x957F;&#x5EA6;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF1A;<code>RPAD(&apos;hi&apos;,4,&apos;??&apos;)</code> &#x8FD4;&#x56DE;
          &quot;hi??&quot;, <code>RPAD(&apos;hi&apos;,1,&apos;??&apos;)</code> &#x8FD4;&#x56DE;
          &quot;h&quot;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">FROM_BASE64(string)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string&#x8FD4;&#x56DE;base64&#x89E3;&#x7801;&#x7684;&#x7ED3;&#x679C;;
          &#x5982;&#x679C;string&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>FROM_BASE64(&apos;aGVsbG8gd29ybGQ=&apos;)</code>&#x8FD4;&#x56DE;&#x201C;hello
          world&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TO_BASE64(string)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string&#x8FD4;&#x56DE;base64&#x7F16;&#x7801;&#x7684;&#x7ED3;&#x679C;;
          &#x5982;&#x679C;string&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>TO_BASE64(&apos;hello world&apos;)</code>&#x8FD4;&#x56DE;&#x201C;aGVsbG8gd29ybGQ
          =&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5B57;&#x7B26;&#x4E32;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING1 <b>+</b> STRING2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING1&#x548C;STRING2&#x7684;&#x4E32;&#x8054;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.charLength()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x4E2D;&#x7684;&#x5B57;&#x7B26;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.upperCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5927;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.lowerCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5C0F;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.position(STRING2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string2&#x4E2D;&#x7B2C;&#x4E00;&#x6B21;&#x51FA;&#x73B0;string1&#x7684;&#x4F4D;&#x7F6E;(&#x4ECE;1&#x5F00;&#x59CB;);&#x5982;&#x679C;&#x5728;string2&#x4E2D;&#x627E;&#x4E0D;&#x5230;string1&#xFF0C;&#x5219;&#x8FD4;&#x56DE;0&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>STRING1<b>.</b>trim(LEADING, STRING2)</p>
        <p>STRING1<b>.</b>trim(TRAILING, STRING2)</p>
        <p>STRING1<b>.</b>trim(BOTH, STRING2)</p>
        <p>STRING1<b>.</b>trim(BOTH)</p>
        <p>STRING1<b>.</b>trim()</p>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;string2&#x4E2D;&#x5220;&#x9664;&#x5B57;&#x7B26;&#x4E32;string1&#x7684;&#x5F00;&#x5934;&#x548C;/&#x6216;&#x7ED3;&#x5C3E;&#x5B57;&#x7B26;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5220;&#x9664;&#x4E24;&#x8FB9;&#x7684;&#x7A7A;&#x767D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.ltrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x5DE6;&#x6D4B;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos; This is a test String.&apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.rtrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x53F3;&#x4FA7;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos;This is a test String. &apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.repeat(INT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x91CD;&#x590D;&#x57FA;&#x672C;STRING
          INT&#x6B21;&#x6570;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;This is a test String.&apos;.repeat(2)</code>&#x8FD4;&#x56DE;&quot;This
          is a test String.This is a test String.&quot;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpReplace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x5176;&#x4E2D;&#x4E0E;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x5339;&#x914D;&#x7684;&#x6240;&#x6709;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#x5C06;&#x88AB;string3&#x8FDE;&#x7EED;&#x66FF;&#x6362;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foobar&apos;.regexpReplace(&apos;oo|ar&apos;, &apos;&apos;)</code>&#x8FD4;&#x56DE;&#x201C;fb&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.overlay(STRING2, INT1)
        <br />STRING1.overlay(STRING2, INT1, INT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;integer1&#x4F4D;&#x7F6E;&#x7528;string2&#x66FF;&#x6362;string1&#x7684;integer2(&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x662F;string2&#x7684;&#x957F;&#x5EA6;)&#x5B57;&#x7B26;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxx&#x201D;; <code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6, 2)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxxst&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.substring(INT1)
        <br />STRING.substring(INT1, INT2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x7684;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;&#x4F4D;&#x7F6E;INT1&#x5F00;&#x59CB;&#xFF0C;&#x957F;&#x5EA6;&#x4E3A;INT2&#xFF08;&#x9ED8;&#x8BA4;&#x4E3A;&#x7ED3;&#x675F;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.replace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x7684;string2</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.replace(&apos;world&apos;, &apos;flink&apos;)</code>&#x8FD4;&#x56DE;&apos;hello
          flink&apos;; <code>&apos;ababab&apos;.replace(&apos;abab&apos;, &apos;z&apos;)</code>&#x8FD4;&#x56DE;&apos;zab&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpExtract(STRING2[, INTEGER1])</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;string2&#x7684;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x503C;&#x3002;</p>
        <p>&#x6CE8;&#x610F;:&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4ECE;1&#x5F00;&#x59CB;&#xFF0C;0&#x8868;&#x793A;&#x5339;&#x914D;&#x6574;&#x4E2A;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x6B64;&#x5916;&#xFF0C;regex&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4E0D;&#x5E94;&#x8D85;&#x8FC7;&#x5B9A;&#x4E49;&#x7684;&#x7EC4;&#x7684;&#x6570;&#x91CF;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foothebar&apos;.regexpExtract(&apos;foo(.*?)(bar)&apos;, 2)&quot;</code>&#x8FD4;&#x56DE;&#x201C;bar&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.initCap()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x79CD;&#x65B0;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x5F62;&#x5F0F;&#xFF0C;&#x5176;&#x4E2D;&#x6BCF;&#x4E2A;&#x5355;&#x8BCD;&#x7684;&#x7B2C;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5927;&#x5199;&#xFF0C;&#x5176;&#x4F59;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5C0F;&#x5199;&#x3002;&#x8FD9;&#x91CC;&#x7684;&#x5355;&#x8BCD;&#x662F;&#x6307;&#x4E00;&#x7CFB;&#x5217;&#x5B57;&#x6BCD;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">concat(STRING1, STRING2, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string1,
          string2&#xFF0C;&#x2026;&#x5982;&#x679C;&#x4EFB;&#x4F55;&#x53C2;&#x6570;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat(&apos;AA&apos;, &apos;BB&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AABBCC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">concat_ws(STRING1, STRING2, STRING3, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string2,
          string3&#xFF0C;&#x2026;&#x4F7F;&#x7528;&#x5206;&#x9694;&#x7B26;string1&#x3002;&#x5206;&#x9694;&#x7B26;&#x6DFB;&#x52A0;&#x5728;&#x8981;&#x8FDE;&#x63A5;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x4E4B;&#x95F4;&#x3002;&#x5982;&#x679C;string1&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;&#x4E0E;CONCAT()&#x76F8;&#x6BD4;&#xFF0C;CONCAT_WS()&#x81EA;&#x52A8;&#x8DF3;&#x8FC7;&#x7A7A;&#x53C2;&#x6570;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat_ws(&apos;~&apos;, &apos;AA&apos;, Null(STRING), &apos;BB&apos;, &apos;&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AA~BB
          ~~ CC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.lpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x5DE6;&#x586B;&#x5145;string2&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x957F;&#x5EA6;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.lpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;??
          hi&#x201D;; <code>&apos;hi&apos;.lpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.rpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;string1&#x53F3;&#x586B;&#x5145;string2&#x5230;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x957F;&#x5EA6;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.rpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;hi
          ??&#x201D;; <code>&apos;hi&apos;.rpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.fromBase64()</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x4E2D;&#x8FD4;&#x56DE;base64&#x89E3;&#x7801;&#x7684;&#x7ED3;&#x679C;;&#x5982;&#x679C;&#x5B57;&#x7B26;&#x4E32;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;aGVsbG8gd29ybGQ=&apos;.fromBase64()</code>&#x8FD4;&#x56DE;&#x201C;hello
          world&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toBase64()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;STRING&#x7684;base64&#x7F16;&#x7801;&#x7ED3;&#x679C;;
          &#x5982;&#x679C;STRING&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.toBase64()</code>&#x8FD4;&#x56DE;&#x201C;aGVsbG8gd29ybGQ
          =&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5B57;&#x7B26;&#x4E32;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING1 <b>+</b> STRING2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING1&#x548C;STRING2&#x7684;&#x4E32;&#x8054;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.charLength()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x4E2D;&#x7684;&#x5B57;&#x7B26;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.upperCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5927;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.lowerCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5C0F;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.position(STRING2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string2&#x4E2D;&#x7B2C;&#x4E00;&#x6B21;&#x51FA;&#x73B0;string1&#x7684;&#x4F4D;&#x7F6E;(&#x4ECE;1&#x5F00;&#x59CB;);&#x5982;&#x679C;&#x5728;string2&#x4E2D;&#x627E;&#x4E0D;&#x5230;string1&#xFF0C;&#x5219;&#x8FD4;&#x56DE;0&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>STRING1<b>.</b>trim(LEADING, STRING2)</p>
        <p>STRING1<b>.</b>trim(TRAILING, STRING2)</p>
        <p>STRING1<b>.</b>trim(BOTH, STRING2)</p>
        <p>STRING1<b>.</b>trim(BOTH)</p>
        <p>STRING1<b>.</b>trim()</p>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;string2&#x4E2D;&#x5220;&#x9664;&#x5B57;&#x7B26;&#x4E32;string1&#x7684;&#x5F00;&#x5934;&#x548C;/&#x6216;&#x7ED3;&#x5C3E;&#x5B57;&#x7B26;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5220;&#x9664;&#x4E24;&#x8FB9;&#x7684;&#x7A7A;&#x767D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.ltrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x5DE6;&#x6D4B;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos; This is a test String.&apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.rtrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x53F3;&#x4FA7;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos;This is a test String. &apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.repeat(INT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x91CD;&#x590D;&#x57FA;&#x672C;STRING
          INT&#x6B21;&#x6570;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;This is a test String.&apos;.repeat(2)</code>&#x8FD4;&#x56DE;&quot;This
          is a test String.This is a test String.&quot;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpReplace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x5176;&#x4E2D;&#x4E0E;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x5339;&#x914D;&#x7684;&#x6240;&#x6709;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#x5C06;&#x88AB;string3&#x8FDE;&#x7EED;&#x66FF;&#x6362;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foobar&apos;.regexpReplace(&apos;oo|ar&apos;, &apos;&apos;)</code>&#x8FD4;&#x56DE;&#x201C;fb&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.overlay(STRING2, INT1)
        <br />STRING1.overlay(STRING2, INT1, INT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;integer1&#x4F4D;&#x7F6E;&#x7528;string2&#x66FF;&#x6362;string1&#x7684;integer2(&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x662F;string2&#x7684;&#x957F;&#x5EA6;)&#x5B57;&#x7B26;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxx&#x201D;; <code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6, 2)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxxst&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.substring(INT1)
        <br />STRING.substring(INT1, INT2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x7684;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;&#x4F4D;&#x7F6E;INT1&#x5F00;&#x59CB;&#xFF0C;&#x957F;&#x5EA6;&#x4E3A;INT2&#xFF08;&#x9ED8;&#x8BA4;&#x4E3A;&#x7ED3;&#x675F;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.replace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x7684;string2</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.replace(&apos;world&apos;, &apos;flink&apos;)</code>&#x8FD4;&#x56DE;&apos;hello
          flink&apos;; <code>&apos;ababab&apos;.replace(&apos;abab&apos;, &apos;z&apos;)</code>&#x8FD4;&#x56DE;&apos;zab&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpExtract(STRING2[, INTEGER1])</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;string2&#x7684;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x503C;&#x3002;</p>
        <p>&#x6CE8;&#x610F;:&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4ECE;1&#x5F00;&#x59CB;&#xFF0C;0&#x8868;&#x793A;&#x5339;&#x914D;&#x6574;&#x4E2A;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x6B64;&#x5916;&#xFF0C;regex&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4E0D;&#x5E94;&#x8D85;&#x8FC7;&#x5B9A;&#x4E49;&#x7684;&#x7EC4;&#x7684;&#x6570;&#x91CF;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foothebar&apos;.regexpExtract(&apos;foo(.*?)(bar)&apos;, 2)&quot;</code>&#x8FD4;&#x56DE;&#x201C;bar&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.initCap()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x79CD;&#x65B0;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x5F62;&#x5F0F;&#xFF0C;&#x5176;&#x4E2D;&#x6BCF;&#x4E2A;&#x5355;&#x8BCD;&#x7684;&#x7B2C;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5927;&#x5199;&#xFF0C;&#x5176;&#x4F59;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5C0F;&#x5199;&#x3002;&#x8FD9;&#x91CC;&#x7684;&#x5355;&#x8BCD;&#x662F;&#x6307;&#x4E00;&#x7CFB;&#x5217;&#x5B57;&#x6BCD;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">concat(STRING1, STRING2, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string1,
          string2&#xFF0C;&#x2026;&#x5982;&#x679C;&#x4EFB;&#x4F55;&#x53C2;&#x6570;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat(&apos;AA&apos;, &apos;BB&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AABBCC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">concat_ws(STRING1, STRING2, STRING3, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string2,
          string3&#xFF0C;&#x2026;&#x4F7F;&#x7528;&#x5206;&#x9694;&#x7B26;string1&#x3002;&#x5206;&#x9694;&#x7B26;&#x6DFB;&#x52A0;&#x5728;&#x8981;&#x8FDE;&#x63A5;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x4E4B;&#x95F4;&#x3002;&#x5982;&#x679C;string1&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;&#x4E0E;CONCAT()&#x76F8;&#x6BD4;&#xFF0C;CONCAT_WS()&#x81EA;&#x52A8;&#x8DF3;&#x8FC7;&#x7A7A;&#x53C2;&#x6570;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat_ws(&apos;~&apos;, &apos;AA&apos;, Null(STRING), &apos;BB&apos;, &apos;&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AA~BB
          ~~ CC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.lpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x5DE6;&#x586B;&#x5145;string2&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x957F;&#x5EA6;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.lpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;??
          hi&#x201D;; <code>&apos;hi&apos;.lpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.rpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;string1&#x53F3;&#x586B;&#x5145;string2&#x5230;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x957F;&#x5EA6;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.rpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;hi
          ??&#x201D;; <code>&apos;hi&apos;.rpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.fromBase64()</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x4E2D;&#x8FD4;&#x56DE;base64&#x89E3;&#x7801;&#x7684;&#x7ED3;&#x679C;;&#x5982;&#x679C;&#x5B57;&#x7B26;&#x4E32;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;aGVsbG8gd29ybGQ=&apos;.fromBase64()</code>&#x8FD4;&#x56DE;&#x201C;hello
          world&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toBase64()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;STRING&#x7684;base64&#x7F16;&#x7801;&#x7ED3;&#x679C;;
          &#x5982;&#x679C;STRING&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.toBase64()</code>&#x8FD4;&#x56DE;&#x201C;aGVsbG8gd29ybGQ
          =&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x5B57;&#x7B26;&#x4E32;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING1 <b>+</b> STRING2</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING1&#x548C;STRING2&#x7684;&#x4E32;&#x8054;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.charLength()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x4E2D;&#x7684;&#x5B57;&#x7B26;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.upperCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5927;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.lowerCase()</td>
      <td style="text-align:left">&#x4EE5;&#x5C0F;&#x5199;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;STRING&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.position(STRING2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;string2&#x4E2D;&#x7B2C;&#x4E00;&#x6B21;&#x51FA;&#x73B0;string1&#x7684;&#x4F4D;&#x7F6E;(&#x4ECE;1&#x5F00;&#x59CB;);&#x5982;&#x679C;&#x5728;string2&#x4E2D;&#x627E;&#x4E0D;&#x5230;string1&#xFF0C;&#x5219;&#x8FD4;&#x56DE;0&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><b>STRING.</b>trim(</p>
        <p>leading <b>=</b>  <b>true</b>,</p>
        <p>trailing <b>=</b>  <b>true</b>,</p>
        <p>character <b>=</b> &quot; &quot;)</p>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;string2&#x4E2D;&#x5220;&#x9664;&#x5B57;&#x7B26;&#x4E32;string1&#x7684;&#x5F00;&#x5934;&#x548C;/&#x6216;&#x7ED3;&#x5C3E;&#x5B57;&#x7B26;&#x3002;&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5220;&#x9664;&#x4E24;&#x8FB9;&#x7684;&#x7A7A;&#x767D;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.ltrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x5DE6;&#x6D4B;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos; This is a test String.&apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.rtrim()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x5220;&#x9664;&#x53F3;&#x4FA7;&#x7A7A;&#x767D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;LTRIM(&apos;This is a test String. &apos;)&#x8FD4;&#x56DE;&apos;
          This is a test String.&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.repeat(INT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x91CD;&#x590D;&#x57FA;&#x672C;STRING
          INT&#x6B21;&#x6570;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;This is a test String.&apos;.repeat(2)</code>&#x8FD4;&#x56DE;&quot;This
          is a test String.This is a test String.&quot;.</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpReplace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x5176;&#x4E2D;&#x4E0E;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;string2&#x5339;&#x914D;&#x7684;&#x6240;&#x6709;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#x5C06;&#x88AB;string3&#x8FDE;&#x7EED;&#x66FF;&#x6362;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foobar&apos;.regexpReplace(&apos;oo|ar&apos;, &apos;&apos;)</code>&#x8FD4;&#x56DE;&#x201C;fb&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.overlay(STRING2, INT1)
        <br />STRING1.overlay(STRING2, INT1, INT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x4ECE;integer1&#x4F4D;&#x7F6E;&#x7528;string2&#x66FF;&#x6362;string1&#x7684;integer2(&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x662F;string2&#x7684;&#x957F;&#x5EA6;)&#x5B57;&#x7B26;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxx&#x201D;; <code>&apos;xxxxxtest&apos;.overlay(&apos;xxxx&apos;, 6, 2)</code>&#x8FD4;&#x56DE;&#x201C;xxxxxxxxxst&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.substring(INT1)
        <br />STRING.substring(INT1, INT2)</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;STRING&#x7684;&#x5B50;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;&#x4F4D;&#x7F6E;INT1&#x5F00;&#x59CB;&#xFF0C;&#x957F;&#x5EA6;&#x4E3A;INT2&#xFF08;&#x9ED8;&#x8BA4;&#x4E3A;&#x7ED3;&#x675F;&#xFF09;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.replace(STRING2, STRING3)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x7684;string2</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.replace(&apos;world&apos;, &apos;flink&apos;)</code>&#x8FD4;&#x56DE;&apos;hello
          flink&apos;; <code>&apos;ababab&apos;.replace(&apos;abab&apos;, &apos;z&apos;)</code>&#x8FD4;&#x56DE;&apos;zab&apos;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.regexpExtract(STRING2[, INTEGER1])</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7528;string1&#x4E2D;&#x7684;string3(&#x975E;&#x91CD;&#x53E0;)&#x66FF;&#x6362;string2&#x7684;&#x6240;&#x6709;&#x51FA;&#x73B0;&#x503C;&#x3002;</p>
        <p>&#x6CE8;&#x610F;:&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4ECE;1&#x5F00;&#x59CB;&#xFF0C;0&#x8868;&#x793A;&#x5339;&#x914D;&#x6574;&#x4E2A;&#x6B63;&#x5219;&#x8868;&#x8FBE;&#x5F0F;&#x3002;&#x6B64;&#x5916;&#xFF0C;regex&#x5339;&#x914D;&#x7EC4;&#x7D22;&#x5F15;&#x4E0D;&#x5E94;&#x8D85;&#x8FC7;&#x5B9A;&#x4E49;&#x7684;&#x7EC4;&#x7684;&#x6570;&#x91CF;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;foothebar&apos;.regexpExtract(&apos;foo(.*?)(bar)&apos;, 2)&quot;</code>&#x8FD4;&#x56DE;&#x201C;bar&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.initCap()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4E00;&#x79CD;&#x65B0;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x5F62;&#x5F0F;&#xFF0C;&#x5176;&#x4E2D;&#x6BCF;&#x4E2A;&#x5355;&#x8BCD;&#x7684;&#x7B2C;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5927;&#x5199;&#xFF0C;&#x5176;&#x4F59;&#x5B57;&#x7B26;&#x8F6C;&#x6362;&#x4E3A;&#x5C0F;&#x5199;&#x3002;&#x8FD9;&#x91CC;&#x7684;&#x5355;&#x8BCD;&#x662F;&#x6307;&#x4E00;&#x7CFB;&#x5217;&#x5B57;&#x6BCD;&#x6570;&#x5B57;&#x5B57;&#x7B26;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">concat(STRING1, STRING2, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string1,
          string2&#xFF0C;&#x2026;&#x5982;&#x679C;&#x4EFB;&#x4F55;&#x53C2;&#x6570;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat(&apos;AA&apos;, &apos;BB&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AABBCC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">concat_ws(STRING1, STRING2, STRING3, ...)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x8FDE;&#x63A5;string2,
          string3&#xFF0C;&#x2026;&#x4F7F;&#x7528;&#x5206;&#x9694;&#x7B26;string1&#x3002;&#x5206;&#x9694;&#x7B26;&#x6DFB;&#x52A0;&#x5728;&#x8981;&#x8FDE;&#x63A5;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x4E4B;&#x95F4;&#x3002;&#x5982;&#x679C;string1&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;&#x4E0E;CONCAT()&#x76F8;&#x6BD4;&#xFF0C;CONCAT_WS()&#x81EA;&#x52A8;&#x8DF3;&#x8FC7;&#x7A7A;&#x53C2;&#x6570;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>concat_ws(&apos;~&apos;, &apos;AA&apos;, Null(STRING), &apos;BB&apos;, &apos;&apos;, &apos;CC&apos;)</code>&#x8FD4;&#x56DE;&#x201C;AA~BB
          ~~ CC&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.lpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x4ECE;string1&#x5DE6;&#x586B;&#x5145;string2&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x8BE5;&#x5B57;&#x7B26;&#x4E32;&#x7684;&#x957F;&#x5EA6;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.lpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;??
          hi&#x201D;; <code>&apos;hi&apos;.lpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING1.rpad(INT, STRING2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x5B57;&#x7B26;&#x4E32;&#xFF0C;&#x4ECE;string1&#x53F3;&#x586B;&#x5145;string2&#x5230;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x957F;&#x5EA6;&#x3002;&#x5982;&#x679C;string1&#x7684;&#x957F;&#x5EA6;&#x5C0F;&#x4E8E;integer&#xFF0C;&#x5219;&#x8FD4;&#x56DE;&#x5C06;string1&#x7F29;&#x77ED;&#x4E3A;&#x6574;&#x6570;&#x5B57;&#x7B26;&#x7684;&#x5B57;&#x7B26;&#x4E32;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hi&apos;.rpad(4, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;hi
          ??&#x201D;; <code>&apos;hi&apos;.rpad(1, &apos;??&apos;)</code>&#x8FD4;&#x56DE;&#x201C;h&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.fromBase64()</td>
      <td style="text-align:left">
        <p>&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x4E2D;&#x8FD4;&#x56DE;base64&#x89E3;&#x7801;&#x7684;&#x7ED3;&#x679C;;&#x5982;&#x679C;&#x5B57;&#x7B26;&#x4E32;&#x4E3A;&#x7A7A;&#xFF0C;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;aGVsbG8gd29ybGQ=&apos;.fromBase64()</code>&#x8FD4;&#x56DE;&#x201C;hello
          world&#x201D;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toBase64()</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;STRING&#x7684;base64&#x7F16;&#x7801;&#x7ED3;&#x679C;;
          &#x5982;&#x679C;STRING&#x4E3A;NULL&#xFF0C;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;hello world&apos;.toBase64()</code>&#x8FD4;&#x56DE;&#x201C;aGVsbG8gd29ybGQ
          =&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 时间函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x65F6;&#x95F4;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">DATE string</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;yyyy-MM-dd&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TIME string</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;HH:mm:ss&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TIMESTAMP string</td>
      <td style="text-align:left">&#x4EE5;&#x201C;yyyy-MM-dd HH:mm:ss[. sss]&#x201D;&#x7684;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">INTERVAL string range</td>
      <td style="text-align:left">
        <p>&#x89E3;&#x6790;&#x5F62;&#x5F0F;&#x4E3A;&#x201C;dd hh:mm:ss&#x201D;&#x7684;&#x533A;&#x95F4;&#x5B57;&#x7B26;&#x4E32;&#x3002;fff&#x201D;&#x8868;&#x793A;&#x6BEB;&#x79D2;&#x7684;SQL&#x95F4;&#x9694;&#xFF0C;&#x6216;&#x201C;yyyy-mm&#x201D;&#x8868;&#x793A;&#x6708;&#x4EFD;&#x7684;SQL&#x95F4;&#x9694;&#x3002;&#x4E00;&#x4E2A;&#x95F4;&#x9694;&#x8303;&#x56F4;&#x53EF;&#x4EE5;&#x662F;&#x5929;&#x3001;&#x5206;&#x949F;&#x3001;&#x5929;&#x5230;&#x5C0F;&#x65F6;&#xFF0C;&#x6216;&#x8005;&#x4EE5;&#x6BEB;&#x79D2;&#x4E3A;&#x95F4;&#x9694;&#x7684;&#x5929;&#x5230;&#x79D2;;&#x5E74;&#x6216;&#x5E74;&#x5230;&#x6708;&#xFF0C;&#x4EE5;&#x6708;&#x4E3A;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;<code>INTERVAL &apos;10 00:00:00.004&apos; DAY TO SECOND</code>&#xFF0C;<code>INTERVAL &apos;10&apos; DAY</code>&#x6216;&#x8005;<code>INTERVAL &apos;2-10&apos; YEAR TO MONTH</code>&#x8FD4;&#x56DE;&#x7684;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">CURRENT_DATE</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">CURRENT_TIME</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">CURRENT_TIMESTAMP</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LOCALTIME</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">LOCALTIMESTAMP</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">EXTRACT(timeintervalunit FROM temporal)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4ECE;temporal&#x7684;timeintervalitit&#x90E8;&#x5206;&#x63D0;&#x53D6;&#x7684;long&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>EXTRACT(DAY FROM DATE &apos;2006-06-05&apos;)</code>&#x8FD4;&#x56DE;5&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">YEAR(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL date &#x65E5;&#x671F;&#x8FD4;&#x56DE;&#x5E74;&#x4EFD;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(YEAR FROM date).</code>
        </p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>YEAR(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;1994&#x5E74;&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">QUARTER(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL date&#x65E5;&#x671F;&#x8FD4;&#x56DE;&#x4E00;&#x5E74;&#x4E2D;&#x7684;&#x7B2C;4&#x4E2A;&#x5B63;&#x5EA6;(1&#x5230;4&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;)&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(QUARTER FROM date)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>QUARTER(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;3&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">MONTH(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL&#x65E5;&#x671F;&#x65E5;&#x671F;&#x8FD4;&#x56DE;&#x4E00;&#x5E74;&#x4E2D;&#x7684;&#x6708;&#x4EFD;&#xFF08;1&#x5230;12&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(MONTH FROM date)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>MONTH(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;9&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">WEEK(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL date date&#x8FD4;&#x56DE;&#x4E00;&#x5E74;&#x4E2D;&#x7684;&#x4E00;&#x5468;&#xFF08;1&#x5230;53&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(WEEK FROM date)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>WEEK(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;39&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DAYOFYEAR(date)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x5E74;&#x7684;&#x4ECE;SQL&#x65E5;&#x671F;&#x5F53;&#x5929;&#xFF08;1&#x548C;366&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x65E5;&#x671F;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(DOY FROM date)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>DAYOFYEAR(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;270&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DAYOFMONTH(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL&#x65E5;&#x671F;&#x65E5;&#x671F;&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x6708;&#x4E2D;&#x7684;&#x67D0;&#x4E00;&#x5929;&#xFF08;1&#x5230;31&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(DAY FROM date)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>DAYOFMONTH(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;27&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DAYOFWEEK(date)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL date date&#x8FD4;&#x56DE;&#x4E00;&#x5468;&#x4E2D;&#x7684;&#x661F;&#x671F;&#x51E0;&#xFF08;1&#x5230;7&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;;&#x661F;&#x671F;&#x65E5;=
          1&#xFF09;<code>EXTRACT(DOW FROM date)</code>&#x3002;&#x7B49;&#x6548;&#x4E8E;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>DAYOFWEEK(DATE &apos;1994-09-27&apos;)</code>&#x8FD4;&#x56DE;3&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">HOUR(timestamp)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL&#x65F6;&#x95F4;&#x6233;&#x65F6;&#x95F4;&#x6233;&#x8FD4;&#x56DE;&#x4E00;&#x5929;&#x4E2D;&#x7684;&#x5C0F;&#x65F6;&#xFF08;0&#x5230;23&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(HOUR FROM timestamp)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>HOUR(TIMESTAMP &apos;1994-09-27 13:14:15&apos;)</code>&#x8FD4;&#x56DE;13&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">MINUTE(timestamp)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL&#x65F6;&#x95F4;&#x6233;&#x8BB0;&#x65F6;&#x95F4;&#x6233;&#x8FD4;&#x56DE;&#x4E00;&#x5C0F;&#x65F6;&#x7684;&#x5206;&#x949F;&#xFF08;0&#x5230;59&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(MINUTE FROM timestamp)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>MINUTE(TIMESTAMP &apos;1994-09-27 13:14:15&apos;)</code>&#x8FD4;&#x56DE;14&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">SECOND(timestamp)</td>
      <td style="text-align:left">
        <p>&#x4ECE;SQL&#x65F6;&#x95F4;&#x6233;&#x8FD4;&#x56DE;&#x7B2C;&#x4E8C;&#x5206;&#x949F;&#xFF08;0&#x5230;59&#x4E4B;&#x95F4;&#x7684;&#x6574;&#x6570;&#xFF09;&#x3002;&#x76F8;&#x5F53;&#x4E8E;<code>EXTRACT(SECOND FROM timestamp)</code>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>SECOND(TIMESTAMP &apos;1994-09-27 13:14:15&apos;)</code>&#x8FD4;&#x56DE;15&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">FLOOR(timepoint TO timeintervalunit)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;&#x201C;timepoint&#x201D;&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;&#x201C;timeintervalunit&#x201D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>FLOOR(TIME &apos;12:44:31&apos; TO MINUTE)</code>&#x8FD4;&#x56DE;12:44:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">CEIL(timepoint TO timeintervalunit)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;&#x201C;timepoint&#x201D;&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;&#x201C;timeintervalunit&#x201D;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>CEIL(TIME &apos;12:44:31&apos; TO MINUTE)</code>&#x8FD4;&#x56DE;12:45:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">(timepoint1, temporal1) OVERLAPS (timepoint2, temporal2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;(timepoint1, temporal1)&#x548C;(timepoint2, temporal2)&#x5B9A;&#x4E49;&#x7684;&#x4E24;&#x4E2A;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x91CD;&#x53E0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE&#x3002;&#x65F6;&#x95F4;&#x503C;&#x53EF;&#x4EE5;&#x662F;&#x4E00;&#x4E2A;&#x65F6;&#x95F4;&#x70B9;&#xFF0C;&#x4E5F;&#x53EF;&#x4EE5;&#x662F;&#x4E00;&#x4E2A;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>(TIME &apos;2:55:00&apos;, INTERVAL &apos;1&apos; HOUR) OVERLAPS (TIME &apos;3:30:00&apos;, INTERVAL &apos;2&apos; HOUR)</code>&#x8FD4;&#x56DE;TRUE; <code>(TIME &apos;9:00:00&apos;, TIME &apos;10:00:00&apos;) OVERLAPS (TIME &apos;10:15:00&apos;, INTERVAL &apos;3&apos; HOUR)</code>&#x8FD4;&#x56DE;FALSE&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">DATE_FORMAT(timestamp, string)</td>
      <td style="text-align:left"><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x6B64;&#x529F;&#x80FD;&#x6709;&#x4E25;&#x91CD;&#x9519;&#x8BEF;&#xFF0C;&#x73B0;&#x5728;&#x4E0D;&#x5E94;&#x8BE5;&#x4F7F;&#x7528;&#x3002;&#x8BF7;&#x6539;&#x4E3A;&#x5B9E;&#x73B0;&#x81EA;&#x5B9A;&#x4E49;UDF&#x6216;&#x4F7F;&#x7528;EXTRACT&#x4F5C;&#x4E3A;&#x89E3;&#x51B3;&#x65B9;&#x6CD5;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TIMESTAMPADD(timeintervalunit, interval, timepoint)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x65F6;&#x95F4;&#x503C;&#xFF0C;&#x8BE5;&#x65F6;&#x95F4;&#x503C;&#x5C06;&#x4E00;&#x4E2A;(&#x5E26;&#x7B26;&#x53F7;&#x7684;)&#x6574;&#x6570;&#x533A;&#x95F4;&#x6DFB;&#x52A0;&#x5230;&#x65F6;&#x95F4;&#x70B9;&#x3002;interval&#x7684;&#x5355;&#x4F4D;&#x7531;unit&#x53C2;&#x6570;&#x7ED9;&#x51FA;&#xFF0C;&#x5B83;&#x5E94;&#x8BE5;&#x662F;&#x4E0B;&#x5217;&#x503C;&#x4E4B;&#x4E00;:SECOND&#x3001;MINUTE&#x3001;HOUR&#x3001;DAY&#x3001;WEEK&#x3001;MONTH&#x3001;QUARTER&#x6216;YEAR&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>TIMESTAMPADD(WEEK, 1, DATE &apos;2003-01-02&apos;)</code>&#x8FD4;&#x56DE;<code>2003-01-09</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMESTAMPDIFF(timepointunit, timepoint1, timepoint2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;timepoint1&#x548C;timepoint2&#x4E4B;&#x95F4;&#x7684;timepointunit(&#x5E26;&#x7B26;&#x53F7;)&#x6570;&#x91CF;&#x3002;&#x95F4;&#x9694;&#x7684;&#x5355;&#x4F4D;&#x7531;&#x7B2C;&#x4E00;&#x4E2A;&#x53C2;&#x6570;&#x7ED9;&#x51FA;&#xFF0C;&#x8BE5;&#x53C2;&#x6570;&#x5E94;&#x8BE5;&#x662F;&#x4E0B;&#x5217;&#x503C;&#x4E4B;&#x4E00;:SECOND&#x3001;MINUTE&#x3001;HOUR&#x3001;DAY&#x3001;WEEK&#x3001;MONTH&#x3001;QUARTER&#x6216;YEAR&#x3002;&#x53C2;&#x89C1;
          <a
          href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#time-interval-and-point-unit-specifiers">&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x548C;&#x70B9;&#x5355;&#x4F4D;&#x8BF4;&#x660E;&#x7B26;&#x8868;</a>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>TIMESTAMPDIFF(DAY, TIMESTAMP &apos;2003-01-02 10:00:00&apos;, TIMESTAMP &apos;2003-01-03 10:00:00&apos;)</code>&#x5BFC;&#x81F4;<code>1</code>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x65F6;&#x95F4;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING.toDate()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;yyyy-MM-dd&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;HH:mm:ss&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTimestamp()</td>
      <td style="text-align:left">&#x4EE5;&#x201C;yyyy-MM-dd HH:mm:ss[. sss]&#x201D;&#x7684;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.year
        <br />NUMERIC.years</td>
      <td style="text-align:left">&#x4E3A;NUMERIC&#x5E74;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.quarter
        <br />NUMERIC.quarters</td>
      <td style="text-align:left">
        <p>&#x4E3A;NUMERIC&#x5B63;&#x5EA6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.quarters</code>&#x8FD4;&#x56DE;6&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.month
        <br />NUMERIC.months</td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.week
        <br />NUMERIC.weeks</td>
      <td style="text-align:left">
        <p>NUMERIC&#x5468;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.weeks</code>&#x8FD4;&#x56DE;1209600000&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.day
        <br />NUMERIC.days</td>
      <td style="text-align:left">NUMERIC&#x5929;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.hour
        <br />NUMERIC.hours</td>
      <td style="text-align:left">NUMERIC&#x5C0F;&#x65F6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.minute
        <br />NUMERIC.minutes</td>
      <td style="text-align:left">NUMERIC&#x5206;&#x949F;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.second</p>
        <p>NUMERIC.seconds</p>
      </td>
      <td style="text-align:left">NUMERIC&#x79D2;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.milli</p>
        <p>NUMERIC.millis</p>
      </td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentDate()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TEMPORAL.extract(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4ECE;temporal&#x7684;TIMEINTERVALUNIT&#x90E8;&#x5206;&#x63D0;&#x53D6;&#x7684;long&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;2006-06-05&apos;.toDate.extract(DAY)</code>&#x8FD4;&#x56DE;5; <code>&apos;2006-06-05&apos;.toDate.extract(QUARTER)</code>&#x8FD4;&#x56DE;2&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.floor(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toDate.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:44:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.ceil(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toTime.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:45:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">temporalOverlaps(TIMEPOINT1, TEMPORAL1, TIMEPOINT2, TEMPORAL2)</td>
      <td
      style="text-align:left">
        <p>&#x5982;&#x679C;&#xFF08;TIMEPOINT1&#xFF0C;TEMPORAL1&#xFF09;&#x548C;&#xFF08;TIMEPOINT2&#xFF0C;TEMPORAL2&#xFF09;&#x5B9A;&#x4E49;&#x7684;&#x4E24;&#x4E2A;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x91CD;&#x53E0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x65F6;&#x95F4;&#x503C;&#x53EF;&#x4EE5;&#x662F;&#x65F6;&#x95F4;&#x70B9;&#x6216;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>temporalOverlaps(&apos;2:55:00&apos;.toTime, 1.hour, &apos;3:30:00&apos;.toTime, 2.hour)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">dateFormat(TIMESTAMP, STRING)</td>
      <td style="text-align:left"><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x8FD9;&#x4E2A;&#x51FD;&#x6570;&#x6709;&#x4E25;&#x91CD;&#x7684;&#x9519;&#x8BEF;&#xFF0C;&#x73B0;&#x5728;&#x4E0D;&#x5E94;&#x8BE5;&#x4F7F;&#x7528;&#x3002;&#x8BF7;&#x5B9E;&#x73B0;&#x81EA;&#x5B9A;&#x4E49;UDF&#xFF0C;&#x6216;&#x8005;&#x4F7F;&#x7528;extract()&#x4F5C;&#x4E3A;&#x89E3;&#x51B3;&#x65B9;&#x6848;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">timestampDiff(TIMEPOINTUNIT, TIMEPOINT1, TIMEPOINT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;TIMEPOINT1&#x548C;TIMEPOINT2&#x4E4B;&#x95F4;&#x7684;TIMEPOINTUNIT(&#x5E26;&#x7B26;&#x53F7;)&#x6570;&#x91CF;&#x3002;&#x95F4;&#x9694;&#x7684;&#x5355;&#x4F4D;&#x7531;&#x7B2C;&#x4E00;&#x4E2A;&#x53C2;&#x6570;&#x7ED9;&#x51FA;&#xFF0C;&#x8BE5;&#x53C2;&#x6570;&#x5E94;&#x8BE5;&#x662F;&#x4E0B;&#x5217;&#x503C;&#x4E4B;&#x4E00;:<code>SECOND</code>, <code>MINUTE</code>, <code>HOUR</code>, <code>DAY</code>, <code>MONTH</code>,
          or <code>YEAR</code>&#x3002;&#x53C2;&#x89C1;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#time-interval-and-point-unit-specifiers">&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x548C;&#x70B9;&#x5355;&#x4F4D;&#x8BF4;&#x660E;&#x7B26;&#x8868;</a>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>timestampDiff(DAY, &apos;2003-01-02 10:00:00&apos;.toTimestamp, &apos;2003-01-03 10:00:00&apos;.toTimestamp)</code>&#x5BFC;&#x81F4;<code>1</code>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x65F6;&#x95F4;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING.toDate()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;yyyy-MM-dd&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;HH:mm:ss&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTimestamp()</td>
      <td style="text-align:left">&#x4EE5;&#x201C;yyyy-MM-dd HH:mm:ss[. sss]&#x201D;&#x7684;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.year
        <br />NUMERIC.years</td>
      <td style="text-align:left">&#x4E3A;NUMERIC&#x5E74;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.quarter
        <br />NUMERIC.quarters</td>
      <td style="text-align:left">
        <p>&#x4E3A;NUMERIC&#x5B63;&#x5EA6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.quarters</code>&#x8FD4;&#x56DE;6&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.month
        <br />NUMERIC.months</td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.week
        <br />NUMERIC.weeks</td>
      <td style="text-align:left">
        <p>NUMERIC&#x5468;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.weeks</code>&#x8FD4;&#x56DE;1209600000&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.day
        <br />NUMERIC.days</td>
      <td style="text-align:left">NUMERIC&#x5929;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.hour
        <br />NUMERIC.hours</td>
      <td style="text-align:left">NUMERIC&#x5C0F;&#x65F6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.minute
        <br />NUMERIC.minutes</td>
      <td style="text-align:left">NUMERIC&#x5206;&#x949F;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.second</p>
        <p>NUMERIC.seconds</p>
      </td>
      <td style="text-align:left">NUMERIC&#x79D2;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.milli</p>
        <p>NUMERIC.millis</p>
      </td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentDate()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TEMPORAL.extract(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4ECE;temporal&#x7684;TIMEINTERVALUNIT&#x90E8;&#x5206;&#x63D0;&#x53D6;&#x7684;long&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;2006-06-05&apos;.toDate.extract(DAY)</code>&#x8FD4;&#x56DE;5; <code>&apos;2006-06-05&apos;.toDate.extract(QUARTER)</code>&#x8FD4;&#x56DE;2&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.floor(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toDate.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:44:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.ceil(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toTime.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:45:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">temporalOverlaps(TIMEPOINT1, TEMPORAL1, TIMEPOINT2, TEMPORAL2)</td>
      <td
      style="text-align:left">
        <p>&#x5982;&#x679C;&#xFF08;TIMEPOINT1&#xFF0C;TEMPORAL1&#xFF09;&#x548C;&#xFF08;TIMEPOINT2&#xFF0C;TEMPORAL2&#xFF09;&#x5B9A;&#x4E49;&#x7684;&#x4E24;&#x4E2A;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x91CD;&#x53E0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x65F6;&#x95F4;&#x503C;&#x53EF;&#x4EE5;&#x662F;&#x65F6;&#x95F4;&#x70B9;&#x6216;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>temporalOverlaps(&apos;2:55:00&apos;.toTime, 1.hour, &apos;3:30:00&apos;.toTime, 2.hour)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">dateFormat(TIMESTAMP, STRING)</td>
      <td style="text-align:left"><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x8FD9;&#x4E2A;&#x51FD;&#x6570;&#x6709;&#x4E25;&#x91CD;&#x7684;&#x9519;&#x8BEF;&#xFF0C;&#x73B0;&#x5728;&#x4E0D;&#x5E94;&#x8BE5;&#x4F7F;&#x7528;&#x3002;&#x8BF7;&#x5B9E;&#x73B0;&#x81EA;&#x5B9A;&#x4E49;UDF&#xFF0C;&#x6216;&#x8005;&#x4F7F;&#x7528;extract()&#x4F5C;&#x4E3A;&#x89E3;&#x51B3;&#x65B9;&#x6848;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">timestampDiff(TIMEPOINTUNIT, TIMEPOINT1, TIMEPOINT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;TIMEPOINT1&#x548C;TIMEPOINT2&#x4E4B;&#x95F4;&#x7684;TIMEPOINTUNIT(&#x5E26;&#x7B26;&#x53F7;)&#x6570;&#x91CF;&#x3002;&#x95F4;&#x9694;&#x7684;&#x5355;&#x4F4D;&#x7531;&#x7B2C;&#x4E00;&#x4E2A;&#x53C2;&#x6570;&#x7ED9;&#x51FA;&#xFF0C;&#x8BE5;&#x53C2;&#x6570;&#x5E94;&#x8BE5;&#x662F;&#x4E0B;&#x5217;&#x503C;&#x4E4B;&#x4E00;:<code>SECOND</code>, <code>MINUTE</code>, <code>HOUR</code>, <code>DAY</code>, <code>MONTH</code>,
          or <code>YEAR</code>&#x3002;&#x53C2;&#x89C1;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#time-interval-and-point-unit-specifiers">&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x548C;&#x70B9;&#x5355;&#x4F4D;&#x8BF4;&#x660E;&#x7B26;&#x8868;</a>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>timestampDiff(DAY, &apos;2003-01-02 10:00:00&apos;.toTimestamp, &apos;2003-01-03 10:00:00&apos;.toTimestamp)</code>&#x5BFC;&#x81F4;<code>1</code>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x65F6;&#x95F4;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">STRING.toDate</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;yyyy-MM-dd&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTime</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x4EE5;&#x201C;HH:mm:ss&#x201D;&#x5F62;&#x5F0F;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">STRING.toTimestamp</td>
      <td style="text-align:left">&#x4EE5;&#x201C;yyyy-MM-dd HH:mm:ss[. sss]&#x201D;&#x7684;&#x5F62;&#x5F0F;&#x8FD4;&#x56DE;&#x4ECE;&#x5B57;&#x7B26;&#x4E32;&#x89E3;&#x6790;&#x7684;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.year
        <br />NUMERIC.years</td>
      <td style="text-align:left">&#x4E3A;NUMERIC&#x5E74;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.quarter
        <br />NUMERIC.quarters</td>
      <td style="text-align:left">
        <p>&#x4E3A;NUMERIC&#x5B63;&#x5EA6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.quarters</code>&#x8FD4;&#x56DE;6&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.month
        <br />NUMERIC.months</td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x4E2A;&#x6708;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.week
        <br />NUMERIC.weeks</td>
      <td style="text-align:left">
        <p>NUMERIC&#x5468;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>2.weeks</code>&#x8FD4;&#x56DE;1209600000&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.day
        <br />NUMERIC.days</td>
      <td style="text-align:left">NUMERIC&#x5929;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.hour
        <br />NUMERIC.hours</td>
      <td style="text-align:left">NUMERIC&#x5C0F;&#x65F6;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NUMERIC.minute
        <br />NUMERIC.minutes</td>
      <td style="text-align:left">NUMERIC&#x5206;&#x949F;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.second</p>
        <p>NUMERIC.seconds</p>
      </td>
      <td style="text-align:left">NUMERIC&#x79D2;&#x521B;&#x5EFA;&#x4E00;&#x4E2A;&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>NUMERIC.milli</p>
        <p>NUMERIC.millis</p>
      </td>
      <td style="text-align:left">&#x521B;&#x5EFA;NUMERIC&#x6BEB;&#x79D2;&#x7684;&#x95F4;&#x9694;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentDate()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65E5;&#x671F;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">currentTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;UTC&#x65F6;&#x533A;&#x4E2D;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTime()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">localTimestamp()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x672C;&#x5730;&#x65F6;&#x533A;&#x7684;&#x5F53;&#x524D;SQL&#x65F6;&#x95F4;&#x6233;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">TEMPORAL.extract(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4ECE;temporal&#x7684;TIMEINTERVALUNIT&#x90E8;&#x5206;&#x63D0;&#x53D6;&#x7684;long&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;2006-06-05&apos;.toDate.extract(DAY)</code>&#x8FD4;&#x56DE;5; <code>&apos;2006-06-05&apos;.toDate.extract(QUARTER)</code>&#x8FD4;&#x56DE;2&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.floor(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toDate.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:44:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">TIMEPOINT.ceil(TIMEINTERVALUNIT)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x503C;&#xFF0C;&#x8BE5;&#x503C;&#x5C06;TIMEPOINT&#x56DB;&#x820D;&#x4E94;&#x5165;&#x5230;&#x65F6;&#x95F4;&#x5355;&#x5143;TIMEINTERVALUNIT&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;12:44:31&apos;.toTime.floor(MINUTE)</code>&#x8FD4;&#x56DE;12:45:00&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">temporalOverlaps(TIMEPOINT1, TEMPORAL1, TIMEPOINT2, TEMPORAL2)</td>
      <td
      style="text-align:left">
        <p>&#x5982;&#x679C;&#xFF08;TIMEPOINT1&#xFF0C;TEMPORAL1&#xFF09;&#x548C;&#xFF08;TIMEPOINT2&#xFF0C;TEMPORAL2&#xFF09;&#x5B9A;&#x4E49;&#x7684;&#x4E24;&#x4E2A;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x91CD;&#x53E0;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;TRUE
          &#x3002;&#x65F6;&#x95F4;&#x503C;&#x53EF;&#x4EE5;&#x662F;&#x65F6;&#x95F4;&#x70B9;&#x6216;&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>temporalOverlaps(&apos;2:55:00&apos;.toTime, 1.hour, &apos;3:30:00&apos;.toTime, 2.hour)</code>&#x8FD4;&#x56DE;TRUE&#x3002;</p>
        </td>
    </tr>
    <tr>
      <td style="text-align:left">dateFormat(TIMESTAMP, STRING)</td>
      <td style="text-align:left"><b>&#x6CE8;&#x610F;&#xFF1A;</b>&#x8FD9;&#x4E2A;&#x51FD;&#x6570;&#x6709;&#x4E25;&#x91CD;&#x7684;&#x9519;&#x8BEF;&#xFF0C;&#x73B0;&#x5728;&#x4E0D;&#x5E94;&#x8BE5;&#x4F7F;&#x7528;&#x3002;&#x8BF7;&#x5B9E;&#x73B0;&#x81EA;&#x5B9A;&#x4E49;UDF&#xFF0C;&#x6216;&#x8005;&#x4F7F;&#x7528;extract()&#x4F5C;&#x4E3A;&#x89E3;&#x51B3;&#x65B9;&#x6848;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">timestampDiff(TIMEPOINTUNIT, TIMEPOINT1, TIMEPOINT2)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;TIMEPOINT1&#x548C;TIMEPOINT2&#x4E4B;&#x95F4;&#x7684;TIMEPOINTUNIT(&#x5E26;&#x7B26;&#x53F7;)&#x6570;&#x91CF;&#x3002;&#x95F4;&#x9694;&#x7684;&#x5355;&#x4F4D;&#x7531;&#x7B2C;&#x4E00;&#x4E2A;&#x53C2;&#x6570;&#x7ED9;&#x51FA;&#xFF0C;&#x8BE5;&#x53C2;&#x6570;&#x5E94;&#x8BE5;&#x662F;&#x4E0B;&#x5217;&#x503C;&#x4E4B;&#x4E00;:<code>SECOND</code>, <code>MINUTE</code>, <code>HOUR</code>, <code>DAY</code>, <code>MONTH</code>,
          or <code>YEAR</code>&#x3002;&#x53C2;&#x89C1;<a href="https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#time-interval-and-point-unit-specifiers">&#x65F6;&#x95F4;&#x95F4;&#x9694;&#x548C;&#x70B9;&#x5355;&#x4F4D;&#x8BF4;&#x660E;&#x7B26;&#x8868;</a>&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>timestampDiff(DAY, &apos;2003-01-02 10:00:00&apos;.toTimestamp, &apos;2003-01-03 10:00:00&apos;.toTimestamp)</code>&#x5BFC;&#x81F4;<code>1</code>&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 条件函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6761;&#x4EF6;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>CASE value</p>
        <p>WHEN value1_1 [, value1_2 ]<em> THEN result1 </em>
        </p>
        <p><em>[ WHEN value2_1 [, value2_2 ]</em> THEN result2 ]*</p>
        <p>[ ELSE resultZ ]</p>
        <p>END</p>
      </td>
      <td style="text-align:left">&#x5728;(valueX_1, valueX_2&#xFF0C;&#x2026;)&#x4E2D;&#x5305;&#x542B;&#x7B2C;&#x4E00;&#x4E2A;&#x65F6;&#x95F4;&#x503C;&#x65F6;&#x8FD4;&#x56DE;resultX&#x3002;&#x5982;&#x679C;&#x6CA1;&#x6709;&#x5339;&#x914D;&#x7684;&#x503C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;resultZ(&#x5982;&#x679C;&#x63D0;&#x4F9B;)&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>CASE WHEN</p>
        <p>condition1 THEN result1</p>
        <p>[ WHEN condition2 THEN result2 ]*</p>
        <p>[ ELSE resultZ ]</p>
        <p>END</p>
      </td>
      <td style="text-align:left">&#x5728;&#x6EE1;&#x8DB3;&#x7B2C;&#x4E00;&#x4E2A;&#x6761;&#x4EF6;&#x65F6;&#x8FD4;&#x56DE;resultX&#x3002;&#x5F53;&#x4E0D;&#x6EE1;&#x8DB3;&#x4EFB;&#x4F55;&#x6761;&#x4EF6;&#x65F6;&#xFF0C;&#x5982;&#x679C;&#x63D0;&#x4F9B;&#x4E86;&#x7ED3;&#x679C;&#xFF0C;&#x5219;&#x8FD4;&#x56DE;resultZ&#xFF0C;&#x5426;&#x5219;&#x8FD4;&#x56DE;NULL&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">NULLIF(value1, value2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;value1&#x7B49;&#x4E8E;value2&#xFF0C;&#x8FD4;&#x56DE;NULL;&#x5426;&#x5219;&#x8FD4;&#x56DE;value1&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;NULLIF(5,5)&#x8FD4;&#x56DE;NULL;NULLIF(5,0)&#x8FD4;&#x56DE;5&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">COALESCE(value1, value2 [, value3 ]* )</td>
      <td style="text-align:left">
        <p>&#x4ECE;value1, value2&#xFF0C;&#x2026;&#x8FD4;&#x56DE;&#x7B2C;&#x4E00;&#x4E2A;&#x4E0D;&#x4E3A;&#x7A7A;&#x7684;&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;COALESCE(NULL, 5)&#x8FD4;&#x56DE;5&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left"><em>&#x6761;&#x4EF6;&#x51FD;&#x6570;</em>
      </th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN.?(VALUE1, VALUE2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN&#x8BC4;&#x4F30;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;VALUE1
          ; &#x5426;&#x5219;&#x8FD4;&#x56DE;VALUE2&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>(42 &gt; 5).?(&apos;A&apos;, &apos;B&apos;)</code>&#x8FD4;&#x56DE;&#x201C;A&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left"><em>&#x6761;&#x4EF6;&#x51FD;&#x6570;</em>
      </th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN.?(VALUE1, VALUE2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN&#x8BC4;&#x4F30;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;VALUE1
          ; &#x5426;&#x5219;&#x8FD4;&#x56DE;VALUE2&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>(42 &gt; 5).?(&apos;A&apos;, &apos;B&apos;)</code>&#x8FD4;&#x56DE;&#x201C;A&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x6761;&#x4EF6;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">BOOLEAN.?(VALUE1, VALUE2)</td>
      <td style="text-align:left">
        <p>&#x5982;&#x679C;BOOLEAN&#x8BC4;&#x4F30;&#x4E3A;TRUE&#xFF0C;&#x5219;&#x8FD4;&#x56DE;VALUE1
          ; &#x5426;&#x5219;&#x8FD4;&#x56DE;VALUE2&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>(42 &gt; 5).?(&quot;A&quot;, &quot;B&quot;)</code>&#x8FD4;&#x56DE;&#x201C;A&#x201D;&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 类型转换函数

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">CAST(value AS type)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x8981;&#x8F6C;&#x6362;&#x4E3A;type&#x7C7B;&#x578B;&#x7684;&#x65B0;&#x503C;&#x3002;&#x5728;&#x8FD9;&#x91CC;&#x67E5;&#x770B;&#x652F;&#x6301;&#x7684;&#x7C7B;&#x578B;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;CAST(&apos;42&apos; AS INT)&#x8FD4;&#x56DE;42;CAST(NULL
          AS VARCHAR)&#x8FD4;&#x56DE;&#x7C7B;&#x578B;&#x4E3A;VARCHAR&#x7684;NULL&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY.cast(TYPE)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x7684;ANY&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;type&#x7C7B;&#x578B;&#x3002;&#x5728;&#x8FD9;&#x91CC;&#x67E5;&#x770B;&#x652F;&#x6301;&#x7684;&#x7C7B;&#x578B;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&apos;42&apos;.cast(INT)&#x8FD4;&#x56DE;42;Null(&#x5B57;&#x7B26;&#x4E32;)&#x8FD4;&#x56DE;&#x5B57;&#x7B26;&#x4E32;&#x7C7B;&#x578B;&#x7684;Null&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY.cast(TYPE)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x7684;ANY&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;type&#x7C7B;&#x578B;&#x3002;&#x5728;&#x8FD9;&#x91CC;&#x67E5;&#x770B;&#x652F;&#x6301;&#x7684;&#x7C7B;&#x578B;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&apos;42&apos;.cast(INT)&#x8FD4;&#x56DE;42;Null(&#x5B57;&#x7B26;&#x4E32;)&#x8FD4;&#x56DE;&#x5B57;&#x7B26;&#x4E32;&#x7C7B;&#x578B;&#x7684;Null&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">ANY.cast(TYPE)</td>
      <td style="text-align:left">
        <p>&#x8FD4;&#x56DE;&#x4E00;&#x4E2A;&#x65B0;&#x7684;ANY&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;type&#x7C7B;&#x578B;&#x3002;&#x5728;&#x8FD9;&#x91CC;&#x67E5;&#x770B;&#x652F;&#x6301;&#x7684;&#x7C7B;&#x578B;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;&#x201C;42&#x201D;.cast(Types.INT)&#x8FD4;&#x56DE;42;Null(Types.STRING)&#x8FD4;&#x56DE;STRING&#x7C7B;&#x578B;&#x7684;Null&#x3002;</p>
      </td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 集合函数

{% tabs %}
{% tab title="SQL" %}
| 集合函数 | 描述 |
| :--- | :--- |
| CARDINALITY\(array\) | 返回数组中的元素数。 |
| array ‘\[’ integer ‘\]’ | 返回数组中位置为整数的元素。指数从1开始。 |
| ELEMENT\(array\) | 返回数组的唯一元素\(其基数应为1\);如果数组为空，返回NULL。如果数组中有多个元素，则引发异常。 |
| CARDINALITY\(map\) | 返回map中的条目数。 |
| map ‘\[’ value ‘\]’ | 返回map中键值指定的值。 |
{% endtab %}

{% tab title="Java" %}
| 集合函数 | 描述 |
| :--- | :--- |
| ARRAY.cardinality\(\) | 返回数组中的元素数。 |
| ARRAY.at\(INT\) | 返回数组中位置INT处的元素。指数从1开始。 |
| ARRAY.element\(\) | 返回数组的唯一元素\(其基数应为1\);如果数组为空，返回NULL。如果数组中有多个元素，则引发异常。 |
| MAP.cardinality\(\) | 返回Map中的条目数。 |
| MAP.at\(ANY\) | 返回MAP中键ANY指定的值。 |
{% endtab %}

{% tab title="Python" %}
| 集合函数 | 描述 |
| :--- | :--- |
| ARRAY.cardinality\(\) | 返回数组中的元素数。 |
| ARRAY.at\(INT\) | 返回数组中位置INT处的元素。指数从1开始。 |
| ARRAY.element\(\) | 返回数组的唯一元素\(其基数应为1\);如果数组为空，返回NULL。如果数组中有多个元素，则引发异常。 |
| MAP.cardinality\(\) | 返回Map中的条目数。 |
| MAP.at\(ANY\) | 返回MAP中键ANY指定的值。 |
{% endtab %}

{% tab title="Scala" %}
| 集合函数 | 描述 |
| :--- | :--- |
| ARRAY.cardinality\(\) | 返回数组中的元素数。 |
| ARRAY.at\(INT\) | 返回数组中位置INT处的元素。指数从1开始。 |
| ARRAY.element\(\) | 返回数组的唯一元素\(其基数应为1\);如果数组为空，返回NULL。如果数组中有多个元素，则引发异常。 |
| MAP.cardinality\(\) | 返回Map中的条目数。 |
| MAP.at\(ANY\) | 返回MAP中键ANY指定的值。 |
{% endtab %}
{% endtabs %}

### 值构建函数

{% tabs %}
{% tab title="SQL" %}
| 值构建函数 | 描述 |
| :--- | :--- |
| ROW\(value1, \[, value2\]_\) \(value1, \[, value2\]_\) | 返回从值列表（value1，value2， ...）创建的行。 |
| ARRAY ‘\[’ value1 \[, value2 \]\* ‘\]’ | 返回从值列表（value1，value2，...）创建的数组。 |
| MAP ‘\[’ value1, value2 \[, value3, value4 \]\* ‘\]’ | 返回从键值对列表（（value1，value2），（value3，value4），...）创建的映射。 |
{% endtab %}

{% tab title="Java" %}
| 值构建函数 | 描述 |
| :--- | :--- |
| row\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的行。行是复合类型，可以通过[值访问函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#value-access-functions)进行访问。 |
| array\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的数组 |
| map\(ANY1, ANY2, ANY3, ANY4, ...\) | 返回从键值对列表（（ANY1，ANY2），（ANY3，ANY4），...）创建的map。 |
| NUMERIC.rows | 创建NUMERIC行间隔（通常用于创建窗口）。 |
{% endtab %}

{% tab title="Python" %}
| 值构建函数 | 描述 |
| :--- | :--- |
| row\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的行。行是复合类型，可以通过[值访问函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#value-access-functions)进行访问。 |
| array\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的数组 |
| map\(ANY1, ANY2, ANY3, ANY4, ...\) | 返回从键值对列表（（ANY1，ANY2），（ANY3，ANY4），...）创建的map。 |
| NUMERIC.rows | 创建NUMERIC行间隔（通常用于创建窗口）。 |
{% endtab %}

{% tab title="Scala" %}
| 值构建函数 | 描述 |
| :--- | :--- |
| row\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的行。行是复合类型，可以通过[值访问函数](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/table/functions.html#value-access-functions)进行访问。 |
| array\(ANY1, ANY2, ...\) | 返回从对象值列表（ANY1，ANY2，...）创建的数组 |
| map\(ANY1, ANY2, ANY3, ANY4, ...\) | 返回从键值对列表（（ANY1，ANY2），（ANY3，ANY4），...）创建的map。 |
| NUMERIC.rows | 创建NUMERIC行间隔（通常用于创建窗口）。 |
{% endtab %}
{% endtabs %}

### 值访问函数

{% tabs %}
{% tab title="SQL" %}
| 值访问函数 | 描述 |
| :--- | :--- |
| tableName.compositeType.field | 按名称返回Flink复合类型（例如，Tuple，POJO）中的字段值。 |
| tableName.compositeType.\* | 返回Flink复合类型\(例如Tuple、POJO\)的平面表示形式，该类型将其每个直接子类型转换为单独的字段。在大多数情况下，平面表示的字段的命名与原始字段类似，但使用美元分隔符\(例如，mypojo$mytuple$f0\)。 |
{% endtab %}

{% tab title="Java" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x503C;&#x8BBF;&#x95EE;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>COMPOSITE.get(STRING)</p>
        <p>COMPOSITE.get(INT)</p>
      </td>
      <td style="text-align:left">
        <p>&#x6309;&#x540D;&#x79F0;&#x6216;&#x7D22;&#x5F15;&#x4ECE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x8FD4;&#x56DE;&#x5B57;&#x6BB5;&#x7684;&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;pojo.get(&quot;myField&quot;)</code>&#x6216;<code>&apos;tuple.get(0)</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.flatten()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x7684;&#x5E73;&#x9762;&#x8868;&#x793A;&#xFF0C;&#x5C06;&#x5176;&#x6BCF;&#x4E2A;&#x76F4;&#x63A5;&#x5B50;&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x72EC;&#x7684;&#x5B57;&#x6BB5;&#x3002;&#x5728;&#x5927;&#x591A;&#x6570;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5E73;&#x9762;&#x8868;&#x793A;&#x7684;&#x5B57;&#x6BB5;&#x7684;&#x547D;&#x540D;&#x65B9;&#x5F0F;&#x4E0E;&#x539F;&#x59CB;&#x5B57;&#x6BB5;&#x7C7B;&#x4F3C;&#xFF0C;&#x4F46;&#x4F7F;&#x7528;&#x7F8E;&#x5143;&#x5206;&#x9694;&#x7B26;&#xFF08;&#x4F8B;&#x5982;<code>mypojo$mytuple$f0</code>&#xFF09;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Python" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x503C;&#x8BBF;&#x95EE;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>COMPOSITE.get(STRING)</p>
        <p>COMPOSITE.get(INT)</p>
      </td>
      <td style="text-align:left">
        <p>&#x6309;&#x540D;&#x79F0;&#x6216;&#x7D22;&#x5F15;&#x4ECE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x8FD4;&#x56DE;&#x5B57;&#x6BB5;&#x7684;&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>&apos;pojo.get(&quot;myField&quot;)</code>&#x6216;<code>&apos;tuple.get(0)</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.flatten()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x7684;&#x5E73;&#x9762;&#x8868;&#x793A;&#xFF0C;&#x5C06;&#x5176;&#x6BCF;&#x4E2A;&#x76F4;&#x63A5;&#x5B50;&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x72EC;&#x7684;&#x5B57;&#x6BB5;&#x3002;&#x5728;&#x5927;&#x591A;&#x6570;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5E73;&#x9762;&#x8868;&#x793A;&#x7684;&#x5B57;&#x6BB5;&#x7684;&#x547D;&#x540D;&#x65B9;&#x5F0F;&#x4E0E;&#x539F;&#x59CB;&#x5B57;&#x6BB5;&#x7C7B;&#x4F3C;&#xFF0C;&#x4F46;&#x4F7F;&#x7528;&#x7F8E;&#x5143;&#x5206;&#x9694;&#x7B26;&#xFF08;&#x4F8B;&#x5982;<code>mypojo$mytuple$f0</code>&#xFF09;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Scala" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x503C;&#x8BBF;&#x95EE;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>COMPOSITE.get(STRING)</p>
        <p>COMPOSITE.get(INT)</p>
      </td>
      <td style="text-align:left">
        <p>&#x6309;&#x540D;&#x79F0;&#x6216;&#x7D22;&#x5F15;&#x4ECE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x8FD4;&#x56DE;&#x5B57;&#x6BB5;&#x7684;&#x503C;&#x3002;</p>
        <p>&#x4F8B;&#x5982;&#xFF0C;<code>pojo.get(&apos;myField&apos;)</code>&#x6216;<code>tuple.get(0)</code>&#x3002;</p>
      </td>
    </tr>
    <tr>
      <td style="text-align:left">ANY.flatten()</td>
      <td style="text-align:left">&#x8FD4;&#x56DE;Flink&#x590D;&#x5408;&#x7C7B;&#x578B;&#xFF08;&#x4F8B;&#x5982;&#xFF0C;Tuple&#xFF0C;POJO&#xFF09;&#x7684;&#x5E73;&#x9762;&#x8868;&#x793A;&#xFF0C;&#x5C06;&#x5176;&#x6BCF;&#x4E2A;&#x76F4;&#x63A5;&#x5B50;&#x7C7B;&#x578B;&#x8F6C;&#x6362;&#x4E3A;&#x5355;&#x72EC;&#x7684;&#x5B57;&#x6BB5;&#x3002;&#x5728;&#x5927;&#x591A;&#x6570;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x5E73;&#x9762;&#x8868;&#x793A;&#x7684;&#x5B57;&#x6BB5;&#x7684;&#x547D;&#x540D;&#x65B9;&#x5F0F;&#x4E0E;&#x539F;&#x59CB;&#x5B57;&#x6BB5;&#x7C7B;&#x4F3C;&#xFF0C;&#x4F46;&#x4F7F;&#x7528;&#x7F8E;&#x5143;&#x5206;&#x9694;&#x7B26;&#xFF08;&#x4F8B;&#x5982;<code>mypojo$mytuple$f0</code>&#xFF09;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}
{% endtabs %}

### 分组函数

{% tabs %}
{% tab title="SQL" %}
| 分组函数 | 描述 |
| :--- | :--- |
| GROUP\_ID\(\) | 返回唯一标识分组键组合的整数。 |
| GROUPING\(expression1 \[, expression2\] _\) GROUPING\_ID\(expression1 \[, expression2\]_ \) | 返回给定分组表达式的位向量。 |
{% endtab %}

{% tab title="Java" %}
| 分组函数 | 描述 |
| :--- | :--- |
{% endtab %}

{% tab title="Python" %}
| 分组函数 | 描述 |
| :--- | :--- |
{% endtab %}

{% tab title="Scala" %}
| 分组函数 | 描述 |
| :--- | :--- |
{% endtab %}
{% endtabs %}

### Hash函数

{% tabs %}
{% tab title="SQL" %}
| Hash函数 | 描述 |
| :--- | :--- |
| `MD5(string)` | 以32位十六进制数字的字符串形式返回字符串的MD5散列;如果字符串为空，返回NULL。 |
| `SHA1(string)` | 以40个十六进制数字的字符串形式返回字符串的SHA-1散列;如果字符串为空，返回NULL。 |
| `SHA224(string)` | 以56位十六进制数字的字符串形式返回SHA-224字符串散列;如果字符串为空，返回NULL。 |
| `SHA256(string)` | 以64位十六进制数字的字符串形式返回SHA-256字符串散列;如果字符串为空，返回NULL。 |
| `SHA384(string)` | 以96位十六进制数字的字符串形式返回字符串的SHA-384散列;如果字符串为空，返回NULL。 |
| `SHA512(string)` | 以128位十六进制数字的字符串形式返回字符串SHA-512散列;如果字符串为空，返回NULL。 |
| `SHA2(string, hashLength)` | 使用SHA-2系列散列函数\(SHA-224、SHA-256、SHA-384或SHA-512\)返回散列。第一个参数字符串是要散列的字符串，第二个参数hashLength是结果的位长\(224、256、384或512\)。如果字符串或hashLength为空，则返回NULL。 |
{% endtab %}

{% tab title="Java" %}
| Hash函数 | 描述 |
| :--- | :--- |
| STRING.md5\(\) | 以32位十六进制数字的字符串形式返回字符串的MD5散列;如果字符串为空，返回NULL。 |
| STRING.sha1\(\) | 以40个十六进制数字的字符串形式返回字符串的SHA-1散列;如果字符串为空，返回NULL。 |
| STRING.sha224\(\) | 以56位十六进制数字的字符串形式返回SHA-224字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha256\(\) | 以64位十六进制数字的字符串形式返回SHA-256字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha384\(\) | 以96位十六进制数字的字符串形式返回字符串的SHA-384散列;如果字符串为空，返回NULL。 |
| STRING.sha512\(\) | 以128位十六进制数字的字符串形式返回字符串SHA-512散列;如果字符串为空，返回NULL。 |
| STRING.sha2\(\) | 返回由INT\(可以是224、256、384或512\)为字符串指定的散列值SHA-2族\(SHA-224、SHA-256、SHA-384或SHA-512\)。如果字符串或INT为空，返回NULL。 |
{% endtab %}

{% tab title="Python" %}
| Hash函数 | 描述 |
| :--- | :--- |
| STRING.md5\(\) | 以32位十六进制数字的字符串形式返回字符串的MD5散列;如果字符串为空，返回NULL。 |
| STRING.sha1\(\) | 以40个十六进制数字的字符串形式返回字符串的SHA-1散列;如果字符串为空，返回NULL。 |
| STRING.sha224\(\) | 以56位十六进制数字的字符串形式返回SHA-224字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha256\(\) | 以64位十六进制数字的字符串形式返回SHA-256字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha384\(\) | 以96位十六进制数字的字符串形式返回字符串的SHA-384散列;如果字符串为空，返回NULL。 |
| STRING.sha512\(\) | 以128位十六进制数字的字符串形式返回字符串SHA-512散列;如果字符串为空，返回NULL。 |
| STRING.sha2\(\) | 返回由INT\(可以是224、256、384或512\)为字符串指定的散列值SHA-2族\(SHA-224、SHA-256、SHA-384或SHA-512\)。如果字符串或INT为空，返回NULL。 |
{% endtab %}

{% tab title="Scala" %}


| Hash函数 | 描述 |
| :--- | :--- |
| STRING.md5\(\) | 以32位十六进制数字的字符串形式返回字符串的MD5散列;如果字符串为空，返回NULL。 |
| STRING.sha1\(\) | 以40个十六进制数字的字符串形式返回字符串的SHA-1散列;如果字符串为空，返回NULL。 |
| STRING.sha224\(\) | 以56位十六进制数字的字符串形式返回SHA-224字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha256\(\) | 以64位十六进制数字的字符串形式返回SHA-256字符串散列;如果字符串为空，返回NULL。 |
| STRING.sha384\(\) | 以96位十六进制数字的字符串形式返回字符串的SHA-384散列;如果字符串为空，返回NULL。 |
| STRING.sha512\(\) | 以128位十六进制数字的字符串形式返回字符串SHA-512散列;如果字符串为空，返回NULL。 |
| STRING.sha2\(\) | 返回由INT\(可以是224、256、384或512\)为字符串指定的散列值SHA-2族\(SHA-224、SHA-256、SHA-384或SHA-512\)。如果字符串或INT为空，返回NULL。 |
{% endtab %}
{% endtabs %}

### 辅助函数

{% tabs %}
{% tab title="SQL" %}
| 辅助函数 | 描述 |
| :--- | :--- |
{% endtab %}

{% tab title="Java" %}
| 辅助函数 | 描述 |
| :--- | :--- |
| `ANY.as(NAME1, NAME2, ...)` | 指定`ANY(field)`的名称。如果表达式扩展到多个字段，则可以指定其他名称。 |
{% endtab %}

{% tab title="Python" %}
| 辅助函数 | 描述 |
| :--- | :--- |
| `ANY.as(NAME1, NAME2, ...)` | 指定`ANY(field)`的名称。如果表达式扩展到多个字段，则可以指定其他名称。 |
{% endtab %}

{% tab title="Scala" %}
| 辅助函数 | 描述 |
| :--- | :--- |
| ANY.as\(NAME1, NAME2, ...\) | 指定`ANY(field)`的名称。如果表达式扩展到多个字段，则可以指定其他名称。 |
{% endtab %}
{% endtabs %}

## 聚合函数

聚合函数将所有行的表达式作为输入，并返回单个聚合值作为结果。

{% tabs %}
{% tab title="SQL" %}
<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x805A;&#x5408;&#x51FD;&#x6570;</th>
      <th style="text-align:left">&#x63CF;&#x8FF0;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>COUNT([ ALL ] expression | DISTINCT expression1 [, expression2]*)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x8868;&#x8FBE;&#x5F0F;&#x4E0D;&#x4E3A;NULL
        &#x7684;&#x8F93;&#x5165;&#x884C;&#x6570;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p><code>COUNT(*)</code>
        </p>
        <p><code>COUNT(1)</code>
        </p>
      </td>
      <td style="text-align:left">&#x8FD4;&#x56DE;&#x8F93;&#x5165;&#x884C;&#x6570;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>AVG([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x5E73;&#x5747;&#x503C;&#xFF08;&#x7B97;&#x672F;&#x5E73;&#x5747;&#x503C;&#xFF09;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>SUM([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x8868;&#x8FBE;&#x5F0F;&#x603B;&#x548C;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MAX([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x6700;&#x5927;&#x503C;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>MIN([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x4E2D;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x6700;&#x5C0F;&#x503C;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>STDDEV_POP([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x603B;&#x4F53;&#x6807;&#x51C6;&#x5DEE;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>STDDEV_SAMP([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x4E2D;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x6837;&#x672C;&#x6807;&#x51C6;&#x5DEE;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>VAR_POP([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x603B;&#x4F53;&#x65B9;&#x5DEE;(&#x603B;&#x4F53;&#x6807;&#x51C6;&#x5DEE;&#x7684;&#x5E73;&#x65B9;)&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>VAR_SAMP([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x4E2D;&#x8868;&#x8FBE;&#x5F0F;&#x7684;&#x6837;&#x672C;&#x65B9;&#x5DEE;&#xFF08;&#x6837;&#x672C;&#x6807;&#x51C6;&#x5DEE;&#x7684;&#x5E73;&#x65B9;&#xFF09;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>COLLECT([ ALL | DISTINCT ] expression)</code>
      </td>
      <td style="text-align:left">&#x9ED8;&#x8BA4;&#x60C5;&#x51B5;&#x4E0B;&#xFF0C;&#x6216;&#x4F7F;&#x7528;&#x5173;&#x952E;&#x5B57;ALL&#xFF0C;&#x8FD4;&#x56DE;&#x8DE8;&#x6240;&#x6709;&#x8F93;&#x5165;&#x884C;&#x7684;&#x591A;&#x7EC4;&#x8868;&#x8FBE;&#x5F0F;&#x3002;NULL&#x503C;&#x5C06;&#x88AB;&#x5FFD;&#x7565;&#x3002;&#x5C06;DISTINCT&#x7528;&#x4E8E;&#x6BCF;&#x4E2A;&#x503C;&#x7684;&#x4E00;&#x4E2A;&#x552F;&#x4E00;&#x5B9E;&#x4F8B;&#x3002;</td>
    </tr>
  </tbody>
</table>
{% endtab %}

{% tab title="Java" %}
| 聚合函数 | 描述 |
| :--- | :--- |
| `FIELD.count` | 返回FIELD不为空的输入行数。 |
| `FIELD.avg` | 返回所有输入行的FIELD的平均值\(算术平均值\)。 |
| `FIELD.sum` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回NULL。 |
| `FIELD.sum0` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回0。 |
| `FIELD.max` | 返回所有输入行中数字字段FIELD的最大值。 |
| `FIELD.min` | 返回所有输入行中数字字段FIELD的最小值。 |
| `FIELD.stddevPop` | 返回所有输入行中数字字段FIELD的总体标准差。 |
| `FIELD.stddevSamp` | 返回所有输入行中数字字段FIELD的样本标准差。 |
| `FIELD.varPop` | 返回所有输入行中数字字段FIELD的总体方差（总体标准差的平方）。 |
| `FIELD.varSamp` | 返回所有输入行中数字字段FIELD的样本方差（样本标准差的平方）。 |
| `FIELD.collect` | 返回跨所有输入行的多组字段。 |
{% endtab %}

{% tab title="Python" %}
| 聚合函数 | 描述 |
| :--- | :--- |
| `FIELD.count` | 返回FIELD不为空的输入行数。 |
| `FIELD.avg` | 返回所有输入行的FIELD的平均值\(算术平均值\)。 |
| `FIELD.sum` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回NULL。 |
| `FIELD.sum0` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回0。 |
| `FIELD.max` | 返回所有输入行中数字字段FIELD的最大值。 |
| `FIELD.min` | 返回所有输入行中数字字段FIELD的最小值。 |
| `FIELD.stddevPop` | 返回所有输入行中数字字段FIELD的总体标准差。 |
| `FIELD.stddevSamp` | 返回所有输入行中数字字段FIELD的样本标准差。 |
| `FIELD.varPop` | 返回所有输入行中数字字段FIELD的总体方差（总体标准差的平方）。 |
| `FIELD.varSamp` | 返回所有输入行中数字字段FIELD的样本方差（样本标准差的平方）。 |
| `FIELD.collect` | 返回跨所有输入行的多组字段。 |
{% endtab %}

{% tab title="Scala" %}
| 聚合函数 | 描述 |
| :--- | :--- |
| `FIELD.count` | 返回FIELD不为空的输入行数。 |
| `FIELD.avg` | 返回所有输入行的FIELD的平均值\(算术平均值\)。 |
| `FIELD.sum` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回NULL。 |
| `FIELD.sum0` | 返回所有输入行中数字字段FIELD的总和。如果所有值都为NULL，则返回0。 |
| `FIELD.max` | 返回所有输入行中数字字段FIELD的最大值。 |
| `FIELD.min` | 返回所有输入行中数字字段FIELD的最小值。 |
| `FIELD.stddevPop` | 返回所有输入行中数字字段FIELD的总体标准差。 |
| `FIELD.stddevSamp` | 返回所有输入行中数字字段FIELD的样本标准差。 |
| `FIELD.varPop` | 返回所有输入行中数字字段FIELD的总体方差（总体标准差的平方）。 |
| `FIELD.varSamp` | 返回所有输入行中数字字段FIELD的样本方差（样本标准差的平方）。 |
| `FIELD.collect` | 返回跨所有输入行的多组字段。 |
{% endtab %}
{% endtabs %}

## 日期格式标识符

下表列出了日期格式函数的说明符。

| 标识符 | 说明 |
| :--- | :--- |
| `%a` | 工作日缩写名称（`Sun`.. `Sat`） |
| `%b` | 月份名称缩写（`Jan`.. `Dec`） |
| `%c` | 月，数字（`1`.. `12`） |
| `%D` | 这个月的一天，英语后缀（`0th`，`1st`，`2nd`，`3rd`，...） |
| `%d` | 每月的某一天，数字（`01`.. `31`） |
| `%e` | 每月的某一天，数字（`1`.. `31`） |
| `%f` | 第二个分数（打印6位数：`000000`... `999000`;解析时为1 - 9位数：`0`..`999999999`）（时间戳被截断为毫秒。） |
| `%H` | 小时（`00`.. `23`） |
| `%h` | 小时（`01`.. `12`） |
| `%I` | 小时（`01`.. `12`） |
| `%i` | 分钟，数字（`00`.. `59`） |
| `%j` | 一年中的一天（`001`.. `366`） |
| `%k` | 小时（`0`.. `23`） |
| `%l` | 小时（`1`.. `12`） |
| `%M` | 月份名称（`January`.. `December`） |
| `%m` | 月，数字（`01`.. `12`） |
| `%p` | `AM` 、 `PM` |
| `%r` | 时间，12小时（`hh:mm:ss`其次是`AM`或`PM`） |
| `%S` | 秒（`00`... `59`） |
| `%s` | 秒（`00`... `59`） |
| `%T` | 时间，24小时（`hh:mm:ss`） |
| `%U` | 周（`00`.. `53`），周日是一周的第一天 |
| `%u` | 周（`00`.. `53`），周一是一周的第一天 |
| `%V` | 周（`01`.. `53`），周日是一周的第一天; 用于`%X` |
| `%v` | 周（`01`.. `53`），周一是一周的第一天; 用于`%x` |
| `%W` | 工作日名称（`Sunday`.. `Saturday`） |
| `%w` | 星期几（`0`.. `6`），星期日是一周的第一天 |
| `%X` | 星期日是星期的第一天的星期，数字，四位数; 用于`%V` |
| `%x` | 一周的年份，星期一是一周的第一天，数字，四位数; 用于`%v` |
| `%Y` | 年份，数字，四位数 |
| `%y` | 年份，数字（两位数） |
| `%%` | 文字`%`字符 |
| `%x` | `x`，对于`x`上面未列出的任何内容 |

## 时间间隔和点单位标识符

下表列出了时间间隔和时间点单位的说明符。

对于Table API，请使用`_`空格（例如`DAY_TO_HOUR`）。

| 时间间隔单位 | 时间点单位 |
| :--- | :--- |
| `MILLENIUM` _\(SQL-only\)_ |  |
| `CENTURY` _\(SQL-only\)_ |  |
| `YEAR` | `YEAR` |
| `YEAR TO MONTH` | \`\` |
| `QUARTER` | `QUARTER` |
| `MONTH` | `MONTH` |
| `WEEK` | `WEEK` |
| `DAY` | `DAY` |
| `DAY TO HOUR` | \`\` |
| `DAY TO MINUTE` | \`\` |
| `DAY TO SECOND` | \`\` |
| `HOUR` | `HOUR` |
| `HOUR TO MINUTE` | \`\` |
| `HOUR TO SECOND` | \`\` |
| `MINUTE` | `MINUTE` |
| `MINUTE TO SECOND` | \`\` |
| `SECOND` | `SECOND` |
| \`\` | `MILLISECOND` |
| \`\` | `MICROSECOND` |
| `DOY` _\(SQL-only\)_ |  |
| `DOW` _\(SQL-only\)_ |  |
|  | `SQL_TSI_YEAR` _\(SQL-only\)_ |
|  | `SQL_TSI_QUARTER` _\(SQL-only\)_ |
|  | `SQL_TSI_MONTH` _\(SQL-only\)_ |
|  | `SQL_TSI_WEEK` _\(SQL-only\)_ |
|  | `SQL_TSI_DAY` _\(SQL-only\)_ |
|  | `SQL_TSI_HOUR` _\(SQL-only\)_ |
|  | `SQL_TSI_MINUTE` _\(SQL-only\)_ |
|  | `SQL_TSI_SECOND` _\(SQL-only\)_ |

## 列函数

列函数用于选择或取消选择表列。

| 语法 | 描述 |
| :--- | :--- |
| withColumns\(…\) | 选择指定的列 |
| withoutColumns\(…\) | 取消选择指定的列 |

详细语法如下：

```text
columnFunction:
    withColumns(columnExprs)
    withoutColumns(columnExprs)

columnExprs:
    columnExpr [, columnExpr]*

columnExpr:
    columnRef | columnIndex to columnIndex | columnName to columnName

columnRef:
    columnName(The field name that exists in the table) | columnIndex(a positive integer starting from 1)
```

下表说明了column函数的用法。（假设我们有一个包含5列的表格：）`(a: Int, b: Long, c: String, d:String, e: String)`：

{% tabs %}
{% tab title="Java" %}
| Api | 用法 | 描述 |
| :--- | :--- | :--- |
| withColumns\(\*\)\|\* | `select("withColumns()") | select("") = select("a, b, c, d, e")` | 所有列 |
| withColumns\(m to n\) | `select("withColumns(2 to 4)") = select("b, c, d")` | 从m到n的列 |
| withColumns\(m, n, k\) | `select("withColumns(1, 3, e)") = select("a, c, e")` | 第m，n，k列 |
| withColumns\(m, n to k\) | `select("withColumns(1, 3 to 5)") = select("a, c, d ,e")` | 上面两种表示的混合 |
| withoutColumns\(m to n\) | `select("withoutColumns(2 to 4)") = select("a, e")` | 取消选择从m到n的列 |
| withoutColumns\(m, n, k\) | `select("withoutColumns(1, 3, 5)") = select("b, d")` | 取消选择列m，n，k |
| withoutColumns\(m, n to k\) | `select("withoutColumns(1, 3 to 5)") = select("b")` | 上面两种表示的混合 |
{% endtab %}

{% tab title="Scala" %}
| Api | 用法 | 描述 |
| :--- | :--- | :--- |
| withColumns\(\*\)\|\* | _`select(withColumns('*)) | select(') = select('a,'b,'c,'d,'e)`_ | 所有列 |
| withColumns\(m to n\) | `select(withColumns(2 to 4)) = select('b,'c,'d)` | 从m到n的列 |
| withColumns\(m, n, k\) | `select(withColumns(1, 3, e)) = select('a,'c,'e)` | 第m，n，k列 |
| withColumns\(m, n to k\) | `select(withColumns(1,3 to 5)) = select('a,'c,'d,'e)` | 上面两种表示的混合 |
| withoutColumns\(m to n\) | `select(withoutColumns(2 to 4)) = select('a,'e)` | 取消选择从m到n的列 |
| withoutColumns\(m, n, k\) | `select(withoutColumns(1, 3, 5)) = select('b,'d)` | 取消选择列m，n，k |
| withoutColumns\(m, n to k\) | `select(withoutColumns(1, 3 to 5)) = select('b)` | 上面两种表示的混合 |
{% endtab %}

{% tab title="Python" %}
| pi | 用法 | 描述 |
| :--- | :--- | :--- |
| withColumns\(\*\)\|\* | `select("withColumns()") | select("") = select("a, b, c, d, e")` | 所有列 |
| withColumns\(m to n\) | `select("withColumns(2 to 4)") = select("b, c, d")` | 从m到n的列 |
| withColumns\(m, n, k\) | `select("withColumns(1, 3, e)") = select("a, c, e")` | 第m，n，k列 |
| withColumns\(m, n to k\) | `select("withColumns(1, 3 to 5)") = select("a, c, d ,e")` | 上面两种表示的混合 |
| withoutColumns\(m to n\) | `select("withoutColumns(2 to 4)") = select("a, e")` | 取消选择从m到n的列 |
| withoutColumns\(m, n, k\) | `select("withoutColumns(1, 3, 5)") = select("b, d")` | 取消选择列m，n，k |
| withoutColumns\(m, n to k\) | `select("withoutColumns(1, 3 to 5)") = select("b")` | 上面两种表示的混合 |
{% endtab %}
{% endtabs %}

列函数可以在所有需要列字段的地方使用，`select, groupBy, orderBy, UDFs etc.`例如：

{% tabs %}
{% tab title="Java" %}
```java
table
   .groupBy("withColumns(1 to 3)")
   .select("withColumns(a to b), myUDAgg(myUDF(withColumns(5 to 20)))")
```
{% endtab %}

{% tab title="Scala" %}
```scala
table
   .groupBy(withColumns(1 to 3))
   .select(withColumns('a to 'b), myUDAgg(myUDF(withColumns(5 to 20))))
```
{% endtab %}

{% tab title="Python" %}
```python
table \
    .group_by("withColumns(1 to 3)") \
    .select("withColumns(a to b), myUDAgg(myUDF(withColumns(5 to 20)))")
```
{% endtab %}
{% endtabs %}

{% hint style="info" %}
注意：列函数仅在Table API中使用。
{% endhint %}

