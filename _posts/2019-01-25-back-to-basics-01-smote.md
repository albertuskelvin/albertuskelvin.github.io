---
title: 'Synthetic Minority Over-sampling Technique (SMOTE)'
date: 2019-01-25
permalink: /posts/2019/01/smote-algorithm/
tags:
  - machine learning
  - oversampling
  - smote
---

First article in 2019.

In this post I'm going to write about one of the most popular sampling techniques in machine learning - that is SMOTE. Hope I can explain the concept in simple language.

Note: I presume that the readers of this post are quite familiar with the concept of imbalanced data.

Sampling in machine learning is used to overcome the problem of imbalanced data. Specifically, sampling has two general categories, namely under-sampling and over-sampling. In under-sampling, we make the amount of majority class to be similar to the amount of minority class. The simplest approach to perform under-sampling would be by randomly selecting instances from the majority class. Nevertheless, the most obvious drawback from this approach is we might lose important or specific information which in fact build the pattern of the majority class.

On the other hand, in over-sampling we augment the number of minority instances with the result that its amount is quite similar to the amount of samples in the majority class. The simplest approach would be by replicating the samples in the minority class. Replicating the samples might result in overfitting.

Synthetic Minority Over-sampling Technique (SMOTE) is classified as over-sampling technique. It's been one of the most popular over-sampling algorithms due to its implementation simplicity. Because of the same reason, many other over-sampling techniques are built based on this SMOTE algorithm. One of the examples is SMOTE-borderline. I'm gonna write about this SMOTE's variant in the next article.

Now, let's take a deep dive into SMOTE's algorithm.

For each minority sample (<b>ms</b>), do the followings:

1. Find <b>k</b> nearest minority samples. For instance, <b>k</b> = 3 would yield result as in the below figure. The figure shows that sample <b>ms</b> = A has sample B, C, D as the first, second, and third nearest sample respectively.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/back-to-basics-01-fig01.png?raw=true" alt="k-nearest neighbors of sample A" />

<b>Fig. 1. k-nearest neighbors of sample A</b>

<br/>

2. Based on the amount of oversampling <b>N</b> (e.g. <b>N</b> = 2 means that you want to synthesize 2 new samples), select <b>N</b> samples from the <b>k</b> samples randomly. For <b>N</b> = 2, the result is shown in the figure below. The selection of sample B and D was performed randomly.

<img src="https://github.com/albertusk95/albertusk95.github.io/blob/master/images/posts/back-to-basics-01-fig02.png?raw=true" alt="Take N random samples from the k-nearest neighbors" />

<b>Fig. 2. Take N random samples from the k-nearest neighbors</b>

<br/>

3. For each sample in <b>N</b> (let's say <b>n</b>), generate the new sample between <b>ms</b> and <b>n</b> using the following steps:

a. Compute the distance of feature vector between <b>ms</b> and <b>n</b>. Let's store the result in <b>diffFV(ms, n)</b>. The result of this step should be in the form of a feature vector. Based on the Fig. 2, the computation of feature vector distance is:

<i>diffFV(A, B) = feature_vec(B) - feature_vec(A)</i><br/>
<i>diffFV(A, D) = feature_vec(D) - feature_vec(A)</i><br/>
<i>diffFV(A, B) = feature_vec(B) - feature_vec(A)</i><br/>
<i>diffFV(A, D) = feature_vec(D) - feature_vec(A)</i><br/>

b. Multiply the <b>diffFV(ms, n)</b> with a random number between 0 and 1. This ensures that the new sample lies between <b>ms</b> and <b>n</b>. Let's assume that the output of this step is <b>diffFV_mul(ms, n)</b>

<i>diffFV_mul(A, B) = diffFV(A, B) * rand(0, 1)</i><br/>
<i>diffFV_mul(A, D) = diffFV(A, D) * rand(0, 1)</i><br/>

c. Add <b>diffFV_mul(ms, n)</b> to <b>ms</b>

<i>new_sample_between(A, B) = feature_vec(A) + diffFV_mul(A, B)</i><br/>
<i>new_sample_between(A, D) = feature_vec(A) + diffFV_mul(A, D)</i><br/>

d. Done

<br/>

Although this article only shows the high-level overview of SMOTE, I hope this brief review may help those who've been trying to grasp the concept of SMOTE.

Feel free to comment if you found any irrelevant or missing information.
