---
title: 'List of Spark Machine Learning Models & Non-overwritten Prediction Columns'
date: 2019-03-30
permalink: /posts/2019/03/spark-model-transform-01/
tags:
  - spark
  - machine learning
  - dataframe
---

I was implementing a paper related to balanced random forest (BRF). Just FYI, a BRF consists of some decision trees where each tree receives instances with a ratio of 1:1 for minority and majority class. A BRF also uses **m** features selected randomly to determine the best split.

However, implementing a BRF in Spark has a limitation. I've mentioned about the limitation <a href="https://albertuskelvin.github.io/posts/2019/03/balanced-random-forest/">here</a>. To deal with such an issue, I made several random forest trees as the replacement to common decision trees. More precisely, I used Spark's random forest.

A problem occurred when the BRF is doing prediction. Since our BRF's model is a list of Spark's random forest classifiers, we need to call **transform()** method for each classifier. This **transform()** method will add the following new columns to the dataframe that is being predicted:

<ul>
	<li>PREDICTION</li>
	<li>PROBABILITY</li>
	<li>rawPrediction</li>
</ul>

For the sake of clarity, here is the code we can use for model's prediction:

<pre>
[1] models = [RandomForestClassifier_01, RandomForestClassifier_02, ..., RandomForestClassifier_0N]
[2] 
[3] for index, model in enumerate(models):
[4]	predicted_df = model.transform(assembled_df)
[5]	
[6]	col_name = 'POSITIVE_PROBA' + str(index)
[7]	positive_probability_udf = F.udf(lambda probas: float(probas[1]), DoubleType())
[8]	assembled_df = predicted_df.withColumn(col_name, positive_probability_udf('PROBABILITY'))
</pre>

From the above code, there are several remarks:

<ol>
	<li><b>RandomForestClassifier</b> is a type of <b>RandomForestClassificationModel</b></li>
	<li><b>assembled_df</b> is the data test which includes the features column (list of features)</li>
	<li><b>positive_probability_udf</b> is our defined function which casts the value of positive class probability to real type</li>
	<li>Line [8] shows that we'd like Spark to add a new column named <b>col_name</b>. The value of this new column is the probability of positive class converted to real type</li>
</ol>

Alright, I executed the code and got these errors:

<pre>
pyspark.sql.utils.IllegalArgumentException: 'requirement failed: Column PREDICTION already exists.'
pyspark.sql.utils.IllegalArgumentException: 'requirement failed: Column PROBABILITY already exists.'
pyspark.sql.utils.IllegalArgumentException: 'requirement failed: Column rawPrediction already exists.'
</pre>

A quick analysis showed that after the first prediction, the dataset has already had those 3 columns. When the 2nd model made a prediction, it couldn't store the values for those 3 columns since they are already exist. I still don't know why Spark doesn't overwrite the values. <b>Anyone knows?</b>

As you might have already known, the solution to this problem is simple. Just add this line after adding the new column (line [8]):

<pre> 
[9]  assembled_df = assembled_df.drop('PROBABILITY', 'PREDICTION', 'rawPrediction')
</pre>

Problem solved.

Thanks for reading.
