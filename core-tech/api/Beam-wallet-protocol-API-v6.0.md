## API Error codes

List of the possible error codes (with messages):

* -32600 InvalidJsonRpc ("Invalid JSON-RPC.") 
* -32601 NotFoundJsonRpc ("Procedure not found.")
* -32602 InvalidParamsJsonRpc ("Invalid parameters.")
* -32603 InternalErrorJsonRpc ("Internal JSON-RPC error.")
* -32001 InvalidTxStatus ("Invalid TX status.")
* -32002 UnknownApiKey ("Unknown API key.")
* -32003 InvalidAddress ("Invalid address.")
* -32004 InvalidTxId ("Invalid transaction ID.")* `totals` is an array of totals per asset. Assets that are included in this array by default:
    * BEAM (asset id 0)
    * All assets which have UTXOs.
    * All owned assets, even if there is no UTXOs for the given asset.

Asset is included in the `totals` array even if all UTXOs are spent or if there are no UTXOs but asset is owned (issued by the wallet). In case when all the UTXOs are spent totals' fields except `asset_id` would be 0 (zero balance). To avoid getting assets with zero balance specify `"params": {"nz_totals": true}`. BEAM is always included, even if balance is 0 and `nz_totals` is true.

Maximum asset emission is 2<sup>128</sup>-1. To ensure compatibility with JavaScript raw number (`available`, `maturing` &c.) returned only if it is less than or equal to `Number.MAX_SAFE_INTEGER` (2<sup>53</sup>-1). If total's value is greater than `Number.MAX_SAFE_INTEGER` only corresponding string representation (`xxxx_str`) is returned.
* -32005 NotSupported ("Feature is not supported")
* -32006 InvalidPaymentProof ("Invalid payment proof provided")
* -32007 PaymentProofExportError ("Cannot export payment proof")
* -32008 SwapFailToParseToken ("Invalid swap token.")
* -32009 SwapFailToAcceptOwn ("Can't accept own swap offer.")
* -32010 SwapNotEnoughtBeams ("Not enought beams.")
* -32011 SwapFailToConnect ("Doesn't have active connection.")
* -32012 DatabaseError ("Database error")
* -32013 DatabaseNotFound ("Database not found")
* -32014 ThrottleError ("Requests limit exceeded")
* -32015 NotOpenedError ("Wallet not opened")
* -32016 NoSwapsError ("Swaps are not enabled")
* -32017 UnexpectedError ("Unexpected call")
* -32018 ContractCompileError ("Failed to compile contract")
* -32019 ContractError ("Contract call failed")
* -32020 NotAllowedError ("Call is not allowed")
* -32021 UserRejected ("Call is rejected by user")

## API Methods

API has the following methods:

- [create_address](#create_address) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [delete_address](#delete_address) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [edit_address](#edit_address) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [addr_list](#addr_list) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [validate_address](#validate_address) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [tx_send](#tx_send) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [tx_asset_issue](#tx_asset_issue) 
- [tx_asset_consume](#tx_asset_consume) 
- [tx_asset_info](#tx_asset_info) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [tx_status](#tx_status) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [tx_split](#tx_split) 
- [tx_cancel](#tx_cancel) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [tx_delete](#tx_delete) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [get_utxo](#get_utxo) 
- [tx_list](#tx_list) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [wallet_status](#wallet_status) 
- [generate_tx_id](#generate_tx_id) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [export_payment_proof](#export_payment_proof) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [verify_payment_proof](#verify_payment_proof) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [get_asset_info](#get_asset_info) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [set_confirmations_count](#set_confirmations_count) 
- [get_confirmations_count](#get_confirmations_count) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)
- [invoke_contract](#invoke_contract) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)![](https://img.shields.io/badge/-aysnc-orange)
- [process_invoke_data](#process_invoke_data) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)![](https://img.shields.io/badge/-aysnc-orange)
- [block_details](#block_details) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)![](https://img.shields.io/badge/-aysnc-orange)
- [calc_change](#calc_change) ![](https://img.shields.io/badge/-DAPPs%20allowed-green)

## SWAP API Methods

If you build `wallet-api` with `BEAM_ATOMIC_SWAP_SUPPORT` then you can use additional [SWAP API methods](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-SWAP-API-(BETA)).

## create_address

Creates new receiver address.

`-->`
```json
{
    "jsonrpc": "2.0", 
    "id": 1,
    "method": "create_address", 
    "params":
    {
        "type": "regular",
        "expiration": "auto",
        "comment": "John Smith",
        "new_style_regular" : true
    }
}
```
* `type` the type of address, possible values are
  * `regular` - regular old-style address, default
  * `offline` - address offline transaction using Lelantus MW shielded pool, could be used more than once, there are no guaranties for extended privacy
  * `max_privacy` - address for guarantied maximum privacy transaction. Could be used only once.
  * `public_offline` - address for donation, could be used as many times as you wish
  * `regular_new` - since v6.0, regular new-style address (base58), this address has SBBS internally 

  Details of some address types are described [in this document](https://github.com/BeamMW/beam/wiki/Lelantus-CLI). Since v6.0, UI gives the sender an opportunity to choose between regular and offline send operation and the default address type provided via the "receive" UI dialog is an `offline` address with `offline_payments=1`

* `expiration` 
  * before v6.0 can be `expired/never/24h`. 
  * since  v6.0 can be `expired/never/24h/auto`.

  Optional, by default address expires in 24 hours. `auto` expiration means that address expires in 61 days (roughly 2 months). `auto` addresses are renewed for another 2 months when any transaction is received with this address.

* `comment` - any string, optional.

* `new_style_regular` - if this flag is `true` new style base58 address will be returned otherwise hexadecimal string will be returned, this option is ignored for the types different `regular`. `type=regular` + `new_style_regular=true` = `type=regular_new`.

* `offline_payments` - number of offline payments embedded into an `offline` address, default value is `1`. Valid only for the `type=offline`.

All these address types could be passed as `address` parameter of the [send](#tx_send) method and/or used with the cli/UI. `--enable_lelantus` should be used in the command line of the `wallet-api` to enable sending/receiving using `offline`, `max_privacy` and `public_offline` transactions. To generate `offline`/`max_privacy`/`public_offline` addresses API must be connected to the own node (node with the wallet's viewer key).

`<--`
```json
{
    "jsonrpc" : "2.0", 
    "id" : 1,
    "result" : "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67"
}
```

## validate_address

Just a simple validations, checks if the address isn't garbage and belongs to our elliptic curve. Also returns `is_mine == true` if address is found in the local wallet DB.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 1,
    "method": "validate_address", 
    "params":
    {
        "address" : "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67"
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 1,
    "result" : 
    {
        "is_valid" : true,
        "is_mine" : false,
        "type": "offline", 
        "payments": 3
    }
}
```

* `type` type of address. Check [create_address](#create_address) for details
* `payments` number of offline payments left for the `offline` address

## addr_list
Get addresses stored in the wallet database.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 8,
    "method":"addr_list",
    "params":
    {
        "own" : true
    }
}
```
* `own`, optional bool. If `true` returns only own addresses, if `false` returns only peer addresses. By default is false.

`<--`
```json
{
    "id": 8,
    "jsonrpc": "2.0",
    "result":
    [{
        "address": "29510b33fac0cb20695fd3b836d835451e600c4224d8fb335dc1a68271deb9b6b5b",
        "category": "",
        "create_time": 1553174321,
        "duration": 1520,
        "expired": true,
        "comment": "",
        "own": true,
        "own_id": 16730903,
        "own_id_str": "16730903",
        "identity": "2d8738b0424ad50631e902fab655e7e1795fbb8d92d47c4c8df7336870fcadf5",
        "type": "regular",
        "wallet_id": "29510b33fac0cb20695fd3b836d835451e600c4224d8fb335dc1a68271deb9b6b5b"
    }] 
}
```

* `address` address that should be distributed among users and used to send funds to
* `own_id` unsigned 64 bit index used to generate given address
* `own_id_str` string representation of index used to generate given address (for JavaScript)
* `identity` identity linked to the given address.
* `type` address type. Consult [create_address](#create_address) method for detailed description
* `wallet_id` - Wallet ID (SBBS address) associated with the given address. This is the same as an SBBS address in CLI/UI. Note that not all address types have an associated SBBS address. For example non-own `max_privacy` addresses do not have `wallet_id`. While own `max_privacy` ones do it is recommended not to publish it. SBBS address can be safely published for `regular`, `regular_new` and `offline` addresses.

### Identity
Starting from version 4.2 we introduce an additional entity in order to make transfer more reliable. It is called `identity`. The main goal of `identity` is to proof and ensure that transaction performs between claimed participants also it could be done using untrusted 3rd parties. The main scenario is _hardware wallet_ when host machine could be compromised, since it has ability to generate SBBS addresses, it can sign payment proof or accept transaction without need to ask hardware wallet about any secret, but if we give our identity to the other side (in addition to SBBS address), and this identity can be generated using hardware wallet only, then only hardware wallet can sign or approve transaction.

## delete_address
Delete specific address from the wallet.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 8,
    "method":"delete_address",
    "params":
    {
        "address" : "29510b33fac0cb20695fd3b836d835451e600c4224d8fb335dc1a68271deb9b6b5b"
    }
}
```
`<--`
```json
{
    "id": 8,
    "jsonrpc": "2.0",
    "result": "done"
}
```

## edit_address
Edit specific address. You can change the `comment` and/or `expiration`. Consult [create_address](#create_address) for more details on acceptable values.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 8,
    "method":"edit_address",
    "params":
    {
        "address" : "29510b33fac0cb20695fd3b836d835451e600c4224d8fb335dc1a68271deb9b6b5b",
        "comment" : "John Smith",
        "expiration" : "expired"
    }
}
```
`<--`
```json
{
    "id": 8,
    "jsonrpc": "2.0",
    "result": "done"
}
```

## tx_send
Send BEAM or asset to a given address.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "method":"tx_send", 
    "params":
    {
        "value": 12342342,
        "fee": 2,
        "from": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
        "address": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
        "comment": "thank you!",
        "asset_id": 1,
        "offline": false
    }
}
```
* `value` how much BEAM or asset to send in groth or asset groth.
* `fee` always in BEAM groth, optional. Omit for default fee.
* `from` own address, optional. If omitted new own address is generated and registered.
* `address` receiver's SBBS address or token. Token is an extension of SBBS address. It is a _base58_ encoded byte buffer which contains serialized parameters of a transaction. At least it contains SBBS `address` and `identity`.
* `comment` transaction comment, optional. 
* `txId` optional, you can provide your own transaction ID.
* `asset_id` asset id to send, optional. Present starting from v5.0 and can be used only after Fork 2. Omit or set to 0 for BEAM transaction. If `asset_id` is non-zero assets must be enabled (`--enable_assets`) or method would fail.
* `offline` - since v6.0 offline addresses by default start the regular online transaction. Specify `"offline":true"` to start an offline transaction. Applied only for `offline` addresses and ignored for all other address types.

You can preselect specific UTXOs for a transaction and fee via optional `coins` array parameter like `"coins" : ["00057e8eca5673476e6f726d000000000000015d3ef79800", "00057e8eca1233476e6f726d000000000000015d3ef79800"]`. In case of asset transaction you can specify both asset coins (to send) and non-asset coins (to pay fee).

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "result":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db"
    }
}
```
Returns transaction id or [error code](#api-error-codes).

## tx_split
Creates a specific set of outputs with given values.

NOTE: The `session` parameter is not IMPLEMENTED and was removed! 

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 5,
    "method":"tx_split", 
    "params":
    {
        "coins" : [11, 12, 13, 50000000],
        "fee" : 100,
        "asset_id": 1
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 5,
    "result":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db"
    }
}
```

* `fee` is always in BEAM groth. Omit for default fee.
* `coins` amounts of coins your want to have after split.
* `txId` optional, provide your own transaction ID.
* `asset_id` asset id to split, optional. Omit or set to 0 to split BEAM coins. Present starting from v5.0 and can be used only after Fork 2. If `asset_id` is non-zero assets must be enabled (`--enable_assets`) or method would fail.

## tx_asset_issue

Available from v5.0, can be used after Fork2. Assets must be enabled (`--enable_assets`) or method would fail.

Mints new asset coins. You must own the asset and info about the asset should be in a local database. Use [tx_asset_info](#tx_asset_info) to retrieve the latest asset info if necessary. Asset minting is free. You need to pay only regular transaction fee.

`-->`
```json
{
    "jsonrpc": "2.0", 
    "id": 2,
    "method": "tx_asset_issue", 
    "params":
    {
        "value": 6,
        "asset_id": 1
    }
}
```

* `value` how much asset to mint, in asset groth.
* `fee` transaction fee in BEAM groth. Omit to use default fee.
* `asset_id` asset id of the asset to mint. 
* `txId` optional,  provide your own transaction ID.
* `asset_meta` obsolete and removed in v6.0.

You can preselect specific BEAM UTXOs for a transaction fee by adding `coins` array parameter, like `"coins" : ["00057e8eca5673476e6f726d000000000000015d3ef79800", "00057e8eca1233476e6f726d000000000000015d3ef79800"]`

`<--`
```json
{
    "jsonrpc": "2.0", 
    "id": 2,
    "result":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db"
    }
}
```
Returns transaction id or [error code](#api-error-codes). 

## tx_asset_consume

Available from v5.0, can be used after Fork2. Assets must be enabled (`--enable_assets`) or method would fail.

Burns existing asset coins. You must own the asset itself as well as asset coins to burn them. You cannot burn asset coins that belong to another wallet. Info about the asset should be in a local database. Use [tx_asset_info](#tx_asset_info) to retrieve the latest asset info if necessary. Asset coins burning is absolutely free. You need to pay only regular transaction fee.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "method":"tx_asset_consume", 
    "params":
    {
        "value": 6,
        "asset_id": 1
    }
}
```

* `value` how much asset to burn, in asset groth.
* `fee` transaction fee in BEAM groth, omit for a default fee.
* `asset_id` id of the asset to consume. 
* `txId` optional, provide your own transaction ID
* `asset_meta` obsolete and removed since v6.0

You can preselect specific UTXOs to burn and to pay fee by adding `coins` array parameter, like `"coins" : ["00057e8eca5673476e6f726d000000000000015d3ef79800", "00057e8eca1233476e6f726d000000000000015d3ef79800"]`. Both asset coins (to burn) and BEAM coins (to pay fee) can be in this array.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "result":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db"
    }
}
```
Returns transaction id or [error code](#api-error-codes). 

## tx_asset_info

Available from v5.0, can be used after Fork2. Assets must be enabled (`--enable_assets`) or method would fail.

Retrieve full info about any registered asset and save in a local database. If asset has been unregistered or has been never registered transaction would fail. After transaction is completed you can use [get_asset_info](#get_asset_info) to read/get asset info from the local database.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "method": "tx_asset_info",
    "params": 
    {
        "asset_id": 1
    }
}
```

* `asset_id` asset id to retrieve info about. Can be used for any asset even if you don't own it.
* `txId` optional, provide your own transaction ID
* `asset_meta` obsolete and removed since v6.0

Asset info transaction is free, i.e. doesn't incur any fees.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "result":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db"
    }
}
```
Returns transaction id or [error code](#api-error-codes). 

## tx_cancel
Cancels running transaction

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method":"tx_cancel", 
    "params":
    {
        "txId" : "a13525181c0d45b0a4c5c1a697c8a7b8"
    }
}
```
* `txId` transaction id to cancel.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result": true
}
```
Returns `true` if successfully canceled or [error code](#api-error-codes) with the reason.

## tx_delete
Removes transaction from the local history. Cannot remove running transaction

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method":"tx_delete", 
    "params":
    {
        "txId" : "a13525181c0d45b0a4c5c1a697c8a7b8"
    }
}
```
* `txId` transaction id to be deleted.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result": true
}
```
Returns `true` if the transaction was successfully deleted or [error code](#api-error-codes) with the reason.

## tx_status

Get status & extended information about a single transaction by its transaction id. The example below is given for a simple transaction. Different transaction types have different status records. Consult [tx_list](#tx_list) method description for details. This method can be called for previously started asset-involving transactions even if assets are not enabled in the current session. 
 
`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method":"tx_status", 
    "params":
    {
        "txId" : "10c4b760c842433cb58339a0fafef3db" 
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result":
    { 
        "txId" : "10c4b760c842433cb58339a0fafef3db",
        "asset_id": 0,
        "comment": "",
        "fee": 100,
        "kernel": "0000000000000000000000000000000000000000000000000000000000000000",
        "receiver": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
        "sender": "f287176bdd517e9c277778e4c012bf6a3e687dd614fc552a1ed22a3fee7d94f2",
        "status": 4,
        "status_string" : "Failed",
        "tx_type": 0,
        "tx_type_string": "simple",
        "failure_reason" : "No inputs",
        "value": 12342342,
        "create_time" : 1551100217,
        "income" : false,
        "sender_identity": "a0a1ebbfeed5c312b309e32715c159e6b4548a6c6c3af25d0dbc16f37a1e9dd6",
        "receiver_identity": "2d8738b0424ad50631e902fab655e7e1795fbb8d92d47c4c8df7336870fcadf5",
        "token": "44pE7ySjZYjbLqwnTJANvr4BudMk1RdvWvaZnBvoCTwFnigfaTSza75bvw7x2GCa377Z4CSRRYZon44Ss9G9joSicNRAgts4u3pL6yV6jDQ6gAVJD9Scyr"
    } 
}
```

## tx_list

Get the transactions list.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 8,
    "method":"tx_list",
    "params":
    {
        "filter" : 
        {
            "status":4,
            "height":1055,
        },
        "skip" : 0,
        "count" : 10
    }
}
```
* `filter.status` optional, filter transactions by status
* `filter.height` optional, filter transactions by height. For simple, asset issue and asset consume transactions denotes height when the transaction was registered in the chain. For asset info transaction means height at which asset confirmation was received. Works only for transactions with status Completed(3). 
* `count` optional, number of transactions to get. By default, all transactions are returned.
* `skip` number of transactions to skip, 0 by default.

If assets are not enabled, the method returns only BEAM transactions. To get asset transactions you should enable assets (`--enable_assets`).

* `filter.asset_id` return only asset transactions with given asset id. Present starting from v5.0. If assets are not enabled and `filter.asset_id` is non-zero nothing would be returned.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 8,
    "result":
    [{ 
        "asset_id": 0,
        "txId" : "10c4b760c842433cb58339a0fafef3db",
        "comment": "",
        "fee": 0,
        "kernel": "0000000000000000000000000000000000000000000000000000000000000000",
        "receiver": "472e17b0419055ffee3b3813b98ae671579b0ac0dcd6f1a23b11a75ab148cc67",
        "sender": "f287176bdd517e9c277778e4c012bf6a3e687dd614fc552a1ed22a3fee7d94f2",
        "status": 4,
        "status_string" : "Failed",
        "failure_reason" : "No inputs",
        "value": 12342342,
        "create_time" : 1551100217,
        "income" : false,
        "token": "44pE7ySjZYjbLqwnTJANvr4BudMk1RdvWvaZnBvoCTwFnigfaTSza75bvw7x2GCa377Z4CSRRYZon44Ss9G9joSicNRAgts4u3pL6yV6jDQ6gAVJD9Scyr"
    },
    {
      "asset_id": 1,
      "asset_meta": "STD:N=Coin;SN=CN;UN=Cgro;NTHUN=Cgroth",
      "comment": "",
      "confirmations": 102,
      "create_time": 1586995332,
      "fee": 0,
      "height": 1908,
      "income": false,
      "receiver": "0",
      "sender": "0",
      "status": 3,
      "status_string": "asset confirmed",
      "txId": "d9f94306127a4ef894733f984b5512cf",
      "tx_type": 6,
      "tx_type_string": "asset info",
      "value": 0
    },
    {
        "asset_id": 1,
        "asset_meta": "STD:N=Coin;SN=CN;UN=Cgro;NTHUN=Cgroth",
        "comment": "",
        "confirmations": 1985,
        "height": 25,
        "create_time": 1586966478,
        "fee": 100,
        "income": false,
        "kernel": "1c9e4a9a61df1dda00db10ab4477f88355e13d4ed06c0db36c39b22a2a66f642",
        "receiver": "0",
        "sender": "0",
        "status": 3,
        "status_string": "asset issued",
        "txId": "77008a76aa4b4da697587040b2d21f1e",
        "tx_type": 2,
        "tx_type_string": "asset issue",
        "value": 500000000
    }]
}
```
* `token` retuned only in case if the transaction was started with a token.
* `height` and `confirmations` will be absent if the transaction isn't in the chain for simple, asset issue & asset consume transactions, or if asset confirmation is not received for asset info transaction.
* `confirmations` the number of blocks which confirm the presence of the transaction. 
* `sender_identity` and `receiver_identity` are present if both participants provided these.
* `status_string` is a string representation of `status` parameter.
* `asset_meta` is an asset metadata string. Present from v5.0.
* `tx_type` is present starting from v5.0.
* `tx_type_string` is a string representation of tx_type parameter. Present starting from v5.0.
* `asset_id` is returned starting from v5.0. For BEAM transactions it is always 0, for transactions that involve assets (including send/receive) it is an asset id. Can be 0 for asset info transaction if the transaction was started with asset_meta and the asset has been not found.
* `asset_meta` is an asset metadata string. Returned starting from v5.0. Always present for asset issue &  consume transactions. Can be empty for asset info transaction if it was started with asset_id and the asset has been not found.
* `kernel` is not returned for asset info transactions. These transactions do not have any kernel and only query node for information.

### IMPORTANT: The recommended number of confirmations for Beam transfers is 80 blocks.

### TX statuses
* pending (0)     - `pending` initial state, the transaction is created, but not sent yet
* in progress (1) - `self sending`/`waiting for sender`/`waiting for receiver`/`in progress`. For simple transactions indicates that the sender or receiver should come online to initiate the transaction. For asset issue/consume/info transactions indicates that transaction is being assembled. For asset info transaction means waiting for node response.
* canceled (2)    - `cancelled` by sender or due to rollback
* completed (3)   - transaction is `completed`/`received`/`sent`/`asset issued`/`asset consumed`/`asset confirmed`
* failed (4)      - `failed` for some reason or `expired`
* registering (5) - `self sending`/`receiving`/`sending`/`in progress` transaction is taken care by the blockchain, miners needs to PoW and to add it to a block, then block should be added to the blockchain. 


### TX types
* simple (0) - simple transaction, send/receive BEAM or asset
* reserved (1) - currently not returned 
* asset issue (2) - issue new asset units
* asset consume (3) - consume asset units
* reserved (4) - currently not returned 
* reserved (5) - currently not returned 
* asset info (6) - asset info transaction, receive full asset information from blockchain
* push transaction (7) - lelantus push transaction, is used for offline/max privacy payments
* contract (12) - starting with v6.0, transaction created by contract request, or to fund or execute node-base contract invocation

**N.B.** tx_list does not return swap, asset register & asset unregister transactions. Transactions are sorted by height in a descending order.


## wallet_status
Get current wallet status.

`-->`
```json
{
	"jsonrpc":"2.0", 
	"id": 6,
	"method":"wallet_status"
}
```

`<--`
```json
{
	"jsonrpc":"2.0", 
	"id": 6,
	"result":
	{
	    "current_height" : 1055,
	    "current_state_hash" : "f287176bdd517e9c277778e4c012bf6a3e687dd614fc552a1ed22a3fee7d94f2",
	    "prev_state_hash" : "bd39333a66a8b7cb3804b5978d42312c841dbfa03a1c31fc2f0627eeed6e43f2",
	    "available": 100500,
	    "receiving": 123,
	    "sending": 0,
	    "maturing": 50,
	    "locked": 30,
	    "difficulty": 2.93914,
	}
}
```
* `available` sum of available UTXOs you can spend  
* `sending/receiving` sum of UTXOs currently sending/receiving  
* `maturing` sum of UTXOs currently maturing
* `locked` not used at the moment, ignore
* `difficulty` the latest blockchain PoW difficulty

Starting from v5.0 assets support has been added. This slightly changes API response: totals array is added which provides amounts for each assets including BEAM (asset id 0). To get the totals array assets should be enabled (`--enable_assets`).

`-->`
```json
{
	"jsonrpc":"2.0", 
	"id": 6,
	"method":"wallet_status"
}
```

`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "current_height" : 1055,
    "current_state_hash" : "f287176bdd517e9c277778e4c012bf6a3e687dd614fc552a1ed22a3fee7d94f2",
    "prev_state_hash" : "bd39333a66a8b7cb3804b5978d42312c841dbfa03a1c31fc2f0627eeed6e43f2",
    "available": 100500,
    "receiving": 123,
    "sending": 0,
    "maturing": 50,
    "difficulty": 2.93914,
    "totals": [
      {
        "asset_id": 0,
        "available": 100500,
        "available_str": "100500",
        "maturing": 50,
        "maturing_str": "50",
        "receiving": 123,
        "receiving_str": "123",
        "sending": 0,
        "sending_str": "0"
      },
      {
        "asset_id": 1,
        "available": 2000000000,
        "available_str": "2000000000",
        "maturing": 0,
        "maturing_str": "0",
        "receiving": 0,
        "receiving_str": "0",
        "sending": 0,
        "sending_str": "0"
      }
    ]
  }
}
```

* `totals` is an array of totals per asset. Assets that are included in this array by default:
    * BEAM (asset id 0)
    * All assets which have UTXOs.
    * All owned assets, even if there is no UTXOs for the given asset.

Asset is included in the `totals` array even if all UTXOs are spent or if there are no UTXOs but asset is owned (issued by the wallet). In case when all the UTXOs are spent totals' fields except `asset_id` would be 0 (zero balance).

Maximum asset emission is 2<sup>128</sup>-1. To ensure compatibility with JavaScript raw number (`available`, `maturing` &c.) returned only if it is less than or equal to `Number.MAX_SAFE_INTEGER` (2<sup>53</sup>-1). If total's value is greater than `Number.MAX_SAFE_INTEGER` only corresponding string representation (`xxxx_str`) is returned.

### wallet_status & DApps

If method is called from DApp balances (`totals`, `available` &c) are not returned due to privacy reason. DApps are not allowed to get user's balances.

## get_utxo
Get list of all unlocked UTXOs.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 6,
    "method":"get_utxo",
    "params" :
    {
        "count": 10,
        "skip": 0,
        "sort": {
            "field": "amount",
            "direction": "asc"
        }
    }
}
```

* `count` number of UTXO to get, by default all the UTXOs are returned.
* `skip` number of UTXO to skip, default is `0`.
* `sort.field` name of field to sort. Available values are "id|asset_id|amount|type|maturity|createTxId|spentTxId|status|status_string"
* `sort.direction` sorting directions "asc" or "desc"

`<--`
```json
{
    "jsonrpc": "2.0", 
    "id": 6,
    "result":
    [{
        "id": 123,
        "asset_id": 0,
        "amount": 12345,
        "maturity": 60,
        "type": "mine",
        "createTxId": "10c4b760c842433cb58339a0fafef3db",
        "spentTxId": "",
        "status": 2,
        "status_string": "maturing"
    }]
}
```

* `asset_id` asset id if coin belongs to an asset and 0 for BEAM coins. By default method returns only BEAM coins. To get asset coins assets should be enabled (`--enable_assets`)
* `filter.asset_id` return only asset coins with given asset id, 0 for BEAM coins. Present starting from v5.0. Assets should be enabled.

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "get_utxo",
    "params": {
        "assets": true,
        "filter": {
            "asset_id": 1
        }
    }
}
```

`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": [
    {
      "amount": 500000000,
      "asset_id": 1,
      "createTxId": "fd705ce6f8c345309c865dc93de9fec5",
      "id": "0000000183ed2de4d94e6bd56e6f726d01000000000000001dcd6500",
      "maturity": 19,
      "spentTxId": "",
      "status": 1,
      "status_string": "available",
      "type": "norm"
    }
  ]
}
```

* `type` can be `fees` (comission), `mine` (coinbase), `norm` (usual coin, for example received via transaction), `chng` (change), `shld` (shielded)
* `status` can be `unavailable (0)`, `available (1)`, `maturing (2)`, `outgoing (3)`, `incoming (4)`, `spent (6)`, `consumed (7)`. `consumed` status is returned only for asset coins starting from v5.0. It means that the coin has been burned (consumed). 

## get_asset_info

Read asset info from local database. Asset info can be refreshed using [tx_asset_info](#tx_asset_info). It is also automatically refreshed during asset transactions (but not always) to ensure that asset operations are safe. This method can be called even if assets are not enabled. 

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 6,
    "method": "get_asset_info",
    "params" :
    {
        "asset_id": 1
    }
}
```

* `asset_id` asset id to retrieve info about. Can be used for any asset even if you don't own it.
* `asset_meta` obsolete and removed since v6.0

`<--`
```json
{
    "id": 1236,
    "jsonrpc": "2.0",
    "result": 
    {
        "asset_id": 1,
        "emission": 2000000000,
        "emission_str": "2000000000",
        "isOwned": 1,
        "lockHeight": 39,
        "metadata": "STD:N=NAME;SN=SNM;UN=UNIT;NTHUN=NTHUNIT",
        "ownerId": "0ae08a49e018e98177774294107dc033790b87538e54a20e99c6b98f1dbd39ce",
        "refreshHeight": 927
    }
}
```
Returns full asset info or [error code](#api-error-codes).

* `asset_id` asset id
* `metadata` asset metadata
* `emission` & `emission_str` total asset emission. Maximum asset emission is 2<sup>128</sup>-1. To ensure compatibility with JavaScript raw number returned only if it is less than or equal to `Number.MAX_SAFE_INTEGER` (2<sup>53</sup>-1). If asset emission is greater than `Number.MAX_SAFE_INTEGER` only corresponding string representation is returned.
* `isOwned` is 1 if you own this asset
* `lockHeight` last block when asset emission turned to/from 0.
* `refreshHeight` block at which asset information has been received. Please note, that all returned fields are valid only for this and previous blocks. In next blocks emission might change, asset become unregistered &c. Use [tx_asset_info](#tx_asset_info) to retrieve the most recent info.

## generate_tx_id
Generates ID for a transaction.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "method":"generate_tx_id"
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 2,
    "result":"10c4b760c842433cb58339a0fafef3db"
}
```

## export_payment_proof
Exports payment proof for given `txId`.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method": "export_payment_proof", 
    "params":
    {
        "txId" : "a13525181c0d45b0a4c5c1a697c8a7b8"
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result": 
    {
        "payment_proof": "8009f28991ef543253c8b6a2caf15cf99e23fb9c2b4ca30dc463c8ceb354d7979e80ef7d4255dd5e885200648abe5826d8e0ba0157d3e8cf9c42dcc8258b036986e50400371789ee82afc25ee29c9c57bcb1018b725a3a94c0ceb1fa7984ea13de4982553e0d78d925a362982182a971e654857b8e407e7ad2e9cb72b2b8228812f8ec50435351000c94e2c85996e9527d9b0c90a1843205a7ec8f99fa534083e5f1d055d9f53894"
    }
}
```

* `payment_proof` is a hex-encoded byte buffer which contains information about sender, receiver, amount and kernel, signed with receiver's private key. This info could be unpacked and verified using [verify_payment_proof](#verify_payment_proof) method by the third party. Receiver creates a payment proof in order give receiver an ability to proof that actual payment was made. Sender does not sign transaction if receiver does not provide him this evidence.

**N.B.** Payment proofs for asset transaction are supported starting from v5.0. These proofs cannot be verified by earlier clients/APIs versions. Regular BEAM payment proofs generated by the latest clients/APIs still would be accepted by earlier versions of clients/APIs.

## verify_payment_proof
Verifies `payment_proof`.

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method":"verify_payment_proof", 
    "params":
    {
        "payment_proof" : "8009f28991ef543253c8b6a2caf15cf99e23fb9c2b4ca30dc463c8ceb354d7979e80ef7d4255dd5e885200648abe5826d8e0ba0157d3e8cf9c42dcc8258b036986e50400371789ee82afc25ee29c9c57bcb1018b725a3a94c0ceb1fa7984ea13de4982553e0d78d925a362982182a971e654857b8e407e7ad2e9cb72b2b8228812f8ec50435351000c94e2c85996e9527d9b0c90a1843205a7ec8f99fa534083e5f1d055d9f53894"
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result": 
    {
        "is_valid": true,
        "asset_id": 0,
        "sender":   "9f28991ef543253c8b6a2caf15cf99e23fb9c2b4ca30dc463c8ceb354d7979e",
        "receiver": "ef7d4255dd5e885200648abe5826d8e0ba0157d3e8cf9c42dcc8258b036986e5",
        "amount":   2300000000,
        "kernel":   "ee82afc25ee29c9c57bcb1018b725a3a94c0ceb1fa7984ea13de4982553e0d78"
    }
}
```

* `is_valid` true if signature is valid,
* `sender` sender's SBBS address
* `receiver` is a receiver's SBBS address, is it is used to verify the signature 
* `amount` amount sent by the transaction in groth
* `asset_id` asset id of the transaction, 0 for BEAM. Returned starting from v5.0
* `kernel` id of the kernel of this transaction

**N.B.** v5.0+ APIs accept proofs generated by older versions.

## invoke_contract

Available from v6.0, can be used after Fork3. 

Invokes application shader. Only application/wallet side shaders are accepted. Use CLI to call a contract shader. 

`-->`
```json
{
    "jsonrpc": "2.0", 
    "id": 2,
    "method": "invoke_contract", 
    "params":
    {
        "contract": [123, 123, 123],
        "args": "role=manager,action=view"
    }
}
```

OR

`-->`
```json
{
    "jsonrpc": "2.0", 
    "id": 2,
    "method": "invoke_contract", 
    "params":
    {
        "contract_file": "....",
        "args": "role=manager,action=view"
    }
}
```

* `contract` contract code as raw bytes array, optional.
* `contract_file` file name to read contract code from, optional. Absolute or relative to the executable CWD.
* `args` contract arguments, optional. Depends on the called shader. Consult shader-specific docs for details.
* `create_tx` bool, pass `true` create transaction automatically if contract wants it and return `txid`. Pass false to avoid transaction creation, get raw tx data that can be passed to [process_invoke_data](#process_invoke_data) later. By default is `true`.

`contract` parameter always takes precedence over the `contract_file`. 

`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "output": "{\"contracts\": [{\"cid\": \"2b105519ee7e170d92abffbc1eb5426d4e20910af2f54bcee43eb84c74dcfa4b\",\"Height\": 27004},{\"cid\": 
               \"55df189ac5c1f9b982fa34fb8050f005c331203023816ddbb2d641a96a3fcb2a\",\"Height\": 27001},{\"cid\": 
               \"6add645863c29d9dc43ab9987af870cb91fd0dfb4e26ae09ca97ac8ff5372f6e\",\"Height\": 532}]}"
  }
}
```

* `output` contract output. Depends on the called shader. Can be empty
* `txid` transaction created by contract (if any)
* `raw_data` raw data returned by the contract based on which transaction can be created using [process_invoke_data](#process_invoke_data)

### Async Notice

invoke_contract method is asynchronous in nature and can take up to several blocks to execute (depends on shader). Please setup your API connection correspondingly (keep alive HTTP or use socket).

### Shader Errors

Please notice that API returns errors only if there is an API error (no shader code, parse error &c.). If there was an error inside the shader itself API call would be successful and shader output should be checked. For example:

### Code caching

If there was an `invoke_contract` call before `contract`/`contract_file` params can be omitted. In this case contract code from previous call would be used. In case of wallet_api code caching works on per-connection level. In case of applications caching works on per-application level.

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "invoke_contract",
    "params": {
        "contract_file": "./faucetManager.wasm",
        "args": "role=manager"
    }
}
```

`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "output": "{\"error\": \"Action not specified\"}"
  }
}
```

## process_invoke_data 

Creates transaction requested by a contract

`-->`
```json
{
    "jsonrpc": "2.0",
    "id": 1236,
    "method": "process_invoke_data",
    "params": {
        "data": [123, 123, 123]
    }
}
```

* `data` raw invoke data bytes array returned by [invoke_contract](#invoke_contract), mandatory

`<--`
```json
{
  "id": 1236,
  "jsonrpc": "2.0",
  "result": {
    "txid": "..."
  }
}
```

* `txid` contract transaction id created as a result of the call


## calc_change
Calculates change for given `amount`

`-->`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "method":"calc_change", 
    "params":
    {
        "amount" : 1234,
        "asset_id": 2,
        "fee": 10000,
        "is_push_transaction": true
    }
}
```

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result":
    {
        "asset_change": 12,
        "asset_change_str": "12",
        "change": 12,
        "change_str": "12",
        "explicit_fee": 1100000,
        "explicit_fee_str": "1100000"
    }
}
```

where

* `amount` is a requested amount we are going to send
* `asset_id` optional asset id of the requested amount. Default is 0 (BEAM). This method can be used for non-BEAM coins even if assets are not enabled.
* `fee` explicit fee in GROTHs chosen by the user
* `asset_change` change amount for requested `asset_id`
* `change` change in for BEAM. `asset_change` and `change` are equal if `asset_id == 0`, i.e. BEAM
* `explicit_fee` the fee which should be used
* `is_push_transaction` `true` if we are going to push transaction output into the shielded pool.

## block_details
Returns block header from blockchain

`-->`
```json
{
    "jsonrpc": "2.0", 
    "id": 2,
    "method": "block_details", 
    "params":
    {
        "height": 200
    }
}
```

* `height` height of block.

`<--`
```json
{
    "jsonrpc":"2.0", 
    "id": 4,
    "result":
    {
        "block_hash": "7aec20e5b56706d57a4dca1df4a4d3104dbdbf8d6005499845558b7a32694d02",
        "chainwork": "000000000000000000000000000000000000000000000000000000c503b09ac0",
        "definition": "0d3f4e63743760c6397d76086e1cc0ebe749c121206c8150738dc84edcd30092",
        "difficulty": 118.4594497680664,
        "height": 200,
        "kernels": "705cdfa154eec35e4b4b0e2163ff9c81f8f035641ee4b0b95c3f3c5b0f170ffe",
        "packed_difficulty": 114939514,
        "pow": "00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000004f04638c1f4cee1b7ad6d906",
        "previous_block": "d195cc69fbd2b402cc33ecb303f5391f0010f34ee059f30b908ed10f0fe5a255",
        "rules_hash": "340bbeabc390f481df1f84aedf0b679c1c7ed1f0e28d4340c9c49b4c41611f85",
        "timestamp": 1619598150
    }
}
```

* `block_hash` is a block hash
* `chainwork` is a chainwork
* `definition` is a definition
* `difficulty` is a difficulty
* `height` is a block height
* `kernels` is a kernels
* `packed_difficulty` is a packed difficulty
* `pow` is a pow
* `previous_block` is hash of previous block
* `rules_hash` is a rules hash
* `timestamp` is a timestamp

### Async Notice

block_details method is asynchronous in nature. Please setup your API connection correspondingly (keep alive HTTP or use socket).
