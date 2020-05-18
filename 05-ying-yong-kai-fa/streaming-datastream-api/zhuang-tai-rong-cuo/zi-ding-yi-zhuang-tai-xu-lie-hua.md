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

从SavePoint还原时，Flink允许更改用于读取和写入先前注册状态的序列化器，这样用户就不会被锁定在任何特定的序列化模式中。恢复状态后，将为该状态注册一个新的序列化器（即，`StateDescriptor`用于访问已恢复作业中的状态的序列化器）。此新的序列化器可能具有与以前的序列化器不同的架构。因此，在实现状态序列化器时，除了读取/写入数据的基本逻辑外，还要牢记的另一件事是将来如何更改序列化架构。

在谈到_模式时_，在此上下文中，该术语在引用状态类型的_数据模型_与状态类型的_序列化二进制格式_之间是可以互换的。一般来说，该模式可以在以下几种情况下进行更改：

1. 状态类型的数据模式已得到发展，即从用作状态的POJO中添加或删除字段。
2. 一般来说，更改数据模式后，需要升级序列化程序的序列化格式。
3. 串行器的配置已更改。

为了使新执行具有有关_书面_状态_模式_的信息并检测_模式_是否已更改，在获取操作员状态的保存点时，需要将状态序列化器的_快照_与状态字节一起写入。这是抽象的`TypeSerializerSnapshot`，将在下一部分中进行说明。

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

## 预定义的便捷`TypeSerializerSnapshot`类

Flink提供了两个抽象`TypeSerializerSnapshot`基类，它们可用于典型方案： `SimpleTypeSerializerSnapshot`和`CompositeTypeSerializerSnapshot`。

提供这些预定义快照作为其序列化程序快照的序列化程序必须始终具有自己的独立子类实现。这与不跨不同序列化程序共享快照类的最佳做法相对应，这将在下一节中进行更详细的说明。

### 实现`SimpleTypeSerializerSnapshot`

SimpleTypeSerializerSnapshot适用于没有任何状态或配置的序列化器，本质上意味着序列化器的序列化模式仅由序列化器的类定义。

当使用SimpleTypeSerializerSnapshot作为你的序列化器的快照类时，只有2种可能的兼容性结果:

* `TypeSerializerSchemaCompatibility.compatibleAsIs()`，如果新的序列化器类保持相同，或者
* `TypeSerializerSchemaCompatibility.incompatible()`，如果新的序列化程序类与前一个不同。

 以下是使用方法的示例`SimpleTypeSerializerSnapshot`，以Flink `IntSerializer`为例：

```java
public class IntSerializerSnapshot extends SimpleTypeSerializerSnapshot<Integer> {
    public IntSerializerSnapshot() {
        super(() -> IntSerializer.INSTANCE);
    }
}
```

`IntSerializer`没有状态或配置。序列化格式仅由序列化器类本身定义，并且只能由另一个IntSerializer读取。因此，它适合`SimpleTypeSerializerSnapshot`的场景。

`SimpleTypeSerializerSnapshot`的基本超级构造函数需要相应的序列化器实例的提供者，不管快照当前是被恢复还是在快照期间被写入。该供应商用于创建恢复序列化器，以及用于验证新序列化器是否属于预期的序列化器类的类型检查。

### 实现`CompositeTypeSerializerSnapshot`

 `CompositeTypeSerializerSnapshot`适用于依靠系列化多重嵌套串行序列化

在进一步解释之前，我们将依赖于多个嵌套序列化器的序列化器称为上下文中的“外部”序列化器。例如`MapSerializer`、`ListSerializer`、`GenericArraySerializer`等等。例如，考虑`MapSerializer`——键和值序列化器是嵌套的序列化器，而MapSerializer本身是“外部的”序列化器。

在这种情况下，外部序列化器的快照还应该包含嵌套序列化器的快照，这样就可以独立地检查嵌套序列化器的兼容性。在解决外部序列化器的兼容性时，需要考虑每个嵌套的序列化器的兼容性。

提供CompositeTypeSerializerSnapshot是为了帮助实现这些组合序列化器的快照。它处理读取和写入嵌套序列化器快照，以及考虑到所有嵌套序列化器的兼容性而解析最终的兼容性结果。

下面是如何使用CompositeTypeSerializerSnapshot的一个例子，使用Flink的MapSerializer作为一个例子:

```java
public class MapSerializerSnapshot<K, V> extends CompositeTypeSerializerSnapshot<Map<K, V>, MapSerializer> {

    private static final int CURRENT_VERSION = 1;

    public MapSerializerSnapshot() {
        super(MapSerializer.class);
    }

    public MapSerializerSnapshot(MapSerializer<K, V> mapSerializer) {
        super(mapSerializer);
    }

    @Override
    public int getCurrentOuterSnapshotVersion() {
        return CURRENT_VERSION;
    }

    @Override
    protected MapSerializer createOuterSerializerWithNestedSerializers(TypeSerializer<?>[] nestedSerializers) {
        TypeSerializer<K> keySerializer = (TypeSerializer<K>) nestedSerializers[0];
        TypeSerializer<V> valueSerializer = (TypeSerializer<V>) nestedSerializers[1];
        return new MapSerializer<>(keySerializer, valueSerializer);
    }

    @Override
    protected TypeSerializer<?>[] getNestedSerializers(MapSerializer outerSerializer) {
        return new TypeSerializer<?>[] { outerSerializer.getKeySerializer(), outerSerializer.getValueSerializer() };
    }
}
```

当实现一个新的序列化器快照作为CompositeTypeSerializerSnapshot的子类时，必须实现以下三个方法:

* `#getCurrentOuterSnapshotVersion()`：此方法定义当前外部序列化器快照的序列化二进制格式的版本。。
* `#getNestedSerializers(TypeSerializer)`：给定外部序列化器，返回其嵌套的序列化器。
* `#createOuterSerializerWithNestedSerializers(TypeSerializer[])`：给定嵌套的序列化器，创建外部序列化器的实例。

上面的例子是一个CompositeTypeSerializerSnapshot，其中除了嵌套的序列化器快照之外，没有其他需要快照的信息。因此，它的外部快照版本永远不需要增加。然而，其他一些序列化器包含一些额外的静态配置，需要与嵌套组件序列化器一起持久化。这方面的一个例子是Flink的GenericArraySerializer，它在配置中除了嵌套元素序列化器之外还包含数组元素类型的类。

在这些情况下，需要在CompositeTypeSerializerSnapshot上实现另外三个方法:

* `#writeOuterSnapshot(DataOutputView)`：定义外部快照信息的写入方式。
* `#readOuterSnapshot(int, DataInputView, ClassLoader)`：定义如何读取外部快照信息。
* `#isOuterSnapshotCompatible(TypeSerializer)`：检查外部快照信息是否相同。

默认情况下，CompositeTypeSerializerSnapshot假定没有任何外部快照信息可读/可写，因此具有上述方法的空的默认实现。如果子类具有外部快照信息，则必须实现所有三个方法。

下面是一个例子，使用Flink的GenericArraySerializer作为例子，说明如何将CompositeTypeSerializerSnapshot用于具有外部快照信息的复合序列化器快照:

```java
public final class GenericArraySerializerSnapshot<C> extends CompositeTypeSerializerSnapshot<C[], GenericArraySerializer> {

    private static final int CURRENT_VERSION = 1;

    private Class<C> componentClass;

    public GenericArraySerializerSnapshot() {
        super(GenericArraySerializer.class);
    }

    public GenericArraySerializerSnapshot(GenericArraySerializer<C> genericArraySerializer) {
        super(genericArraySerializer);
        this.componentClass = genericArraySerializer.getComponentClass();
    }

    @Override
    protected int getCurrentOuterSnapshotVersion() {
        return CURRENT_VERSION;
    }

    @Override
    protected void writeOuterSnapshot(DataOutputView out) throws IOException {
        out.writeUTF(componentClass.getName());
    }

    @Override
    protected void readOuterSnapshot(int readOuterSnapshotVersion, DataInputView in, ClassLoader userCodeClassLoader) throws IOException {
        this.componentClass = InstantiationUtil.resolveClassByName(in, userCodeClassLoader);
    }

    @Override
    protected boolean isOuterSnapshotCompatible(GenericArraySerializer newSerializer) {
        return this.componentClass == newSerializer.getComponentClass();
    }

    @Override
    protected GenericArraySerializer createOuterSerializerWithNestedSerializers(TypeSerializer<?>[] nestedSerializers) {
        TypeSerializer<C> componentSerializer = (TypeSerializer<C>) nestedSerializers[0];
        return new GenericArraySerializer<>(componentClass, componentSerializer);
    }

    @Override
    protected TypeSerializer<?>[] getNestedSerializers(GenericArraySerializer outerSerializer) {
        return new TypeSerializer<?>[] { outerSerializer.getComponentSerializer() };
    }
}
```

在上面的代码片段中有两件重要的事情需要注意。首先，由于这个CompositeTypeSerializerSnapshot实现具有作为快照的一部分写入的外部快照信息，所以每当外部快照信息的序列化格式发生变化时，必须选中由getCurrentOuterSnapshotVersion\(\)定义的外部快照版本。

其次，注意我们在编写组件类时如何避免使用Java序列化，只编写类名并在读取快照时动态加载它。在编写序列化器快照的内容时避免使用Java序列化通常是一个很好的实践。关于这一点的更多细节将在下一节中介绍。

## 实现说明和最佳做法

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



