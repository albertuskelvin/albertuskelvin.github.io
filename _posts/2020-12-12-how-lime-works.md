---
title: 'Local Interpretable Model-Agnostic Explanations (LIME)'
date: 2020-12-13
permalink: /posts/2020/12/how-lime-works-for-tabular-data/
tags:
  - machine learning
  - lime
  - explainable ai
  - local
  - interpretable
  - model agnostic
  - explanation
  - tabular data
---

LIME is a python library used to explain predictions from any machine learning classifier.

It's short for Local Interpretable Model-Agnostic Explanations. Each term in the name denotes the characteristic we expect in an explanation.
- <b>Local</b>: it refers to local fidelity where we want to know how the classifier behaves around the instance being predicted
- <b>Interpretable</b>: the explanation is useless unless a human can make sense of it
- <b>Model-Agnostic</b>: LIME is able to explain any ML classifier without needing to make any assumption about the classifier

In this post, we're going to look at how LIME works for <b>tabular data</b> in a high-level overview. For more detail explanation, please refer to the following <a href="https://arxiv.org/pdf/1602.04938v1.pdf">paper</a>.

---

## How LIME Works

<b>Step 0.</b> Let the instance being explained is denoted as <b>X</b>. Also, suppose that we want to get explanation for a certain class <b>C</b>.

<b>Step 1.</b> Since we'd like to evaluate the local behavior of the model around <b>X</b>, we'll need to generate several samples both in the vicinity of <b>X</b> and far from <b>X</b>.

The sampling is performed basically by modifying the input variables. For instance, in a tabular data where the instance <b>X</b> consists of four variables, such as `0.5, 1.0, 3.5, 30`. We modify the variables resulting in some new instances, such as `0.5, 3.0, 3.5, 3.0`, `5.5, 1.0, 3.5, 30`, `10, 1.0, 7.7, 1.0`, and so on.

<b>Step 2.</b> After having the generated instances, we generate the probability of each instance belongs to class <b>C</b>. In other words, we call the classifier to output the probability (including <b>X</b>).

For instance, doing this step will yield the following example dataset. Let's call it as <b>Z</b>.

```
Dataset Z

var_a | var_b | var_c | var_d | proba_C
=======================================
0.5   | 1.0   | 3.5   | 30    | 0.9
0.5   | 3.0   | 3.5   | 3.0   | 0.8
5.5   | 1.0   | 3.5   | 30    | 0.75
10    | 1.0   | 7.7   | 1.0   | 0.7
. . .
. . .
```

<b>Step 3.</b> To strengthen instances residing close to <b>X</b>, we assign a weight to each instance (`wi`). Bigger weight simply denotes that the instance is closer to <b>X</b>.

<b>Step 4.</b> We train a linear model with weighted least squares. To be specific, for a given set of weights `w1, w2, ..., wn` (where `n` is the number of instances) we want to find coefficients `b0, b1, b2, ..., bk` so as to minimize the following (sum of errors).

```
SUM(i=1 to n) wi * (yi - y_hat_i)^2
```

<b>Step 5.</b> We got the linear model with its coefficients `b0, b1, b2, ..., bk`. These coefficients basically demonstrate how the original classifier behaves with respect to instances around <b>X</b>.

---

## How to Interpret the Generated Linear Model

The linar model's coefficients provide information regarding the correlation between each predictor (`var_a, var_b, var_c, var_d`) and the class probability (`proba_c`).

Positive coefficient simply means that the predictor has a positive correlation with the dependent variable. On the other hand, negative coefficient means that the predictor has a negative correlation with the dependent variable.

For instance, suppose our linear model is `var_a + (0.5 * var_b) + (3.5 * var_c) - (0.03 * var_d)`. From the coefficients, we know that `var_a`, `var_b` and `var_c` are positively correlated with `proba_c`. Meanwhile, `var_d` is negatively correlated with `proba_c`.

Knowing the coefficients helps a lot in evaluating why the model yield a certain probability for the class. The reason is that we know how much each variable contributes to the class probability (in the context of explanation performed around the instance being explained).

For instance, an instance <b>P</b> with variables `0.5, 1.0, 3.0, 353` with the yielded probability of `0.9`. The question would be why such variables yield a probability of 0.9?

Without knowing the coefficients, we couldn't know how much each variable contributes. For instance, does `var_a = 0.5` contributes 1.8 times and the rest contribute 0 times? If so, then we conclude that around the instance, our original model only considers `var_a` in computing the probability.

How about `var_a = 0.5` contributes 3 times and `var_c = 3.0` contributes -0.2 times? 

We need to know the contribution from each variable to the end result of `0.9`. Using the above linear model (`var_a + (0.5 * var_b) + (3.5 * var_c) - (0.03 * var_d)`), we know that such an instance yield a probability of 0.9 because `var_a = 0.5`, `var_b = 1.0`, `var_c = 3.0`, and `var_d = 353` contribute 1 time, 0.5 times, 3.5 times, and -0.03 times respectively to the probability.

In other terms, we know that <i>around instance</i> <b>P</b>, the original model tends to consider `var_a, var_b, var_c` and `var_d` to contribute `1, 0.5, 3.5` and `-0.03` times respectively to the class probability.

---

## Drawbacks

LIME has several drawbacks, such as:

<b>Point 1.</b> The process of generating the explanation for each instance might take a great amount of time. It happens because for each instance LIME needs to generate samples around the instance, call the original model to predict the class probability, and train a local linear model.

Another point related to the speed matter is that the current LIME version doesn't support multiprocessing. I've mentioned this issue on a previous post <a href="https://albertuskelvin.github.io/posts/2020/01/pseudo-distributed-lime-spark-udf/">here</a>. In that post, I wrote about my exploration on how to perform a pseudo-distributed LIME via PySpark UDF.

<b>Point 2.</b> Since the samples around the instance are generated randomly, the generated coefficients might be different each time an explanation is generated for the same instance.

<b>Point 3.</b> The trained local linear model might not fit the perturbed instances well. It could be improved, however, through enough hyperparameter tuning. Consequently, the explanation generation process might take a longer time.
