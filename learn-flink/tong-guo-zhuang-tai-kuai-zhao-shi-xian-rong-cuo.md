# 通过状态快照实现容错

## 状态后端

<table>
  <thead>
    <tr>
      <th style="text-align:left">&#x540D;&#x79F0;</th>
      <th style="text-align:left">&#x5DE5;&#x4F5C;&#x72B6;&#x6001;</th>
      <th style="text-align:left">&#x72B6;&#x6001;&#x5907;&#x4EFD;</th>
      <th style="text-align:left">&#x5FEB;&#x7167;</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">RocksDBStateBackend</td>
      <td style="text-align:left">&#x672C;&#x5730;&#x78C1;&#x76D8;&#xFF08;tmp&#x76EE;&#x5F55;&#xFF09;</td>
      <td
      style="text-align:left">&#x5206;&#x5E03;&#x5F0F;&#x6587;&#x4EF6;&#x7CFB;&#x7EDF;</td>
        <td style="text-align:left">&#x5168;/&#x589E;&#x91CF;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <ul>
          <li>&#x652F;&#x6301;&#x5927;&#x4E8E;&#x53EF;&#x7528;&#x5185;&#x5B58;&#x7684;&#x72B6;&#x6001;</li>
          <li>&#x7ECF;&#x9A8C;&#x6CD5;&#x5219;&#xFF1A;&#x6BD4;&#x57FA;&#x4E8E;&#x5806;&#x7684;&#x540E;&#x7AEF;&#x6162;10&#x500D;</li>
        </ul>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">FsStateBackend</td>
      <td style="text-align:left">JVM&#x5806;</td>
      <td style="text-align:left">&#x5206;&#x5E03;&#x5F0F;&#x6587;&#x4EF6;&#x7CFB;&#x7EDF;</td>
      <td style="text-align:left">&#x5168;&#x91CF;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <ul>
          <li>&#x5FEB;&#x901F;&#xFF0C;&#x9700;&#x8981;&#x5927;&#x5806;</li>
          <li>&#x53D7;&#x5236;&#x4E8E;GC</li>
        </ul>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
    <tr>
      <td style="text-align:left">MemoryStateBackend</td>
      <td style="text-align:left">JVM&#x5806;</td>
      <td style="text-align:left">JobManager JVM&#x5806;</td>
      <td style="text-align:left">&#x5168;&#x91CF;</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <ul>
          <li>&#x9002;&#x7528;&#x4E8E;&#x5C0F;&#x72B6;&#x6001;&#xFF08;&#x672C;&#x5730;&#xFF09;&#x7684;&#x6D4B;&#x8BD5;&#x548C;&#x5B9E;&#x9A8C;</li>
        </ul>
      </td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
      <td style="text-align:left"></td>
    </tr>
  </tbody>
</table>

## 状态快照

### 定义

### 状态快照如何工作？

### 一次保证

如果在流处理应用程序中出现问题，则可能会丢失或重复结果。使用Flink，根据你为应用程序做出的选择以及运行它的集群，可能会产生以下任何结果：

* Flink毫不费力地从故障中恢复（_最多一次_）
* 没有任何损失，但是您可能会遇到重复的结果（_至少一次_）
* 没有任何损失或重复（_仅一次_）

### 端到端恰好一次

为了精确地端到端恰好一次，以便源中的每个事件都仅一次影响接收器，必须满足以下条件：

1. Source必须可重播
2. Sink必须是事务性的（或幂等的）

## 

