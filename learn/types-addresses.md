
## Types of Addresses
The addresses are a blake2b-256 hash of the relevant verifying/public keys concatenated with some metadata that can be stored on the Cardano blockchain. Shelley introduces four different types of addresses:

Base addresses.

Pointer addresses.

Enterprise addresses.

Reward account addresses.

In addition to those new addresses, Shelley continues to support bootstrap addresses and script addresses as introduced in Byron. But only the new base and pointer addresses carry stake rights. Therefore, addresses consist of some serialized data specified in the ledger specification that are stored in the blockhain’s blocks, e.g. an UtxO address. The serialized data (address) contains the following two parts:

Metadata, that is used for interpreting.

Payload: the raw or encoded data.

Base Addresses
A base address directly specifies the staking key that should control the stake for the address. The staking rights associated with funds held in this address may be exercised by the owner of the staking key. Base addresses can be used in transactions without registering the staking key in advance.

The stake rights can only be exercised by registering the stake key and delegating to a stake pool. Once the stake key is registered, the stake rights can be exercised for base addresses used in transactions before or after the key registration.

Pointer Addresses
A pointer address indirectly specifies the staking key that should control the stake for the address. It references a stake key by a stake key pointer which is a location on the blockchain of the stake key registration certificate for that key. Pointer addresses can be used in transactions even if their target is not an active stake key registration. This covers the scenario where the key was unregistered after (or indeed before), the transaction, and also covers pointers to targets that are plainly invalid. The reason for allowing such invalid targets is so that nodes only need to track the currently active stake keys.

The pointer can be considerably shorter than the hash used in base addresses. There is a subtlety with pointer addresses:

It might happen that a stake key registration certificate that is referenced by a pointer address might be lost due to a rollback. This should not lead to a loss of funds. To prevent that, the system considers pointer addresses with an invalid pointer to be valid for the purpose of using funds stored therein as inputs for transactions (but ignores them for the purpose of PoS participation). Optionally, a wallet might refuse to create transactions to pointer addresses before the referenced certificate has become immutable, in order to prevent funds from being excluded from the PoS, in the case of rollbacks.

Enterprise Addresses
Enterprise addresses carry no stake rights and thus using them means that you are opting out of participation in the proof of stake protocol.

Exchanges or other organizations that control large amounts of ada – but hold it on behalf of other users – may wish to follow a policy of not exercising stake rights. By using enterprise addresses, exchanges can demonstrate that they follow this policy. Since enterprise addresses are not associated with any staking key, they are automatically excluded from the mechanisms that influence the slot leadership schedule. Note that using addresses with no stake rights effectively decreases the total amount of stake, which plays into the hands of the adversary.

Reward Account Addresses
A reward address is a cryptographic hash of the public staking key of the address. Reward account addresses are used to distribute rewards for participating in the PoS protocol (either directly or via delegation). They have the following properties:

account-style accounting is used, not UTxO-style.

funds cannot be received via transactions. Instead, their balance is only increased when rewards are distributed.

a one-to-one correspondence exists between registered staking keys and reward account addresses.

This key is used whenever funds are withdrawn from the address. Furthermore, stake associated with funds in the address contributes to the stake of this key. Just as in the case of enterprise addresses, the staking object for a reward address does not need to contain information.