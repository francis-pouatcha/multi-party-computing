# Computational Hardness Assumption
A [Computational Hardness Assumption](https://en.wikipedia.org/wiki/Computational_hardness_assumption) is the hypothesis that a particular computational problem cannot be solved efficiently.

## Integer Factorization Problem
The [Integer Factorization Problem](https://en.wikipedia.org/wiki/Integer_factorization). E.g: fatorizing $30$ into $2 * 3 * 5$ looks easy, but for a sufficiently large number like 18848997161, it takes a lot of time for a computer to find the prime factors.

## RSA Problem
The [RSA Problem](https://en.wikipedia.org/wiki/RSA_problem) states that:
- given a plain text message $m$,
- given a composite number $n$, whose _factors_ are not known (e.g. $n = p \times q$),
- given an exponent $e$,
- given the number $c$ sucht that $c \equiv m^e \pmod n$,
- it hard to compute the message $m$.

In matter of public key cryptography,
- the pair $(n, e)$ is known as the __public key__,
- the number $c$ is the cypher text of the plain message $m$, 
- factors $(p, q)$ are known together as the __private key__. Recall $n = p \times q$. This means if you know the factors of $n$ (e.g if $n=30=2*3*5$) it becomes easy to compute $m$.

## Residuosity problems
The [residuosity problem](https://en.wikipedia.org/wiki/Higher_residuosity_problem) states that:
- given a composite number $n$, whose _factors_ are not known (e.g. $n = p \times q$), 
- given integers $(y,d)$,  
- it is hard to find an $x$ such that $x^d \equiv y \pmod n$. 

If you know the factors of $n$, it becomes easy to compute $x$.

Remark that the RSA problem and the residuosity problem both build on top of the integer factorization problem and leverage the power of the modular arithmetic.

## Decisional Composite Residuosity Assumption
States:
- given two unknown factors $(p,q)$ of a compoiste number $n=p \times q$,
- given an integer $z$, 
- it is hard to decide whether $z$ is an $n$-residue module $n^2$.

Means it is hard to decide whether $\exists_y, z \equiv y^n \pmod {n^2}$. If you know the factors $(p, q)$ of $n$, the problem is easy to solve.

## Discrete Log Problem (DLP)
Given the group $(\mathbb{G}, ., q)$ writen multiplicatively, given two group elements $a$ and $b$, find the [discrete log](https://en.wikipedia.org/wiki/Discrete_logarithm) $k$ such that $a \equiv b^k \pmod q$. Where 
- $a=g^k$ stands for the $k$-time application of the group operation "$.$" to the generator $b$ and 
- $k=log_ga$ represents the inverse transformation of $a=g^k \pmod q$, which is known to be a difficult problem to solve.

Recall that by exception, we are using small letters $a, g$ to represent group elements in this section so reader can associate them to representations found in the literature.
