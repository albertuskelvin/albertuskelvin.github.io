---
title: "Proof of Wallis Product for Pi with Euler's Infinite Product for Sine"
date: 2020-05-22
permalink: /posts/2020/05/wallis-product-proof-with-euler-sine-infinite-product/
tags:
  - mathematics
  - calculus
  - wallis product
  - euler
---

In the previous <a href="https://albertuskelvin.github.io/posts/2020/05/wallis-product-proof/">post</a> I mentioned about how to demonstrate the Wallis product for pi by starting from the powered sine integration.

This time, we’re gonna see how to derive the Wallis product with Euler’s infinite product representation for sine function.

As a refresher, here’s the Wallis product for pi that we are trying to prove.

```
Wallis Product for Pi

pi / 2 = PRODUCT(n=1 to inf) [(2n / (2n - 1)) . (2n / (2n + 1))]
```

Recall that the infinite product representation for sine proposed by Euler is illustrated as the following.

```
sin(x) = x . PRODUCT(n = 1 to inf) [1 - (x^2 / (n^2 . pi^2))]
```

How we’re gonna leverage the above equation should be simple enough. Take `x = pi / 2` and work on the equation as demonstrated below.

```
x = pi / 2

sin(pi / 2) = pi / 2 . PRODUCT(n = 1 to inf) [1 - (pi^2 / 4) / (n^2 . pi^2)]
1 = pi / 2 . PRODUCT(n = 1 to inf) [1 - (pi^2 / 4) / (n^2 . pi^2)]

2 / pi = PRODUCT(n = 1 to inf) [1 - (pi^2 / 4) / (n^2 . pi^2)]

2 / pi = PRODUCT(n = 1 to inf) [1 - (1 / 4 n^2)]
2 / pi = PRODUCT(n = 1 to inf) [(4n^2 - 1) / 4 n^2]
```

Seems that we can rewrite the final line from the above illustration in terms of the roots. In other words, let’s modify `4n^2 - 1` to `(2n-1)(2n+1)` and modify `4n^2` to `(2n)^2`.

```
2 / pi = PRODUCT(n = 1 to inf) [(2n-1)(2n+1) / (2n)^2]
```

If we take a closer look to the RHS, we know that the value for each element (`2n-1`, `2n+1`, and `2n`) can never be zero. In other words, the RHS is invertible.

So the final equation would be like the following.

```
pi / 2 = 1 /  ( PRODUCT(n = 1 to inf) [(2n - 1)(2n + 1) / (2n)^2] )

pi / 2 = PRODUCT(n = 1 to inf) [(2n)^2 / (2n - 1)(2n + 1)]
```

Rewriting the above equation, we’ll get what we want.

```
pi / 2 = PRODUCT(n = 1 to inf) [(2n / (2n - 1)) . (2n / (2n + 1))]
```

Q.E.D
