# Wallet Recovery Service
The finality of transactions processed by the network is one of the key fundaments of crypto networks. 

Because of this finality, assets held on crypto networks are lost forever if:
- asset owner loses her key,
- asset owner loses the ability to exercise her will (clinical incapacity),
- asset trust decide to utilize the asset for other purpose or claims to have suffered a hacking attack,
- asset owner loses the piece of paper containing the seed phrase.

We also refer to wallet recovery and not to key recovery because, in many cases regaining access to the key material alone won't be sufficient to recover control on assets backed by those keys. This is because some transaction information such as alternative paths defined by BIP341 (taproot), or the combination of keys necessary to produce a threshold signature might not be exposed to the blockchain. For these reasons, recovering off-chain transaction information will end up being as important as recovering the key material.  

## Wallet Recovery
In this work, we assume wallets are maintained by zero custody wallet services. These are in charge of maintaining wallet transaction data on behalf of their owners.

Nevertheless, it is a good practice for a wallet owner to periodically export and produce alternative wallet backups, that can be use for recovery in case wallet owner:
- loses connectivity to the wallet service, or
- decides to migrate to another service. 

## Key vs. Asset Recovery
There are many reasons why key recovery is not a wishful action. As the very act of recovering presumes that a copy might be existing somewhere, like in the lost device.

For that reason, it might be meaningful instead to proceed with the movement of controlled assets to a new wallet. For this to happen, all assets controlled by the lost key, must have been under shared custody, meaning that some sort ot threshold mechanism or alternative spending paths give a possibility to move the asset with the combination of other parties involved in the alternative spending path.
