# BIP32 Key Derivation

[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) defines a procedure for deriving ec-keypairs from parent keys.

## Extended Key
Given an [elliptic-curve group](./ecgroups.md#elliptic-curves-ec) $(\mathbb{E}_{(\mathbb{Z_q})}, \circ, O, G, p)$, an extended key is a keypair with following properties:
- the extended private key is a pair $(a,c) \in \mathbb{Z^2_q}$ where
- a is the normal private key
- c is the chain code
- the extended public key is the pair $(A, c) \in \mathbb{E} \times \mathbb{Z_q}$, where
- $A = aG$ is the normal public key, and
- c the chain code.

Each extended key has :
- $2^{31}$ normal child keys, and
- $2^{31}$ hardened child keys.

Each of these child keys has an index:
- the normal child keys use indices $[0, \dots, 2^{31}-1]$,
- the hardened child keys use indices $[2^{31}, \dots, 2^{32}-1]$

To ease notation for hardened key indices, we use the representation $i_H = i+2^{31}$

Definitions of string as used here are found in [strings expressions](./conv-ser-enc.md).
## Hardened Derivation

A hardened derivation is only permitted for $i_{child} \ge 2^{31}$. It uses the parent private key bytes to produce the new random number. The key derivation function $HM_{<b:512>} \equiv \text{HMAC-SHA512}$ produces a $512$ bit string using the following function:

$$z_{h<b:512>} = HM_{<b:512>}(c_{parent<o(be):32>} || \text{0x00}_{<o:1>} || a_{parent<o(be):256:} || i_{child:<o(be):4>})$$

Where
- $c_{parent<o(be):32>}$ is the $32$ octets big endian representation of the parent chain code,
- $\text{0x00}_{<o:1>}$ is a one octet padding used to achieve the same input length like normal key derivation on public keys,
- $a_{parent<o(be):256:}$ is the $32$ octet big endian representation of the parent private key, and
- $i_{child:<o(be):4>}$ is the $4$ octets big endian representation of the child index. Recall $i_{child} \ge 2^{31}$ for hardened derivation.

The resulting bit string $z_{h<b:512>}$ is divided into two, $z_{hL}$ and $z_{hR}$ such that:
- $n_{priv:i<o(be):32>} = z_{hL} = z_{h<b:512:[:255]>}$ is the child's random number from the parent private key. $z_{h<[:255]>}$ selects the first $256$ bits of the string. Those are reorganized into a $32$-octet string in big endian byte order and read as a $32$ bytes integer.
- $a_{child:i} \equiv n_{priv:i} + a_{parent} \pmod q$, this random number is added to the parent private key to form the child's private key,
- $c_{child:i<o(be):32>} = z_{hR} = z_{h<b:512:[256:]>}$ is the child's chain code
- $A_{child:i} = a_{child:i}G$, is the child's public key.

### Output Validation
In case :
- $n_{priv:i} \ge q$ or
- $a_{child:i} \equiv n_{priv:i} + a_{parent} \pmod q \equiv 0$

the resulting key is invalid, and one should proceed with the next value for $i_{child}$. (Note: this has probability lower than $1 \text{ in } 2^{127}$)

## Neutered Derivation
A neutered derivation is only permitted for $i_{child} \lt 2^{31}$. It uses the parent public key bytes to produce the new random number. The key derivation function $HM_{<b:512>}$ produces a $512$ bits string using the following function:

$$z_{n<b:512>} = HM_{<b:512>}(c_{parent<o(be):32>} || A_{parent<(ec):33>} || i_{child:<o(be):4>})$$

Where
- $c_{parent<o(be):32>}$ is the $32$ octets big endian representation of the parent chain code,
- $A_{parent<(ec):33>}$ is the $33$ octets compressed representation of the parent public key point. This is also known as elliptic curve point encoding (ec), and
- $i_{child:<o(be):4>}$ is the $4$ octets big endian representation of the child index. Recall $i_{child} \lt 2^{31}$ for neutered derivation.

The resulting bit string $z_{n<b:512>}$ is splitted into two, $z_{nL}$ and $z_{nR}$ such that:
- $n_{pub:i<o(be):32>} = z_{nL} = z_{n<b:512:[:255]>}$ is the child's random number from the parent public key. $z_{n<[:255]>}$ selects the first $256$ bits of the string. Those are reorganized into a $32$-octet string in big endian byte order and read as a $32$ byte integer.
- $A_{child:i} = n_{pub:i}G \circ A_{parent}$, is the child's public key. This is a simple point addition as the same random number is added to the parent private key to form the child's private key.
- $c_{child:i<o(be):32>} = z_{nR} = z_{n<b:512:[256:]>}$ is the child's chain code

When this computation is performed by the holder of the parent private key, the child private key can also be computed with
- $a_{child} = n_{pub} + a_{parent}$.

This procedure works because:
$$
\begin{aligned}
A_{child} &= n_{pub}G \circ a_{parent}G
\\
A_{child} &= (n_{pub} + a_{parent})G
\\
A_{child} &= a_{child}G
\end{aligned}
$$

This works as the [curve arithmetic](./ecgroups.md#finite-groups-over-elliptic-curve) allows $(a + b)G \equiv aG \circ bG$

### Output Validation

In case :
- $n_{pub} \ge q$ or
- $A_{child} = O$ (Identity element)

the resulting key is invalid, and one should proceed with the next value for $i_{child}$. (Note: this has probability lower than $1 \text{ in } 2^{127}$)

# Security Properties
For security properties of BIP32 keys, visit the original proposal at [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

For a more formal analysis of the security of BIP32 keys, visit: [The Exact Security of BIP32 Wallets](https://eprint.iacr.org/2021/1287).