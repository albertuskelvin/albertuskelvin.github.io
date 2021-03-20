---
title: "Euler's Pi for the Sum of Inverse Squares Proof"
date: 2020-05-29
permalink: /posts/2020/05/euler-pi-sum-of-inverse-squares-proof/
tags:
  - mathematics
  - euler
  - pi
---

Given an infinite series of inverse squares of the natural numbers, what is the sum?

```
X = 1/(1^2) + 1/(2^2) + 1/(3^2) + 1/(4^2) + ...

X = SUM(n=1 to inf) 1/(n^2)
```

The above is the Basel problem that asks for the precise sum of the inverse squares of the natural numbers. In addition, the problem also asks for the proof to the solution.

Leonhard Euler announced that the sum of the series was `(pi^2) / 6` in 1735.

In this post, we're going to see how Euler's approach to the above problem.

Basically, Euler attacks the problem by leveraging the Maclaurin series for sine function which is shown below.

```
sin x = x - (x^3)/(3!) + (x^5)/(5!) - (x^7)/(7!) + ...
```

Since the Maclaurin series is a polynomial, we could factorize it and end up with the roots such as `(x-p)(x-q)(x-r)...`.

Now, we know that the roots of sine function are 0, +/-pi, +/-2pi, +/-3pi, +/-4pi, and so on. In other words, we could get the product of roots such as the following.

```
sin x = it's Maclaurin series = x(x-pi)(x+pi)(x-2pi)(x+2pi)(x-3pi)(x+3pi)(x-4pi)(x+4pi)...
```

Simplify the above products.

```
sin x = it's Maclaurin series = x(x^2-pi^2)(x^2-4pi^2)(x^2-9pi^2)(x^2-16pi^2)...

OR

sin x = it's Maclaurin series = x(1-(x^2)/(pi^2))(1-(x^2)/(4pi^2))(1-(x^2)/(9pi^2))(1-(x^2)/(16pi^2))...
```

Here's the trick.

Let's divide both sides by `x`.

```
(sin x) / x = it's Maclaurin series / x = (1-(x^2)/(pi^2))(1-(x^2)/(4pi^2))(1-(x^2)/(9pi^2))(1-(x^2)/(16pi^2))...
```

Afterwards, we're gonna find out what the coefficient of `x^2` is. This is simply because its coefficient yields the series of inverse squares that we're going to prove.

It's pretty simple. Let's take a look at several of its coefficients.

```
(1 - (x^2)/(pi^2))(1 - (x^2)/(4pi^2))(1 - (x^2)/(9pi^2))...

(1 - (x^2)/(4pi^2) - (x^2)/(pi^2) + (x^4)/(4pi^4))(1 - (x^2)/(9pi^2))...

(1 - (x^2)/(9pi^2) - (x^2)/(4pi^2) + (x^4)/(36pi^2) - (x^2)/(pi^2) + (x^4)/(9pi^4) + (x^4)/(4pi^4) - (x^6)/(36pi^4))(1 - x^2/16pi^2) ...
```

Factor out the coefficients of `x^2` we'll get the following.

```
(-1/(pi^2) - 1/(4pi^2) - 1/(9pi^2) - 1/(16pi^2) + ...) (x^2)
```

Equates the above with the coefficient of `x^2` from the Maclaurin series (after being divided by `x`) which is `-1/3!` or `-1/6`.

```
-1/6 = (-1/(pi^2) - 1/(4pi^2) - 1/(9pi^2) - 1/(16pi^2) + ...)
```

And just solve the above equation.

```
-1/6 = (-1/(pi^2) - 1/(4pi^2) - 1/(9pi^2) - 1/(16pi^2) - ...)
-1/6 = (1/(pi^2))(-1/(1) - 1/(4) - 1/(9) - 1/(16) - ...)

1/6 = (1/(pi^2))(1/(1) + 1/(4) + 1/(9) + 1/(16) + ...)

(pi^2)/6 = (1/(1) + 1/(4) + 1/(9) + 1/(16) + ...)
(pi^2)/6 = (1/(1^2) + 1/(2^2) + 1/(3^2) + 1/(4^2) + ...)
```

Finally,

```
(pi^2)/6 = (1/(1^2) + 1/(2^2) + 1/(3^2) + 1/(4^2) + ...)

(pi^2)/6 = SUM(n=1 to inf) 1/(n^2)
```

Q.E.D
