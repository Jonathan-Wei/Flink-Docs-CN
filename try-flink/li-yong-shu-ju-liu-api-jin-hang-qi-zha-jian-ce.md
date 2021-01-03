---
description: >-
  Apache Flink提供了一个DataStream
  API，用于构建健壮的、有状态的流应用程序。它提供了对状态和时间的细粒度控制，从而允许实现高级事件驱动系统。在这个分步指导中，您将学习如何使用Flink的DataStream
  API构建一个有状态流应用程序。
---

# 利用数据流API进行欺诈检测

## 你在构建什么?

在数字时代，信用卡诈骗越来越令人担忧。犯罪分子通过行骗或入侵不安全的系统来窃取信用卡号码。通过一次或多次小的购买来测试被盗号码，通常是一美元或更少。如果这可行的话，他们就会购买更重要的物品来出售或保留自己的物品。

在本教程中，将构建一个欺诈检测系统，用于对可疑的信用卡交易发出警报。通过使用一组简单的规则，你将看到Flink如何允许我们实现高级业务逻辑和实时操作。

## 先决条件

本演练假设你熟悉Java或Scala，但是即使你来自不同的编程语言，你也应该能够跟上。

## 救命，我卡住了！

 如果遇到困难，请查看[社区支持资源](https://flink.apache.org/gettinghelp.html)。特别是，Apache Flink的[用户邮件列表](https://flink.apache.org/community.html#mailing-lists)一直被评为所有Apache项目中最活跃的[邮件列表](https://flink.apache.org/community.html#mailing-lists)之一，并且是快速获得帮助的好方法。

## 如何跟进

如果你想跟随着一起实现这个程序，你需要在电脑上安装以下组件:

* Java 8 or 11
* Maven

提供的Flink Maven原型将快速创建包含所有必要依赖项的框架项目，因此你只需要专注于填充业务逻辑。这些依赖项包括`Flink -stream -java`，它是所有Flink流应用程序的核心依赖项;还有`Flink -walkthrough-common`，它具有特定于此演练的数据生成器和其他类。

{% hint style="info" %}
**注意**:为简洁起见，本演练中的每个代码块可能不包含周围的完整类。完整的代码可以在页面的底部找到。
{% endhint %}

{% tabs %}
{% tab title="Java" %}
```text
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-java \
    -DarchetypeVersion=1.11.0 \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```
{% endtab %}

{% tab title="Scala" %}
```text
$ mvn archetype:generate \
    -DarchetypeGroupId=org.apache.flink \
    -DarchetypeArtifactId=flink-walkthrough-datastream-scala \
    -DarchetypeVersion=1.11.0 \
    -DgroupId=frauddetection \
    -DartifactId=frauddetection \
    -Dversion=0.1 \
    -Dpackage=spendreport \
    -DinteractiveMode=false
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Java" %}
**FraudDetectionJob.java**

```java
package spendreport;

import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.walkthrough.common.sink.AlertSink;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;
import org.apache.flink.walkthrough.common.source.TransactionSource;

public class FraudDetectionJob {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

        DataStream<Transaction> transactions = env
            .addSource(new TransactionSource())
            .name("transactions");
        
        DataStream<Alert> alerts = transactions
            .keyBy(Transaction::getAccountId)
            .process(new FraudDetector())
            .name("fraud-detector");

        alerts
            .addSink(new AlertSink())
            .name("send-alerts");

        env.execute("Fraud Detection");
    }
}
```

**FraudDetector.java**

```java
package spendreport;

import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;

public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final long serialVersionUID = 1L;

    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {

        Alert alert = new Alert();
        alert.setId(transaction.getAccountId());

        collector.collect(alert);
    }
}
```
{% endtab %}

{% tab title="Scala" %}
**FraudDetectionJob.scala**

```scala
package spendreport

import org.apache.flink.streaming.api.scala._
import org.apache.flink.walkthrough.common.sink.AlertSink
import org.apache.flink.walkthrough.common.entity.Alert
import org.apache.flink.walkthrough.common.entity.Transaction
import org.apache.flink.walkthrough.common.source.TransactionSource

object FraudDetectionJob {

  @throws[Exception]
  def main(args: Array[String]): Unit = {
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment

    val transactions: DataStream[Transaction] = env
      .addSource(new TransactionSource)
      .name("transactions")

    val alerts: DataStream[Alert] = transactions
      .keyBy(transaction => transaction.getAccountId)
      .process(new FraudDetector)
      .name("fraud-detector")

    alerts
      .addSink(new AlertSink)
      .name("send-alerts")

    env.execute("Fraud Detection")
  }
}
```

**FraudDetector.scala**

```scala
package spendreport

import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.util.Collector
import org.apache.flink.walkthrough.common.entity.Alert
import org.apache.flink.walkthrough.common.entity.Transaction

object FraudDetector {
  val SMALL_AMOUNT: Double = 1.00
  val LARGE_AMOUNT: Double = 500.00
  val ONE_MINUTE: Long     = 60 * 1000L
}

@SerialVersionUID(1L)
class FraudDetector extends KeyedProcessFunction[Long, Transaction, Alert] {

  @throws[Exception]
  def processElement(
      transaction: Transaction,
      context: KeyedProcessFunction[Long, Transaction, Alert]#Context,
      collector: Collector[Alert]): Unit = {

    val alert = new Alert
    alert.setId(transaction.getAccountId)

    collector.collect(alert)
  }
}
```
{% endtab %}
{% endtabs %}



## 破解代码

### 执行环境

{% tabs %}
{% tab title="Java" %}
```java
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```
{% endtab %}

{% tab title="Scala" %}
```scala
val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
```
{% endtab %}
{% endtabs %}

### 创建Source

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Transaction> transactions = env
    .addSource(new TransactionSource())
    .name("transactions");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val transactions: DataStream[Transaction] = env
  .addSource(new TransactionSource)
  .name("transactions")
```
{% endtab %}
{% endtabs %}

### 划分事件和检测欺诈

{% tabs %}
{% tab title="Java" %}
```java
DataStream<Alert> alerts = transactions
    .keyBy(Transaction::getAccountId)
    .process(new FraudDetector())
    .name("fraud-detector");
```
{% endtab %}

{% tab title="Scala" %}
```scala
val alerts: DataStream[Alert] = transactions
  .keyBy(transaction => transaction.getAccountId)
  .process(new FraudDetector)
  .name("fraud-detector")
```
{% endtab %}
{% endtabs %}

### 输出结果

{% tabs %}
{% tab title="Java" %}
```java
alerts.addSink(new AlertSink());
```
{% endtab %}

{% tab title="Scala" %}
```scala
alerts.addSink(new AlertSink)
```
{% endtab %}
{% endtabs %}

### 执行Job

{% tabs %}
{% tab title="Java" %}
```java
env.execute("Fraud Detection");
```
{% endtab %}

{% tab title="Scala" %}
```scala
env.execute("Fraud Detection")
```
{% endtab %}
{% endtabs %}

### 欺诈探测器

{% tabs %}
{% tab title="Java" %}
```java
public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {
  
        Alert alert = new Alert();
        alert.setId(transaction.getAccountId());

        collector.collect(alert);
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
object FraudDetector {
  val SMALL_AMOUNT: Double = 1.00
  val LARGE_AMOUNT: Double = 500.00
  val ONE_MINUTE: Long     = 60 * 1000L
}

@SerialVersionUID(1L)
class FraudDetector extends KeyedProcessFunction[Long, Transaction, Alert] {

  @throws[Exception]
  def processElement(
      transaction: Transaction,
      context: KeyedProcessFunction[Long, Transaction, Alert]#Context,
      collector: Collector[Alert]): Unit = {

    val alert = new Alert
    alert.setId(transaction.getAccountId)

    collector.collect(alert)
  }
}
```
{% endtab %}
{% endtabs %}

## 编写一个真正的应用程序\(v1\)

![](../.gitbook/assets/fraud-transactions.svg)

## 欺诈探测器v2:状态+时间=❤️

## 完整应用

{% tabs %}
{% tab title="Java" %}
```java
package spendreport;

import org.apache.flink.api.common.state.ValueState;
import org.apache.flink.api.common.state.ValueStateDescriptor;
import org.apache.flink.api.common.typeinfo.Types;
import org.apache.flink.configuration.Configuration;
import org.apache.flink.streaming.api.functions.KeyedProcessFunction;
import org.apache.flink.util.Collector;
import org.apache.flink.walkthrough.common.entity.Alert;
import org.apache.flink.walkthrough.common.entity.Transaction;

public class FraudDetector extends KeyedProcessFunction<Long, Transaction, Alert> {

    private static final long serialVersionUID = 1L;

    private static final double SMALL_AMOUNT = 1.00;
    private static final double LARGE_AMOUNT = 500.00;
    private static final long ONE_MINUTE = 60 * 1000;

    private transient ValueState<Boolean> flagState;
    private transient ValueState<Long> timerState;

    @Override
    public void open(Configuration parameters) {
        ValueStateDescriptor<Boolean> flagDescriptor = new ValueStateDescriptor<>(
                "flag",
                Types.BOOLEAN);
        flagState = getRuntimeContext().getState(flagDescriptor);

        ValueStateDescriptor<Long> timerDescriptor = new ValueStateDescriptor<>(
                "timer-state",
                Types.LONG);
        timerState = getRuntimeContext().getState(timerDescriptor);
    }

    @Override
    public void processElement(
            Transaction transaction,
            Context context,
            Collector<Alert> collector) throws Exception {

        // Get the current state for the current key
        Boolean lastTransactionWasSmall = flagState.value();

        // Check if the flag is set
        if (lastTransactionWasSmall != null) {
            if (transaction.getAmount() > LARGE_AMOUNT) {
                //Output an alert downstream
                Alert alert = new Alert();
                alert.setId(transaction.getAccountId());

                collector.collect(alert);
            }
            // Clean up our state
            cleanUp(context);
        }

        if (transaction.getAmount() < SMALL_AMOUNT) {
            // set the flag to true
            flagState.update(true);

            long timer = context.timerService().currentProcessingTime() + ONE_MINUTE;
            context.timerService().registerProcessingTimeTimer(timer);

            timerState.update(timer);
        }
    }

    @Override
    public void onTimer(long timestamp, OnTimerContext ctx, Collector<Alert> out) {
        // remove flag after 1 minute
        timerState.clear();
        flagState.clear();
    }

    private void cleanUp(Context ctx) throws Exception {
        // delete timer
        Long timer = timerState.value();
        ctx.timerService().deleteProcessingTimeTimer(timer);

        // clean up all state
        timerState.clear();
        flagState.clear();
    }
}
```
{% endtab %}

{% tab title="Scala" %}
```scala
package spendreport

import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.api.scala.typeutils.Types
import org.apache.flink.configuration.Configuration
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.util.Collector
import org.apache.flink.walkthrough.common.entity.Alert
import org.apache.flink.walkthrough.common.entity.Transaction

object FraudDetector {
  val SMALL_AMOUNT: Double = 1.00
  val LARGE_AMOUNT: Double = 500.00
  val ONE_MINUTE: Long     = 60 * 1000L
}

@SerialVersionUID(1L)
class FraudDetector extends KeyedProcessFunction[Long, Transaction, Alert] {

  @transient private var flagState: ValueState[java.lang.Boolean] = _
  @transient private var timerState: ValueState[java.lang.Long] = _

  @throws[Exception]
  override def open(parameters: Configuration): Unit = {
    val flagDescriptor = new ValueStateDescriptor("flag", Types.BOOLEAN)
    flagState = getRuntimeContext.getState(flagDescriptor)

    val timerDescriptor = new ValueStateDescriptor("timer-state", Types.LONG)
    timerState = getRuntimeContext.getState(timerDescriptor)
  }

  override def processElement(
      transaction: Transaction,
      context: KeyedProcessFunction[Long, Transaction, Alert]#Context,
      collector: Collector[Alert]): Unit = {

    // Get the current state for the current key
    val lastTransactionWasSmall = flagState.value

    // Check if the flag is set
    if (lastTransactionWasSmall != null) {
      if (transaction.getAmount > FraudDetector.LARGE_AMOUNT) {
        // Output an alert downstream
        val alert = new Alert
        alert.setId(transaction.getAccountId)

        collector.collect(alert)
      }
      // Clean up our state
      cleanUp(context)
    }

    if (transaction.getAmount < FraudDetector.SMALL_AMOUNT) {
      // set the flag to true
      flagState.update(true)
      val timer = context.timerService.currentProcessingTime + FraudDetector.ONE_MINUTE

      context.timerService.registerProcessingTimeTimer(timer)
      timerState.update(timer)
    }
  }

  override def onTimer(
      timestamp: Long,
      ctx: KeyedProcessFunction[Long, Transaction, Alert]#OnTimerContext,
      out: Collector[Alert]): Unit = {
    // remove flag after 1 minute
    timerState.clear()
    flagState.clear()
  }

  @throws[Exception]
  private def cleanUp(ctx: KeyedProcessFunction[Long, Transaction, Alert]#Context): Unit = {
    // delete timer
    val timer = timerState.value
    ctx.timerService.deleteProcessingTimeTimer(timer)

    // clean up all states
    timerState.clear()
    flagState.clear()
  }
}
```
{% endtab %}
{% endtabs %}

### 预期输出

使用提供的TransactionSource运行此代码将为帐户3发出欺诈警报。你应该看到以下输出在你的任务管理器日志:

```text
2019-08-19 14:22:06,220 INFO  org.apache.flink.walkthrough.common.sink.AlertSink            - Alert{id=3}
2019-08-19 14:22:11,383 INFO  org.apache.flink.walkthrough.common.sink.AlertSink            - Alert{id=3}
2019-08-19 14:22:16,551 INFO  org.apache.flink.walkthrough.common.sink.AlertSink            - Alert{id=3}
2019-08-19 14:22:21,723 INFO  org.apache.flink.walkthrough.common.sink.AlertSink            - Alert{id=3}
2019-08-19 14:22:26,896 INFO  org.apache.flink.walkthrough.common.sink.AlertSink            - Alert{id=3}
```

