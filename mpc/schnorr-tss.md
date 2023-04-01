# Schnorr Signature

[Schnorr Signature](https://en.wikipedia.org/wiki/Schnorr_signature) is a digital signature scheme known for its simplicity and whose security is based on the intractability of certain discrete logarithm problems. It is efficient and generates short signatures.

## Schnorr Groups vs. Elliptic Curve Group
Even though the original application of Schnorr Signature was defined over [Schnorr Groups](https://en.wikipedia.org/wiki/Schnorr_group), the most popular application defines Schnorr over Elliptic Curve Groups.

## Elliptic Curves
Let the expression $\mathbb{Z_q}$ represent the field of integers modulo $q$ written $(\mathbb{Z}, +, \times, q)$.

Let the expression $\mathbb{E_{(\mathbb{Z_q})}}$ represent an [elliptic-curve group](./ecgroups.md) $(\mathbb{E_{(\mathbb{Z_q})}}, \circ, O, G, p)$, defined over $\mathbb{Z_q}$ where
- $\circ$ is the binary group operation, a.k.a addition of points,
- O is the identity element, a.k.a neutral element
- $G$ is the generator of the group,
- p is the order of the generator $G$,

Reviewing [finite cyclic elliptic curve groups](./ecgroups.md#finite-cyclic-groups-over-elliptic-curves) is essential for the understanding of ECDSA maths below.

## EC Schnorr Signature vs. ECDSA
The Schnorr Signature as defined in [BIP340](https://github.com/bitcoin/bips/blob/master/bip-0340.mediawiki) shows a lot of advantages over ECDSA:
### Provable security
EC Schnorr Signature is strongly unforgeable under chosen message attack (SUF-CMA) in the random oracle model assuming the hardness of the elliptic curve discrete logarithm problem (ECDLP).
### Non-malleability
The SUF-CMA security of Schnorr signatures implies that they are non-malleable. On the other hand, ECDSA signatures are inherently malleable. This issue is discussed in [BIP62](https://github.com/bitcoin/bips/blob/master/bip-0062.mediawiki) and [BIP146](https://github.com/bitcoin/bips/blob/master/bip-0146.mediawiki).
### Linearity
Schnorr signatures provide a simple and efficient method that enables multiple collaborating parties to produce a signature that is valid for the sum of their public keys. This is the foundation for multi signatures, aggregated signatures and threshold signature schemes.

## EC Schnorr Signature (BIP340)
### Point Encoding
BIP340 specifies a 64-byte Schnorr signature scheme over the elliptic curve secp256k1. EC Point are therefore encoded in the format

$$P=(x_P,y_P) \rightarrow x_{R<(be:o):32>}||\mathrm{0x02}_{<(o):1>}$$

This is the $32$ bytes big endian encoding of the $x$-coordinate, which $y$-coordinate, as BIP340 always __implicitly__ chooses the even $y$-coordinate that is even. Despite the implicit selection, BIP340 adds the indication of the $y$-coordinate, always $\mathrm{0x02}$ to keep backward compatibility with compressed point encoding of ECDSA. In this document we use the notation $Point_{<(bip340:o):33>}$ to represent an encoded point.

### Signature
As defined by BIP340, EC Schnorr signature for
- message $M$ and
- public key $A=aG$

is the string $\sigma_{<(o):65>} = R_{<(bip340:o):33>}||s_{<(be:o):32>}$, where
- the point $R = rG, r \in \mathbb{Z_p}$ is a randomly selected large prime from $\mathbb{Z_p}$,
- the number $s \equiv r + (u \times a) \pmod p$, whereby $s, r, u, a \in \mathbb{Z_p}$ and
- $u = sha256(TAG_{<(b):?>}||R_{<(bip340:o):33>}||A_{<(bip340:o):33>}||M_{<(b):?>})$

## Verification
A signature is verified by applying

$$
\begin{aligned}
R \circ uA &= sG
\\
R \circ uA &= (r + (u \times a))G
\\
R \circ uA &= rG \circ (u \times a)G
\\
R \circ uA &= rG \circ u(aG)
\\
R \circ uA &= R \circ uA
\end{aligned}
$$

## Key Prefixing
In order to prevent the [related key attack](https://en.wikipedia.org/wiki/Related-key_attack), BIP340 prepends the public key to the message $u = sha256(TAG_{<(b):?>}||R_{<(bip340:o):33>}||A_{<(bip340:o):33>}||M_{<(b):?>})$ see above. In this model, the public key $A$ must be present to allow for signature verification, as $A$ is part of the message hash, it can not just be extracted from the signature (like is the case with DSA).

# Schnorr Signature Aggregation
The linearity of Schnorr allows the possibility of implementing signature aggregation.

## Forgeable Aggregation
Let consider following signatures and their verification in $\mathbb{E}$

$$
s_1 = r_1 + (u_1 \times a_1) \implies s_1G = R_1 \circ u_1A_1
\\
s_2 = r_2 + (u_2 \times a_2) \implies s_2G = R_2 \circ u_2A_2
$$

A naïve aggregation will sum up the signatures to:

$$
AS = \sum_{h}(s_hG) = \sum_{h}R_h \circ \sum_{h}(u_hA_h)
$$

This aggregation is exposed to the weakness that each signer $P_h$ can use his key $a_h$ to forge a signature that can fit into the aggregated signature. Find more detail the [original paper YY22](https://eprint.iacr.org/2022/222)

## Half Aggregated Signature
Let $\mu_{<(be:o):32>} = u_{1<(be:o):32>}||u_{2<(be:o):32>}||\dots||u_{n<(be:o):32>}$ be the concatenated bit string of all message hashes.

Let $\mu_h=sha256(\mu_{<(be:o):32>}||h_{<(be:o):32>})$, the global message for entry $h$

Let $\sigma_h = \mu_h \times s_h$, the aggregate signature entry for signature entry $h$

The Half Aggregated signature is the tuple $(\gamma_{<(be:o):32>}, R_{1<(bip340:o):33>},\dots,R_{n<(bip340:o):33>})$, where

$$
\gamma = \sum_{h} \sigma_h
$$

Storing $n$ signatures would require $(n \times 65)$ octets while storing a half aggregated signature with $n$ entries will require $(32 + (n \times 33))$, reducing storage capacity for signatures by $(32 \times (n-1)) \text{ or } 50\\%$ with increasing number $n$.

### Verification of Half Aggregated Signature
In order to verify this signature:

$$
\begin{aligned}
\gamma G &= (\sum_{h} \sigma_h)G
\\
\gamma G &= (\sum_{h} (\mu_h \times s_h))G
\\
\gamma G &= \sum_{h} (\mu_h (s_hG))
\\
\gamma G &= \sum_{h} (\mu_h(r_h + (u_h \times a_h))G)
\\
\gamma G &= \sum_{h} (\mu_h(r_hG \circ (u_h \times a_h)G))
\\
\gamma G &= \sum_{h} (\mu_h(r_hG \circ u_h(a_hG)))
\\
\gamma G &= \sum_{h} (\mu_h(R_h \circ u_hA_h)) \text{ contains only public info}
\end{aligned}
$$

The last expression containing only publicly available information, shows that it is a valid verification of the half aggregated signature $\sigma$. This assertion means that all contained signatures are valid.

### Verifying the Membership of an Entry
Verifying the membership of a single entry boils down to verifying the whole aggregated signature, and is therefore not necessary. Below is the proof.

Recall the half aggregate signature is:

$$
\begin{aligned}
\gamma G &= (\sum_{h} \sigma_h)G
\end{aligned}
$$

A single entry is reconstructed following these steps:

$$
\begin{aligned}
\mu_m &= sha256(\mu_{<(be:o):32>}||m_{<(be:o):32>})
\\
\sigma_m &= \mu_m \times s_m
\end{aligned}
$$

Recall that the stored half aggregated signature does not store the value $s_m$. But is assume $s'_m$ the valid signature, then

$$
\begin{aligned}
\sigma_mG &= (\mu_m \times s_m)G
\\
\sigma_mG &= \mu_m(s_mG)
\\
\sigma_mG &= \mu_m(R_m \circ u_mA_m) \text{ given } s_mG = R_m \circ u_mA_m
\\
\delta_m G &= (\gamma - \sigma_m) G
\\
\delta_m G &= \gamma G \ominus \sigma_m G
\end{aligned}
$$

Where $\ominus$ is the inverse group operation, or the group subtraction.

As we can also compute $\delta_m$ using the sum of public images, the signature $s_m$ in included if

$$
\begin{aligned}
\delta_m G &= \sum_{h \ne m} (\mu_h(R_h \circ u_hA_h)) \equiv \gamma G \ominus \sigma_m G
\end{aligned}
$$

This shows that verifying the membership of a single entry means verifying the validity of the delta and per consequence verifying the whole signature.

### Verifying a known Membership
The the prover can provide the signature $s_m$, and by verifying that $s_mG = R_m \circ u_mA_m$, we also verify that $s_m$ is included in the aggregated $\gamma$ with

## Incremental Aggregated Signature
For the half aggregated signature described above, the aggregator has to wait for all signatures to present before starting with the aggregation process. Aggregating incrementally might save space and improve asynchronicity.

Let
- $\mu_{1<(be:o):32>} = sha256(u_{1<(be:o):32>})$
- $\mu_{2<(be:o):32>} = sha256(\mu_{1<(be:o):32>}|| u_{2<(be:o):32>})$
- $\dots$
- $\mu_{n<(be:o):32>} = sha256(\mu_{{n-1}<(be:o):32>}|| u_{n<(be:o):32>})$

Let $\sigma_h = \mu_h \times s_h$, the aggregate signature entry for signature entry $h$

The incremental aggregated signature is the tuple $(\gamma_{n<(be:o):32>}, R_{1<(bip340:o):33>},\dots,R_{n<(bip340:o):33>})$, where

$$
\begin{aligned}
\gamma_1 &= \sigma_1
\\
&\text{and}
\\
\gamma_h &= \gamma_{h-1} + \sigma_h
\end{aligned}
$$

# Schnorr n-of-n Multi Signature
A signature aggregation scheme for Schnorr allows $n$-of-$n$ multi signatures which, from a verifier's perspective, are no different from ordinary signatures.

Let a group of $n$ signers with the set $L = \{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\}$ be their public keys, means the common secret $a = \sum_{h=1}^na_h$.

In order to sign a message $M$, each signer $P_h$
- selects a large random prime $r_h \in \mathbb{Z_p}$ and
- computes and publishes $R_h = r_hG \text{ and } A_h=a_hG$

Upon receiving all $R_h, A_h$, each signer $P_h$
- computes the random number $R = \sum_{h=1}^n R_h$. these are simple point additions
  
$$
\begin{aligned}
R &= \sum_{h=1}^n R_h \equiv (\sum_{h=1}^nr_h)G = rG \text{, where }
\\
r &\equiv \sum_{h=1}^nr_h \pmod p
\end{aligned}
$$

- computes the public key 

$$A = \sum_{h=1}^n A_h$$

Having the values $R, A$, each signer
- computes the message hash $u = sha256(TAG_{<(b):?>}||R_{<(bip340:o):33>}||A_{<(bip340:o):33>}||M_{<(b):?>})$
- computes the partial signature scalar $s_h = r_h + (u \times a_h)$
- publishes the signature scalar $s_h$

Upon receiving all $s_h$, each signer $P_h$ can compute $s = \sum_{h=1}^ns_h$ which is equivalent to:
$$
\begin{aligned}
s &= \sum_{h=1}^ns_h
\\
s &= \sum_{h=1}^n(r_h + (u \times a_h))
\\
s &= \sum_{i=1}^nr_h + \sum_{h=1}^n(u \times a_h)
\\
s &= \sum_{h=1}^nr_h + (u \times \sum_{h=1}^na_h)
\\
s &= r + (u \times a)
\end{aligned}
$$

Each party can publish the signature string $\sigma_{<(o):65>} = R_{<(bip340:o):33>}||s_{<(be:o):32>}$

## Rogue-key attack
This protocol is vulnerable to a rogue-key attack where a corrupted signer reports its public key $A_n = A - \sum_{h=1}^{n-1}A_h$ to orchestrate the production of a preknown public key $A = aG$. The reporter will then be able to use a known private key $a$ to produce signatures without the collaboration of other parties. Details can be found [here](https://courses.csail.mit.edu/6.857/2020/projects/4-Elbahrawy-Lovejoy-Ouyang-Perez.pdf).

## BIP32 HD Keys for n-of-n
Using [BIP32](./bip32.md) to produce deterministic $A_h$ and $R_h$ from the same pretested child key indexes will help parties prevent one of them to be malicious.

Recall children key indexes might derive invalide keys, all parties agree upon idex ranges that are tested by all parties.

# Schnorr Threshold Signature
## Threshold Secret Sharing
Threshold signature requires a $t$ of $n$ secret sharing scheme as defined in [DKG-TSS](./dkg-tss.md).

## Threshold Signature
Assume a group of $n$ signers $P_h, h \in H = \\{1, \dots, n\\}$, with the set $L = \\{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\\}$ of their respective public keys. The common public key 

$$A = \sum_{h=1}^n A_h$$

## Interpolation of Individual Shares of $w_c$
As soon as we know the set of participants $C=\\{1, \dots, t+1\\} \subset H$, involved in the distributed computation, each $P_c$ interpolate its share $a_c \equiv f(x_c)$ into $t+1$ additive parts $w_c$ of the distributed secret $a$ such that

$$a = \sum_{c=1}^{t+1}w_c$$

This is explained in detail in [retrieving the secret](./dkg-tss.md#retrieving-the-secret).

Remark that we are using the symbol $a$ to represent the secret instead of $s$, as the last one is used in the page to reference the signature.

## Distributed Computation
The TSS challenge consists in coordinating a distributed computation among members of subset $C = \\{1, \dots, t+1\\} \subset H$, to jointly produce the signature of the message $M$.

Each $P_c$ must select a random number $r_c \in_R \mathbb{Z_p}$ and compute and publish $R_c = r_cG$.

Upon receiving all $R_c$, each member $P_c$ must:
- compute the random number

$$R = \sum_{c=1}^{t+1} R_c$$

- compute the common message hash $u = sha256(TAG_{<(b):?>}||R_{<(bip340:o):33>}||A_{<(bip340:o):33>}||M_{<(b):?>})$
- compute the partial signature scalar $s_c = r_c + (u \times w_c)$
- publish $s_c$

Upon receiving $s_c$, each party can compute the common signature

$$s = \sum_{h=1}^{t+1}s_c$$

This signature will verify like any other schnorr signature.

## Schnorr TSS and BIP32 HD Keys
### Preventing Rogue-key attack
Using [BIP32](./bip32.md) key derivation to enforce generation of $A_h$ and $R_h$ will help mitigate rogue key attack like described in [BIP32 HD Keys for n-of-n](#bip32-hd-keys-for-n-of-n)

### Reducing communication while Using BIP32
Using BIP32, we create the opportunity of generating public keys $A_h$ without having to know the secret scala $a_h$. This works because we use the linear property of elliptic curve groups.

Having proceeded with an $t \text{ of } n$ secret sharing to produce a threshold key generation, we can also generate additive shares that work with interpolated secret shares $w_c$.

Knowing


we can generate a [neutered random number](./bip32.md#neutered-derivation) $n_{pub:i<(be:o):32>}$ for the child key index $i$.

As we know the number of co signers $|C|=t+1$, we can divide the neutered random number by $t+1$ and augment the result to the interpolated secret shares.

$$
\begin{aligned}
a_{parent} &= \sum_{c=1}^{t+1}w_{parent:c}
\\
a_{child:i} &= n_{pub:i} + a_{parent}
\\
a_{child:i} &= {(t+1) \times n_{pub:i} \over {t+1}} + \sum_{c=1}^{t+1}w_{parent:c}
\\
a_{child:i} &= \sum_{c=1}^{t+1}({n_{pub:i} \over {t+1}}) + \sum_{c=1}^{t+1}w_{parent:c}
\\
a_{child:i} &= \sum_{c=1}^{t+1}({n_{pub:i} \over {t+1}} + w_{parent:c})
\\
w_{child:c:i} &= {n_{pub:i} \over {t+1}} + w_{parent:c}
\end{aligned}
$$

The public key $A_{child:i} = n_{pub:i}G \circ A_{parent}$ can be computed by any party.

Knowing each child secret share $w_{child:c:i}$ we can proceed with the signature aggregate

$$s_{child:c:i} = r_{child:c:i'} + (u \times w_{child:c:i})$$

and therefore produce a threshold signature from a child neutered key pair

$$s_{child:i} = \sum_{h=1}^{t+1}s_{child:c:i}$$

Remark that we can produce $r_{child:c:i'}$ using simple key derivation on the index $i' \ne i$.

This all makes the combination of BIP32 and BIP340 very powerful for the implementation of key recovery processes with offline parties. As the only interactions needed are for the threshold DKG. Aggregate signatures can be gathered by any party.