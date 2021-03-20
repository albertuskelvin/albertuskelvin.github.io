---
title: 'Infinitely Many Prime Numbers by Euclid'
date: 2019-07-13
permalink: /posts/2019/07/euclid-infinite-prime-proof/
tags:
  - euclid proof
  - infinite prime numbers
---

To me, prime numbers are really interesting in terms of their position as the building blocks of other numbers. According to the **Fundamental Theorem of Arithmetic**, every positive integer N can be written as a product of P1, P2, P3, ..., and Pk where Pi are all prime numbers.

The question, however, lies on whether the number of primes is infinite. Well, the answer is **yes**. Euclid proved that there are infinitely many prime numbers and his proof is shown below.

Suppose we have a list **L** consisting of **all** the prime numbers. Let's multiply all of the prime numbers in **L** and add 1 to the result. Mathematically, we can write it like the following:

<pre>
L = p1 . p2 . p3 ... where pi are all prime numbers
N = L + 1
</pre>

After the above step, we now have two possible outcomes. The first one is that **N** is a prime number. If so, then our initial hypothesis telling that **L** has already included all the primes is wrong as we still have a new prime number **N** that is not in the list.

Meanwhile, the second possible outcome is that **N** is not a prime number. In this case, we may say that **N** has a prime factor since the **Fundamental Theorem of Arithmetic** states that all positive integers have a unique prime factorization. Let's call this prime factor **k**. The thing is that **k** may or may not be in our list **L**. If **k** is in the list, then **N** should not be divisible by **k** since the division will have the remaining 1. To make it more precise, **N** is not divisible by any of prime numbers in **L**. Therefore, there must be a prime factor outside **L** that divides **N**. This prime factor may or may not be **N** itself. Based on this fact, we know that we can always find another prime number outside our list **L**.

Q.E.D
