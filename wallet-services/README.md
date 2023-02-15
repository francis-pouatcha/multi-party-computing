# Wallet
A wallet is classically defined as a flat folding case for holding money and plastic cards. In the context of MPC, we will mostly be dealing with __digital wallets__. These are softwares that can be used to hold and manage your digital assets. The digital wallet can be in:

- Self Custody: A standalone application installed on you phone, interacting with public ledgers,
- Exchange Custody: A server providing you with a web based user interface or a mobile front end app, but managing your assets on your behalf,
- Social Custody: Sort of having oder entities in your environment share access to your wallet key for recovery purpose,
- Any combination of those.

The purpose of this work is to provide the digital asset owner with the best user experience. 

For TLDR see [content](#content).

With all problems caused by the nature of centralized crypto exchanges in 2022, we won't limit the definition of user experience to the way you interact with a wallet. 

Loosing your digital belongings is also a bad user experience. Let it be as a consequence of:
- a centralized custodian going bankrupt while gambling with asset under custody,
- you loosing the phone storing your keys,
- a malicious software stealing the key from your phone,
- you forgetting the physical location of your seed phrase.

Unlike the sate of wallets out there:
- having to type down a seed phrase is also a bad user experience,
- having to configure and use a hardware wallet is a bad user experience,
- having to entrust you assets to a custodian without an FDIC insurance can lead to fatal user experiences.

If there is a reason why we don't keep substantial amount of cash or gold in the house, it is because we might loose it to bugler, flood, fire and others. Some asset don't even exist in a form that can be held in self custody. This might have been the reason while the banking and trust industry filled in gap of the asset custody service in the past centuries.

# Crypto Wallet
## Specifics of Crypto Assets
The concept of digital assets is evolving with their storage and management on cryptographic computer networks. A crypto asset network defines consensus mechanisms on how to represent, evaluate and transact given classes of assets.

A consensus mechanism generally encompasses:
- digital signature schemes allowing to prove possession of as asset
- a ledger scheme allowing the valuation of assets
- additional rules dictating how and under which conditions the ownership state of the asset changes. Among others:
  - simple hash value based or time based locks,
  - rule based constraints embedded into scripts or mathematical constructions like merkel trees,
  - more or less complex smart contract languages like solidity.

## Properties and Purpose of a Crypto Wallet
A crypto wallet shall allow the holder of a digital asset to:
- have a access to the information on the quantity and value of the held asset,
- transact the whole or part of the asset with other members of the network,
- to perform instant transactions.

A usable crypto wallet shall:
- be simple enough to be usable by a common smart phone user,
- require no extraordinary key backup or management action from the holder,

A safe crypto wallet shall:
- be built light enough not to allow uncontrolled infiltration of malware due to complex maintenance operation (supply chain attack),
- protect asset holder privacy, not disclosing details associated with user transactions to unauthorized parties,
- allow a user to share decision on when to transact funds held by the wallet.

# Content
In order to allow for the construction of simple end user applications, I present the concept of [zero custody wallet services](./zero-custody-wallet-service.md). Server side wallet can encompass all the complexity of crypto protocols, but do not access to end user signing keys.

I then visit the idea of [shared custody wallet services](./shared-custody-service.md) as an attempt to prevent having the full signature key of a user at a single location.

Recycling the waste out of the shared custody wallet processes, I look into the concept of [key and wallet recovery](./wallet-recovery-services.md).

Without claim of completeness, recovery procedures force the introduction of [wallet authentication services](./wallet-authentication-services.md).


