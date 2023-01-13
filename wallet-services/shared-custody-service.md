# Shared Custody Service
The shared custody paradigm wants us to distribute the responsibility of disposing an asset among multiple parties. The shared custody can be manifested in:
- having many parties jointly produce a single signature to prove possession of the asset being disposed,
- having the network implement multi signature rules that allow a set of parties to produce a quorum of signatures for the decision of disposing an asset,
- having the network implement constraints and constraint languages that enable the construction of alternative decision paths (known as spending paths) for a given asset (see BIP341 taproot, SegWith, solidity)

The process of involving multiple parties into the disposition of an asset opens room for the improvement of the overall usage experience for crypto asset owners.

# Implementation Strategies

## Co-Signing Parties
The simplest way of implementing share custody is having other parties co-sign transactions. 

This scheme can be implemented by a signature workflow service, that exposes a communication workflow used to coordinate distributed key generation and signature aggregation for the parties.

The workflow service can be redeemed 
- either by including a payment for the workflow service in each transaction,
- or by implementing some sort of subscription for the asset holders.

## Co-Signing Application
If an asset owner does not want to collaborate with another party, she can delegate co-signature responsibility to a co-signing application.

Co-signing application shall run on a separated devices in the control of the same asset owner.

If the co-signature application is running on the same device, user shall be ware losing that device exposes full access to user controlled assets.

## Co-Signing Server
The third possibility for an asset owner is to delegate co-signature responsibility to a co-signing server.

Co-signing server shall be operated by a trusted and qualified provider, an built such as to prevent insider and trojan access to asset owners partial key material.

# Multiple Co-Signature Paths
Well designed, co-signature can be set up such as to allow multiple co-signature devices. This can allow the asset owner to keep a co-signature device offline, and use it when he loses access to one of the two active devices.

