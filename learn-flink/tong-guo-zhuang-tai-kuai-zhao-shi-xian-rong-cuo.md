# 通过状态快照实现容错

## 状态后端

由Flink管理的键控状态是一种分片的键/值存储，键控状态的每个项目的工作副本都保存在负责该键的任务管理器本地。运算符状态也是需要它的机器的本地状态。Flink定期获取所有状态的持久快照，并将这些快照复制到更持久的位置，例如分布式文件系统。

发生故障时，Flink可以恢复应用程序的完整状态并恢复处理，就好像什么都没出错。

 Flink管理的_状态_存储在_状态后端中_。状态后端有两种实现方式-一种基于RocksDB，一种嵌入式键/值存储将其工作状态保留在磁盘上；另一种基于堆的状态后端将其工作状态保留在内存中，在Java堆上。这种基于堆的状态后端有两种形式：将状态快照持久保存到分布式文件系统的FsStateBackend，以及使用JobManager堆的MemoryStateBackend。

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

当使用保持在基于堆的状态后端的状态时，访问和更新涉及在堆上读写对象。 但是对于保存在`RocksDBStateBackend`对象中的，访问和更新涉及序列化和反序列化，因此代价更高。但是RocksDB可以拥有的状态数量仅受本地磁盘大小的限制。还要注意，只有 `RocksDBStateBackend`能够执行增量快照，这对于具有大量缓慢变化的状态的应用程序来说是一个很大的好处。

所有这些状态后端都可以执行异步快照，这意味着它们可以拍摄快照而不会妨碍正在进行的流处理。

## 状态快照

### 定义

* _快照_–通用术语，指的是Flink作业状态的全局一致图像。快照包括指向每个数据源的指针（例如，到文件或Kafka分区的偏移量），以及每个作业的有状态运算符的状态副本，该状态副本是由于处理了所有事件直至源中的那些位置。
* _CheckPoint_– Flink为能够从故障中恢复而自动拍摄的快照，CheckPoint可以是增量的，并且可以进行快速恢复的优化。
*  _Externalized Checkpoint（外部检查点）_–通常，检查点不旨在由用户操纵。Flink在作业运行时仅保留_n_个最近的检查点（_n可_配置），并在取消作业时将其删除。但是你可以将它们配置为保留，在这种情况下，您可以从中手动恢复。
* _SavePoint_–由用户手动触发的快照（或API调用），用于某些操作目的，例如有状态的重新部署/升级/缩放操作。保存点始终是完整的，并且针对操作灵活性进行了优化。

### 状态快照如何工作？

 Flink使用[Chandy-Lamport算法的](https://en.wikipedia.org/wiki/Chandy-Lamport_algorithm)一种变体，称为_异步屏障快照\(_ _asynchronous barrier snapshotting\)_。

 当检查点协调器（作业管理器的一部分）指示任务管理器开始检查点时，它会让所有源记录其偏移量并将编号的_检查点屏障_ 插入其流中。这些屏障通过作业图流动，指示每个检查点前后的流部分。

![](../.gitbook/assets/image%20%2857%29.png)

 检查点_n_将包含由于消耗了**检查点屏障**_**n**_**之前的每个事件**而导致的所有运算符的状态**，而在它之后没有任何事件**。

 当作业图中的每个操作员接收到其中一个屏障时，它会记录其状态。具有两个输入流（如CoProcessFunction）的运算符执行屏障对齐，这样快照将反映从两个输入流直到（但不超过）两个屏障的消费事件所产生的状态。

![](../.gitbook/assets/image%20%2856%29.png)

Flink的状态后端使用写时复制机制来允许在对旧版本的状态进行异步快照时不受阻碍地继续进行流处理。仅当快照已持久保存时，才会对这些状态的旧版本进行垃圾收集。

### 一次保证

如果在流处理应用程序中出现问题，则可能会丢失或重复结果。使用Flink，根据你为应用程序做出的选择以及运行它的集群，可能会产生以下任何结果：

* Flink毫不费力地从故障中恢复（_最多一次_）
* 没有任何损失，但是您可能会遇到重复的结果（_至少一次_）
* 没有任何损失或重复（_仅一次_）

假设Flink通过倒带和重放源数据流从故障中恢复，当理想情况被描述为精确一次时，这并不意味着每个事件都将被处理一次。相反，这意味着_每个事件将只影响一次由Flink管理的状态_。

屏障对齐只需要提供一次保证。如果不需要它，可以通过将Flink配置`CheckpointingMode.AT_LEAST_ONCE`为use来获得一些性能，这将禁用屏障对齐。  


### 端到端恰好一次

为了精确地端到端恰好一次，以便源中的每个事件都仅一次影响接收器，必须满足以下条件：

1. Source必须可重播
2. Sink必须是事务性的（或幂等的）

## 

