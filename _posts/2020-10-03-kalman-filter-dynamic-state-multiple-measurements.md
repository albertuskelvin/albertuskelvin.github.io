---
title: 'Kalman Filter for Dynamic State & Multiple Measurements'
date: 2020-10-03
permalink: /posts/2020/10/kalman-filter-dynamic-state-multiple-measurements/
tags:
  - kalman filter
  - uncertainty
  - dynamic state
  - multiple measurements
  - control theory
  - statistics
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/09/kalman-filter-static-single-measurement/">post</a>, we discuss about the implementation of Kalman filter for static state (the true value of the object’s states are constant over time). In addition, the Kalman filter algorithm is applied to estimate single true value.

This time, we're going to look at how to apply the algorithm on the context of dynamic state (the true value of the object being measured is governed by a certain equation). To be more specific, the algorithm is used to estimate multiple true values of an object being measured.

# Code

You can find the implementation of Kalman filter in Python on my Github <a href="https://github.com/albertuskelvin/kalman-filter">repo</a>.

Feel free to send a pull request!

# How it works?

Suppose we're going to estimate the true value of <b>position</b> & <b>velocity</b> of a moving object in a single direction (x-axis).

Here are the general steps in applying Kalman filter.

<b>NB:</b> Variables with all capital letters denote matrix (ex: `VAR_NAME` refers to a matrix called `VAR_NAME`)

<i>For each measurement (observation) iteration, do the followings.</i>

## Calculate the predicted state estimate

Taking the initial estimate of position & velocity as the `PREVIOUS_STATE`, calculate the predicted state estimate by the following.

```
PREDICTED_STATE_ESTIMATE = STATE_MULTIPLIER * PREVIOUS_STATE 
                              + CONTROL_VARIABLE_MULTIPLIER * CONTROL_VARIABLE \
                              + STATE_PREDICTION_PROCESS_ERROR
```

## Calculate the predicted state covariance matrix

Taking the initial estimate covariance matrix as the `PREVIOUS_STATE_COVARIANCE_MATRIX`, calculate the predicted state of the covariance matrix by the following.

```                           
PREDICTED_STATE_COVARIANCE_MATRIX = STATE_MULTIPLIER * PREVIOUS_STATE_COVARIANCE_MATRIX * STATE_MULTIPLIER_transposed \
                                    + PREDICTED_STATE_COVARIANCE_MATRIX_PROCESS_ERROR
```

## Calculate the Kalman gain

```
OBSERVATION_ERRORS_COVARIANCE = [[(OBSERVATION_ERROR_POSITION)^2, 0.0] 
                                 [0.0, (OBSERVATION_ERROR_VELOCITY)^2)]]

TRANSFORMER_H = np.array([[1.0, 0.0], [0.0, 1.0]])

KALMAN_GAIN = (PREDICTED_STATE_COVARIANCE_MATRIX * TRANSFORMER_H_TRANSPOSED) \
               / ((TRANSFORMER_H * PREDICTED_STATE_COVARIANCE_MATRIX)) * TRANSFORMER_H_TRANSPOSED) \
               + OBSERVATION_ERRORS_COVARIANCE)
```

## Calculate observations where non-observation errors are included

```
TRANSFORMER_C = [[1.0, 0.0]
                 [0.0, 1.0]]

OBSERVATION_WITH_NON_OBS_ERRORS = (TRANSFORMER_C * OBSERVATION) + NEW_OBSERVATION_PROCESS_ERROR
```

## Calculate current state estimate

```
TRANSFORMER_H = [[1.0, 0.0]
                 [0.0, 1.0]]
                 
OBSERVATION_AND_PREDICTED_STATE_ESTIMATE_DIFF = OBSERVATION_WITH_NON_OBS_ERRORS - (TRANSFORMER_H * PREDICTED_STATE_ESTIMATE)

CURRENT_STATE_ESTIMATE = PREDICTED_STATE_ESTIMATE + (KALMAN_GAIN * OBSERVATION_AND_PREDICTED_STATE_ESTIMATE_DIFF)
```

## Calculate current state estimate covariance matrix

```
TRANSFORMER_H = [[1.0, 0.0]
                 [0.0, 1.0]]

I = [[1.0, 0.0]
     [0.0, 1.0]]

CURRENT_STATE_ESTIMATE_COVARIANCE_MATRIX = (I - (KALMAN_GAIN * TRANSFORMER_H)) * PREDICTED_STATE_COVARIANCE_MATRIX
```

---

Current state estimate & current state estimate covariance matrix becomes the previous state for the next measurement iteration.

The next measurement iteration follows the same steps as above.
