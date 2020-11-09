# Cardano Architecture Overview

This section describes the high-level architecture of the Cardano blockchain. It provides details on the core components and their interactions, and briefly discusses Cardano's eras and implementations.

## Cardano blockchain - High Level Architecture

The following diagram outlines the interaction between the system components:
![Cardano Components](cardano_components_version_main@2x.png)

## System Components

The *current* implementation of Cardano is highly modular. It includes the following components (different deployment use cases will use different combinations of components):

* Node
* Command line interface (CLI)
* Daedalus wallet
* Cardano db-sync
* GraphQL API server (Apollo)
* REST API components
* SMASH server 



### Nodes and Remote Nodes

A blockchain system consists of a set of nodes distributed across a network that communicate with each other to achieve consensus about the system’s state.

Nodes are responsible for: 

* Executing the Ouroboros protocol
* Validating and relaying blocks
* Producing blocks (some nodes)
* Providing information about the state of the blockchain to other local clients

You can only trust nodes run by you or your organization. This is why Daedalus runs a node in the background.

### Node Process

The cardano-node is the top level for the node and consists of the other subsystems, of which the most significant are consensus, ledger and networking with ancillary configuration, CLI, logging, and monitoring.

### Node-to-Node IPC Protocol

The purpose of the node-to-node Inter-Process Communication (IPC) protocol is to allow for the exchange of blocks and transactions between nodes as part of the Ouroboros consensus algorithm.

The node-to-node protocol is a composite protocol, consisting of three ‘mini-protocols’:

* **chain-sync**: Used for following the chain and getting block headers.
* **block-fetch**: Used for getting block bodies.
* **tx-submission**: Used for forwarding transactions.

These mini-protocols are multiplexed on a single long-running Transmission Control Protocol (TCP) connection between nodes. They can be run in *both* directions on the same TCP connection to allow for peer-to-peer (P2P) settings.

The overall protocol -and each mini-protocol- is designed for a trustless setting where both sides need to guard against Denial-of-Service (DoS) attacks. For example, each mini-protocol uses consumer-driven control flow, so a node only requests more work when it is ready, rather than having work *pushed* upon it.

The protocol design is modular and evolvable: version negotiation is used to agree the set of mini-protocols to use, which allows additional or updated mini-protocols to be added over time without causing compatibility issues.

### Node-to-Client IPC

The purpose of the node-to-client IPC protocol is to allow local applications to interact with the blockchain via the node. This includes applications such as wallet backends or blockchain explorers. The node-to-client protocol enables these applications to access the raw chain data and to query the current ledger state. It also provides the ability to submit new transactions to the system.

The node-to-client protocol uses the same design as the node-to-node protocol, but with a different set of mini-protocols, and using local pipes rather than TCP connections. As such, it is a relatively low-level narrow interface that exposes only what the node can provide natively. For example, the node provides access to all the raw chain data but does not provide a way to query data on the chain. The job of providing data services and more convenient higher level APIs is delegated to dedicated clients, such as cardano-db-sync and the wallet backend.

The node-to-client protocol consists of three mini-protocols:

* **chain-sync**: Used for following the chain and getting blocks
* **local-tx-submission**: Used for submitting transactions
* local-state-query**: Used for querying the ledger state

The node-to-client version of chain sync uses *full* blocks, rather than just block headers. This is why no separate block-fetch protocol is needed. The local-tx-submission protocol is like the node-to-node tx-submission protocol but simpler, and it returns the details of transaction validation failures. The local state query protocol provides query access to the current ledger state, which contains a lot of interesting data that is *not* directly reflected on the chain itself.

### Command Line Interface (CLI)

The node’s CLI tool is the “swiss army knife” of the system. It can do almost everything, but it is quite low level and not very convenient because it’s text-based and lacks a graphical user interface (GUI).

The CLI tool can:

* Query the node for information
* Submit transactions
* Build and sign transactions
* Manage cryptographic keys

### Daedalus Wallet

Daedalus is a full node wallet that helps users to manage their ada, and can send and receive payments on the Cardano blockchain. Daedalus consists of a wallet frontend and a backend. The frontend is the graphical application that users see and interact with. The backend is a service process that monitors the state of the user’s wallet and does all the 'heavy lifting', such as coin selection, transaction construction, and submission. The backend interacts with a local node via the node-to-client IPC protocol, and interacts with the frontend via a HTTP API. The backend also provides a CLI that enables interaction with the wallet. The wallet backend can also be used on its own -without Daedalus- via its API. This is a convenient way for software developers to integrate Cardano with other applications and systems.

We advise that most advanced users intending to use Cardano start with Daedalus.

### cardano-db-sync

The cardano node stores only the blockchain itself and associated information needed to validate the blockchain. This design principle is about minimising code complexity, and reducing computational cost and resource use, to keep the node's local interfaces as minimal as possible and to use external clients to provide a variety of convenient interfaces and extra functionality. In particular, the node does not provide a convenient query interface for historical information on the blockchain. This data service is provided by a separate component using an Structured Query Language (SQL) database.

The cardano-db-sync component uses a local node (via its node-to-client IPC protocol) to follow the blockchain and continuously reflect the chain data into a PostgreSQL database. This database then provides all of the functionality to efficiently and conveniently query the chain data.

The database is the basis for other applications that need a flexible way to query the chain data. There are also GraphQL and REST APIs that provide views onto this database. This gives developers writing applications that integrate with Cardano the option of two flexible query languages: GraphQL or SQL, depending on the developer’s technology stack preferences. The overall deployment for an application using the SQL interface consists of a Cardano node, the cardano-db-sync process, PostgreSQL, and the user application.

The PostgreSQL database is designed to be used in a read-only fashion by applications, with only the cardano-db-sync component writing to the database to keep it in sync with the chain. The database schema is highly normalised. Several useful database queries are provided as Postgres views.

### GraphQL API Server (Apollo)

The GraphQL API provides a query interface to all the blockchain data via GraphQL, which is a convenient choice for client applications based on web technologies (applications written in JavaScript, or any other browser-based languages, for example) that use HTTP/REST APIs to talk to other services. It is an alternative to the database SQL interface. Application developers can choose between SQL and GraphQL for accessing the chain data.

The implementation of the API is based on the Apollo Server, an open-source, spec-compliant GraphQL server that's compatible with any GraphQL client (including the Apollo Client).

## REST API Components

There are two Cardano components that provide an HTTP REST API for interacting with a local node:

* A dedicated transaction submission component, which has a single endpoint for submitting transactions.
* A query component to access blockchain data. This is a legacy component provided to aid migration from the Byron era. Any applications currently using it should plan to migrate to the GraphQL API or the SQL interface.

The rationale for this is that the REST API is deprecated, so there is no reason for new applications to use it for querying chain data. The submission API is, for now, the only HTTP based API for tx submission, as GraphQL does not yet support tx submission, so any application authors that want to use web-tech APIs (rather than scripting APIs or low level or Haskell APIs) can use the REST API for tx submission.

### Stake Pool Metadata Aggregation Server (SMASH)

The purpose of the Stake Pool Metadata Aggregation Server (SMASH) is to aggregate off-chain metadata that stake pools provide when they register on the Cardano blockchain. This metadata includes the name of the stake pool, its ticker name, web homepage, and so on.

The rationale for a metadata aggregation server in the Cardano architecture is two-fold:

1. To keep the stake pool metadata stored off-chain; and
2. To retain the ability to moderate stake pool metadata, without any centralized censor.

The metadata is hosted off-chain and referred to from the on-chain pool registration. SMASH collects the off-chain data to make it more convenient, performant, and reliable for wallets and other applications to access it.

The SMASH server also addresses the desire to moderate the content of stake pool metadata without a centralized censoring entity. For example, most wallet users and stake pool operators would like to have the ability to treat stake pool ticker names as if they were unique trademarks. It would be too complex to have a fair, on-chain system to resolve ticker name disputes. Instead of enforcing uniqueness on chain, this can optionally be enforced by filtering as part of metadata aggregation. Multiple aggregation services can be run by different organizations following different policies on filtering stake pool metadata. This enables wallet users and other consumers of stake pool metadata to choose which policy to follow, if any.

SMASH can be configured with policies to filter out metadata based on block lists or reserved ticker names. Daedalus can be configured to use any SMASH server.

## About the Eras and Implementations of Cardano

Cardano is a third-generation distributed ledger. It is based on Ouroboros, a peer-reviewed proof-of-stake (PoS) blockchain consensus algorithm that first appeared in the top research conference in cryptology world-wide (the International Association for Cryptologic Research 37th International Cryptology CXonference - Crypto 2017).

The name Cardano is the general name given to the platform, which has gone through multiple *eras* and *implementations*. These concepts need further explanation.

### Eras

There are several eras within the evolution of Cardano. Each era (Byron, Shelley, Goguen, Basho, and Voltaire) refers to the rules of the ledger. For example, what transaction types and what data is stored in the ledger, or the validity and meaning of the transactions. 

The evolution of the Cardano mainnet began with the Byron ledger rules (Byron era). The  mainnet underwent a hard fork in late July 2020 to switch from the Byron rules to the Shelley ledger rules. Hence, this hard fork marked the beginning of the Shelley era.

### Implementations

Cardano’s first implementation was introduced at the start of the Cardano mainnet, back in September 2017. This implementation supported the Byron ledger rules exclusively. 

We then undertook a full reimplementation of Cardano, which enabled two fundamental changes: the support for multiple sets of ledger rules, and the management of the hard fork process of switching from one set of rules to the next. In other words, the new implementation can support *both* the Byron rules and the Shelley rules, which meant that, when it was deployed to the mainnet in early 2020, the implementation was also fully compatible with the Byron rules. This allowed for a smooth transition from the old to the new implementation. Once all Cardano users had upgraded their nodes to the new implementation, it became possible to invoke the hard fork and switch to the Shelley rules. 

A third Cardano implementation was used on the Shelley Incentivized Testnet (ITN). This system supported a significant subset of the Shelley rules, and we used it to test the economic and social dynamics of the Shelley delegation system.

This Cardano Architecture overview reflects the *current* Cardano implementation deployed on the mainnet, *not* the original or ITN implementations.







