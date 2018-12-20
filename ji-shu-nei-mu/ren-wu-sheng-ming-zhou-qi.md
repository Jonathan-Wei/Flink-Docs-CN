# Task生命周期

Flink中的Task是执行的基本单元。它是一个操作符的每个并行实例作为示例执行的地方，并行度为5的操作符将由一个单独的Task执行其每个实例。

StreamTask是Flink流引擎中所有不同Task子类型的基础。本文将介绍StreamTask生命周期中的不同阶段，并描述表示每个阶段的主要方法。

## 简言之，Operator生命周期

因为Task是执行操作符的并行实例的实体，所以它的生命周期与Operator的生命周期紧密集成。因此，在深入研究StreamTask本身之前，我们将简要介绍表示Operator生命周期的基本方法。下面列出了调用每个方法的顺序。假设一个Operator可以有一个用户定义的函数\(UDF\)，在每个Operator方法下面，我们还在它调用的UDF生命周期中呈现\(缩进\)方法。如果Operator扩展了AbstractUdfStreamOperator\(它是执行udf的所有Operator的基本类\)，那么这些方法是可用的。

```text
// initialization phase
    OPERATOR::setup
        UDF::setRuntimeContext
    OPERATOR::initializeState
    OPERATOR::open
        UDF::open
    
    // processing phase (called on every element/watermark)
    OPERATOR::processElement
        UDF::run
    OPERATOR::processWatermark
    
    // checkpointing phase (called asynchronously on every checkpoint)
    OPERATOR::snapshotState
            
    // termination phase
    OPERATOR::close
        UDF::close
    OPERATOR::dispose
```

简而言之，setup\(\)被调用来初始化一些特定于Operator的机制，比如它的RuntimeContext和它的度量集合数据结构。在此之后，initializeState\(\)为Operator提供初始状态，open\(\)方法执行任何特定于Operator的初始化，比如在AbstractUdfStreamOperator情况下打开用户定义的函数。

{% hint style="danger" %}
注意：initializeState\(\)既包含在初始执行期间初始化操作符状态的逻辑\(例如注册任何键控状态\)，也包含在失败后从检查点检索其状态的逻辑。更多关于这一点，请参阅本页面的其余部分。
{% endhint %}

## Task生命周期

### 正常执行

```text
TASK::setInitialState
    TASK::invoke
	    create basic utils (config, etc) and load the chain of operators
	    setup-operators
	    task-specific-init
	    initialize-operator-states
   	    open-operators
	    run
	    close-operators
	    dispose-operators
	    task-specific-cleanup
	    common-cleanup
```

{% hint style="danger" %}
注意：任务中的连续操作符的打开顺序是从最后一个到第一个。
{% endhint %}

{% hint style="danger" %}
注意：任务中的连续操作符的关闭顺序是从第一个到最后一个。
{% endhint %}

### 中断执行

在前几节中，我们描述了一个任务运行到完成的生命周期。如果任务在任何时候被取消，那么正常的执行将被中断，从那时起执行的唯一操作是计时器服务关闭、特定于任务的清理、操作符的处理和一般任务清理，如上所述。

