# Multi Party Computing
Welcome to our GitHub project on Multiparty Computing (MPC)!

This repository aims to provide a comprehensive and reader-friendly description of secure computation among multiple parties. MPC enables different parties with private data to collaboratively perform computations without revealing their individual inputs. 

MPC allows developers to build secure, privacy-preserving applications in various domains, such as finance, healthcare, and machine learning. I invite you to explore my work, contribute to the project with questions and critics, and join me in pushing the boundaries of privacy preserving computation.

The following figure is at the heart of distributed key generation.

![Sharmir Secret Sharing](/mpc/img/polynomial_add_3x2.png)

Each line hides a secret. As you need two points to draw a line.

1. Alice draws a secret line (blue) and shares only the point $(3,5)$ with Bob.
2. Bob draws a secret line (red) and shares only the point $(2,5)$ with Alice.
3. The sum of both lines is a distributed secret line (green) whose coordinate are unknown to a single party.
4. The distributed secret line hides the distributed secret at $(0,3)$.

I hope simple enough, not to scare away the reader.

# Content

## Fundamentals
[Computational hardness assumptions](./mpc/cha.md)

[Distributed key generation](./mpc/dkg-tss.md)

## EC Cryptography
[Fields, groups, elliptic curves and associated hardness assumptions](./mpc/ecgroups.md)

[Threshold signature scheme (TSS) on ECDSA](./mpc/ecdsa-tss.md). See [this for ECDSA basics ](./mpc/ecdsa.md).

[Threshold signature scheme (TSS) on Schnorr Signature - BIP340](./mpc/schnorr-tss.md)

[Threshold signature scheme (TSS) on EDDSA](./mpc/eddsa-tss.md)

## Pairings
[BLS on pairings](./mpc/pairings.md)

## Privacy
[ECDSA - BIP32](./mpc/bip32.md)

[EDDSA - BIP32](./mpc/ed-bip32.md)

# Rationales

## Missing Simple Publications
Swimming through research papers and implementation of MPC solutions has been a tough journey for me. On one side, I find interesting research papers from hyper mathematicians, on the other side, I fall on undocumented code from super programmers focused on having an MPC scheme up and running.

## Simplicity
This work is not a formal publication, but a developer note. If there is anything inaccurate in what I describe herein, I beg to open a issue or send a pull request with your suggestion.

Being neither a mathematician nor an advanced programmer, whatever I can understand and describe shall be consumable by a common person. So do not shy away from sending a questions, might there be is anything you do not understand. I will give my best to explain it even simpler.

## The Vision
What I am currently missing is standardization initiatives, that strive to bring MPC concepts down to understandable idioms for common people, define common MPC techniques, workflows and serialization formats, provide basic and open source libraries in major programming languages.

My ultimate goal is to contribute to the evolvement of a new MPC application ecosystem with service providers that offer MPC services to wallets owners and wallet providers which integrate MPC services in their software to allow for richer self and shared custody experiences.