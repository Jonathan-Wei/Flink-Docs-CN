# 日志

Flink中的日志记录是使用slf4j日志记录界面实现的。作为底层日志记录框架，使用log4j。我们还提供了logback配置文件，并将它们作为属性传递给JVM。愿意使用logback而不是log4j的用户可以只排除log4j（或从lib /文件夹中删除它）。

## 配置Log4j

使用属性文件控制Log4j。在Flink的情况下，通常会调用该文件`log4j.properties`。我们使用`-Dlog4j.configuration=`参数将此文件的文件名和位置传递给JVM。

Flink附带以下默认属性文件：

* `log4j-cli.properties`：由Flink命令行客户端（例如`flink run`）使用（不是在集群上执行的代码）
* `log4j-yarn-session.properties`：启动YARN会话时由Flink命令行客户端使用（`yarn-session.sh`）
* `log4j.properties`：JobManager / Taskmanager日志（独立和YARN）

## 配置logback

对于用户和开发人员来说，控制日志框架非常重要。日志记录框架的配置仅由配置文件完成。配置文件必须通过设置环境属性-Dlogback.configurationFile=或通过将logback.xml放入类路径来指定。conf目录包含一个logback.xml文件，该文件可以被修改，如果Flink是在IDE之外启动的，并且带有提供的启动脚本，则使用该文件。提供的logback.xml具有以下形式：

```markup
<configuration>
    <appender name="file" class="ch.qos.logback.core.FileAppender">
        <file>${log.file}</file>
        <append>false</append>
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{60} %X{sourceThread} - %msg%n</pattern>
        </encoder>
    </appender>

    <root level="INFO">
        <appender-ref ref="file"/>
    </root>
</configuration>
```

例如，为了控制org.apache.flink.runtime.job..JobGraph的日志记录级别，必须向配置文件添加以下行。

```text
<logger name="org.apache.flink.runtime.jobgraph.JobGraph" level="DEBUG"/>
```

有关配置logback的更多信息，请参阅[logback手册](http://logback.qos.ch/manual/configuration.html)。

## 开发人员的最佳实践

使用slf4j的日志记录器通过调用创建

```text
import org.slf4j.LoggerFactory
import org.slf4j.Logger

Logger LOG = LoggerFactory.getLogger(Foobar.class)
```

为了从slf4j中获益最多，建议使用其占位符机制。使用占位符可以避免不必要的字符串构造，以防日志记录级别设置得太高，以至于不会记录消息。占位符的语法如下：

```text
LOG.info("This message contains {} placeholders. {}", 2, "Yippie");
```

占位符还可以与需要记录的异常一起使用。

```text
catch(Exception exception){
	LOG.error("An {} occurred.", "error", exception);
}
```

