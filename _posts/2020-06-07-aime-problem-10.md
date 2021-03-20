---
title: "Vieta Triple Roots: 2019 American Invitational Mathematics Examination (AIME) I Problem 10"
date: 2020-06-07
permalink: /posts/2020/06/aime-2019-problem-10/
tags:
  - mathematics
  - vieta
  - aime
  - triple roots
---

Let's take a look at the problem statement.

---

<p>
  For distinct complex numbers <b>z1, z2, z3, . . ., z673</b> the polynomial

<br/><br/>

<b>(x - z1)^3 . (x - z2)^3 . (x - z3)^3 . . . (x - z673)^3</b>

<br/><br/>

can be expressed as <b>x^2019 + 20x^2018 + 19x^2017 + g(x)</b>, where <b>g(x)</b> is a polynomial with complex coefficients and with degree at most 2016.

<br/><br/>

The value of <b>abs(SUM[1 <= j < k <= 673] zj . zk)</b> can be expressed in the form <b>m / n</b> where <b>m</b> and <b>n</b> are relatively prime positive integers.

<br/><br/>

Find <b>m + n</b>.
</p>

---

The above problem is basically similar to its simpler version, that is `(x-r1)(x-r2)(x-r3)...(x-rn)` where `r1, r2, r3, ..., rn` are the roots. Therefore, we could leverage the same rule of the sum and product of the polynomial roots.

Recall that we have the following rule for polynomial roots and coefficient interaction.

```
Any general polynomial P(x) with degree n,

P(x) = An.x^n + An-1.x^n-1 + An-2.x^n-2 + ... + A0

where An, An-1, An-2, ..., A0 are real or complex coefficients and An != 0.

Such a polynomial have n (not necessarily distinct) complex roots r1, r2, r3, ..., rn 
according to the fundamental theorem of algebra.

The polynomial P(x) can also be rewritten as An(x-r1)(x-r2)(x-r3)...(x-rn).

Based on Vieta's formula, the relationship between the coefficients and the roots are stated as the followings.

r1 + r2 + r3 + ... + rn = -(An-1 / An)

(r1.r2 + r1.r3 + ... + r1.rn) + (r2.r3 + r2.r4 + ... + r2.rn) + ... + rn-1.rn = An-2 / An

.
.
.

r1.r2...rn = (-1)^n A0 / An
```

Let's apply the above information to our problem.

Recall that we can rewrite the polynomial to the following form.

```
P(x) = x^2019 + 20x^2018 + 19x^2017 + g(x) = (x-z1)(x-z1)(x-z1)(x-z2)(x-z2)(x-z2)...(x-z673)(x-z673)(x-z673)

where

r1 = z1; r2 = z1; r3 = z1
r4 = z2; r5 = z2; r6 = z2
r7 = z3; r8 = z3; r9 = z3
...
r2017 = z673; r2018 = z673; r2019 = z673
```

Based on the above formula, we know that

```
19 = (r1.r2 + r1.r3 + ... + r1.rn) + (r2.r3 + r2.r4 + ... + r2.rn) + ... + rn-1.rn

19 = (z1.z1 + z1.z1 + z1.z2 + z1.z2 + ... + z1.z673) + (z1.z1 + z1.z2 + ... + z1.z673) + ... + z673.z673
```

If you expand the above root products a bit, you may notice that the square of each root (e.g. `z1^2`, `z2^2`, ..., `zn^2`) occurs three times. Meanwhile, the product of roots with different index (e.g. `z1.z2`, `z1.z5`, etc) each occurs nine times.

Therefore, the above equation can be rewritten in the following form.

```
19 = 3.(z1.z1 + z2.z2 + z3.z3 + ... + zn.zn) + 9.(z1.z2 + z1.z3 + ... + zn-1.zn)
```

Note that the `(z1.z2 + z1.z3 + ... + zn-1.zn)` sum is the same expression as `abs(SUM(1<=j<k<=673) zj.zk)`.

For simplicity, let's denote `(z1.z1 + z2.z2 + z3.z3 + ... + zn.zn)` as <b>X</b> and `(z1.z2 + z1.z3 + ... + zn-1.zn)` as <b>Y</b>.

Thus, we have our first equation here.

```
19 = 3X + 9Y    ( * )
```

Now let's leverage another coefficient. Below is the equivalent expression for the coefficient of `x^2018`.

```
-20 = z1 + z1 + z1 + z2 + z2 + z2 + ... + zn + zn + zn

-20 = 3.(z1 + z2 + ... + zn)

-20/3 = z1 + z2 + ... + zn
```

See what happens if we take the square of the above equation?

```
(-20/3)^2 = (z1 + z2 + ... + zn)^2

400/9 = z1^2 + z2^2 + z3^2 + ... + zn^2 + 2.(z1.z2 + z1.z3 + ... + zn-1.zn)
```

It seems similar to what we've had already. The above equation can be rewritten as the following.

```
400/9 = X + 2Y    ( ** )
```

So we now have two equations and clearly we can solve for <b>Y</b>.

```
19     = 3X + 9Y
400/9  = X  + 2Y

19     = 3X + 9Y
400/3  = 3X + 6Y
--------------- (-)
-343/3 = 3Y

Y = -343/9
```

From the above, we know that `Y = (z1.z2 + z1.z3 + ... + zn-1.zn) = -343/9`.

Therefore, `abs(SUM(1<=j<k<=673) zj.zk)` should be equal to `abs(-343/9)` or 343/9.

The last line of the problem statement states that the `abs(SUM(1<=j<k<=673) zj.zk)` can be expressed as `m/n` where `m` and `n` are relatively prime. This simply means that `m` and `n` don't have common factors that divide both of them except 1. Well, that's true for 343 and 9.

Finally, `m + n` equals <b>352</b>.
