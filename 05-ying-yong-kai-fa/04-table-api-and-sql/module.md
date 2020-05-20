# 模块\(Module\)

模块\(Module\)允许用户扩展Flink的内置对象，例如定义行为类似于Flink内置函数的函数。它们是可插入的，尽管Flink提供了一些预构建的模块\(Module\)，但用户可以编写自己的模块\(Module\)。

例如，用户可以定义自己的地理函数，并将其作为内置函数插入Flink，以在Flink SQL和Table API中使用。另一个示例是，用户可以加载现成的Hive模块\(Module\)以将Hive内置功能用作Flink内置功能。

## 模块类型

### 核心模块\(CoreModule\)

 `CoreModule` 包含Flink的所有系统（内置）功能，默认情况下已加载。

### Hive模块\(HiveModule\)

 在`HiveModule`提供Hive内置功能弗林克的系统功能，SQL和表API的用户。Flink的[Hive文档](https://ci.apache.org/projects/flink/flink-docs-release-1.10/dev/table/hive/hive_functions.html)提供了有关设置模块的完整详细信息。

### 用户自定义模块

用户可以通过实现`Module`界面来开发自定义模块。要在SQL CLI中使用自定义模块，用户应通过实现`ModuleFactory`接口来开发模块及其相应的模块工厂。

`ModuleFactory`定义了一组属性，用于在SQL CLI引导时配置模块。属性被传递给一个发现服务，该服务试图将属性匹配到一个`ModuleFactory`并实例化相应的模块实例。

## 命名空间和解析顺序

模块\(Module\)提供的对象被认为是Flink系统（内置）对象的一部分。因此，它们没有任何名称空间。

当两个模块\(Module\)中有两个同名对象时，Flink始终将对象引用解析为第一个已加载模块中的对象。

## Module API

### 加载和卸载模块

用户可以在现有的Flink会话中加载和卸载模块。

{% tabs %}
{% tab title="Java/Scala" %}
```text
tableEnv.loadModule("myModule", new CustomModule());
tableEnv.unloadModule("myModule");
```
{% endtab %}

{% tab title="" %}
 使用YAML定义的所有模块必须提供`type`指定类型的属性。开箱即用地支持以下类型。

| 目录 | 类型值 |
| :--- | :--- |
| core模块 | core |
| Hive模块 | hive |

```
modules:
   - name: core
     type: core
   - name: myhive
     type: hive
     hive-version: 1.2.1
```
{% endtab %}
{% endtabs %}

### 列出可用的模块

{% tabs %}
{% tab title="Java/Scala" %}
```text
tableEnv.listModules();
```
{% endtab %}

{% tab title="SQL" %}
```
Flink SQL> SHOW MODULES;
```
{% endtab %}
{% endtabs %}

