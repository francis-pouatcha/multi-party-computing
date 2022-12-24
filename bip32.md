# BIP32 Key Derivation

[BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki) defines a procedure for deriving ec-keypairs from parent keys.

## Extended Key
Given an elliptic curve group $(\mathbb{E}_{(\mathbb{Z_q})}, +, p, G)$, an extended key is a keypair with following properties:
- the private key is a pair $(a,c) \in \mathbb{Z^2_q}$ where
  - a is the normal private key 
  - c is the chain code
- the public key is tha pair $(A, c) \in \mathbb{E} \times \mathbb{Z_q}$, where
  - $A = aG$ is the normal public key, and
  - c the chain code.

Each extended key has :
- $2^{31}$ normal child keys, and 
- $2^{31}$ hardened child keys. 

Each of these child keys has an index: 
- the normal child keys use indices $[0, \dots, 2^{31}-1]$,
- the hardened child keys use indices $[2^{31}, \dots, 2^{32}-1]$

To ease notation for hardened key indices, a number $i_H = i+2^{31}$

## Conventions
Conventions:
- The key derivation function $H_{64}(\dots)$ is HMAC-SHA512. It takes concatenated derivation data and returns a $64$ octet string.
- $a || b$ is used to concatenante bytes sequences $a$ and $b$
- $N(\dots_{32})$ is used to parse a $32$ octet string into a number $n$

## Hardened Derivation 
A hardened derivation is only permited for $i_{child} \ge 2^{31}$. It uses the parent private key bytes to produce the new random number. The key derivation function $H$ produces a $64$ octet string using the following function:

$$S = H(c_{parent:32} || 0x00_{1} || a_{parent:32} || i_{child:4})$$

Where
- $c_{parent:32}$ is the $32$ bytes representation of the parent chain code $c_{parent}$,
- $0x00_{1}$ is a one byte padding used to achieve the same input length like normal key derication using public keys,
- $a_{parent:32}$ is the $32$ bytes representaiton of the parent private key $a_{parent}$, and
- $i_{child:4}$ is the $4$ bytes representation of the child index $i_{child}$. Recall $i_{child} \ge 2^{31}$ for hardened derivation.

The resulting octet string $S_{64}$ is divided into two: $S_{[0,..31]} \text{ or } S_{l:32}$ and $S_{[32,..63]} \text{ or } S_{r:32}$:
- $n_{priv} = N(S_{l:32})$ is the childs random number fromt the parent private key,
- $a_{child} \equiv n_{priv} + a_{parent} \pmod q$, is the child's private key,
- $c_{child:32} = S_{r:32}$, is the child's chain code,
- $A_{child} = a_{child}G$, is the child's public key.

### Invalide Output
In case :
- $n_{priv} \ge q$ or 
- $a_{child} \equiv n_{priv} + a_{parent} \pmod q \equiv 0$

the resulting key is invalid, and one should proceed with the next value for $i_{child}$. (Note: this has probability lower than $1 \text{ in } 2^{127}$)

## Neutered Derivation
A neutered derivation is only permitted for $i_{child} \lt 2^{31}$. It uses the parent public key bytes to produce the new random number. The key derivation function $H$ produces a $64$ octet string using the following function:

$$S = H(c_{parent:32} || A_{parent:33} || i_{child:4})$$

Where
- $c_{parent:32}$ is the $32$ bytes representation of the parent chain code $c_{parent}$,
- $A_{parent:33}$ is the $33$ bytes compressed representaiton of the parent public key point $A_{parent}$, and
- $i_{child:4}$ is the $4$ bytes representation of the child index $i_{child}$. Recall $i_{child} \lt 2^{31}$ for neutered derivation.

The resulting octet string $S_{64}$ is divided into two: $S_{[0,..31]} \text{ or } S_{l:32}$ and $S_{[32,..63]} \text{ or } S_{r:32}$:
- $n_{pub} = N(S_{l:32}) is the childs random number, from the parent public key,
- $A_{child} = n_{pub}G + A_{parent}$, is the child's public key,
- $c_{child:32} = S_{r:32}$, is the child's chain code.

When this computation is performed by the holder of the parent private key, the child private key can also be computed with
- $a_{child} = n_{pub} + a_{parent}$.

This procedure works because:
$$
\begin{aligned}
A_{child} &= n_{pub}G + a_{parent}G
\\
A_{child} &= (n_{pub} + a_{parent})G
\\
A_{child} &= a_{child}G
\end{aligned}
$$

### Invalide Output
In case :
- $n_{pub} \ge q$ or 
- $A_{child} = n_{priv} + A_{parent} = O$ (point at infinity)

the resulting key is invalid, and one should proceed with the next value for $i_{child}$. (Note: this has probability lower than $1 \text{ in } 2^{127}$)

# Security Properties
For security properties of BIP32 keys, visit the original proposal at [BIP32](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki).

For a more formal analysis of the security of BIP32 keys, visit: [The Exact Security of BIP32 Wallets](https://eprint.iacr.org/2021/1287).

# Ed25519-BIP32
This work originate from [Ed25519-BIP](https://input-output-hk.github.io/adrestia/static/Ed25519_BIP.pdf)
## Extended Master Key
Following steps produce a key used with ed255519:
- Generate a random secret seed $s_{<b:256>}$ for the user,
- compute $k_{<b:512>} = sha512(FLAG_{<b:?>} || s_{<b:256>})$,
  - If $k_{<b:[253]>}\ne0_{b:1}$, then discard $s_{<b:256>}$
  - set $k_{<b:[0:2]>}=0_{<b:[0:2]>}$, clearing first 3 lower bits of $k_L$
    - $c = log_2(8) = 3$, $8$ being the  cofactor of the curve, this construction clears the first 3 (low order) bits of the private key.
  - set $k_{<b:[255]>}=0_{<b:1>}$, clearing last bit of $k_L$
    - $n = 254$. Starting with $2^{254}$, we set lower bound of the secret scallar and clear the high order bit at $255$.
  - set $k_{<b:[254]>}=1_{<b:1>}$, setting the second highest bit of $k_L$
    - Secures all secret scalars are $255$ bits.
- compute the private key scalar $a_{o:32:le} = k_{<b[0:255]> = k_L}$,
- compute the random scalar $r_{o:32:le} = k_{<b[256:511]> = k_R}$,
- compute the root chaincode $c_{o:32:le} = H_{256}(0x01_{<o:1>}||s)$,
- the resulting extended private key is the tuple $(a, r, c) = (k_L, k_R, c)$,
  - Per convetion we also call this tuple $(a_{parent},r_{parrent},c_{parrent})$
- compute the public key $A = aG$,

## Private Child Key Derivation
Extended private key for child $i$ is the tuple $(a_{child:i},r_{child:i},c_{child:i}) = (k_{L:i}, k_{R:i}, c_i)$ is produced as follows:

Let $HM_{512}(key,value)$ be the function HMAC-SHA512 hash, allways using the chain code as the key,

For hardened derivation, mean $i \ge 2^{31}$, we have 
$$
\begin{aligned}
Z_{h<b:512>} &= HM_{512}(c_{parent<o:32>}, 0x00_{<o:1>} || a_{parent:<o:32>} || r_{parent<o:32>} || i_{child:<o:4>})
\\
c_{child:i<o:32>} &= HM_{512}(c_{parent<o:32>}, 0x01_{<o:1>} || a_{parent:<o:32>} || r_{parent<o:32>} || i_{child:<o:4>})_{<o:[32:63]>}
\end{aligned}
$$

For neutered key derivation, means $i \le 2^{31}$, we have
$$
\begin{aligned}
Z_{n<b:512>} &= HM_{512}(c_{parent:<o:32>},  0x02_{<o:1>} || A_{parent:<e:32>} || i_{child:<o:4>})
\\
c_{child:i<o:32>} &= HM_{512}(c_{parent:<o:32>},  0x03_{<o:1>} || A_{parent:<e:32>} || i_{child:<o:4>})_{<o:[32:63]>}

\end{aligned}
$$

Both $Z_h$ and $Z_n$ are represented here as $Z$ below.

Also recall that the hash result for chain codes are truncated to the right 32 bytes.

Constructing the child key:
- $z_{L:i<o:28:le>} = Z_{<o:[0:27]>}$, the first $28$ octets of $Z$,
- $z_{R:i<o:32:le>} = Z_{<o:[32:63]>}$, the last $28$ octets of $Z$,
- $a_{child:i} = k_{L:i} = 8 \times z_{L:i<o:28:le>} + a_{parent<o:32:le>}$
    - If a is divisible by $|G|$, discard i.
- $r_{child:i} = k_{R:i} = z_{R:i<o:32:le>} + r_{parent<o:32:le>} \pmod {2^{256}}$

The resulting extended key for child $i$ is the tuple: $(a_{child:i},r_{child:i},c_{child:i})$

## Public Child Key Derivation
Knowing the extended public key, we can proceed with a neutered derivation of the child public key. Recall this is only in neutered case with $i \le 2^{31}$.

$$
\begin{aligned}
Z_{n<b:512>} &= HM_{512}(c_{parent:<o:32>},  0x02_{<o:1>} || A_{parent:<e:32>} || i_{child:<o:4>})
\\
c_{child:i<o:32>} &= HM_{512}(c_{parent:<o:32>},  0x03_{<o:1>} || A_{parent:<e:32>} || i_{child:<o:4>})_{<o:[32:63]>}

\end{aligned}
$$

Constructing the child key:
- $z_{L:i<o:28:le>} = Z_{n<o:[0:27]>}$, the first $28$ octets of $Z_n$,
- $A_{child:i} = A_{parent<o:32:le>} + (8 \times z_{L:i<o:28:le>})G$
    - If A is the identity element $0,1$, discard the child $i$
