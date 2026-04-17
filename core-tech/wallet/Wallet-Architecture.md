# Wallet Architecture

The Beam wallet engine lives in `wallet/core/` and is responsible for four concerns:

1. **UTXO tracking** — watching the blockchain for events that affect coins owned by this wallet
2. **Transaction lifecycle** — running the multi-round signing state machines for each in-flight transaction
3. **Wallet-to-wallet messaging** — routing encrypted `SetTxParameter` packets between parties via SBBS or a direct connection
4. **Key management** — abstracting access to secret key material through `IPrivateKeyKeeper2`

---

## Wallet Modes

`IWalletDB` supports three operational modes depending on which keys are available:

| Mode | Keys Present | Capabilities |
|------|-------------|--------------|
| Full (seed) | Master KDF, Owner KDF, SBBS KDF | All transaction types, SBBS communication |
| KeyKeeper (trustless) | Owner KDF + KeyKeeper device | Simple transactions only; master key stays on device |
| Read-only | Owner KDF only | Coin detection via event scanning; no tx creation, no SBBS |

---

## The `Wallet` Class

`Wallet` (`wallet/core/wallet.h`) is the central engine. It inherits from three interfaces:

- `proto::FlyClient` — receives chain tip updates and async responses from the node
- `INegotiatorGateway` — dispatched by active `BaseTransaction` instances to request confirmations and register transactions
- `IWalletMessageConsumer` — receives decrypted SBBS packets from registered `IWalletMessageEndpoint` instances

### Initialization

```cpp
Wallet(IWalletDB::Ptr walletDB,
       TxCompletedAction&& action = {},
       UpdateCompletedAction&& updateCompleted = {});

void SetNodeEndpoint(proto::FlyClient::INetwork::Ptr);
void AddMessageEndpoint(IWalletMessageEndpoint::Ptr);
```

Multiple message endpoints can be registered simultaneously (e.g., SBBS and a cold-wallet file channel).

---

## Node Synchronization

When a new chain tip arrives, `Wallet::OnNewTip()` drives the sync loop:

1. `RequestEvents()` — fetches UTXO ownership events from the node since the last processed height
2. `ProcessEventUtxo()` / `ProcessEventShieldedUtxo()` — updates `Coin` / `ShieldedCoin` records in the DB
3. `ProcessEventAsset()` — updates `WalletAsset` records
4. `CheckSyncDone()` — once all pending requests complete, dequeues `m_SyncActionsQueue` and resumes blocked transactions

Wallets that opt into **body download** (mobile node mode, enabled via `EnableBodyRequests(true)`) additionally call `RequestBodies()` to receive full block bodies so they can scan coins independently of the node's event index.

### Sync-Gated Actions

```cpp
void DoInSyncedWallet(OnSyncAction&& action);
```

Appends an action to `m_SyncActionsQueue`. If the wallet is already in sync the action fires immediately; otherwise it waits until `CheckSyncDone()` drains the queue. `StartTransaction` uses this internally so transactions never begin against a stale chain state.

---

## Transaction Lifecycle

### Registration

Each transaction type is registered at startup by its factory:

```cpp
wallet.RegisterTransactionType(TxType::Simple,          creator_simple);
wallet.RegisterTransactionType(TxType::PushTransaction, creator_push);
// ...
```

`TxType` enum values (defined in `wallet/core/common.h`):

| Value | Name | Description |
|-------|------|-------------|
| 0 | `Simple` | Standard interactive BEAM transfer |
| 1 | `AtomicSwap` | Cross-chain atomic swap |
| 2 | `AssetIssue` | Issue Confidential Asset tokens |
| 3 | `AssetConsume` | Burn Confidential Asset tokens |
| 4 | `AssetReg` | Register a new Confidential Asset |
| 5 | `AssetUnreg` | Unregister a Confidential Asset |
| 6 | `AssetInfo` | Fetch asset metadata from chain |
| 7 | `PushTransaction` | Lelantus shield (UTXO → shielded pool) |
| 8 | `PullTransaction` | Lelantus unshield (shielded pool → UTXO) |
| 9 | `VoucherRequest` | Request shielded vouchers from peer |
| 10 | `VoucherResponse` | Deliver shielded vouchers to peer |
| 11 | `UnlinkFunds` | Shielded coin re-randomization |
| 12 | `Contract` | BVM contract invocation |
| 13 | `DexSimpleSwap` | On-chain DEX asset swap |
| 14 | `InstantSbbsMessage` | Instant messaging over SBBS |

### Active Transaction Map

```cpp
std::map<TxID, BaseTransaction::Ptr> m_ActiveTransactions;
```

`StartTransaction(TxParameters)` constructs the appropriate `BaseTransaction` subtype via the registered `Creator`, inserts it into `m_ActiveTransactions`, and calls `ProcessTransaction`. If the wallet is not yet in sync the start is deferred via `DoInSyncedWallet`.

### Update Loop

The wallet drives each transaction by calling `tx->Update()` when:
- A new block tip arrives (`OnNewTip`)
- A node request completes (kernel confirmation, UTXO proof, etc.)
- An incoming SBBS message is directed at this transaction
- The transaction fires its own async update event (`UpdateAsync`)

`m_NextTipTransactionToUpdate` holds transactions that explicitly request a retry on the next block.

### Completion and Cleanup

On success: `on_tx_completed(txID)` → fires `m_TxCompletedAction` → removes the transaction from `m_ActiveTransactions`. On failure: `on_tx_failed(txID)` marks the DB record as `Failed`.

---

## SBBS Network Layer

The default transport for interactive transactions is the Secure Bulletin Board System. The class hierarchy is:

```
IWalletMessageEndpoint
  └── BaseMessageEndpoint        (address book + ECDH decryption)
        └── WalletNetworkViaBbs  (SBBS channel subscriptions, DB observer)
                uses BbsProcessor       (outgoing message queue, subscribe/unsubscribe)
                uses TimestampHolder    (per-channel last-seen timestamps for dedup)
```

### `BaseMessageEndpoint`

Maintains a set of `Addr` objects — one per watched `WalletID`. Each `Addr` holds:
- The private SBBS key (`ECC::Scalar::Native m_sk`)
- The corresponding BBS channel number
- An expiration timestamp derived from `WalletAddress::m_duration`

Incoming `proto::BbsMsg` packets are ECDH-decrypted and forwarded to `IWalletMessageConsumer::OnWalletMessage`, which dispatches to the relevant active transaction.

### `WalletNetworkViaBbs`

Subscribes to BBS channels when addresses are added (`OnChannelAdded`), unsubscribes when addresses expire or are deleted, and implements `IWalletDbObserver` to react to address changes automatically.

### Channel Derivation

An address's BBS channel is derived from the lower bits of its `WalletID`. The node maintains per-channel message queues and delivers messages to subscribed fly clients. `TimestampHolder` records the latest `proto::BbsMsg::m_TimePosted` per channel to skip already-seen messages on reconnect.

---

## `BaseTransaction` Extension Pattern

All transaction types derive from `BaseTransaction` (`wallet/core/base_transaction.h`).

### `ITransaction` Interface

```cpp
struct ITransaction {
    virtual TxType GetType()          const = 0;
    virtual void   Update()                 = 0;
    virtual bool   CanCancel()        const = 0;
    virtual void   Cancel()                 = 0;
    virtual bool   Rollback(Height h)       = 0;
    virtual bool   IsInSafety()       const = 0;
};
```

`IsInSafety()` returns true once negotiation is complete and all data has been sent to the node — the point after which cancellation is no longer possible without blockchain intervention.

### What `BaseTransaction` Provides

**Parameter storage** — all state is written to the `txparams` SQLite table via typed helpers:

```cpp
template<typename T>
bool GetParameter(TxParameterID paramID, T& value) const;

template<typename T>
bool SetParameter(TxParameterID paramID, const T& value);
```

Parameters are keyed by `(TxID, SubTxID, TxParameterID)`. This means the entire negotiation transcript is persisted and survives a wallet restart.

**State machine helpers**:

```cpp
template<typename EnumT> void SetState(EnumT state);
template<typename EnumT> EnumT GetState() const;
```

Both read/write `TxParameterID::State` for the current `SubTxID`.

**Gateway calls** — `ConfirmKernel()`, `CompleteTx()`, `OnFailed()`, `SendTxParameters()` delegate to `INegotiatorGateway` (implemented by `Wallet`).

### Adding a New Transaction Type

1. Define a new `TxType` enum value.
2. Subclass `BaseTransaction`, implement `UpdateImpl()` as the per-step state machine body. Read peer parameters via `GetParameter`, write local state via `SetParameter`, call `SendTxParameters` to send to peer, and call `CompleteTx` or `OnFailed` at the end.
3. Subclass `BaseTransaction::Creator`, implement `Create(TxContext)` and optionally `CheckAndCompleteParameters`.
4. Register at wallet startup: `wallet.RegisterTransactionType(TxType::MyType, creator)`.

The `TxContext` bundles the `Wallet&`, the `INegotiatorGateway&`, the `TxID`, and the `SubTxID` (sub-phases are used in atomic swaps: lock tx and redeem tx each have their own `SubTxID`).

---

## Address Types and Token Encoding

The wallet supports five address token types, distinguished by `TokenType` (`wallet/core/wallet_db.h`):

| `TokenType` | Description | Transport | Receiver online? |
|-------------|-------------|-----------|-----------------|
| `RegularOldStyle` | Legacy SBBS address (WalletID only) | SBBS | Yes |
| `RegularNewStyle` | Modern SBBS address with endpoint identity | SBBS | Yes |
| `Offline` | Pre-generated vouchers embedded in token | Direct push | No |
| `MaxPrivacy` | Lelantus push; voucher exchange via SBBS | SBBS + Lelantus | For voucher exchange |
| `Public` | Single offline voucher encoded in permanent token | Direct push | No |

A `WalletAddress` record stores:

| Field | Description |
|-------|-------------|
| `m_BbsAddr` | `WalletID` — BBS channel address derived from `m_OwnID` |
| `m_Endpoint` | `PeerID` — public key for direct endpoint identification |
| `m_Token` | Serialized `TxParameters` encoding the address type and embedded vouchers |
| `m_OwnID` | Key derivation index; `0` for contacts |
| `m_duration` | Expiry window in seconds; `0` = never expires |

Token generation functions (from `wallet/core/wallet_db.h`):

```cpp
std::string GenerateRegularNewToken (const WalletAddress&, Amount, Asset::ID, const std::string& clientVersion);
std::string GenerateOfflineToken    (const WalletAddress&, const IWalletDB&, Amount, Asset::ID,
                                     const std::string& clientVersion, uint32_t offlineCount = 10);
std::string GenerateMaxPrivacyToken (const WalletAddress&, const IWalletDB&, Amount, Asset::ID, const std::string& clientVersion);
std::string GeneratePublicToken     (const WalletAddress&, const IWalletDB&, const std::string& clientVersion);
```

For address encoding and key derivation details see [Wallet Addresses and Key Derivation](Wallet-Addresses-And-Key-Derivation).

---

## Voucher Management

Max-privacy and offline sends embed `ShieldedTxo::Voucher` objects — one-time commitment blinding keys generated by the receiver's key keeper. The `VoucherManager` struct inside `Wallet` handles the async cycle:

1. Sender calls `RequestVouchersFrom(peerID, myID, count)` → sends a `VoucherRequest` SBBS message
2. Receiver's wallet responds with a `VoucherResponse` containing freshly generated vouchers
3. `OnVouchersFrom()` stores vouchers in the DB (`vouchers` table) and unblocks waiting transactions

A retry timer re-sends stalled requests. If no vouchers arrive within the deadline, the transaction fails with `TxFailureReason::CannotGetVouchers`.

---

## User-Facing Status Labels

The following status strings are shown to users in the wallet UI and transaction history. They map to internal state machine transitions in `BaseTransaction` and `SimpleTransaction`.

### Transaction Lifecycle Status Strings

| Status string | Meaning |
|---|---|
| `"Waiting for network sync to complete"` | Transaction cannot start before the wallet has finished syncing with the node |
| `"Waiting for Sender"` / `"Waiting for Receiver"` | The other party needs to come online to continue negotiating the transaction |
| `"Handshaking"` | Both wallets are actively negotiating transaction details (exchanging parameters) |
| `"Syncing with blockchain"` | Transaction has been submitted to a node; waiting for inclusion in a block |
| `"Sending"` / `"Receiving"` | Transaction is in a block and propagating; awaiting sufficient confirmations |
| `"Sent"` / `"Received"` | Transaction is confirmed and complete |
| `"Cancelled"` | Transaction was cancelled by the sender (or by a chain rollback) |
| `"Expired"` | Transaction lifetime elapsed before completion (no block confirmation within `MaxHeight`) |
| `"Failed"` | Transaction failed; accompanied by a user-readable reason code |

For Atomic Swap transactions the short status uses: `"In progress"` → `"Completed"` / `"Cancelled"` / `"Failed"` / `"Expired"`.

### UTXO Status Display

| `Coin::Status` | Display name | Notes |
|---|---|---|
| `Available` | Available | Spendable immediately |
| `Maturing` | Reserved till block `<height>` | Coinbase or treasury output; maturity not yet reached |
| `Incoming` | Incoming (draft) / Incoming | UTXO created by an in-progress incoming transaction |
| `Outgoing` | Outgoing (locked) | UTXO selected as input for an in-progress outgoing transaction |
| `Spent` | Spent | UTXO was consumed by a confirmed transaction |
| `Unavailable` | Unavailable | UTXO was rolled back (e.g., mining reward rollback) |

Coin types displayed alongside status: `Regular`, `Regular (change)`, `Transaction fee`, `Coinbase`, `Treasury`.

---

## See Also

- [Wallet Database Schema](Wallet-Database-Schema) — SQLite table definitions
- [Fly Client Protocol](Node-Fly-Client-Protocol) — the node communication layer `Wallet` extends
- [Secure Bulletin Board System (SBBS)](Secure-bulletin-board-system-(SBBS)) — encrypted wallet-to-wallet messaging
- [Core Transaction Elements](Core-transaction-elements) — kernel and UTXO primitives used by transactions
