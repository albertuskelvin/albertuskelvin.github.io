---
title: 'Sample Size is Matter for Mean Difference Testing'
date: 2020-09-08
permalink: /posts/2020/09/sample-size-hypothesis-testing/
tags:
  - statistics
  - hypothesis testing
  - sample size
  - z-test
---

It's quite bothering when reading a publication that only provides a "statistically significant" result without telling much about the analysis prior to conducting the experiment.

I think on of the most critical points is how the sample size was determined.

Let's take a look at an example.

A scientist performs mean difference testing. She'd like to examine whether the population mean inferred from a sample is different enough from the hypothesized population mean.

Let's say that two studies (<b>A</b> & <b>B</b>) were performed. She collected a sample with <b>M</b> items for study <b>A</b>, while <b>N</b> items for study <b>B</b>. Suppose that <b>M < N</b>.

In addition, just assume, once again, the sample mean for both studies were the same.

After performing a one-sample Z-test, she found that the result from study <b>B</b> is statistically significant, while the one from study <b>A</b> is not.

For the same sample mean and hypothesized mean, both studies are different in terms of the significance of the mean difference.

This can be explained mathematically as follows.

The statistical test (one-sample Z-test in this example) has the sample size as one of its variables.

Recall that the Z-score is evaluated by dividing the difference between the sample mean and the hypothesized mean (<b>xbar - mu</b>) with the standard error of the mean (<b>stddev_pop / sqrt(sample_size)</b>).

When the sample size increases, the standard error becomes smaller. Consequently, smaller standard error yields a larger Z-score. Furthermore, this large Z-score will reside closer to the corner of the sampling distribution of the mean (unlikely observation area).

Applying the above reasoning on our example, the Z-score for study <b>B</b> is located closer to the distribution corner than the Z-score for study <b>A</b>. Or in other words, the p-value for study <b>B</b> is less than the one for study <b>A</b>.

The important part is that the sample size is matter. It's one of the variables that should be taken into consideration prior to conducting the study. Power analysis is a common technique used to address such an issue.
