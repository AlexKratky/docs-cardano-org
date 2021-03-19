## About hard forks

The term hard fork is used to describe a radical change in the blockchain, for example, a change from one protocol to another. In most blockchains, a hard fork indicates block changes or a change to their interpretation. Traditionally, when conducting a hard fork, the current protocol would stop operating, new rules and changes would be implemented, and the chain would restart. It is important to note that a hard-forked chain *will be different* from the previous version and that the history of the pre-forked blockchain will no longer be available.

The Cardano blockchain has recently hard forked from a Byron federated model to a Shelley decentralized one. However, this hard fork was unique. Instead of implementing radical changes, we ensured a smooth transition to a new protocol while saving the history of the previous blocks. This means that the chain did not change radically, instead, it contains Byron blocks, and after a transition period, adds Shelley blocks. There was no fundamental restart point that erased the history of previous activities.

### What is a hard fork combinator?

A combinator is a technical term used to indicate the combination of certain processes or things. In the case of Cardano, a hard fork combinator combines protocols, thereby enabling the Byron-to-Shelley transition without system interruption or restart. It ensures that Byron and Shelley ledgers appear as one ledger. Shifting from BFT to Ouroboros Praos does not require all nodes to update simultaneously. Instead, nodes can update gradually, in fact, some can run Byron blocks, others - Shelley blocks.

The hard fork combinator is designed to enable the combination of several protocols, without having to make significant adjustments. The current Cardano chain combines Byron and Shelley blocks, and after future transitions, it will also combine Goguen, Basho, and Voltaire blocks - all as a single property.  This combinator facilitates the transition from Shelley to Goguen and beyond by simplifying the previous Byron-to-Shelley evolution.

### Moving from Byron Ouroboros Classic to Shelley Ouroboros Praos

Cardano Byron mainnet ran on the Ouroboros *Classic* consensus protocol. Cardano Shelley mainnet, which is the current development era, transitions to a decentralized network running on the new Ouroboros *Praos* consensus protocol, which allows for more extended capabilities while also supporting the staking process with monetary rewards for ada holders and stake pool owners.

To enable orderly transitions in Cardano without any diversions in the system, it was necessary to update the code to support the new protocol’s conditions. Doing so in a single update might have caused a range of complexities, so Cardano decided to take a two-stage approach, using the Ouroboros *Byzantine Fault Tolerance* (BFT) protocol as an intermediary.

A shift from Ouroboros Classic to BFT (that happened on February 20, 2020)  is the only traditional hard fork within the Cardano blockchain. This forking event restarted the Byron mainnet to run the BFT protocol and enable a smoother transition to Ouroboros Praos without any further chain interruptions. The BFT protocol was carefully designed so that blockchain history would remain unchanged, and the blockchain would appear as a single entity. 

### Token locking: Shelley protocol update

*Token locking* is a new feature that is being added to the Shelley protocol to enable various kinds of smart contract use cases, including creating and transacting with multi-asset tokens, as well as establishing support for the Voltaire voting mechanism. Token locking is the process of ‘reserving’ a certain amount of assets and committing not to dispose of them for a specified period of time. This feature is enabled in the *Allegra* (token locking) upgrade and will allow recording of that a specific token is being used for a certain purpose during the *Mary* (multi-asset support) upgrade. The *token* can represent an item that is accounted for by the blockchain ledger, including ada, but soon will include other custom token types.

**Token locking use cases**

Support for token locking is crucial to enable complex deal settlement and funds accounting. It can be used in the following scenarios:

+ **Contractual agreement** - when someone enters into a contractual agreement, to sell a property, for instance, it is important to promise that this property will not be sold to another person – only to the person who actually pays the money. In this case, the token can represent the property and the ‘promise’ – the actual token locking. If the property is sold to a different third party, then the contract becomes void. 
+ **Vote registry** - within the Voltaire voting mechanism, token locking will enable users to lock a certain amount of their tokens to represent their voting rights. Ada holders who participate in the voting process will be required to ‘lock’ their tokens. This will represent their voting rights, according to the stake that they hold, and eliminate the risks associated with scenarios such as double-counting votes, allocating more votes than possible, contradictory votes, or vote duplication. 
+ **Multi-asset tokens** - Cardano will soon provide support for multi-asset tokens, where the ledger will support the creation and use of multiple custom token types, besides ada. Token locking will allow ada tokens to be ‘locked’, for example, to create another custom asset of equivalent value. 

**How does it affect the community?**

The introduction of token locking will not negatively affect general ada holders, since wallets such as Daedalus and Yoroi will update automatically without requiring any action from ada holders.

All the nodes that are involved in the network maintenance will have to ‘agree’ on this transition to a new protocol, that is, they will need to reach a consensus. Stake pool operators and exchanges that are running nodes will simply have to download the new version of the node software and verify its operation on the Cardano testnet. 




