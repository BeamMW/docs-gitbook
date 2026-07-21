# Laser Channels (Payment Channels / Lightning)

Laser Beam is Beam's payment-channel layer — an off-chain protocol that lets two parties lock funds on-chain once and then exchange value instantly, without fees, and without broadcasting every transfer. The on-chain footprint is one opening transaction and one closing transaction regardless of how many off-chain payments occur in between.

---

## Overview

A channel has three phases:

1. **Open** — both parties fund a shared multi-signed UTXO (`msig0`) via an on-chain transaction.
2. **Update** (repeated) — parties negotiate a new fund partition off-chain and generate a fresh set of withdrawal transactions, revoking the old ones.
3. **Close** — the channel is settled on-chain, either cooperatively (graceful) or unilaterally (one-side).

Because Mimblewimble has no scripts, all logic is implemented through *scriptless scripts*: multi-party Schnorr signing, relative timelocks encoded in kernel fields, and blinding-factor revelation as a revocation mechanism.

---

## Key Constants

Defined in `core/lightning.h`:

| Constant | Default | Meaning |
|---|---|---|
| `kDefaultRevisionMaxLifeTime` | 20,160 blocks (~14 days) | Maximum lifetime of a single withdrawal revision |
| `kDefaultLockTime` | 1,440 blocks (~1 day) | Relative timelock between Phase 1 and Phase 2 of a withdrawal |
| `kDefaultPostLockReserve` | 1,440 blocks | Extra safety margin before a revision expires |
| `kDefaultOpenTxDh` | 120 blocks (~2 hours) | Max height range for the opening transaction kernel |

---

## Building Blocks

### Multi-Signed UTXO (MultiSig)

A `MultiSig` UTXO is one whose blinding factor is the sum of each party's private blinding contribution. It is cryptographically indistinguishable from a regular UTXO. Creating or spending it requires both parties to cooperate on:

- Building the commitment (`ECC::Point m_Comm`)
- Generating a valid Bulletproof range proof (split into parts exchanged during negotiation)
- Signing any kernel that spends it

In code: `Negotiator::Multisig` (in `core/negotiator.h`) handles the two-party creation of a MultiSig UTXO in 1 round-trip.

### Relative Timelock

Each `TxKernelStd` can carry an optional relative lock: a reference to another kernel ID and a minimum height difference. The node validates that the referenced kernel exists in the blockchain and has the required maturity before the new kernel is accepted.

This is used in the two-phase withdrawal: Phase 2 (the output-creating transaction) becomes valid only after Phase 1 (the intermediate `msigN` transaction) is confirmed and `m_hLockTime` blocks have passed.

### Withdrawal (Refund) Procedure

A single withdrawal path for party X consists of two pre-signed transactions:

```
Tx1: msig0   →  msigN.X          (Phase 1, kept private to X)
Tx2: msigN.X →  Outputs.N        (Phase 2, timelocked relative to Tx1, shared with peer)
```

Both parties hold their own private Phase 1 transaction but share each other's Phase 2 transaction. This asymmetry matters for revocation: if X's `msigN.X` appears on-chain, the peer can immediately claim the UTXO because it holds `Tx2` for the peer's matching path.

In code: `Negotiator::WithdrawTx` composes three sub-negotiators (`Multisig m_MSig`, `MultiTx m_Tx1`, `MultiTx m_Tx2`).

### Revocation

To revoke revision N, a party **reveals the blinding factor scalar** it contributed to `msigN.X`. Once revealed:

- The peer can reconstruct the full blinding factor of `msigN.X`.
- If X subsequently broadcasts Tx1 (the revoked path), `msigN.X` becomes visible on-chain.
- The peer immediately broadcasts a *punishment transaction* spending `msigN.X` to itself, claiming all funds — before the timelock expires.

The timelock window (`kDefaultLockTime`) gives the honest party time to detect and respond to a cheating attempt. The `Channel::CreatePunishmentTx()` method handles this.

---

## Channel State Machine

`Lightning::Channel::State::Enum` in `core/lightning.h`:

| State | Meaning |
|---|---|
| `None` | No channel data allocated |
| `Opening0` | Negotiating; no-return barrier not yet crossed (safe to abandon) |
| `Opening1` | Negotiating; barrier crossed — cannot safely forget |
| `Opening2` | Negotiation complete; waiting for on-chain confirmation |
| `OpenFailed` | Opening transaction not confirmed before `hrLimit.m_Max` |
| `Open` | Channel is live |
| `Updating` | Negotiating a new revision |
| `Closing1` | Unilateral close initiated |
| `Closing2` | Phase 1 withdrawal confirmed on-chain; waiting for timelock |
| `Closed` | Phase 2 withdrawal confirmed |
| `Expired` | Channel revision has exceeded its max lifetime |

State transitions are driven by `Channel::Update()`, which is called on each new block tip.

---

## Channel Open

**Participants:** Role 0 (initiator, funds opening tx) and Role 1 (responder).

**Negotiation class:** `Negotiator::ChannelOpen`

**Sub-negotiations run in parallel:**
- `Multisig m_MSig` — creates `msig0`
- `MultiTx m_Tx0` — builds the opening transaction `Inputs → msig0 + Change`
- `WithdrawTx m_WdA` — builds Role 0's initial withdrawal path (Exit-A)
- `WithdrawTx m_WdB` — builds Role 1's initial withdrawal path (Exit-B)

**Message flow (3 full round-trips, field-level breakdown):**

```
A → B  591 bytes
       MultiSig.Partial Commitment
       MultiSig.Bulletproof T1,T2
       Tx-Open.Excess Commitment
       Tx-Open.Nonce Commitment
       Exit-A.MultiSig.Partial Commitment
       Exit-A.MultiSig.Bulletproof T1,T2
       Exit-A.Tx-TLock.Excess Commitment
       Exit-A.Tx-TLock.Nonce Commitment
       Exit-A.Tx-Final.Excess Commitment
       Exit-A.Tx-Final.Nonce Commitment
       Exit-B.MultiSig.Partial Commitment
       Exit-B.MultiSig.Bulletproof T1,T2
B → A 1754 bytes
       MultiSig.Partial Commitment
       MultiSig.Bulletproof T1,T2
       MultiSig.Bulletproof TauX
       Tx-Open.Excess Commitment
       Tx-Open.Nonce Commitment
       Tx-Open.Partial Kernel Signature
       Exit-A.MultiSig.Partial Commitment
       Exit-A.MultiSig.Bulletproof T1,T2
       Exit-A.MultiSig.Bulletproof TauX
       Exit-A.Tx-TLock.Excess Commitment
       Exit-A.Tx-TLock.Nonce Commitment
       Exit-A.Tx-TLock.Partial Kernel Signature
       Exit-A.Tx-Final.Excess Commitment
       Exit-A.Tx-Final.Nonce Commitment
       Exit-A.Tx-Final.Partial Kernel Signature
       Exit-A.Tx-Final.Partial Transaction
       Exit-B.MultiSig.Partial Commitment
       Exit-B.MultiSig.Bulletproof T1,T2
       Exit-B.MultiSig.Bulletproof TauX
       Exit-B.Tx-TLock.Excess Commitment
       Exit-B.Tx-TLock.Nonce Commitment
       Exit-B.Tx-Final.Excess Commitment
       Exit-B.Tx-Final.Nonce Commitment
A → B 1968 bytes
       Exit-A.Tx-Final.Partial Transaction
       Exit-B.MultiSig.Bulletproof TauX
       Exit-B.Tx-TLock.Excess Commitment
       Exit-B.Tx-TLock.Nonce Commitment
       Exit-B.Tx-TLock.Partial Kernel Signature
       Exit-B.Tx-Final.Excess Commitment
       Exit-B.Tx-Final.Nonce Commitment
       Exit-B.Tx-Final.Partial Kernel Signature
       Exit-B.Tx-Final.Partial Transaction
B → A  977 bytes
       Exit-A.Tx-TLock.Partial Transaction
       Exit-B.Tx-Final.Partial Transaction
A → B   52 bytes
       Exit-B.Tx-TLock.Partial Transaction
B → A   85 bytes
       Tx-Open.Partial Transaction
B done
A done
```

**Ordering invariant:** Neither party completes its Phase 1 transaction for the peer until it has the peer's Phase 2 transaction. This prevents the peer from locking funds permanently. Specifically, B delays delivering Exit-A.Tx-TLock to A until B has received and validated Exit-A.Tx-Final from A.

**On-chain:** Only Role 0 broadcasts the opening transaction. Both parties monitor for `msig0` confirmation via fly-client kernel proof (`m_hvKernel0`).

---

## Channel Update (Off-Chain Transfer)

**Negotiation class:** `Negotiator::ChannelUpdate`

Each update creates revision N+1 and revokes revision N. The update builds:
- Two new withdrawal paths (Exit-A.N+1 and Exit-B.N+1) with updated output amounts
- Reveals the blinding factor scalar for each party's old `msigN` (revocation)

**Message flow (3 full round-trips, field-level breakdown):**

```
A → B  394 bytes
       Exit-A.MultiSig.Partial Commitment
       Exit-A.MultiSig.Bulletproof T1,T2
       Exit-A.Tx-TLock.Excess Commitment
       Exit-A.Tx-TLock.Nonce Commitment
       Exit-A.Tx-Final.Excess Commitment
       Exit-A.Tx-Final.Nonce Commitment
       Exit-B.MultiSig.Partial Commitment
       Exit-B.MultiSig.Bulletproof T1,T2
B → A 1477 bytes
       Exit-A.MultiSig.Partial Commitment
       Exit-A.MultiSig.Bulletproof T1,T2
       Exit-A.MultiSig.Bulletproof TauX
       Exit-A.Tx-TLock.Excess Commitment
       Exit-A.Tx-TLock.Nonce Commitment
       Exit-A.Tx-TLock.Partial Kernel Signature
       Exit-A.Tx-Final.Excess Commitment
       Exit-A.Tx-Final.Nonce Commitment
       Exit-A.Tx-Final.Partial Kernel Signature
       Exit-A.Tx-Final.Partial Transaction
       Exit-B.MultiSig.Partial Commitment
       Exit-B.MultiSig.Bulletproof T1,T2
       Exit-B.MultiSig.Bulletproof TauX
       Exit-B.Tx-TLock.Excess Commitment
       Exit-B.Tx-TLock.Nonce Commitment
       Exit-B.Tx-Final.Excess Commitment
       Exit-B.Tx-Final.Nonce Commitment
A → B 1968 bytes
       Exit-A.Tx-Final.Partial Transaction
       Exit-B.MultiSig.Bulletproof TauX
       Exit-B.Tx-TLock.Excess Commitment
       Exit-B.Tx-TLock.Nonce Commitment
       Exit-B.Tx-TLock.Partial Kernel Signature
       Exit-B.Tx-Final.Excess Commitment
       Exit-B.Tx-Final.Nonce Commitment
       Exit-B.Tx-Final.Partial Kernel Signature
       Exit-B.Tx-Final.Partial Transaction
B → A  977 bytes
       Exit-A.Tx-TLock.Partial Transaction
       Exit-B.Tx-Final.Partial Transaction
A → B   92 bytes
       Reveal Previous Blinding Factor
       Exit-B.Tx-TLock.Partial Transaction
B → A   40 bytes
       Reveal Previous Blinding Factor
B done
A done
```

**Revocation exchange:** Both parties reveal their previous-revision blinding factor in the same step. `ChannelUpdate::Codes::PeerBlindingFactor` / `SelfKeyRevealed` / `PeerKeyValid` track whether the exchange is complete before declaring success.

`Channel::m_nRevision` tracks the current revision number. Old `DataUpdate` entries are garbage-collected from `m_lstUpdates` once both parties have revealed their keys for them.

---

## Channel Close

### Graceful (Cooperative) Close

Both parties are online and agree to close. They negotiate a single `MultiTx` (`NegotiationCtx_Close`) that spends `msig0` directly to the agreed final outputs — no intermediate `msigN` and no timelock required. Settlement is immediate in one block.

Called via `Channel::Transfer(amount, /*bCloseGraceful=*/true)` or `Mediator::GracefulClose()`.

### Unilateral (One-Side) Close

One party is unresponsive or has cheated. The honest party calls `Channel::Close()`, which sets `m_State.m_Terminate = true`.

**Phase 1:** Broadcast the most recent private `Tx1` (msig0 → msigN). The `Channel::SelectWithdrawalPath()` method selects the latest valid revision. This is the "point of no return."

**Phase 2:** After `kDefaultLockTime` blocks, broadcast `Tx2` (msigN → outputs). The timelock is relative to the Phase 1 kernel, enforced by the node.

**Cheat detection:** On every new block tip, `Channel::Update()` checks whether a `msigN` belonging to a *revoked* revision has appeared on-chain. If so, `Channel::CreatePunishmentTx()` fires immediately, spending the fraudulent `msigN` to the honest party before the timelock expires.

`Channel::IsUnfairPeerClosed()` returns true when the channel detected and responded to a cheating attempt.

---

## `Negotiator` Framework

The `Negotiator` namespace in `core/negotiator.h` provides a generic multi-round negotiation framework used by both Laser channels and the basic transaction protocol.

### Key Types

| Type | Role |
|---|---|
| `IBase` | Abstract negotiator; holds `m_pStorage`, `m_pGateway`, position counter `m_Pos` |
| `Storage::Map` | In-memory key-value store for negotiator state; `map<uint32_t, ByteBuffer>` |
| `Gateway::IBase` | Abstraction for sending data to the peer (`Send(code, ByteBuffer)`) |
| `Gateway::Direct` | Local gateway that writes directly into the peer's `Storage::IBase` (used in tests) |
| `IBase::Router` | Remaps storage/gateway offsets so sub-negotiators share a single store without key collisions |

### Code Namespace

`Negotiator::Codes` defines the key space:

- `0–127` — peer-writable variables (peer can send once, cannot overwrite)
- `128–189` — private (local only)
- `129–189` — input parameters set by the caller
- `190–220` — internal working variables
- `221` — `Status` (Pending / Success / Error)
- `230+` — output results

### Composing Negotiators

Complex protocols like `ChannelOpen` aggregate simpler ones (`Multisig`, `MultiTx`, `WithdrawTx`). Each sub-negotiator is wired through an `IBase::Router` that shifts its key space by a channel offset, preventing collisions. Multiple sub-negotiators may advance in the same `Update()` call, achieving parallel negotiation in a single round-trip.

The `RaiseTo(pos)` helper advances the logical position counter; a sub-negotiator only sends data when its position is first reached, preventing duplicate transmissions on retries.

### Primitive Negotiation Traces

The traces below show individual building blocks in isolation to illustrate the base round-trip cost before they are composed into a full channel operation.

**MultiSig (1 round-trip):**

```
A → B  115 bytes
       Partial Commitment
       Bulletproof T1,T2
B → A  155 bytes
       Partial Commitment
       Bulletproof T1,T2
       Bulletproof TauX
B done
A done
```

Both parties receive the commitment. Only A holds the complete Bulletproof (TauX is sent from B to A only), so only A can spend the UTXO unilaterally — which is the intended asymmetry for the party whose withdrawal path uses this `msigN`.

**One-sided Refund (`WithdrawTx`, 2 round-trips):**

```
A → B  279 bytes
       MultiSig.Partial Commitment
       MultiSig.Bulletproof T1,T2
       Tx-TLock.Excess Commitment
       Tx-TLock.Nonce Commitment
       Tx-Final.Excess Commitment
       Tx-Final.Nonce Commitment
B → A 1158 bytes
       MultiSig.Partial Commitment
       MultiSig.Bulletproof T1,T2
       MultiSig.Bulletproof TauX
       Tx-TLock.Excess Commitment
       Tx-TLock.Nonce Commitment
       Tx-TLock.Partial Kernel Signature
       Tx-Final.Excess Commitment
       Tx-Final.Nonce Commitment
       Tx-Final.Partial Kernel Signature
       Tx-Final.Partial Transaction
A → B  925 bytes
       Tx-Final.Partial Transaction
B → A   52 bytes
       Tx-TLock.Partial Transaction
B done
A done
```

B deliberately delays sending Tx-TLock (Phase 1) to A until it has received and validated Tx-Final (Phase 2) from A. This enforces the ordering invariant: A cannot initiate withdrawal without Phase 2 already in B's possession.

---

## Wallet-Layer Classes

### `laser::Channel` (`wallet/laser/channel.h`)

Extends `Lightning::Channel` with wallet-specific I/O:

- `get_Kdf()` — provides the wallet's key derivation function
- `SelectInputs()` — queries `WalletDB` for UTXOs to fund the opening transaction
- `AllocTxoID()` — assigns fresh `CoinID` values for new UTXOs
- `SendPeer()` — routes negotiation messages through the SBBS gateway
- `OnCoin()` — notifies `WalletDB` of coin state changes (locked/confirmed/spent)

State is persisted to `WalletDB` via `UpdateRestorePoint()`. The `ByteBuffer m_data` field stores the full serialized channel state for recovery after a wallet restart.

A channel is identified by `ChannelID` — a 128-bit (`uintBig_t<16>`) random value chosen at open time.

### `laser::Mediator` (`wallet/laser/mediator.h`)

The top-level manager for all channels in a wallet. Acts as a `proto::FlyClient` to monitor the chain.

**Key responsibilities:**
- `WaitIncoming()` / `OpenChannel()` — initiate or listen for new channels
- `Transfer()` / `Close()` / `GracefulClose()` — delegate to the appropriate `Channel`
- `UpdateChannels()` — called on each `OnNewTip()`, runs `Channel::Update()` for every active channel
- `ListenClosedChannelsWithPossibleRollback()` — re-checks recently closed channels after a chain rollback

**Observer pattern:** Components subscribe via `AddObserver(Observer*)` and receive callbacks:

| Callback | Trigger |
|---|---|
| `OnOpened` | Opening transaction confirmed |
| `OnOpenFailed` | Opening tx not confirmed in time |
| `OnClosed` | Phase 2 close confirmed |
| `OnUpdateStarted` / `OnUpdateFinished` | Transfer negotiation started/completed |
| `OnTransferFailed` | Update negotiation failed |
| `OnExpired` | Channel revision lifetime exceeded |

### `laser::Connection` (`wallet/laser/connection.h`)

A thin wrapper around `proto::FlyClient::NetworkStd` that provides the `INetwork` interface to `Mediator`. BBS subscription is forwarded to the underlying network, enabling channels to receive peer messages via SBBS.

---

## Channel Lifecycle in Practice

```
┌─────────────────────────────────────────────────────┐
│  Mediator::OpenChannel()                            │
│    → laser::Channel constructed (Opening0)          │
│    → ChannelOpen negotiation starts via SBBS        │
│    → Opening tx broadcast (Role 0 only)             │
│    → Wait for msig0 kernel confirmation             │
│  State: Opening0 → Opening1 → Opening2 → Open       │
│                                                     │
│  Mediator::Transfer()                               │
│    → ChannelUpdate negotiation via SBBS             │
│    → Blinding factors exchanged (revocation)        │
│  State: Open → Updating → Open                      │
│                                                     │
│  Mediator::GracefulClose()  (both online)           │
│    → MultiTx msig0 → outputs negotiated             │
│    → Single on-chain tx                             │
│  State: Open → Closing1 → Closed                    │
│                                                     │
│  Mediator::Close()  (unilateral)                    │
│    → Broadcast Tx1 (msig0 → msigN)                  │
│    → Wait kDefaultLockTime blocks                   │
│    → Broadcast Tx2 (msigN → outputs)                │
│  State: Open → Closing1 → Closing2 → Closed         │
└─────────────────────────────────────────────────────┘
```

---

## Blockchain Monitoring

Both parties must monitor the chain to detect one-sided closures and respond within the timelock window. `Mediator::UpdateChannels()` runs this check on every `OnNewTip()` callback. The logic per channel:

1. **Is `msig0` still in the UTXO set?**
   - Yes → channel is open, no action needed.
   - No → the channel is being closed (or failed to open).

2. **Was the channel ever confirmed open?**
   - No → still waiting for the opening transaction; check against `hrLimit.m_Max`.
   - Yes → search for a `msigN.X` that matches one of the withdrawal revision commitments.

3. **Does the `msigN.X` on-chain correspond to a revoked revision?**
   - Yes → **cheat detected**: call `CreatePunishmentTx()` immediately to claim the UTXO before the timelock window closes.
   - No → valid withdrawal; wait for `kDefaultLockTime` blocks, then broadcast the Phase 2 transaction.

The monitoring interval does not need to match every block. Because all timelocks use `kDefaultLockTime` (default: 1,440 blocks ≈ 1 day), a wallet that checks once every few hundred blocks is sufficient to detect fraud in time. The `kMaxBlackoutTime` constant (6 blocks) sets the hard minimum check interval below which the wallet is considered too out-of-sync to safely monitor channels.

---

## Current Limitations vs. Full Lightning Network

| Feature | Beam Laser | LN (Bitcoin) |
|---|---|---|
| Multi-hop routing | Not implemented | Yes (HTLCs + onion routing) |
| Number of parties | 2 only | 2 per channel, multi-hop via network |
| Asset support | BEAM only (CA channels not implemented) | BTC only |
| Watch towers | Local monitoring only | Third-party watch towers possible |
| Channel factories | Not implemented | Research stage |
| HTLC-based swaps | Not implemented | Core building block |

Beam Laser channels are direct peer-to-peer payment channels. Routing across channels is not supported; each channel requires a direct on-chain funding transaction between the two parties.

---

## CLI Reference

All channel operations use the `beam-wallet laser` subcommand.

**1. List channels**

```
./beam-wallet laser --laser_channels_list
```

**2. Wait for an incoming connection (generates a receive address)**

```
./beam-wallet laser --laser_receive --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>
```

Example:

```
./beam-wallet laser --laser_receive --laser_my_locked_amount 1.1 --laser_remote_locked_amount 1.1 --laser_fee 100
```

**3. Open a channel (connect to a peer)**

```
./beam-wallet laser --laser_open --laser_address <address> --laser_my_locked_amount <amount in beam> --laser_remote_locked_amount <amount in beam> --laser_fee <amount in groth>
```

Example:

```
./beam-wallet laser --laser_open --laser_address 285a776d78e6e0ee285a196282e61768b87c7c108a7d8cf7622a094555d2cfeb80e --laser_my_locked_amount 0.9 --laser_remote_locked_amount 1.1 --laser_fee 100
```

**4. Listen on channels**

```
./beam-wallet laser --laser_listen [channel id 1,channel id 2, ... channel id N]
```

Examples:

```
./beam-wallet laser --laser_listen
./beam-wallet laser --laser_listen 4bd5ee31b264f6102709dc145cf37b55
./beam-wallet laser --laser_listen 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37
```

**5. Send coins off-chain**

```
./beam-wallet laser --laser_send <amount in beam> --laser_channel <channel id>
```

Example:

```
./beam-wallet laser --laser_send 0.1 --laser_channel 4bd5ee31b264f6102709dc145cf37b55
```

**6. Drop channels (unilateral close — after lock time expires or peer is offline)**

```
./beam-wallet laser --laser_drop <channel id 1,channel id 2, ... channel id N>
```

Examples:

```
./beam-wallet laser --laser_drop 4bd5ee31b264f6102709dc145cf37b55
./beam-wallet laser --laser_drop 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37
```

**7. Close channels gracefully (cooperative close — both parties online, before lock time expires)**

```
./beam-wallet laser --laser_close <channel id 1,channel id 2, ... channel id N>
```

Examples:

```
./beam-wallet laser --laser_close 4bd5ee31b264f6102709dc145cf37b55
./beam-wallet laser --laser_close 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37
```

**8. Delete channels from the database (only for already-closed channels)**

```
./beam-wallet laser --laser_delete <channel id 1,channel id 2, ... channel id N>
```

Examples:

```
./beam-wallet laser --laser_delete 4bd5ee31b264f6102709dc145cf37b55
./beam-wallet laser --laser_delete 4bd5ee31b264f6102709dc145cf37b55,73e5af986eb3ea165f71bbb54ebfad37
```

A working demo covering graceful open/close, one-side closure, and cheat/punishment scenarios is available in the Beam repository at `node/utils/laser_beam_demo.cpp`. It uses the standard Beam node configured with fake PoW — no special test builds required. All broadcasted transactions and timelocks are fully validated by the node.

---

## Related Pages

- [Core-Cryptographic-Primitives](../core/Core-Cryptographic-Primitives.md) — Pedersen commitments, Bulletproofs, Schnorr multi-sig
- [Core-Transaction-Elements](../core/Core-transaction-elements.md) — `TxKernelStd` relative lock fields
- [Transactions-Creation-Protocol](Transactions-Creation-Protocol.md) — `BaseTransaction` / negotiator framework used for standard txs
- [Wallet-SBBS](../wallet/Wallet-SBBS.md) — message transport used for channel negotiation
- [Node-Fly-Client-Protocol](../node/Node-Fly-Client-Protocol.md) — chain monitoring used by `Mediator`
