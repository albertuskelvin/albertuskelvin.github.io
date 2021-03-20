---
title: 'Wallis Product for Pi with Integration: Want the Proof!'
date: 2020-05-21
permalink: /posts/2020/05/wallis-product-proof/
tags:
  - mathematics
  - calculus
  - wallis product
---

The Wallis' infinite product for pi states the following.

```
pi / 2 = PRODUCT(n=1 to inf) [(2n / (2n-1)) . (2n / (2n+1))]

OR

pi / 2 = (2/1).(2/3).(4/3).(4/5).(6/5).(6/7)...(2n/2n-1).(2n/2n+1)
```

In theory, we can prove the above with various ways. Two of them that I love are by leveraging the Euler's sine infinite product and the power of sine integration.

In this post, we're going to see how to prove the Wallis' product by evaluating the power of sine integration.

Know that the integral with which we're going to play is formed as the following.

```
I(n) = INTEGRAL(0 to pi) [sin(x)^n dx]

For the sake of clarity, sin(x)^n denotes the value of sin(x) multiplied n times.
```

Let's try to solve the above `I(n)`.

We'll leverage integration by parts to do so. Recall that `INTEGRAL [u dv] = uv - INTEGRAL [v du]`.

```
I(n) = INTEGRAL(0 to pi) [sin(x)^n dx]
I(n) = INTEGRAL(0 to pi) [sin(x)^(n-1) . sin(x) dx]

Let's take u = sin(x)^(n-1) and dv = sin(x) dx.

I(n) = [sin(x)^(n-1) . (-cos(x)](from 0 to pi) - INTEGRAL(0 to pi) [-cos(x) . (n-1) . sin(x)^(n-2) . cos(x) dx]

I(n) = 0 - INTEGRAL(0 to pi) [-cos(x)^2 . (n-1) . sin(x)^(n-2) dx]

I(n) = (n-1) . INTEGRAL(0 to pi) [cos(x)^2 . sin(x)^(n-2) dx]

Let's replace cos(x)^2 with 1 - sin(x)^2.

I(n) = (n-1) . INTEGRAL(0 to pi) [(1-sin(x)^2) . sin(x)^(n-2) dx]

I(n) = (n-1) . INTEGRAL(0 to pi) [sin(x)^(n-2) - sin(x)^n dx]

I(n) = (n-1) . [INTEGRAL(0 to pi) [sin(x)^(n-2) dx] - INTEGRAL(0 to pi) [sin(x)^n dx]]

I(n) = (n-1) . [I(n-2) - I(n)]

I(n) = [(n-1) / n] . I(n-2)
```

Well, we're not really solve the `I(n)` though. However, we got an interesting equation here.

Let's try another idea to get closer to what we're going to prove.

We know that `sin(x)^k <= sin(x)^(k-1)` and `sin(x)^k >= sin(x)^(k+1)`. Therefore, the following inequation holds.

```
sin(x)^(k+1) <= sin(x)^k <= sin(x)^(k-1)
```

Since integration doesn't modify the inequation, we can apply integration to the above without changing the inequality sign.

```
INTEGRAL(0 to pi) [sin(x)^(k+1) dx] <= INTEGRAL(0 to pi) [sin(x)^k dx] <= INTEGRAL(0 to pi) [sin(x)^(k-1) dx]

The above will yield to the following.

I(k+1) <= I(k) <= I(k-1)
```

Next, let's divide all of them with `I(k+1)` which returns the following.

```
1 <= I(k) / I(k+1) <= I(k-1) / I(k+1)
```

To play with it further, let's try to see the value of `I(k-1)` and `I(k+1)`.

```
I(k) = [(k-1) / k] . I(k-2)

I(k-1) = [(k-2) / (k-1)] . I(k-3)
I(k-3) = [(k-4) / (k-3)] . I(k-5)
I(k-5) = [(k-6) / (k-5)] . I(k-7)
I(k-7) = [(k-8) / (k-7)] . I(k-9)

and so on ...

So from the pattern above we got the following.

I(k-1) = [(k-2) / (k-1)] . [(k-4) / (k-3)] . [(k-6) / (k-5)] . [(k-8) / (k-7)] ... [(P) / (Q)]

Since the value of P and Q depend on whether k is even or odd, let's fix the value by setting k equals to even numbers.
In other words, k = 2n and this makes our recurrence possible to end at I(3) = 2/3 . I(1).

I(2n-1) =  [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] . [(2n-8) / (2n-7)] ... I(1)

The value of I(1) is INTEGRAL(0 to pi) [sin(x)^1 dx] which equals to 2.

With this new information, we can get the below products.

I(2n-1) =  [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] . [(2n-8) / (2n-7)] ... 2
```

Let's do the same with `I(k+1)`.

```
I(k) = [(k-1) / k] . I(k-2)

I(k+1) = [(k) / (k+1)] . I(k-1)
I(k-1) = [(k-2) / (k-1)] . I(k-3)
I(k-3) = [(k-4) / (k-3)] . I(k-5)
I(k-5) = [(k-6) / (k-5)] . I(k-7)

and so on...

So from the pattern above we got the following.

I(k+1) = [(k) / (k+1)] . [(k-2) / (k-1)] . [(k-4) / (k-3)] . [(k-6) / (k-5)] ... [(P) / (Q)]

Since the value of P and Q depend on whether k is even or odd, let's fix the value by setting k equals to even numbers.
In other words, k = 2n and this makes our recurrence possible to end at I(3) = 2/3 . I(1).

I(2n+1) =  [(2n) / (2n+1)] . [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] ... I(1).

The value of I(1) is INTEGRAL(0 to pi) [sin(x)^1 dx] which equals to 2.

With this new information, we can get the below products.

I(2n+1) =  [(2n) / (2n+1)] . [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] ... 2.
```

Let's back to our inequality of `1 <= I(k) / I(k+1) <= I(k-1) / I(k+1)` or now becomes `1 <= I(2n) / I(2n+1) <= I(2n-1) / I(2n+1)`.

With out calculation for `I(2n-1)` and `I(2n+1)`, let's compute the division.

```
I(2n-1) = [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] . [(2n-8) / (2n-7)] ... 2
-------   ------------------------------------------------------------------------------------
I(2n+1) = [(2n) / (2n+1)] . [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] . [(2n-6) / (2n-5)] ... 2

I(2n-1) =        1
-------   ---------------
I(2n+1)   [(2n) / (2n+1)]
```

Think of the following. If we take an extremely large value for `n` (limit to infinity), the above inequation will result in the following.

```
lim(n to inf) 1 = 1
lim(n to inf) [I(2n-1) / I(2n+1)] = lim(n to inf) [1 / [2n / (2n+1)]] = 1

Therefore, with n goes to infinity, our inequation becomes 1 <= I(2n) / I(2n+1) <= 1.
```

We got an interesting info here stating that as n approaches infinity, `1 <= I(2n) / I(2n+1) <= 1` holds.

Moreover, according to the squeeze theorem, the value of `I(2n) / I(2n+1)` must also be equal to 1.

Let's finish it.

```
Recall that I(k) = [(k-1) / k] . I(k-2)

I(2n) = [(2n-1)/2n] . I(2n-2)
I(2n-2) = [(2n-3)/(2n-2)] . I(2n-4)
I(2n-4) = [(2n-5)/(2n-4)] . I(2n-6)
I(2n-6) = [(2n-7)/(2n-6)] . I(2n-8)

and so on until we get I(2) = 1/2 . I(0).

Therefore, I(2n) = [(2n-1)/2n] . [(2n-3)/(2n-2)] . [(2n-5)/(2n-4)] . [(2n-7)/(2n-6)] ... [1/2] . I(0).

Since I(0) = pi, then I(2n) = [(2n-1)/2n] . [(2n-3)/(2n-2)] . [(2n-5)/(2n-4)] . [(2n-7)/(2n-6)] ... [1/2] . pi.
```

Continue with our `I(2n) = I(2n+1)`,

```
I(2n) = I(2n+1)

[(2n-1)/2n] . [(2n-3)/(2n-2)] . [(2n-5)/(2n-4)] ... [1/2] . pi = [(2n) / (2n+1)] . [(2n-2) / (2n-1)] . [(2n-4) / (2n-3)] ... [2/3] . 2

pi/2 = (2n.2n)/(2n-1)(2n+1) . (2n-2)(2n-2)/(2n-3)(2n-1) . (2n-4)(2n-4)/(2n-3)(2n-5) . (2n-6)(2n-6)/(2n-5)(2n-7) ... (2/1)(2/3)

pi/2 = (2n.2n)/(2n-1)(2n+1) . (2(n-1))(2(n-1))/(2(n-1)-1)(2(n-1)+1) . (2(n-2))(2(n-2))/(2(n-2)+1)(2(n-2)-1) ... (2/1)(2/3)

pi/2 = PRODUCT(n=1 to inf) [(2n / (2n-1)) . (2n / (2n+1))]
```

Q.E.D
