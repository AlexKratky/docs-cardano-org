# Working with DB Sync

The purpose of this document is to explain how Cardano users can retrieve blockchain data from the database. Here, we look into the ways of working with the PostgreSQL database and explore its various queries.

## How to retrieve data from the DB Sync PostgreSQL database

### Installation Prerequisites

* PostgreSQL library. This can be installed on Linux using e.g. `apt install libpq-dev`.
* PostgreSQL server. This can be installed on Linux using e.g. `apt install postgresql`.
* Сardano-node package. This provides the core Cardano node functionality and runs as a passive node.
* Сardano-db-sync package. This acts as a library for database support and connects to the Cardano node.

Alternatively, you can run these components using Docker: the `cardano-db-sync` repository contains a [docker-compose.yml](https://github.com/input-output-hk/cardano-db-sync/blob/master/docker-compose.yml) file, which lets you start the node, Postgres, and db-sync with a single command:

 `docker-compose up`

### How to set up and access the database

1.  You will first need to install [PostgreSQL](https://www.postgresql.org/download/).
2.  Create the database and [manage schemas](https://github.com/input-output-hk/cardano-db-sync/blob/master/doc/schema-management.md).
3.  Build and run the [cardano-node](https://github.com/input-output-hk/cardano-node) and [cardano-db-sync](https://github.com/input-output-hk/cardano-db-sync/blob/master/doc/building-running.md). 

**Useful PostgreSQL tutorials:**

* [SELECT](https://www.postgresql.org/docs/11/tutorial-select.html)
* [JOINs](https://www.postgresql.org/docs/11/tutorial-join.html)

### How to get data from the tables

On-chain data is stored in tables the mapping of which is structured (normalized) in a way that avoids data duplication. However, this also means that to extract certain data (like transaction information, for example), you will need to [join multiple tables in a query](https://www.postgresql.org/docs/11/tutorial-join.html).

To show all tables, run this command:

`\dt`

To show details about a specific table:

`\d+ TABLE_NAME`

Use the queries from the section below to get the blockchain data you are looking for. 

**Using DB management tools**

Another convenient way to explore database tables is to use a DB management tool like [https://www.pgadmin.org/](https://www.pgadmin.org/).

This tool allows browsing the tables and their definitions (like foreign key dependencies) or developing and executing complex queries.
  
Follow the [getting started instructions](https://www.pgadmin.org/docs/pgadmin4/latest/index.html) to set everything up. Now you can use the queries listed below to get the blockchain data you are looking for.

### Useful queries 

These SQL queries are run against the `db-sync` database and use the `psql` executable distributed with PostgreSQL. 

To connect to the database, run the following command in the `cardano-db-sync` git checkout:

```
PGPASSFILE=config/pgpass-mainnet psql cexplorer
```

> Note: Some of the queries have Haskell/Esqueleto equivalents in the file [Query.hs][Query.hs], such names are included in parentheses.

**Chain metadata (`queryMeta`)**

```
cexplorer=# select * from meta ;
 id |     start_time      | network_name
----+---------------------+--------------
  1 | 2017-09-23 21:44:51 | mainnet
(1 row)
```

**Current total on-chain supply of ada (`queryTotalSupply`)**

Note: 1 ADA == 1,000,000 Lovelace

This query shows the UTxO set for unspent transaction outputs. It does not include staking rewards that have not yet been withdrawn. Before being withdrawn, rewards exist in the ledger state, not on-chain.

```
cexplorer=# select sum (value) / 1000000 as current_supply from tx_out as tx_outer where
              not exists
                ( select tx_out.id from tx_out inner join tx_in
                    on tx_out.tx_id = tx_in.tx_out_id and tx_out.index = tx_in.tx_out_index
                    where tx_outer.id = tx_out.id
                  ) ;
    current_supply
----------------------
 31112120630.27526800
```

**Slot number of the most recent block (`queryLatestSlotNo`)**

```
cexplorer=# select slot_no from block where block_no is not null
              order by block_no desc limit 1 ;
 slot_no
---------
 4011090
(1 row)
```

**Size of the cexplorer database**

```
cexplorer=# select pg_size_pretty (pg_database_size ('cexplorer'));
 pg_size_pretty
----------------
 4067 MB
(1 row)
```

**Current valid pools:**

The database is generally operated in an append-only manner. Stake pools can retire and when pool certificates get updated, the data changes respectively. To get the latest pool registration for every pool that is valid run the following:

```
cexplorer=# select * from pool_update
              where registered_tx_id in (select max(registered_tx_id) from pool_update group by hash_id)
              and not exists
                ( select * from pool_retire where pool_retire.hash_id = pool_update.hash_id
                            and pool_retire.retiring_epoch <= (select max (epoch_no) from block)
                ) ;
```

To include the pool hash in the query output:

```
cexplorer=# select * from pool_update inner join pool_hash on pool_update.hash_id = pool_hash.id
              where registered_tx_id in (select max(registered_tx_id) from pool_update group by hash_id)
              and not exists
                ( select * from pool_retire where pool_retire.hash_id = pool_update.hash_id
                            and pool_retire.retiring_epoch <= (select max (epoch_no) from block)
                ) ;
```

**Transaction fee for specified transaction hash:**

```
cexplorer=# select tx.id, tx.fee from tx
              where tx.hash = '\xf9c0997afc8159dbe0568eadf0823112e0cc29cd097c8dc939ff44c372388bc0' ;
   id    |  fee
---------+--------
 1000000 | 172433
```

**Transaction outputs for specified transaction hash:**

```
cexplorer=# select tx_out.* from tx_out inner join tx on tx_out.tx_id = tx.id
              where tx.hash = '\xf9c0997afc8159dbe0568eadf0823112e0cc29cd097c8dc939ff44c372388bc0' ;
   id    |  tx_id  | index |         address         |    value     |        address_raw        | payment_cred
---------+---------+-------+-------------------------+--------------+---------------------------+--------------
 2205593 | 1000000 |     1 | DdzFFzCqrh...u6v9fWDrML | 149693067531 | \x82d8185842...1a20a42e6f |
 2205592 | 1000000 |     0 | DdzFFzCqrh...DoV2nEACWf |   8991998000 | \x82d8185842...1a150033dc |
```

**Transaction inputs for specified transaction hash:**

```
cexplorer=# select tx_out.* from tx_out
              inner join tx_in on tx_out.tx_id = tx_in.tx_out_id
              inner join tx on tx.id = tx_in.tx_in_id and tx_in.tx_out_index = tx_out.index
              where tx.hash = '\xf9c0997afc8159dbe0568eadf0823112e0cc29cd097c8dc939ff44c372388bc0' ;
   id    | tx_id  | index |        address          |    value     |        address_raw        | payment_cred
---------+--------+-------+-------------------------+--------------+---------------------------+--------------
 2195714 | 996126 |     4 | DdzFFzCqrh...dtq1FQQSCN | 158685237964 | \x82d8185842...1a330b42df |
```

**Transaction withdrawals for specified transaction hash:**

Withdrawals can be featured within certain transactions; such functionality is supported starting from Shelley and further. 

```
cexplorer=# select withdrawal.* from withdrawal
              inner join tx on withdrawal.tx_id = tx.id
              where tx.hash = '\x0b8c5be678209bb051a02904dd18896a929f9aca8aecd48850939a590175f7e8' ;
  id   | addr_id |  amount   |  tx_id
-------+---------+-----------+---------
 27684 |   30399 | 154619825 | 2788211
```

**Get the stake distribution for each pool for a given epoch:**

The simplest query is:

```
cexplorer=# select pool_id, sum (amount) from epoch_stake
              where epoch_no = 216 group by pool_id ;

 pool_id |       sum
---------+-----------------
       1 |  25326935163066
       2 | 112825285842751
...
    1172 |       498620686
    1173 |     15024987189
(1114 rows)
```

Or, use the Bech32 pool identifier instead of the Postgres generated `pool_id` field:

```
cexplorer=# select pool_hash.view, sum (amount) as lovelace from epoch_stake
              inner join pool_hash on epoch_stake.pool_id = pool_hash.id
              where epoch_no = 216 group by pool_hash.id ;
                           view                           |    lovelace
----------------------------------------------------------+-----------------
 pool10p6wd9k0fwk2zqkqnqr8efyr7gms627ujk9dxgk6majskhawr6r |    789466838780
 pool1vvh72z8dktfy2x965w0yp5psmnyv3845pmm37nerhl6jk6m2njw |   1427697660218
...
 pool1tq03j8aa5myedlr8xj6tltstdsn5eprxq4cd4qr54mhv25unsyk |     50297430457
 pool1nux6acnlx0du7ss9fhg2phjlaqe87l4wcurln5r6f0k8xreluez |   5401615743207
(1114 rows)
```

**Get the delegation history for a specified stake address:**

```
cexplorer=# select delegation.active_epoch_no, pool_hash.view from delegation
              inner join stake_address on delegation.addr_id = stake_address.id
              inner join pool_hash on delegation.pool_hash_id = pool_hash.id
              where stake_address.view = 'stake1u8gsndukzghdukmqdsd7r7wd6kvamvjv2pzcgag8v6jd69qfqyl5h'
              order by active_epoch_no asc;
 active_epoch_no |                           view
-----------------+----------------------------------------------------------
             212 | pool1hwlghkwnjsjk8370qt3dvp23d7urwm36f95fmxcz3np2kghknj9
             214 | pool1xxhs2zw5xa4g54d5p62j46nlqzwp8jklqvuv2agjlapwjx9qkg9
             216 | pool1hwlghkwnjsjk8370qt3dvp23d7urwm36f95fmxcz3np2kghknj9
             217 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq
(4 rows)
```

This shows that the first delegation became active (i.e. earning rewards) in epoch 212 and that the address owner subsequently re-delegated their stake with those re-delegations becoming active in epochs 214, 216 and 217.

**Get the reward history for a specified stake address:**

```
cexplorer=# select reward.epoch_no, pool_hash.view as delegated_pool, reward.amount as lovelace
              from reward inner join stake_address on reward.addr_id = stake_address.id
              inner join pool_hash on reward.pool_id = pool_hash.id
              where stake_address.view = 'stake1u8gsndukzghdukmqdsd7r7wd6kvamvjv2pzcgag8v6jd69qfqyl5h'
              order by epoch_no asc ;
 epoch_no |                      delegated_pool                      | lovelace
----------+----------------------------------------------------------+----------
      212 | pool1hwlghkwnjsjk8370qt3dvp23d7urwm36f95fmxcz3np2kghknj9 |  2953284
      213 | pool1hwlghkwnjsjk8370qt3dvp23d7urwm36f95fmxcz3np2kghknj9 |  3333940
      214 | pool1xxhs2zw5xa4g54d5p62j46nlqzwp8jklqvuv2agjlapwjx9qkg9 |  3005843
      215 | pool1xxhs2zw5xa4g54d5p62j46nlqzwp8jklqvuv2agjlapwjx9qkg9 |  3552293
      216 | pool1hwlghkwnjsjk8370qt3dvp23d7urwm36f95fmxcz3np2kghknj9 |  3130673
      217 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq | 34339994
      218 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq | 30384189
      219 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq | 27293239
      220 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq | 36947267
      221 | pool15yyxtkhz64p7a8cnax9l7u82s9t9hdhyxsa3tdm977qhgpnsuhq | 27016952
(10 rows)
```

**Get the number of blocks created in an epoch by a specified pool:**

```
cexplorer=# select block.block_no, block.epoch_no, pool_hash.view as pool_view
              from block inner join slot_leader on block.slot_leader_id = slot_leader.id
              inner join pool_hash on slot_leader.pool_hash_id = pool_hash.id
              where block.epoch_no = 220
                and pool_hash.view = 'pool137x32lrkprphrd0aa8x4jqz98z6lc0wawlc88hdjeps4qe408ad' ;
 block_no | epoch_no |                        pool_view
----------+----------+----------------------------------------------------------
  4760198 |      220 | pool137x32lrkprphrd0aa8x4jqz98z6lc0wawlc88hdjeps4qe408ad
  4759847 |      220 | pool137x32lrkprphrd0aa8x4jqz98z6lc0wawlc88hdjeps4qe408ad
(2 rows)
```

**Get the number of blocks created by a specified pool for each epoch:**

```
cexplorer=# select block.epoch_no, count (*) as block_count
              from block inner join slot_leader on block.slot_leader_id = slot_leader.id
              inner join pool_hash on slot_leader.pool_hash_id = pool_hash.id
              where pool_hash.view = 'pool1nux6acnlx0du7ss9fhg2phjlaqe87l4wcurln5r6f0k8xreluez'
              group by block.epoch_no, pool_hash.view ;
 epoch_no | block_count
----------+-------------
      212 |           1
      213 |           2
      214 |           2
      215 |           4
....
```

**Get the tx_id, tx_block_id, tx_out_address of a voting registration:**

```
cexplorer=# select tx.id as tx_id, tx.block_id as tx_block_id, tx_out.address as tx_out_address
              from tx inner join tx_out on tx.id = tx_out.tx_id
              where tx.hash = '\x9053a4cf0c6c9fb29792c78e688c5915a02909d0073371d8fff1abba0bed3065';
  tx_id  | tx_block_id |                       tx_out_address
---------+-------------+------------------------------------------------------------
 3192730 |     5083822 | addr1vy6d0htdaa9k8du2262p2ju74s25g6rjyjsc9x2fky9r6jq402r08
....
```

To find a transaction hash of your voter registration:

```
cardano-cli transaction txid --tx-file metadata.txsigned
9053a4cf0c6c9fb29792c78e688c5915a02909d0073371d8fff1abba0bed3065
```

**Get the amount delegated by epoch for a specified address:**

```
cexplorer=# select stake_address.view as stake_address, epoch_stake.epoch_no, epoch_stake.amount
              from stake_address inner join epoch_stake on stake_address.id = epoch_stake.addr_id
              where stake_address.view = 'stake1u8mt5gqclkq0swmvzx9lvq4jgwsnx9yh030yrxwqwllu0mq2m0l4n' ;
                        stake_address                        | epoch_no |   amount
-------------------------------------------------------------+----------+-------------
 stake1u8mt5gqclkq0swmvzx9lvq4jgwsnx9yh030yrxwqwllu0mq2m0l4n |      211 |  1561003730
 stake1u8mt5gqclkq0swmvzx9lvq4jgwsnx9yh030yrxwqwllu0mq2m0l4n |      212 |  1561003730
...
```

**Get the total orphaned rewards:**

Orphaned rewards for epoch `N` are rewards that accrue to a stake address that was registered before the end of epoch `N - 2` but for which the stake address has been de-registered before the rewards could be distributed (which happens at the start of epoch `N + 2`).

```
cexplorer=# select max (epoch_no) as max_epoch_no, sum (amount) / 1000000 as orphaned_reward_ada
              from orphaned_reward ;
 max_epoch_no | orphaned_reward_ada
--------------+---------------------
          238 |  67129.771592000000
```




