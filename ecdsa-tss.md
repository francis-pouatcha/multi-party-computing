# ECDSA Treshold Signature
In order to enlighten the mistery behind TSS (treshold signature schemes), we want to elaborate details of our reading of following documents: [GG20](https://eprint.iacr.org/2020/540) and [CGGMP](https://eprint.iacr.org/2021/060).

# Treshold ECDSA
For threshold ECDSA, we need a way to compute the signature $(r,s) \in \mathbb{F^2_p}$ where: 
- $R = {1 \over k}G = (x_R, y_R)$,
- $r=x_R$, the $x$ coordinate of the point $R$, and 
- $s=k(m+rS)$, where $S$ is the private key,

whithout disclosing neither $k$ nor $S$ to a single party. 

## Interpolation of Individual Shares of S
The permanent secret value $S$ is the result of a (t, n) interpolation of individual shares $S_h$ held by respective parties $P_h, h \in H = \{1, \dots, n\}$.

As soon as we know the set of participants $C=\{1, \dots, t+1\} \subset H$, involved into the distributed computation, each $P_c$ can interpolate their share $S_c$ into $t+1$ additive parts $w_c$ of $S$ such that $S = \sum_{c=1}^{t+1}w_c$.

 As we can compute the lagrange coefficients for the $(t, t+1)$ secret sharing to be,
$$
l_c(x_s) = \prod_{m=1, m \ne c}^{t+1} \frac{x_s - x_m}{x_c-x_m}
$$

Recalling that $S_c = f_c(x_s)$, the distributed secret could theoreticaly be recovered with the formula
$$
f(x_s) = \sum_{c=1}^{t+1}f_c(x_s)l_c(x_s)= \sum_{c=1}^{t+1}S_cl_c(x_s)
$$

Because each $S_c$ is only known only to party $P_c, c \in C$, none of the parties can compute $f(x_s)$ alone, even though each party $P_c$ is holding is own additive part
$$
w_c = S_cl_c(x_s)
$$

Because $w_c$ is the additive part of the distributed secret, the distributed secret is
$$
f(x_s) = \sum_{c=1}^{t+1}w_c
$$

## Distributed Computation
The TSS challenge consists in coordinating a distributed computation among members of subset $C = \{1, \dots, t+1\} \subset H$, allowing each party $P_c$ to produce a sub-signature $s_c= k_cm + r\sigma_c$, so that
$$
k = \sum_{c \in C}k_c \texttt{ and } kS= \sum_{c \in C}\sigma_c
$$
leading to 
$$
\begin{aligned}
s &= \sum_{c \in C}s_c=\sum_{c \in C}(k_cm + r\sigma_c)
\\
s &= \sum_{c \in C}k_cm + \sum_{c \in C}r\sigma_c
\\
s &= m\sum_{c \in C}k_c + r\sum_{c \in C}\sigma_c
\\
s &= m(k) + r(kS) = k(m + rS)
\end{aligned}
$$

## Generation of Random Paramater (k)
As mentioned above, the parameter $k$, if disclosed leads to the computation of the secret $S$. This scheme muss therefore allow the computation of additive sub-signatures, that digest a $k$ and it public image $r$ without giving the possibility of exposing the value of $k$.

In order to additively produce and use $r$ without disclosing $k$, we selects a new random parameter $\gamma \in_R \mathbb{F_q}$ that multiplies $k$. Even though $k\gamma$ is disclosed, we rely on the [integer factorization](./cha.md#integer-factorization-problem) hardness assumption, to state that a party knowing $k\gamma$ can neither compute $k$ nor $\gamma$ in polynomial time.

To start, each party $P_c, c \in C$ produces two random numbers $k_c$ and $\gamma_c$ so that

$$
k=\sum_{c \in C}k_c \texttt{ and } \gamma=\sum_{c \in C}\gamma_c
$$

then multiplying both sets on indexes $(c,h)$ results in

$$
k\gamma = \sum_{c \in C}k_c * \sum_{c \in C}\gamma_c = \sum_{c,h \in C}k_c\gamma_h
$$

## Commiting to $\gamma_c$
Each party $P_c$ computes a commitment to the generated value $\gamma_c$. The commitment is of the form $[C(\gamma_c), D(\gamma_c)]=Com(g^{\gamma_c})$.

__Broadcast-1__: Party $P_c$ broadcasts $C(\gamma_c)$ to all other parties. Recall that knowing $C(\gamma_c)$ does not disclose the value of $g^{\gamma_c}$.

## Additive Sharing of $k\gamma$
We can use the multiplicative-to-additive share conversion protocol (MtA) to distribute the value $k_c\gamma_h$ additively between $P_c$ and $P_h$. The result of the MtA for $k_c\gamma_h$ is two values $\alpha_{ch}$ and $\beta_{hc}$ so that $k_c\gamma_h = \alpha_{ch} + \beta_{hc}$. 

At the end of the subprotocol, each party $P_c$ is in possession of an additive share of $k\gamma$ called 

$$
\delta_c =k_c\gamma_c + \sum_{h \ne c, h \in C}\alpha_{ch} + \sum_{h \ne c, h \in C}\beta_{hc}
$$

Recall that 

$$
k\gamma = \sum_{c \in C}k_c * \sum_{c \in C}\gamma_c = \sum_{c,h \in C}k_c\gamma_h = \sum_{c \in C}\delta_c
$$

# Production of Secret S
## Individual Shares of Secret (S)
Using the polynomial interpolation, each party $P_c, c \in C$ can compute an [aditive share](./dkg.md#retrieving-the-secret) $w_c$ such that $S=f(x_s) = \sum_{c=1}^{t+1}w_c$.

Even though we can colaboratively compute $S$, we do not want to have $S$ aggregated in a single process space. Further, the value we need in the ECDSA signature is the product $kS$
## Additive Shares of masked Secret (kS)
Recall that 
 $$
 S=\sum_{c \in C}w_c
 $$
 and multiplying both sets $k$ and $S$ on indexes $(c,h)$,
 $$
 kS=\sum_{c \in C}k_c * \sum_{h \in C}w_h = \sum_{c,h \in C}k_cw_h
 $$

We will then use the multiplicative-to-additive share conversion protocol with check (MtAwc) to distribute the value $k_cw_h$ additively between $P_c$ and $P_h$. The result of the MtAwc for $k_cw_h$ will be two values $\mu_{ch}$ and $\nu_{hc}$ so that $k_cw_h = \mu_{ch} + \nu_{hc}$. At the end of the subprotocol, each party $P_c$ is in possession of an additive share of $kS$ called 

$$
\sigma_c =k_cw_c + \sum_{h \ne c, h \in C}\mu_{ch} + \sum_{h \ne c, h \in C}\nu_{hc}
$$

Recall that 
$$
kS = \sum_{c \in C}k_c  * \sum_{h \in C}w_h = \sum_{c,h \in C}k_cw_h = \sum_{c \in C}\sigma_c
$$

# Messaging on Share Conversion
The MtA share conversion performed above will result in pairwise messaging among parties $c,h \in C$, where party $P_c$ will send two messages to party $P_h$.

Both share convertion messages can be batched. They can also be run for many versions of $k, \gamma$ in a prestep as values of messages are not required to run them. 

# Accumulation of Individual Shares
We will rely on the multiparty ECDH assumption to accumulate public images of individual shares. 

Recall that each party $P_c$ has the shares: $k_c, \sigma_c$ such that:
$$kS=\sum_{c \in C}\sigma_c$$ 
and 
$$k=\sum_{c \in C}k_c$$ 

In the first message, each party $P_c$ computes and broadcasts:
- the additive share of $k\gamma$ called $\delta_c$. 
- the public image of the generated random $\gamma_c$ in the form of $\gamma_cG$

Upon receiving the first message, each party can compute
$$
R = {1 \over k\gamma} (\sum_{c \in C}\gamma_cG)= {1 \over k\gamma} (\gamma G)={1 \over k}G
$$
The value of $r$ is the $x$-coordinate of the point $R$, represented as: $$r=(R)_x$$

# Verifying the Public Key
Each party $P_c$ computes and broadcasts the public image of $\sigma_c$ in the form of $\Gamma_c=\sigma_cR$.

Upon reception of all pieces, each party can recompute the public key:
$$
A= \sum_{c \in C} \Gamma_c = (\sum_{c \in C}\sigma_c)R=kS(R)=kS({1 \over k}G)={kS \over k}G=SG
$$

# Additive Signatures
Each party $P_c$ computes and broadcasts the additive signature 
$$s_c=mk_c + r\sigma_c$$

Upon receiving all signature shares, each party can compute the aggregate signature:
$$s = \sum_{c \in C}s_c=m\sum_{c \in C}k_c + r\sum_{c \in C}\sigma_c=mk + rkS = k(m+rS)$$
