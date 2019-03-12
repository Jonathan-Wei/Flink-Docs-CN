# 将Flink导入IDE

以下部分描述了如何将Flink项目导入IDE以开发Flink本身。有关编写Flink程序的信息，请参阅[Java API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/projectsetup/java_api_quickstart.html) 和[Scala API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/projectsetup/scala_api_quickstart.html) 快速入门指南。

**注意：**每当IDE中的某些内容无效时，请首先尝试使用Maven命令行，`mvn clean package -DskipTests`因为它可能是您的IDE有错误或未正确设置。

## 准备

首先，请先从我们的某个[存储库中](https://flink.apache.org/community.html#source-code)获取Flink源代码 ，例如：

```text
git clone https://github.com/apache/flink.git
```

## IntelliJ IDEA

关于如何设置IntelliJ IDEA IDE以开发Flink核心的简要指南。众所周知，Eclipse存在混合Scala和Java项目的问题，越来越多的贡献者正在迁移到IntelliJ IDEA。

以下文档描述了使用Flink源设置IntelliJ IDEA 2016.2.5（[https://www.jetbrains.com/idea/download/](https://www.jetbrains.com/idea/download/)）的步骤。

### 安装Scala插件

IntelliJ安装设置提供安装Scala插件。如果未安装，请在导入Flink之前按照这些说明启用对Scala项目和文件的支持：

1. 转到IntelliJ插件设置（IntelliJ IDEA - &gt;首选项 - &gt;插件），然后单击“安装Jetbrains插件...”。
2. 选择并安装“Scala”插件。
3. 重启IntelliJ

### 导入Flink

1. 启动IntelliJ IDEA并选择“Import Project”
2. 选择Flink存储库的根文件夹
3. 选择“Import project from external model”并选择“Maven”
4. 保留默认选项并单击“Next”，直到您点击SDK部分。
5. 如果没有SDK，请创建一个左上角带有“+”符号的SDK，然后单击“JDK”，选择您的JDK主目录并单击“确定”。否则只需选择您的SDK。
6. 继续再次单击“下一步”并完成导入。
7. 右键单击导入的Flink project -&gt; Maven -&gt; Generate Sources 和 Update Folders。请注意，这将在您的本地Maven存储库中安装Flink库，即“/ home / _-your-user-_ /.m2/repository/org/apache/flink/”。或者，`mvn clean package -DskipTests`还可以为IDE创建必要的文件，但不安装库。
8. 构建项目（Build -&gt; Make Project）

### Java Checkstyle

IntelliJ使用Checkstyle-IDEA插件支持IDE中的checkstyle。

1. 从IntelliJ插件存储库安装“Checkstyle-IDEA”插件。
2. 通过转到Settings -&gt; Other Settings -&gt; Checkstyle来配置插件。
3. 将“Scan Scope” 设置为 “Only Java sources \(including tests\)”
4. 在“Checkstyle Version”下拉列表中选择_8.9_，然后单击“Apply”。**这一步很重要，不要跳过它！**
5. 在“配置文件”窗格中，使用加号图标添加新配置：
   1. 将“Description”设置为“Flink”。
   2. 选择“Use a local Checkstyle file”，并将其指向 `"tools/maven/checkstyle.xml"`存储库中。
   3. 选中“Store relative to project location“复选框，然后单击“Next”。
   4. 将“checkstyle.suppressions.file”属性值配置为 `"suppressions.xml"`，然后单击“下一步”，然后单击“完成”。
6. 选择“Flink”作为唯一的活动配置文件，然后单击“Apply”和“OK”。
7. Checkstyle现在会在编辑器中针对任何Checkstyle违规行为发出警告。

安装插件后，可以通过转到Scheme下拉框旁边的 Settings -&gt; Editor -&gt; Code Style -&gt; Java -&gt; Gear Icon 直接导入`"tools/maven/checkstyle.xml"`。例如，这将自动调整导入布局。

可以通过打开Checkstyle工具窗口并单击“检查模块”按钮来扫描整个模块。扫描应报告没有错误。

{% hint style="info" %}
一些模块没有完全被CheckStyle的，其中包括`flink-core`，`flink-optimizer`和`flink-runtime`。不过请确保您在这些模块中添加/修改的代码仍然符合checkstyle规则。
{% endhint %}

### 适用于Scala的Checkstyle

通过选择“Settings -&gt; Editor -&gt; Inspections”，然后搜索“Scala样式检查”，在Intellij中启用scalastyle。也`"tools/maven/scalastyle_config.xml"`放在`"<root>/.idea"`或`"<root>/project"`目录中。

## Eclipse

**注意：**根据我们的经验，由于与Scala IDE 3.0.3捆绑在一起的旧Eclipse版本的不足或者与Scala IDE 4.4.1中捆绑的Scala版本的版本不兼容，此设置不适用于Flink。

**我们建议改用IntelliJ（见**[**上文**](https://ci.apache.org/projects/flink/flink-docs-release-1.7/flinkDev/ide_setup.html#intellij-idea)**）**

