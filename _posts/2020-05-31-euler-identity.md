---
title: 'The Most Beautiful Equation'
date: 2020-05-31
permalink: /posts/2020/05/the-most-beautiful-equation-euler-identity/
tags:
  - mathematics
  - euler identity
  - maclaurin series
---

Euler's formula is stated as the following.

<h4>
e^(ix) = cos(x) + i.sin(x)
<br/><br/>
where i = sqrt(-1)
</h4>
<br/>

Meanwhile, Euler's identity is stated as the following.

<h4>
e^(i.pi) + 1 = 0
<br/><br/>
where i = sqrt(-1)
</h4>
<br/>

In this post, we're going to prove the above beautiful equations.

The main approach here is by leveraging the Maclaurin series for `e^x` which is formed as `1 + (x)/(1!) + (x^2)/(2!) + (x^3)/(3!) + (x^4)/(4!) + . . .`.

Simply by replacing `x` with `i.x` we'll get a new Maclaurin series as shown below.

<h4>
e^(i.x) = 1 + [(i.x) / (1!)] + [(i^2.x^2) / (2!)] + [(i^3.x^3) / (3!)] + [(i^4.x^4) / (4!)] + . . .
</h4>
<br/>

Simplifying the `i` part of the above equation yield the following.

<h4>
e^(i.x) = 1 + [(i.x) / (1!)] + [(-1.x^2) / (2!)] + [(-i.x^3) / (3!)] + [(1.x^4) / (4!)] + . . .
<br/><br/>
e^(i.x) = 1 + [(i.x) / (1!)] - [(x^2) / (2!)] - [(i.x^3) / (3!)] + [(x^4) / (4!)] + . . .
</h4>
<br/>

Notice that the above series seems to have a similar pattern to the Maclaurin series of certain functions. Those functions are the sine and cosine. As a refresher, let's take a look at the Maclaurin series for both of these functions as well as the above `e^(ix)`.

## Maclaurin series for Sine

<h4>
sin(x) = x - [(x^3) / (3!)] + [(x^5) / (5!)] - [(x^7) / (7!)] + . . .
</h4>

## Maclaurin series for Cosine

<h4>
cos(x) = 1 - [(x^2) / (2!)] + [(x^4) / (4!)] - [(x^6) / (6!)] + . . .
</h4>

## Maclaurin series for e^(i.x)

<h4>
e^(i.x) = 1 + [(i.x) / (1!)] - [(x^2) / (2!)] - [(i.x^3) / (3!)] + [(x^4) / (4!)] + [(i.x^5) / (5!)] - . . .
<br/><br/>
e^(i.x) = {1 - [(x^2) / (2!)] + [(x^4) / (4!)] - . . .} + i {[(x) / (1!)] - [(x^3) / (3!)] + [(x^5) / (5!)] - . . .}
</h4>

---

Take a look at the above Maclaurin series for `e^(i.x)`. The group of terms without `i` is the Maclaurin series for `cos(x)`, while the ones with `i` is the Maclaurin series for `sin(x)`.

Therefore, we can rewrite the Maclaurin series of `e^(i.x)` to the following.

<h4>
e^(i.x) = cos(x) + i.sin(x)
</h4>
<br/>

Well, we've just proved the Euler's formula!

Now, how about the Euler's identity?

Recall that the Euler's identity is stated as the following.

<h4>
e^(i.pi) + 1 = 0
<br/><br/>
where i = sqrt(-1)
</h4>
<br/>

Well, what's the result of `e^(i.pi)` then? Seems that working with `x = pi` in the Euler's formula might be the easiest way to prove this identity.

<h4>
e^(i.pi) = cos(pi) + i.sin(pi)
<br/><br/>
e^(i.pi) = -1
<br/><br/>
e^(i.pi) + 1 = 0
</h4>
<br/>

Q.E.D
