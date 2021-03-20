---
title: 'Questions on Balanced Random Forest & Its Implementation'
date: 2019-03-15
permalink: /posts/2019/03/balanced-random-forest/
tags:
  - machine learning
  - balanced random forest
---

I came across a research paper related to balanced random forest for imbalanced data. For the sake of clarity, the following is the algorithm of BRF taken from the paper:

<pre>
1. For each iteration in random forest, draw a bootstrap sample from the minority class.
   Randomly draw the same number of cases, with replacement, from the majority class.
2. Induce a classification tree from the data to maximum size, without pruning. The tree is induced with the CART algorithm,
   with the following modification: At each node, instead of searching through all variables for the optimal split,
   only search through a set of <b>Mtry</b> randomly selected variables.
3. Repeat the two steps above for the number of times desired. Aggregate the predictions of the ensemble and make the
   final prediction.
</pre>

### Sampling with Replacement

A question popped up in my mind after finished reading the 1st point. So basically, we're doing random under-sampling in which the ratio of minority and majority class is 1:1. By bootstrapping, it simply means that we take some random samples with replacement. However, one thing to note here is when the number of minority class is much less than the number of majority class, e.g. 1:100000, we won't need to do bootstrapping at all (just take all the minority instances).

Then, I got confused by the 2nd statement on the same point. As you can see, it states that we need to do sampling on the majority class with replacement as well. For those who don't know yet about sampling with or without replacement, the concept is greatly simple. Sampling with replacement means that after you fetched a sample from a data pool, you put it back to the pool. Consequently, the probability of drawing a sample at a certain point of time is independent from the previous drawing. Vice versa, sampling without replacement simply means that we don't put the sample back to the pool. In other words, the probability of drawing a sample changes at every point of drawing time.

Back to the topic. By sampling with replacement, the consequence is very evident. Yup, the sampled data might contain duplicates. I'm still not sure with my own hypothesis, though. Probably my understanding is less accurate which means that what the author means by <i>with replacement</i> here might be different from my perspective. What do you think?

### Implementation

I was going to implement this algorithm using PySpark, yet the 2nd point made me thinking a little bit deeper. I thought that Spark ML's Decision Tree Classifier has a parameter for specifying the number of features used to select the best split at each node. But after searching through some references online, I didn't manage to find it. Unlike on Random Forest Classifier where we can specify the number of features via a param called <b>featureSubsetStrategy</b>, I didn't find the same thing on Decision Tree Classifier.

Well, I think it's funny because BRF implementation should use Decision Tree class at its core. To handle such a case, we could still use Random Forest by modifying its behavior such that it acts like a common decision tree. For instance, for each tree induction, we can set the number of trees to 1, impurity=gini, subsamplingrate=1, etc. What do you think?

---

So, what do you think? Any suggestion? And CMIIW.
