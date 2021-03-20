---
title: 'Standard Error of Mean Estimate Derivation'
date: 2020-05-06
permalink: /posts/2020/05/standard-error-of-mean-derivation/
tags:
  - machine learning
  - statistics
  - standard error of mean
---

Suppose we conduct <b>K</b> experiments on a kind of measurement. On each experiment, we take <b>N</b> observations. In other words, we’ll have <b>N * K</b> data at the end.

Let’s take a look at the following illustration.

```
Experiment 1
H11, H12, H13, ..., H1n

Experiment 2
H21, H22, H23, ..., H2n

. . .

Experiment K
HK1, HK2, HK3, ..., HKn
```

Afterwards, we calculate the mean of each experiment illustrated as the following.

```
Experiment Means

E(experiment 1) = SUM(i=1 to n) H1i / n
E(experiment 2) = SUM(i=1 to n) H2i / n

. . .

E(experiment k) = SUM(i=1 to n) Hki / n
```

We got the mean of each experiment. Plot the means on a graph and we got what’s called as sampling distribution. In other words, we collect several observations in each experiment (call it a sample) and the average of each experiment denotes the distribution of the sample means.

Standard error of the mean is simply the <b>standard deviation of the above experiment means data</b>. It tells us how precise our estimated mean from the population mean. Small standard error of the mean indicates that our estimated mean is close to the population mean.

The problem is, conducting a lot of experiments (with a number of observations) might require huge resources. We can, fortunately, estimate the standard error of the mean with the following formula.

```
SE(mean) = standard deviation of population / sqrt(number of experiments)
```

In this post, we’re going to look how the above estimate is derived.

## Standard Error of Mean (Estimated) Derivation

Before taking a deep dive into the maths, let’s clarify several things.

<b>Case.</b> We perform <b>K</b> experiments in which we take <b>N</b> observations from each experiment. This means that we should have <b>N * K</b> data at the end.

<b>Quick Analysis.</b> Based on the above case, we may say that within each experiment, we have a random variable <b>EXPERIMENT</b> which denotes the <b>N</b> samples of a certain kind of measurement. Well, it also implies that we’ll have <b>K</b> random variables of <b>EXPERIMENT</b>.

After we calculate the mean of every experiment, we can also say that we got a random variable <b>EXPECTED_EXP</b> which denotes the <b>K</b> experiment means.

Note that in this case, in each experiment, we collect observations from the same distribution. Moreover, each experiment was conducted independently. Therefore, we can conclude that our <b>N</b> random variables of <b>EXPERIMENT</b> are independently and identically distributed. Furthermore, we can also conclude that all the <b>N</b> random variables of <b>EXPERIMENT</b> have the same mean and variance (identically distributed).

<b>Derivation.</b> So, our objective is to calculate the standard deviation of the sample means. We will start from the variance formula.

```
Var(EXPECTED_EXP) = Var(SUM(i=1 to k) EXPERIMENT(i) / k)

where k = number of experiments
```

Know that `Var(mX) = m^2 Var(X)` for a constant <b>m</b> and `Var(X + Y) = Var(X) + Var(Y)` for random variables <b>X</b> and <b>Y</b>.

```
Var(EXPECTED_EXP) = (1/k^2) * [Var(SUM(i=1 to k) EXPERIMENT(i))]
Var(EXPECTED_EXP) = (1/k^2) * [Var(EXPERIMENT(1) + EXPERIMENT(2) + ... + EXPERIMENT(k))]
Var(EXPECTED_EXP) = (1/k^2) * [Var(EXPERIMENT(1)) + Var(EXPERIMENT(2)) + ... + Var(EXPERIMENT(k))]

Var(EXPECTED_EXP) = (1/k^2) * [SUM(i=1 to k) Var(EXPERIMENT(i))]
```

Since all the experiments have the same variance, we could rewrite the above to the following.

```
Var(EXPECTED_EXP) = (1/k^2) * (k) * Var(EXPERIMENT)
Var(EXPECTED_EXP) = (1/k) * Var(EXPERIMENT)
```

Hence, the standard deviation would be the square root of the above variance.

```
SE(mean) = sqrt(Var(EXPECTED_EXP))
SE(mean) = sqrt(Var(EXPERIMENT)) / sqrt(k)

SE(mean) = stdev(EXPERIMENT) / sqrt(k)
```

From the final formula, we can see that the standard error of the mean consists of two elements, namely the standard deviation of the random variable <b>EXPERIMENT</b> and the number of experiments (samples).

This formula simply shows that with the same standard deviation for all the random variable <b>EXPERIMENT</b>, conducting more experiments should reduce the standard error of the mean. Consequently, the estimated mean is said to be closer to the true mean.
