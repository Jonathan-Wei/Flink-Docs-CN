# 状态模式演化

Apache Flink流应用程序通常设计为无限期或长时间运行。对于所有长期运行的服务，需要更新应用程序以适应不断变化的需求。对于应用程序所针对的数据模式也是如此;它们随着应用程序的变化而变化。

此页面概述了如何演进状态类型的数据模式。当前的限制因不同类型和状态结构\(ValueState、ListState等\)而异。

请注意，只有在使用由Flink自己的类型序列化框架生成的状态序列化器时，此页面上的信息才是相关的。也就是说，在声明状态时，所提供的状态描述符没有配置为使用特定的类型序列化器或类型信息，在这种情况下，Flink会推断出关于状态类型的信息:

```java
ListStateDescriptor<MyPojoType> descriptor =
    new ListStateDescriptor<>(
        "state-name",
        MyPojoType.class);

checkpointedState = getRuntimeContext().getListState(descriptor);
```

实际上，状态模式是否可以演化取决于用于读写持久状态字节的Serializer。简单地说，注册状态的模式只有在其Serializer正确支持它的情况下才能演化。这是由Flink类型序列化框架生成的Serializer透明地处理的\(当前支持范围[如下所示](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/schema_evolution.html#supported-data-types-for-schema-evolution)\)。

如果打算为状态类型实现自定义类型Serializer，并且想了解如何实现该Serializer以支持状态模式演化，请参阅[自定义状态序列化](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/custom_serialization.html)。那里的文档还包括关于状态序列化器和Flink的状态后端之间相互作用的必要内部细节，以支持状态模式演化。

## 状态模式演化

要演化给定状态类型的模式，您可以采取以下步骤:

1. 获取Flink流媒体作业的保存点\(SavePoint\)。
2. 更新应用程序中的状态类型（例如，修改Avro类型架构）。
3. 从保存点还原作业。首次访问状态时，Flink将评估模式是否已针对状态进行更改，并在必要时迁移状态模式。

迁移状态以适应更改的模式的过程自动发生，并且对每个状态独立发生。这个过程由Flink在内部执行，首先检查状态的新Serializer是否具有与前一个Serializer不同的序列化模式;如果是，则使用前面的Serializer将状态读入对象，并使用新的Serializer再次将状态写入字节。

有关迁移过程的更多详细信息超出了本文档的范围; 请参考 [这里](https://ci.apache.org/projects/flink/flink-docs-master/dev/stream/state/custom_serialization.html)。

## 模式演化支持的数据类型

目前，仅支持 POJO 和 Avro 类型的 schema 升级 因此，如果你比较关注于状态数据结构的升级，那么目前来看强烈推荐使用 Pojo 或者 Avro 状态数据类型。

我们有计划支持更多的复合类型；更多的细节可以参考 [FLINK-10896](https://issues.apache.org/jira/browse/FLINK-10896)。

### POJO 类型

 Flink 基于下面的规则来支持 [POJO 类型](https://ci.apache.org/projects/flink/flink-docs-release-1.10/zh/dev/types_serialization.html#pojo-%E7%B1%BB%E5%9E%8B%E7%9A%84%E8%A7%84%E5%88%99)结构的升级:

1. 可以删除字段。一旦删除，被删除字段的前值将会在将来的 checkpoints 以及 savepoints 中删除。
2. 可以添加字段。新字段会使用类型对应的默认值进行初始化，比如 [Java 类型](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/datatypes.html)。
3. 不可以修改字段的声明类型。
4. 不可以改变 POJO 类型的类名，包括类的命名空间。

需要注意，只有从 1.8.0 及以上版本的 Flink 生产的 savepoint 进行恢复时，POJO 类型的状态才可以进行升级。 对 1.8.0 版本之前的 Flink 是没有办法进行 POJO 类型升级的。

### Avro类型

Flink 完全支持 Avro 状态类型的升级，只要数据结构的修改是被 [Avro 的数据结构解析规则](http://avro.apache.org/docs/current/spec.html#Schema+Resolution)认为兼容的即可。

一个例外是如果新的 Avro 数据 schema 生成的类无法被重定位或者使用了不同的命名空间，在作业恢复时状态数据会被认为是不兼容的。

## 模式迁移限制

 Flink的模式迁移具有一些限制，以确保正确性。对于需要解决这些限制并了解它们在特定用例中是安全的用户，请考虑使用[自定义序列化程序](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/dev/datastream/fault-tolerance/custom_serialization/)或 [状态处理器api](https://ci.apache.org/projects/flink/flink-docs-release-1.13/docs/libs/state_processor_api/)。

### 不支持密钥的模式演变

密钥的结构无法迁移，因为这可能导致不确定的行为。例如，如果将POJO用作键，并且丢弃了一个字段，则可能突然有多个单独的键现在相同。Flink无法合并相应的值。

此外，RocksDB状态后端依赖于二进制对象身份，而不是`hashCode`方法。密钥对象结构的任何更改都可能导致不确定的行为。

### **Kryo**不能用于模式演变

使用Kryo时，模式无法验证是否进行了任何不兼容的更改。

