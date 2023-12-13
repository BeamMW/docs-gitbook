---
description: This documents describes CLI Confidential Assets workflow.
---

# Using Confidential Assets

## Overview

### Assets Support

{% hint style="info" %}
Confidential Assets (CA) support requires [`Fork2`](https://github.com/BeamMW/beam/discussions/1889) and at least [`v5.0`](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API) CLI/API. Any CA operation before `Fork2` would fail with the `AssetsDisabledFork2 (45)` error code.
{% endhint %}

In `v5.0` CA support is disabled by default to prevent compatibility issues. Currently, CA transactions are not supported in the GUI wallet, and the GUI wallet would reject all incoming CA transactions with the `AssetsDisabled (43)` error code. Rejected CA transactions do not display in the GUI wallet. Full GUI assets support is planned for later releases.

Users must specify the `--enable_assets` flag to perform  CA transactions with the CLI and API Without this flag, CA transactions would be rejected with the `AssetsDisabled (43)` error code.

Information about Assets UTXOs/Shielded Coins/Assets Transactions/Assets Summary can be displayed using `--assets` or `--asset_id` ID parameters added to the corresponding commands.

### Fees

All fees (transaction fees and registration deposit) are always paid in BEAM/Groth. It is not possible to pay any fees using assets.

### Asset Owner

`Asset Owner` is the person (wallet) who registered the asset. The asset owner owns the private key used for signing asset `issue/consume/unreg` operations.

### Asset ID

The asset is identified to the network by its`Asset ID` which is an unsigned integer. All asset-related actions are performed using the asset ID. The node automatically assigns the first available asset ID when registering an asset.

Technically there can be a situation when one asset is unregistered, `asset id` becomes available, and the next registered asset takes the id that has been previously used. A [Lock period](./#lock-period) is introduced to ensure that asset receiver would never receive an unexpected (forged) asset.

`Asset ID` of 0 is reserved and represents an original BEAM.

### Asset Info

`Asset info` is information about the asset. It includes asset metadata, total emission and lock height. Asset info is received by the wallet automatically during asset transactions or can be requested manually using [asset\_info](./#getting-asset-info) command.

Some parts of the asset info are valid only at the height the asset info has been received (`refresh height`). In subsequent blocks total asset emission can change or the asset become unregistered. Unregistering the asset invalidates all the information associated with the particular asset id.

### Limits

Maximum Asset emission is $$2^{128}-1$$ asset nth units. Maximum amount for a single asset transaction (issue, consume, send, receive \&c.) is $$2^{64}-1$$ asset nth units.

### Lock period

Lock period is a timeframe when several asset operations are restricted for safety reasons. This ensures that asset is not changed during rollback and/or by the asset owner and that receiver of the asset would receive exactly the expected asset.

Asset becomes locked every time when its total emission reaches 0 or leaves 0. Currently lock period duration is set to `1440` blocks (24 hours roughly).

Restricted operations are the following:

* Asset unreg
* Asset send/receive

### Restore

General rules apply to the restore process. You can restore your UTXOs but not transactions. To restore shielded UTXOs node used for the restore process should be running with your owner key. Asset info is not restored automatically. You would need to execute the `asset_info` command for each restored asset manually after the restore process is completed.

## Working with assets

### Asset registration

Before asset can be used it should be registered on chain using the `asset_reg` command:

```
./beam-wallet asset_reg --pass 1 -n 127.0.0.1:10000 --asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth;CUSTOM1=VAL1;CUSTOM2=VAL2" --fee 100000 --enable_assets
```

You must specify your wallet password, node address, asset meta, optional transaction fee and add `--enable_assets` flag.

### Fees

There is a fixed fee for asset registration of 3000 BEAM. This fee is mandatory, cannot be changed and deduced from your wallet automatically. Registration fee ensures that the network would not be spammed with dummy assets. Registration fee is returned to the owner of the asset as soon as the asset is unregistered. So basically 3000 BEAM are locked for the asset lifetime. You must also pay regular transactions fees. The `--fee` param applies only to the transaction fee, not the registration fee.

### Asset meta

Asset meta is a byte buffer associated with the asset and stored on chain. It is provided on asset registration and cannot be changed afterwards. Currently CLI expects UTF8 string with several mandatory `Key=Value` pairs. It is not possible to register an asset without meta. Please consult the [Asset Descriptor](asset-metadata-descriptor.md) document for more details.

### Asset ID

After successful asset registration it is associated with the `Asset ID` issued by the node. Asset becomes known to the world by its ID. Asset id is used in any asset operations performed by any person. Asset owner can perform asset operations using as asset id or asset meta. Consult the [Asset ID](./#asset-id) section for more details.

### Lock Period

Immediately after the asset registration it becomes locked for 24h hours. Consult the [Lock period](./#lock-period) section for more details.

### Privacy

Transaction kernel including all the asset meta becomes visible to the world. Node would know that you/your IP is the owner of the asset.

### API Restriction

Asset registration can be performed only via CLI. There is no API call for asset registration for safety reasons.

## Asset issue

After asset registration its total emission is set to 0. Before performing any asset transactions the owner must mint (issue) asset coins using the `issue` command. Only asset owner can mint asset coins.

```
./beam-wallet issue --pass 1 --asset_id 1 -n 127.0.0.1:10000 --amount 10 --fee 100000 --enable_assets
```

```
./beam-wallet issue --pass 1 --asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth;CUSTOM1=VAL1;CUSTOM2=VAL2" -n 127.0.0.1:10000 --amount 10 --fee 100000 --enable_assets
```

You must specify your wallet password, node address, asset id or asset meta, issue amount, optional transaction fee and add `--enable_assets` flag.

#### Coins availability

New Asset coins cannot be used until the issue transaction is completed. Please ensure that the minting process is completed and new coins are present in your wallet and marked as available before sending them.

#### Fees

Asset emission is absolutely free. You can mint as many coins as you want. You pay only regular transaction fee.

#### Limits

Maximum Asset emission is $$2^{128}-1$$ asset nth units. Maximum amount for a single issue transaction is $$2^{64}-1$$ asset nth units.

#### Lock Period

If after the issue operation total asset emission leaves 0 asset becomes locked. Consult the [Lock period](./#lock-period) section for more details. Asset issue can be performed even if asset is locked. Subsequent asset issue operations do not extend the lock period.

#### Privacy

Asset information is forcibly refreshed (received from node) during this operation. Operation fails if node doesn't confirm the asset. Transaction kernel including the issued asset amount and asset id becomes visible to the world. Node would know that you/your IP is the owner of the asset.

## Asset consume

Owner of the asset can consume (burn) asset coins. To burn the asset you must both be the owner of the asset and the owner of the particular coin/utxo. Asset can be consumed using the `consume` command.

```
./beam-wallet consume --pass 1 --asset_id 1 -n 127.0.0.1:10000 --amount 10 --fee 100000 --enable_assets
```

```
./beam-wallet consume --pass 1 --asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth;CUSTOM1=VAL1;CUSTOM2=VAL2" -n 127.0.0.1:10000 --amount 10  --fee 100000 --enable_assets
```

You must specify your wallet password, node address, asset id or asset meta, consume amount, optional transaction fee and add `--enable_assets` flag.

#### Fees

Asset consumption is free, requiring only regular transaction fees.

#### Limits

Maximum amount for a single consume transaction is $$2^{64}-1$$ asset nth units.

#### Lock Period

If after the consume operation total asset emission reaches 0 asset becomes locked. Consult the [Lock period](./#lock-period) section for more details.

#### Privacy

Asset information is forcibly refreshed (received from node) during this operation. Operation fails if node doesn't confirm the asset. Transaction kernel, including the consumed asset amount and asset id becomes visible to the world. Node would know that you/your IP is the owner of the asset.

## Asset unreg

Asset can be unregistered from chain by the owner. To unregister the asset its emission should be 0 and it should be not locked. Asset is unregistered using the `asset_unreg`command.

```
./beam-wallet asset_unreg --pass 1 -n 127.0.0.1:10000 --asset_meta "STD:SCH_VER=1;N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth;CUSTOM1=VAL1;CUSTOM2=VAL2" --fee 100000  --enable_assets
```

```
./beam-wallet asset_unreg --pass 1 -n 127.0.0.1:10000 --asset_id 1 --fee 100000 --enable_assets
```

You must specify your wallet password, node address, asset meta or asset id, optional transaction fee and add `--enable_assets` flag.

#### Fees

Asset unregistration is free. You pay only regular transaction fee. This operation also returns 3000 BEAM locked during the asset registration.

#### Asset ID

Asset ID becomes free after this operation and can be reused by another asset.

#### Limits

Maximum amount for a single consume transaction is $$2^{64}-1$$ asset nth units.

#### Privacy

Asset information is forcibly refreshed (received from node) during this operation. Operation fails if node doesn't confirm the asset. Transaction kernel which includes asset id becomes visible to the world. Node would know that you/your IP is the owner of the asset.

#### API Restriction

Asset unregistration can be performed only via CLI. There is no API call for asset unregistration for safety reasons.

## Getting Asset Info

Asset info can be received from node using `asset_info` command for any asset registered on chain. `asset_info` command stores the received info in a local database for future usage.

```
./beam-wallet asset_info --pass 1 -n 127.0.0.1:10000 --asset_id 1 --enable_assets
```

`asset_info` command always receives the latest information from node. To view locally stored information regular info command should be used with `--assets` or `--asset_id` ID parameters.

Asset owner can query asset info using asset meta (`--asset_meta "STD:...."`) as well. Non-owners can use only asset id.

#### Fees

`asset_info` operation is free. You do not pay any fees.

#### Privacy

`asset_info` operation only communicates with the node and doesn't leave any traces in the blockchain. Node would know that particular IP has been interested in given asset.

## Sending assets

Asset can be sent using regular `send` command. The only difference from regular BEAM transaction is the `--asset_id` and `--enable_assets` parameters.

```
./beam-wallet send -n 127.0.0.1:10000 --pass 1 -r 1ec08b72ea25cd471ec37f9088a1ae0dcb8f4526eff3b8ab38f8e23901e2adf48a2 --amount 5 --asset_id 1 --enable_assets
```

#### Limits

Maximum amount for a single send transaction is $$2^{64}-1$$ asset nth units.

#### Enable flag

Both sender and receiver should specify `--enable_assets` flag in command line otherwise transaction would fail with the `AssetsDisabled (43)` error code.

#### Lock period

Asset cannot be sent to non-owner during the lock period. Transaction would be rejected by the receiving party with the `AssetLocked (34)` error code.

#### Privacy

Send transaction doesn't refresh asset info. Asset ID is hidden during the send transaction though it is visible that the transaction is performed on some asset. General Mimblewimble rules apply to the transaction. Amount, sender and receiver are not disclosed.

## Receiving assets

Process of receiving assets is mostly identical to the receiving of regular BEAM.

#### Enable flag

CLI/API receives assets automatically if it is running (listens) with the `--assets_enable` option. If assets are not enabled all incoming asset transaction would be rejected with the `AssetsDisabled (43)` error code.

#### Lock period

Asset cannot be received by non-owner during the lock period. Transaction would be rejected by the receiving party with the `AssetLocked (34)` error code.

#### Privacy

Receive operation does not refresh asset info if there are any unspent UTXOs of the asset being received in the wallet. If there are no unspent UTXOs asset info might be refreshed if wallet doesn't have info about the asset (first receive) or info is older than the lock period duration.

Asset ID is hidden during the receive transaction though it is visible that the transaction is performed on some asset. General Mimblewimble rules apply to the transaction. Amount, sender and receiver are not disclosed.
