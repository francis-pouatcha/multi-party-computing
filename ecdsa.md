# ECDSA Signature
## Signature
If a random $S \in_R \mathbb{F_p}$ is the private key, the signature is a pair $(s,r) \in \mathbb{F^2_p}$ where:
- $r$ represents the coordinate $x_R$ of the image $R = ({1 \over k}G)$ of a random parameter $k \in_R \mathbb{F_p}$. Random $k$ is used to mask the secret key,
- $s$ is the signature, computed with the formula $s = k(m+rS)$, where $m=\Eta (M)$ and $\Eta : M \rightarrow \mathbb{F_q}$ is a hash function used to embed the message $M$ into a field of $\mathbb{F_q}$.

Key implementation observations:
- A new random value $k$ must be chosen if $r=0 \texttt{ or }s = 0$.
- The value of $k$ is never disclosed to the public, as knowledge of $k$ will lead to the computation of the private key $S = (sk^{-1} - m)/r$
## Verification
The verification formula requires to reproduce the public image $R'$ of the random point $k$ and compare the coordinatres $r'=x_{R'}=x_R=r$.

In order to compute $R'$ withouth knowing k, we proceed like:

$$
\begin{aligned}
&s = k(m + rS)
\\
&{1 \over k} = {(m + rS) \over s}
\\
&{1 \over k} = {m \over s} + {rS \over s}
\\
&R' = {1 \over k}G = {m \over s}G + {rS \over s}G 
\\
&\texttt{as we know } m, r, s \texttt{ and } A=SG
\\
&R' = {m \over s}G + {r \over s}A
\end{aligned}
$$

The signature is valid if $r = x_{R'}$ and following validation test are positive.

## Further Validations
A real signation validation will require:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{F_q}$ and $1 \lt r,s \lt q$
- validate tha the public key $A = (x_A, y_A)$ is a valid curve point with:
  - is not the identity element $A \ne O$
  - $x_A, y_A \in \mathbb{F_p}$
  - $y_A = E(x_A)$
  - $nA = O$, that A is generated from G.
- validate $|m| \le |p|$. The value of $m$ can be higher than $p$, but the lenght of $m$ in bits can not be longer than the size of $p$ in bits. Thefore $m$ is the $L_n$ most significant bits of H(M).

## Recovering the Public Key A
In the case of public ledger like bitcoin, the public key $A$ is not know, but an image or precisely address $a = \Eta'(A)$. For bitcoin $\Eta'$ is SHA256_RIPMD160.

Knowing the signature imformation $(r, s)$ and the message $M \rightarrow \Eta(M)=m$, we can compute the public key proceeding like:
- verify that $r \ne 0 \texttt{ and } s \ne 0$
- verify that $r,s \in \mathbb{F_q}$ and $1 \lt r,s \lt q$
- compute $u_1 = {m \over r}$ and $u_2 = {s \over r}$
- $\forall_{i\in \mathbb{Z}, r+i \in \mathbb{Z_q}}$:
  - compute the curve point $R_i = (x_{R_i}, y_{R_i})$
  - compute the curve point $A_i = (x_{A_i}, y_{A_i})= u_1G + u_2R_i$
  - the inverse point $-A_i = (x_{A_i}, -y_{A_i})$ is also a valid point.
  - if $a = H'(A_i)$, then A = A_i, return,
  - if $a = H'(-A_i)$, then $A = -A_i$ return.
- If none of the conditions satisfies the given address $a$, then the signature is forged.
