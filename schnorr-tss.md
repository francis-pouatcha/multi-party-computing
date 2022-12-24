# Schnorr Signature

[Schnoor Signature](https://en.wikipedia.org/wiki/Schnorr_signature) is a digital signature scheme known for its simplicity and whose security is based on the intractability of certain discrete logarithm problems. It is efficient and generates short signatures.

## Schnorr Groups vs. Elliptic Curve Group
Even though the original application of Schnorr Signature was defined over [Schnorr Groups](https://en.wikipedia.org/wiki/Schnorr_group), the most popular application defines Schnorr over Elliptic Curve Groups.

## EC Schnorr Signature vs. ECDSA
The Schnorr Signature as defined in BIP340 shows a lot of advantages over ECDSA:
### Provable security
EC Schnorr Signature is strongly unforgeable under chosen message attack (SUF-CMA) in the random oracle model assuming the hardness of the elliptic curve discrete logarithm problem (ECDLP).
### Non-malleability
The SUF-CMA security of Schnorr signatures implies that they are non-malleable. On the other hand, ECDSA signatures are inherently malleable. This issue is discussed in [BIP62](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki) and [BIP146](https://github.com/bitcoin/bips/blob/master/bip-0146.mediawiki).
### Linearity
Schnorr signatures provide a simple and efficient method that enables multiple collaborating parties to produce a signature that is valid for the sum of their public keys. This is the foundation for multisignatures, aggregated signatures and treshold signature schemes.

## Signature
As defined by BIP340, EC Schnorr signature for 
- message $m$ and 
- public key $A=aG$ 

is a pair $(R, s) \in \mathbb{E_{(F_p)}} \times \mathbb{Z_p}$, where
- the point $R = rG$, where $r \in \mathbb{Z_p}$ is a randomly selected large prime from $\mathbb{Z_p}$,
- the number $s \equiv r + u \times a \pmod p$, whereby $s, r, u, a \in \mathbb{Z_p}$ and 
  - $u = H(R, A, m)$: the hash of the concatenated random point $R$, public key $A$ and message $m$, and 
  - $H$ a hash function. In the case of BIP340 this is a tagged hash function to avoid collision with existing functionality.

## Verification
A signature is verified by applying 
$$
\begin{aligned}
R+uA &= sG
\\
R+uA &= (r + u \times a)G
\\
R+uA &= rG + (u \times a)G
\\
R+uA &= rG + u(aG)
\\
R+uA &= R + uA
\end{aligned}
$$
## Key Prefixing
In order to prevent the [related key attack](https://en.wikipedia.org/wiki/Related-key_attack), BIP340 prepends the public key to the message $u = hash\_tagged(R || A || m)$. In this model, the public key $A$ must be present to allow for signature verification, as $A$ is part of the message hash, it can not just be extracted from the signature (like is the case with DSA).

## Encoding R and public key point P
For the purpose of backward compatibility, BIP340 goes for compressed representation of both public key $A$ and random point $R$, resulting to $33$-bits keys, but __implicitely__ allways choses the points with the __even__ $y$-coordinate.

# Schnorr Signature Aggregation
A signature aggregation scheme for Schnorr allows $n$-of-$n$ multisignatures which, from a verifier's perspective, are no different from ordinary signatures.

Let a group of $n$ signers with the set $L = \{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\}$ be their public keys, means the common secret $a = \sum_{h=1}^na_h$.

In order to sign a message $m$, each signer $P_h$
- selects a large random prime $r_h \in \mathbb{Z_p}$ and 
- computes and publishes $R_h = r_hG$

Upon receiving all $R_h$, each signer $P_h$
- computes the random number $R = \sum_{h=1}^n R_h$
  - these are simple point additions
  - $R = \sum_{h=1}^n R_h \equiv (\sum_{h=1}^nr_h)G = rG$, where
  - $r \equiv \sum_{h=1}^nr_h \pmod p$
- computes the public key $A = \sum_{h=1}^n A_h$
- computes the message hash $u = H(R, A, m)$
- computes the aggregate $s_h = r_h + u \times a_h$
- publishes $s_h$

Upon receiving all $s_h$, each signer $P_h$ can compute $s = \sum_{h=1}^ns_h$ which is equivalent to:
$$
\begin{aligned}
s &= \sum_{h=1}^ns_h
\\
s &= \sum_{h=1}^n(r_h + u \times a_h)
\\
s &= \sum_{i=1}^nr_h + \sum_{h=1}^n(u \times a_h)
\\ 
s &= \sum_{h=1}^nr_h + u \times \sum_{h=1}^na_h
\\
s &= r + u \times a
\end{aligned}
$$

## Rogue-key attack
This protocol is vulnerable to a rogue-key attack where a corrupted signer reports its public key $A_n = A - \sum_{h=1}^{n-1}A_h$ to orchestrate the production of a preknown public key $A = aG$. The reporter will then be able to use known private key $a$ to produce signatures without the collaboration of other parties.

## BIP32 HD Keys
Using [BIP32](./bip32.md) to derive all $A_h$ from the same chain code will allow parties to prevent one of them to be malicious.

In this use case, each party $P_h$ :
- will publish an extended public key $EX_h$. 
- will compute the common extended public key $EX = \sum_{h=1}^n EX_h$

Before publishing a public key $X$, participants have to aggree on a common chain code. A verifiable random function can be used to produce common random nonce . E.g. $nonce$ being the hash of the last block on a network.

The hash function $H_1(EX, nonce)$ will be used to produce the common chain code used by all parties to derive $X_h$ from $EX_h$.

In order to sign a message $m$, a hash function $H_2(EX, m)$ will be used to compute the chaincode used to derive $R_h$ from $EX_h$. Thus removing any presign communication rounds.

With this approach, each party $P_h$ will compute X and R without having to communicate with each other.

# Schnorr Treshold Signature
## Treshold Secret Sharing
Treshold signature requires a $t$ of $n$ secret sharing scheme as defined in [DKG-TSS](./dkg-tss.md).

## Treshold Schnorr
Let a group of $n$ signers with the set $L = \{X_1=x_1G, x_2=x_2G, \dots, X_n=x_nG\}$ be their public keys. The common public key $X = \sum_{i=1}^n X_i$.

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
In order to sign a message $m$, we need $t+1$ signers $P_c, c \in C=\{1, \dots, t+1\} \subset H$ to each:
- select a ramdom number $r_h \in_R \mathbb{Z_p}$ and compute and publishes $R_c = r_cG$
- compute the random number $R = \sum_{c=1}^{t+1} R_c$
- compute the message hash $\gamma = H(R, X, m)$
- compute the aggregate $s_c = r_c + \gamma w_c$, where $w_c$ is the additive share held by $P_c$ for the distributed secret $S = \sum_{h \in H}x_h = \sum_{c \in C}w_c$
- publish $s_h$
- compute $S = \sum_{h=1}^{t+1}s_h$

## BIP32 HD Keys
Using [BIP32](./bip32.md) key derivation to enforce generation of $X_h$ and $R_h$ will help mitigate rogue key attack.
