# Confidential Assets

Confidential Assets (known as "Tokens" in the Beam ecosystem) are new assets created on the Beam blockchain with native support via Beam nodes, wallets, and DAPPS. New tokens created on the Beam blockchain offer the same privacy and security as Beam coins

Use the command line to register a Confidential Asset.

{% hint style="danger" %}
**Warning**

1. **Lock period** **will limit several asset operations for 1440 blocks (24 hours).** \
   An asset will "lock" when its total emission reaches or leaves 0. Lock periods prevent an asset owner from altering an asset during rollback, guaranteeing that a Receiver will receive the expected asset.
2.  **All incoming asset transactions will return an`assetsDisabled (43)`error code if CA is not enabled**.

    In the [CLI](broken-reference) and [API](https://github.com/BeamMW/beam/wiki/Beam-wallet-protocol-API#assets-support), specify the`--enable_assets`flag to perform any CA transactions.
3. **The fixed fee for asset registration is 3000 Beam**.\
   Users must register all CAs on-chain Before before usage.
{% endhint %}

## Register a Confidential Asset

#### Mandatory asset meta keys:

| Meta key | Description                                                 |
| -------- | ----------------------------------------------------------- |
| N        | Human-readable asset name.                                  |
| SN       | Human-readable short name (asset code).                     |
| UN       | Human-readable asset unit name.                             |
| NTHUN    | Human-readable smallest asset unit name ($$nth-unit$$name). |

To register a new asset, enter the following command:

```
./beam-wallet asset_reg -n <node address and port> --asset_meta <meta keys> –f <fee in Groth> --enable_assets <flag for assets>
```

Enter the meta keys:

```
./beam-wallet asset_reg -n 127.0.0.1:10000 --asset_meta "STD:N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth" -f 100 --enable_assets
```

### How it will look in your wallet logs

The wallet log should resemble the following:

```
  I 2020-06-05.11:43:19.757 [f1495c421611491d8432ccd03787e332] Asset with the owner ID 0bcb88af18ebec53 successfully registered
  I 2020-06-05.11:43:19.758 [f1495c421611491d8432ccd03787e332][1] Get proof for asset with the owner ID: 0bcb88af18ebec53
  I 2020-06-05.11:43:19.760 [f1495c421611491d8432ccd03787e332][1] Received proof for Asset with ID 8
  I 2020-06-05.11:43:19.761 [f1495c421611491d8432ccd03787e332][1] Asset ID: 1
  I 2020-06-05.11:43:19.761 [f1495c421611491d8432ccd03787e332][1] Owner ID: 0bcb88af18ebec53
  I 2020-06-05.11:43:19.762 [f1495c421611491d8432ccd03787e332][1] Issued amount: 0 agroth
  I 2020-06-05.11:43:19.762 [f1495c421611491d8432ccd03787e332][1] Lock Height: 41453
  I 2020-06-05.11:43:19.763 [f1495c421611491d8432ccd03787e332][1] Refresh height: 41453
  I 2020-06-05.11:43:19.763 [f1495c421611491d8432ccd03787e332][1] Metadata size: 70 bytes
  I 2020-06-05.11:43:19.765 [f1495c421611491d8432ccd03787e332][1]  N=Beam Coin
  I 2020-06-05.11:43:19.766 [f1495c421611491d8432ccd03787e332][1]  NTHUN=Groth
  I 2020-06-05.11:43:19.767 [f1495c421611491d8432ccd03787e332][1]  SN=BEAM
  I 2020-06-05.11:43:19.767 [f1495c421611491d8432ccd03787e332][1]  UN=Beam
  I 2020-06-05.11:43:19.768 [f1495c421611491d8432ccd03787e332][1] You own this asset
  D 2020-06-05.11:43:19.769 Async update started!
  I 2020-06-05.11:43:19.770 [f1495c421611491d8432ccd03787e332] Transaction completed
```

## Unregister an Asset

The owner of a Confidential Asset can unregister the asset from the blockchain. Asset emission must be zero and not in a lock period. The owner will pay a regular transaction fee and receive their initial asset registration fee of 3000 Beam.

To unregister an asset, enter the following command:

```
./beam-wallet asset_unreg --asset_id <your asset id> -n <node address and port> –f <fee in Groth> --enable_assets <flag for assets>
```

Enter your asset [meta keys](confidential-assets.md#mandatory-asset-meta-keys):

```
./beam-wallet asset_unreg --asset_meta <meta keys> -n <node address and port> –f <fee in Groth>  --enable_assets <flag for assets>
```

### How it looks in your wallet logs

Entering the`unreg --asset`command meta keys should resemble the following:

```
./beam-wallet asset_unreg --asset_id 1 -n 127.0.0.1:10000 –f 100 --enable_assets
```

```
./beam-wallet asset_unreg --asset_meta "STD:N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth" -n 127.0.0.1:10000 –f 100 --enable_assets
```

The wallet log output should be similar to following:

```
  I 2020-06-05.11:41:44.289 [95baa65a1f7e4432a29d6cfc61f9e2ca] Unregistering asset with the owner id a04e59ab3bbd2f7a. Refund amount is 1000 beams saving 100 groth transaction fee
  I 2020-06-05.11:41:44.290 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Get proof for asset with the owner ID: a04e59ab3bbd2f7a
  I 2020-06-05.11:41:44.300 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Received proof for Asset with ID 8
  I 2020-06-05.11:41:44.300 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Asset ID: 1
  I 2020-06-05.11:41:44.301 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Owner ID: a04e59ab3bbd2f7a
  I 2020-06-05.11:41:44.301 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Issued amount: 0 agroth
  I 2020-06-05.11:41:44.302 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Lock Height: 41440
  I 2020-06-05.11:41:44.302 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Refresh height: 41449
  I 2020-06-05.11:41:44.302 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Metadata size: 29 bytes
  I 2020-06-05.11:41:44.303 [95baa65a1f7e4432a29d6cfc61f9e2ca][1]  N=BEAM
  I 2020-06-05.11:41:44.304 [95baa65a1f7e4432a29d6cfc61f9e2ca][1]  NTHUN=M
  I 2020-06-05.11:41:44.304 [95baa65a1f7e4432a29d6cfc61f9e2ca][1]  SN=B
  I 2020-06-05.11:41:44.304 [95baa65a1f7e4432a29d6cfc61f9e2ca][1]  UN=E
  I 2020-06-05.11:41:44.305 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] You own this asset
  I 2020-06-05.11:41:44.309 [95baa65a1f7e4432a29d6cfc61f9e2ca] Creating BEAM coin:999 beams 99999900 groth, id fccf8897c24edd726e6f726d01000000000000174876e79c
  I 2020-06-05.11:41:44.338 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Transaction created. Kernel: 5c670069f696afe8fc01109b7a31b262ca44135740246b81b120da5d6519f8b3, min height: 41449, max height: 41569
  D 2020-06-05.11:41:44.344 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] sending tx for registration
  D 2020-06-05.11:41:44.346 Async update finished!
  D 2020-06-05.11:41:44.354 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] register status 1
  D 2020-06-05.11:41:44.355 Async update started!
  I 2020-06-05.11:41:44.356 [95baa65a1f7e4432a29d6cfc61f9e2ca][1] Get proof for kernel: 5c670069f696afe8
  I 2020-06-05.11:42:43.531 CoinID: Key=norm-1:0:18216929203118333298, Value=99999999900 Maturity=41451 Confirmed, Height=41451
  I 2020-06-05.11:42:43.536 [95baa65a1f7e4432a29d6cfc61f9e2ca] Transaction completed
```

## Issue Assets

{% hint style="info" %}
Maximum Asset emission is 2128-1 asset$$nth$$units. Maximum amount for a single issue transaction is 264-1 asset$$nth$$units.
{% endhint %}

New asset registration starts the total emission at zero. The owner must issue asset coins before performing transactions. Asset emissions are free, and the asset owner can mints coins as they deem necessary.

To issue an asset, enter following command:

```
./beam-wallet issue --asset_id <your asset id> -n <node address and port> -a <amount in Beams> -f <fee in Groth> --enable_assets <flag for assets>
```

Enter asset [meta keys](confidential-assets.md#mandatory-asset-meta-keys):

```
./beam-wallet issue --asset_meta <meta keys> -n <node address and port> -a <amount in Beams> –f <fee in Groth> --enable_assets <flag for assets>
```

### How it looks in your wallet logs

Entering the`issue --asset` command and meta keys should resemble the following:

```
./beam-wallet issue --asset_id 1 -n 127.0.0.1:10000 -a 10 --fee 100 --enable_assets
```

```
./beam-wallet issue --asset_meta "STD:N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth" -n 127.0.0.1:10000 -a 10 --f 100 --enable_assets
```

The wallet log output should resemble the following:

```
  I 2020-06-05.12:51:36.090 [c4ef8f97a8b54c29af48164f49a826f2] Generating asset with owner id 0bcb88af18ebec53. Amount: 10 ASSET
  I 2020-06-05.12:51:36.092 [c4ef8f97a8b54c29af48164f49a826f2] Creating BEAM coin (change):98450 groth, id 576672717e6f73cd63686e67010000000000000000018092
  I 2020-06-05.12:51:36.092 [c4ef8f97a8b54c29af48164f49a826f2] Creating ASSET coin:10 assets, asset id 8, id 0000000838c000f12e27ca8a6e6f726d01000000000000003b9aca00
  I 2020-06-05.12:51:36.160 [c4ef8f97a8b54c29af48164f49a826f2][1] Transaction created. Kernel: b551842338c73a3184b7c15e335716b15cbc933ae7754e649e50087fc6b04f62, min height: 41523, max height: 41643
  D 2020-06-05.12:51:36.176 [c4ef8f97a8b54c29af48164f49a826f2][1] sending tx for registration
  I 2020-06-05.12:51:36.179 [176aab9fcb3843719adfaf834a5d8c58] Generating asset with owner id 0bcb88af18ebec53. Amount: 10 ASSET
  I 2020-06-05.12:51:36.180 [176aab9fcb3843719adfaf834a5d8c58] Creating BEAM coin (change):186100 groth, id 30dd898c857eb7fa63686e6701000000000000000002d6f4
  I 2020-06-05.12:51:36.181 [176aab9fcb3843719adfaf834a5d8c58] Creating ASSET coin:10 assets, asset id 8, id 00000008732454300f0c4ae66e6f726d01000000000000003b9aca00
  I 2020-06-05.12:51:36.250 [176aab9fcb3843719adfaf834a5d8c58][1] Transaction created. Kernel: 2f1e6ba4c25d88405077e0d86d6feef9862bc052dec69d1141479107f76a3e93, min height: 41523, max height: 41643
  D 2020-06-05.12:51:36.265 [176aab9fcb3843719adfaf834a5d8c58][1] sending tx for registration
  D 2020-06-05.12:51:36.266 Async update finished!
  D 2020-06-05.12:51:36.273 [c4ef8f97a8b54c29af48164f49a826f2][1] register status 1
  D 2020-06-05.12:51:36.275 Async update started!
  I 2020-06-05.12:51:36.276 [c4ef8f97a8b54c29af48164f49a826f2][1] Get proof for kernel: b551842338c73a31
  D 2020-06-05.12:51:36.277 Async update finished!
  D 2020-06-05.12:51:36.281 [176aab9fcb3843719adfaf834a5d8c58][1] register status 1
```

## Consume Assets

The asset owner can consume (burn) asset coins but only if they also own the UTXOs in question.

To consume assets, etner the following command:

```
./beam-wallet consume --asset_id <your asset id> -n <node address and port> -a <amount in Beams> -f <fee in Groth> --enable_assets <flag for assets>
```

Enter asset [meta keys](confidential-assets.md#mandatory-asset-meta-keys):

```
./beam-wallet consume --asset_meta <meta keys> -n <node address and port> -a <amount in Beams> –f <fee in Groth> --enable_assets <flag for assets>
```

### How it looks in your wallet logs

Entering the`consume --asset`command meta keys should resemble the following:

```
./beam-wallet consume --asset_id 1 -n 127.0.0.1:10000 -a 10 -f 100 --enable_assets
```

```
./beam-wallet consume --asset_meta "STD:N=Beam Coin;SN=BEAM;UN=Beam;NTHUN=Groth" -n 127.0.0.1:10000 -a 10 -f 100 --enable_assets
```

The wallet log should look similar to something like:

```
[fa4a9b389df3452e8c00a2d81a283e3c] Consuming asset with owner id 0bcb88af18ebec53. Amount: 10 ASSET
  I 2020-06-05.15:02:24.636 [fa4a9b389df3452e8c00a2d81a283e3c] Creating BEAM coin (change):98250 groth, id 268a699b3a2e1b4463686e67010000000000000000017fca
  I 2020-06-05.15:02:24.667 [fa4a9b389df3452e8c00a2d81a283e3c][1] Transaction created. Kernel: 04d4127153f20d63c45ca2c0adb0e444f425068846e307e2e93ac34f645ef0fc, min height: 41663, max height: 41783
  D 2020-06-05.15:02:24.673 [fa4a9b389df3452e8c00a2d81a283e3c][1] sending tx for registration
  D 2020-06-05.15:02:24.674 Async update finished!
  D 2020-06-05.15:02:24.685 [fa4a9b389df3452e8c00a2d81a283e3c][1] register status 1
  D 2020-06-05.15:02:24.686 Async update started!
  I 2020-06-05.15:04:11.825 [40a4bb961f1842fc8c013c766c3982c2] Transaction completed
  I 2020-06-05.15:02:24.688 [fa4a9b389df3452e8c00a2d81a283e3c][1] Get proof for kernel: 04d4127153f20d63
```

## Send an Asset

To send a Confidential Asset, include the`--asset_id` and `--enable_assets` flag.

To send an asset, enter the following command:

```
./beam-wallet -n <node address and port> send -r <address or token> --a <amount> --asset_id <your asset id or an existing asset>  –f <fee in Groth>  --enable_assets <flag for assets>
```

The command with all necessary parameters should resemble the following:

```
./beam-wallet -n 127.0.0.1:10000 send -r 3a4ba5bab0a834cafa4201e74b44cbf6959794c9139e118bf7c70ab8e65f62886df -a 10 --asset_id 1 –f 100 --enable_assets
```

### How it looks in your wallet logs

The wallet log output should be similar to the following:

```
  I 2020-06-05.13:04:58.720 [60b40deb3f4e4792896f520f3eaa69bf] Sending 10 assets (fee: 100 groth), asset ID: 8
  I 2020-06-05.13:04:58.720 [60b40deb3f4e4792896f520f3eaa69bf] Max height for response: 42257
  4 [60b40deb3f4e4792896f520f3eaa69bf][1] Received PeerSig:       492b6ca53d539e17
  D 2020-06-05.13:05:01.286 Async update finished!
  D 2020-06-05.13:05:01.287 Async update started!
  D 2020-06-05.13:05:01.287 [60b40deb3f4e4792896f520f3eaa69bf][1] Received PeerSig:       492b6ca53d539e17
  I 2020-06-05.13:05:01.288 [60b40deb3f4e4792896f520f3eaa69bf][1] Transaction created. Kernel: ecc3b9fb2f44a6f6e76a4938a5f6e488e907c50ae5020e2919f4968887203359, min height: 41537, max height: 41657
  D 2020-06-05.13:05:01.304 [60b40deb3f4e4792896f520f3eaa69bf][1] sending tx for registration
  D 2020-06-05.13:05:01.305 Async update finished!
  D 2020-06-05.13:05:01.315 [60b40deb3f4e4792896f520f3eaa69bf][1] register status 1
  D 2020-06-05.13:05:01.316 Async update started!
  I 2020-06-05.13:05:01.317 [60b40deb3f4e4792896f520f3eaa69bf][1] Get proof for kernel: ecc3b9fb2f44a6f6
  I 2020-06-05.13:10:18.274 CoinID: Key=norm-1:0:4089269497513888394, Value=1000000000, AssetID=8 Maturity=41524 Spent, Height=41540
  I 2020-06-05.13:10:18.275 CoinID: Key=chng-1:0:6297846960702321613, Value=98450 Maturity=41524 Spent, Height=41540
  I 2020-06-05.13:10:18.277 CoinID: Key=chng-1:0:485144351240496243, Value=98350 Maturity=41540 Confirmed, Height=41540
  I 2020-06-05.13:10:18.278 Synchronizing with node: 50% (1/2)
  I 2020-06-05.13:10:18.278 Synchronizing with node: 100% (2/2)
  I 2020-06-05.13:10:18.279 Current state is 41540-b61ecf2e24003159
  D 2020-06-05.13:10:18.280 Async update started!
  I 2020-06-05.13:10:18.281 [60b40deb3f4e4792896f520f3eaa69bf][1] Get proof for kernel: ecc3b9fb2f44a6f6
  D 2020-06-05.13:10:18.282 Async update finished!
  D 2020-06-05.13:10:18.283 Async update started!
  I 2020-06-05.13:10:18.284 [60b40deb3f4e4792896f520f3eaa69bf] Transaction completed
```

{% hint style="danger" %}
**Warning**

Maximum amount for a single send transaction is 264-1 asset nth units.
{% endhint %}

## Receive an Asset

To recieve an a Confidential Asset, include the`--enable_assets`flag.

To receive an asset, enter the following command:

```
./beam-wallet listen -n <node address and port>  --enable_assets <flag for assets>
```

The command with all necessary parameters should resemble the following:

```
./beam-wallet listen -n 127.0.0.1:10000 --enable_assets
```

### How it looks in your wallet logs

The wallet log output should be similar to the following:

```
  I 2020-06-05.13:05:00.544 [60b40deb3f4e4792896f520f3eaa69bf] Receiving 10 assets (fee: 100 groth), asset ID: 8
  I 2020-06-05.13:05:00.585 [60b40deb3f4e4792896f520f3eaa69bf][1] Get proof for asset with id: 8
  D 2020-06-05.13:05:00.586 Async update finished!
  I 2020-06-05.13:05:00.587 [60b40deb3f4e4792896f520f3eaa69bf][1] Received proof for Asset with ID 8
  I 2020-06-05.13:05:00.587 [60b40deb3f4e4792896f520f3eaa69bf][1] Asset ID: 1
  I 2020-06-05.13:05:00.588 [60b40deb3f4e4792896f520f3eaa69bf][1] Owner ID: 0bcb88af18ebec53
  I 2020-06-05.13:05:00.588 [60b40deb3f4e4792896f520f3eaa69bf][1] Issued amount: 20 assets
  I 2020-06-05.13:05:00.588 [60b40deb3f4e4792896f520f3eaa69bf][1] Lock Height: 41524
  I 2020-06-05.13:05:00.589 [60b40deb3f4e4792896f520f3eaa69bf][1] Refresh height: 41537
  I 2020-06-05.13:05:00.589 [60b40deb3f4e4792896f520f3eaa69bf][1] Metadata size: 70 bytes
  I 2020-06-05.13:05:00.592 [60b40deb3f4e4792896f520f3eaa69bf][1]  N=Beam Coin
  I 2020-06-05.13:05:00.593 [60b40deb3f4e4792896f520f3eaa69bf][1]  NTHUN=Groth
  I 2020-06-05.13:05:00.593 [60b40deb3f4e4792896f520f3eaa69bf][1]  SN=BEAM
  I 2020-06-05.13:05:00.594 [60b40deb3f4e4792896f520f3eaa69bf][1]  UN=Beam
  D 2020-06-05.13:05:00.594 Async update started!
  I 2020-06-05.13:05:00.595 [60b40deb3f4e4792896f520f3eaa69bf] Transaction accepted. Kernel: ecc3b9fb2f44a6f6e76a4938a5f6e488e907c50ae5020e2919f4968887203359
  I 2020-06-05.13:05:00.599 [60b40deb3f4e4792896f520f3eaa69bf][1] Get proof for kernel: ecc3b9fb2f44a6f6
  D 2020-06-05.13:10:18.276 Async update started!
  I 2020-06-05.13:10:18.277 [60b40deb3f4e4792896f520f3eaa69bf][1] Get proof for kernel: ecc3b9fb2f44a6f6
  D 2020-06-05.13:10:18.278 Async update finished!
  D 2020-06-05.13:10:18.279 Async update started!
  I 2020-06-05.13:10:18.280 [60b40deb3f4e4792896f520f3eaa69bf] Transaction completed
```

## Asset info

The`asset_info`command has no required fees.

To view information on an on-chain registered asset, enter the following command:

```
./beam-wallet asset_info -n <node address and port> --asset_id <any asset registered>--enable_assets <flag  for assets>
```

The command with all necessary parameters should resemble the following:

```
./beam-wallet asset_info -n 127.0.0.1:10000 --asset_id 3 --enable_assets
```

### How it looks in your wallet logs

The wallet log output should be similar to following:

```
  I 2020-06-05.13:48:55.144 [07c80437b7e54edb91ba6240c36759a4][1] Get proof for asset with id: 3
  D 2020-06-05.13:48:55.144 Async update finished!
  I 2020-06-05.13:48:55.152 Sync up to 41590-693edc94cd712258
  I 2020-06-05.13:48:55.153 Synchronizing with node: 0% (0/1)
  I 2020-06-05.13:48:55.155 [07c80437b7e54edb91ba6240c36759a4][1] Received proof for Asset with ID 3
  I 2020-06-05.13:48:55.156 [07c80437b7e54edb91ba6240c36759a4][1] Asset ID: 3
  I 2020-06-05.13:48:55.156 [07c80437b7e54edb91ba6240c36759a4][1] Owner ID: 32de0a8f388d1243
  I 2020-06-05.13:48:55.157 [07c80437b7e54edb91ba6240c36759a4][1] Issued amount: 1 assets 20000000 agroth
  I 2020-06-05.13:48:55.158 [07c80437b7e54edb91ba6240c36759a4][1] Lock Height: 8714
  I 2020-06-05.13:48:55.158 [07c80437b7e54edb91ba6240c36759a4][1] Refresh height: 41589
  I 2020-06-05.13:48:55.159 [07c80437b7e54edb91ba6240c36759a4][1] Metadata size: 45 bytes
  I 2020-06-05.13:48:55.160 [07c80437b7e54edb91ba6240c36759a4][1]  N=Assets-Beam
  I 2020-06-05.13:48:55.160 [07c80437b7e54edb91ba6240c36759a4][1]  NTHUN=Groth
  I 2020-06-05.13:48:55.161 [07c80437b7e54edb91ba6240c36759a4][1]  SN=ASB
  I 2020-06-05.13:48:55.162 [07c80437b7e54edb91ba6240c36759a4][1]  UN=Beam
```

## Display Assets

By default, the CLI wallet doesn't display any information about assets you own or have.

To view assets info, assets UTXOs & assets transactions use the following commands:

### **View all assets**

#### **Asset info**

```
./beam-wallet info –assets
```

#### Asset transaction history

```
./beam-wallet info –assets –tx_history
```

#### Shielded UTXO (Lelantus) asset info

```
./beam-wallet info –assets –shielded_utxos
```

#### Shielded UTXOs transaction history

```
./beam-wallet info –assets –shielded_tx_history
```

### **View a specific asset**

#### Asset info

```
./beam-wallet info –asset_id 1
```

#### Asset transaction history

```
./beam-wallet info –asset_id 1 –tx_history
```

#### Shielded UTXO (Lelantus) asset info

```
./beam-wallet info –asset_id 1 –shielded_utxos
```

#### Shielded UTXO transaction history

```
./beam-wallet info –-asset_id 1 –shielded_tx_history
```

## One-sides payments (Lelantus)

LelantusMW shielded pool transactions fully support Confidential Assets. To insert or extract assets from the shielded pool, include the`--asset_id`and`--enable_assets`flag when creating a transaction.

