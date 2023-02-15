# Type of Digital Signatures
The purpose of this document is just to list the type of digital signature we will be encountering throughout this work.

## Digital Signature
Mathematic procedure used to secure the integrity and (origin) authenticity of an electronic.

We can produce a digital signature using:
- A private key: in asymmetric cryptography. The public key will be used to verify the signature.
- A secret key: in symmetric cryptography. The same secret key will be used to verify the authenticity of the document.

## Multi Signature
A single document is secured with multiple signing keys (symmetric or asymmetric). This is, there a multiple signatures of the same document.

## Quorum Signature
I a multi signature scheme where a subset $t$ of a total of $n$ signature are required to consider the document authentic.

## Threshold Signature
A single digital signature, but produced from the joint effort of $t$  key share holders. The secret key (or private key) is assumed shared among $n$ parties (or share holders), so that only a subset $t$ of thereof can jointly produce a valid single digital signature.

## Merkle Signatures or Lamport Signatures
Digital signature scheme based on Hash Trees.
