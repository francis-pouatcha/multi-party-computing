# Computational Hardness Assumption
A [computational hardness assumption](https://en.wikipedia.org/wiki/Computational_hardness_assumption) is the hypothesis that a particular computational problem cannot be solved efficiently.

## Integer Factorization Problem
The [integer factorization problem](https://en.wikipedia.org/wiki/Integer_factorization). E.g: fatorizing $30$ into $2 \times 3 \times 5$ looks easy, but for a sufficiently large number like 18848997161, it takes a lot of time for a computer to find the prime factors.

## RSA Problem
The [RSA problem](https://en.wikipedia.org/wiki/RSA_problem) states that:
- given a plain text message $m$,
- given a composite number $n$, whose _factors_ are not known (e.g. $n = p \times q$),
- given an exponent $e$,
- given the number $c$ such that $c \equiv m^e \pmod n$,
- it is hard to compute the message $m$.

In matter of public key cryptography,
- the pair $(n, e)$ is known as the __public key__,
- the number $c$ is the ciphertext of the plain message $m$, 
- factors $(p, q)$ are known together as the __private key__. Recall $n = p \times q$. This means if you know the factors of $n$, e.g: if $n=30=2 \times 3 \times 5$ it becomes easy to compute $m$.

## Residuosity problems
The [residuosity problem](https://en.wikipedia.org/wiki/Higher_residuosity_problem) states that:
- given a composite number $n$, whose _factors_ are not known (e.g. $n = p \times q$), 
- given integers $(y,d)$,  
- it is hard to find an $x$ such that $x^d \equiv y \pmod n$. 

If you know the factors of $n$, it becomes easy to compute $x$.

Remark that the RSA problem and the residuosity problem both build on top of the integer factorization problem and leverage the power of modular arithmetic.

## Discrete Log Problem (DLP)
Given: 
- the group $(\mathbb{G}, \times, q)$, 
- two group elements $a$ and $b$, 

find the [discrete log](https://en.wikipedia.org/wiki/Discrete_logarithm) $k$ such that $a \equiv b^k \pmod q$.

Recall in this group, $k=log_{b}{(a)}$ is assumed to be hard to compute.

