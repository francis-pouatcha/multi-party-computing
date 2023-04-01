# ECDSA Signature
ECDSA specifies the use of [elliptic curves](./ecgroups.md) for the construction of DSA signatures.

The definition of bit and byte strings as used in this document is found in [strings expressions](./conv-ser-enc.md).

## Elliptic Curves
Let the expression $\mathbb{Z_q}$ represent the field of integers modulo $q$ written $(\mathbb{Z}, +, \times, q)$.

Let the expression $\mathbb{E_{(\mathbb{Z_q})}}$ represent an [elliptic-curve group](./ecgroups.md) $(\mathbb{E_{(\mathbb{Z_q})}}, \circ, O, G, p)$, defined over $\mathbb{Z_q}$ where
- $\circ$ is the binary group operation, a.k.a addition of points,
- O is the identity element, a.k.a neutral element
- $G$ is the generator of the group,
- p is the order of the generator $G$,

Reviewing [finite cyclic elliptic curve groups](./ecgroups.md#finite-cyclic-groups-over-elliptic-curves) is essential for the understanding of ECDSA maths below.


## ECDSA Signature
Let $a \in_{rand} \mathbb{Z_q}$ be the secret key, a random number uniformly sampled from the field $\mathbb{Z_q}$, the ECDSA signature is the string $r_{<(be:o):32>}||s_{<(be:o):32>}$ where:
- the pair $(s,r) \in \mathbb{Z^2_q}$ and
- $r$ represents the coordinate $x_R$ of the public image $R = ({1 \over k}G)$ of a random parameter $k \in_{rand} \mathbb{Z_q}$. Random $k$ is used to mask the secret key,
- $s$ is the signature, computed with the formula $s = k(m+ (r \times a))$, where $m=H (M)$ and $H : M \rightarrow \mathbb{Z_q}$ is a hash function used to embed the message $M$ into an element $m \in \mathbb{Z_q}$, or in short, into a number.

Key implementation observations:
- A new random value $k$ must be chosen if $r=0 \texttt{ or }s = 0$.
- The value of $k$ is never disclosed to the public, as knowledge of $k$ will lead to the computation of the secret key

$$a = ((s \times k^{-1}) - m) \times r^{-1}$$

## Verification
The verification formula requires to reproduce the public image $R'$ of the random point $k$ and compare the coordinates $r'=x_{R'}=x_R=r$.

In order to compute $R'$ without knowing k, we proceed like:

$$
\begin{aligned}
&s = k(m + (r \times a))
\\
&{1 \over k} = {(m + (r \times a)) \over s}
\\
&{1 \over k} = {m \over s} + {(r \times a) \over s}
\\
&\text{operations above are in } \mathbb{Z_q}
\\
&\text{operations bellow are in } \mathbb{E_{(\mathbb{Z_q})}}
\\
&R' = {1 \over k}G = ({m \over s} + {(r \times a) \over s})G
\\
&R' = {1 \over k}G = {m \over s}G \circ {(r \times a) \over s}G
\\
&R' = {1 \over k}G = {m \over s}G \circ ({r \over s} \times a)G
\\
&R' = {1 \over k}G = {m \over s}G \circ ({r \over s})aG
\\
&\text{as we know } m, r, s \texttt{ and } A=aG
\\
&R' = {m \over s}G \circ {r \over s}A
\end{aligned}
$$

As you can see observe, the computation of the signature $s$ happens in $\mathbb{Zq}$ but the verification is done performing point operations in $\mathbb{E_{(\mathbb{Z_q})}}$. This means elliptic curves can allow the verification of the linear constraint $a = ((s \times k^{-1}) - m) \times r^{-1}$ by checking the same constraint on public images in $\mathbb{E_{(\mathbb{Z_q})}}$. The only information not available is the secret number $a$, but we have its public image $A=aG$.

The signature is valid if $r = x_{R'}$ and the following validation test are positive.

## Further Validations
A signature validation will further require to:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{Z_q}$ and $1 \lt r,s \lt q$
- validate that the public key $A = (x_A, y_A)$ is a valid curve point meaning:
- $A$ is not the identity element $A \ne O$
- $x_A, y_A \in \mathbb{Z_q}$
- $y_A = E(x_A)$
- $pA = O$, therefore verifying that $A$ is generated from $G$. Recall $p$ is the order of $G$,
- validate $|m| \le |p|$. The value of $m$ can be higher than $p$, but the length of $m$ in bits can not be longer than the size of $p$ in bits. Therefore $m$ is the $l_p$ most significant bits of H(M). Where $l_p = |p|$, is the length of $p$ in bits.

## Recovering the Public Key A
In the case of public ledgers like bitcoin, the public key $A$ must not be known, but an image or precisely address $\alpha = H'(A)$ is registered with the ledger. For bitcoin $H'$ is the combined hash algorithm SHA256_RIPMD160.

Knowing the signature information $(r, s)$ and the message $M \rightarrow H(M)=m$, we can compute the public key proceeding like:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{Z_q}$ and $1 \lt r,s \lt q$
- compute $u_1 = {m \over r}$ and $u_2 = {s \over r}$
- $\forall_{i\in \mathbb{Z}, r+i \in \mathbb{Z_q}}$:
- compute the curve point $R_i = (r+i)G = (x_{R_i}, y_{R_i})$
- compute the curve point $A_i = (x_{A_i}, y_{A_i})= u_1G \circ u_2R_i$
- recall the inverse point $-A_i = (x_{A_i}, -y_{A_i})$ is also a valid point.
- if address $\alpha \equiv H'(A_i)$, then $A = A_i$, return,
- if address $\alpha \equiv H'(-A_i)$, then $A = -A_i$ return.
- If none of the conditions satisfies the given address $\alpha$, then the signature is forged.