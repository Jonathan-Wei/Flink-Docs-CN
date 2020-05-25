# 插件\(Plugins\)

插件通过受限的类加载器促进严格的代码分离。插件无法访问其他插件或Flink中未明确列入白名单的类。这种严格的隔离允许插件包含相同库的冲突版本，而无需重新定位类或收敛到通用版本。当前，只有文件系统是可插拔的，但是将来，连接器，格式甚至用户代码也应该是可插拔的。

## 隔离和插件结构

插件位于它们自己的文件夹中，并且可以包含多个jar。插件文件夹的名称是任意的。

```text
flink-dist
├── conf
├── lib
...
└── plugins
    ├── s3
    │   ├── aws-credential-provider.jar
    │   └── flink-s3-fs-hadoop.jar
    └── azure
        └── flink-azure-fs-hadoop.jar
```

每个插件都是通过自己的类加载器加载的，并且与其他插件完全隔离。因此，flink-s3-fs-hadoop和flink-az -fs-hadoop可以依赖于不同的冲突库版本。在创建fat jar\(shading\)的过程中不需要重新定位任何类。

插件可以从Flink的lib/文件夹访问某些白名单包。特别是，所有必需的服务提供程序接口\(SPI\)都是通过系统类加载器加载的，这样就不会有org.apache.flink.core.fs的两个版本。文件系统在任何给定的时间都存在，即使用户不小心将它捆绑到他们的fat jar中。这个单例类需求是非常必要的，这样Flink运行时就有了一个进入插件的入口点。服务类是通过java.util发现的。ServiceLoader，所以确保在shading期间在META-INF/services中保留服务定义。

{% hint style="warning" %}
_当前，随着我们完善SPI系统，仍然可以从插件访问更多Flink核心类。_
{% endhint %}

此外，最常用的记录器框架被列入白名单，这样就可以在Flink核心，插件和用户代码之间统一进行记录。

## 文件系统

除了MapR之外，所有文件系统都是可插入的。这意味着它们可以并且应该作为插件使用。要使用可插入的文件系统，在启动Flink之前，将相应的JAR文件从opt目录复制到Flink发行版的plugins目录下的一个目录，例如。

```text
mkdir ./plugins/s3-fs-hadoop
cp ./opt/flink-s3-fs-hadoop-1.10.0.jar ./plugins/s3-fs-hadoop/
```

{% hint style="warning" %}
 警告：**s3文件系统（`flink-s3-fs-presto`和 `flink-s3-fs-hadoop`）只能用作插件，因为我们已经删除了重定位。**将它们放在libs /中会导致系统故障。
{% endhint %}

{% hint style="warning" %}
 注意：**由于严格的\[隔离\]（＃isolation-and-plugin-structure），文件系统再也无法访问lib /中的凭据提供程序。**请将任何需要的提供程序添加到相应的插件文件夹中。
{% endhint %}

