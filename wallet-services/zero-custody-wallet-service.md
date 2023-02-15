# Zero Custody Wallet Service (ZCWS)
The purpose of a zero custody wallet service is to provide the an asset owner with the functionality of managing assets held on a crypto network, without the need to access a private key. 

## Features of a Wallet Service
In summary, a zero custody wallet service will:
- allow the user to specify public keys or initial addresses he controls on that network. 
  - In order to improve the quality of service provided, wallet service can require a proof of possession of the corresponding private keys.
  - Hierarchical deterministic keys will improve the user experience, limiting proofs to root keys.
- allow the user to query the balance of his belongings,
- allow the user to register transactions, both finalized and expected,
- allow the user to register non public details associated with a transaction, 
- produce new transactions for signing upon user request,
- produce user transaction reports,
- archive user transaction for future access.

## Protecting Access to User Data
Even though the wallet service can access the user's private key, user transaction details disclose too much information, what can turn out to be harmful to the user.

In order to prevent illicit access to user transactions, a wallet service can persist the user wallet as individually encrypted data record.

## Receiving Transactions
A well designed wallet service can implement some sort of inbox, that allows user wallet to receive third party produced transaction sent to the user or even allow the user wallet to process notification associated with finalized transactions on networks.

Reasonable security level can be asserted if user wallet data are not accessible without user interaction. Meaning that the user authentication process shall be providing the key used to decrypt user wallet data.

# Off Chain Transaction Data
The growing need for more features and privacy pushes for less transparency in crypto networks. We witness features like zero knowledge EVMs, Taproot, or totally private blockchains which don't leave enough extractable information in transaction data. Working in those environments, information needed to properly dispose an asset might have to be stored off chain by its legitimate owner. For example: 

## Threshold Addresses
An aggregated or a threshold address just looks like a normal address, given the user no indication on how and who can help produce a matching signature. 

In a company business, a single threshold address can be used to collect revenue. In the case of a natural entity (person), preventing the corelation of money received or spent by the entity is essential for privacy. In the classical bitcoin environment, using BIP32 keys allows the user to produce a new addresses for each incoming transaction. Working with threshold addresses, producing BIP32 threshold child addresses might be tricky.

## BIP341 Taproot
BIP341 transactions embed spending rules into a merkel tree. The produce address contains the resulting merkel root, but give no information on the alternative spending paths. It is therefore in the interest of the owner of the output to store source information off chain, so this can be used to produce a legitimate signature when needed.

# On Chain Custody Rules
When a platform supports smart contracts, wallet custody and recovery rules can be programmed into smart contracts. Some proposals like the [Ethereum Account Abstraction](https://eips.ethereum.org/EIPS/eip-4337) to simplify integration of these features into the native consensus mechanism.