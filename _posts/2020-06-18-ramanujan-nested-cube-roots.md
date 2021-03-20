---
title: "Ramanujan's Nested Cube Roots Proof"
date: 2020-06-18
permalink: /posts/2020/06/ramanujan-nested-cube-roots/
tags:
  - maths
  - ramanujan
  - nested cubic roots
---

The theorem of nested cube roots (Ramanujan) states the following.

---

Let <b>p, q, r</b> be roots of the following equation.
<br/><br/>
x^3 - a.x^2 + b.x - 1 = 0
<br/><br/>
Then
<br/><br/>
crt(<b>p</b>) + crt(<b>q</b>) + crt(<b>r</b>) = crt(a + b + 3t) => <b>( P1 )</b>
<br/><br/>
Where
<br/><br/>
t^3 - 3(a + b + 3)t - (ab + 6(a + b) + 9) = 0 => <b>( P2 )</b>

---

<b>Notes:</b>
- "crt" stands for cube root

---

In this post, we’re going to prove the above theorem.

First, know that a cubic equation can be written in terms of its root products, such as:

```
x^3 - a.x^2 + b.x - 1 = (x - p)(x - q)(x - r) = 0			( * )
```

After expanding the root products, we know that `p . q . r = 1`.

Furthermore, we also know that `crt(p) . crt(q) . crt(r) = 1`.

Now, let’s define two new coefficients `m` and `n`. Put them to a new cubic equation (similar to the previous one) which then will yield the following.

```
z^3 - m.z^2 + n.z - 1 = (z - crt(p))(z - crt(q))(z - crt(r)) = 0		( ** )
```

Rearranging the above equation to `z^3 - 1 = m.z^2 - n.z` and take the cubic product for both sides yield the following.

```
(z^3 - 1)^3 = (m.z^2 - n.z)^3

(z^3 - 1)^3 = m^3.z^6 - 3.m^2.z^5.n + 3.m.z^4.n^2 - n^3.z^3
(z^3 - 1)^3 = m^3.z^6 - n^3.z^3 - 3.m.n.z^3 (m.z^2 - z.n)

Know that (m.z^2 - z.n) = z^3 - 1, therefore

(z^3 - 1)^3 = m^3.z^6 - n^3.z^3 - 3.m.n.z^3 (z^3 - 1)

Move the RHS to LHS and we’ll get

(z^3 - 1)^3 - m^3.z^6 + n^3.z^3 + 3.m.n.z^3 (z^3 - 1) = 0
```

Up til now, we know that the values of `z` for the above equation are `crt(p), crt(q)` and `crt(r)`.

Now, let’s create another variable `u` which stores the value of `z^3`, meaning that `u = z^3`. Plug in this `u` to the above equation and we’ll get the following.

```
(u - 1)^3 - m^3.u^2 + n^3.u + 3.m.n.u (u - 1) = 0
```

Expand the above equation a bit and we’ll get the following.

```
(u - 1)^3 - m^3.u^2 + n^3.u + 3.m.n.u^2 - 3.m.n.u = 0

u^3 - 3u^2 + 3u - 1 - m^3.u^2 + n^3.u + 3.m.n.u^2 - 3.m.n.u = 0

u^3 - (m^3 - 3.m.n + 3).u^2 + (n^3 - 3.m.n + 3).u - 1 = 0		( *** )
```

The form of ( *** ) seems similar to ( * ). In addition, since we know that the roots of `z` are `crt(p), crt(q), crt(r)`, then the roots of `u` are `p, q, r`. In other words, the roots of `u` are the same with the roots of `x`. Therefore, we could equate the coefficient like the following.

```
a = m^3 - 3.m.n + 3
b = n^3 - 3.m.n + 3

OR

m^3 = a + 3.m.n - 3
n^3 = b + 3.m.n - 3
```

Now, what we’re going to do is to introduce a new variable `t` which equals to `m.n - 3`. This means that `m.n = t + 3`. Plugging in this `m.n` in terms of `t` to the above equation of `m^3` and `n^3` yields the following.

```
m^3 = a + 3.(t + 3) - 3 = a + 3.t + 6
n^3 = b + 3.(t + 3) - 3 = b + 3.t + 6
```

Recall that we defined a new cubic equation for `z` ( *** ) and if we expand the root products, we’ll get the following.

```
z^3 - m.z^2 + n.z - 1 = (z - crt(p))(z - crt(q))(z - crt(r)) = 0

z^3 - m.z^2 + n.z - 1 = z^2 - crt(q).z - crt(p).z + crt(pq)(z-crt(r)) = z^3 - (crt(p) + crt(q) + crt(r)).z^2 + (crt(pq) + crt(pr) + crt(qr)).z - crt(pqr)

Since crt(pqr) = 1,

z^3 - m.z^2 + n.z - 1 = z^2 - crt(q).z - crt(p).z + crt(pq)(z-crt(r)) = z^3 - (crt(p) + crt(q) + crt(r)).z^2 + (crt(pq) + crt(pr) + crt(qr)).z - 1
``` 

Now, let’s equate the coefficient.

```
m = crt(p) + crt(q) + crt(r)
n = crt(pq) + crt(pr) + crt(qr)
```

Since we already know the value of `m` and `n`, let’s plug them in.

```
crt(a + 3.t + 6) = crt(p) + crt(q) + crt(r)			( **** )
crt(b + 3.t + 6) = crt(pq) + crt(pr) + crt(qr)
```

Well, our <b>( **** )</b> already proves <b>( P1 )</b>.

---

Let’s move on to proving <b>( P2 )</b>.

But beforehand, let’s recap first. Up til now we’ve got the following variables that support our proof.

```
m.n = t + 3

m^3 = a + 3.(t + 3) - 3 = a + 3.t + 6
n^3 = b + 3.(t + 3) - 3 = b + 3.t + 6
```

Let’s take the cubic product for both sides of `m.n = t + 3`.

```
m^3.n^3 = (t + 3)^3
```

Use the value for m^3 and n^3.

```
(a + 3.t + 6)( b + 3.t + 6) = (t + 3)^3
```

Expand both sides and perform the calculation.

```
a.b + 3.a.t + 6.a + 3.b.t + 9.t^2 + 18.t + 6b + 18.t + 36 = t^3 + 9.t^2 + 27.t + 27

a.b + 3.a.t + 3.b.t + 6.a + 6.b + 9.t^2 + 36.t + 36 =  t^3 + 9.t^2 + 27.t + 27

a.b + 3.a.t + 3.b.t + 6.a + 6.b + 36.t + 36 =  t^3 + 27.t + 27
a.b + 3.a.t + 3.b.t + 6.a + 6.b + 9.t + 9 =  t^3
```

Let’s move out all terms to the RHS and left the LHS to zero.

```
0 =  t^3 - a.b - 3.a.t - 3.b.t - 6.a - 6.b - 9.t - 9
0 = t^3 - (3.a + 3.b + 9).t - (a.b + 6.a + 6.b + 9)
0 = t^3 - 3.(a + b + 3).t - (a.b + 6.(a + b) + 9)

OR

t^3 - 3.(a + b + 3).t - (a.b + 6.(a + b) + 9) = 0		( ***** )
```

Therefore, our <b>( ***** )</b> proves <b>( P2 )</b>.

And we’ve completed our proof.

Q.E.D
