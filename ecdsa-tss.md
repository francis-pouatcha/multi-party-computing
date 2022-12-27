# ECDSA Threshold Signature
In order to enlighten the mystery behind TSS (threshold signature schemes), we want to elaborate details of our reading of following documents: [GG20](https://eprint.iacr.org/2020/540) and [CGGMP](https://eprint.iacr.org/2021/060).

# Threshold ECDSA
We elaborate on ECDSA in the document [ECDSA](./ecdsa.md). For threshold ECDSA, we need a way to compute the signature $(r,s) \in \mathbb{Z^2_p}$ where:
- $R = {1 \over k}G = (x_R, y_R)$,
- $r=x_R$, the $x$-coordinate of the point $R$, and
- $s=k \times (m+(r \times a))$, where $a$ is the private key,

without disclosing neither $k$ nor $a$ to a single party.

## Computing Individual Secret Shares $a_h$
We assume secret shares are generated according to the procedure defined in [computing the distributed secret polynomial](./dkg-tss.md#computing-a-distributed-secret-polynomial). At the end of the key generation process, each party $P_h, h \in H = \\{1, \dots, n\\}$ is in possession of a point $(x_h, f(x_h))$ on the distributed secret polynomial $f(x)$. The value $a_h=f(x_h)$ is called the secret share of $P_h$ and is known only to $P_h$.

## Interpolation of Individual Shares of $w_c$
As soon as we know the set of participants $C=\\{1, \dots, t+1\\} \subset H$, involved into the distributed computation, each $P_c$ can interpolate their share $a_c$ into $t+1$ additive parts $w_c$ of distributed secret $a$ such that 

$$a = \sum_{c=1}^{t+1}w_c$$

This is done in detail in [retrieving the secret](./dkg-tss.md#retrieving-the-secret).

Remark that we are using the symbol $a$ to represent the secret instead of $s$, as the last one is used in the page to reference the signature.

## Distributed Computation
The TSS challenge consists in coordinating a distributed computation among members of subset $C = \\{1, \dots, t+1\\} \subset H$, to jointly produce the signature

$$
\begin{aligned}
s &= k \times (m+ r \times a)
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
s &= \sum_{c \in C}s_c=\sum_{c \in C}(k_c \times m + r \times \sigma_c)
\\
s &= \sum_{c \in C}k_c \times m + \sum_{c \in C}(r \times \sigma_c)
\\
s &= m \times \sum_{c \in C}k_c + r \times \sum_{c \in C}\sigma_c
\\
s &= m \times k + (r\times k \times a)
\\
s &= k \times (m + (r \times a))
\end{aligned}
$$

The aggregation of the individual signatures $s_c$ is therefore equivalent to a complete signature performed with the secret key $a$.

## Generation of Random Parameter (k)
As mentioned above, the parameter $k$, if disclosed, leads to the computation of the secret $a$. This scheme muss therefore allow the computation of additive sub-signatures, that digest a $k$ and it public image of its inverse $r = x_R \leftarrow R = {1 \over k}G$ without giving the possibility of exposing the value of $k$.

In order to additively produce and use $r$ without disclosing $k$, we select a new random parameter $\gamma \in_{rand} \mathbb{Z_q}$ that multiplies $k$. Even though $k\gamma$ is disclosed, we rely on the [integer factorization](./cha.md#integer-factorization-problem) hardness assumption, to state that a party knowing $k\gamma$ can neither compute $k$ nor $\gamma$ in polynomial time.

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

__Broadcast-1__: Party $P_c$ broadcasts $C(\gamma_c)$ to all other parties. Recall that knowing $C(\gamma_c)$ does not disclose the value of $g^{\gamma_c}$.

## Additive Sharing of $k\gamma$
We then use the multiplicative-to-additive share conversion protocol ([MtA - see GG20](https://eprint.iacr.org/2020/540)) to distribute the value $k_c\gamma_h$ additively between $P_c$ and $P_h$. The result of the MtA for $k_c\gamma_h$ is two values $\alpha_{ch}$ and $\beta_{hc}$ so that $k_c\gamma_h = \alpha_{ch} + \beta_{hc}$.

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
## Additive Shares of masked Secret ($k \times a$)
Recall that
$$
a=\sum_{c \in C}w_c
$$
and multiplying both sets $k$ and $a$ on indexes $(c,h)$ results in:

$$
(k \times a)=\sum_{c \in C}k_c * \sum_{h \in C}w_h = \sum_{c,h \in C}(k_c \times w_h)
$$

We will then use the multiplicative-to-additive share conversion protocol with check (MtAwc) to distribute the value $(k_c \times w_h)$ additively between $P_c$ and $P_h$. The result of the MtAwc for $(k_c \times w_h)$ will be two values $\mu_{ch}$ and $\nu_{hc}$ so that $(k_c \times w_h) = \mu_{ch} + \nu_{hc}$. At the end of the subprotocol, each party $P_c$ is in possession of an additive share of $(k \times a)$ called

$$
\sigma_c =k_cw_c + \sum_{h \ne c, h \in C}\mu_{ch} + \sum_{h \ne c, h \in C}\nu_{hc}
$$

Recall that
$$
(k \times a) \equiv \sum_{c \in C}k_c * \sum_{h \in C}w_h = \sum_{c,h \in C}k_cw_h = \sum_{c \in C}\sigma_c
$$

# Messaging on Share Conversion
The MtA share conversion performed above will result in pairwise messaging among parties $c,h \in C$, where party $P_c$ will send two messages to party $P_h$.

Both share conversion messages can be batched. They can also be run for many versions of $k, \gamma$ in a prestep, as values of the message to sign are not required to run them.

# Accumulation of Individual Shares
We will rely on the multiparty ECDH assumption to accumulate public images of individual shares.

Recall that each party $P_c$ has the shares: $k_c, \sigma_c$ such that:

$$(k \times a)=\sum_{c \in C}\sigma_c$$

and

$$k=\sum_{c \in C}k_c$$

In the first message, each party $P_c$ computes and broadcasts:
- the additive share of $k\gamma$ called $\delta_c$.
- the public image of the generated random $\gamma_c$ in the form of $\gamma_cG$

Upon receiving the first message, each party can compute

$$
R = {1 \over k\gamma} (\sum_{c \in C}\gamma_cG)= {1 \over k\gamma} (\gamma G)={1 \over k}G
$$

The value of $r$ is the $x$-coordinate of the point $R$, represented as: $$r=x_R$$

# Verifying the Public Key
Each party $P_c$ computes and broadcasts the public image of $\sigma_c$ in the form of $\Gamma_c=\sigma_cR$.

Upon reception of all pieces, each party can recompute the public key:
$$
A= \sum_{c \in C} \Gamma_c = (\sum_{c \in C}\sigma_c)R=(k \times a)(R)=(k \times a)({1 \over k}G)={(k \times a) \over k}G=aG
$$

# Additive Signatures
Each party $P_c$ computes and broadcasts the additive signature
$$s_c=m \times k_c + r \times \sigma_c$$

Upon receiving all signature shares, each party can compute the aggregate signature:
$$
\begin{aligned}
s &= \sum_{c \in C}s_c
\\
s &=m \times \sum_{c \in C}k_c + r \times \sum_{c \in C}\sigma_c
\\
s &=m \times k + r \times k \times a
\\
s &=k \times (m+ r \times a)
\end{aligned}
$$