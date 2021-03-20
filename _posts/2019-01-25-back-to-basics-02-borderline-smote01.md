---
title: 'Borderline-SMOTE01'
date: 2019-01-25
permalink: /posts/2019/01/borderline-smote01/
tags:
  - machine learning
  - oversampling
  - borderline smote01
---

<b>Relevant Paper:</b> <a href="https://sci2s.ugr.es/keel/keel-dataset/pdfs/2005-Han-LNCS.pdf">Borderline-SMOTE: A New Over-Sampling Method in Imbalanced Data Sets Learning</a>

In this post I'm going to write about one of the variants of SMOTE algorithm - that is borderline-SMOTE01. You can find my previous post related to SMOTE <a href="https://albertuskelvin.github.io/posts/2019/01/smote-algorithm/">here</a>.

Basically, borderline-SMOTE01 was developed to improve the performance of SMOTE. This means that borderline-SMOTE01's core algorithm is derived from SMOTE's algorithm. The only difference lies on the source of the creation of synthetic samples. In SMOTE, synthetic samples are created from every original minority samples. Meanwhile, in borderline-SMOTE01, synthetic samples are created from minority samples resided on and nearby the borderline (we'll call them with borderline samples).

The primary rationale behind generating new samples from borderline samples is borderline samples are more crucial in the process of classification than the ones far from the borderline. In other words, borderline samples are more inclined to be misclassified. Therefore, it's important to strengthen the borderline samples.

Alright, let's delve into the algorithm!

<b>Step 1.</b> For each minority sample (let's call it as <b>min_s</b>), calculate its <b>m</b> nearest neighbors (including the majority samples). The number of majority samples in its m nearest neighbors is denoted by <b>maj_num</b> in which <b>0</b> <= <b>maj_num</b> <= <b>m</b>

<b>Step 2.</b> In this step we'll specify whether <b>min_s</b> is a borderline sample using the following conditional rules:

<b>I. </b>If <b>maj_num = m</b>, then all the nearest neighbors of <b>min_s</b> are majority samples. In this case, <b>min_s</b> is considered as noise and will not be processed in the following steps.

<b>II. </b>If <b>m</b> / 2 <= <b>maj_num</b> <= <b>m</b>, then the majority samples occupy more than 50% of the nearest neighbors of <b>min_s</b> (the number of majority samples is larger than the number of minority samples). In this case, <b>min_s</b> is proned to be misclassified and therefore considered to be included in the list of borderline samples (<b>borderline_samples</b>).

<b>III. </b>If 0 <= <b>maj_num</b> < <b>m</b> / 2, then <b>min_s</b> is not considered to be the borderline sample. In this case, min_s will not be processed further.

<b>Step 3.</b> For each minority sample (<b>b_sample</b>) in <b>borderline_samples</b>, do the followings:

a) Find its <b>k</b> nearest neighbors (only includes the minority samples)

b) Randomly select <b>x</b> nearest neighbors from its <b>k</b> nearest neighbors (<b>1</b> <= <b>x</b> <= <b>k</b>)

c) For each sample in the <b>x</b> nearest neighbors (<b>x_nearest</b>):

c.1) Calculate the difference between <b>b_sample</b> and <b>x_nearest</b> (<b>diff_b_sample_x</b>). The output is in the form of feature vector

c.2) Multiple <b>diff_b_sample_x</b> with a random number between 0 and 1. Suppose the output of this multiplication step is <b>multi_diff_randnum</b>

c.3) Add <b>multi_diff_randum</b> to <b>b_sample</b>


I hope this post might give you a basic understanding of borderline-SMOTE01 algorithm. Feel free to comment if you found any irrelevant or missing information.
