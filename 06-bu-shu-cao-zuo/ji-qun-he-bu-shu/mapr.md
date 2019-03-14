# MAPR

本文档提供了有关如何在[MapR](https://mapr.com/)群集上为YARN执行准备Flink的说明。

## 使用MapR在YARN上运行Flink

以下说明假定MapR版本5.2.0。它们将指导你能够开始[在YARN](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/yarn_setup.html) 作业[上](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/yarn_setup.html)提交[Flink](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/deployment/yarn_setup.html)或向MapR群集提交会话。

### 为MapR构建Flink

为了在MapR上运行Flink，需要使用MapR自己的Hadoop和Zookeeper分发构建Flink。只需使用Maven使用项目根目录中的以下命令构建Flink：

```text
mvn clean install -DskipTests -Pvendor-repos,mapr \
    -Dhadoop.version=2.7.0-mapr-1607 \
    -Dzookeeper.version=3.4.5-mapr-1604
```

vendor-repos构建配置文件将MapR的存储库添加到构建中，这样就可以获取MapR的Hadoop / Zookeeper依赖项。mapr构建配置文件还解决了mapr和Flink之间的一些依赖冲突，并确保使用集群节点上的本地mapr库。必须激活这两个配置文件。

默认情况下，`mapr`配置文件使用MapR版本5.2.0的Hadoop / Zookeeper依赖关系构建，因此无需显式覆盖`hadoop.version`和`zookeeper.version`属性。对于不同的MapR版本，只需将这些属性覆盖为适当的值即可。每个MapR版本的相应Hadoop / Zookeeper发行版都可以在MapR文档中找到，例如 [此处](http://maprdocs.mapr.com/home/DevelopmentGuide/MavenArtifacts.html)。

### 作业提交客户端设置

向MapR提交Flink作业的客户还需要使用以下设置进行准备。

确保配置MapR的JAAS配置文件以避免登录失败：

```text
export JVM_ARGS=-Djava.security.auth.login.config=/opt/mapr/conf/mapr.login.conf
```

确保`yarn-site.xml`中`yarn.nodemanager.resource.cpu-vcores`属性设置：

```text
<!-- in /opt/mapr/hadoop/hadoop-2.7.0/etc/hadoop/yarn-site.xml -->

<configuration>
...

<property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>...</value>
</property>

...
</configuration>
```

`yarn-site.xml`文件中的`YARN_CONF_DIR`或`HADOOP_CONF_DIR`环境变量设置为所在的路径：

```text
export YARN_CONF_DIR=/opt/mapr/hadoop/hadoop-2.7.0/etc/hadoop/
export HADOOP_CONF_DIR=/opt/mapr/hadoop/hadoop-2.7.0/etc/hadoop/
```

确保在类路径中配置MapR本机库：

```text
export FLINK_CLASSPATH=/opt/mapr/lib/*
```

如果通过`yarn-session.sh`在YARN会话中启动Flink ，则还需要以下内容：

```text
export CC_CLASSPATH=/opt/mapr/lib/*
```

## 使用安全的MapR群集运行Flink

_注意：在Flink 1.2.0中，Flink用于YARN执行的Kerberos身份验证有一个错误，禁止它与MapR Security一起使用。请升级到更高版本的Flink版本，以便将Flink与安全的MapR群集一起使用。有关详细信息，请参阅_[_FLINK-5949_](https://issues.apache.org/jira/browse/FLINK-5949)_。_

Flink的[Kerberos身份验证](https://ci.apache.org/projects/flink/flink-docs-release-1.7/ops/security-kerberos.html)独立于 [MapR的安全身份验证](http://maprdocs.mapr.com/home/SecurityGuide/Configuring-MapR-Security.html)。通过上述构建过程和环境变量设置，Flink不需要任何其他配置即可使用MapR Security。

用户只需使用MapR的`maprlogin`身份验证实用程序登录即可。未获取MapR登录凭据的用户将无法提交Flink作业，并出现以下错误：

```text
java.lang.Exception: unable to establish the security context
Caused by: o.a.f.r.security.modules.SecurityModule$SecurityInstallException: Unable to set the Hadoop login user
Caused by: java.io.IOException: failure to login: Unable to obtain MapR credentials
```

