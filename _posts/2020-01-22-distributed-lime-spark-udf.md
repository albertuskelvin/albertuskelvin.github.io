---
title: 'Pseudo-distributed LIME via PySpark UDF'
date: 2020-01-22
permalink: /posts/2020/01/pseudo-distributed-lime-spark-udf/
tags:
  - lime
  - spark
  - udf
  - distributed
---

The initial question that popped up in my mind was how to make LIME performs faster. This should be useful enough when the data to explain is big enough.

Please refer to this Github <a href="https://github.com/marcotcr/lime">repo</a> for more info about LIME.

Firstly, does LIME support multiprocessing?

Based on <a href="https://github.com/marcotcr/lime/blob/master/lime/lime_base.py">lime_base.py</a> and <a href="https://github.com/marcotcr/lime/blob/master/lime/lime_tabular.py">lime_tabular.py</a> modules, there is no any part of code that shows the use of python's multiprocessing module. Therefore, I think, we can safely say that currently LIME doesn't support multiprocessing.

Secondly, is it possible to execute LIME in distributed mode via Spark UDF?

I tested the above question recently. However, the simulation didn't exactly describe the problem specification where the environment should be purely distributed. I simulated the process in pseudo-distributed mode where a single machine consists of many workers. Additionally, I leveraged PySpark UDF.

The followings are the steps used to come up with the answer.

**Step 1.** Create a sample dataframe and convert the data and labels to a numpy array. Suppose that the dataframe is stored in a variable called df

```python
feature_cols = <list_of_feature_columns>
np_data = np.array(df.select(*feature_cols).collect())
np_label = np.array(df.select('label').collect()).ravel()
```

**Step 2.** Train the black-box model. A random forest classifier was used during the investigation

```python
model = RandomForestClassifier(max_depth=2, random_state=0).fit(np_data, np_label)
```

**Step 3.** Broadcast the trained model to the executors since UDF can not receive non-column parameters

```python
broadcasted_model = sc.broadcast(model)
```

**Step 4.** Create the LIME's tabular explainer object

```python
explainer = LimeTabularExplainer(np_data, feature_names=feature_cols)
```

**Step 5.** Broadcast the explainer object to the executors

```python
broadcasted_explainer = dill.loads(sc.broadcast(dill.dumps(explainer)).value)
```

**Step 6.** Create the UDF

```python
schema = ArrayType(StructType([
	StructField("feature_explanation", StringType(), False),
	StructField("weight", FloatType(), False)
]))

@udf(returnType=schema)
def explainer_udf(*feature_cols):
	np_row = np.array([value for value in feature_cols])

	# just for distributed mode checking purpose
	print(np_row)
	print(broadcasted_model.value)
	print(broadcasted_explainer)

	explained_instance = broadcasted_explainer.explain_instance(
		np_row,
		broadcasted_model.value.predict_proba
	).as_list()

	return explained_instance
```

**Step 7.** Apply the UDF on the dataframe

```python
df = df.withColumn('explanations', explainer_udf(*feature_cols))
df.show(truncate=False)
```

The pseudo-distributed environment was created by leveraging Spark's standalone cluster. The cluster consists of two workers (same cores and memory) within a single node. Please refer to this <a href="https://albertuskelvin.github.io/posts/2020/01/spark-standalone-multi-workers-single-node/">post</a> for how to set up a Spark's standalone cluster consisting of multiple workers within a single machine.

After submitting the above Spark job to the cluster, we can check the job history via the master web UI. Turns out that each active worker allocated for the job executes the UDF. The three information (<i>np_row, broadcasted model,</i> and <i>broadcasted explainer</i>) were printed on the worker's `stderr`. This simply shows that <b>we can make LIME runs on pseudo-distributed mode via PySpark UDF.</b>
