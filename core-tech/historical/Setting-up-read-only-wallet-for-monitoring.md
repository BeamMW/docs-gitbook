> **Historical Document** — This document describes a read-only monitoring wallet setup for early Beam wallet versions. It is preserved for historical reference only.

## Overview
The intention of this scenario is to have a some kind of monitoring wallet,  which can provide info about the list of the transactions, possibly, the list of UTXO, but without any ability to create or accept a new transaction and without access to any secrets.

In this scenario user should have:
* a regular (live ) wallet connected to the node, but initialized in the way, that it has separate storage for private data (master key) and for public data. User should be able to get this public database and provide it to monitoring wallet. 
* a read-only (monitoring) wallet api server, also connected to the node. It should have access to the public data only. This server should have set up Access control list (ACL). This wallet does not hold any secret keys

## Setup live wallet

1. Init live wallet with separated private data (--cold_wallet flag allows us to do this)

      `beam-wallet init --cold_wallet`

      it creates new wallet with two databases: `wallet.db` for public data and `wallet.db.private` for private.

1. Create "own" node for this wallet, for this you should export wallet's owner key and pass it to the node. This will let this wallet get notifications about UTXO which belong to this wallet.

      `beam-wallet export_owner_key`

      `beam-node --owner_key=<output from previous command> --peer=<url to existing node on target network>`

1. Use this wallet for sending/receiving beams. It should work as regular BEAM wallet with no exceptions. You even may put `wallet.db` and `wallet.db.private` to the application data folder of UI wallet and it should work.

## Setup read only monitoring wallet

1. On a different machine, setup another instance of wallet API
1. [Setup access to you wallet API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API#user-authorization).
1. Stop monitoring wallet
1. Copy `wallet.db` from live wallet to a monitoring wallet.
1. Launch wallet API server.
1. Repeat steps 3 to 5 with updated `wallet.db` from live wallet to get latest transactions.

`wallet-api -n <url of a node with port> --port=<port> --use_acl=1 --acl_path=wallet_api.acl`

This wallet API server should allow to perform:

```
validate_address
addr_list
delete_address
edit_address
tx_status
tx_list
tx_cancel
wallet_status
get_utxo
```
whereas the following methods should be prohibited (due to lack of secret key)

```
create_address
tx_send
tx_split
```

