# Transactions: Asset Swaps (DEX)

Beam's decentralized exchange (DEX) allows two parties to atomically swap any two [Confidential Assets](Transactions-Confidential-Assets.md) — or an asset against BEAM — without a trusted third party. The mechanism is built entirely on the wallet layer: orders are propagated over the [SBBS broadcast network](../wallet/Wallet-SBBS.md) and settled through an on-chain [interactive signing protocol](Transactions-Creation-Protocol.md).

---

## Overview

The DEX has three distinct components:

| Component | Responsibility |
|---|---|
| `DexOrder` | Data structure representing a standing offer |
| `DexBoard` | In-process order book: receives, stores, and publishes orders |
| `DexTransaction` | On-chain settlement transaction that atomically moves both assets |

---

## Order Structure

An order (`DexOrder`) is the advertised intent of a maker to exchange a fixed amount of one asset for a fixed amount of another.

```cpp
struct DexOrder {
    uint32_t   _version;          // currently kCurrentOfferVer = 1
    DexOrderID _orderID;          // 16-byte UUID
    WalletID   _sbbsID;           // SBBS address where maker listens for taker contact
    uint64_t   _sbbsKeyIDX;       // key index used to derive the SBBS key

    Asset::ID  _assetIdFirst;     // "first" leg asset ID (0 = BEAM)
    Asset::ID  _assetIdSecond;    // "second" leg asset ID
    std::string _assetSnameFirst;
    std::string _assetSnameSecond;
    Amount     _assetAmountFirst;
    Amount     _assetAmountSecond;

    Timestamp  _createTime;       // wall-clock seconds, set at construction
    Timestamp  _expireTime;       // createTime + expireMinutes * 60

    bool       _isCanceled;
    bool       _isAccepted;       // set once a taker begins settlement
};
```

`DexOrderID` is a randomly generated 16-byte UUID serialized as a hex string.

**Perspective fields.** From the perspective of the *maker* (the order creator), `_assetIdFirst`/`_assetAmountFirst` is what they send and `_assetIdSecond`/`_assetAmountSecond` is what they receive. For the *taker* (the acceptor) the sides are reversed. The helper methods `getSendAmount()` / `getReceiveAmount()` return the correct value for whichever party holds the order locally.

**Expiry.** `isExpired()` compares `_expireTime` against the current wall-clock timestamp. Expired orders are pruned from both the in-memory map and the database on the next chain state update.

**Serialization.** The order body is serialized with the standard Beam `SERIALIZE` macro. The signature is a `SignatureHandler` Schnorr signature over the serialized body, signed with the maker's SBBS private key. Receivers verify the signature against `_sbbsID.m_Pk` before accepting an order.

---

## Order Book: DexBoard

`DexBoard` is the in-process component that maintains the local view of the DEX order book. It implements three interfaces:

- `IBroadcastListener` — receives incoming broadcast messages and parses them as `DexOrder` objects.
- `ISimpleSwapHandler` — handles the wallet-level callbacks when a taker's DEX transaction arrives.
- `IWalletDbObserver` — listens for chain state changes to prune expired orders.

### Order Persistence

On startup `DexBoard` loads all previously seen orders from the wallet database (`IWalletDB::loadDexOffers`). Non-expired orders are re-populated into the in-memory `_orders` map and any associated SBBS listening subscriptions are restored.

### Order Propagation

Orders are propagated using the **broadcast router** over SBBS with content type `BroadcastContentType::DexOffers`. The BBS TTL for broadcast messages is 12 hours (`IBroadcastMsgGateway::m_bbsTimeWindow`). Any wallet subscribed to the broadcast channel receives and stores visible orders.

```
Maker wallet                      SBBS broadcast network               Taker wallet
    │                                      │                                 │
    │── publishOrder(DexOrder) ───────────>│                                 │
    │   [signed BroadcastMsg]              │── onMessage(BroadcastMsg) ─────>│
    │                                      │   [DexBoard.handleDexOrder]     │
```

**Publishing.** `DexBoard::publishOrder` serializes the order, signs it with the maker's SBBS private key, and calls `IBroadcastMsgGateway::sendMessage`. The same path is used to publish cancellation and acceptance state updates.

**Receiving.** `DexBoard::onMessage` parses the broadcast message, verifies the Schnorr signature, reconstructs the `DexOrder`, determines whether `_sbbsID` matches a locally derived key (to set `_isMine`), and calls `handleDexOrder`.

### Order Lifecycle

```
Created (maker publishes)
    └─> Seen by takers (propagated via SBBS broadcast)
            └─> Accepted: taker calls assets_swap_accept
                    └─> Maker marks _isAccepted = true, re-publishes
                    └─> DexTransaction created and submitted on-chain
            OR
            └─> Canceled: maker calls assets_swap_cancel
                    └─> _isCanceled = true, re-published
            OR
            └─> Expired: pruned on next chain state change
```

On cancellation, `DexBoard::cancelDexOrder` stops listening on the order's SBBS address, sets `_isCanceled = true`, and re-broadcasts the updated order so peers remove it from their books.

---

## Maker and Taker Roles

| Party | Action | API method |
|---|---|---|
| **Maker** | Creates and publishes an order | `assets_swap_create` |
| **Maker** | Cancels their standing order | `assets_swap_cancel` |
| **Taker** | Views available orders | `assets_swap_offers_list` |
| **Taker** | Accepts an order and begins settlement | `assets_swap_accept` |

**Maker** sets `expireMinutes`, `sendAmount`, `sendAsset`, `receiveAmount`, `receiveAsset`. The DEX board derives the SBBS address from the wallet's key tree using `_sbbsKeyIDX` and embeds it in the order. The maker's wallet then listens on that address for incoming `DexTransaction` parameters.

**Taker** selects an order from the board, calls `assets_swap_accept` with the order ID. This triggers `DexBoard::acceptIncomingDexSS`, which validates that the order is still active, marks it accepted, re-publishes it, and starts the settlement transaction toward the maker's SBBS address.

Only the maker can cancel their own order (`isMine()` check in `cancelDexOrder`).

---

## Settlement: DexTransaction

Settlement is a standard Beam interactive transaction (`TxType::DexSimpleSwap`) using the mutual signing protocol documented in [Transactions: Creation Protocol](Transactions-Creation-Protocol.md). Both parties contribute inputs for the asset they send and create one output for the asset they receive.

### Builder: DexSimpleSwapBuilder

`DexSimpleSwapBuilder` extends `MutualTxBuilder` with two extra fields:

```cpp
class DexSimpleSwapBuilder : public MutualTxBuilder {
    Asset::ID  m_ReceiveAssetID;   // asset to receive
    Amount     m_ReceiveAmount;    // amount to receive
    DexOrderID m_orderID;
};
```

`IsConventional()` returns `false`, meaning the standard "sender pays, receiver gets" amount encoding is overridden. Each party independently sets up its own balance sheet:

- Subtract `m_Amount` of `m_AssetID` (what you send).
- Add `m_ReceiveAmount` of `m_ReceiveAssetID` (what you receive), creating a new output UTXO.

### Fee Model

**The initiating party (taker) pays all fees**, always in BEAM. The fee covers the full transaction: both parties' inputs, two outputs (one per party), and the kernel. Before signing, the initiating party calls `CheckMinimumFee` with an extra `TxStats` count of one additional output (the peer's output) to ensure the fee is sufficient.

The receiving party (maker) pays no fee — it simply signs its portion of the transaction.

### Parameter Exchange

In `DexSimpleSwapBuilder::SendToPeer`, the taker sends swapped parameters to the maker so both sides compute the same transaction:

| Parameter sent by taker | What the maker sees it as |
|---|---|
| `Amount` = taker's receive amount | Maker's send amount |
| `AssetID` = taker's receive asset | Maker's send asset |
| `DexReceiveAmount` = taker's send amount | Maker's receive amount |
| `DexReceiveAsset` = taker's send asset | Maker's receive asset |
| `ExternalDexOrderID` | Used by `DexBoard` to look up the order |

The `ExternalDexOrderID` parameter is also how the `DexBoard::acceptIncomingDexSS` callback authenticates which order is being exercised. Only orders that are locally known, not expired, not canceled, and not yet accepted will be accepted.

### Transaction States

```cpp
enum class State : uint8_t {
    Initial,
    Registration,          // taker has submitted the tx to the node
    KernelConfirmation,    // waiting for the kernel to appear in a block
};
```

The transaction enters `IsInSafety()` once in `KernelConfirmation` state; at that point, both UTXO moves are committed on-chain and the swap is irreversible.

### Validation Constraints

- Both asset IDs must be different (swapping an asset for itself is rejected).
- Self-transactions (same wallet on both sides) are rejected.
- The `IsSelfTx` flag must be present and false.

---

## Database Storage

Orders are persisted by `IWalletDB` in the `dex_offers` table. Each row stores the serialized order body and a boolean indicating ownership (`isMine`). Expired orders are dropped on the next `onSystemStateChanged` callback.

DEX transactions are stored in the standard transaction tables alongside all other wallet transactions, distinguished by `TxType::DexSimpleSwap`.

---

## Relationship to Atomic Swaps

[Atomic Swaps](Transactions-Atomic-Swaps.md) in Beam use HTLC kernels to swap BEAM against Bitcoin or other external chains. The DEX described here is a different, purely on-chain mechanism for swapping two Beam Confidential Assets within the same Beam transaction. No time-lock or hash reveal is required because both sides of the exchange are committed in a single kernel.

---

## API Reference

The DEX is exposed through the `v7_2` Wallet API (requires `BEAM_ASSET_SWAP_SUPPORT` build flag):

| Method | Access | Description |
|---|---|---|
| `assets_swap_offers_list` | Read | Return all currently known non-expired orders |
| `assets_swap_create` | Write | Create and publish a new maker order |
| `assets_swap_cancel` | Write | Cancel a maker order by ID |
| `assets_swap_accept` | Write | Accept a taker order, start settlement transaction |

See the [Wallet API](../api/README.md) reference for full request/response schemas.
