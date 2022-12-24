# ECDSA Signature
ECDSA specifies the use of [elliptic curves](./ecgroups.md) for the construction of DSA signature. 
## Elliptic Curves

Let $\mathbb{Z_q}$ represents the field of integers modulo $q$ writen $(\mathbb{Z}, +, \times, q)$.

Let $\mathbb{E}$ be an [elleiptic curve group](./ecgroups.md) $(\mathbb{E}_{(\mathbb{Z_q})}, \circ, p, G)$, defined over $\mathbb{Z_q}$ where
- $\circ$ is the binary group operation, a.k.a addition of points,
- $G$ is the generator of the group,
- p is the order of the generator $G$,

Review following as essetial for the understanding of ECDSA math below:
- Ellements of $\mathbb{Z}$ are integer numbers and always writen using small letters, e.g.: $a, r$
- Elements of $\mathbb{E}$ are points on the curve and always writen using capital letters. e.g.: $G, A, R$, where $x_R, y_R$ are integer coordinate of the point $R$.

### Operations in $(\mathbb{Z}, +, \times, q)$
- $a + b$ denotes the addition of two integers $a, b \in \mathbb{Z_q}$. They are allways performed modulo $q$ even if omited.
- $a \times b$ denotes the multiplication of two integers $a, b \in \mathbb{Z_q}$. They are allways performed modulo $q$ even if omited.

### Operations in $(\mathbb{E}_{(\mathbb{Z_q})}, \circ, p, G)$
- $A \circ B$ denotes the addition of two points $A=(x_A, y_A) \text{ and } B=(x_B, y_B)$, both $A, B \in \mathbb{E}$. Operations on points coordinates $x_A, y_A, x_B, y_B$ are performed in $\mathbb{Z_q}$, means modulo $q$.
- $nA$ is the $n$-times addition of the point $A$ to itself. Means $nA=G \circ_0 A \circ_1 A \dots \circ_{n-1} A$. This is called the scalar multiplication of the point $A \in \mathbb{E}$ by the integer $n \in \mathbb{Z_q}$.
- Recall that on $\mathbb{E}$, following applies:
  - $aG \circ bG = (a + b)G$
  - $a (bG) = (a \times b)G$

## ECDSA Signature
Let $a \in_R \mathbb{F_p}$ be the secret key, a random number unifromly sampled from the field $\mathbb{Z_q}$, the ECDSA signature is a pair $(s,r) \in \mathbb{Z^2_q}$ where:
- $r$ represents the coordinate $x_R$ of the public image $R = ({1 \over k}G)$ of a random parameter $k \in_R \mathbb{Z_q}$. Random $k$ is used to mask the secret key,
- $s$ is the signature, computed with the formula $s = k(m+ (r \times a))$, where $m=\Eta (M)$ and $\Eta : M \rightarrow \mathbb{Z_q}$ is a hash function used to embed the message $M$ into an element of $\mathbb{Z_q}$, or in short, into a number.

Key implementation observations:
- A new random value $k$ must be chosen if $r=0 \texttt{ or }s = 0$.
- The value of $k$ is never disclosed to the public, as knowledge of $k$ will lead to the computation of the secret key 

$$a = ((s \times k^{-1}) - m) \times r^{-1}$$

## Verification
The verification formula requires to reproduce the public image $R'$ of the random point $k$ and compare the coordinatres $r'=x_{R'}=x_R=r$.

In order to compute $R'$ withouth knowing k, we proceed like:

$$
\begin{aligned}
&s = k(m + (r \times a))
\\
&{1 \over k} = {(m + (r \times a)) \over s}
\\
&{1 \over k} = {m \over s} + {(r \times a) \over s}
\\
&\text{operation above are in } \mathbb{Z_q}
\\
&\text{operation bellow are in } \mathbb{E}
\\
&R' = {1 \over k}G = ({m \over s} + {(r \times a) \over s})G
\\
&R' = {1 \over k}G = {m \over s}G \circ {(r \times a) \over s}G
\\
&R' = {1 \over k}G = {m \over s}G \circ ({r \over s}  \times a)G
\\
&R' = {1 \over k}G = {m \over s}G \circ ({r \over s})aG 
\\
&\text{as we know } m, r, s \texttt{ and } A=aG
\\
&R' = {m \over s}G \circ {r \over s}A
\end{aligned}
$$

As you can observe, the computation of the signature $s$ happens in $\mathbb{Zq}$ but the verification is done performing point operations on $\mathbb{E}$. This means elliptic curves can allow the verification of the linear constraint $a = ((s \times k^{-1}) - m) \times r^{-1}$ by checking the same constraint on public images in $\mathbb{E}$. The only information no available is the secret number $a$, but we have its public image $A=aG$.

The signature is valid if $r = x_{R'}$ and following validation test are positive.

## Further Validations
A signature validation will further require to:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{Z_q}$ and $1 \lt r,s \lt q$
- validate tha the public key $A = (x_A, y_A)$ is a valid curve point meaning:
  - $A$ is not the identity element $A \ne O$
  - $x_A, y_A \in \mathbb{Z_q}$
  - $y_A = E(x_A)$
  - $pA = O$, that A is generated from G. Recall $p$ is the order of $G$,
- validate $|m| \le |p|$. The value of $m$ can be higher than $p$, but the lenght of $m$ in bits can not be longer than the size of $p$ in bits. Thefore $m$ is the $l_p$ most significant bits of H(M). Where $l_p = |p|$.

## Recovering the Public Key A
In the case of public ledger like bitcoin, the public key $A$ is not known, but an image or precisely address $\alpha = \Eta'(A)$. For bitcoin $\Eta'$ is SHA256_RIPMD160.

Knowing the signature imformation $(r, s)$ and the message $M \rightarrow \Eta(M)=m$, we can compute the public key proceeding like:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{Z_q}$ and $1 \lt r,s \lt q$
- compute $u_1 = {m \over r}$ and $u_2 = {s \over r}$
- $\forall_{i\in \mathbb{Z}, r+i \in \mathbb{Z_q}}$:
  - compute the curve point $R_i = (r+i)G = (x_{R_i}, y_{R_i})$
  - compute the curve point $A_i = (x_{A_i}, y_{A_i})= u_1G \circ u_2R_i$
  - the inverse point $-A_i = (x_{A_i}, -y_{A_i})$ is also a valid point.
  - if address $\alpha = H'(A_i)$, then $A = A_i$, return,
  - if address $\alpha = H'(-A_i)$, then $A = -A_i$ return.
- If none of the conditions satisfies the given address $\alpha$, then the signature is forged.
