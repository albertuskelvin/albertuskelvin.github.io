---
title: 'Covariate Shift Detection with Machine Learning Based Approach'
date: 2020-03-07
permalink: /posts/2020/03/covariate-shift-detection-machine-learning-based-approach/
tags:
  - covariate shift
  - machine learning
---

Covariate shift happens when the distribution of train data differs with the distribution of test data. Take a look at the following probability equation.

<p>
<h3>
P(y | x) = (P(x | y) . P(y)) / P(x)
</h3>
</p>

In covariate shift, the `P(x | y)` and `P(x)` from the above equation are different between train & test data. However, the `P(y | x)` for both train & test are the same.

If we look at the above equation further, we can transform it into another form such as the following.

<p>
<h3>
P(y | x) = P(y & x) / P(x)
</h3>
</p>

Based on the above equation, since for both of train & test data the `P(y | x)` does not change and the `P(x)` changes, then the joint distribution (`P(y & x)`) for both train & test will be different.

This post won’t discuss about covariate shift concepts, however.

A few days ago I came across this covariate shift issue in classification problem. Just wondering whether we could detect the occurrence of covariate shift automatically. Turns out that a simple technique is available for this task.

The technique leverages machine learning-based approach to detect whether covariate shift happens in a feature. In other words, we build a machine learning model to learn the characteristics of train & test data. Here’s how it works in general.

<ul>
<li>For each sample in the train & test data, add a new feature that states whether the sample comes from train or test data. Let’s call it as the <b>origin</b> feature</li>
<li>Merged both of train & test data (already consists of the <b>origin</b> feature)</li>
<li>Build a classifier for each feature (except the label) and the <b>origin</b> feature from the merged data</li>
<li>If the classifier is pretty good in classifying whether a sample belongs to the train or test data, then the feature might be shift</li>
</ul>

For the sake of clarity, take a look at the following example.

```
TRAIN DATA
=========

		feature_A	|	feature_B	
		---------------------------------
		100		|	10
		150		|	15
		180		|	20
		200		|	25
		---------------------------------
Median	|	165		|	17.5


TEST DATA
=========

		feature_A	|	feature_B
		---------------------------------
		700		|	10
		750		|	20
		780		|	18
		800		|	15
		---------------------------------
Median	|	765		|	16.5
```

Using the above example for the train & test data, let’s execute the algorithm.

<b>Step 1.</b> Add origin feature telling whether a sample comes from train or test distribution.

```
TRAIN DATA
=========

feature_A	|	feature_B	|	origin
------------------------------------------------------
100		|	10		|	train
150		|	15		|	train
180		|	20		|	train
200		|	25		|	train
------------------------------------------------------


TEST DATA
=========

feature_A	|	feature_B	|	origin
------------------------------------------------------
700		|	10		|	test
750		|	20		|	test
780		|	18		|	test
800		|	15		|	test
------------------------------------------------------
```

<b>Step 2.</b> Merged both the train & test data. Should be clear enough.

<b>Step 3.</b> Build a classifier trained on each feature and the origin label. In other words, the training data for the classifier should be look like the followings.

```
feature_A	|	origin
---------------------------------
100		|	train
150		|	train
180		|	train
200		|	train
700		|	test
750		|	test
780		|	test
800		|	test
---------------------------------

feature_B	|	origin
---------------------------------
10		|	train
15		|	train
20		|	train
25		|	train
10		|	test
20		|	test
18		|	test
15		|	test
---------------------------------
```

<b>Step 4.</b> Evaluate the classifier performance in separating between the train and test origin. Presume that we use Matthew Correlation Coefficient (MCC) as the metric. Basically, we can adjust the threshold to denote whether a feature is shift. Suppose the applied rule is when the MCC > 0.2, then the feature might be shift.

```
Suppose here are the MCC values for each feature.

feature_A	=	0.7
feature_B	=	0.1

Covariate shift in feature_A (MCC > 0.2)
```

Based on the evaluation results, covariate shift happens in <I>feature_A</I>. It should be clear enough by just looking at its median in the train & test data. However, <I>feature_B</I> does not experience a covariate shift although the median differs by one. I think we could leverage the value of MCC to denote the level of covariate shift (greater MCC denotes a more extreme shift).

Here comes the part that is important to consider as well.

Before performing covariate shift detection, we need to specify whether we would like to detect the distribution changes based on the raw data or transformed data (handling skewed distributions, normalisation, standardisation, etc.). It needs to be considered as the raw data might experience shift, while the transformed one might not (or vice versa). Both raw and transformed data might be detected as both shifted (or not), though.

In my case, I chose to perform the detection towards the raw data. In other words, only imputation was performed. The imputation was performed independently which means that the test data did not use the statistics of the train data for imputation (such as mean, median, etc.). This was done to eliminate the assumption that the test data came from the same distribution as the train data.
