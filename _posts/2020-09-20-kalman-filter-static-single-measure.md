---
title: 'Kalman Filter for Static State & Single Measurement'
date: 2020-09-20
permalink: /posts/2020/09/kalman-filter-static-single-measurement/
tags:
  - kalman filter
  - statistics
  - control theory
  - static model
  - single measurement
  - uncertainty
---

Kalman filter is an iterative mathematical process applied on consecutive data inputs to quickly estimate the true value (position, velocity, weight, temperature, etc) of the object being measured, when the measured values contain random error or uncertainty.

In this post, we're going to look at how to implement Kalman filter in the context of static model where the true value of the object's states are constant over time. In addition, for simplicity, we'll only consider single measurement where there's only one object's state being measured.

## Code

You can find the Kalman filter implementation in Python on this <a href="https://github.com/albertuskelvin/kalman-filter">repo</a>.

## How it works?

Suppose that we receive the following inputs prior to performing Kalman filter:
- Initial estimate
- Initial estimate error
- Measurement error (<i>assumed to be constant over time</i>)

There are three main calculations:
- Kalman gain: `previous_estimate_error / (previous_estimate_error + measurement_error)`
- Current estimate: `previous_estimate + (kalman_gain * (measurement - previous_estimate))`
- Current estimate error: `(1 - kalman_gain) * previous_estimate_error`

For our example, suppose that we'd like to measure a temperature. Here are some basic information:

- The true temperature: `72`
- Initial estimate: `68`
- Initial estimate error: `2`
- Initial measurement: `75`
- Measurement error: `4`

The measured temperature values are `75`, `71`, `70`, and `74`.

We'd like to estimate the true temperature value based on the above data.

Performing Kalman filter calculation will yield the following results.

<table>
  <tr>
    <th>Time</th>
    <th>Measurement</th>
    <th>Measurement Error</th>
    <th>Estimate</th>
    <th>Estimate Error</th>
    <th>Kalman Gain</th>
  </tr>
  <tr>
    <td>t-1</td>
    <td>-</td>
    <td>-</td>
    <td>68</td>
    <td>2</td>
    <td>-</td>
  </tr>
  <tr>
    <td>t</td>
    <td>75</td>
    <td>4</td>
    <td>70.33</td>
    <td>1.33</td>
    <td>0.33</td>
  </tr>
  <tr>
    <td>t+1</td>
    <td>71</td>
    <td>4</td>
    <td>70.5</td>
    <td>1</td>
    <td>0.25</td>
  </tr>
  <tr>
    <td>t+2</td>
    <td>70</td>
    <td>4</td>
    <td>70.4</td>
    <td>0.8</td>
    <td>0.2</td>
  </tr>
  <tr>
    <td>t+3</td>
    <td>74</td>
    <td>4</td>
    <td>71</td>
    <td>0.66</td>
    <td>0.17</td>
  </tr>
</table>

According to the given data, the true value estimate is `71`.
