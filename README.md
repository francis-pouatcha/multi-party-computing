# Multi Party Computing
This project is used to document my understanding of multi party computing (MPC). As solving the
- self custody and
- wallet recovery problem

is essential for the proliferation of
- blockchain technology and
- self souverain solutions

into society.

## Missing Simple Publications
Swimming through research papers and implementation of MPC solutions has been a tough journey for me. On one side, I find interesting research papers from hyper mathematicians, on the other side, I fall on undocumented code from super programmers focused on having an MPC scheme up and running.

## About this Note
This work is not a formal publication, but a developer note. If there is anything inaccurate in what I describe herein, I beg to open an issue and let me adjust it.

## Simplicity Shall be Key
Being neither a mathematician nor an advanced programmer, whatever I can understand and describe shal be consumable by a common person. So do not stay away from sending a question, if there is anything you don't understand. I will give my best to explain it even simpler.

## The Vision
What I am currently missing is standardization initiatives, that strive to:
- bring MPC concepts down to understandable idioms for common people,
- define common MPC techniques, workflows and serialization formats,
- provide basic and open source libraries in major programming languages.

Ultimate goal will be to allow for the evolvement of a new MPC service ecosystem with:
- service providers that offer MPC services to wallets owners and
- wallet providers which integrate MPC services in their software to allow for
- a wallet-user-experience for common people.

# Content
- I start by dealing with [computational hardness assumptions](./cha.md), as they are the fundament of cryptography,
- I then try to understand [distributed key generation](./dkg-tss.md). In this context, I provide what I think is the simplest explanation of shamir secret sharing,
- I then move over to dealing with the algebra of [fields, groups, elliptic curves](./ecgroups.md) and associated hardness assumptions, as these are the fundamentals of modern cryptographic constructions,
- I finally get a look at signature standards such as:
  - [ECDSA](./ecdsa.md) used by bitcoin and ethereum, trying to understand how to implement a [threshold signature scheme (TSS) on ECDSA](./ecdsa-tss.md). From a privacy preserving perspective, I also drop a deeper look into hierarchical deterministic keys as defined by [BIP32](./bip32.md). Disappointed by the complexity of MPC on ECDSA, I try to look further and find
  - [Schnorr signature](./schnorr-tss.md), its usage by bitcoin and the possible application for aggregated and threshold signatures. Looks more promising than ECDSA, but not standardized enough, fortunately
  - [EdDSA](./eddsa-tss.md) comes to the rescue as the standardization path for schnorr signature schemes. Aggregated signatures and threshold signatures seem as simple here as they are on the original schnorr signature scheme.

I could stop here and start thinking on how to save the world, but to my disappointment, I notice that ethereum transactions only support ECDSA, despite the fact that the ethereum PoS validators use a simpler signature scheme named BLS to document consensus. This curiosity leads to my suicidal attempt to understand [pairings](./pairings.md).

# On my plate next
- describe BLS signatures- ...
- find out how to save the world (third attempt!)