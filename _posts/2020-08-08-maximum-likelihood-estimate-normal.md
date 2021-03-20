---
title: 'Maximum Likelihood Estimation - Normal Distribution'
date: 2020-08-08
permalink: /posts/2020/08/maximum-likelihood-estimation-normal-distribution/
tags:
  - statistics
  - machine learning
  - normal distribution
  - maximum likelihood estimation
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/07/maximum-likelihood-estimation/">post</a>, I mentioned about the basic concept of maximum likelihood estimation (MLE). Please visit the post if you need a refresher.

This time, we're going to look at how to apply MLE for a normal distribution. In other words, we'd like to find the best estimate for the normal distribution's parameters given a set of observations.

The implementation in Spark (Scala) can be found in my GitHub <a href="https://github.com/albertuskelvin/maximum-likelihood-estimator">repo</a>.

Recall that the probability density function (pdf) for normal distribution is as the following.

```
f(x;mean;stddev) = (1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x-mean)/stddev)^2)
```

Assumming that the observations (x1, x2, ..., xn) were collected i.i.d, then the likelihood function is given as the following.

```
L(params | x1, x2, ..., xn) = [(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x1-mean)/stddev)^2)] . [(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x2-mean)/stddev)^2)] ... [(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((xn-mean)/stddev)^2)]
```

The previous post already shows that taking the log of the likelihood function and operate on the log result will yield the same result.

For simplicity, we'll use natural log for this case.

```
ln(L(params | x1, x2, ..., xn)) = ln([(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x1-mean)/stddev)^2)] . [(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x2-mean)/stddev)^2)] ... [(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((xn-mean)/stddev)^2)])
```

Let's operate on the above equation.

```
ln(L(params | x1, x2, ..., xn)) = ln([(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x1-mean)/stddev)^2)]) + ln([(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((x2-mean)/stddev)^2)]) + ... + ln([(1/sqrt(2.pi.(stddev^2))) . exp(-1/2 * ((xn-mean)/stddev)^2)]))
```

Solving the R.H.S will yield the following.

```
ln([(1/sqrt(2.pi.(stddev^2)))) + ln(exp(-1/2 * ((x1-mean)/stddev)^2)]) + ln([(1/sqrt(2.pi.(stddev^2)))) + ln(exp(-1/2 * ((x2-mean)/stddev)^2)]) + ... + ln([(1/sqrt(2.pi.(stddev^2)))) + ln(exp(-1/2 * ((xn-mean)/stddev)^2)]))
```

Or simplified to the following.

```
ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((x1-mean)/stddev)^2 + ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((x2-mean)/stddev)^2 + ... + ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((xn-mean)/stddev)^2
```

Let's first find the MLE of mean by taking the derivative of the log-likelihood with respect to the mean.

```
((x1-mean)/stddev^2) + ((x2-mean)/stddev^2) + ... + ((xn-mean)/stddev^2)
```

Finally, set the above to zero to find the mean that gives the maximum likelihood.

```
((x1-mean)/stddev^2) + ((x2-mean)/stddev^2) + ... + ((xn-mean)/stddev^2) = 0

The numerator must be zero.

x1 + x2 + ... + xn - (n.mean) = 0

mean = (x1 + x2 + ... + xn) / n
```

Last but not least, let's do the same for the standard deviation (stddev).

Starting from the following.

```
ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((x1-mean)/stddev)^2 + ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((x2-mean)/stddev)^2 + ... + ln((1/sqrt(2.pi.(stddev^2)))) + -1/2 * ((xn-mean)/stddev)^2
```

To simplify the derivation, let's expand the above part.

```
ln(1) - ln(sqrt(2.pi.(stddev^2))) + -1/2 * ((x1-mean)/stddev)^2 + ln(1) - ln(sqrt(2.pi.(stddev^2))) + -1/2 * ((x2-mean)/stddev)^2 + ... + ln(1) - ln(sqrt(2.pi.(stddev^2))) + -1/2 * ((xn-mean)/stddev)^2

ln(1) - 1/2.ln(2.pi.stddev^2) + -1/2 * ((x1-mean)/stddev)^2 + ln(1) - 1/2.ln(2.pi.stddev^2) + -1/2 * ((x2-mean)/stddev)^2 + ... + ln(1) - 1/2.ln(2.pi.stddev^2) + -1/2 * ((xn-mean)/stddev)^2

ln(1) - 1/2.[ln(2)+ln(pi)+2ln(stddev)] + -1/2 * ((x1-mean)/stddev)^2 + ln(1) - 1/2.[ln(2)+ln(pi)+2ln(stddev)] + -1/2 * ((x2-mean)/stddev)^2 + ... + ln(1) - 1/2.[ln(2)+ln(pi)+2ln(stddev)] + -1/2 * ((xn-mean)/stddev)^2

ln(1) - 1/2.[ln(2)+ln(pi)] - ln(stddev) + -1/2 * ((x1-mean)/stddev)^2 + ln(1) - 1/2.[ln(2)+ln(pi)] - ln(stddev) + -1/2 * ((x2-mean)/stddev)^2 + ... + ln(1) - 1/2.[ln(2)+ln(pi)] - ln(stddev) + -1/2 * ((xn-mean)/stddev)^2
```

And we take the derivative with respect to stddev.

```
-1/stddev + (x1-mean)^2/stddev^3 - 1/stddev + (x2-mean)^2/stddev^3 + ... + -1/stddev + (x3-mean)^2/stddev^3
```

Setting the above to zero.

```
0 = -stddev^2/stddev^3 + (x1-mean)^2/stddev^3 - stddev^2/stddev + (x2-mean)^2/stddev^3 + ... + -stddev^2/stddev + (xn-mean)^2/stddev^3

0 = -(n.stddev^2) + (x1-mean)^2 + (x2-mean)^2 + ... + (xn-mean)^2

stddev^2 = 1/n . [(x1-mean)^2 + (x2-mean)^2 + ... + (xn-mean)^2]

stddev = sqrt(1/n . [(x1-mean)^2 + (x2-mean)^2 + ... + (xn-mean)^2])
```

Therefore, we conclude that the MLE of mean and stddev are represented with the followings.

```
mean = (x1 + x2 + ... + xn) / n
stddev = sqrt(1/n . [(x1-mean)^2 + (x2-mean)^2 + ... + (xn-mean)^2])
```

Seems obvious, yet we now know that we have proved the formula.
