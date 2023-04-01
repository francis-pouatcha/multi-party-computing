# ECDSA Threshold Signature
In order to enlighten the mystery behind TSS (threshold signature schemes), we want to elaborate details of our reading of following documents: [GG20](https://eprint.iacr.org/2020/540) and [CGGMP](https://eprint.iacr.org/2021/060).

The definition of bit and byte strings as used in this document is found in [strings expressions](./conv-ser-enc.md).

# Threshold ECDSA
We elaborate on ECDSA in the document [ECDSA](./ecdsa.md). Terms and idioms used here are also defined in [ECDSA](./ecdsa.md).

For threshold ECDSA, we need a way to compute the signature $r_{<(be:o):32>}||s_{<(be:o):32>}$ where 
- $(r,s) \in \mathbb{Z^2_p}$,
- $R = {1 \over k}G = (x_R, y_R)$,
- $r_{<(be:o):32>}=x_R$, the $x$-coordinate of the point $R$, and
- $s_{<(be:o):32>}=k \times (m+(r \times a))$, where $a$ is the private key,

without disclosing neither $k$ nor $a$ to a single party.

Remark that we use $1 \over k$ to compute $R$ instead of $k$. We get the same result, as we use $k$ in the signature computation instead of $1 \over k$. But this approach increases the linearity of the signature computation and makes additive sub-signatures easy to compute.

## Computing Individual Secret Shares $a_h$
We assume secret shares are generated according to the procedure defined in [computing the distributed secret polynomial](./dkg-tss.md#computing-a-distributed-secret-polynomial). At the end of the key generation process, each party $P_h, h \in H = \set{1, \dots, n}$ is in possession of a point $(x_h, f(x_h))$ on the distributed secret polynomial $f(x)$. The value $a_h=f(x_h)$ is called the secret share of $P_h$ and is only known to $P_h$.

## Interpolation of Individual Shares of $w_c$
As soon as we know the set of participants $C=\set{1, \dots, t+1} \subset H$, involved in the distributed computation, each $P_c$ interpolate its share $a_c \equiv f(x_c)$ into $t+1$ additive parts $w_c$ of the distributed secret $a$ such that

$$a = \sum_{c=1}^{t+1}w_c$$

This is explained in detail in [retrieving the secret](./dkg-tss.md#retrieving-the-secret).

Remark that we are using the symbol $a$ to represent the secret instead of $s$, as the last one is used in the page to reference the signature.

## Distributed Computation
The TSS challenge consists in coordinating a distributed computation among members of subset $C = \set{1, \dots, t+1} \subset H$, to jointly produce the signature

$$
\begin{aligned}
s &= k \times (m + (r \times a))
\\
s &= (k \times m) + (r \times k \times a)
\end{aligned}
$$

by allowing each party $P_c$ to produce a sub-signature

$$s_c= (k_c \times m) + (r \times \sigma_c)$$

so that

$$
k = \sum_{c \in C}k_c \texttt{ and } (k \times a)= \sum_{c \in C}\sigma_c
$$

leading to

$$
\begin{aligned}
s &= \sum_{c \in C}s_c
\\
s &=\sum_{c \in C}((k_c \times m) + (r \times \sigma_c))
\\
s &= \sum_{c \in C}(k_c \times m) + \sum_{c \in C}(r \times \sigma_c)
\\
s &= (m \times \sum_{c \in C}k_c) + (r \times \sum_{c \in C}\sigma_c)
\\
s &= (m \times k) + (r\times k \times a)
\\
s &= k \times (m + (r \times a))
\end{aligned}
$$

The aggregation of the individual signatures $s_c$ is therefore equivalent to a complete signature performed with the secret key $a$.

## Generation of Random Parameter $k$
As mentioned above, the parameter $k$, if disclosed, leads to the computation of the secret $a$. This scheme must therefore allow the computation of additive sub-signatures, that digest $k$ and the public image of its inverse $r = x_R$, where $R = {1 \over k}G$ without exposing the value of $k$.

In order to additively produce and use $r$ without disclosing $k$, we select a new random parameter $\gamma$ from $\mathbb{Z_q}$ that multiplies $k$. Even though $k\gamma$ is disclosed, we rely on the [integer factorization](./cha.md#integer-factorization-problem) hardness assumption, to state that a party knowing $k\gamma$ can neither compute $k$ nor $\gamma$ in polynomial time.

We nevertheless state that exposing $k\gamma$ formally weakens the security of threshold ECDSA when compared to the single party ECDSA, as it introduces an additional assumption.

To start, each party $P_c, c \in C$ produces two random numbers $k_c$ and $\gamma_c$ so that

$$
k=\sum_{c \in C}k_c \texttt{ and } \gamma=\sum_{c \in C}\gamma_c
$$

then multiplying both sets on indexes $(c,h)$ results in

$$
k\gamma = \sum_{c \in C}k_c * \sum_{c \in C}\gamma_c = \sum_{c,h \in C}k_c\gamma_h
$$

## Committing to $\gamma_c$
Each party $P_c$ computes a commitment to the generated value $\gamma_c$. The commitment is of the form $[C(\gamma_c), D(\gamma_c)]=Com(g^{\gamma_c})$.

Each Party $P_c$ broadcasts $C(\gamma_c)$ to all other parties. Recall that knowing $C(\gamma_c)$ does not disclose the value of $g^{\gamma_c}$.

## Additive Sharing of $k\gamma$
We then use the multiplicative-to-additive share conversion protocol ([MtA - see GG20](https://eprint.iacr.org/2020/540)) to distribute the value $k_c\gamma_h$ additively between every pairwise combination parties $P_c, P_h \in C$. The result of the MtA for $k_c\gamma_h$ is two values $\alpha_{ch}$ and $\beta_{hc}$ so that $k_c\gamma_h = \alpha_{ch} + \beta_{hc}$.

At the end of the subprotocol, each party $P_c$ is in possession of an additive share of $k\gamma$ called

$$
\delta_c =k_c\gamma_c + \sum_{h \ne c, h \in C}\alpha_{ch} + \sum_{h \ne c, h \in C}\beta_{hc}
$$

Recall that

$$
k\gamma = \sum_{c \in C}k_c * \sum_{c \in C}\gamma_c = \sum_{c,h \in C}k_c\gamma_h = \sum_{c \in C}\delta_c
$$

# Production of Secret $a$
## Individual Shares of Secret $w_c$
Using the polynomial interpolation, each party $P_c, c \in C$ computes an [additive share](./dkg.md#retrieving-the-secret) $w_c$ such that

$$a = \sum_{c=1}^{t+1}w_c$$

Even though we can collaboratively compute $a$, we do not want to have $a$ aggregated in a single process space. Further, the value we need in the ECDSA signature is the product $(k \times a)$
## Additive Shares of masked Secret $(k \times a)$
Recall that

$$
a=\sum_{c \in C}w_c
$$

and multiplying both sets $k$ and $a$ on indexes $(c,h)$ results in:

$$
(k \times a)=\sum_{c \in C}k_c \times \sum_{h \in C}w_h = \sum_{c,h \in C}(k_c \times w_h)
$$

We use the multiplicative-to-additive share conversion protocol with check ([MtAwc - see GG20](https://eprint.iacr.org/2020/540)) to distribute the value $(k_c \times w_h)$ additively between $P_c$ and $P_h$. The result of the MtAwc for $(k_c \times w_h)$ are values $\mu_{ch}$ and $\nu_{hc}$ so that $(k_c \times w_h) = \mu_{ch} + \nu_{hc}$. At the end of the subprotocol, each party $P_c$ is in possession of an additive share of $(k \times a)$ called

$$
\sigma_c =k_cw_c + \sum_{h \ne c, h \in C}\mu_{ch} + \sum_{h \ne c, h \in C}\nu_{hc}
$$

Recall that

$$
(k \times a) \equiv \sum_{c \in C}k_c \times \sum_{h \in C}w_h = \sum_{c,h \in C}k_cw_h = \sum_{c \in C}\sigma_c
$$

# Communication Rounds during Share Conversion
The MtA share conversion performed above results in pairwise communications among parties $c,h \in C$, where each party $P_c$ sends two messages to party $P_h$.

Both share conversion messages can be batched. They can also be run for many versions of $k, \gamma$ in a prestep, as values of $k \text{ and } \gamma$ are independent of the message to sign.

# Accumulation of Additive Shares of $k\gamma$
Recall that each party $P_c$ has the shares: $k_c, \sigma_c$ such that:

$$
(k \times a)=\sum_{c \in C}\sigma_c \text{ and } k=\sum_{c \in C}k_c
$$

In the first message, each party $P_c$ computes and broadcasts:
- the additive share of $k\gamma$ called $\delta_c$, and
- the public image of the generated random $\gamma_c$ in the form of $\gamma_cG$, a.k.a decommitment.

Upon receiving the first message, each party $P_c$:

- verifies the integrity of sent $\gamma_c$
- computes

$$
R = {1 \over k\gamma} (\sum_{c \in C}\gamma_cG)= {1 \over k\gamma} (\gamma G)={1 \over k}G
$$

The value of $r$ is the $x$-coordinate of the point $R$, represented as: $r=x_R$.

# Verifying the Public Key
Each party $P_c$ computes and broadcasts the public image of $\sigma_c$ in the form of $\Gamma_c=\sigma_cR$.

Upon reception of all pieces, each party can recompute the public key:

$$
\begin{aligned}
A&= \sum_{c \in C} \Gamma_c
\\
A&= (\sum_{c \in C}\sigma_c)R
\\
A&=(k \times a)(R)
\\
A&=(k \times a)({1 \over k}G)
\\
A&={(k \times a) \over k}G
\\
A&=aG
\end{aligned}
$$

# Additive Signatures
Each party $P_c$ computes and broadcasts the additive signature
$$s_c=(m \times k_c) + (r \times \sigma_c)$$

Upon receiving all signature shares, each party can compute the aggregate signature:

$$
\begin{aligned}
s &= \sum_{c \in C}s_c
\\
s &=m \times \sum_{c \in C}k_c + r \times \sum_{c \in C}\sigma_c
\\
s &=(m \times k) + (r \times k \times a)
\\
s &=k \times (m+ (r \times a))
\end{aligned}
$$

# Security Consideration
The MtAwc requires a range proof of values shared among parties, as [GG20](https://eprint.iacr.org/2020/540) and [CGGMP](https://eprint.iacr.org/2021/060) use the paillier homomorphic encryption to perform MtA.

This aspect was omitted here for simplicity.

# Next
Proceed with [Threshold signature scheme (TSS) on Schnorr Signature - BIP340](./schnorr-tss.md).
