# Spark MLLib

## 特性
* 机器学习设计大量迭代计算
* 基于磁盘的 mapreduce 不是和进行迭代计算
* Spark 提供了基于海量数据的机器学习库，提供了 常用机器学习算法的分布式实现

## spark 机器学习库
* spark.mllib 基于 RDD 的原始算法 API
* spark.ml 基于 DataFrames 高层次的 API

## spark 机器学习流水线 （pipeline）
* DataFrame
* Transformer：转换器。将一个 DataFrame 转换为新的 DataFrame。
* Estimator：评估器。在 Pipeline 里 通常是被用来操作 DataFrame 数据并生成一个 Transformer。从技术上讲，Estimator实现了一个方法 fit()，它接受一个DataFrame并产生一个转换器。比如， 一个随机森林算法就是一个 Estimator，它可以调用 fit()，通过训练特征数据而得到一个随机森林模型。
* PipeLine：翻译为流水线或者管道。流水线将多个工作流 阶段（转换器和估计器）连接在一起，形成机器学习的工作 流，并获得结果输出



### spark lr

```python
from pyspark.ml import Pipeline
# 引入 lr 模型
from pyspark.ml.classification import LogisticRegression
# 引入分词、向量化包
from pyspark.ml.feature import HashingTF, Tokenizer
# 构造训练数据
training = spark.createDataFrame([ (0, "a b c d e spark", 1.0), (1, "b d", 0.0), (2, "spark f g h", 1.0), (3, "hadoop mapreduce", 0.0) ], ["id", "text", "label"])
# 构造分词的转换器
tokenizer = Tokenizer(inputCol="text", outputCol="words")
# 构造向量化的转换器
hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
# 构造 lr 模型转换器
lr = LogisticRegression(maxIter=10, regParam=0.001)
# 构造流水线 pipeline
pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])
# 对流水线进行 fit
model = pipeline.fit(training)
# 构造测试数据
test = spark.createDataFrame([ (4, "spark i j k"), (5, "l m n"), (6, "spark hadoop spark"), (7, "apache hadoop") ], ["id", "text"])
# 测试
prediction = model.transform(test)
selected = prediction.select("id", "text", "probability", "prediction")
for i in selected.collect():
    rei, text, prob, prediction = i
    print (rei, text, prob, prediction)
```



### spark tf-idf

```python
from pyspark.ml.feature import HashingTF,IDF,Tokenizer
from pyspark.ml import Pipeline
sentenceData = spark.createDataFrame([(0, "I heard about Spark and I love Spark"),(0, "I wish Java could use case classes"),(1, "Logistic regression models are neat")]).toDF("label", "sentence")
tokenizer = Tokenizer(inputCol="sentence", outputCol="words")
hashingTF = HashingTF(inputCol="words", outputCol="rawFeatures", numFeatures=2000)
idf = IDF(inputCol="rawFeatures", outputCol="features")

wordsData = tokenizer.transform(sentenceData)
featurizedData = hashingTF.transform(wordsData)
idfModel = idf.fit(featurizedData)
rescaledData = idfModel.transform(featurizedData)
rescaledData.select("features", "label").show(truncate=False)
```



## 类型转换

### string 转 index，将字符型标签转换为索引

```python
# 引入包
from pyspark.ml.feature import StringIndexer
# 创建 DataFrame
df = spark.createDataFrame([(0, "a"), (1, "b"), (2, "c"), (3, "a"), (4, "a"), (5, "c")],["id", "category"])
indexer = StringIndexer(inputCol="category", outputCol="categoryIndex")
model = indexer.fit(df)
indexed = model.transform(df)
indexed.collect()
```







## 转换器的 fit、transform