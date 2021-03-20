---
title: 'Gradient Boosting Algorithm for Regression Problem'
date: 2020-12-01
permalink: /posts/2020/12/gradient-boosting-regression/
tags:
  - machine learning
  - gradient boosting
  - ensemble
  - regression
---

In this post, we're going to look at how Gradient Boosting algorithm works in a regression problem.

Gradient Boosting is one of the boosting types along with Adaptive Boosting and Extreme Gradient Boosting (XGBoost).

The followings are the inputs required before applying the algorithm.
- training data
- differentiable loss function `L(y, F(x))` where `y` is the actual value and `F(x)` is the prediction value
- number of trees (`M`)

For the sake of clarity, let's use the following example instances as our training data (`y` is the dependent variable).

```
A  | B  | C  | y
=================
A1 | B1 | C1 | 50
A2 | B2 | C2 | 70
A3 | B3 | C3 | 30
```

Let's execute the algorithm.

---

## Initialize model with a constant value

As the very first step, we build a simple model that just returns a constant value `C`. This `C` is the prediction value where the loss function is minimum.

Formally, we'd like to solve for the following optimization problem.

```
F0(x) = argmin(C) SUM(i=1 to n) L(yi, C)
```

Let's perform this step on our training data.

Know that the loss function for an instance is defined as `L(yi, F(xi)) = 0.5 * (yi - F(xi))^2`. Or in other words, the sum of our loss function becomes the following.

```
sum_loss_func = L(y1, F(x1)) + L(y2, F(x2)) + L(y3, F(x3))
sum_loss_func = 0.5 * (y1 - F(x1))^2 + 0.5 * (y2 - F(x2))^2 + 0.5 * (y3 - F(x3))^2
```

Since the prediction values are the same over the train data, we can replace `F(x1), F(x2), F(x3)` to a constant of `C`. Plugging in the actual value to the above equation becomes the following.

```
sum_loss_func = 0.5 * (50 - C)^2 + 0.5 * (70 - C)^2 + 0.5 * (30 - C)^2
```

And solve for the optimization problem can be performed by taking the first derivative of `sum_loss_func` with respect to `C` and assign the equation to zero, such as the following.

```
sum_loss_func = 0.5 * (50 - C)^2 + 0.5 * (70 - C)^2 + 0.5 * (30 - C)^2

d(sum_loss_function) = -(50 - C) - (70 - C) - (30 - C) = -150 + 3C
--------------------
         dC

Assigning the result to zero yields 200 = 3C where C = 150 / 3 = 50.
```

Our initial model is constructed so that it always return 50 for every data point.

---

## Construct the decision tree regressor

In this step, we iterate `M` times (number of decision tree specified before) and perform the following actions.

### a) Compute so-called pseudo-residuals

For each instance in the training data, we compute the following.

```
residual_i_m = - [ dL(yi, F(xi)) / dF(xi) ] where F(x) = Fm-1(x) for i = 1, ..., n
```

If we take a look at the above `residual_i_m`, it's simply the difference between the actual and the predicted value. Here's how we'd derive it.

```
Know that L(y, F(x)) = 0.5 * (y - F(x))^2.

Taking the derivative of the loss function with respect to F(x) yields dL(y, F(x)) / dF(x) = -(y - F(x)).

Moving the minus sign to the LHS yields -dL(y, F(x)) / dF(x) = y - F(x).

Or to be more specific, we'll finally have -dL(y, Fm-1(x)) / dFm-1(x) = y - Fm-1(x).
```

The last line of the above snippet concludes that `residual_i_m` is the same as `yi - Fm-1(xi)` for an instance `i`.

Applying this step to our train data will yield the following result. In the below table, `residual_1` means the error values for the first tree regressor.

```
A  | B  | C  | y  | F0(x) | residual_1
========================================
A1 | B1 | C1 | 50 | 50    |    0
A2 | B2 | C2 | 70 | 50    |    20
A3 | B3 | C3 | 30 | 50    |    -20
```

### b) Fit a base learner to pseudo-residuals

In this step, we train a weak model (decision tree regressor) on the training data where `residual_i_m` becomes the dependent variable.

Let's call the model `h_m(x)`.

For instance, let's apply our `h_m(x)` to our previously training data. In this case, `h_m(x)` becomes `h_1(x)` since it's our first weak model.

Note that the values of `h_1(x)` are dummy values.

```
A  | B  | C  | y  | F0(x) | residual_1 | h_1(x)
===============================================
A1 | B1 | C1 | 50 | 50    |    0       |  0.5
A2 | B2 | C2 | 70 | 50    |    20      |  18
A3 | B3 | C3 | 30 | 50    |    -20     |  -18
```

### c) Compute multiplier for the weak model

In this step, we solve the following optimization problem. We'd like to find the multiplier (`gamma`) that minimizes the sum of loss functions.

```
gamma_min = argmin(gamma) SUM(i=1 to n) L(yi, Fm-1(xi) + gamma * h_m(xi))
```

Let's apply this step on our training data (the one with `h_m(x)` column).

```
sum_loss_func = L(y1, F0(x1) + gamma * h_1(x1)) + L(y2, F0(x2) + gamma * h_1(x2)) + L(y3, F0(x3) + gamma * h_1(x3))

sum_loss_func = 0.5 * (y1 - (F0(x1) + gamma * h_1(x1)))^2 \
                + 0.5 * (y2 - (F0(x2) + gamma * h_1(x2)))^2 \
                + 0.5 * (y3 - (F0(x3) + gamma * h_1(x3)))^2
                
sum_loss_func = 0.5 * (50 - (50 + gamma*0.5))^2 \
                + 0.5 * (70 - (50 + gamma*18))^2 \
                + 0.5 * (30 - (50 + gamma*(-18)))^2
                
sum_loss_func = 0.5 * (-gamma*0.5))^2 \
                + 0.5 * (20 + gamma*18)^2 \
                + 0.5 * (-20 + gamma*(-18))^2
```

Afterwards, let's take the first derivative of `sum_loss_func` with respect to `gamma` and set the equation to zero.

```
d(sum_loss_func) = (-gamma*0.5))*(-0.5) + (20 + gamma*18)*(18) + (-20 + gamma*(-18))*(-18)
----------------
    d(gamma)
    
d(sum_loss_func) = gamma*0.25 + 360 + gamma*324 + 360 + gamma*(324)
----------------
    d(gamma)
    
d(sum_loss_func) = gamma*648.25 + 720
----------------
    d(gamma)
    
Setting the above to zero yields -720 = 648.25*gamma, therefore gamma = -1.11
```

From the above calculation, we get `gamma_min` equals to `-1.11`.

### d) Update the model

In this step, we calculate the following.

```
Fm(x) = Fm-1(x) + gamma_min * h_m(x)
```

Applying the above formula to our training data, we'll calculate `F1(x) = F0(x) + gamma_min * h_1(x)` and get the following.

```
A  | B  | C  | y  | F0(x) | residual_1 | h_1(x) |        F1(x)
======================================================================
A1 | B1 | C1 | 50 | 50    |    0       |  0.5   | 50 + (-1.11) * 0.5
A2 | B2 | C2 | 70 | 50    |    20      |  18    | 50 + (-1.11) * 18
A3 | B3 | C3 | 30 | 50    |    -20     |  -18   | 50 + (-1.11) * (-18)
```

### e) Iterate process a) to d) until M base learners are constructed

---

## Output the final model

After constructing `M` base learners for pseudo-residuals, we finally come up with our final model, that is `FM(x)`.
