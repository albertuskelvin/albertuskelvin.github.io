---
title: "Vieta's Infinite Products Representation for Pi: Show Me the Proof!"
date: 2020-05-23
permalink: /posts/2020/05/vieta-product-for-pi-proof/
tags:
  - mathematics
  - vieta product
  - pi
  - infinity
---

Last time I wrote about the infinite products representation for pi that is regarded as the Wallis' product for pi.

Turns out that there's also another infinite products representation for pi called Vieta's product. 

The Vieta's products for pi was demonstrated before the Wallis' products. These are the two oldest infinite products representations for pi.

It's stated as the following.

```
2 / pi = [sqrt(2) / 2] . [sqrt(2 + sqrt(2)) / 2] . [sqrt(2 + sqrt(2 + sqrt(2))) / 2] ...
```

As usual, let's try to prove the above pi representation.

We'll start from the most basic trigonometric equation. I'm sure that you're familiar with the following equation.

```
sin(x) = 2 . sin(x/2) . cos(x/2)
```

Let's expand the `sin(q)` further using the same base formula knowing that `sin(x/2) = 2 . sin(x/4) . cos(x/4)`.

```
sin(x) = 2 . 2 . sin(x/4) . cos(x/4) . cos(x/2)
```

OK. Let's experiment more with sine expansion.

```
Know that sin(x/4) = 2 . sin(x/8) . cos(x/8)

sin(x) = 2 . 2 . 2 . sin(x/8) . cos(x/8) . cos(x/4) . cos(x/2)

Know that sin(x/8) = 2 . sin(x/16) . cos(x/16)

sin(x) = 2 . 2 . 2 . 2 . sin(x/16) . cos(x/16) . cos(x/8) . cos(x/4) . cos(x/2)

One more time.

Know that sin(x/16) = 2 . sin(x/32) . cos(x/32)

sin(x) = 2 . 2 . 2 . 2 . 2 . sin(x/32) . cos(x/32) . cos(x/16) . cos(x/8) . cos(x/4) . cos(x/2)
```

Now, where should we go now from there?

Since we've observed the pattern already, let's write the above `sin(x)` in a more compact form.

```
sin(x) = 2^(n) . sin(x / (2^n)) . PRODUCT(k=1 to n) cos(x / (2^k))
```

The above equation doesn't yield any interesting info. Let's take a look what happen when the value of n approaches infinity.

```
lim(n to inf) sin(x) = lim(n to inf) [ 2^(n) . sin(x / (2^n)) . PRODUCT(k=1 to n) cos(x / (2^k)) ]

sin(x) = lim(n to inf) [ 2^(n) . sin(x / (2^n)) ] . lim(n to inf) PRODUCT(k=1 to n) cos(x / (2^k))
```

Let's see what the result of this limit (n approaches infinity) is.

### Work on lim(n to inf) ( 2^(n) . sin(x / (2^n)) )

It's hard to evaluate the resulting value of this term. As n approaches infinity, `2^(n)` will become bigger and bigger and be considered as infinity. Similarly, the value of `sin(x/(2^n))` will approach to `sin(0) = 0` as n goes toward infinity.

Let's take a different route. How about representing the sine with its Taylor series evaluated at 0?

Recall that the Taylor series for sine at 0 is the following.

```
sin(x) = SUM(i=0 to inf) [(-1)^i / (2i + 1)!] . [x^(2i + 1)]
```

Withou further ado, let's find the limit!

```
Replace the sine with its MacLaurin series (Taylor series evaluated at 0).

sin(x/(2^n)) = SUM(i=0 to inf) [(-1)^i / (2i + 1)!] . [(x/(2^n))^(2i + 1)]

Expand the above series.

sin(x/(2^n)) = [ (1/(1!))(x/(2^n)) - (1/(3!))(x/(2^n))^3 + (1/(5!))(x/(2^n))^ 5 - (1/(7!))(x/(2^n))^7 + ... ]

Don't forget that we still have the `2^n` term. Let's include it into the series.

P = [ 2^(n) . sin(x/(2^n)) ]

P = (2^n)(1/(1!))(x/(2^n)) - (2^n)(1/(3!))(x/(2^n))^3 + (2^n)(1/(5!))(x/(2^n))^5 - (2^n)(1/(7!))(x/(2^n))^7 + ... ]
```

So, we already represent the sine with its Taylor series and include the `2^n` term into the series.

Now, let's take the limit.

```
lim(n to inf) P = lim(n to inf) (2^n)(1/(1!))(x/(2^n)) - (2^n)(1/(3!))(x/(2^n))^3 + (2^n)(1/(5!))(x/(2^n))^5 - (2^n)(1/(7!))(x/(2^n))^7 + ... ]

As you can see, the LHS will still be `P`. Meanwhile, the RHS will be evaluated as follows.

- lim(n to inf) (2^n)(1/(1!))(x/(2^n)) = x
- lim(n to inf) (2^n)(1/(3!))(x/(2^n))^3 approaches 0 because the denominator (2^(2n)) will become bigger and bigger.
- lim(n to inf) (2^n)(1/(5!))(x/(2^n))^5 approaches 0 because the denominator (2^(4n)) will become bigger and bigger.
- This concludes that the rest of the series will approach 0 as the denominator will become bigger and bigger.

This lefts us with the first term of the series which is x.
```

Hence, we got `lim(n to inf) [ 2^(n) . sin(x/(2^n)) ] = x`.

---

### Work on lim(n to inf) PRODUCT(k=1 to n) cos(x / (2^k))

This one is a bit easier.

Let's expand the products so that we could see the pattern.

```
cos(x/2) . cos(x/4) . cos(x/8) . cos(x/16) . cos(x/32) ... cos(x/2^n)
```

Based on the above, as `n` approaches to infinity, the `(x/2^n)` will approach 0. However, we can't determine a specific value for this products since there would be various results.

For instance, take `x = pi/4`. Then we compute the above infinite products as the following.

```
K1 = cos(pi/8) . cos(pi/16) . cos(pi/32) ... cos(0)
```

Next, take `x = 3pi/4` and do the same as above.

```
K2 = cos(3pi/8) . cos(3pi/16) . cos(3pi/32) ... cos(0)
```

From the above two calculations, it's obvious that `K1 != K2`.

Although there are certain cases where the value of cosine is zero, still this fact isn't enough to make us conclude that this products limit approaches a certain number. We just proved this by using `x = pi/4` and `x = 3pi/4`.

Therefore, we can safely write this part in the following form.

```
P = lim(n to inf) PRODUCT(k=1 to n) cos(x/(2^k))

P = PRODUCT(k=1 to inf) cos(x/(2^k))
```

---

### Continuing the Proof

Alright, so we already compute the limit for both terms. Here's the equation.

```
sin(x) = lim(n to inf) [ 2^(n) . sin(x / (2^n)) ] . lim(n to inf) PRODUCT(k=1 to n) cos(x / (2^k))

sin(x) = x . PRODUCT(k=1 to inf) cos(x / (2^k))
```

Recall that we have the half-angle formula as the following.

```
cos(x/2) = sqrt[(1 + cos(x)) / 2]
```

As the final work, let's take `x = pi / 2` and express the cosine with the half-angle formula.

```
k = 1 -> cos(x/2) = sqrt[(1 + cos(x)) / 2]
k = 2 -> cos(x/4) = cos((x/2)/2) = sqrt[(1 + cos(x/2)) / 2]
k = 3 -> cos(x/8) = cos((x/4)/2) = sqrt[(1 + cos(x/4)) / 2]
k = 4 -> cos(x/16) = cos((x/8)/2) = sqrt[(1 + cos(x/8)) / 2]

and so on...
```

As you can see from the above application of the half-angle formula, we got a recursive function.

For instance, the value of `cos(x/8)` in `k=4` can leverage the value of `cos(x/8)` calculated in `k=3`.

We got the following.

```
PRODUCT(k=1 to inf) sqrt( (1 + cos(x/(2^(k-1)))) / 2 )
```

Continuing with our original equation.

```
sin(pi / 2) = (pi / 2) . PRODUCT(k=1 to inf) sqrt[(1 + cos((pi / 2) / (2^(k-1)))) / 2]

2 / pi = PRODUCT(k=1 to inf) sqrt[(1 + cos((pi / 2)/(2^(k-1)))) / 2]

2 / pi = sqrt[(1 + cos(pi / 2)) / 2] . sqrt[(1 + cos(pi / 4)) / 2] . sqrt[(1 + cos(pi / 8)) / 2] . sqrt[(1 + cos(pi / 16)) / 2] ...
```

Lots of things to do since we're dealing with recurrence.

Let's see the recursive function in action for cos(pi/8).

```
2 / pi = sqrt[1/2] . sqrt[(1 + sqrt(2)/2) / 2] . sqrt[(1 + cos(pi/8)) / 2] ...
2 / pi = sqrt[1/2] . sqrt[(1 + sqrt(2)/2) / 2] . sqrt[(1 + sqrt[(1 + cos(pi/4)) / 2]) / 2] ...
2 / pi = sqrt[1/2] . sqrt[(1 + sqrt(2)/2) / 2] . sqrt[(1 + sqrt[(1 + sqrt[(1 + cos(pi/2)) / 2]) / 2]) / 2] ...
2 / pi = sqrt[1/2] . sqrt[(1 + sqrt(2)/2) / 2] . sqrt[(1 + sqrt[(1 + sqrt[(1 + sqrt[(1 + cos(pi)) / 2]) / 2]) / 2]) / 2] ...

2 / pi = sqrt[1/2] . sqrt[(1 + sqrt(2)/2) / 2] . sqrt[(1 + sqrt[(1 + sqrt[(1 + sqrt[(1 + -1) / 2]) / 2]) / 2]) / 2] ...

2 / pi = sqrt(2)/2 . sqrt[2 + sqrt(2)] / 2 . sqrt[2 + sqrt[2 + sqrt(2)]] / 2 ...
```

The last line shows the Vieta's product for pi.

```
2 / pi = ( sqrt(2) / 2 ) . ( sqrt[2 + sqrt(2)] / 2 ) . ( sqrt[2 + sqrt[2 + sqrt(2)]] / 2 ) ...
```

Q.E.D
