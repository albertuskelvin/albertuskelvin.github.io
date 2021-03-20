---
title: 'Maximum Likelihood Estimation'
date: 2020-07-24
permalink: /posts/2020/07/maximum-likelihood-estimation/
tags:
  - statistics
  - maximum likelihood estimation
  - mle
  - probability
  - distribution
---

If in the probability context we state that `P(x1, x2, ..., xn | params)` means the probability of getting a set of observations `x1`, `x2`, ..., and `xn` given the distribution parameters, then in the likelihood context we get the following.

<b>L(params | x1, x2, ..., xn)</b>

Which means that the likelihood for the distribution parameters given a set of observations`x1`, `x2` , ..., and `xn`.

In other words, in likelihood, a set of observations is fixed. Our task becomes finding the distribution parameters based on the given observations.

Maximum likelihood estimation simply means that we’re looking for the distribution parameters that yield the maximum likelihood for the given observations.

## Assumption

Applying MLE requires an assumption in which the observations were collected in independent and identically distributed (i.i.d) manner.

This assumption simplifies the generation of the likelihood function.

Given a set of observations (i.i.d), the likelihood of the distribution parameters is calculated as the following.

`L(params | x1, x2, x3, ..., xn)` = `P(x1, x2, x3, ..., xn | params)`

Since the observations are independent and identically distributed, we get the following.

`L(params | x1, x2, x3, ..., xn)` = `P(x1 | params)` . `P(x2 | params)` . `P(x3 | params)` ... `P(xn | params)`

## Maths

The MLEs are calculated by taking the derivative of the log-likelihood which respects to the parameter and setting it to zero.

Start with the likelihood function.

`L(params | x1, x2, x3, ..., xn)` = `P(x1 | params)` . `P(x2 | params)` . `P(x3 | params)` ... `P(xn | params)`

Generating the log-likelihood function.

`l` = `log(L(params | x1, x2, x3, ..., xn))` = `log(P(x1 | params) . P(x2 | params) . P(x3 | params) ... P(xn | params))`

`l` = `log(L(params | x1, x2, x3, ..., xn))` = `log(P(x1 | params))` + `log(P(x2 | params))` + `log(P(x3 | params))` + ... + `log(P(xn | params))`

Taking the derivative which respects to the parameter.

```
  dl   =  d(log(P(x1 | params))) + d(log(P(x2 | params))) + d(log(P(x3 | params))) + ... + d(log(P(xn | params)))
------    ----------------------   ----------------------   ----------------------         ----------------------
dparam           dparam                    dparam                   dparam                          dparam
```

Set the derivative to zero & create an equation for the param.

```
MLE of the param =    dl     =  0
                   --------
                    dparam
```

The derivation uses log-likelihood since besides it’s simpler to calculate, the resulting MLE is also the same with the one resulted from the original likelihood function.

Here’s a quick mathematical proof.

```
L = p(x,y) . q(x,y) . r(x,y)

L' =  dL  = d(p(x,y) . q(x,y) . r(x,y))
     ----   ---------------------------
      dx                dx
```

To find the MLE for `x`, the task becomes solving for `L' = 0`.

Now take the log of L.

```
log(L) = log(p(x,y) . q(x,y) . r(x,y))
```

Since `dlog(f(x)) / dx = f'(x) / [f(x) ln(10)]`, then

```
dlog(L) = L' / (L ln(10))
-------
  dx
```

To find the MLE for `x`, the task becomes solving for `L' / (L ln(10)) = 0`.

It simply states that `L' = 0` since the denominator can't be zero.
