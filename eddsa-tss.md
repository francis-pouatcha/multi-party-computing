# EdDSA Signature

EdDSA is a variant of the [Schnoor Signature](https://en.wikipedia.org/wiki/Schnorr_signature). EdDSA is standardized by the IETF in [RFC8032](https://www.rfc-editor.org/rfc/rfc8032). The most recomended instantiation is ed25519, which implements EdDSA on the twisted edwards curve edwards25519.

## Twisted Edwards Curves
A twisted edwards curve is a twist of an [edwards curve](https://en.wikipedia.org/wiki/Edwards_curve). Of interest here is a twisted Edwards curve $\mathbb{E_{(F_q)}}$ over a field $\mathbb{F_q}$ with characteristic $char(\mathbb{F_q}) \ge 3$ and the equation:
$$
\mathbb{E}_{TEd} : ax^2 + y^2 = 1 + dx^2y^2
$$

where $a, d \in \mathbb{F_q}$, with $a,d \ne 0$ and $a \ne d$. The special case $a=1$ is untwisted, because the curve reduces to an ordinary edwards curve.

## ED Arithmetic
Arithmetic on edwards curves is similar to [arithmetic on classical ec](./ecgroups.md#elliptic-curves-arithmetic-eca), twisted edwards curves can form a group under following conditions:
- $a$ is a square in $\mathbb{F_q}$ and $b$ is a non square in $\mathbb{F_q}$,
- an identity element at $O = (0, 1)$,
- a point addition operation where, for $P=(x_1, y_1), Q=(x_2, y_2) \in \mathbb{E}_{TEd}$, the sum $P + Q = (x_3, y_3) \in \mathbb{E}_{TEd}$,
- an additive inverse element of $P=(x_P, y_P)$ is $-P = (-x_P, y_P) \in \mathbb{E}_{TEd}$.

Edwards25519 is birationally equivalent to Montgomery curve curve25519, which due to its efficient arithmetic implementation, allows for constructions of very performant cryptographic applications. 

Edwards25519 is defined over the finite field $\mathbb{F_q}$ with:
- curve order $q = 2^{255} - 19$, is the number of points on the curve,
- curve equation: $−x^2 + y^2 = 1 − (121665/121666) * x^2 * y^2$.
- generator point $G = (x, 4/5) \text{ and } x>0$, x can be computed from the curve.
- generator order $l = 2^{252} + 27742317777372353535851937790883648493$

Curve25519 is defined over the finite field $\mathbb{F_q}$ with: 
- curve order $q = 2^{255} - 19$ and 
- curve equation: $y^2 = x^3 + 486662x^2 + x$.

## Notation and Conventions
### Scalar and Point Encoding
Notational conventions
- $n_{<o:32:le>}$ the $32$ bytes little endian representation of the scalar number $n$,
- $n_{<o:[i]>}$ the octet at position $i$,
- $n_{<o:[i:j]>}$ the octet sequence from the position $i$ to $j$,
- $s_{<b:512>}$ the bit string of length $512$, 
- $s_{<b:[i]>}$ the bit at the position $i$,
- $s_{<b:[i:j]>}$ the bit sequence from the position $i$ to the position $j$,
- $P_{<e:256>}$ is the encoded representation of the point $P$, where
  - $P = (x_P,y_P)$ is a point on the curve,
  - $P_{<e:256>}$ stands for a $256$ bit string, where
    - $P_{<b:[0:254]>}$ stores the bynary representation of the $y$-coordinate a the position $0, \dots, 254$,
    - $P_{<[b:255]>}$ stores the sign of the $x$-coordinate (least significan bit) at the position 255.
- $s_{<b:n>}||t_{<b:m>} = u_{<b:n+m>}$ concatenated bit strings $s_{b:n}$ and $t_{b:m}$.

### Prehash
[RFC-8032](https://www.rfc-editor.org/rfc/rfc8032#page-13) forsees the usage of a prehash function for follwoing variants:
- PureEdDSA uses the identity function as a prehash, resulting in $phm_{<b:?>} = PH(M_{<b:?>})=M_{<b:?>}$
- HashEdDSA used a collision resistant hash function like sha512, resulting in $phm_{<b:?>}=sha512(M_{<b:?>})$ 

In the rest of the document, we will use $phm_{<b:?>}$ to represent the bitstring of the prehashed message to be signed.

### Context Information
[RFC-8032](https://www.rfc-editor.org/rfc/rfc8032#page-13) allows the inclusion of context information in the computation of the hash. Both the prehash flag and the context info are specified with the function $dom2(phflag,context)$ like follows:
- For Ed25519: this information is an empty string, means $|dom2(phflag,context)|=0$ ensuring compatibility to previous versions.
- For Ed25519ctx:
  - $phflag=0$, means prehash function is the identity function (means PureEdDSA),
  - $context$ is a non empty octet string of length $1<= length <= 32$. The value is agreed upon by signer and verifiers.
  - let $flag_{<b:32>} = ASCII(\text{SigEd25519 no Ed25519 collisions})$
  - $dom2(x,y)_{<b:?>} = flag_{<b:32>} || x_{<o:1>} || l_{<o:1>} ||y_{<b:l>}$, where
    - phflag $x_{<o:1>} \equiv 0_{<o:1>}$ 
    - $l_{<o:1>}$ is the length of $y \text{ with } 1<= l <= 32$
    - $y_{<b:l>}$ is the bit string of the context information os size $l$
- For Ed25519ph:
  - phflag=1, with the prehash function $sha512$ 
  - let $flag_{<b:32>} = ASCII(\text{SigEd25519 no Ed25519 collisions})$
  - $dom2(x,y)_{<b:?>} = flag_{<b:32>} || x_{<o:1>} || l_{<o:1>} ||y_{<b:l>}$, where
    - phflag $x=1$ 
    - $l$ is the length of $y \text{ with } 0 <= l <= 255$
    - $y_{<b:l>}$ is the bit string of the context information

Finaly as notational convention
- let $FLAG_{<b:?>} \equiv dom2(phflag,context)$ in the rest of the document,
- let $phm_{<b:?>}$ be the prehashed message as defined above.

## Key Generation
Following steps produce a key used with ed255519:
- Generate a random secret seed $k_{<b:256>}$ for the user,
- compute $buf_{<b:512>} = sha512(FLAG_{<b:?>} || k_{<b:256>})$,
- compute the private key scalar $a_{<o:32:le>} = 2^n \sum_{i=c}^{n-1} 2^ih_{<b:[i]>}$, clearing high and low order bits, where
  - $c = log_2(8) = 3$, $8$ being the  cofactor of the curve, this construction clears the first 3 (low order) bits of the private key.
  - $n = 254$. Starting with $2^{254}$, we set lower bound of the secret scallar and clear the high order bit at $255$. Therefore all secret scalars are $255$ bits.
- compute the public key $A = aG$,
- return $(A_{e:256}, k_{b:256})$

## Signature
Following steps produce a ed255519 signature:
- let $m_{b:?}$ be the bit representation of the message,
- compute $h_{b:512} = sha512(FLAG_{b:?} || k_{b:256})$,
- compute the private key scalar $a_{o:32:le} = 2^n \sum_{i=c}^{n-1} 2^ih_{[b:i]}$,
- compute the random scalar $r_{o:64:le} = sha512(FLAG_{b:?} || h_{b:[256:511]}||phm_{b:?})$,
- compute the random point $R=rG$,
- compute the message scalar $u_{o:64:le} = sha512(FLAG_{b:?} || R_{e:256}||A_{e:256}||phm_{b:?})$,
  - For efficiency, compute $u \equiv u_{o:64:le} \pmod l$, where $l$ is the order of $G$,
- compute the signature scalar $s \equiv r + ua \pmod l$,
- return the encoded signature $\sigma_{b:512} = (R_{e:256}, s_{o:32:le})$

## Verification
### Available Information
The verifier has access to:
- the encoded signature $\sigma_{<b:512>} = (R_{<e:256>}, s_{<o:32:le>})$
- the public key point $A$

The verifier can:
- verify the public key point $A \in \mathbb{E}_{TEd(F_q)}$
- compute the random point $R \leftarrow R_{e:256}$
- verify $R \in \mathbb{E}_{TEd(F_q)}$
- verify the signature scalar $s \in \mathbb{F_q}$
### Further Checks
Further variant specific checks
- check that $0 \lt s \lt l$, where $l = |G|$, the order of the generator G
### Verification
A signature is verified by applying 
$$
\begin{aligned}
2^c(R+uA) &= 2^c(aG)
\\
2^c(R+uA) &= 2^c((r + u \times a)G)
\\
2^c(R+uA) &= 2^c(rG + 2^c(u \times a)G)
\\
2^c(R+uA) &= 2^c(rG + 2^cu(aG))
\\
2^c(R+uA) &= 2^c(R + uA)
\end{aligned}
$$

# Signature Aggregation
As does Schnorr signature, EdDSA allows $n$-of-$n$ multisignatures which, from a verifier's perspective, are not different from ordinary signatures.

Let a group of $n$ signers with the set $L = \{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\}$ be their public keys, means the common secret $a = \sum_{h=1}^na_h$.

The key generation is identical to single user signature.

## Signature
Following steps produce a ed255519 signature:
- let $m_{(b:?)}$ be the bit representation of the message,

Each signer $P_h$:
- computes $buf_{h<b:512>} = sha512(FLAG_{<b:?>} || k_{h<b:256>})$,
- computes the private key scalar $a_{h<o:32:le>} = 2^n \sum_{i=c}^{n-1} 2^ibuf_{h<[b:i]>}$,
- computes the random scalar $r_{h<o:64:le>} = sha512(FLAG_{<b:?>} || buf_{h<b:[256:511]>}||phm_{<b:?>})$,
- computes the random point $R_h=r_hG$,
- computes the public key point $A_h=a_hG$,
- publishes the random and public key points ($R_h, A_h$) to all other parties,

Upon receiving all random and public key points ($R_h, A_h$), each signer $P_h$:
- computes the commont random point $R = \sum_{h=1}^n R_h$, with simple point additions
  - $R = \sum_{h=1}^n R_h \equiv (\sum_{h=1}^nr_h)G = rG$, where
  - $r \equiv \sum_{h=1}^nr_h \pmod p$
- computes the commont public key point $A = \sum_{h=1}^n A_h$,
- computes the message scalar $u_{<o:64:le>} = sha512(FLAG_{<b:?>} || R_{<e:256>}||A_{<e:256>}||phm_{<b:?>})$,
  - $u_{<o:64:le>}$ shall be identical for all signer $P_h$
  - For efficiency, computes $u \equiv u_{<o:64:le>} \pmod l$, where $l$ is the order of $G$,
- computes his aggregate of the signature scalar $s_h \equiv r_h + u \times a_h \pmod l$,
- publishes the encoded aggregate of the signature $s_{h<o:32:le>}$

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

## Ed25519-BIP32 Keys
Using [Ed25519-BIP32](./bip32.md#ed25519-bip32) to derive all $A_h$ from the same child indexes allows parties $P_h$ to prevent one of them from being malicious.

Prior to engaging into distributed computation, each party $P_h$ prepares and publishes an extended public key $(A_{ext:h}, c_h)$. Where $c_h$ is the chain code. 

Upon receiving all extended public keys, each party computes the common publi key A.

Prior to producing the key, each party $P_h$ 
- uses a common but unique random messages in a random oracle model to compute n child key indexes. 
- these child key indexes are testet by each party against for the generation of valide private/public keys. Invalide indexes are discarded.
- for valid keys $i$, each party computes and publishes:
  - the child public key $A_{h:i}$
  - the child random point $R_{h:i}$
  - recall that this keys have to be neutered to allow for verification of the produced child keys.
- at the end of the key prepartion process, all parties hold a set of valid child key indexes and corresponding public parameters.

Upon receiving a message to sign and a selected child key index, each signer $P_h$ compute and publishes the aggregate signature.

# EdDSA Treshold Signature
## Treshold Secret Sharing
Treshold signature requires a $t$ of $n$ secret sharing scheme as defined in [DKG-TSS](./dkg-tss.md).

## Treshold EdDSA
Let a group of $n$ signers with the set $L = \{A_1=a_1G, A_2=a_2G, \dots, A_n=a_nG\}$ be their public keys. The common public key $A = \sum_{i=1}^n A_i$.

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
