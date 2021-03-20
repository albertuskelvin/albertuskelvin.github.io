---
title: 'Parzen Window Density Estimation - Kernel and Bandwidth'
date: 2020-12-11
permalink: /posts/2020/12/parzen-window-density-estimation/
tags:
  - machine learning
  - statistics
  - parzen window
  - density estimation
---

Imagine that you have some data `x1, x2, x3, ..., xn` originating from an unknown continuous distribution `f`. You'd like to estimate `f`.

There are several approaches of achieving the task. One of them is by estimating `f` with the empirical distribution. However, if the data is continuous, the empirical distribution might result in an uniform distribution since each data would most likely appear once, therefore has the same probability to occur.

Another approach is by bucketizing the data into certain intervals. However, this approach will yield some number of buckets (discrete distribution) rather than a continuous distribution.

The next approach is by leveraging what's called as Parzen Window Density Estimation. It's a nonparametric method for estimating the probability density function given the data. In addition, it's just another name for Kernel Density Estimation (KDE).

Basically, KDE estimates the true distribution by a mixture of kernels that are centered at `xi` and have bandwidth (scale) equals to `h`. Here's the formula of KDE.

```
f_hat(x) = (1/(n.h)) * SUM(i=1 to n) K((x - xi) / h)

K: kernel
n: number of samples
h: bandwidth
```

Kernel can be described as a probability density function that is used to estimate density of data located close to a datapoint `xi`. It could be any function though as long as the required properties are fulfilled. Those properties are the sum of probability (area under the PDF) is one, the function is symetric (K(-u) = K(u)) and the function is a non-negative one. Several kernel examples are gaussian, uniform, triangular, Epanechnikov, biweight, triweight, and so forth. If you browse them, you may notice that most of the kernel functions have a support of -1 to 1 (when the function is centered at 0).

So, why would we need to have kernel and bandwidth in the first place?

Since the datapoint `xi` is just a sample coming from a continuous distribution `f`, we might presume that there might be some unknown (nonzero) density around `xi`. In other words, there might be some datapoints from the population that are very close to `xi`. These close datapoints are represented with a density function called kernel.

However, how close the datapoints are from `xi` (because a term "close" is relative)? This is determined by the bandwidth (`h`). The following paragraph might describe this a bit clearer.

If you look at the formula of `f_hat(x)`, the term `K((x - xi) / h)` denotes that the kernel function is centered at `xi` and it calculates the value of PDF for point `x` and scaled by `h`. As I've mentioned before on the previous paragraphs, most of the kernel functions (centered at 0) have a range (support) from -1 to 1. If the kernel function is centered at 0, then the minimum and maximum data in the kernel function are -1 and 1 respectively. Now, if the kernel function is centered at `xi`, then the minimum and maximum data are `xi - 1` and `xi + 1` respectively. The same concept applies when a bandwidth `h` is implemented. The calculation goes as follows:
- min value: `(x - xi) / h = -1` which yields `x_min = xi - h`
- max value: `(x - xi) / h = 1` which yields `x_max = xi + h`

The above can be summarized as that the bandwidth `h` makes the range of the kernel function to `[xi - h, xi + h]`.

Using `f_hat(x)`, we might estimate the probability density of any point `x` by these following steps:
- Determine the bandwidth `h` that will be used
- For each sample data `xi`, calculate the probability density for `x` using kernel function centered at `xi`. Specifically, calculate `K((x - xi) / h)`. Let's denote the result as `PD(x, i)`
- Sum up `PD(x, i)` for `i = 1 to n` (`n` is the number of sample data)
