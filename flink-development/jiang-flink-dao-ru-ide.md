# 将Flink导入IDE

以下部分描述了如何将Flink项目导入IDE以开发Flink本身。有关编写Flink程序的信息，请参阅[Java API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/projectsetup/java_api_quickstart.html) 和[Scala API](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/projectsetup/scala_api_quickstart.html) 快速入门指南。

**注意：**每当IDE中的某些内容无效时，请首先尝试使用Maven命令行，`mvn clean package -DskipTests`因为它可能是您的IDE有错误或未正确设置。

## 准备

首先，请先从我们的GitHub[存储库中](https://flink.apache.org/community.html#source-code)获取Flink源代码 ，例如：

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

### Scala Checkstyle

通过选择“Settings -&gt; Editor -&gt; Inspections”，然后搜索“Scala样式检查”，在Intellij中启用scalastyle。也`"tools/maven/scalastyle_config.xml"`放在`"<root>/.idea"`或`"<root>/project"`目录中。

### 常问问题

本节列出了开发人员过去使用IntelliJ时遇到的问题：

* 编译失败 `invalid flag: --add-expots=java.base/sun.net.util=ALL-UNNAMED`

这意味着IntelliJ激活了java11配置文件，尽管使用的是一个更老的JDK。打开Maven工具窗口\(View -&gt; tool Windows -&gt; Maven\)，取消选中java11概要文件并重新导入项目。

*  编译失败 `cannot find symbol: symbol: method defineClass(...) location: class sun.misc.Unsafe`

这意味着IntelliJ在项目中使用的是JDK 11，而你使用的是一个不支持Java 11的Flink版本。当你将IntelliJ设置为使用JDK 11并签出较老版本的Flink\(&lt;= 1.9\)时，通常会发生这种情况。打开项目设置窗口\(文件-&gt;项目结构-&gt;项目设置:项目\)，选择JDK 8作为项目SDK。如果您想使用JDK 11，那么在切换回新的Flink版本之后，您可能需要恢复这个设置。

*  使用Flink类`NoClassDefFoundError`失败示例。

 这可能是由于将Flink依赖项设置为“Provided”，导致它们未自动放置在类路径上。您可以在运行配置中勾选“包括具有'Provided'作用域的依赖关系”框，也可以创建一个调用`main()`示例方法的测试（`provided`依赖关系在测试类路径中可用）。

## Eclipse

**注意：**根据我们的经验，由于与Scala IDE 3.0.3捆绑在一起的旧Eclipse版本的不足或者与Scala IDE 4.4.1中捆绑的Scala版本的版本不兼容，此设置不适用于Flink。

**我们建议改用IntelliJ（见**[**上文**](https://ci.apache.org/projects/flink/flink-docs-release-1.7/flinkDev/ide_setup.html#intellij-idea)**）**

## PyCharm

关于如何为开发flink-python模块设置PyCharm的简要指南。

以下文档介绍了使用Flink Python源设置PyCharm 2019.1.3（[https://www.jetbrains.com/pycharm/download/](https://www.jetbrains.com/pycharm/download/)）的步骤。

### 导入flink-python

如果您位于PyCharm启动界面中：

1. 启动PyCharm，然后选择“打开”
2. 在克隆的Flink存储库中选择flink-python文件夹

如果您使用PyCharm打开了一个项目：

1. 选择“文件-&gt;打开”
2. 在克隆的Flink存储库中选择flink-python文件夹

### Python ****Checkstyle

Apache Flink的Python代码checkstyle应该在项目中创建一个flake8外部工具。

1. 将flake8安装在使用的Python解释器中（请参阅（[https://pypi.org/project/flake8/](https://pypi.org/project/flake8/)）。
2. 选择“ PyCharm-&gt;首选项...-&gt;工具-&gt;外部工具-&gt; +（右侧页面的左下角）”，然后配置外部工具。
3. 将“名称”设置为“ flake8”。
4. 将“描述”设置为“代码样式检查”。
5. 将“程序”设置为Python解释器路径（例如/ usr / bin / python）。
6. 将“参数”设置为“ -m flake8 --config = tox.ini”
7. 将“工作目录”设置为“ $ ProjectFileDir $”

现在，您可以通过以下操作检查您的Python代码样式：“右键单击flink-python项目中的任何文件或文件夹-&gt;外部工具-&gt; flake8”

