# 项目配置

每个Flink应用程序都依赖于一组Flink库。至少，该应用程序取决于Flink API。此外，许多应用程序还依赖于某些连接器库（例如Kafka，Cassandra等）。当运行Flink应用程序时（在分布式部署中或在IDE中进行测试），Flink运行时库也必须可用。

## Flink核心和应用程序依赖性

## 项目配置：基本依赖性

## Scala版本

## Hadoop依赖关系

## Maven快速入门

### **要求**

唯一的要求是需要**Maven 3.0.4**（或更高版本）和**Java8.x**。

### 创建项目

 使用以下命令之一**创建项目**：

{% tabs %}
{% tab title="使用maven原型" %}
```bash
$ mvn archetype:generate                               \
  -DarchetypeGroupId=org.apache.flink              \
  -DarchetypeArtifactId=flink-quickstart-java      \
  -DarchetypeVersion=1.11.2
```
{% endtab %}

{% tab title="运行quickstart脚本" %}
```bash
$ curl https://flink.apache.org/q/quickstart.sh | bash -s 1.11.2
```
{% endtab %}
{% endtabs %}

### 构建项目

## Gradle

### 要求

 唯一的要求是可以使用**Gradle 3.x**（或更高版本）和**Java 8.x**安装。

### 创建项目

 使用以下命令之一**创建项目**：

{% tabs %}
{% tab title="Gradle示例" %}
```text
### build.gradle

buildscript {
    repositories {
        jcenter() // this applies only to the Gradle 'Shadow' plugin
    }
    dependencies {
        classpath 'com.github.jengelman.gradle.plugins:shadow:2.0.4'
    }
}

plugins {
    id 'java'
    id 'application'
    // shadow plugin to produce fat JARs
    id 'com.github.johnrengelman.shadow' version '2.0.4'
}


// artifact properties
group = 'org.myorg.quickstart'
version = '0.1-SNAPSHOT'
mainClassName = 'org.myorg.quickstart.StreamingJob'
description = """Flink Quickstart Job"""

ext {
    javaVersion = '1.8'
    flinkVersion = '1.11.2'
    scalaBinaryVersion = '2.11'
    slf4jVersion = '1.7.15'
    log4jVersion = '2.12.1'
}


sourceCompatibility = javaVersion
targetCompatibility = javaVersion
tasks.withType(JavaCompile) {
	options.encoding = 'UTF-8'
}

applicationDefaultJvmArgs = ["-Dlog4j.configurationFile=log4j2.properties"]

task wrapper(type: Wrapper) {
    gradleVersion = '3.1'
}

// declare where to find the dependencies of your project
repositories {
    mavenCentral()
    maven { url "https://repository.apache.org/content/repositories/snapshots/" }
}

// NOTE: We cannot use "compileOnly" or "shadow" configurations since then we could not run code
// in the IDE or with "gradle run". We also cannot exclude transitive dependencies from the
// shadowJar yet (see https://github.com/johnrengelman/shadow/issues/159).
// -> Explicitly define the // libraries we want to be included in the "flinkShadowJar" configuration!
configurations {
    flinkShadowJar // dependencies which go into the shadowJar

    // always exclude these (also from transitive dependencies) since they are provided by Flink
    flinkShadowJar.exclude group: 'org.apache.flink', module: 'force-shading'
    flinkShadowJar.exclude group: 'com.google.code.findbugs', module: 'jsr305'
    flinkShadowJar.exclude group: 'org.slf4j'
    flinkShadowJar.exclude group: 'org.apache.logging.log4j'
}

// declare the dependencies for your production and test code
dependencies {
    // --------------------------------------------------------------
    // Compile-time dependencies that should NOT be part of the
    // shadow jar and are provided in the lib folder of Flink
    // --------------------------------------------------------------
    compile "org.apache.flink:flink-streaming-java_${scalaBinaryVersion}:${flinkVersion}"

    // --------------------------------------------------------------
    // Dependencies that should be part of the shadow jar, e.g.
    // connectors. These must be in the flinkShadowJar configuration!
    // --------------------------------------------------------------
    //flinkShadowJar "org.apache.flink:flink-connector-kafka-0.11_${scalaBinaryVersion}:${flinkVersion}"

    compile "org.apache.logging.log4j:log4j-api:${log4jVersion}"
    compile "org.apache.logging.log4j:log4j-core:${log4jVersion}"
    compile "org.apache.logging.log4j:log4j-slf4j-impl:${log4jVersion}"
    compile "org.slf4j:slf4j-log4j12:${slf4jVersion}"

    // Add test dependencies here.
    // testCompile "junit:junit:4.12"
}

// make compileOnly dependencies available for tests:
sourceSets {
    main.compileClasspath += configurations.flinkShadowJar
    main.runtimeClasspath += configurations.flinkShadowJar

    test.compileClasspath += configurations.flinkShadowJar
    test.runtimeClasspath += configurations.flinkShadowJar

    javadoc.classpath += configurations.flinkShadowJar
}

run.classpath = sourceSets.main.runtimeClasspath

jar {
    manifest {
        attributes 'Built-By': System.getProperty('user.name'),
                'Build-Jdk': System.getProperty('java.version')
    }
}

shadowJar {
    configurations = [project.configurations.flinkShadowJar]
}
```

```text
### settings.gradle

rootProject.name = 'quickstart'
```
{% endtab %}

{% tab title="运行quickstart脚本" %}

{% endtab %}
{% endtabs %}

## SBT

### 项目创建

可以通过以下两种方法之一来创建新项目：

{% tabs %}
{% tab title="使用SBT模板" %}
```text
$ sbt new tillrohrmann/flink-project.g8
```

这里会提示你输入几个参数（项目名称，flink版本...），然后从[flink-project模板](https://github.com/tillrohrmann/flink-project.g8)创建一个Flink项目。需要sbt&gt; = 0.13.13才能执行此命令。如有必要，可以按照本[安装指南](http://www.scala-sbt.org/download.html)进行获取。
{% endtab %}

{% tab title="使用quickstart脚本" %}
```text
$ bash <(curl https://flink.apache.org/q/sbt-quickstart.sh)
```

 这将在**指定的**项目目录中创建一个Flink项目。
{% endtab %}
{% endtabs %}

### 构建项目

 构建项目，需要执行`sbt clean assembly`命令。这将在**target / scala\_your-major-scala-version /**目录中创建fat-jar **your-project-name-assembly-0.1-SNAPSHOT.jar**。

### **运行项目**

为了运行你的项目，必须执行`sbt run`命令。

默认情况下，这将在sbt运行的相同JVM中运行您的作业。。为了在不同的JVM中运行您的作业，将以下行添加到`build.sbt`

```text
fork in run := true
```

### **IntelliJ**

官方建议使用[IntelliJ](https://www.jetbrains.com/idea/) 来进行Flink作业开发。首先，必须将新创建的项目导入IntelliJ。你可以通过`File -> New -> Project from Existing Sources...`选择项目目录来执行此操作。然后，IntelliJ将自动检测`build.sbt`文件并进行所有设置。

为了运行Flink作业，建议选择`mainRunner`模块作为**Run / Debug Configuration**的类路径。这将确保设置为_提供的_所有依赖项在执行时均可用。你可以通过`Run -> Edit Configurations...`来配置**运行/调试配置**，然后然后从module dropbox的使用类路径中选择mainRunner。。

### **Eclipse**

 为了将新创建的项目导入[Eclipse中](https://eclipse.org/)，首先必须为其创建Eclipse项目文件。这些项目文件可以通过[sbteclipse](https://github.com/typesafehub/sbteclipse)插件创建。将以下行添加到你的`PROJECT_DIR/project/plugins.sbt`文件中：

```text
addSbtPlugin("com.typesafe.sbteclipse" % "sbteclipse-plugin" % "4.0.0")
```

 在`sbt`使用以下命令创建Eclipse项目文件

```text
> eclipse
```

 现在，可以通过将项目导入Eclipse `File -> Import... -> Existing Projects into Workspace`，然后选择项目目录。

## 附录：用于构建具有依赖关系的Jar的模板

要构建包含声明的连接器和库所需的所有依赖关系的应用程序JAR，可以使用以下maven插件定义：

```markup
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <artifactSet>
                            <excludes>
                                <exclude>com.google.code.findbugs:jsr305</exclude>
                                <exclude>org.slf4j:*</exclude>
                                <exclude>log4j:*</exclude>
                            </excludes>
                        </artifactSet>
                        <filters>
                            <filter>
                                <!-- Do not copy the signatures in the META-INF folder.
                                Otherwise, this might cause SecurityExceptions when using the JAR. -->
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                        <transformers>
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>my.programs.main.clazz</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

