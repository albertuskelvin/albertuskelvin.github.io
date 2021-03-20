---
title: 'The Levinson-Durbin Recursion Example'
date: 2021-01-22
permalink: /posts/2021/01/levinson-durbin-example/
tags:
  - maths
  - levinson durbin
  - toeplitz
  - recursion
---

In the previous <a href="https://albertuskelvin.github.io/posts/2021/01/levinson-durbin-derivation/">post</a> I wrote about how to derive the Levinson-Durbin recursion.

In this post, we're going to look at how this recursion is applied for an example problem.

---

## Problem

Suppose we'd like to solve for the following linear equation.

```
A1 + 2A2 + 3A3 = 2
2A1 + A2 + 2A3 = 3
3A1 + 2A2 + A3 = 4

Find the value of A1, A2 and A3.
```

Note that the above equation can be expressed in the following matrix form.

```
[ 1   2   3 ]   [ A1 ]       [ 2 ]
[ 2   1   2 ]   [ A2 ]   =   [ 3 ]
[ 3   2   1 ]   [ A3 ]       [ 4 ]
```

As you can see, the left matrix is a Toeplitz matrix `T_p`. The right matrix can be considered as `r_p`. We'd like to find the solution for the middle vector `alpha_p` ([ A1, A2, A3 ]).

Another thing to note is that the problem deals with `p` (order) equals to 3.

With the above information, let's solve the problem with the Levinson-Durbin recursion.

---

## The Levinson-Durbin Recursion

As a refresher, below is the recursion formula.

```
Definitions


          [ a_1{p}  ]              [ a_p{p}     ]           [ phi[1] ]              [ phi[p]   ]
          [ a_2{p}  ]              [ a_(p-1){p} ]           [ phi[2] ]              [ phi[p-1] ]
alpha_p = [   .     ]     beta_p = [    .       ]     r_p = [    .   ]      rho_p = [    .     ]
          [   .     ]              [    .       ]           [    .   ]              [    .     ]
          [   .     ]              [    .       ]           [    .   ]              [    .     ]
          [ a_p{p}  ]              [ a_1{p}     ]           [ phi[p] ]              [ phi[1]   ]

===

Solution for p = 1

a_1{1} = phi[1] / phi[0]

===

eps_p = - beta_p k_(p+1)

k_(p+1) = phi[p+1] - (rho_p)_transpose alpha_p
          --------------------------------------
            phi[0] - (r_p)_transpose alpha_p
            
alpha_(p+1) = [ alpha_p ]           [   beta_p   ]
              [ ------- ] - k_(p+1) [ ---------- ]
              [    0    ]           [     -1     ]  
```

---

## Solution

Since `p` equals to 3, we'll start the recursion from `p` equals to 2.

---

### Set up p = 2

Calculate `alpha_(p+1)` or `alpha_3`.

```
alpha_3 = [ alpha_2 ]       [   beta_2   ]
          [ ------- ] - k_3 [ ---------- ]
          [    0    ]       [     -1     ]
```

Calculate `k_3`.

```
k_3   =   phi[3] - (rho_2)_transpose alpha_2
          --------------------------------------
            phi[0] - (r_2)_transpose alpha_2

k_3   =   4 - [ 3 2 ] alpha_2
          -------------------
          1 - [ 2 3 ] alpha_2
```

---

### Set up p = 1

Calculate `alpha_(p+1)` or `alpha_2`.

```
alpha_2 = [ alpha_1 ]       [   beta_1   ]
          [ ------- ] - k_2 [ ---------- ]
          [    0    ]       [     -1     ]
```

Calculate `k_2`.

```
k_2   =   phi[2] - (rho_1)_transpose alpha_1
          --------------------------------------
            phi[0] - (r_1)_transpose alpha_1
            
k_2   =   3 - [ 2 ] alpha_1
          -----------------
          1 - [ 2 ] alpha_1
```

---

### Run the recursion

We know that the solution for `alpha_1` is given by `a_1{1} = phi[1] / phi[0]` or `a_1{1} = 2`.

Plugging in `alpha_1` to `k_2` returns `k_2 = 1 / 3`.

Plugging in `alpha_1` and `k_2` to `alpha_2` returns the following. Notice that `beta_1` is the same as `alpha_1`.

```
alpha_2   =   [   2   ]       [   2   ]
              [ ----- ] - 1/3 [ ----- ]
              [   0   ]       [  -1   ]
          
alpha_2   =   [  4/3  ]
              [ ----- ]
              [  1/3  ]
```

Next, we can find `k_3` from the following.

```
k_3   =   4 - [ 3 2 ] alpha_2
          -------------------
          1 - [ 2 3 ] alpha_2
          
k_3   =   4 - [ 3 2 ] [ 4/3 ]
                      [ --- ]
                      [ 1/3 ]
          -----------------------
          1 - [ 2 3 ] [ 4/3 ]
                      [ --- ]
                      [ 1/3 ]
          
k_3   =   1/4
```

Knowing the value of `k_3`, we can now solve for `alpha_3`. Notice that `beta_2` is the reversed version of `alpha_2`.

```
alpha_3   =   [   4/3   ]       [   1/3   ]
              [ ------- ] - 1/4 [ ------- ]
              [   1/3   ]       [   4/3   ]
              [ ------- ]       [ ------- ]
              [    0    ]       [    -1   ]
          
alpha_3   =   [  15/12  ]
              [ ------- ]
              [    0    ]
              [ ------- ]
              [   1/4   ]
```

From `alpha_3`, we finally solve for A1, A2 and A3 which are 15/12, 0 and 1/4 respectively.

---

## Final Check

Let's plug in `alpha_3` to the equation to see whether it's the correct solution.

```
[ 1   2   3 ]   [ 15/12 ]     [ 2 ]
[ 2   1   2 ]   [   0   ]  =  [ 3 ]
[ 3   2   1 ]   [  1/4  ]     [ 4 ]
```

which is a match equation.
