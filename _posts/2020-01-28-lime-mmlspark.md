---
title: 'Distributed LIME with PySpark UDF vs MMLSpark'
date: 2020-01-28
permalink: /posts/2020/01/distributed-lime-with-mmlspark/
tags:
  - lime
  - mmlspark
  - machine learning
  - spark
  - distributed system
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/01/pseudo-distributed-lime-spark-udf/">post</a>, I wrote about how to make LIME run in pseudo-distributed mode with PySpark UDF.

At the end of the post, I also mentioned that I came across a LIME package provided by MMLSpark. You can find its repo <a href="https://github.com/Azure/mmlspark">here</a>. According to the repo, the provided package is the distributed implementation of LIME.

The initial question that popped up in my mind was if this MMLSpark's LIME could be executed in distributed mode, what was the difference with implementing the distributed environment via Spark UDF (refer to the previous post)?

This question was addressed by investigating the source code directly. To be precise, the answer was concluded from <a href="https://github.com/Azure/mmlspark/blob/master/src/main/scala/com/microsoft/ml/spark/lime/LIME.scala"> LIME.scala</a> module.

Turns out that MMLSpark also leverages Spark's UDF to perform LIME's computation.

However, there are several differences between MMLSpark's LIME and marcotcr's LIME, such as:
<ul>
<li>The marcotcr's LIME requires to broadcast the explainer and model object, while MMLSpark's LIME does not</li><li>MMLSpark's LIME has its own implementation for LIME. It does not leverage marcotcr's LIME package</li>
<li>The implementation approach differs a bit for both sources (see below for the explanation)</li>
</ul>

For the sake of clarity, here's how both approaches work in general.

<h3>Marcotcr's LIME</h3>

Apply the following steps to each instance:
<p>
a) Perturb the features to generate local samples<br/>
b) Compute the samples' score using the black-box model<br/>
c) Train the local model
</p>

<h3>MMLSpark's LIME</h3>

<p>
a) For each instance, perturb the features to generate local samples (implemented via UDF)<br/>
	b) <b>After step (a) has been applied to all the instances</b>, compute each sample's score using the black-box model<br/>
	c) <b>After step (b) has been applied to all the samples</b>, train the local model on the samples of each instance (implemented via UDF)
</p>

The following code performs the above steps (MMLSpark's LIME).

```scala
val mapped = df.withColumn(idCol, monotonically_increasing_id())
		  .withColumnRenamed(getInputCol, inputCol2)
		  .withColumn(getInputCol, explode_outer(perturbedDenseVectorsUDF(col(inputCol2))))
		  .mlTransform(getModel)

LIMEUtils.localAggregateBy(mapped, idCol, Seq(getInputCol, getPredictionCol))
     .withColumn(getInputCol, arrToMatUDF(col(getInputCol)))
     .withColumn(getPredictionCol, arrToVectUDF(col(getPredictionCol)))
     .withColumn(getOutputCol, fitLassoUDF(col(getInputCol), col(getPredictionCol), lit(getRegularization)))
     .drop(statesCol, getPredictionCol, idCol, getInputCol)
     .withColumnRenamed(inputCol2, getInputCol)
```
