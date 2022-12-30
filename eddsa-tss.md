# EdDSA Signature

EdDSA is a variant of the [Schnorr Signature](https://en.wikipedia.org/wiki/Schnorr_signature). EdDSA is standardized by the IETF in [RFC8032](https://www.rfc-editor.org/rfc/rfc8032). The most recommended instantiation is ed25519, which implements EdDSA on the twisted edwards curve edwards25519.

## Twisted Edwards Curves
Let the expression $\mathbb{Z_{p^n}}$ represent the field of integers modulo $q$ written $(\mathbb{Z}, +, \times, q={c^n})$.

A twisted edwards curve is a twist of an [edwards curve](https://en.wikipedia.org/wiki/Edwards_curve). Curves of interest here are the family of twisted edwards curves $E: \mathbb{Z_q} \rightarrow \mathbb{Z_q}$, meaning over a field $\mathbb{Z_q}$ with characteristic $char(\mathbb{Z_q}) \equiv c \ge 3$ and the equation:

$$
E_{TEd}(y,x) : ax^2 + y^2 = 1 + dx^2y^2
$$

where
- where $a, d \in \mathbb{Z_q}$, with $a,d \ne 0$ and $a \ne d$. The special case $a=1$ is untwisted, because the curve reduces to an ordinary Edwards curve.
- $a$ is a square in $\mathbb{Z_q}$ and $b$ is a non square in $\mathbb{Z_q}$,

Let the expression $\mathbb{E_{(\mathbb{Z_q})}}$ represent the set of points constrained by the [elliptic-curve group](./ecgroups.md) $(\mathbb{E_{(\mathbb{Z_q})}}, \circ, O, G, p)$, over the curve equation $E_{TEd}$ in the field $\mathbb{Z_q}$ where:
- $\circ$ is a point addition operation such that, for $P=(x_1, y_1), Q=(x_2, y_2) \in \mathbb{E_{(\mathbb{Z_q})}}$, the sum $P \circ Q = (x_3, y_3) \in \mathbb{E}_{TEd}$,
- $O$ is the identity element at the point $O = (0, 1)$,
- there is an additive inverse of $P=(x_P, y_P)$ is $(-P) = (-x_P, y_P) \in \mathbb{E}_{TEd}$.
- $G$ is the generator of the group,
- p is the order of the generator $G$,

## Reviewing Notation
Reviewing [finite cyclic elliptic curve groups](./ecgroups.md#finite-cyclic-groups-over-elliptic-curves) is essential for the understanding of ECDSA maths below.

## Example of Curve Parameters
Edwards25519 is birationally equivalent to Montgomery curve curve25519, which due to its efficient arithmetic implementation, allows for constructions of very performant cryptographic applications.

Edwards25519 is defined over the finite field $\mathbb{Z_q}$ with:
- curve order $q = 2^{255} - 19$, is the number of points on the curve,
- curve equation: $−x^2 + y^2 = 1 − (121665/121666) * x^2 * y^2$.
- generator point $G = (x, 4/5) \text{ and } x>0$, x can be computed from the curve.
- generator order $p = 2^{252} + 27742317777372353535851937790883648493$

Curve25519 is defined over the finite field $\mathbb{Z_q}$ with:
- curve order $q = 2^{255} - 19$ and
- curve equation: $y^2 = x^3 + 486662x^2 + x$.

### Point Encoding
The definition of bit and byte strings as used here is found in [strings expressions](./conv-ser-enc.md).

EdDSA points are encoded in the format:

- Coordinates of point $P=(x_P, y_P), x_P, y_P \in \mathbb{Z_q}$ are serialized to bytes in little endian byte order.
- The scalar $x_P$ is therefore represented as $x_{P<(le:o):32>}$ where
  - $o$ stands for octet string
  - $le$ stands for little endian bytes order
  - $32$ is the size in octets of the string.
  - to present this in bytes, we can write $x_{P<(le:o)(b):256>}$ meaning the binary representation of the octet string.
- The sign of a the scalar $x_P$ could be represented as $x_{P<(sign:le\\:o\\:b):1>} = x_{P<(le:o\\:b):256:[0]>}$. This representation only tells the reader that the expression contains only the sign of $x_P$. But also indicates that the byte conversion from the scalar $x_P$ occured in the little endian order.
- The encoding of point $P$ ends up looking like

$$P=(x_P,y_P) \rightarrow P_{<(ed:b):256>} = x_{<P(sign:le\\:o\\:b):1>}||y_{P<(le:o\\:b):256:[1:]>}$$

This will add the sign bit of $x$ to the last 255 bytes of $y$.

Never write an expression like $P_{<(le:o):32>}$ because we do not want to read that byte string as a number. We always have to prefix with encoding details $P_{<(ed:le:o):32>}$, so the reader knows that it is the encoding of an EdDSA point. Safest is to write $P_{<(ed:b):256>}$

### Prehash
[RFC-8032](https://www.rfc-editor.org/rfc/rfc8032#page-13) foresees the usage of a pre hash function for following variants:
- PureEdDSA uses the identity function as a prehash, resulting in $phm_{<(b):?>} = PH(M_{<(b):?>})=M_{<(b):?>}$
- HashEdDSA used a collision resistant hash function like sha512, resulting in $phm_{<(b):?>}=sha512(M_{<(b):?>})$

In the rest of the document, we will use $phm_{<(b):?>}$ to represent the bitstring of the pre hashed message to be signed.

### Context Information
[RFC-8032](https://www.rfc-editor.org/rfc/rfc8032#page-13) allows the inclusion of context information in the computation of the hash. Both the prehash flag and the context info are specified with the function $dom2(phflag,context)$ like follows:
- For Ed25519: this information is an empty string, means $|dom2(phflag,context)|=0$ ensuring compatibility to previous versions.
- For Ed25519ctx:
  - $phflag=0$, means prehash function is the identity function (means PureEdDSA),
  - $context$ is a non-empty octet string of length $1<= length <= 32$. The value is agreed upon by signer and verifiers.
  - let $flag_{<(b):32>} = ASCII(\text{SigEd25519 no Ed25519 collisions})$
  - $dom2_{<(b):?>}(x,y) = flag_{<(b):32>} || x_{<(o):1>} || l_{<(o):1>} ||y_{<(b):l>}$, where
    - phflag $x_{<(o):1>} \equiv 0_{<(o):1>}$
    - $l_{<(o):1>}$ is the length of $y \text{ with } 1<= l <= 32$
    - $y_{<(b):l>}$ is the bit string of the context information os size $l$
- For Ed25519ph:
  - $phflag=1$, with the pre hash function $sha512$
  - let $flag_{<(b):256>} = ASCII(\text{SigEd25519 no Ed25519 collisions})$
  - $dom2_{<(b):?>}(x,y) = flag_{<(b):256>} || x_{<(o):1>} || l_{<(o):1>} ||y_{<(b):l>}$, where
    - phflag $x=1$
    - $l$ is the length of $y \text{ with } 0 <= l <= 255$
    - $y_{<(b):l>}$ is the bit string of the context information

Finally as notational convention
- let $FLAG_{<(b):?>} \equiv dom2(phflag,context)$ in the rest of the document,
- let $phm_{<(b):?>}$ be the pre hashed message as defined above.

# Key Generation
Following steps produce a key used with ed255519:
- Generate a random secret seed $k_{<(b):256>}$ for the user,
- compute $z_{<(b):512>} = sha512(FLAG_{<(b):?>} || k_{<(b):256>})$,
- compute the private key scalar 
  
  $$a_{<(le:o):32>} = 2^n \sum_{i=c}^{n-1} 2^iz_{<(b):512:[i]>}$$ 
  
  clearing high and low order bits, where
  - $c = log_2(8) = 3$, $8$ being the cofactor of the curve, this construction clears the first 3 (low order) bits of the private key.
  - $n = 254$. Starting with $2^{254}$, we set the lower bound of the secret scalar and clear the high order bit at $255$. Therefore all secret scalars are $255$ bits.
- compute the public key $A = aG$,
- return $A_{<(ed:b):256>}||k_{<(b):256>}$

# Signature
Following steps produce a ed255519 signature:
- let $M_{<(b):?>}$ be the bit representation of the message,
- compute $z_{<(b):512>} = sha512(FLAG_{<(b):?>} || k_{<(b):256>})$,
- compute the private key scalar $a_{<(le:o):32>} = 2^n \sum_{i=c}^{n-1} 2^iz_{<(b):512:[i]>}$,
- compute the deterministic secret scalar

$$r_{<(le:o):64}> = sha512(FLAG_{<(b):?>} || z_{<(b):512:[256:511]>}||phm_{<(b):?>})$$

- compute the secret point $R=rG$,
- compute the message scalar

$$u_{<(le:o):64>} = sha512(FLAG_{<(b):?>} || R_{<(ed:b):256>}||A_{<(ed:b):256>>}||phm_{<(b):?>})$$

- For efficiency, compute $u \equiv u_{<(le:o):64>} \pmod p$, where $p$ is the order of $G$,
- compute the signature scalar $s \equiv r + (u \times a) \pmod p$,
- return the encoded signature $\sigma_{<(b):512>} = R_{<(ed:b):256>}||s_{<(le:o\\:b):256>}$

# Verification
## Available Information
The verifier has access to:
- the encoded signature $\sigma_{<(b):512>} = R_{<(ed:b):256>}||s_{<(le:o\\:b):256>}$
- the public key point $A$

The verifier can:
- verify the public key point $A \in \mathbb{E}_{(Z_q)}$
- compute the random point $R \leftarrow R_{<(ed:b):256>}$
- verify $R \in \mathbb{E}_{(Z_q)}$
- verify the signature scalar $s \in \mathbb{Z_q}$
## Further Checks
Further variant specific checks
- check that $0 \lt s \lt p$, where $p = |G|$, the order of the generator G
## Verification
A signature is verified by applying

$$
\begin{aligned}
2^c(R \circ uA) &= 2^c(aG)
\\
2^c(R \circ uA) &= 2^c((r + (u \times a))G)
\\
2^c(R \circ uA) &= 2^c(rG \circ 2^c(u \times a)G)
\\
2^c(R \circ uA) &= 2^c(rG \circ 2^cu(aG))
\\
2^c(R \circ uA) &= 2^c(R \circ uA)
\end{aligned}
$$

# Signature Aggregation
EdDSA signature aggregation follows the same procedure as [Schnorr signature aggregation](./schnorr-tss.md#schnorr-signature-aggregation).

# EdDSA n-of-n Multi Signature

EdDSA multi signature also follows the same procedure as [Schnorr multi signature](./schnorr-tss.md#schnorr-n-of-n-multi-signature).

But as EdDSA is constructed over a nonlinear key space, the HD key derivation process is different.

## BIP32 HD Keys for n-of-n
Using [Ed25519-BIP32](./ed-bip32.md) to produce deterministic $A_h$ and $R_h$ from the same pretested child key indexes will help parties prevent one of them to be malicious.

Recall children key indexes might derive invalide keys, therefore all parties must agree upon idex ranges that are tested by all parties.

Recall that for EdDSA we use the right side of the secret seed to produce a deterministic secret point $R$. This removes the need of having to produce a deterministic secret scalar $r$ from a child key index $i'$ like we did BIP340-BIP32.

# EdDSA Threshold Signature
Threshold signature requires a $t$ of $n$ secret sharing scheme as defined in [DKG-TSS](./dkg-tss.md).

## Threshold Signature
Assume a group of $n$ signers $P_h, h \in H = \\{1, \dots, n\\}$, with the set $L = \\{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\\}$ of their respective public keys. The common public key

$$A = \sum_{h=1}^n A_h$$

## Interpolation of Individual Shares of $w_c$
As soon as we know the set of participants $C=\\{1, \dots, t+1\\} \subset H$, involved in the distributed computation, each $P_c$ interpolate its share $a_c \equiv f(x_c)$ into $t+1$ additive parts $w_c$ of the distributed secret $a$ such that

$$a = \sum_{c=1}^{t+1}w_c$$

This is explained in detail in [retrieving the secret](./dkg-tss.md#retrieving-the-secret).

Remark that we are using the symbol $a$ to represent the secret instead of $s$, as the last one is used in the page to reference the signature.

## Threshold Signing of a Given Message $M$
The TSS challenge consists in coordinating a distributed computation among members of subset $C = \\{1, \dots, t+1\\} \subset H$, to jointly produce the signature of the message $M$.

### Aggregation of Deterministic Random $R$
Per schnorr signature, the random point $R$ is part of the message hash. For a threshold signature, signing parties $P_c$ can agree upon using the sum of their $r_c$, ignoring the $r_{h-c}$ of parties not involved in the signature process. The only drawback of this process is that each set of signers $C' \ne C$ will produce a different signature.

Recall $r_c$ is computed using:

$$r_{c<(le:o):64>} = sha512(FLAG_{<(b):?>} || z_{L<(b):256:[256:511]>}||phm_{<(b):?>})$$

where $z$ is the output of the random oracle from which the secret scalar $a$ was produced. Recall from the secret seed $k_{<(b):256>}$, we produce $z$ with:

$$z_{<(b):512>} = sha512(FLAG_{<(b):?>} || k_{<(b):256>})$$

It is good to remember that the secret key $a$ always comes from the right therefore $k_R$ and the deterministic random $r$ always comes from the left therefore $k_L$.

Recall $phm$ is the pre hashed message, [see Prehash above](#prehash).

Each $P_c$ must compute and publish $R_c = r_cG$

Upon receiving all $R_c$, each signer $P_c$ must compute the random number

$$R = \sum_{c=1}^{t+1} R_c$$

### Computing the Message Scalar $u$

With $R$ available, each signer $P_c$ must compute the message scalar

$$u_{<(le:o):64>} = sha512(FLAG_{<(b):?>} || R_{<(ed:b):256>}||A_{<(ed:b):256>>}||phm_{<(b):?>})$$

### Partial Signature $s_c$

With $r_c, u$ available, each signer $P_c$ must compute and publish the partial signature scalar

$$s_c = r_c + (u \times w_c)$$

### Common Signature $s$
Upon receiving all $s_c$, each party can compute the common signature

$$s = \sum_{h=1}^{t+1}s_c$$

This signature will verify like any other EdDSA signature.

## BIP32 for Preventing Rogue-key attack
Using [ED-BIP32](./ed-bip32.md) key derivation to enforce generation of $A_h$ and $R_h$ will help mitigate rogue key attack like described in [BIP32 HD Keys for n-of-n](./schnorr-tss.md#bip32-hd-keys-for-n-of-n)

## Reducing communication while Using BIP32
Using BIP32, we create the opportunity of generating public keys $A_h$ without having to know the secret scala $a_h$. This works because we use the linear property of elliptic curve groups.

Having proceeded with an $t \text{ of } n$ secret sharing to produce a threshold key generation, we can also generate additive shares that work with interpolated secret shares $w_c$.

Knowing

We can generate a [neutered children random numbers](./ed-bip32.md#neutered-child-key-derivation) $z_{nL} and $z_{nR}$ for the child key index $i$. Following value will be computed
Constructing the child key:
- $z_{nL<(o:le):28>} = z_{n<(o):64:[:27]>}$, we take the first $28$ octets of $z_n$,
- $z_{nR<(o:le):32>} = z_{n<(o):64:[32:]>}$, we take the last $32$ octets of $z_n$,

Knowing that the derivation algorithm for a single key signature produces this,
- $a_{child:i} = (8 \times z_{nL}) + a_{parent}$
- $r_{child:i} = z_{nR} + r_{parent} \pmod {2^{256}}$

### Computing the Common Child Public Key $A_{child:i}$
The public key $A_{child:i} = (8 \times z_{nL})G \circ A_{parent}$ can be computed by any party.

### Computing the Child Secret Share $w_{child:i:c}$
Out of the DKG procedure, we have

$$
\begin{aligned}
a_{parent} &= \sum_{c=1}^{t+1}w_{parent:c}
\end{aligned}
$$

Instead of adding entire $8 \times z_{nL}$-number to each secret share, we divide it by the number of signers $|C|=t+1$ and add a chunk to each record, resulting in:

$$
\begin{aligned}
w_{child:i:c} &= {(8 \times z_{nL}) \over {t+1}} + w_{parent:c}
\end{aligned}
$$


### Computing the Deterministic Random $R$
The public image of the random key $R_{child:i} = z_{nR}G \circ A_{parent}$ can be computed by any party.

### Computing the Child Random Scalar $r_{child:i:c}$
The only constraint we have here is that single $r_{child:i:c}$ must be computed such that

$$
\begin{aligned}
r_{child:i} &= \sum_{c=1}^{t+1}r_{child:i:c}
\end{aligned}
$$

In order to achieve this, we divide the derived $z_{nR}$ by the number of signers $|C|=t+1$ and add a chunk to each signers parent random scallar, resulting in:

$$
\begin{aligned}
r_{child:i:c} &= {z_{nR} \over {t+1}} + r_{parent:c}
\end{aligned}
$$

### Computing the Message Scalar $u$
Recall that using HD key derivation and knowing:
- the preimage of the message to be signed $phm_{<(b):?>} \leftarrow M$,
- the child index to use $i$,

Each signing party $P_c$ can compute $A_{child:i} and R_{child:i}$ without having to communicate with other parties (Powerfull).

With $A_{child:i} and R_{child:i}$ available, each signer $P_c$ must compute the message scalar

$$u_{child:i<(le:o):64>} = sha512(FLAG_{<(b):?>} || R_{child:i<(ed:b):256>}||A_{child:i<(ed:b):256>>}||phm_{<(b):?>})$$

### Partial Signature $s_{child:i:c}$

With $r_{child:i:c}, u_{child:i}$ available, each signer $P_c$ must compute and publish the partial signature scalar

$$s_{child:i:c} = r_{child:i:c} + (u_{child:i} \times w_{child:i:c})$$

### Common Signature $s_{child:i}$
Upon receiving all $s_{child:i:c}$, each party can compute the common signature

$$s_{child:i} = \sum_{h=1}^{t+1}s_{child:i:c}$$

This looks like a powerful tool for key recovery procedures including the use of offline parties, as interaction is barely needed for the signature of messages.