# What is DB Sync?

[DB Sync](https://docs.cardano.org/projects/cardano-db-sync/en/latest/) is one of the core architecture components of Cardano, as it provides a convenient way to find and query historical information from the Cardano blockchain. This service is enabled through the use of a Structured Query Language (SQL) relational database. Db-sync connects as a client to the local Cardano node and synchronizes with the on-chain activity. The PostgreSQL database then serves as a mapping of the on-chain information to the relational model.

DB Sync can be used by Cardano users and developers who wish to find out specific details in regards to block production and recent transactions. These details do not include cryptographic signatures, however, they do include block information that allows users to follow the chain and explore transactions within blocks.

## Architecture

Cardano-db-sync consists of a set of components:

-   `cardano-db` – defines common data types and functions used by any application that needs to interact with the database from Haskell. In particular, it defines the database schema.
-   `cardano-db-tool` – a tool used to manage cardano-db-sync databases (create, run, validate and analyze migrations).
-   `cardano-db-sync` – acts as a Cardano node, following the chain and inserting data from the chain into a PostgreSQL database.
-   `cardano-db-sync-extended` – a relatively simple extension to cardano-db-sync, which maintains an extra table containing epoch data.

The two versions `cardano-db-sync` and `cardano-db-sync-extended` are fully compatible and use identical database schema. The only difference is that the extended version maintains an `Epoch` table. The non-extended version will still create this table but will not maintain it.

The `db-sync node` is written in a highly modular fashion to allow it to be as flexible as possible. It connects to a locally running cardano-node (i.e. the one connected to other nodes in the Cardano network over the internet with TCP/IP) using a Unix domain socket, retrieves blocks, updates its internal ledger state and stores parts of each block in a local PostgreSQL database. 

[PostgreSQL](https://www.postgresql.org/) is a generic relational database used for the mapping of on-chain information to a relational model. It addresses specific user needs and requirements using the normalization technique to store relational data without duplication. 

GraphQL and REST API export an interface to the data stored in the database to be accessed from spreadsheets, for example. [Frontend explorer](https://explorer.cardano.org/en) is the most user-oriented tool; it fetches data from the main database and reflects it in a straightforward and convenient web interface, which grants useful navigation options. Another component, the [SMASH](https://docs.cardano.org/en/latest/getting-started/stake-pool-operators/SMASH-metadata-management.html) server, aggregates stake pool metadata and provides pool operators and delegators with a list of valid stake pools with verified metadata.

### GraphQL API Server (Apollo)

The GraphQL API provides a query interface to all the blockchain data via GraphQL, which is a convenient choice for client applications based on web technologies (applications written in JavaScript, or any other browser-based languages, for example) that use HTTP/REST APIs to talk to other services. It is an alternative to the database SQL interface. Application developers can choose between SQL and GraphQL for accessing the chain data.

The implementation of the API is based on the Apollo Server, an open-source, spec-compliant GraphQL server that's compatible with any GraphQL client (including the Apollo Client).

### REST API Components

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
