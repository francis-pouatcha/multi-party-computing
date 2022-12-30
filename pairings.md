# Linear Maps
Let $\mathbb{X}$ and $\mathbb{Y}$ be two vector spaces over the field $\mathbb{K}$, a function $f : \mathbb{X} \rightarrow \mathbb{Y}$ is a linear map 
- if f preserves the addition operation,

$$
\forall{u,v \in \mathbb{X}}, 
\\
f(u + v) = f(u) + f(v)
$$

- and f preserves the scalar multiplication,

$$
\forall{u\in \mathbb{X}} \text{ and } \forall{c \in \mathbb{K}}, 
\\
f(c \times u)=c \times f(u)
$$

# Multilinear Maps
Let define a group $(\mathbb{G}, \circ, O, G, q)$ where:
- $\circ$ is the binary group operation,
- $O$ is the identity element,
- $G$ is the group generator,
- $q$ is the group order.

Given $\mathbb{G_i}, i \in \\{1, \dots, n\\}$, each denoting the group $(\mathbb{G}_i, \circ_i, O_i, G_i, q)$ and the target group $(\mathbb{G}_t, \circ_t, O_t, G_t, q)$, all groups in which the discrete log problem is hard, a [multilinear map](https://en.wikipedia.org/wiki/Cryptographic_multilinear_map) is a function $f: \mathbb{G_1} \times \dots \times \mathbb{G_n} \rightarrow \mathbb{G}_t$ where:

for any integers $(a_1, \dots, a_n) \in \mathbb{Z^n_q}$,

## $f$ Does Not Degenerate
The generator of the target group is the application of the function $f$ to all generators of the source groups and is not the identity element:

$$
G_t = f({G_1}, \dots, {G_n}) \text{ and } G_t \ne O_t
$$

## $f$ Preserves Scalar Multiplication
The function $f$ maps the tuple of $a_iG_i$ to the product of scalars $a_i$ applied the generator of the target group $G_t$:

$$
f(a_1G_1, \dots, a_nG_n) = (\prod_{i=1}^{n}a_i)G_t = a_tG_t \text{ where } a_t=\prod_{i=1}^{n}a_i
$$

## $f$ Preserves Linearity of all Group Operations
the function preserves the linearity of each operation $\circ_j$ of the source group $G_i$, meaning for an element $B_j=b_jG_j$

$$
f(a_1G_1, \dots, a_jG_j \circ_j B_j , \dots, a_nG_n) = (\prod_{i=1}^{n}a_i)G_t \circ_t ((\prod_{i=1, i\ne j}^{n}a_i) \times b_j)G_t
$$

## $f$ Computes Efficiently
$f$ is efficient to compute. This is the most difficult condition to fulfill. Intensive research is going on for a special form of multilinear maps  where $n=2$ is called bilinear maps or pairings.

# Bilinear Maps
Given $(\mathbb{G}_1, \circ_1, O_1, G_1, q)$ and $(\mathbb{G}_2, \circ_2, O_2, G_2, q)$ in which the discrete log problem is hard, and the target group $(\mathbb{G}_t, \circ_t, O_t, G_t, q)$, a bilinear map is a function $f: \mathbb{G_1} \times \mathbb{G_2} \rightarrow \mathbb{G}_t$ where:

for any pair integers ($a_1, a_2) \in \mathbb{Z_q}^2$,

- $f$ Does Not Degenerate

$$
G_t = f({G_1}, {G_2}) \text{ and } G_t \ne O_t
$$

- $f$ Preserves Scalar Multiplication

$$
f(a_1G_1, a_2G_2) = (a_1 \times a_2)G_t
$$

- $f$ Preserves Linearity of all Group Operations, means let $A_1=a_1G_1 \text{ and } B_1=b_1G_1 \text{ and } A_2=a_2G_2  \text{ and } B_2=b_2G_2$

$$
f(A_1, A_2 \circ_2 B_2) = f(A_1, A_2) \circ_t f(A_1, B_2)
\\ 
f(A_1 \circ_1 B_1, A_2) = f(A_1, A_2) \circ_t f(B_1, A_2)
$$

## DLP
The DLP is hard in $\mathbb{G_1}$ if knowing $A_1$, it is hard to find $a_1$ such that $A_1 = a_1G_1$.

Let $A_1=a_1G_1$ and $f(A_1, G_2) = a_1G_t$, if we can compute $a_1$ knowing $a_1G_t$ then we can compute $a_1$ knowing $A_1$. This means if we solve DLP in the target group, we can solve DLP in any of the source groups.

## Bilinear DHP (BDHP)
Recall that with only two secret scalars $(a, b)$ and the public images $aG_1, bG_1, aG_2, bG_2$, we can not define DHP on $\mathbb{G_t}$, as we $have $f(aG_1, bG2) = (a \times b)G_t = (b \times a)G_t = f(bG_1, aG_2)$.

If we add another secret scalar scalar $c$, so that we also publish $cG_1$ and $cG_2$, then we can make it challenging to compute $(a \times b \times c)G_t$. Recall that using $f$, we can compute $(a \times b)G_t, (a \times c)G_t \text{ and } (b \times c)G_t$.

Only the owners of any of the secret scallars $a, b \text{ or } c$ can compute the common secret element $(a \times b \times c)G_t$.

## Decisional DHP
Given secret scalars $a, b, c$, and public images $aG_1, bG_1, cG_1$, the DDHP wants to decide if $cG_1 = (a \times b)G_1$ meaning if $c = a \times b$.

Using the function $f$, it is easy to compute 

$$
\begin{aligned}
f(aG_1 \circ_1 bG_1, G_2) &= f(aG_1, G_2) \circ_t f(bG_1, G_2)
\\
f(aG_1 \circ_1 bG_1, G_2) &= af(G_1, G_2) \circ_t bf(G_1, G_2)
\\
f(aG_1 \circ_1 bG_1, G_2) &= (a \times b)f(G_1, G_2)
\\
f(aG_1 \circ_1 bG_1, G_2) &= f((a \times b)G_1, G_2)
\end{aligned}
$$

This is, even though we can not compute $(a \times b)G_1$, we can assume that $cG_1 = (a \times b)G_1$ if we can verify that $f(aG_1 \circ_1 bG_1, G_2) \equiv f(cG_1, G_2)$.

Therefore, the decisional DHP can be efficiently solved using multilinear maps.

## Pairings on Elliptic Curves
Bilinear maps are also known as pairings. Practically:

Let $p$ be a prime, $n$ be an integer and $(\mathbb{F}_{p^n}, +, \times)$ be :
- a field over integer $\mathbb{F}_{p^n} == \mathbb{Z}_{p^n}$ 
- with characteristic $p$, generally a prime,
- with extension $n \in \mathbb{Z}$,
- where $p^n$ is the field order.

Let $(\mathbb{E_{(\mathbb{F}_{p^n})}}, \circ, O, G, q)$ be the definition of an elliptic curve group where:
- $\circ$ is the points addition operation defined above, including the additive inverse and the identity element at $O$, 
- $G$ is the generator point,
- $q$ is the order of the group generator $G$, or the number of points on curve $\mathbb{E}$ that can be generated from $G$, 
  - meaning that $qG = O$
  - meaning that all operations on the saclar $n$ such that $nG \in \mathbb{E}$ are performed modulo $q$
- Coordinates of point $P = (x_p, y_p)$ are elements of $\mathbb{F_{p^n}}$. This means all operations on $x_p, y_p$ are done modulo $p^n$.
- Recall that $q \le p^n$

Let
- The elliptic curve group $(\mathbb{G_{1(\mathbb{F}_{p^n})}}, +, O_1, G_1, q_1)$ be the first source group,
- The elliptic curve group  $(\mathbb{G_{2(\mathbb{F}_{p^n})}}, +, O_2, G_2, q_2)$ be the second source group, 
- The integer field $\mathbb{F_{p^n}}$ be the target group $\mathbb{G_{t(\mathbb{F_{p^n})}}}$

If $a, b, c \in \mathbb{F_{p^n}}$ are secret numbers with corresponding public images $aG_1, bG_1, cG_1$, then we can leverage those public information to check if $c = a \times b$, using a pairings function $f: \mathbb{G_1} \times \mathbb{G_2} \rightarrow G_t$.

This is, even though we can not compute $(a \times b)G_1$, we can conclude that $cG_1 = (a \times b)G_1$ if we can verify that $f(aG_1 \circ_1 bG_1, G_2) \equiv f(cG_1, G_2)$. Reducing the problem to 1 point operation and 2 pairings operations.

Therefore, if we can define a computable pairing function on a curve, then we solve the DDHP on that curve. Note that the Computational DHP and the DLP remain hard.

# Pairing for Constraint Systems
## Checking Linear Constraints
When DLP is hard solve on a curve $(\mathbb{G_{(\mathbb{F}_{p^n})}}, \circ, O, G, q)$, we can use it to check linear constraints between undisclosed numbers. 

If $a, b, c \in \mathbb{F_{p^n}}$ are secret numbers with corresponding public images $A=aG_, B=bG, C=cG$, 

then given three know integer $n,m,k$, we can check the constraint 

$$
(n \times a) + (m \times b) = k \times c
$$

by just verifying

$$
nA \circ mB = kC
$$

and without having to disclose the secret scalars $a, b, c$.

## Quadratic Constraints
When DLP is hard solve on a curve $(\mathbb{G_{(\mathbb{F_{p^n}})}}, \circ, O, G, q)$, we can use it to check linear constraints between undisclosed numbers, but not quadratic constraints.

If $a, b, c \in \mathbb{F_{p^n}}$ are secret numbers with corresponding public images $A=aG_, B=bG, C=cG$, it is difficult to check

$$
(a \times b) = c
$$

Using a pairing function like $f$ defined above, open room for the validation of quadratic constraints.

## Cryptographic Applications
The ability to use elliptic curves and pairings to check linear and quadratic constraints on integers open room for the construction of cryptographic signature schemes, zero knowledge quadratic arithmetic programms and many more. 

More to this to come.s