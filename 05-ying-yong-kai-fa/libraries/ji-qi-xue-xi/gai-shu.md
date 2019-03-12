# 概述

FlinkML是Flink的机器学习（ML）库。这是Flink社区的一项新工作，其中包含越来越多的算法和贡献者。使用FlinkML，我们的目标是提供可伸缩的ML算法、直观的API和帮助最小化端到端ML系统中的粘合代码的工具。您可以在我们的[愿景和路线图中查看](https://cwiki.apache.org/confluence/display/FLINK/FlinkML%3A+Vision+and+Roadmap)有关我们目标的更多详细信息以及Library的发展方向。

## 支持的算法

FlinkML目前支持以下算法：

### 监督学习

* [使用通信高效分布式双坐标上升（CoCoA）的SVM](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/svm.html)
* [多元线性回归](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/multiple_linear_regression.html)
* [优化框架](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/optimization.html)

### 无监督学习

* [k-最近邻居加入](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/knn.html)

### 数据预处理

* [多项式特征](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/polynomial_features.html)
* [标准比例尺](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/standard_scaler.html)
* [MinMax Scaler](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/min_max_scaler.html)

### 推荐

* [交替最小二乘（ALS）](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/als.html)

### 离群选择

* [随机异常值选择（SOS）](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/sos.html)

### 公用程式

* [距离指标](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/distance_metrics.html)
* [交叉验证](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/cross_validation.html)

## 入门

可以查看我们的[快速入门指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/quickstart.html)，了解全面的入门示例。

如果你想直接进入，必须[设置一个Flink程序](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking_with_flink.html)。接下来，必须将FlinkML依赖项添加到项目的依赖项中`pom.xml`。

```markup
<dependency>
  <groupId>org.apache.flink</groupId>
  <artifactId>flink-ml_2.11</artifactId>
  <version>1.7.2</version>
</dependency>
```

请注意，FlinkML目前不是二进制分发的一部分。请参阅[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/linking.html)与其链接以进行群集执行。

现在你可以开始解决分析任务了。以下代码片段展示了训练多元线性回归模型是多么容易。

```scala
// LabeledVector is a feature vector with a label (class or real value)
val trainingData: DataSet[LabeledVector] = ...
val testingData: DataSet[Vector] = ...

// Alternatively, a Splitter is used to break up a DataSet into training and testing data.
val dataSet: DataSet[LabeledVector] = ...
val trainTestData: DataSet[TrainTestDataSet] = Splitter.trainTestSplit(dataSet)
val trainingData: DataSet[LabeledVector] = trainTestData.training
val testingData: DataSet[Vector] = trainTestData.testing.map(lv => lv.vector)

val mlr = MultipleLinearRegression()
  .setStepsize(1.0)
  .setIterations(100)
  .setConvergenceThreshold(0.001)

mlr.fit(trainingData)

// The fitted model can now be used to make predictions
val predictions: DataSet[LabeledVector] = mlr.predict(testingData)
```

## 管道

FlinkML的一个关键概念是其受[scikit-learn](http://scikit-learn.org/)启发的管道机制。它允许您快速构建复杂的数据分析管道，使其出现在每个数据科学家的日常工作中。可以在[此处](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/pipelines.html)找到有关FlinkML管道及其内部工作的深入描述。

以下示例代码显示了使用FlinkML设置分析管道是多么容易。

```scala
val trainingData: DataSet[LabeledVector] = ...
val testingData: DataSet[Vector] = ...

val scaler = StandardScaler()
val polyFeatures = PolynomialFeatures().setDegree(3)
val mlr = MultipleLinearRegression()

// Construct pipeline of standard scaler, polynomial features and multiple linear regression
val pipeline = scaler.chainTransformer(polyFeatures).chainPredictor(mlr)

// Train pipeline
pipeline.fit(trainingData)

// Calculate predictions
val predictions: DataSet[LabeledVector] = pipeline.predict(testingData)
```

可以通过调用方法将a链接`Transformer`到另一个`Transformer`或一组链接。如果想要链接到一个或一组链接，则必须调用该方法。`TransformerschainTransformerPredictorTransformerTransformerschainPredictor`

## 如何贡献

Flink社区欢迎所有希望参与Flink及其图书馆开发的贡献者。为了快速开始为FlinkML做出贡献，请阅读我们的官方 [贡献指南](https://ci.apache.org/projects/flink/flink-docs-release-1.7/dev/libs/ml/contribution_guide.html)。

