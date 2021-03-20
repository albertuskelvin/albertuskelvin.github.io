---
title: 'The Levinson-Durbin Recursion Derivation'
date: 2021-01-20
permalink: /posts/2021/01/levinson-durbin-derivation/
tags:
  - maths
  - levinson durbin
  - toeplitz
  - recursion
---

I was exploring about the Levinson-Durbin recursion and curious about the formula's derivation.

Came across this <a href="https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-341-discrete-time-signal-processing-fall-2005/lecture-notes/lec13.pdf">paper</a> demonstrating how to derive the formula.

The Levinson-Durbin recursion is leveraged to find solution for the vector `alpha_p` (`a_1, a_2, ..., a_p`) from the following matrix equation.

```
[ phi[0]    phi[1]    ... phi[p-1] ]  [ a_1  ]   [ phi[1] ]
[ phi[1]    phi[0]    ... phi[p-2] ]  [ a_2  ]   [ phi[2] ]
[ .         .         .   .        ]  [ .    ] = [ .      ]
[ .         .         .   .        ]  [ .    ]   [ .      ]
[ .         .         .   .        ]  [ .    ]   [ .      ]
[ phi[p-1]  phi[p-2]  ... phi[0]   ]  [ a_p  ]   [ phi[p] ]
```

The above equation can also be expressed as `T_p alpha_p = r_p`. Note that `T_p` is a Toeplitz matrix (diagonal-constant matrix).

The vector `alpha_p` could be solved via `alpha_p = T_p_inv r_p`. However, we might want to look for a more elegant way without involving the inverse matrix computation.

The Levinson-Durbin recursion which is used to find the vector `alpha_p` is shown below.

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

Recursion

k_(p+1) = phi[p+1] - (rho_p)_transpose alpha_p
          ------------------------------------
            phi[0] - (r_p)_transpose alpha_p
            
eps_p = - beta_p k_(p+1)

                                          [ a_1{p} ]          [   a_p{p}   ]
                                          [ a_2{p} ]          [ a_(p-1){p} ]
                                          [   .    ]          [     .      ]
alpha_(p+1) = [ alpha_p ]   [  eps_p  ] = [   .    ] - k_(p+1)[     .      ]
              [ ------- ] + [ ------- ]   [   .    ]          [     .      ]
              [    0    ]   [ k_(p+1) ]   [ a_p{p} ]          [   a_1{p}   ]
                                          [   0    ]          [    -1      ]
```

Note that `a_1{p}` means the value of `a_1` for the `p`th-order model

We're going to derive the above formula.

Let's start with what'll happen when `p = 1`.

The matrix equation will be in the form of `[ phi[0] ] [ a1 ] = [ phi[1] ]`. Consequently, `a1 = phi[1] / phi[0]`.

Now we're going to look at how to develop the recursion formula for `p > 1`.

The following is the matrix form for order `p + 1`.

```
[ phi[0]    phi[1]    ... phi[p-1] phi[p]   ]  [ a_1{p+1}     ]   [ phi[1]   ]
[ phi[1]    phi[0]    ... phi[p-2] phi[p-1] ]  [ a_2{p+1}     ]   [ phi[2]   ]
[ .         .         .   .        .        ]  [ .            ] = [ .        ]
[ .         .         .   .        .        ]  [ .            ]   [ .        ]
[ .         .         .   .        .        ]  [ .            ]   [ .        ]
[ phi[p-1]  phi[p-2]  ... phi[0]   phi[1]   ]  [ a_p{p+1}     ]   [ phi[p]   ]
[ phi[p]    phi[p-1]  ... phi[1]   phi[0]   ]  [ a_(p+1){p+1} ]   [ phi[p+1] ]
```

Notice that the above matrix `T_(p+1)` can be expressed alternatively in terms of order `T_p`, such as the following.

```
[                                |  phi[p]   ]
[                                |  phi[p-1] ]
[             T_p                |  .        ]
[                                |  .        ]
[                                |  .        ]
[                                |  phi[1]   ]
[ -------------------------------|---------- ]
[ phi[p]    phi[p-1]  ... phi[1] |  phi[0]   ]
```

The same also applies for matrix `r_(p+1)`, such as below.

```
[    r_p     ]
[ ---------- ]
[  phi[p+1]  ]
```

Additionally, we also introduce a new matrix called `rho_p` which is basically is the reversed version `r_p`.

```
rho_p = [   phi[p]    ]
        [   phi[p-1]  ]
        [      .      ]
        [      .      ]
        [      .      ]
        [   phi[1]    ]
```

With all the above representations, our `T_(p+1)` becomes such as the following.

```
T_(p+1) = [       T_p         | rho_p  ]
          [ ------------------|------- ]
          [ (rho_p)_transpose | phi[0] ]
```

Since vector `alpha_p` might change with different orders (e.g., `p`, `p+1`, `p+2`), we're going to represent vector `alpha_(p+1)` with the following form.

```
alpha_(p+1) = [ alpha_p ]   [  eps_p  ]
              [ ------- ] + [ ------- ]
              [    0    ]   [ k_(p+1) ]
```

In the above form, vector `[a_1{p+1}, a_2{p+1}, ..., a_p{p+1}]` is calculated as the adjustment of `alpha_p` (`[a_1{p}, a_2{p}, ..., a_p{p}]`) with vector `eps_p` (which can be thought of as a correction vector). Similarly, `a_(p+1){p+1}` is initialized with zero which will then be adjusted with `k_(p+1)`.

Recall that the main matrix equation is given by `T_p alpha_p = r_p`. For order `p + 1`, the equation becomes `T_(p+1) alpha_(p+1) = r_(p+1)`. Applying the above representations to the equation yields the following.

```
T_(p+1) alpha_(p+1) = r_(p+1)

[       T_p         | rho_p  ]  { [ alpha_p ]   [  eps_p  ] }       [    r_p     ]
[ ------------------|------- ]  { [ ------- ] + [ ------- ] }   =   [ ---------- ]
[ (rho_p)_transpose | phi[0] ]  { [    0    ]   [ k_(p+1) ] }       [  phi[p+1]  ]
```

Let's execute the above operation considering that the left-most, middle, and right-most matrix are `2 x 2`, `2 x 1` and `2 x 1` matrix respectively.

```
EQUATION (1)

[T_p (alpha_p + eps_p)] + [rho_p k_(p+1)] = r_p
[T_p alpha_p] + [T_p eps_p] + [rho_p k_(p+1)] = r_p

EQUATION (2)

[(rho_p)_transpose (alpha_p + eps_p)] + [phi[0] k_(p+1)] = phi[p+1]
[(rho_p)_transpose alpha_p] + [(rho_p)_transpose eps_p] + [phi[0] k_(p+1)] = phi[p+1]
```

Plugging in `T_p alpha_p = r_p` equation to equation `(1)` yields `T_p eps_p k_(p+1)_inv = -rho_p`.

Since `T_p` is a Toeplitz matrix, then reversing the `alpha_p` and `r_p` yields the same result as the original form. Take a look at the below snippet.

```
[ phi[0]    phi[1]    ... phi[p-1] ]  [ a_p{p}      ]       [ phi[p]   ]
[ phi[1]    phi[0]    ... phi[p-2] ]  [ a_(p-1){p}  ]       [ phi[p-1] ]
[ .         .         .   .        ]  [   .         ]   =   [ .        ]
[ .         .         .   .        ]  [   .         ]       [ .        ]
[ .         .         .   .        ]  [   .         ]       [ .        ]
[ phi[p-1]  phi[p-2]  ... phi[0]   ]  [ a_1{p}      ]       [ phi[1]   ]
```

From the above snippet, let's represent the middle matrix with `beta_p`. Notice that the right-most matrix is our `rho_p`.

Substituting equation `(1)` with the above reverse version yields the following.

```
[T_p alpha_p] + [T_p eps_p] + [rho_p k_(p+1)] = r_p

[T_p alpha_p] + [T_p eps_p] + [T_p beta_p k_(p+1)] = T_p alpha_p

[T_p eps_p] + [T_p beta_p k_(p+1)] = 0

EQUATION (3)

eps_p = - beta_p k_(p+1)
```

Substituting the expression of `eps_p` to equation `(2)` yields the following.

```
[(rho_p)_transpose alpha_p] - [(rho_p)_transpose beta_p k_(p+1)] + [phi[0] k_(p+1)] = phi[p+1]
```

Since `k_(p+1)` doesn't consists of `beta_p`, let's see whether we could express `beta_p` in form of other variables, such as `r_p`, and `alpha_p`.

Know that `beta_p` is a reversed form of `alpha_p` and `rho_p` is a reversed form of `r_p`. Therefore, the result of `(rho_p)_transpose beta_p` equals to `(r_p)_transpose alpha_p`.

Using this equality to equation `(2)` yields the following.

```
[(rho_p)_transpose alpha_p] - [(r_p)_transpose alpha_p k_(p+1)] + [phi[0] k_(p+1)] = phi[p+1]
```

From the above we get `k_(p+1)` shown below.

```
EQUATION (4)

k_(p+1) = phi[p+1] - (rho_p)_transpose alpha_p
          --------------------------------------
            phi[0] - (r_p)_transpose alpha_p
```

Last but not least, replacing `eps_p` with `- beta_p k_(p+1)` for `alpha_(p+1)` yields the following.

```
alpha_(p+1) = [ alpha_p ]   [  - beta_p k_(p+1)  ]
              [ ------- ] + [ ------------------ ]
              [    0    ]   [       k_(p+1)      ]
              
EQUATION (5)

alpha_(p+1) = [ alpha_p ]           [   beta_p   ]
              [ ------- ] - k_(p+1) [ ---------- ]
              [    0    ]           [     -1     ]        
```

---

To conclude, here are the derived equations.

```
EQUATION (3)
eps_p = - beta_p k_(p+1)

EQUATION (4)
k_(p+1) = phi[p+1] - (rho_p)_transpose alpha_p
          --------------------------------------
            phi[0] - (r_p)_transpose alpha_p
            
EQUATION (5)
alpha_(p+1) = [ alpha_p ]           [   beta_p   ]
              [ ------- ] - k_(p+1) [ ---------- ]
              [    0    ]           [     -1     ]  
```

With the supporting information.

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
```

Done.
