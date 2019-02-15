# 自定义状态序列化

此页面的目标是为需要对其状态使用自定义序列化的用户提供指导方针，包括如何提供自定义状态序列化器，以及实现允许状态模式演化的序列化器的指导方针和最佳实践。

如果您只是使用Flink自己的序列化器，那么这个页面是无关紧要的，可以忽略它。

## 使用自定义状态序列化器

在注册托管操作符或键控状态时，需要一个状态描述符来指定状态的名称以及关于状态类型的信息。Flink的类型序列化框架使用类型信息为状态创建适当的序列化器。

也可以完全绕过它，让Flink使用您自己的自定义序列化器来序列化托管状态，只需使用您自己的`TypeSerializer`实现直接实例化`StateDescriptor`:

{% tabs %}
{% tab title="Java" %}
```java
public class CustomTypeSerializer extends TypeSerializer<Tuple2<String, Integer>> {...};

ListStateDescriptor<Tuple2<String, Integer>> descriptor =
    new ListStateDescriptor<>(
        "state-name",
        new CustomTypeSerializer());

checkpointedState = getRuntimeContext().getListState(descriptor);
```
{% endtab %}

{% tab title="Scala" %}
```scala
class CustomTypeSerializer extends TypeSerializer[(String, Integer)] {...}

val descriptor = new ListStateDescriptor[(String, Integer)](
    "state-name",
    new CustomTypeSerializer)
)

checkpointedState = getRuntimeContext.getListState(descriptor)
```
{% endtab %}
{% endtabs %}

## 状态序列化器和模式演变

本节介绍与状态序列化和模式演变相关的面向用户的抽象，以及有关Flink如何与这些抽象交互的必要内部详细信息。

### `TypeSerializerSnapshot`抽象

```java
public interface TypeSerializerSnapshot<T> {
    int getCurrentVersion();
    void writeSnapshot(DataOuputView out) throws IOException;
    void readSnapshot(int readVersion, DataInputView in, ClassLoader userCodeClassLoader) throws IOException;
    TypeSerializerSchemaCompatibility<T> resolveSchemaCompatibility(TypeSerializer<T> newSerializer);
    TypeSerializer<T> restoreSerializer();
}
```

```java
public abstract class TypeSerializer<T> {    
    
    // ...
    
    public abstract TypeSerializerSnapshot<T> snapshotConfiguration();
}
```

序列化器的`TypeSerializerSnapshot`是一个时间点信息，它作为状态序列化器的写模式的唯一真实来源，以及恢复与给定时间点相同的序列化器所必需的任何附加信息。在`writeSnapshot`和`readSnapshot`方法中定义了关于在还原时作为序列化器快照应该写入和读取什么内容的逻辑。

注意，快照自己的写模式也可能需要随着时间的推移而改变\(例如，当您希望向快照中添加关于序列化器的更多信息时\)。为了方便实现这一点，需要对快照进行版本控制，并在`getCurrentVersion`方法中定义当前版本号。在还原时，当从保存点读取序列化器快照时，写入快照的模式的版本将提供给`readSnapshot`方法，以便读取实现能够处理不同的版本。

在还原时，应该在`resolveSchemaCompatibility`方法中实现检测新序列化器的模式是否更改的逻辑。当在操作符的恢复执行中，使用新的序列化器再次注册以前的已注册状态时，将通过此方法将新的序列化器提供给前一个序列化器的快照。该方法返回一个`TypeSerializerSchemaCompatibility`，表示兼容性解析的结果，该结果可以是以下情况之一:

1. **`TypeSerializerSchemaCompatibility.compatibleAsIs()`**:这个结果表明新的序列化器是兼容的，这意味着新的序列化器具有与前一个序列化器相同的模式。可能在`resolveSchemaCompatibility`方法中重新配置了新的序列化器，使其兼容。
2. **`TypeSerializerSchemaCompatibility.compatibleAfterMigration()`**:这个结果表明新的序列化器有不同的串行化模式,并可以从旧模式通过使用前面的序列化器\(承认旧模式\)读取字节状态对象,然后重写对象回到字节与新的序列化器\(承认新模式\)
3. **`TypeSerializerSchemaCompatibility.incompatible()`**:这个结果表明新的序列化器具有不同的序列化模式，但是不可能从旧模式迁移。

最后一点细节是在需要迁移的情况下如何获得前面的序列化器。序列化器的`TypeSerializerSnapshot`的另一个重要作用是，它作为一个工厂来恢复之前的序列化器。更具体地说，`TypeSerializerSnapshot`应该实现`restoreSerializer`方法来实例化一个序列化器实例，该实例识别前一个序列化器的模式和配置，因此可以安全地读取前一个序列化器编写的数据。

### Flink如何与TypeSerializer和TypeSerializerSnapshot抽象交互

最后，本节总结Flink，或者更具体地说是状态后端，如何与抽象进行交互。根据状态后端，交互略有不同，但这与状态序列化器及其序列化器快照的实现是正交的。

**堆外状态后端（例如RocksDBStateBackend）**

1. **使用具有模式**_**A**_**的状态序列化程序注册新状态**
   * 注册`TypeSerializer`的状态用于在每个状态访问时读/写状态。
   * 状态写在模式_A中_。
2. **获取一个保存点**
   * 通过该`TypeSerializer#snapshotConfiguration`方法提取序列化程序快照。
   * 序列化程序快照将写入保存点，以及已经序列化的状态字节（使用架构_A_）。
3. **恢复的执行使用具有模式**_**B的**_**新状态序列化器重新访问恢复的状态字节**
   * 恢复先前的状态序列化程序的快照。
   * 状态字节在还原时不反序列化，仅加载回状态后端（因此，仍在模式_A中_）。
   * 收到新的序列化程序后，通过`TypeSerializer#resolveSchemaCompatibility`检查架构兼容性，将其提供给恢复的先前序列化程序的快照 。
4. **将后端中的状态字节从架构**_**A**_**迁移到架构**_**B.**_
   * 如果兼容性解决方案反映了架构已更改并且可以进行迁移，则会执行架构迁移。识别模式_A_的先前状态序列化器将从串行器快照获取，通过 `TypeSerializerSnapshot#restoreSerializer()`，并用于将状态字节反序列化为对象，然后使用新的序列化器再次重写，后者识别模式_B_以完成迁移。在继续处理之前，将访问状态的所有条目全部迁移。
   * 如果解析信号不兼容，则状态访问失败并出现异常

**堆状态后端（例如MemoryStateBackend，FsStateBackend）**

1. **使用具有模式**_**A**_**的状态序列化程序注册新状态**
   * 注册`TypeSerializer`由状态后端维护。
2. **获取保存点，使用模式**_**A**_**序列化所有状态**
   * 通过该`TypeSerializer#snapshotConfiguration`方法提取序列化程序快照。
   * 序列化程序快照将写入保存点。
   * 现在，状态对象被序列化为保存点，以模式_A_编写。
3. **在还原时，将状态反序列化为堆中的对象**
   * 恢复先前的状态序列化程序的快照。
   * 识别模式_A_的先前序列化程序是从序列化程序快照获取的， `TypeSerializerSnapshot#restoreSerializer()`用于将状态字节反序列化为对象。
   * 从现在开始，所有的状态都已经反序列化了。
4. **恢复的执行使用具有模式**_**B的**_**新状态序列化器重新访问先前的状态**
   * 收到新的序列化程序后，通过`TypeSerializer#resolveSchemaCompatibility`检查架构兼容性，将其提供给恢复的先前序列化程序的快照 。
   * 如果兼容性检查发出需要迁移的信号，则在这种情况下不会发生任何事情，因为对于堆后端，所有状态都已反序列化为对象。
   * 如果解析信号不兼容，则状态访问失败并出现异常。
5. **拿另一个保存点，使用模式**_**B**_**序列化所有状态**
   * 与步骤2相同，但现在状态字节都在模式_B中_。

## 实施说明和最佳做法

**1. Flink通过使用类名实例化它们来恢复序列化程序快照**

序列化器的快照是注册状态如何序列化的唯一真实来源，它充当保存点中读取状态的入口点。为了能够恢复和访问以前的状态，必须能够恢复以前的状态序列化器的快照。

Flink通过首先实例化`TypeSerializerSnapshot`及其类名\(与快照字节一起编写\)来恢复序列化器快照。因此，为了避免意外更改类名或实例化失败，`TypeSerializerSnapshot`类应该:

* 避免被实现为匿名类或嵌套类， 
* 实例化有一个公共的空构造函数

**2.避免TypeSerializerSnapshot跨不同的序列化程序共享同一个类**

由于模式兼容性检查通过序列化程序快照进行，因此让多个序列化程序返回`TypeSerializerSnapshot`与其快照相同的类会使`TypeSerializerSnapshot#resolveSchemaCompatibility`和`TypeSerializerSnapshot#restoreSerializer()`方法的实现复杂化 。

**3.将该CompositeSerializerSnapshot实用程序用于包含嵌套序列化程序的序列化程序**

在某些情况下，类型序列化器依赖于其他嵌套的类型序列化器;以Flink的`TupleSerializer`为例，其中为`tuple`字段配置了嵌套的类型序列化器。在这种情况下，最外层序列化器的快照还应该包含嵌套序列化器的快照。

`CompositeSerializerSnapshot`可以专门用于这个场景。它封装了解析复合序列化器的总体模式兼容性检查结果的逻辑。关于如何使用它的示例，可以参考Flink的[`ListSerializerSnapshot`](https://github.com/apache/flink/blob/master/flink-core/src/main/java/org/apache/flink/api/common/typeutils/base/ListSerializerSnapshot.java)实现。

## 在Flink 1.7之前从已弃用的序列化程序快照API迁移

本节是从Flink 1.7之前存在的序列化程序和序列化程序快照进行API迁移的指南。

在Flink 1.7之前，序列化器快照是作为`TypeSerializerConfigSnapshot`实现的\(现在已经弃用了，将来会被删除，完全由新的`TypeSerializerSnapshot`接口替代\)。此外，序列化器模式兼容性检查的职责存在于`TypeSerializer`中，在`TypeSerializer#ensureCompatibility(TypeSerializerConfigSnapshot)`方法中实现。

新旧抽象之间的另一个主要区别是，已弃用的`TypeSerializerConfigSnapshot`不具备实例化前一个序列化器的能力。因此，如果序列化器仍然返回`TypeSerializerConfigSnapshot`的子类作为其快照，则序列化器实例本身将始终使用Java序列化被写入保存点，以便在还原时可以使用上一个序列化器。这是非常不可取的，因为恢复作业是否成功取决于前一个序列化器的类的可用性，或者通常情况下，是否可以在还原时使用Java序列化读回序列化器实例。这意味着您的状态只能使用相同的序列化器，一旦您想要升级序列化器类或执行模式迁移，就会出现问题。

为了面向未来并具有迁移状态序列化程序和模式的灵活性，强烈建议从旧的抽象中进行迁移。执行此操作的步骤如下：

1. 实现`TypeSerializerSnapshot`的新子类。这将是序列化器的新快照。
2. 在`TypeSerializer#snapshotConfiguration()`方法中，返回新的`TypeSerializerSnapshot`作为序列化器的序列化器快照。 
3. 从Flink 1.7之前存在的保存点恢复作业，然后再次获取保存点。注意，在此步骤中，序列化器的旧`TypeSerializerConfigSnapshot`必须仍然存在于类路径中，并且不能删除`TypeSerializer#ensureCompatibility(TypeSerializerConfigSnapshot)`方法的实现。这个过程的目的是用序列化器新实现的`TypeSerializerSnapshot`替换用旧保存点编写的`TypeSerializerConfigSnapshot`。 
4. 一旦您使用Flink 1.7获取了一个保存点，保存点将包含`TypeSerializerSnapshot`作为状态序列化器快照，并且序列化器实例将不再被写入保存点。现在，可以安全地从序列化器中删除旧抽象的所有实现\(删除旧的`TypeSerializerConfigSnapshot`实现，以及从序列化器中删除`TypeSerializer#ensureCompatibility(TypeSerializerConfigSnapshot))。`



