# 应用程序分析

## Apache Flink自定义日志的概述

每个独立的JobManager、TaskManager、HistoryServer和ZooKeeper守护进程将stdout和stderr重定向到具有.out文件名后缀的文件，并将内部日志写到具有.log后缀的文件。用户可以配置Java Opts\(包含`env.java.opts`，`env.java.opts.jobmanager`，`env.java.opts.taskmanager`，`env.java.opts.historyserver`），同样也可以使用脚本变量FLINK\_LOG\_PREFIX定义日志文件，并将选项括在双引号中，以便后期计算。使用FLINK\_LOG\_PREFIX的日志文件与默认的.out和. Log文件一起轮训。

## 使用Java Flight Recorder进行性能分析 <a id="profiling-with-java-flight-recorder"></a>

Java Flight Recorder是Oracle JDK中内置的分析和事件收集框架。 [Java Mission Control](http://www.oracle.com/technetwork/java/javaseproducts/mission-control/java-mission-control-1998576.html) 是一套先进的工具，可以对Java Flight Recorder收集的大量数据进行高效，详细的分析。配置示例：

```bash
env.java.opts: "-XX:+UnlockCommercialFeatures -XX:+UnlockDiagnosticVMOptions -XX:+FlightRecorder -XX:+DebugNonSafepoints -XX:FlightRecorderOptions=defaultrecording=true,dumponexit=true,dumponexitpath=${FLINK_LOG_PREFIX}.jfr"
```

## 使用JITWatch进行分析 <a id="profiling-with-jitwatch"></a>

[JITWatch](https://github.com/AdoptOpenJDK/jitwatch/wiki)是Java HotSpot JIT编译器的日志分析器和可视化工具，用于检查内联决策，热方法，字节码和汇编。配置示例：

```bash
env.java.opts: "-XX:+UnlockDiagnosticVMOptions -XX:+TraceClassLoading -XX:+LogCompilation -XX:LogFile=${FLINK_LOG_PREFIX}.jit -XX:+PrintAssembly"
```

