# 流分析

## 事件时间和Watermarks

### 介绍

Flink明确支持三种不同的时间概念：

* _事件时间\(_ _event time\)：_事件发生的时间，由产生（或存储）事件的设备记录的时间
* _抽取时间\(_ _ingestion time\)：_ Flink抽取事件时记录的时间戳记
* _处理时间\(_ _processing time\)：_管道中特定运算符处理事件的时间

### 使用事件时间

```text
final StreamExecutionEnvironment env =
    StreamExecutionEnvironment.getExecutionEnvironment();
env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime);
```

### Watermarks



···23 19 22 24 21 14 17 13 12 15 9 11 7 2 4→

### 延迟和完整性

### 延迟

 延迟是相对于Watermarks定义的。`Watermark(t)`断言流在时间_t之前_已完成；时间戳≤t的水印后的任何事件都是迟到的。

### 使用Watermarks

```text
DataStream<Event> stream = ...

WatermarkStrategy<Event> strategy = WatermarkStrategy
        .<Event>forBoundedOutOfOrderness(Duration.ofSeconds(20))
        .withTimestampAssigner((event, timestamp) -> event.timestamp);

DataStream<Event> withTimestampsAndWatermarks =
    stream.assignTimestampsAndWatermarks(strategy);
```

## 窗口（Windows）

Flink具有非常有表现力的窗口语义。

在本节中，你将学习：

* 如何使用窗口来计算无边界流上的聚合，
* Flink支持哪些类型的Windows，以及
* 如何使用窗口聚合实现数据流程序

### 介绍

在进行流处理时，自然要对流的有限子集计算聚合分析以回答如下问题：

* 每分钟的页面浏览量
* 每个用户每周的会话数
* 每分钟每传感器的最高温度



```text
stream.
    .keyBy(<key selector>)
    .window(<window assigner>)    
    .reduce|aggregate|process(<window function>)
```



```text
stream.
    .windowAll(<window assigner>)
    .reduce|aggregate|process(<window function>)
```

### 窗口分配器（Window Assigners）

![](../.gitbook/assets/image%20%2858%29.png)

窗口函数



```text
DataStream<SensorReading> input = ...

input
    .keyBy(x -> x.key)
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .process(new MyWastefulMax());

public static class MyWastefulMax extends ProcessWindowFunction<
        SensorReading,                  // input type
        Tuple3<String, Long, Integer>,  // output type
        String,                         // key type
        TimeWindow> {                   // window type
    
    @Override
    public void process(
            String key,
            Context context, 
            Iterable<SensorReading> events,
            Collector<Tuple3<String, Long, Integer>> out) {

        int max = 0;
        for (SensorReading event : events) {
            max = Math.max(event.value, max);
        }
        out.collect(Tuple3.of(key, context.window().getEnd(), max));
    }
}
```



```text
public abstract class Context implements java.io.Serializable {
    public abstract W window();
    
    public abstract long currentProcessingTime();
    public abstract long currentWatermark();

    public abstract KeyedStateStore windowState();
    public abstract KeyedStateStore globalState();
}
```



```text
DataStream<SensorReading> input = ...

input
    .keyBy(x -> x.key)
    .window(TumblingEventTimeWindows.of(Time.minutes(1)))
    .reduce(new MyReducingMax(), new MyWindowFunction());

private static class MyReducingMax implements ReduceFunction<SensorReading> {
    public SensorReading reduce(SensorReading r1, SensorReading r2) {
        return r1.value() > r2.value() ? r1 : r2;
    }
}

private static class MyWindowFunction extends ProcessWindowFunction<
    SensorReading, Tuple3<String, Long, SensorReading>, String, TimeWindow> {

    @Override
    public void process(
            String key,
            Context context,
            Iterable<SensorReading> maxReading,
            Collector<Tuple3<String, Long, SensorReading>> out) {

        SensorReading max = maxReading.iterator().next();
        out.collect(Tuple3.of(key, context.window().getEnd(), max));
    }
}
```

### 延迟事件



```text
OutputTag<Event> lateTag = new OutputTag<Event>("late"){};

SingleOutputStreamOperator<Event> result = stream.
    .keyBy(...)
    .window(...)
    .sideOutputLateData(lateTag)
    .process(...);
  
DataStream<Event> lateStream = result.getSideOutput(lateTag);
```



```text
stream.
    .keyBy(...)
    .window(...)
    .allowedLateness(Time.seconds(10))
    .process(...);
```

### 惊喜

