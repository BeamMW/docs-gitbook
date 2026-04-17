# Wallet Database Schema

The Beam wallet stores all local state in a SQLite database managed by `WalletDB` (`wallet/core/wallet_db.h`, `wallet/core/wallet_db.cpp`). The public interface is `IWalletDB`, which extends `IVariablesDB`.

All amounts are in Groth (1 BEAM = 100,000,000 Groth).

---

## Overview

Because Mimblewimble's blockchain records only UTXOs — not transaction history, addresses, or balances — the wallet must maintain its own local state. The DB tracks:

- **Coins** — transparent UTXOs and their lifecycle status
- **Shielded coins** — Lelantus outputs in the shielded pool
- **Addresses** — own addresses (key indices) and contacts (imported tokens)
- **Transaction parameters** — full negotiation state for every transaction
- **Assets** — Confidential Asset registry entries
- **Chain state** — block header history for FlyClient SPV proofs
- **Auxiliary** — vouchers, notifications, exchange rates, events, DEX orders, SBBS message queues

---

## Table Reference

### `storage` — Transparent UTXOs (Normal Coins)

Each row represents one UTXO visible to this wallet. Defined by `ENUM_ALL_STORAGE_FIELDS` in `wallet_db.cpp`.

| Column | SQLite Type | C++ Field | Notes |
|--------|-------------|-----------|-------|
| `Type` | INTEGER NOT NULL | `CoinID.m_Type` | `Key::Type` — Regular, Coinbase, Fee, Treasury, Change |
| `SubKey` | INTEGER NOT NULL | `CoinID.m_SubIdx` | Sub-key index |
| `Number` | INTEGER NOT NULL | `CoinID.m_Idx` | Key index within the sub-key |
| `amount` | INTEGER NOT NULL | `CoinID.m_Value` | Amount in Groth |
| `assetId` | INTEGER | `CoinID.m_AssetID` | `0` for BEAM |
| `maturity` | INTEGER NOT NULL | `Coin::m_maturity` | Height at which this coin becomes spendable |
| `confirmHeight` | INTEGER | `Coin::m_confirmHeight` | Height at which output was confirmed on-chain |
| `spentHeight` | INTEGER | `Coin::m_spentHeight` | Height at which input was spent on-chain |
| `createTxId` | BLOB | `Coin::m_createTxId` | TxID of the transaction that created this coin (optional) |
| `spentTxId` | BLOB | `Coin::m_spentTxId` | TxID of the transaction that spent this coin (optional) |
| `sessionId` | INTEGER | (obsolete) | Formerly used for API-level session coin locking; no longer used |

**Primary key:** `(Type, SubKey, Number, amount, assetId)` — this composite is the `CoinID`.

**Coin selection:** When building a transaction, `selectCoins(Amount)` is called to pick UTXOs covering the required amount. The implementation (`CoinSelector3` in `wallet_db.cpp`) uses a greedy strategy with heuristics to minimize the number of coins selected and the resulting change output size. The name "3" reflects three successive implementation attempts to handle large coin sets efficiently.

**Indices:** unique index on the primary key; secondary index on `confirmHeight`.

**Derived status** (not stored, computed at read time from heights and ongoing transactions):

| `Coin::Status` | Condition |
|----------------|-----------|
| `Spent` | `spentHeight != MaxHeight` |
| `Maturing` | Confirmed but `maturity > currentHeight` (coinbase, treasury) |
| `Outgoing` | Confirmed and available, but locked by an ongoing outgoing tx |
| `Available` | Confirmed, maturity met, not in any active tx |
| `Incoming` | Not yet confirmed; creating tx is still in progress |
| `Unavailable` | Not confirmed, no ongoing creating tx (e.g., after rollback) |
| `Consumed` | Asset UTXO converted back to BEAM |

---

### `ShieldedCoins` — Lelantus Shielded UTXOs

Each row represents one output in the Lelantus shielded pool. Defined by `ENUM_SHIELDED_COIN_FIELDS`.

| Column | SQLite Type | C++ Field | Notes |
|--------|-------------|-----------|-------|
| `Key` | BLOB NOT NULL PRIMARY KEY | `ShieldedCoin::m_CoinID` (BaseKey) | The spending key commitment |
| `User` | BLOB | `ShieldedTxo::ID::User` | Per-coin user data (packed into the shielded output) |
| `ID` | INTEGER NOT NULL | `ShieldedCoin::m_TxoID` | Global sequential shielded output index |
| `assetID` | INTEGER | `CoinID.m_AssetID` | `0` for BEAM |
| `value` | INTEGER NOT NULL | `CoinID.m_Value` | Amount in Groth |
| `confirmHeight` | INTEGER | `ShieldedCoin::m_confirmHeight` | Height at which the shielded output appeared |
| `spentHeight` | INTEGER | `ShieldedCoin::m_spentHeight` | Height at which the nullifier was included in a block |
| `createTxId` | BLOB | `ShieldedCoin::m_createTxId` | TxID of the push (shield) transaction |
| `spentTxId` | BLOB | `ShieldedCoin::m_spentTxId` | TxID of the pull (unshield) transaction |

Shielded coin status mirrors transparent coin status. The `UnlinkStatus` struct (computed from the global shielded output count, not stored) provides an anonymity window health metric used by the wallet's automatic unlink scheduler.

---

### `addresses` / `LaserAddresses` — Wallet Address Book

Both tables share the same schema (`ENUM_ADDRESS_FIELDS`). `LaserAddresses` stores Laser channel endpoints.

| Column | SQLite Type | C++ Field | Notes |
|--------|-------------|-----------|-------|
| `Address` | TEXT NOT NULL PRIMARY KEY | `WalletAddress::m_Token` | Serialized `TxParameters` encoding address type and embedded data |
| `walletID` | BLOB | `WalletAddress::m_BbsAddr` | `WalletID` used as the BBS channel address |
| `label` | TEXT NOT NULL | `WalletAddress::m_label` | User-visible display name |
| `category` | TEXT | `WalletAddress::m_category` | Optional group/category label |
| `createTime` | INTEGER | `WalletAddress::m_createTime` | Creation Unix timestamp |
| `duration` | INTEGER | `WalletAddress::m_duration` | Seconds until expiry; `0` means never-expiring |
| `OwnID` | INTEGER NOT NULL | `WalletAddress::m_OwnID` | Key derivation index; `0` for contacts |
| `Identity` | BLOB | `WalletAddress::m_Endpoint` | `PeerID` public key (endpoint identity) |

**Primary key:** `Token` (the address token string).
**Secondary index:** on `WalletID`.

Own addresses (`OwnID > 0`) are derived from the wallet's master key at index `OwnID`. Contact addresses (`OwnID = 0`) are imported from external token strings.

Address types encoded in `m_Token` via `TokenType`:

| `TokenType` | Meaning |
|-------------|---------|
| `RegularOldStyle` | Legacy SBBS address (WalletID only) |
| `RegularNewStyle` | SBBS address with endpoint identity |
| `Offline` | Pre-generated Lelantus vouchers embedded in token (10 by default) |
| `MaxPrivacy` | Lelantus push with live SBBS voucher exchange |
| `Public` | Single permanent offline voucher (public token) |

---

### `txparams` — Transaction Parameters

All transaction state is stored as serialized blobs in this table. The `TxDescription` view of a transaction is assembled from multiple rows with the same `txID`.

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `txID` | BLOB NOT NULL | 16-byte transaction identifier |
| `subTxID` | INTEGER NOT NULL | Sub-transaction index (`1` = primary; `>1` for swap sub-transactions) |
| `paramID` | INTEGER NOT NULL | `TxParameterID` enum value |
| `value` | BLOB | Serialized parameter value (via `yas` serializer) |

**Primary key:** `(txID, subTxID, paramID)`.
**Secondary index:** on `txID` for full-transaction queries.

Key `TxParameterID` values that form `TxDescription`:

| Parameter | Type | Notes |
|-----------|------|-------|
| `TransactionType` | `TxType` | See table below |
| `Amount` | `Amount` | Transfer amount in Groth |
| `Fee` | `Amount` | Transaction fee in Groth |
| `AssetID` | `Asset::ID` | `0` for BEAM |
| `MinHeight` | `Height` | Kernel min-height (lifetime start) |
| `PeerAddr` | `WalletID` | Counterparty BBS address |
| `MyAddr` | `WalletID` | Own BBS address used for this tx |
| `CreateTime` | `Timestamp` | Creation timestamp |
| `ModifyTime` | `Timestamp` | Last modification timestamp |
| `IsSender` | `bool` | True if this wallet initiated the transfer |
| `Status` | `TxStatus` | Lifecycle status |
| `KernelID` | `Merkle::Hash` | Transaction kernel ID (set after registration) |
| `FailureReason` | `TxFailureReason` | Failure code if status is `Failed` |
| `State` | (tx-type enum) | Per-transaction-type state machine step |

`TxStatus` values: `Pending`, `InProgress`, `Canceled`, `Completed`, `Failed`, `Registering`, `Confirming`.

`TxType` values:

| Value | Name |
|-------|------|
| 0 | `Simple` |
| 1 | `AtomicSwap` |
| 2–5 | `AssetIssue`, `AssetConsume`, `AssetReg`, `AssetUnreg` |
| 6 | `AssetInfo` |
| 7 | `PushTransaction` (Lelantus shield) |
| 8 | `PullTransaction` (Lelantus unshield) |
| 9–10 | `VoucherRequest`, `VoucherResponse` |
| 11 | `UnlinkFunds` |
| 12 | `Contract` |
| 13 | `DexSimpleSwap` |
| 14 | `InstantSbbsMessage` |

---

### `Assets` — Confidential Asset Registry

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `ID` | INTEGER NOT NULL PRIMARY KEY | `Asset::ID` |
| `Value` | BLOB | Currently issued amount (serialized `beam::Amount`) |
| `Owner` | BLOB NOT NULL | Owner public key (`PeerID`) |
| `LockHeight` | INTEGER | Block height at which the asset lock begins (for unregister) |
| `Metadata` | BLOB | Serialized asset metadata blob |
| `RefreshHeight` | INTEGER NOT NULL | Height of last on-chain confirmation |
| `IsOwned` | INTEGER | `1` if this wallet owns the asset key |
| `Deposit` | INTEGER | Registration deposit held in Groth |

**Indices:** unique index on `Owner`; secondary index on `RefreshHeight`.

---

### `notifications` — Wallet Notifications

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `ID` | BLOB NOT NULL PRIMARY KEY | Unique notification identifier |
| `type` | INTEGER | Notification category |
| `state` | INTEGER | Read/unread state |
| `createTime` | INTEGER | Creation Unix timestamp |
| `content` | BLOB NOT NULL | Serialized notification payload |

---

### `variables` / `PrivateVariables` — Key-Value Store

Both tables share the schema `(name TEXT UNIQUE, value BLOB)`.

- `variables` — public wallet state: sync height, BBS timestamps, shielded output count, max-privacy lock time limit, AID maximum, etc.
- `PrivateVariables` — encrypted master seed and other secret material. May reside in a separate database file for additional isolation.

---

### `States` — Block Header History (FlyClient)

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `Height` | INTEGER NOT NULL PRIMARY KEY | Block height |
| `State` | BLOB NOT NULL | Serialized `Block::SystemState::Full` |

Provides the header chain the FlyClient uses to verify Merkle proofs from the node. See [Fly Client Protocol](../node/Node-Fly-Client-Protocol.md) for how this table is consumed.

---

### `WalletMessages` / `IncomingWalletMessages` — Cold Wallet SBBS Staging

Used when the wallet operates in cold (air-gapped) mode. Messages are buffered here for manual import/export rather than sent directly over the network.

**Outgoing (`WalletMessages`):**

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `ID` | INTEGER AUTOINCREMENT | Internal identifier |
| `PeerID` | BLOB | Destination `WalletID` |
| `Message` | BLOB | Serialized `SetTxParameter` message |

**Incoming (`IncomingWalletMessages`):**

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `ID` | INTEGER AUTOINCREMENT | Internal identifier |
| `Channel` | INTEGER | BBS channel number |
| `Message` | BLOB | Raw encrypted BBS message |

---

### `LaserChannels` — Payment Channel State

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `chID` | BLOB PRIMARY KEY | 128-bit channel identifier |
| `myBbsID` | INTEGER | Own address key index (`OwnID`) |
| `trgWID` | BLOB | Counterparty `WalletID` |
| `State` | INTEGER | Channel state machine value |
| `fee` | INTEGER | Per-update routing fee in Groth |
| `Locktime` | INTEGER | Unilateral close lock height delta |
| `amountMy` | INTEGER | Opening balance (local side) in Groth |
| `amountTrg` | INTEGER | Opening balance (remote side) in Groth |
| `amountCurrentMy` | INTEGER | Current balance (local side) |
| `amountCurrentTrg` | INTEGER | Current balance (remote side) |
| `lockHeight` | INTEGER | Block height of the funding lock output |
| `bbsTimestamp` | INTEGER | Last SBBS activity timestamp |
| `data` | BLOB | Serialized commitment and revocation data |

`LaserAddresses` uses the same schema as `addresses` and stores the BBS addresses for each channel endpoint.

---

### `vouchers` — Shielded Voucher Store

Pre-generated `ShieldedTxo::Voucher` objects received from remote wallets. Each voucher is a one-time blinding key commitment that allows the sender to construct a shielded output addressed to the receiver without a live round-trip. Vouchers are consumed (deleted) by `grabVoucher()` when used by a send transaction.

---

### `events` — Blockchain Event Log

| Column | Notes |
|--------|-------|
| `Height` | Block height of the event |
| `Key` | Event type discriminant (BLOB) |
| `Body` | Serialized event data |

Indexed by `(Height, Key)`. Provides a local cache of processed blockchain events so that wallet rescans restart from the last confirmed height rather than genesis.

---

### `AppData` — BVM Application Key-Value Store

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `name` | BLOB NOT NULL | Application shader identifier |
| `key` | BLOB NOT NULL | Key within the application's namespace |
| `val` | BLOB NOT NULL | Stored value |

**Primary key:** `(name, key)`. Written by app shaders executed in the wallet process.

---

### `dex_offers` — DEX Order Book

Persists open DEX orders as serialized blobs. Each row has an `offerId` (primary key), the serialized offer body, and a flag indicating whether the order was created by this wallet.

---

### `exchangeRates` / `exchangeRatesHistory` — Price Data

Exchange rates fetched from news channels for fiat display in the wallet UI. Not used in transaction construction or consensus.

---

### `assetsVerification` — Asset Metadata Verification

| Column | SQLite Type | Notes |
|--------|-------------|-------|
| `assetID` | INTEGER | `Asset::ID` |
| `verified` | INTEGER | Verification flag |
| `icon` | TEXT | Icon URL or data URI |
| `color` | TEXT | Display color hex string |
| `updateTime` | INTEGER | Last update timestamp |

---

## Observer Pattern

`IWalletDbObserver` provides change callbacks for all collections:

```cpp
struct IWalletDbObserver {
    virtual void onCoinsChanged         (ChangeAction, const std::vector<Coin>&);
    virtual void onTransactionChanged   (ChangeAction, const std::vector<TxDescription>&);
    virtual void onSystemStateChanged   (const HeightHash&);
    virtual void onAddressChanged       (ChangeAction, const std::vector<WalletAddress>&);
    virtual void onShieldedCoinsChanged (ChangeAction, const std::vector<ShieldedCoin>&);
    virtual void onAssetChanged         (ChangeAction, Asset::ID);
    virtual void onIMSaved              (Timestamp, const WalletID&, const std::string&, bool isIncome);
};
```

`ChangeAction` values: `Added`, `Removed`, `Updated`, `Reset`.

---

## See Also

- [Wallet Architecture](Wallet-Architecture.md) — `Wallet` class, transaction lifecycle, SBBS transport
- [Core Transaction Elements](../core/Core-transaction-elements.md) — kernel and UTXO on-chain structures
- [Fly Client Protocol](../node/Node-Fly-Client-Protocol.md) — how `States` table powers SPV proofs
- [Secure Bulletin Board System (SBBS)](Wallet-SBBS.md) — messaging layer backed by `WalletMessages` tables
