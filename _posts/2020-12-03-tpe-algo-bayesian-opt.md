---
title: 'Tree Parzen Estimator in Bayesian Optimization for Hyperparameter Tuning'
date: 2020-12-03
permalink: /posts/2020/12/tree-structured-parzen-estimator-bayesian-optimization/
tags:
  - machine learning
  - hyperparameter tuning
  - bayesian optimization
  - tree structured parzen estimator
---

One of the techniques in hyperparameter tuning is called Bayesian Optimization. It selects the next hyperparameter to evaluate based on the previous trials.

The basic idea is described by the followings:
- The objective function is expensive to call
- Should not call the objective function too often
- Estimate the objective function by a surrogate function
- Select the next hyperparameters to evaluate from the surrogate function
- Update the surrogate function based on the result of the objective function given the selected hyperparameters

There are two primary matters here:
- How to represent the surrogate function
- How to choose the next hyperparameter value that will likely yield an improvement to the objective function

In this post, we're going to look at those two matters.

## How the TPE Algorithm Works?

One of the algorithms for representing the surrogate function called Tree-structured Parzen Estimator (TPE) algorithm. This TPE algorithm is implemented on Hyperopt (a library for hyperparams tuning with bayesian optimization in Python).

Let's say we'd like to perform tuning for a single model hyperparameter so that the objective function is <b>minimized</b>. So far we've already evaluated several choices of the hyperparameter and got the values returned by the objective function.

This can be interpreted as the following.

```
hyperparameter | obj_func_return_value
======================================
     10        |         0.3
     10.5      |         0.5
     11        |         0.45
     14        |         0.35
     20        |         0.75
     24        |         0.8
     25        |         0.9
     30        |         0.85
```

Next, we define a threshold `T` to separate the "good" and "bad" hyperparameter values. Suppose we define `T` equals to 0.5. This means that the first group is those hyperparameter values that yield the objective function return value less than 0.5. Otherwise for the second group.

```
-- GOOD HYPERPARAMETER VALUES (l(x))

hyperparameter | obj_func_return_value
======================================
     10        |         0.3
     10.5      |         0.5
     11        |         0.45
     14        |         0.35
     
-- BAD HYPERPARAMETER VALUES (g(x))

hyperparameter | obj_func_return_value
======================================
     20        |         0.75
     24        |         0.8
     25        |         0.9
     30        |         0.85
```

Upon defining the threshold, we build a hyperparameter distribution for the "good" and "bad" hyperparameter values. We represent "good" ones with `l(x)` and "bad" ones with `g(x)`. Based on the sample hyperparameter values, we apply Kernel Density Estimation (KDE) to estimate the underlying distribution for `l(x)` and `g(x)`.

The point is that `l(x)` and `g(x)` represent the surrogate function.

---

## How to Choose the Next Hyperparameter Value that Will Likely Yield an Improvement to the Objective Function

After knowing the most updated surrogate function, our next task will be selecting the most convincing hyperparameter value. We leverage a function called the <i>acquisition function</i>. One of such a function is called Expected Improvement (EI).

I won't go into the details of EI in this post. However, the idea is basically simple. Based on the EI formula, it's known that we need to minimize `g(x) / l(x)` in order to maximize EI.

This is reasonable since we're more likely to choose hyperparameter values from `l(x)` rather than `g(x)` because we already know that values yielding the minimum cost reside in `l(x)`. Therefore, we'd like to maximize the value of `l(x) / g(x)`. In other words, we'll minimize the value of `g(x) / l(x)`.

---

Upon selecting the most convincing hyperparameter value, we call the objective function given the value. The value returned by the objective function is used to update `l(x)` and `g(x)`.
