# Quick Start

## 介绍

FlinkML旨在使您的数据学习成为一个直接的过程，从而消除了大数据学习任务通常带来的复杂性。在本快速入门指南中，我们将展示使用FlinkML解决简单的监督学习问题是多么容易。但首先是一些基础知识，如果您已经熟悉机器学习（ML），可以跳过接下来的几行。

正如Murphy [\[1\]](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/quickstart.html#murphy)所定义的，ML涉及检测数据中的模式，并使用这些学习模式来预测未来。我们可以将大多数ML算法分为两大类：监督和非监督学习。

* **监督学习**涉及学习从一组输入（特征）到一组输出的函数（映射）。使用我们用来近似映射函数的（输入，输出）对_训练集_来完成学习。监督学习问题进一步分为分类和回归问题。在分类问题中，我们尝试预测示例所属的_类_，例如用户是否要点击广告。另一方面，回归问题是关于预测（实际）数值，通常称为因变量，例如明天的温度。
* **无监督学习**涉及发现数据中的模式和规律。一个例子是_聚类_，我们尝试从描述性特征中发现数据的分组。无监督学习也可用于特征选择，例如通过[主成分分析](https://en.wikipedia.org/wiki/Principal_component_analysis)。

## 链接FlinkML

要在项目中使用FlinkML，首先必须 [设置Flink程序](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking_with_flink.html)。接下来，必须将FlinkML依赖项添加到项目的依赖项中`pom.xml`：

```text
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-ml_2.11</artifactId>
  <version>1.7.2</version>
</dependency>
```

## 加载数据

要加载与FlinkML一起使用的数据，我们可以使用Flink的ETL功能，或者格式化数据的专用函数，比如`LibSVM`格式。对于监督学习问题，通常使用`LabeledVector`类来表示\(`label`, `features`\)示例。`LabeledVector`对象将有一个FlinkML向量成员表示示例的特征，一个双成员表示标签，标签可以是分类问题中的类，也可以是回归问题的因变量。

例如，我们可以使用Haberman的生存数据集，您可以从UCI ML存储库下载该数据集。该数据集“包含了一项关于乳腺癌手术患者存活率的研究的案例”。逗号分隔文件中的数据,前三列和最后一列的特性,和第四列表明病人是否存活5年或更长时间\(标签1\),五年内或死亡\(标签2\)。你可以检查UCI页面数据的更多信息。

我们可以先加载数据集\[String\]:

```scala
import org.apache.flink.api.scala._

val env = ExecutionEnvironment.getExecutionEnvironment

val survival = env.readCsvFile[(String, String, String, String)]("/path/to/haberman.data")
```

我们现在可以将数据转换为DataSet\[LabeledVector\]。这将允许我们使用FlinkML分类算法的数据集。数据集的第4个元素是类标签，其余的是特征，所以我们可以像这样构建LabeledVector元素:

```scala
import org.apache.flink.ml.common.LabeledVector
import org.apache.flink.ml.math.DenseVector

val survivalLV = survival
  .map{tuple =>
    val list = tuple.productIterator.toList
    val numList = list.map(_.asInstanceOf[String].toDouble)
    LabeledVector(numList(3), DenseVector(numList.take(3).toArray))
  }
```

然后我们可以用这些数据来训练学习者。不过，我们将使用另一个数据集来举例说明如何构建学习者;这将允许我们展示如何导入其他数据集格式。

**LibSVM文件**

ML数据集的一种常见格式是libsvm格式，使用该格式的许多数据集可以在[LibSVM数据集网站](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/)上找到。FlinkML提供了一些实用程序，可以通过`MLUtils`对象提供的`readlibsvm`函数使用libsvm格式加载数据集。还可以使用writelibsvm函数以libsvm格式保存数据集。让我们导入svmguide1数据集。你可以在[这里](http://www.csie.ntu.edu.tw/~cjlin/libsvmtools/datasets/binary/svmguide1)下载训练集和测试集。这是一个天体粒子二元分类数据集，由Hsu等人使用。\[3\]在他们的实际支持向量机（SVM）指南中。它包含4个数字特性和类标签。

我们可以使用以下方法简单地导入数据集：

```scala
import org.apache.flink.ml.MLUtils

val astroTrainLibSVM: DataSet[LabeledVector] = MLUtils.readLibSVM(env, "/path/to/svmguide1")
val astroTestLibSVM: DataSet[LabeledVector] = MLUtils.readLibSVM(env, "/path/to/svmguide1.t")
```

这为我们提供了两个`DataSet`对象，我们将在下一节中使用它来创建分类器。

## 分类

导入训练和测试数据集后，需要为分类做好准备。由于Flink SVM只支持阈值为+1.0和-1.0的二进制值，加载LibSVM数据集后需要进行转换，因为它使用了1和0进行标记。

转换可以用一个简单的归一化映射函数:

```scala
import org.apache.flink.ml.math.Vector

def normalizer : LabeledVector => LabeledVector = { 
    lv => LabeledVector(if (lv.label > 0.0) 1.0 else -1.0, lv.vector)
}
val astroTrain: DataSet[LabeledVector] = astroTrainLibSVM.map(normalizer)
val astroTest: DataSet[(Vector, Double)] = astroTestLibSVM.map(normalizer).map(x => (x.vector, x.label))
```

一旦我们转换了数据集，我们就可以训练预测器，例如线性SVM分类器。 我们可以为分类器设置许多参数。 这里我们设置Blocks参数，用于通过底层CoCoA算法\[2\]使用来分割输入。 正则化参数确定所应用的l2正则化的量，其用于避免过度拟合。 步长确定权重向量更新对下一个权重向量值的贡献。 此参数设置初始步长。

```text
import org.apache.flink.ml.classification.SVM

val svm = SVM()
  .setBlocks(env.getParallelism)
  .setIterations(100)
  .setRegularization(0.001)
  .setStepsize(0.1)
  .setSeed(42)

svm.fit(astroTrain)
```

我们现在可以对测试集进行预测，并使用`evaluate`函数创建（真值，预测）对。

```text
val evaluationPairs: DataSet[(Double, Double)] = svm.evaluate(astroTest)
```

接下来，我们将看到我们如何预处理数据，并使用FlinkML的ML管道功能。

## 数据预处理和管道

当使用SVM分类时，通常鼓励\[3\]的预处理步骤是将输入特征缩放到\[0,1\]范围，以避免极值特征占主导地位。FlinkML有许多转换器\(如用于预处理数据的MinMaxScaler\)，其中一个关键特性是能够将转换器和预测器链接在一起。这允许我们运行相同的转换管道，并以一种直接且类型安全的方式对火车和测试数据进行预测。你可以在pipeline文档中阅读更多关于FlinkML管道系统的信息。

让我们首先为数据集中的要素创建一个规范化转换器，并将其链接到一个新的SVM分类器。

```text
import org.apache.flink.ml.preprocessing.MinMaxScaler

val scaler = MinMaxScaler()

val scaledSVM = scaler.chainPredictor(svm)
```

我们现在可以使用新创建的管道来对测试集进行预测。首先，我们再次调用fit，来训练缩放器和SVM分类器。然后，测试集的数据将被自动缩放，然后传递给SVM进行预测。

```text
scaledSVM.fit(astroTrain)

val evaluationPairsScaled: DataSet[(Double, Double)] = scaledSVM.evaluate(astroTest)
```

缩放的输入应该为我们提供更好的预测性能。

