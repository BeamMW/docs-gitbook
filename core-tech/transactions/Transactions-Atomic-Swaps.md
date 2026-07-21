# Atomic Swaps

Beam supports trustless cross-chain atomic swaps with Bitcoin-family UTXOchains and Ethereum-family chains. The protocol exchanges BEAM for an external coin without any third-party custodian: either both sides receive their funds, or both can reclaim what they locked.

**Supported chains:** Bitcoin, Litecoin, Qtum, Bitcoin Cash, Dogecoin, Dash (UTXO-based); Ethereum, DAI, USDT, WBTC (EVM-based, via smart contract).

---

## Protocol Overview

Two parties exchange assets:

- **Alice** — holds BEAM, wants the external coin.
- **Bob** — holds the external coin, wants BEAM.

The swap proceeds in three phases:

1. **Lock** — both sides lock their funds into time-bounded contracts or shared UTXOs on their respective chains.
2. **Exchange** — Bob broadcasts the Beam redeem transaction, which reveals the hash preimage. Alice uses that preimage to claim the external coin.
3. **Refund** — if the exchange never completes before the lock timeouts expire, each party can reclaim their own locked funds independently.

The asymmetry in lock timeouts is the core safety property: the external-chain locktime must be substantially larger than the Beam-side locktime, so Alice always has time to claim the external coin after she observes Bob's Beam redeem transaction.

---

## Sub-Transactions

Each atomic swap is composed of six sub-transactions tracked by `SubTxIndex`:

| Index | Constant | Description |
|-------|----------|-------------|
| 2 | `BEAM_LOCK_TX` | Creates the shared Beam UTXO (Alice + Bob, 2-party blinding factor) |
| 3 | `BEAM_REFUND_TX` | Time-locked; returns shared Beam UTXO to Alice on timeout |
| 4 | `BEAM_REDEEM_TX` | Bob claims the shared Beam UTXO, revealing the hash preimage |
| 5 | `LOCK_TX` | External chain: Bob's HTLC / smart contract lock |
| 6 | `REFUND_TX` | External chain: Bob reclaims his coin after timeout |
| 7 | `REDEEM_TX` | External chain: Alice claims Bob's coin using the revealed preimage |

---

## Beam-Side HTLC

Because Beam has no scripting language, the HTLC equivalent is constructed using two complementary `TxKernel` fields and a jointly-owned UTXO:

### Shared UTXO (Lock)

Alice and Bob each randomly choose a part of the blinding factor for a new UTXO:

```
shared_blinding = sfa + sfb   (sfa from Alice, sfb from Bob)
```

A Bulletproof for this output requires three interactive rounds. The `LockTxBuilder` (extends `MutualTxBuilder`) drives this multi-round exchange, accumulating `PeerPublicSharedBlindingFactor`, `PeerSharedBulletProofPart2`, and `PeerSharedBulletProofPart3` parameters from the counterparty.

The lock transaction spends Alice's existing UTXOs and creates the shared UTXO. Alice broadcasts it to the Beam network.

### Refund Transaction

Before the shared UTXO is created, Alice and Bob collaboratively sign a refund kernel:

- Kernel has `m_MinHeight` set to a future height — it cannot be broadcast until that block is reached.
- Alice stores this transaction locally; it is her safety net if Bob disappears.

Lock time constant:
```cpp
constexpr Height kBeamLockTimeInBlocks = 6 * 60;  // ~6 hours at 1 min/block
```

### Redeem Transaction (Exchange Kernel)

Alice and Bob each contribute half a Schnorr signature to a kernel that includes a `HashLock` field (from `TxKernel`). The kernel's hash-image (`hi`) is known to both parties, but the hash-preimage (`hpi`) is only known to Bob.

- Alice signs first, committing to the known `hi`.
- Bob adds his signature part **and substitutes the `hpi`** into the kernel to make it valid.
- Bob broadcasts the complete transaction.

Once this transaction is confirmed, the `hpi` is visible in the kernel on the Beam chain. Alice reads it and uses it to claim the external coin.

```
Beam time safety margin:
kMaxSentTimeOfBeamRedeemInBlocks = kBeamLockTimeInBlocks - 60  // Bob must redeem with 1h to spare
```

---

## External-Chain HTLC

### UTXO Chains (Bitcoin and derivatives)

The Bitcoin bridge (`BitcoinSide`) constructs a P2SH output using an `AtomicSwapContract` script:

```
IF                           -- redeem path
    OP_2
    <publicKeyB>             -- Bob's BTC key
    <publicKeySecret>        -- EC public key derived from the hash preimage
    OP_2
    OP_CHECKMULTISIG
ELSE                         -- refund path
    <locktime>
    OP_CHECKLOCKTIMEVERIFY
    OP_DROP
    <publicKeyA>             -- Alice's BTC key
    OP_CHECKSIG
ENDIF
```

The `publicKeySecret` is treated as a public key whose corresponding private key is the hash preimage scalar `hpi`. This is the EC-scalar ("aggregate signature") variant: the secret doubles as both the hash preimage in the Beam kernel and as a private key on the Bitcoin side.

**Redeem path:** Bob signs with his key, and also signs with `hpi` as a private key (2-of-2 multisig). To redeem the output Alice must supply `hpi` as a private key — she learns it from the Beam chain.

**Refund path:** After `locktime` blocks (BTC height), Bob can reclaim with his own key alone.

Segwit (P2WSH) is supported where available (`IsSegwitSupported()`, version-gated via `kSwapSegwitSupportMinProtoVersion = 5`).

Fee parameters are stored per sub-transaction:
- `Fee` on `LOCK_TX` sub-index = fee rate for the Bitcoin lock tx
- `Fee` on `REFUND_TX` / `REDEEM_TX` = rate for the respective withdrawal tx

### EVM Chains (Ethereum, ERC-20 tokens)

The Ethereum bridge (`EthereumSide`) interacts with a deployed swap smart contract instead of constructing scripts. The contract exposes three methods:

| Method | Who calls | Purpose |
|--------|-----------|---------|
| `lock` | Bob | Lock ETH / ERC-20 tokens with hash-image and timelock |
| `redeem` | Alice | Claim locked funds by supplying the preimage |
| `refund` | Bob | Reclaim locked funds after the timelock expires |

Method selectors are looked up via `ethereum::swap_contract::GetLockMethodHash()`, `GetRedeemMethodHash()`, `GetRefundMethodHash()`. ERC-20 tokens (`DAI`, `USDT`, `WBTC`) use a flag (`IsERC20Token()`) that appends the token contract address and value to the call data.

Hash-lock and aggregate-signature variants are both supported for EVM (`IsHashLockScheme()`), controlled by the method hash selected.

---

## State Machine

`AtomicSwapTransaction::State` drives both Alice's and Bob's sides through distinct paths.

### Alice's path (Beam side, `isBeamSide = true`)

```
Initial
  → BuildingBeamLockTX      (LockTxBuilder multi-round with Bob)
  → BuildingBeamRefundTX    (SharedTxBuilder: time-locked kernel)
  → BuildingBeamRedeemTX    (SharedTxBuilder: half-signed hash-lock kernel)
  → HandlingContractTX      (waiting for Bob's external lock; validates timelock)
  → SendingBeamLockTX       (broadcasts BEAM_LOCK_TX to Beam network)
  → SendingBeamRedeemTX     (monitors for Bob's BEAM_REDEEM_TX; extracts hpi)
  → [Bob broadcasts BEAM_REDEEM_TX, Alice observes hpi]
  → SendingRedeemTX         (Alice broadcasts REDEEM_TX on external chain using hpi)
  → CompleteSwap

Refund path (Beam lock expired, Bob never redeemed):
  SendingBeamRefundTX → Refunded
```

### Bob's path (external-coin side, `isBeamSide = false`)

```
Initial
  → HandlingContractTX      (builds and broadcasts external LOCK_TX)
  → SendingBeamLockTX       (waits for Alice's Beam lock to confirm)
  → SendingBeamRedeemTX     (broadcasts BEAM_REDEEM_TX, embedding hpi)
  → CompleteSwap

Refund path (Beam lock expired):
  SendingRefundTX → Refunded
```

### Terminal states

| State | Meaning |
|-------|---------|
| `CompleteSwap` | Both sides have received their funds |
| `Refunded` | Lock expired; party reclaimed their original funds |
| `Canceled` | Swap canceled before any lock was broadcast |
| `Failed` | Unrecoverable error |

---

## Swap Token Format

Swap offers are propagated between wallets as **swap tokens** — serialized `TxParameters` encoded in Base58. The token allows users to exchange offer details out-of-band (SBBS, clipboard, QR code) before either side broadcasts anything.

Token wire format:

```
| flags (1 byte) | optional TxID (1 or 17 bytes) | TxParameters list |
```

- **`flags`** — high bit (`0x80`) indicates this is a swap token. Lower bits are reserved.
- **`TxID`** — `0x01` followed by 16-byte UUID if present; `0x00` if absent.
- **`TxParameters`** — sequence of `(key, value)` pairs where keys are `TxParameterID` values and all integers are big-endian. Minimum size ~34 bytes (at least one address parameter required).

Key parameters embedded in a swap token:

| Parameter | Description |
|-----------|-------------|
| `Amount` | BEAM amount offered |
| `AtomicSwapAmount` | External coin amount offered |
| `AtomicSwapCoin` | Which coin (`Bitcoin`, `Ethereum`, etc.) |
| `AtomicSwapIsBeamSide` | Whether token creator holds the BEAM |
| `MinHeight` | Minimum Beam chain height for the swap |
| `PeerResponseTime` | How long the counterparty has to respond |
| `Lifetime` | Offer expiry in Beam blocks |
| `PeerAddr` | SBBS address of the offer creator |

`PrepareSwapTxParamsForTokenization()` strips private and session-specific fields before creating a shareable token. `MirrorSwapTxParams()` inverts the perspective (flips `IsBeamSide`, `IsSender`, `IsInitiator`, and swaps `MyAddr`/`PeerAddr`) to produce the accepting party's parameter set.

---

## Bridge Abstraction

Adding swap support for a new chain requires implementing the `SecondSide` interface:

```cpp
class SecondSide {
    virtual bool Initialize() = 0;        // load settings, init keys
    virtual bool InitLockTime() = 0;      // set external lock timelock
    virtual bool ValidateLockTime() = 0;  // verify peer's timelock is safe
    virtual void AddTxDetails(SetTxParameter&) = 0;  // inject bridge params into SBBS message
    virtual bool ConfirmLockTx() = 0;     // wait for N confirmations on external lock
    virtual bool ConfirmRefundTx() = 0;
    virtual bool ConfirmRedeemTx() = 0;
    virtual bool SendLockTx() = 0;        // broadcast external lock tx
    virtual bool SendRefund() = 0;
    virtual bool SendRedeem() = 0;
    virtual bool IsLockTimeExpired() = 0;
    virtual bool HasEnoughTimeToProcessLockTx() = 0;
    virtual bool IsQuickRefundAvailable() = 0;
};
```

A bridge is registered with the `AtomicSwapTransaction::Creator` via:

```cpp
creator.RegisterFactory(AtomicSwapCoin::Bitcoin,
    MakeSecondSideFactory<BitcoinSide, bitcoin::IBridge, bitcoin::ISettingsProvider>(
        bridgeCreator, settingsProvider));
```

Each registered factory maps a `AtomicSwapCoin` enum value to a `SecondSide` implementation. The core `AtomicSwapTransaction` is coin-agnostic; all chain-specific logic lives in the bridge.

---

## Privacy Considerations

The hash-preimage approach links the swap transactions on both chains: anyone who observes both blockchains and finds matching hash images (`hi`) can identify that a cross-chain swap occurred between the two transactions. This is a known privacy trade-off of the HTLC design.

The EC-scalar variant (used in the Bitcoin bridge via `publicKeySecret`) partially mitigates this by making the link non-obvious to on-chain observers who don't know the relationship between the Beam kernel excess and the Bitcoin redeem key, but correlation by timing and amounts remains possible.

See also: [Transactions-Lelantus-Shielded-Pool](Transactions-Lelantus-Shielded-Pool.md) for shielding BEAM funds before or after a swap to break the transaction graph.

---

## In-Depth Flow Diagram

![Atomic Swap Diagram](https://user-images.githubusercontent.com/2501619/60335463-abf39900-99a6-11e9-83ae-9494ea9e3577.png)
