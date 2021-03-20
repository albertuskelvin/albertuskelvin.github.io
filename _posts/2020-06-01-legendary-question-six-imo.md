---
title: "The Legendary Question Six IMO 1988"
date: 2020-06-01
permalink: /posts/2020/06/the-legendary-question-six-imo-proof/
tags:
  - mathematics
  - number theory
  - imo
  - olympiad
  - legendary question
  - contradiction
---

The final problem of the International Mathematics Olympiad (IMO) 1988 is considered to be the most difficult problem on the contest.

Arthur Engel wrote the following about the problem's difficulty:

<i>
Nobody of the six members of the Australian problem committee could solve it. Two of the members were husband and wife George and Esther Szekeres, both famous problem solvers and problem creators. Since it was a number theoretic problem it was sent to the four most renowned Australian number theorists. They were asked to work on it for six hours. None of them could solve it in this time. The problem committee submitted it to the jury of the XXIX IMO marked with a double asterisk, which meant a superhard problem, possibly too hard to pose. After a long discussion, the jury finally had the courage to choose it as the last problem of the competition. Eleven students gave perfect solutions.
</i>

<br/>

## Problem

<h4>
Let <i>a</i> and <i>b</i> be positive integers such that <i>(ab + 1)</i> divides <i>(a^2 + b^2)</i>.
<br/><br/>
Show that <i>(a^2 + b^2) / (ab + 1)</i> is the square of an integer.
</h4>

<br/>

Alright, let's try to solve the above problem.

## Intro

First, let's denote that `k = (a^2 + b^2) / (ab + 1)    ( * )`.

We know that `a` and `b` are positive integers (`a > 0` and `b > 0`). Consequently, if `k` is an integer, then it must be positive (`k > 0`).

From the problem statement, we know that `k` is not always an integer (e.g. `a` = 3 and `b` = 1). If so, we can ignore this case.

## Proof

We're going to prove the statement with proof by contradiction approach.

This means that we need to make an assumption that the statement we're going to prove is false. Recall that the original statement is <i><b>k is the square of an integer</b></i>.

Therefore, our assumption would be <i><b>k is NOT the square of an integer</b></i>.

In other words, we're trying to see what would happen if we work under such an assumption.

### a & b

Since we assume that the solutions to `( * )` where `k` is not the square of an integer are exists, we may take the smallest value for each of `a` and `b` from the list of solutions to `( * )`.

Let's assume that `a1` and `b1` are the <b>smallest</b> solution to `( * )` where `k` is <b>not</b> the square of an integer. This also means that `a1 + b1` should yield the minimum value compared with other pairs of `a` and `b`.

### Rearrange the Equation

Let's rearrange `( * )` into a more usual form.

```
a^2 - kba + b^2 - k = 0
```

Well, the above is a quadratic function.

We know that a polynomial equation can be expressed in terms of its root products such as the following.

```
f(x) = a.x^n + b.x^(n-1) + c.x^(n-2) + ... + constant = a(x-p)(x-q)(x-r)...

where p, q, r, etc. are the roots (f(x) = 0).
```

Same as the above, if we consider `a` as the variable and the others fixed, then our quadratic can be expressed as `(a - p)(a - q)`.

According to the rule of sum and product of polynomial roots, we know the followings.

```
Adding the roots yields -b/a.

Multiplying the roots yields:

* z/a (for even degree polynomials)
* -z/a (for odd degree polynomials)

Where:
* "a" is the coefficient of x^n.
* "b" is the coefficient of x^(n-1)
* "z" is the constant at the end
```

So, we can apply the above rules into our quadratic (even degree polynomial).

Recall that our quadratics is factored into `(a - p)(a - q)` which means that the roots are `p` and `q`.

Previously we also assume that `( * )` has `a1` and `b1` as the smallest solution. Since the quadratic is formed by considering `a` as the variable (the others are fixed), we may say that `a1` is one of the roots of the quadratic. Consequently, the other root `a2` can be calculated by leveraging the above rules.

```
f(a) = a^2 - (k.b1)a + (b1)^2 - k

The roots of f(a) are "a1" and "a2".

From the above rules, we know that:

a1 . a2 = ((b1)^2 - k)
a1 + a2 = (k.b1)

Therefore,

a2 = ((b1)^2 - k) / a1    ( ** )

OR

a2 = (k.b1) - a1    ( *** )
```

Now, let's check several characteristics of `a2`.

From `( ** )` we know that `a2` will never be zero because `k` is <b>not</b> the square of an integer. In other words, `k` will never be the same as `b^2`.

From `( *** )` we know that `a2` is an integer because `k, b1,` and `a1` are all integers.

Up til now we know that `a2` is an integer and not zero. However, is it a positive or negative integer?

To find out the sign of integer, let's back to our original formula `( * )`.

```
Since a2 is also the solution to (*), we'll get the following.

k = ((a2)^2 + (b1)^2) / ((a2.b1) + 1)
```

In the <b>Intro</b> section, we know that `k` is a positive integer (`k > 0`).

We also know that the numerator (`( (a2)^2 + (b1)^2 )`) will always be positive.

In addition, `b1` in the denominator is a positive integer. To be consistent with the fact that `k > 0`, we know for sure that `a2` must always be positive so that the denominator yields positive value as well.

In conclusion, we finally know that `a2` is a positive integer.

### Meet the Contradiction

Alright, the next question would be what's the relationship between `a1` and `a2`?

Let's assume that `a1 >= b1`.

If we square both sides, the inequality will still hold. So we got `(a1)^2 >= (b1)^2`.

Now, observe that `(b1)^2 > (b1)^2 - k` since `k > 0`. Consequently, `(a1)^2 > (b1)^2 - k`.

If we divide both sides by `a1`, the inequality will still hold. So we got `a1 > [(b1)^2 - k] / a1`.

Recall that from `( ** )`, `((b1)^2 - k) / a1` equals `a2`.

Therefore, we know that `a1 > a2    ( **** )`.

It's weird here.

Initially we assumed that `a1` and `b1` are the <b>smallest</b> solution to `( * )`. This consequently makes `a1 + b1` as the minimum value.

From `( **** )` we know that `a2 < a1`. This obviously should make `a2 + b1` as the minimum value and <b>NOT</b> `a1 + b1`. In other words, this states that `a2` and `b1` are the smallest solution to `( * )`. This information clearly contradicts our assumption that `a1` and `b1` are the smallest solution to `( * )`.

Well, in this step, we might say that our assumption that `a1 >= b1` doesn't hold true.

How about if we assume that `a1 < b1` instead?

Doing the same trick, if we square both sides, the inequality will still hold. So we got `(a1)^2 < (b1)^2`.

Also, observe that `(a1)^2 < (b1)^2 - k`. If we divide both sides by `a1` we'll get `a1 < [(b1)^2 - k] / a1`.

However, the inequality in `a1 < [(b1)^2 - k] / a1` will stay the same for all `k` that makes `[(b1)^2 - k] / a1` always greater than `a1`. In other words, this inequality will stay the same for <b>some</b> values of `k` only.

Eventually, we'd end up the same as before in which `a1 > a2`, therefore, contradicts our assumption that `a1` and `b1` are the smallest solution to `( * )`.

This information states that our assumption that `a1 < b1` doesn't hold true.

## What does it all mean then?

Well, we might say that this what would happen if we assume that `a1` and `b1` are the smallest solution to `( * )` where `k` is not a perfect square.

However, the assumption of the smallest solution can't be falsified because since we assume that there exists solutions to `( * )` where `k` is not a perfect square, we can always pick the smallest value of `a` and `b` from the pool of solutions to `( * )` where `k` is not the square of an integer. And in this case, we decided to choose `a1` and `b1` as the smallest positive integers.

If on the other hand, we assumed that `a1` and `b1` are <b>not</b> the smallest solutions to `( * )`, we might not end up with such a contradiction. However, this doesn't merely show that `( * )` where `k` is not a perfect square holds true since a contradiction comes up when we work with the smallest solutions.

Therefore, we conclude that there might be no solutions to `( * )` where `k` is not a perfect square. In other words, our first assumption that there exists solutions to `( * )` where `k` is not a perfect square is <b>false</b>.

In similar terms, we say that any value of `k` where it is not the square of an integer should not be exist since it doesn't have any solution.

Finally, we conclude that `k` must be the square of an integer.

Q.E.D
