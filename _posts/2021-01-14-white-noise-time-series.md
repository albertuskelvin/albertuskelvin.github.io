---
title: 'White Noise Time Series'
date: 2021-01-14
permalink: /posts/2021/01/white-noise-time-series/
tags:
  - statistics
  - time series
  - white noise
---

White noise series has the following properties:
- Mean equals to zero
- Standard deviation is constant
- Correlation between lags (lag > 0) is close to zero (each autocorrelation lies within the bound which shows no statistically significant difference from zero)

One thing to note is that white noise is not predictable. This can be shown obviously from the property that the correlation between lags is close to zero.

The properties of white noise might be leveraged to evaluate whether the time series model is good enough to estimate the actual observations. Here's a simple way of performing such an evaluation.
- Know that the actual observation is the sum of signal and noise (`actual = signal + noise`). In this context, `signal` represents the time series model, while `noise` represents the difference between the actual and the predicted value.
- Extract the noise with `actual - signal`.
- From the extracted noise, check whether it's a white noise by leveraging the above properties.
- If the extracted noise is white noise, then the time series model has performed the job well in capturing the pattern.

Several ways of identifying whether a time series is a white noise:
- Visual check
- Global and local check (for mean and standard deviation)
- Plot the autocorrelation on a graph. The graph consists of the autocorrelation for each lag (> 0). If all of the autocorrelations reside within the bounds, then the third property is fulfilled. The positive and negative bounds for 5% level of significance are approximated with `2 / sqrt(T)` and `-2 / sqrt(T)` respectively where `T` is the length of the time series.
