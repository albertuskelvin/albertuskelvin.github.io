---
title: 'One-sample Z-test with p-value Approach'
date: 2020-08-15
permalink: /posts/2020/08/one-sample-z-test-with-p-value/
tags:
  - statistics
  - maths
  - z-test
  - z-score
  - statistical test
  - mean
  - p-value
---

One sample z-test is used to examine whether the difference between a population mean and a certain value is significant.

In this post, we’re going to look at how to perform this statistical test with p-value approach.

Recall that the test statistic is calculated with the following.

```
z-score = (xbar - mu) / (sigma / sqrt(n))

xbar: sample mean
mu: population mean
sigma: population standard deviation
n: number of observations in the sample
```

In addition, there are several notes to consider when applying this test, such as:

<ul>
<li>The population standard deviation is known. However, in some references, you might find that it can be estimated with the sample standard deviation when the sample size is more than 30</li>
<li>Data is independent and identically distributed</li>
<li>Data is normally distributed. However, to compare the means, a sample with size is more or equal to 30 is sufficient thanks to the Central Limit Theorem</li>
</ul>

Speaking of the testing steps, here’s the general ones with p-value approach:

<ul>
<li>Specify the null and alternate hypothesis</li>
<li>Decide whether to perform one-tailed test (left or right) or two-tailed test</li>
<li>Compute the test statistic</li>
<li>Compute the p-value based on the test statistic</li>
<li>Decide whether to reject or fail to reject the null hypothesis</li>
</ul>

## A Bit of Intuition Behind the Mean Difference Testing

Suppose that the commonly accepted fact is that the population mean of a random variable is <b>mu</b>.

We argue that the population mean has changed to a value that is <I>less than</I> <b>mu</b>.

To prove our argument, we basically <b>create a model under the assumption that the null hypothesis is true</b>. Afterwards, we perform a study by collecting a sample. We then perform a statistical test (still under the assumption) to examine whether the sample mean difference with the hypothesized mean is statistically significant.

First, let’s take a look at how we’d create the model.

<ul>
<li>Suppose we’re going to perform <b>S</b> studies</li>
<li>For each study, we collect a sample with a certain number of items. For clarity, suppose we collected <b>N</b> samples with <b>K</b> items in each sample</li>
<li>For each study, we calculate the mean of its sample. By doing so, we should have <b>N</b> <I>sample means</I> (could be the same or different with each other)</li>
<li>Plotting the above <b>N</b> <I>sample means</I> in a graph, we should have our sampling distribution of the mean. According to the Central Limit Theorem, with a large number of items in a sample (<b>K</b> >= 30), this sampling distribution should be in the form of a bell-curve</li>
</ul>

Since we <b>assume that the null hypothesis is true</b>, the resulting sampling distribution of mean should have <b>mu</b> as the mean. <I>This is what should happen to the model under our assumption about the null hypothesis</I>.

Suppose after performing a study, we got a sample <b>s</b> with <b>K</b> items. We calculate the mean of this sample <b>s</b> as <b>our_mu</b>.

We then see that the sample mean is <b>extremely less</b> than <b>mu</b>. In this case, let’s assume <b>our_mu</b> resides within the rejection area in the sampling distribution of the mean.

Under the assumption that the null hypothesis is true, we expect that our sample mean should have small deviation to the true mean (<b>mu</b>). In other words, the sample mean should be in the 95% or 99% (according to the confidence level) area of the sampling distribution of the mean.

Since <b>our_mu</b> is located within the rejection area, we may conclude that <b>there might be another distribution that fits our sample better</b>, such as a distribution whose true mean is near <b>our_mu</b>.

---

For simplicity, we’re going to perform the test at a 95% confidence level (or at 5% significance level).

## One-tailed Test

### Left-tailed Test

We perform this test category when the alternate hypothesis includes the “less than” sign. In other words, the following hypothesis statement is applied.

```
H0: mean = X
H1: mean < X

OR

H0: mean >= X
H1: mean < X
```

For the sake of clarity, we’re going to use the following problem for the demonstration.

```
The average weight of all residents in town XYZ is 168 lbs.
A nutritionist believes the true mean is LESS than that.
She measured the weight of 36 individuals and found the mean to be 165 lbs with a standard deviation of 4.

Is there enough evidence to discard the null hypothesis?
```

Let’s first define the null and alternate hypothesis.

```
H0: mu = 168
H1: mu < 168
```

And we also have the following information.

```
sample size = 36
sample mean = 165
sample stddev = 4
```

Since the sample size is more or equal to 30, we can use the sample stddev to estimate the population stddev. In addition, this sample size should be sufficient to produce a normal sampling distribution of the mean according to the CLT.

Next, we compute the test statistic (z-score) with the previous formula.

```
z-score = (165 - 168) / (4 / sqrt(36))
z-score = -3 / (4 / 6)
z-score = -18 / 4
z-score = -4.5
```

Therefore, our test statistic is `-4.5`.

Next, we need to find out the area of the curve at the <b>left</b> side of `-4.5`. This area will become the p-value.

According to the p-value, if it’s less than 0.05 (alpha), then we can reject the null hypothesis. Otherwise, we say that we failed to reject the null hypothesis.

### Right-tailed Test

Basically, the procedure for performing this test is quite similar to the one used for the left-tailed test.

The following hypothesis declaration is used for this right-tailed test.

```
H0: mean = X
H1: mean > X

OR

H0: mean <= X
H1: mean > X
```

Therefore, the problem statement is modified slightly to the following.

```
The average weight of all residents in town XYZ is 168 lbs.
A nutritionist believes the true mean is MORE than that.
She measured the weight of 36 individuals and found the mean to be 169 lbs with a standard deviation of 4.

Is there enough evidence to discard the null hypothesis?
```

Here, we may apply the same procedure to calculate the test statistic (z-score).

Based on the test statistic, we then need to find out the area of the curve at the <b>right</b> side of `-4.5`. This area will become the p-value.

According to the p-value, if it’s less than 0.05 (alpha), then we can reject the null hypothesis. Otherwise, we say that we failed to reject the null hypothesis.

## Two-tailed Test

Once again, the general procedure is basically similar to the one-tailed test

However, we need to divide the significance level with two because we now have two rejection areas (at the far left and right side of the curve).

The following hypothesis declaration is used for this two-tailed test.

```
H0: mean = X
H1: mean != X

NB:
“!=“ means “not equal to”
```

With the two-tailed test, the problem statement is modified slightly to the following.

```
The average weight of all residents in town XYZ is 168 lbs.
A nutritionist believes this average DOES NOT represent the true mean.
She measured the weight of 36 individuals and found the mean to be 165 lbs with a standard deviation of 4.

Is there enough evidence to discard the null hypothesis?
```

We can then proceed by calculating the z-score with the same formula, such as the following.

```
z-score = (165 - 168) / (4 / sqrt(36))
z-score = -3 / (4 / 6)
z-score = -18 / 4
z-score = -4.5
```

Since we’re performing two-tailed test, we now have two z-scores. One for the left-tail (-4.5) and the other one for the right-tail (+4.5).

Our next task is to compute the left side area of -4.5 and the right side area of +4.5. The sum of these areas become the p-value.

As the final conclusion, we reject the null hypothesis is the p-value is less than 0.5 (alpha). Otherwise, we failed to reject the null hypothesis.
