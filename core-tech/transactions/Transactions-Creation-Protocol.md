# Transaction Creation Protocol

Creating a Beam transaction is an **interactive** process: the sender and receiver must exchange cryptographic material before a valid transaction can be assembled. This page documents the wire protocol, the internal state machine, and the cryptographic mechanics of partial-signature aggregation.

Related pages: [Core-Transaction-Elements](../core/Core-transaction-elements.md), [Wallet-SBBS](../wallet/Wallet-SBBS.md), [Wallet-Addresses-And-Key-Derivation](../wallet/Wallet-Addresses-And-Key-Derivation.md), [Transactions-Confidential-Assets](Transactions-Confidential-Assets.md)

---

## Overview

A Beam transaction satisfies the Mimblewimble balance equation:

```
sum(output commitments) − sum(input commitments) = kernel excess + fee·H
```

No single party holds all the blinding factors, so both parties must contribute partial signatures to produce the joint Schnorr signature on the kernel. This requires at least one round trip between the two wallets.

---

## Transaction Types

| Type | Parties | Description |
|---|---|---|
| **Mutual (interactive)** | Sender + Receiver | Standard BEAM or CA transfer; two-party signing |
| **Self / Split** | Single wallet | Coin split, change consolidation; no peer needed |
| **Max-Privacy** | Sender + Receiver (via voucher) | Receiver pre-issues Lelantus shielded vouchers; sender uses them without a synchronous reply |

The `SimpleTransaction` class dispatches on this distinction in `UpdateImpl()`: if the peer address resolves to an own address (`IsSelfTx()`), a `SimpleTxBuilder` is used; otherwise a `MutualTxBuilder` handles the interactive flow.

---

## SetTxParameter Message

All wallet-to-wallet communication uses the `SetTxParameter` message:

```cpp
struct SetTxParameter {
    WalletID m_From;    // SBBS address of the sender for reply-to
    TxID     m_TxID;    // 16-byte random transaction identifier (set by initiator)
    TxType   m_Type;    // Simple, AtomicSwap, Assets, etc.
    std::vector<std::pair<TxParameterID, ByteBuffer>> m_Parameters;
};
```

**Parameter ID space:**

| Range | Access | Meaning |
|---|---|---|
| `[0, 127]` | Public (peer-settable) | Once written, cannot be overridden by peer |
| `[128, 255]` | Private | Internal to each wallet; never transmitted |

This prevents a malicious peer from overwriting critical values (such as amount or blinding factors) after the first message is received.

The message itself is transport-agnostic. It may be delivered via [SBBS](../wallet/Wallet-SBBS.md), a direct wallet-to-wallet TCP connection, or the offline token mechanism.

---

## Mutual Transaction: Round Structure

### Cryptographic Setup

Both parties maintain:
- A **blinding excess** scalar `ke` derived from their coins' blinding factors
- A **nonce** scalar `r` chosen freshly per transaction
- A **public excess** `X = ke·G`
- A **public nonce** `R = r·G`

The aggregate kernel excess is `X = Xs + Xr` and aggregate nonce is `R = Rs + Rr`. The Schnorr challenge is:

```
e = H(R | kernel_message)   // kernel_message encodes fee, min/max height, X
```

Each party computes a partial signature scalar:

```
s_i = r_i + e · ke_i
```

The final kernel signature is `s = s_s + s_r`, verified against the aggregate nonce and excess.

---

### Round 1 — Sender Invitation

The sender (`Wallet A`) initiates by selecting coins, computing its public excess and nonce, then sending:

```
SetTxParameter {
    m_From: <Wallet A SBBS address>
    m_TxID: <newly generated 16-byte ID>
    m_Type: TxType::Simple
    Parameters: [
        (Amount,              amount),
        (Fee,                 fee),
        (MinHeight,           minHeight),
        (MaxHeight,           maxHeight),
        (IsSender,            false),          // receiver flag for Wallet B
        (PeerProtoVersion,    version),
        (PeerPublicExcess,    Xs),             // sender's public excess
        (PeerPublicNonce,     Rs),             // sender's public nonce
        (PeerResponseHeight,  responseDeadline) // height limit for response
    ]
}
```

- `MinHeight` / `MaxHeight`: block range during which the kernel is valid. The sender sets `MaxHeight` to the current height plus the transaction **lifetime** (`kDefaultTxLifetime = 120 blocks ≈ 2 hours`).
- `PeerResponseHeight`: if the receiver does not respond before this chain height, the sender marks the transaction as expired.
- `IsSender = false` tells the recipient it is the receiver in this transaction.

**Sender internal state after Round 1:** `SndHalf → SndHalfSent`

---

### Round 2 — Receiver Confirmation

`Wallet B` creates output UTXOs for the received amount, computes its blinding excess and nonce, computes its partial signature, then replies:

```
SetTxParameter {
    m_From: <Wallet B SBBS address>
    m_TxID: <same TxID>
    m_Type: TxType::Simple
    Parameters: [
        (PeerProtoVersion,    version),
        (PeerPublicExcess,    Xr),             // receiver's public excess
        (PeerPublicNonce,     Rr),             // receiver's public nonce
        (PeerSignature,       s_r),            // receiver's partial Schnorr scalar
        (PeerOutputs,         [output UTXOs]),
        (PeerInputs,          [inputs, if any]),
        (PeerOffset,          offset_r),       // receiver's blinding offset
        (PaymentConfirmation, sig)             // optional payment proof signature
    ]
}
```

- `PeerOutputs`: the newly created Pedersen commitments and range proofs for the amount.
- `PeerOffset`: a blinding offset component (`offset_r`) that keeps the transaction balanced without revealing individual blinding factors.
- `PaymentConfirmation`: an optional Schnorr signature over the kernel ID and sender's public key, produced by the receiver's key keeper. Enables the sender to prove payment to a third party.

**Receiver internal state:** `None → RcvHalf → RcvFullHalfSig → RcvFullHalfSigSent`

---

### Round 3 — Sender Finalization (No Reply)

Upon receiving Round 2, the sender:

1. **Aggregates public values:**
   ```
   X = Xs + Xr
   R = Rs + Rr
   ```

2. **Recomputes the Schnorr challenge** `e = H(R | kernel_message)`.

3. **Verifies the receiver's partial signature** `s_r`: checks `s_r·G == Rr + e·Xr`.

4. **Produces its own partial signature** `s_s = rs + e · ke_s`.

5. **Finalizes the kernel**: `m_Commitment = X`, `m_Signature = (R, s_s + s_r)`.

6. **Assembles the full transaction**: combines sender inputs/change outputs, receiver outputs, both offsets (`offset = offset_s + offset_r`), and the kernel.

7. **Broadcasts** to the node via `register_tx()`.

There is no Round 3 message to the receiver; the receiver's safety point (`IsInSafety()`) is reached at `KernelConfirmation` state — meaning once the receiver has sent its partial signature it can no longer cancel.

**Sender internal state:** `SndHalfSent → SndFullHalfSig → SndFull → Registration → KernelConfirmation → Completed`

---

## State Machine Summary

```
Sender (MutualTxBuilder::Status):
  None ──[sign initial]──► SndHalf
  SndHalf ──[send to peer]──► SndHalfSent
  SndHalfSent ──[peer response received]──► SndFullHalfSig
  SndFullHalfSig ──[sign final]──► SndFull
  SndFull ──[broadcast]──► Registration
  Registration ──[node confirms]──► KernelConfirmation → Completed

Receiver (MutualTxBuilder::Status):
  None ──[peer nonce/excess received]──► RcvHalf
  RcvHalf ──[sign]──► RcvFullHalfSig
  RcvFullHalfSig ──[send to peer]──► RcvFullHalfSigSent → Completed
```

```
SimpleTransaction::State (stored in DB):
  Initial → Invitation → PeerConfirmation → Registration
          → KernelConfirmation → OutputsConfirmation → Completed
```

---

## Partial Signature Aggregation

The blinding excess and offset separation ensures neither party can reconstruct the other's private key:

```
Transaction offset = offset_s + offset_r   (sum of both scalar offsets)
Kernel excess     = X = ke_s·G + ke_r·G    (aggregated in group, not field)
```

The key keeper signs via `IPrivateKeyKeeper2::Method::TxMutual`, which includes:
- The method's `m_kOffset` — the caller's offset contribution
- `m_PaymentProofSignature` — optional proof of receipt signed by the receiver

The `AddOffset()` call in `BaseTxBuilder` accumulates scalar offsets; the final `m_Offset` field in the `Transaction` struct holds the sum.

---

## Self-Transaction (Split)

When `IsSelfTx()` returns true (peer address is a self-owned address), no interactive round-trip occurs:

- Uses `SimpleTxBuilder` directly (not `MutualTxBuilder`).
- Calls `SignSplit()` which dispatches `IPrivateKeyKeeper2::Method::SignSplit` to the key keeper.
- The key keeper produces a fully signed kernel in a single call.
- Used for: coin splitting, fee payment against self, offline transaction preparation.

---

## Transaction Lifetime and Expiry

The **lifetime** parameter (`Lifetime`, default `120` blocks ≈ 2 hours at 1 min/block) governs how long a kernel remains valid:

| Parameter | Set by | Meaning |
|---|---|---|
| `MinHeight` | Sender | Kernel not valid below this chain height |
| `MaxHeight` | Sender / negotiated | Kernel invalid above this height |
| `Lifetime` | Sender | Used to compute `MaxHeight = tip + Lifetime` |
| `PeerResponseHeight` | Sender | If receiver doesn't respond by this height, tx expires |

`CheckExpired()` runs after each `UpdateImpl()` call:
1. If the current tip exceeds `MaxHeight` and the transaction is not yet registered, `OnFailed(TxFailureReason::TransactionExpired)` is called.
2. If a `KernelUnconfirmedHeight` exceeds `MaxHeight`, the transaction is also failed.

The receiver validates that `MaxHeight` is not excessively high (`MaxHeightIsUnacceptable`) to prevent senders from creating very long-lived kernels.

---

## Failure Handling

Either party may abort by sending a `SetTxParameter` carrying only:

```
Parameters: [
    (FailureReason, <TxFailureReason code>)
]
```

Selected `TxFailureReason` codes relevant to the protocol:

| Code | Name | Condition |
|---|---|---|
| 1 | `Canceled` | Explicit user cancellation |
| 2 | `InvalidPeerSignature` | Receiver's partial signature fails verification |
| 6 | `FailedToSendParameters` | SBBS/transport failure |
| 10 | `TransactionExpired` | `MaxHeight` exceeded before broadcast |
| 12 | `MaxHeightIsUnacceptable` | Receiver rejects proposed `MaxHeight` |
| 11 | `NoPaymentProof` | Sender required proof but receiver did not provide it |
| 44 | `NoVoucher` | Max-privacy tx: sender could not obtain a shielded voucher |

**Cancellation window:** The receiver can safely cancel before sending Round 2. Once Round 2 is sent, the sender may complete the transaction without further interaction; the receiver cannot prevent it.

### Wire Message Examples

**Round 1 — Sender invitation (SBBS encoded):**
```javascript
SetTxParameter {
    m_From: "XXXXXX",          // Wallet A SBBS response address
    m_TxID: 651798,            // newly generated random ID
    m_Type: TxType::Simple,
    params: [
        { TxParameterID::Amount,           amount },
        { TxParameterID::Fee,              fee },
        { TxParameterID::MinHeight,        minHeight },
        { TxParameterID::MaxHeight,        maxHeight },
        { TxParameterID::IsSender,         false },
        { TxParameterID::PeerProtoVersion, version },   // current: 1
        { TxParameterID::PeerPublicExcess, publicExcess },
        { TxParameterID::PeerPublicNonce,  publicNonce }
    ]
}
```

**Round 2 — Receiver confirmation:**
```javascript
SetTxParameter {
    m_From: "YYYYYY",          // Wallet B SBBS response address
    m_TxID: 651798,
    m_Type: TxType::Simple,
    params: [
        { TxParameterID::PeerProtoVersion, version },
        { TxParameterID::PeerPublicExcess, peerPublicExcess },
        { TxParameterID::PeerSignature,    receiversPartialSignature },
        { TxParameterID::PeerPublicNonce,  publicNonce },
        { TxParameterID::PeerOutputs,      outputs },
        { TxParameterID::PeerOffset,       offset }
    ]
}
```

**Cancellation message:**
```javascript
SetTxParameter {
    m_From: "ZZZZZZ",
    m_TxID: 651798,
    m_Type: TxType::Simple,
    params: [
        { TxParameterID::FailureReason, reason }   // 32-bit failure code
    ]
}
```

---

## Max-Privacy Variant

Max-privacy transactions use Lelantus shielded outputs. Instead of an interactive round-trip, the receiver pre-generates **shielded vouchers** (one-time encrypted output descriptors) and publishes them via SBBS. The sender:

1. Requests vouchers from the receiver's wallet via `RequestVouchersFrom()`.
2. The `VoucherManager` caches vouchers per peer address (threshold: 5 vouchers).
3. The sender picks a voucher (`get_UniqueVoucher()`), constructs a shielded output without a live reply from the receiver.
4. The receiver detects the incoming shielded coin on its next wallet sync.

This eliminates the synchronous requirement but adds a pre-negotiation phase for voucher exchange. See [Transactions-Lelantus-Shielded-Pool](Transactions-Lelantus-Shielded-Pool.md) for shielded proof construction details.

---

## Negotiator Framework (Lightning Channels)

The `Negotiator` namespace (`core/negotiator.h`) provides a generic multi-round negotiation framework used by Lightning channel operations. It is **not** used by the standard simple transaction path.

Key classes:

| Class | Purpose |
|---|---|
| `Negotiator::IBase` | Abstract negotiator with `Update()` driver and `m_Pos` sequencing |
| `Negotiator::Multisig` | Negotiate a jointly-owned multi-sig UTXO |
| `Negotiator::MultiTx` | Build a transaction with arbitrary inputs/outputs and optional multi-sig endings |
| `Negotiator::WithdrawTx` | Composed of three `MultiTx`/`Multisig` sub-negotiations (for Lightning withdrawal) |
| `Negotiator::ChannelOpen` | Opens a Lightning channel (funding + initial withdrawal paths) |
| `Negotiator::ChannelUpdate` | Updates an open channel, revoking the previous state |

The `Gateway::IBase` interface abstracts message delivery; `Storage::IBase` abstracts persistence. The `Router` inner class remaps storage and gateway namespaces for sub-negotiations inside composed protocols.

For full channel lifecycle documentation see [Transactions-Laser-Channels](Transactions-Laser-Channels.md).

---

## `BaseTransaction` / `BaseTxBuilder` Extension Pattern

Custom transaction types extend `BaseTransaction` and provide a `Creator`:

```cpp
class MyTransaction : public BaseTransaction {
    void UpdateImpl() override;     // state machine entry point
    bool IsTxParameterExternalSettable(TxParameterID, SubTxID) const override;
};

class MyTransaction::Creator : public BaseTransaction::Creator {
    BaseTransaction::Ptr Create(const TxContext&) override;
    TxParameters CheckAndCompleteParameters(const TxParameters&) override;
};
```

`BaseTxBuilder` manages:
- Coin selection results (`m_Coins.m_Input`, `m_Output`, `m_InputShielded`)
- Kernel pointer (`m_pKrn`) and height range (`m_Height`)
- Async in/out generation (`GenerateInOuts()`) and signing (`SignTx()`)
- Final assembly and verification (`FinalyzeTx()`, `VerifyTx()`)

`MutualTxBuilder` adds the peer-exchange state machine on top of `SimpleTxBuilder`, coordinating the `SignTxSender()` / `SignTxReceiver()` switch and the `SendToPeer()` callback (implemented per concrete transaction class).
