---
title: 'How to Solve This Extreme Algebra?'
date: 2020-06-20
permalink: /posts/2020/06/extreme-algebra/
tags:
  - mathematics
  - algebra
  - extreme algebra
---

Here we're gonna look at how to solve the following algebra problem.

<h3>Problem</h3>

From the following system of equation, find the value of <b>a + b + c + d</b>.

```
a   - b   + c   - d   = 2	( * )
a^2 - b^2 + c^2 - d^2 = 6	( ** )
a^3 - b^3 + c^3 - d^3 = 20	( *** )
a^4 - b^4 + c^4 - d^4 = 66	( **** )
```

---

<h3>Solution</h3>

The demonstrated solution might be one of various solutions out there.

First, let's define that <b>a + b + c + d = 2m</b>. We're going to apply this definition on <b>( * )</b>. We use `2m` simply just for mathematical convenience. Let's take a look for the sake of clarity.

Here's what we'll get after subtracting the following equations.

```
a + b + c + d = 2m
a - b + c - d = 2
------------------ (-)
2b + 2d       = 2m - 2

OR

b + d         = m - 1		( A )
``` 

On the other hand, let's see what we'll get after adding both equations.

```
a + b + c + d = 2m
a - b + c - d = 2
------------------ (+)
2a + 2c       = 2m + 2

OR

a + c         = m + 1		( B )
```

We're done with <b>( * )</b>.

Next, we'll do some manipulations with <b>( ** )</b>.

Know that we may rewrite <b>( ** )</b> to the following form.

```
6 = a^2 - b^2 + c^2 - d^2

6 = a^2 + c^2 - (b^2 + d^2)

6 = (a + c)^2 - 2ac - ((b + d)^2 - 2bd)
```

We chose to group `a` with `c` and group `b` with `d` just because we already get the relation between `a` and `c` (`a + c = m + 1`), as well as `b` and `d` (`b + d = m - 1`).

Since we know the value of `a + c` and `b + d` in terms of `m`, let's plug them in.

```
6 = (a + c)^2 - 2ac - ((b + d)^2 - 2bd)

6 = (m + 1)^2 - 2ac - ((m - 1)^2 - 2bd)

6 = m^2 + 2m + 1 - 2ac - (m^2 + 1 - 2m - 2bd)

6 = 4m - 2ac + 2bd

3 = 2m - ac + bd

3 - 2m = bd - ac	( C )
```

Alright, we finally got the relation between `bd` and `ac` from <b>( ** )</b>.

Let's proceed by working on <b>( *** )</b>.

```
20 = a^3 - b^3 + c^3 - d^3

20 = a^3 + c^3 - (b^3 + d^3)

20 = (a + c)^3 - 3a^2.c - 3a.c^2 - ((b + d)^3 - 3b^2.d - 3b.d^2)

20 = (a + c)^3 - 3ac(a + c) - ((b + d)^3 - 3bd(b + d))
```

We know the value for `(a + c)` and `(b + d)` in terms of `m`. Let's plug them in.

```
20 = (a + c)^3 - 3ac(a + c) - ((b + d)^3 - 3bd(b + d))

20 = (m + 1)^3 - 3ac(m + 1) - ((m - 1)^3 - 3bd(m - 1))

20 = m^3 + 3m^2 + 3m + 1 - 3ac(m + 1) - (m^3 - 3m^2 + 3m - 1 - 3bd(m - 1))

20 = 6m^2 + 2 - 3ac(m + 1) + 3bd(m - 1)

20 = 6m^2 + 2 - 3ac.m - 3ac + 3bd.m - 3bd

20 = 6m^2 + 2 + 3m(bd - ac) - 3(ac + bd)
```

Plug in the value of `(bd - ac)` to the above equation.

```
20 = 6m^2 + 2 + 3m(bd - ac) - 3(ac + bd)

20 = 6m^2 + 2 + 3m(3 - 2m) - 3(ac + bd)

20 = 6m^2 + 2 + 9m - 6m^2 - 3(ac + bd)

20 = 2 + 9m - 3(ac + bd)

18 = 9m - 3(ac + bd)

6 = 3m - (ac + bd)

3m - 6 = ac + bd	( D )
```

We're done with <b>( *** )</b>. Let's proceed with <b>( **** )</b>.

```
66 = a^4 - b^4 + c^4 - d^4

66 = a^4 + c^4 - (b^4 + d^4)

66 = (a + c)^4 - 4a^3.c - 6a^2.c^2 - 4a.c^3 - ((b + d)^4 - 4b^3.d - 6b^2.d^2 - 4b.d^3)

66 = (a + c)^4 - 2ac(2a^2 + 3ac + 2c^2) - ((b + d)^4 - 2bd(2b^2 + 3bd + 2d^2))

66 = (a + c)^4 - 2ac(2(a^2 + c^2) + 3ac) - ((b + d)^4 - 2bd(2(b^2 + d^2) + 3bd))

66 = (a + c)^4 - 2ac(2((a + c)^2 - 2ac) + 3ac) - ((b + d)^4 - 2bd(2((b + d)^2 - 2bd) + 3bd))

66 = (a + c)^4 - 2ac(2(a + c)^2 - ac) - ((b + d)^4 - 2bd(2(b + d)^2 - bd))

66 = (a + c)^4 - 4ac(a + c)^2 + 2a^2.c^2 - ((b + d)^4 - 4bd(b + d)^2 + 2b^2.d^2)

66 = (a + c)^4 - 4ac(a + c)^2 + 2a^2.c^2 - (b + d)^4 + 4bd(b + d)^2 - 2b^2.d^2

66 = (a + c)^4 - 4ac(a + c)^2 + 2(a^2.c^2 - b^2.d^2) - (b + d)^4 + 4bd(b + d)^2

66 = (a + c)^4 - 4ac(a + c)^2 + 2((ac + bd)(ac - bd)) - (b + d)^4 + 4bd(b + d)^2
```

Let's plug in the value for each expression in terms of `m`.

```
66 = (m + 1)^4 - 4ac(m + 1)^2 + 2((3m - 6)(2m - 3)) - (m - 1)^4 + 4bd(m - 1)^2

66 = m^4 + 4m^3 + 6m^2 + 4m + 1 - 4ac(m^2 + 2m + 1) + 2(6m^2 - 21m + 18) - (m^4 - 4m^3 + 6m^2 - 4m + 1) + 4bd(m^2 - 2m + 1)

66 = 8m^3 + 8m - 4ac(m^2) - 8ac(m) - 4ac + 12m^2 - 42m + 36 + 4bd(m^2) - 8bd(m) + 4bd

66 = 8m^3 - 34m - 4m^2(ac - bd - 3) - 8m(ac + bd) - 4(ac - bd) + 36

66 = 8m^3 - 34m - 4m^2(2m - 6) - 8m(3m - 6) - 4(2m - 3) + 36

66 = 8m^3 - 34m - 8m^3 + 24m^2 - 24m^2 + 48m - 8m + 12 + 36

66 = 6m + 48

18 = 6m

m = 3
```

Since we already know that `m = 3`, then <b>a + b + c + d = 2m = 2(3) = 6</b>.
