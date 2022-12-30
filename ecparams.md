# Elliptic-Curve Domain Parameters
EcDSA, EdDSA (all valiants of [DSA](https://en.wikipedia.org/wiki/Digital_Signature_Algorithm)), some other Schnorr signatures (like BIP340), are all built on top of ECDLP.

They are all based on the family of [finite cyclic elliptic-curve groups](./cha.md#finite-groups-over-elliptic-curve) $(\mathbb{E_{(\mathbb{Z_q})}}, \circ, O, G, p)$, where:
- $\mathbb{E_{(\mathbb{Z_q})}}$ represent a set of points $P=(x_P, y_P)$ on the elliptic curve $E(x,y)$,
- $\circ$ is the points addition operation, including the additive inverse and the identity element $O$ (point at infinity),
- $G$ is the generator point, meaning that all points of the group can be computed by multiplying this point by a scalar number $n \in \mathbb{Z_q}$,
- $\mathbb{Z_q}$ is the field of order $q \in \mathbb{Z}$, such that $\forall_{P=(x,y)}, \forall_{Q=nP}, x, y, n \in \mathbb{Z_q}$. This means all point coordinates and all scalar numbers used to produced group elements are element of $\mathbb{Z_q}$,
- $p$ is the order of the generator $G$, or the number of points so that $\forall_{n \in \mathbb{Z_q}}, nG \in (\mathbb{E_{(\mathbb{Z_q})}}$.

For the construction of cryptographic primitives, special secure curves have to be defined. Elliptic-curve construction distinguishes between prime case and binary case curves.

## Binary Case Elliptic-Curves
In the binary case, curves parameters for the family of elliptic-curve groups $(\mathbb{E}_{(\mathbb{F_{2^m}})}, \circ, O, G, n)$ can be defined using the notation $(m,f,a,b,G,n,h)$ where:
- $m$ is the extension of the field $\mathbb{F_{2^m}}$. E.g $\mathbb{F_{2^8}}$ or $\mathbb{F_{256}}$ is used to implement AES,
- $f$ is the auxiliary curve,
- $\mathbb{Z_{2^m}}$ is the field of order $2^m$, such that $\forall_{T=(x_T,y_T)}, \forall_{Q=nT=(x_Q, y_Q)}, x_T, y_T, n, x_Q, y_Q \in \mathbb{Z_p}$,
- $a, b$ are the parameters of the curve equation $y^2= x^3 + ax + b$ (equation written in Weierstrass form),
- $G$ is the generator point,
- $n$ is the order of the point $G$, meaning that the smallest number $k \in \mathbb{Z_{2^m}}$ such that $kG = O$.

## Prime Case Elliptic-Curves
In the prime case, the elliptic-curve group $(\mathbb{E_{(\mathbb{Z_p})}}, \circ, O, G, n)$ can be defined using the notation $(p,a,b,G,n,h)$ where:
- $p$ is a large prime and the order of the field $\mathbb{Z_p}$ (number of elements in $\mathbb{Z_p} \text{ or } |\mathbb{Z_p}|$),
- $\mathbb{Z_p}$ is the field of order $p \in \mathbb{Z}$, such that $\forall_{T=(x_T,y_T)}, \forall_{Q=nP=(x_Q, y_Q)}, x_T, y_T, n, x_Q, y_Q \in \mathbb{Z_p}$,
- $a, b$ are the parameters of the curve equation $y^2= x^3 + ax + b$ (equation written in Weierstrass form),
- $G$ is the generator point,
- $n$ is the order of the point $G$, meaning that the smallest number $k \in \mathbb{Z_p}$ such that $kG = O$,
- $h$ the cofactor, given $h={1 \over n}|\mathbb{E_{(\mathbb{Z_p})}}|$. $h$ is used to limit subgroup of points on which to operate, for some security reasons e.g: [Pohlig–Hellman algorithm](https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm) on curve25519.

## Sample Prime Case Curve secp256k1
An example is the curve used by bitcoin is secp256k1 and has following parameters:
- $p = \text{FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE FFFFFC2F}$
- This is $2^{256} - 2^{32} - 2^9 - 2^8 - 2^7 - 2^6 - 2^4 -1$

The curve equation $E(x,y) \implies y^2= x^3 + ax + b$ over $\mathbb{Z_{2^8}}$ is defined by $y^2=x^3 + 7$, meaning:
- $\text{a = 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000}$
- $\text{b = 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000007}$

The generator point $G$ in uncompressed form is:
- $\text{G = 04 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798 483ADA77 26A3C465 5DA4FBFC 0E1108A8 FD17B448 A6855419 9C47D08F FB10D4B8
}$

The generator point $G$ in compressed form is:
- $\text{G = 02 79BE667E F9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798}$

Recall that the compressed representation of a weierstrass curve point only provides the $x$-coordinate and the indication of the sign of the $y$-coordinate $\mathtt{0x02} \text{ or } \mathtt{0x03}$ (least significant bit). As the $y$-coordinate can be computed using the curve equation $E(x,y)$.

The order $n$ of the generator $G$ is the number of points in the group $(\mathbb{E}_{(\mathbb{F_p})}, \circ, n, G)$ that can be generated from $G$, and is:
- $\text{n = FFFFFFFF FFFFFFFF FFFFFFFF FFFFFFFE BAAEDCE6 AF48A03B BFD25E8C D0364141}$

It is the smallest number $n \in \mathbb{F_{2^8}}$ such that $nG=O$.

The cofactor $h$ defines the subgroup $\mathbb{S_{r}}$ of prime order $r= { n \over h}$. Meaning for each element $kG$ of the subgroup, $k$ is a multiple of $h$. In case of secp256k1:
- $h=01$. Means all points of the group can be used.

secp256k1 is the foundation of the bitcoin EcDSA and BIP340 (Schnorr Signature).

## Sample Prime Case Curve with non Trivial Cofactor curve25519
An elliptic-curve with a non trivial cofactor is the Montgomery [curve25519](https://en.wikipedia.org/wiki/Curve25519), with following properties:
- curve equation $E(x,y) \implies y^2 = x^3 + 486662x^2 + x$. Recall that montgomery curves have the general equation $M_{A,B}: By^2 = x^3 + Ax^2 + x$ over $\mathbb{Z_p}$, where $A,B \in \mathbb{Z_p} \text{ and } B(A^2 -4) \ne 0$. We use capital letters here to represent coefficients, with the purpose of matching the original description of the curve.
- curve25519 uses the generator point at $x=9$ and a cofactor $h=8$ to generate a cyclic group with prime order $p = 2^{252} + 27742317777372353535851937790883648493$.

Using a subgroup in this case prevents the [Pohlig–Hellman algorithm](https://en.wikipedia.org/wiki/Pohlig%E2%80%93Hellman_algorithm) attack.