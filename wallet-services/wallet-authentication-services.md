# Wallet Authentication Service

## Rationale
The adoption of open crypto networks is slowed down by the transparency character of information held on those networks. 

In order to prevent disclosure of transaction performed by parties, one time addresses can be used to perform operations. Hierarchical deterministic keys (a.k.a BIP32) present an excellent way of producing one time keys that in turn can be used to transact with parties in a anonymous and deterministic way.

Despite these possibilities of anonymizing transactions, abiding to reporting and disclosure rules, required by societies we live in, still call for ways to prove ownership of those crypto addresses to some institutions.

In some legal situations, proving that the owner controlled some assets prior to a certain date might be essential for correct accounting. This is for example the case if:
- an asset owner wants to write off losses in his tax declaration,
- gain realized on an asset want to be booked as income or capital gains, depending on whether this asset was held for less or more that a year (a prescribed by many tax laws around the globe).

In those cases, just showing present control of the private key proving possession to those assets does not states if the owner was in control of that private key prior to the given date, or if the holder just acquired the private key, for the purpose of achieving the intended accounting goal.

The taxability of asset transfers among citizen is also a rationale behind having to link citizen and crypto assets.

We can navigate through an infinite extent of societal, commercial and economical reasons, why it makes sense to implement some sort time stamped association between a person (natural or legal) and a crypto asset.

## Public Anonymous Wallet Registration Service
The wallet registration service is the place where a key holder can register herself as the legitimate owner of a cryptographic key used to take part to crypto transaction processes.

### Anonymous Registration Service
A registration service can be designed around having a record posted to a public ledger with the hash of a predefined data structure containing legal information on the holder of a master key.

The holder of the master key will then be able to derive child keys from the master key to perform transactions. A neutered key derivation with an undisclosed child key index is a perfect way of producing anonymous keys, whose ownership can be proven afterward.

### Preregistered Ownership
In some cases, it will be necessary to have the ownership relationship preregistered with an authority (e.g. the SEC). In those cases, following steps will allow for legal identification of the record holder:
- disclosure of the private record source of the published authentication hash,
- sort of off chain proof of identity to allow authority to verify that key controller matches the person described in the private record.

## Onchain Proof of Identity
Onchain identity attributes are necessary in some use cases:
- a market place wants to limit participation to authenticated participants, 
- a market place wants to limit participation to:
  - citizen of a given list of countries,
  - creditors with a certain accreditation.

Even with onchain identity, there might still be a need to keep transaction anonymous.

In both cases, the key that is taking part to a transaction need to have some on chain identity properties. Onchain identity properties can be implemented by:
- Maintaining a whitelist of authorized keys (or addresses) in an identity contract
- Maintaining a whitelist of authorizer keys in an identity contract and associating each transaction with a signed record of each participating address