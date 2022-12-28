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
