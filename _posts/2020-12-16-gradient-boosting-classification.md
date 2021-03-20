---
title: 'Gradient Boosting Algorithm for Classification Problem'
date: 2020-12-17
permalink: /posts/2020/12/gradient-boosting-for-classification/
tags:
  - machine learning
  - gradient boosting
  - classification
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/12/gradient-boosting-regression/">post</a> I mentioned about how Gradient Boosting algorithm works for a regression problem.

This time, we're gonna look at how the same algorithm works for a classification problem.

The followings are the inputs required before applying the algorithm.
- training data
- differentiable loss function `L(y, F(x))` where `y` is the actual value and `F(x)` is the prediction value
- number of trees (`M`)

For the sake of clarity, let’s use the following example instances as our training data (`y` is the dependent variable).

```
A  | B  | C  | y
=================
A1 | B1 | C1 | P
A2 | B2 | C2 | P
A3 | B3 | C3 | Q
```

Let’s execute the algorithm.

---

## Initialize the loss function

In this demonstration, we'll use the following as the loss function.

```
L(yi, F(xi)) = -yi * log(odds) + log(1 + e^log(odds))

where odds = F(xi) / (1 - F(xi)) and F(xi) is the predicted probability for an instance i
```

---

## Initialize model with a constant value

As the very first step, we build a simple model that just returns a constant probability `C`. We'd like to search for a constant `C` that minimizes the sum of loss function.

Formally, we’d like to solve for the following optimization problem.

```
F0(x) = argmin(C) SUM(i=1 to n) L(yi, C)
```

Since the loss function is expressed in terms of `log(odds)`, we could define `C` equals to the predicted `log(odds)` rather than the predicted probability.

Let's expand the above equation by plugging in the loss function. Note that each of them will have the same `odds` and `log(odds)` for the base model.

```
L(y1, C) = -1 * log(odds) + log(1 + e^log(odds))
L(y2, C) = -1 * log(odds) + log(1 + e^log(odds))
L(y3, C) = -0 * log(odds) + log(1 + e^log(odds))

SUM = L(y1, C) + L(y2, C) + L(y3, C)
```

Searching for `C` that minimizes `SUM` may be accomplished by taking the derivative of `SUM` with respect to `log(odds)` and set it to zero.

```
dL(y1, C)/d(log(odds)) = -1 + (e^log(odds) / (1 + e^log(odds)))
dL(y2, C)/d(log(odds)) = -1 + (e^log(odds) / (1 + e^log(odds)))
dL(y3, C)/d(log(odds)) = -0 + (e^log(odds) / (1 + e^log(odds)))

Let p = e^log(odds) / (1 + e^log(odds))

Sum of the above derivatives are -1 + p + -1 + p + p = 3p - 2

Setting the sum to zero will yield p = 2/3 or log(odds) = ln(2)
```

So, our base model will return `ln(2)` for the `log(odds)` of each instance.

---

## Construct the decision tree regressor

In this step, we iterate `M` times (number of decision tree specified before) and perform the following actions.

### a) Compute the pseudo-residuals

For each instance in the training data, we compute the following.

```
residual_i_m = - [ dL(yi, F(xi)) / dF(xi) ] where F(x) = Fm-1(x) for i = 1, ..., n
```

Let's take a look at what `- [ dL(yi, F(xi)) / dF(xi) ]` really means.

```
Know that L(yi, F(xi)) = -yi * log(odds) + log(1 + e^log(odds))

And notice that F(xi) is represented by log(odds).

Taking the derivative of the loss function with respect to log(odds) yields dL(yi, F(xi)) / d(log(odds)) = -yi + (e^log(odds) / (1 + e^log(odds))).

Moving the minus sign to the LHS yields -dL(y, F(x)) / d(log(odds)) = yi - (e^log(odds) / (1 + e^log(odds))).

Or to be more specific, since F(x) = Fm-1(x), then our residual becomes -dL(y, Fm-1(x)) / d(log(odds)) = yi - (e^log(odds) / (1 + e^log(odds))) where we use log(odds) for Fm-1(x).
```

Applying this step to our train data will yield the following result. In the below table, `residual_1` means the error values for the first tree regressor.

```
A  | B  | C  | y  | F0(x) | residual_1
============================================================================
A1 | B1 | C1 | P  | ln(2) | 1-(e^ln(2) / (1 + e^ln(2))) = 1-(2/(1+2)) = 1/3
A2 | B2 | C2 | P  | ln(2) | 1-(e^ln(2) / (1 + e^ln(2))) = 1-(2/(1+2)) = 1/3
A3 | B3 | C3 | Q  | ln(2) | 0-(e^ln(2) / (1 + e^ln(2))) = 0-(2/(1+2)) = -2/3
```

---

### b) Fit a base learner to the pseudo-residuals

In this step, we train a weak model (decision tree regressor) on the training data where `residual_i_m` becomes the dependent variable.

Let’s call the model `h_m(x)`.

For instance, let’s apply our `h_m(x)` to our previously training data. In this case, `h_m(x)` becomes `h_1(x)` since it’s our first weak model.

Note that the values of `h_1(x)` are dummy values.

```
A  | B  | C  | y  | F0(x) | residual_1 | h_1(x)
================================================
A1 | B1 | C1 | P  | ln(2) |    1/3     |  0.5
A2 | B2 | C2 | P  | ln(2) |    1/3     |  0.3
A3 | B3 | C3 | Q  | ln(2) |   -2/3     |  0.3
```

---

### c) Compute multiplier for the weak model

In this step, we solve the following optimization problem. We’d like to find the multiplier (`gamma`) that minimizes the sum of loss functions.

```
gamma_min = argmin(gamma) SUM(i=1 to n) L(yi, Fm-1(xi) + gamma * h_m(xi))
```

Let’s apply this step on our training data (the one with `h_m(x)` column).

```
Know that L(yi, F(xi)) = -yi * log(odds) + log(1 + e^log(odds))

And notice that F(xi) is represented by log(odds).

sum_loss_func = L(y1, F0(x1) + gamma * h_1(x1)) + L(y2, F0(x2) + gamma * h_1(x2)) + L(y3, F0(x3) + gamma * h_1(x3))

sum_loss_func = -y1 * (F0(x1) + gamma * h_1(x1)) + log(1 + e^(F0(x1) + gamma * h_1(x1))) \
                + -y2 * (F0(x2) + gamma * h_1(x2)) + log(1 + e^(F0(x2) + gamma * h_1(x2))) \
                + -y3 * (F0(x3) + gamma * h_1(x3)) + log(1 + e^(F0(x3) + gamma * h_1(x3)))
                
sum_loss_func = -1 * (ln(2) + gamma * 0.5) + log(1 + e^(ln(2) + gamma * 0.5)) \
                + -1 * (ln(2) + gamma * 0.3) + log(1 + e^(ln(2) + gamma * 0.3)) \
                + -0 * (ln(2) + gamma * 0.3) + log(1 + e^(ln(2) + gamma * 0.3))
                
sum_loss_func = -(ln(2) + gamma * 0.5) + log(1 + e^(ln(2) + gamma * 0.5)) \
                + -(ln(2) + gamma * 0.3) + log(1 + e^(ln(2) + gamma * 0.3)) \
                + log(1 + e^(ln(2) + gamma * 0.3))
```

Afterwards, let’s take the first derivative of `sum_loss_func` with respect to `gamma` and set the equation to zero.

```
d(sum_loss_func) = {-0.5 + [0.5 * (e^(ln(2) + gamma * 0.5)) / (1 + e^(ln(2) + gamma * 0.5))]} \ 
----------------   + {-0.3 + [0.3 * (e^(ln(2) + gamma * 0.3)) / (1 + e^(ln(2) + gamma * 0.3))]} \
    d(gamma)       + {[0.3 * (e^(ln(2) + gamma * 0.3)) / (1 + e^(ln(2) + gamma * 0.3))]}
```
    
Setting the above to zero yields means that the numerator is zero and the denumerator can't be zero.

```
d(sum_loss_func) = {-0.8 + [0.5 * (e^(ln(2) + gamma * 0.5)) / (1 + e^(ln(2) + gamma * 0.5))]} \ 
----------------   + {[0.3 * (e^(ln(2) + gamma * 0.3)) / (1 + e^(ln(2) + gamma * 0.3))]} \
    d(gamma)       + {[0.3 * (e^(ln(2) + gamma * 0.3)) / (1 + e^(ln(2) + gamma * 0.3))]}

0 = [-0.8 * {(1 + e^(ln(2) + gamma * 0.5))} * {(1 + e^(ln(2) + gamma * 0.3))}] + {0.5 * (e^(ln(2) + gamma * 0.5))} * {(1 + e^(ln(2) + gamma * 0.3))} \ 
+ {0.6 * (e^(ln(2) + gamma * 0.3))} * {(1 + e^(ln(2) + gamma * 0.5))}

0 = [-0.8 - 0.8*(e^ln(2) * e^(gamma * 0.5)) * {1 + (e^ln(2) * e^(gamma * 0.3))}] + {0.5 * e^ln(2) * e^(gamma * 0.5)} * {1 + (e^ln(2) * e^(gamma * 0.3))} \
+ {0.6 * e^ln(2) * e^(gamma * 0.3)} * {1 + (e^ln(2) * e^+(gamma * 0.5))}

...
...
```

Well, as you can see, finding the `gamma` by taking the derivative of the sum of loss function with respect to `gamma` is hard.

Fortunately, we can approximate the loss function with the second order Taylor polynomials shown below.

```
Let p = gamma * h_m(xi),

L(yi, Fm-1(xi) + p) ~ L(yi, Fm-1(xi)) + p * d(yi, Fm-1(xi))/dF() + 0.5 * p^2 * d^2(yi, Fm-1(xi))/dF()^2
```

Now, let's apply the above approximation on each instance, sum them up, take the derivative with respect to `gamma`, and set the equation to zero.

```
p = gamma * h_1(x1)
q = gamma * h_1(x2)
r = gamma * h_1(x3)

L(y1, F0(x1) + p) ~ L(y1, F0(x1)) + [p * d(y1, F0(x1))/dF()] + [0.5 * p^2 * d^2(y1, F0(x1))/dF()^2]
L(y2, F0(x2) + q) ~ L(y2, F0(x2)) + [q * d(y2, F0(x2))/dF()] + [0.5 * q^2 * d^2(y2, F0(x2))/dF()^2]
L(y3, F0(x3) + r) ~ L(y3, F0(x3)) + [r * d(y3, F0(x3))/dF()] + [0.5 * r^2 * d^2(y3, F0(x3))/dF()^2]

sum_loss_func = L(y1, F0(x1)) + [p * d(y1, F0(x1))/dF()] + [0.5 * p^2 * d^2(y1, F0(x1))/dF()^2] \
                + L(y2, F0(x2)) + [q * d(y2, F0(x2))/dF()] + [0.5 * q^2 * d^2(y2, F0(x2))/dF()^2] \
                + L(y3, F0(x3)) + [r * d(y3, F0(x3))/dF()] + [0.5 * r^2 * d^2(y3, F0(x3))/dF()^2]

d(sum_loss_func) / d(gamma) = [h_1(x1) * d(y1, F0(x1))/dF()] + [(gamma * h_1(x1)^2) * d^2(y1, F0(x1))/dF()^2] \
                              + [h_1(x2) * d(y2, F0(x2))/dF()] + [(gamma * h_1(x2)^2) * d^2(y2, F0(x2))/dF()^2] \
                              + [h_1(x3) * d(y3, F0(x3))/dF()] + [(gamma * h_1(x3)^2) * d^2(y3, F0(x3))/dF()^2]

d(sum_loss_func) / d(gamma) = [h_1(x1) * (-y1 + (e^log(odds) / (1 + e^log(odds))))] \
                              + [h_1(x2) * (-y2 + (e^log(odds) / (1 + e^log(odds))))] \
                              + [h_1(x3) * (-y3 + (e^log(odds) / (1 + e^log(odds))))] \
                              + [gamma * h_1(x1)^2 * e^log(odds) / (1 + e^log(odds))^2] \
                              + [gamma * h_1(x2)^2 * e^log(odds) / (1 + e^log(odds))^2] \
                              + [gamma * h_1(x3)^2 * e^log(odds) / (1 + e^log(odds))^2]

For simplicity, let's denote e^log(odds) / (1 + e^log(odds)) as the predicted probability of each instance.

d(sum_loss_func) / d(gamma) = [h_1(x1) * (-y1 + p1)] \
                              + [h_1(x2) * (-y2 + p2)] \
                              + [h_1(x3) * (-y3 + p3)] \
                              + [gamma * h_1(x1)^2 * (p1 * (1 - p1))] \
                              + [gamma * h_1(x2)^2 * (p2 * (1 - p2))] \
                              + [gamma * h_1(x3)^2 * (p3 * (1 - p3))]
```

Let's set the above equation to zero to get the optimal `gamma`.

```
0 = [h_1(x1) * (-y1 + p1)] \
    + [h_1(x2) * (-y2 + p2)] \
    + [h_1(x3) * (-y3 + p3)] \
    + [gamma * h_1(x1)^2 * (p1 * (1 - p1))] \
    + [gamma * h_1(x2)^2 * (p2 * (1 - p2))] \
    + [gamma * h_1(x3)^2 * (p3 * (1 - p3))]
    
-[h_1(x1) * (-y1 + p1)] \
-[h_1(x2) * (-y2 + p2)] \
-[h_1(x3) * (-y3 + p3)] = [gamma * h_1(x1)^2 * (p1 * (1 - p1))] \
                          + [gamma * h_1(x2)^2 * (p2 * (1 - p2))] \
                          + [gamma * h_1(x3)^2 * (p3 * (1 - p3))]
```

Since we're building the first tree (`m` = 1), then the predicted `log(odds)` is `F0(x)` or the constant value for the base model-that is `ln(2)`.

Converting the predicted `log(odds)` to the predicted probability is achieved by the following.

```
p = e^log(odds) / (1 + e^log(odds))
p = e^ln(2) / (1 + e^ln(2))
p = 2 / 3
```

Since `F0(x)` is the predicted `log(odds)` from the base model for each instance, then `p1, p2, p3` will all have the same value, namely 2/3.

Plugging in the value to the above equation yields the following.

```
Recall that h_1(x1) = 0.5, h_1(x2) = 0.3 and h_1(x3) = 0.3

-[0.5 * (-1 + 2/3)] \
-[0.3 * (-1 + 2/3)] \
-[0.3 * (0 + 2/3)] = [gamma * 0.5^2 * (2/3 * (1 - 2/3))] \
                          + [gamma * 0.3^2 * (2/3 * (1 - 2/3))] \
                          + [gamma * 0.3^2 * (2/3 * (1 - 2/3))]

1/15 = 43/450 * gamma

gamma = 30/43
```

Finally, we got the optimal `gamma` equals to 30/43.

---

### d) Update the model

In this step, we calculate the following.

```
Fm(x) = Fm-1(x) + gamma_min * h_m(x)

where gamma_min is the gamma calculated on the previous step.
```

Applying the above formula to our training data, we’ll calculate `F1(x) = F0(x) + gamma_min * h_1(x)` and get the following.

```
A  | B  | C  | y  | F0(x) | residual_1 | h_1(x) |        F1(x)
=======================================================================
A1 | B1 | C1 | P  | ln(2) |    1/3     |  0.5   | ln(2) + (30/43) * 0.5
A2 | B2 | C2 | P  | ln(2) |    1/3     |  0.3   | ln(2) + (30/43) * 0.3
A3 | B3 | C3 | Q  | ln(2) |   -2/3     |  0.3   | ln(2) + (30/43) * 0.3
```

---

### e) Iterate process a) to d) until M base learners are constructed

---

## Output the final model

After constructing `M` base learners for pseudo-residuals, we finally come up with our final model, that is `FM(x)`.
