---
title: 'Moment Generating Function'
date: 2020-08-25
permalink: /posts/2020/08/moment-generating-function/
tags:
  - statistics
  - maths
  - moments
  - moment generating function
---

As the name suggests, moment generating function (MGF) provides a function that generates moments, such as `E[X]`, `E[X^2]`, `E[X^3]`, and so forth.

MGF formula is given by the following.

```
MGFx(t) = E[e^(t.X)] where X is a random variable
```

Since `E[e^(t.X)]` simply means `(e^(t.x1)) . P(x1) + (e^(t.x2)) . P(x2) + (e^(t.x3)) . P(x3) + ... + (e^(t.xn)) . P(xn)`, we can generate the respective formula for discrete and continuous random variable, such as the following.

```
Discrete RV

MGFx(t) = SUM(i=1 to n) (e^(t.xi)) . P(xi)

Continuous RV

MGFx(t) = INTEGRAL(-inf to inf) (e^(t.x)) . PDF(x) dx
```

After performing the above `MGFx(t)` calculation, we can generate the moments by taking its derivative with respect to `t` and evaluated at `t = 0` like the following.

```
MGFx’(t) evaluated at t = 0 => E[X]
MGFx’’(t) evaluated at t = 0 => E[X^2]
MGFx’’’(t) evaluated at t = 0 => E[X^3]
.
.
.
```

## Proof

We’re gonna prove that taking the n-th derivative of the MGF (respect to `t` and evaluated at `t = 0`) results in `E[X^n]`.

Recall that the Taylor series of exponential function is given below.

```
e^x = SUM(n = 0 to inf) (x^n) / n! = 1 + x + (x^2) / 2! + (x^3) / 3! + ...
```

Applying the above Taylor series to `e^(t.X)` yields the following.

```
e^(t.X) = [1 + t.X + ((tX)^2) / 2! + ((tX)^3) / 3! + ...]
```

Taking the expected value of the above.

```
E[e^(t.X)] = E[1 + t.X + ((tX)^2) / 2! + ((tX)^3) / 3! + ...]

E[e^(t.X)] = 1 + t . E[X] + (t^2 / 2!) . E[X^2] + (t^3 / 3!) . E[X^3] + ...
```

We already have the MGF represented in the Taylor series of `e^(t.X)`.

Take a look at what happens when we take the derivatives with respect to `t`.

### 1st derivative

```
MGFx’(t) = E[X] + 2 . t / 2! . E[X^2] + 3 . t^2 / 3! . E[X^3] + ...
```

Evaluating the above at `t = 0` yields the following.

```
MGFx’(0) = E[X]
```

### 2nd derivative

```
MGFx’’(t) = 2 / 2! . E[X^2] + 3 . 2 . t / 3! . E[X^3] + ...
```

Evaluating the above at `t = 0` yields the following.

```
MGFx’’(0) = E[X^2]
```

### 3rd derivative

```
MGFx’’’(t) = 3 . 2 / 3! . E[X^3] + ...
```

Evaluating the above at `t = 0` yields the following.

```
MGFx’’’(0) = E[X^3]
```

### n-th derivative

```
MGFx^n(t) = n . (n-1) . (n-2) ... 1 / n! . E[X^n] + ...
```

Evaluating the above at `t = 0` yields the following.

```
MGFx^n(0) = E[X^n]
```

## Final Note

For centralized moment, such as `E[X-mu]`, `E[(X-mu)^2]`, `E[(X-mu)^3]` and so on, we can just replace the `X` in the MGF formula with `X-mu`, such as `E[e^(t.(X-mu))]`. The rests should be the same.
