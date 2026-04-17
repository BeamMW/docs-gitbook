Beam delivers real-time news and exchange rates to wallet users through signed messages broadcast over the **Bulletin Board System (BBS)** — a private, encrypted message-passing layer built into the Beam network protocol. Messages are cryptographically signed so wallets can verify the publisher's identity before acting on the content.

Each broadcast message is transmitted to a dedicated BBS channel, propagates through nodes, and is received by listening wallet applications. Messages remain available on the network for **12 hours**.

## Broadcaster utility

The `broadcaster` binary is a command-line tool for creating and dispatching signed broadcast messages. It can also generate a publisher key pair.

**Mandatory options for all transmit operations:**

| Option | Description |
|--------|-------------|
| `-n`, `--node_addr` | Node address used as network entry point (e.g. `eu-node02.masternet.beam.mw:8100`) |
| `--key`, `--private_key` | 64-character hex private key used to sign messages |
| `--msg_type` | Message type: `update`, `exchange`, or `averify` |

**General options:**

| Option | Description |
|--------|-------------|
| `--command` | Command to run: `generate_keys` or `transmit` (default: `transmit`) |
| `--node_poll_period` | Node poll interval in milliseconds (default: `0` = persistent connection) |
| `--log_cleanup_days` | Log file retention in days (default: `5`) |
| `--config` | Path to configuration file (default: `bbs.cfg`) |

## Commands

### generate_keys

Generates a new publisher key pair. The private key signs outgoing messages; the public key is embedded in wallet applications to validate incoming messages.

```
./broadcaster --command generate_keys
```

Output:
```
Private key: f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59
Public key:  db617cedb17543375b602036ab223b67b06f8648de2bb04de047f485e7a9daec
```

Keep the private key secret. Share the public key with anyone who should receive and trust your broadcast messages.

### transmit

Sends a signed broadcast message to the network. `--command transmit` can be omitted — transmit is the default action.

The node address, private key, and message type are always required. Additional options depend on the message type.

## Message types

### update — wallet version notification

Notifies wallets that a new version of a client application is available.

| Option | Description |
|--------|-------------|
| `--upd_ver`, `--update_version` | New version string: `x.y.z` (mobile) or `x.y.z.r` (desktop, where `r` is UI revision) |
| `--upd_type`, `--update_type` | Application type: `desktop`, `android`, or `ios` |

_Example:_ announce desktop wallet v1.8.9:

```
./broadcaster -n "eu-node02.masternet.beam.mw:8100" \
  --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" \
  --msg_type 'update' \
  --upd_ver '1.8.9' \
  --upd_type 'desktop'
```

### exchange — currency exchange rate

Publishes an exchange rate for a currency pair.

| Option | Description |
|--------|-------------|
| `--exch_curr`, `--exchange_curr` | Source currency: `beam`, `btc`, `ltc`, `qtum`, `doge`, `dash`, `ethereum`, `dai`, `usdt`, `wbtc`, `bch` |
| `--exch_rate`, `--exchange_rate` | Rate in fixed-point format where `100000000` = 1 unit |
| `--exch_unit`, `--exchange_unit` | Target currency: `usd` (default) or `btc` |

_Example 1:_ 1 BEAM = 7.89654123 USD:

```
./broadcaster -n "eu-node02.masternet.beam.mw:8100" \
  --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" \
  --msg_type 'exchange' \
  --exch_curr 'beam' \
  --exch_rate '789654123'
```

_Example 2:_ 1 BEAM = 1.23456789 BTC:

```
./broadcaster -n "eu-node02.masternet.beam.mw:8100" \
  --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" \
  --msg_type 'exchange' \
  --exch_curr 'beam' \
  --exch_rate '123456789' \
  --exch_unit 'btc'
```

### averify — confidential asset verification

Publishes verification metadata for a [Confidential Asset](Confidential-assets). Wallets use this to display verified asset status, icons, and color branding.

| Option | Description |
|--------|-------------|
| `--asset_id` | Asset ID (32-bit unsigned integer; cannot be `0`, which is reserved for BEAM) |
| `--verified` | Verification status: `true` or `false` |
| `--predefined_icon` | Icon identifier string |
| `--predefined_color` | Color identifier string |

_Example:_

```
./broadcaster -n "eu-node02.masternet.beam.mw:8100" \
  --key "f70c36f2d8342b66e3081ea4d87543566d6ad242c6e61dbf926d57ff42de0c59" \
  --msg_type 'averify' \
  --asset_id 1 \
  --verified true \
  --predefined_icon 'beam_logo' \
  --predefined_color 'green'
```

## Security model

Only messages signed by a known publisher key are accepted by wallet applications. The broadcaster signs each message using the ECC (Schnorr) scheme. The corresponding public key must be compiled into the wallet application; messages signed with unknown keys are silently discarded.

The private key must be passed in 64-character hexadecimal format. Guard it carefully — anyone with the private key can publish trusted messages to all wallets that trust the corresponding public key.

### Embedded publisher public keys (beam-ui)

The desktop wallet ([beam-ui](https://github.com/BeamMW/beam-ui)) embeds one publisher public key per network, resolved at compile time via `get_BroadcastValidatorPublicKey()` in `beam/wallet/core/common.cpp`:

| Network | Public key |
|---------|------------|
| Mainnet | `8ea783eced5d65139bbdf432814a6ed91ebefe8079395f63a13beed1dfce39da` |
| Testnet | `dc3df1d8cd489c3fe990eb8b4b8a58089a7706a5fc3b61b9c098047aac2c2812` |
| Masternet | `db617cedb17543375b602036ab223b67b06f8648de2bb04de047f485e7a9daec` |
| Dappnet | `4c5b0b58caf69542490d1bef077467010a396cd20a4d1bbba269c8dff41da44e` |

The `BroadcastMsgValidator` loads the appropriate key on startup and rejects any message whose signature does not verify against it.