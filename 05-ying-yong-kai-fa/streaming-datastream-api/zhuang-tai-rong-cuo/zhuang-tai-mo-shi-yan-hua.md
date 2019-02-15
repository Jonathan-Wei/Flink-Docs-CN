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

目前，仅Avro支持模式演进。因此，如果您关心状态的模式演变，目前建议始终将Avro用于状态数据类型。

有计划扩展对更多复合类型的支持，例如POJO; 有关详细信息，请参阅[FLINK-10897](https://issues.apache.org/jira/browse/FLINK-10897)。

### Avro类型

Flink完全支持Avro类型状态的演进模式，只要模式更改被[Avro的模式解析规则](http://avro.apache.org/docs/current/spec.html#Schema+Resolution)视为兼容 。

一个限制是，当作业恢复时，Avro生成的用作状态类型的类不能被重新定位或具有不同的名称空间。

