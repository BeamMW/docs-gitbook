# Beam Warp: dPoS / PBFT Consensus

Beam Warp is a hybrid consensus layer that combines Beam's existing Proof-of-Work with a delegated Proof-of-Stake (dPoS) protocol based on Practical Byzantine Fault Tolerance (PBFT). In Warp mode, block production is driven by a rotating validator committee rather than by raw hash power, while the existing Beam transaction model, UTXO set, and BVM execution environment remain unchanged.

---

## Overview

Beam's consensus type is selected per-network through `Rules::m_Consensus`:

```cpp
enum struct Consensus {
    PoW    = 0,   // standard BeamHash III proof-of-work
    FakePoW = 1,  // testing only: trivial PoW
    Pbft   = 2,   // PBFT / Beam Warp
};
```

When `Pbft` is selected, `SetParamsPbft(nTarget_ms)` configures the network:

```cpp
void Rules::SetParamsPbft(uint32_t nTarget_ms) {
    m_Consensus        = Consensus::Pbft;
    DA.Target_ms       = nTarget_ms;         // slot duration in milliseconds
    DA.Difficulty0.m_Packed = 0;             // no PoW difficulty
    m_Pbft.m_RoundUp_ms    = nTarget_ms / 4; // each new round adds 25% more time
    ZeroObject(Emission);
    Maturity.Coinbase  = 0;
}
```

Key differences from PoW mode:
- `DA.Target_ms` is now in **milliseconds** (changed from seconds) to support sub-second PBFT timeslot precision.
- Mining emission is zeroed — all rewards come from transaction fees forwarded to the PBFT contract.
- `IsConstantSpan()` returns `false` under PBFT, because round duration is variable (increasing per view-change round).

---

## Network Configurations

Two development networks use PBFT consensus; mainnet activation is not yet scheduled.

| Network | Target slot | All forks from | Whitelist mode |
|---|---|---|---|
| `dappnet2` | 15 s | block 0 | 1-of-1 |
| `warp_dev3` | 3 s | block 0 | 2-of-4 |

`warp_dev3` is the primary Beam Warp testbed. It activates all forks at genesis, uses a 3-second slot target, and requires at least 2 of 4 pre-configured validator addresses to be present in any quorum certificate.

---

## Block Header in PBFT Mode

Under PBFT, the PoW field in the block header (`Block::SystemState::Full::PoW`) is reinterpreted as `Block::Pbft::HdrData`:

```cpp
struct Block::Pbft::HdrData {
    ECC::Hash::Value    m_hvVsBoth;   // hash(prev_block | next_block) — binds this slot to its neighbors
    uintBigFor<uint16_t>::Type m_Time_ms; // millisecond timestamp offset within the epoch
    QC                  m_QC;         // quorum certificate from the previous slot
    uint8_t             m_Flags1;
    Difficulty          m_Difficulty;
    // ...
};
```

`get_Timestamp_ms()` retrieves the millisecond-resolution timestamp from a PBFT block header.

### Quorum Certificate

```cpp
struct Block::Pbft::QC {
    ECC::Signature             m_Signature; // aggregated Schnorr signature
    Bitmask<s_MaxValidators>   m_Mask;      // bitmask of which validators signed
};
```

- Up to `s_MaxValidators = 96` validators are supported.
- The QC carried in block `N+1` attests to the finality of block `N`.
- `CheckQuorum(msg, qc)` verifies the aggregated signature against the validator set and checks that the supermajority threshold is met.

### Supermajority Rule

```cpp
bool IsMajorityReached(uint64_t wVoted, uint64_t wTotal, uint32_t nWhite) {
    if (nWhite < Rules::get().m_Pbft.m_Whitelist.m_NumRequired)
        return false;
    return (wVoted * 3 > wTotal * 2); // strict 2/3+ of total weight
}
```

A quorum is valid only if:
1. The sum of voting weights exceeds 2/3 of total validator weight (stake-weighted), **and**
2. At least `m_NumRequired` whitelisted validator addresses are among the signatories.

---

## Rules::Pbft Parameters

```cpp
struct Rules::Pbft {
    struct Whitelist {
        std::vector<PeerID> m_Addresses; // sorted list of trusted validator peer IDs
        uint32_t            m_NumRequired; // minimum whitelist members required in any QC
    } m_Whitelist;

    uint32_t m_RoundUp_ms; // extra time added per PBFT view-change round (= Target_ms / 4)
};
```

`IsPbftWhitelistMode()` returns `true` when `m_NumRequired > 0`. In whitelist mode, a quorum is only valid if it includes the required number of pre-approved validators, preventing a purely stake-weighted takeover by unknown delegators.

---

## Validator Address Derivation

Each validator node derives its PBFT address from its owner key:

```cpp
void Block::Pbft::DeriveValidatorAddress(Key::IKdf& kdf, Address& addr, ECC::Scalar::Native& sk) {
    kdf.DeriveKey(sk, Key::ID(0, Key::Type::Coinbase));
    addr.FromSk(sk);
}
```

The validator's `Address` is a `HashValue` (32 bytes) derived from the coinbase key slot. This is the identity used in both the PBFT protocol and the on-chain dPoS contract.

---

## dPoS Contract: `PBFT_DPOS`

The dynamic validator set and staking mechanics are implemented as a BVM contract (`pbft_dpos`). The node interacts with this contract at the protocol level for reward collection and validator status updates; everything else is user-facing.

### Settings

```cpp
struct PBFT_DPOS::Settings {
    AssetID  m_aidStake;          // asset used for staking (0 = native BEAM)
    uint32_t m_hUnbondLock;       // blocks to wait before unbonded stake can be withdrawn
    Amount   m_MinValidatorStake; // minimum stake required to register a validator
};
```

### Validator Lifecycle

```
Active → Jailed      (unresponsive, by node)
Jailed → Active      (unjailed, by node)
Active/Jailed → Suspended (slashed, by node)
Any → Tombed         (permanently disabled, by validators or by self)
```

| Status | Voting power | Eligible for reward | Leader eligible |
|---|---|---|---|
| Active | Yes | Yes | Yes |
| Jailed | Yes | **No** | **No** |
| Suspended | **No** | No | No |
| Tombed | No | No | No |

`Slash` is a transition event (not a persistent status): it burns 10% of the validator's stake, emits a `Slash` log event, increments `m_NumSlashed`, and transitions the validator to `Suspended`. If already `Tombed`, slashing still burns stake but does not change the status.

```cpp
evt.m_StakeBurned = stake / 10;          // 10% burned
vctx.m_Val.m_Weight -= evt.m_StakeBurned;
```

### Reward Distribution

Rewards flow into the contract via the node calling `Method_4 (AddReward)` once per block, forwarding the block's fee amount. Internally, a global accumulator tracks the reward per unit of weight:

```cpp
void Global::FlushRewardPending() {
    if (m_RewardPending) {
        m_accReward.Add(m_RewardPending, m_TotakStakeNonJailed);
        // ...
        m_RewardPending = 0;
    }
}
```

Per-validator and per-delegator reward cursors record their position in the accumulator, so any individual's accrued reward can be computed in O(1) without iteration. Validator commission (in centi-percents, max 8000 = 80%) is subtracted first; the remainder is distributed to delegators proportionally.

### Contract Methods

| Method | Caller | Purpose |
|---|---|---|
| 0 `Ctor` | Deployer | Initialize with `Settings` |
| 3 `ValidatorStatusUpdate` | Node | Jail, unjail, suspend, tomb, or slash a validator |
| 4 `AddReward` | Node | Forward block fees into the reward pool |
| 5 `DelegatorUpdate` | User | Deposit/withdraw stake, claim rewards |
| 6 `ValidatorRegister` | User | Register as a new validator with initial stake |
| 7 `ValidatorUpdate` | Validator | Lower commission or voluntarily tomb the validator |

### Delegation and Unbonding

- Any user can bond stake to any `Active` or `Jailed` validator via `DelegatorUpdate`.
- Reducing bonded stake moves the amount into an **unbonded** record keyed by `(delegator, lock_height)`. Unbonded stake cannot be withdrawn until `lock_height` is reached (i.e., `current_height ≥ lock_height`).
- `m_hUnbondLock = 0` means immediate withdrawal is possible.
- Unbonded stake can be re-bonded to a different validator before the lock expires, without waiting for the unbonding period to complete.
- A validator's own stake must remain at or above `m_MinValidatorStake` as long as the validator is not `Tombed`.

---

## Static Validator Contract: `PBFT_STAT`

For simpler deployments where the validator set is fixed (e.g., early testnet phases), `pbft_stat` provides the same I_PBFT interface but without delegation or dynamic stake:

```cpp
struct PBFT_STAT::Method::Create {
    uint32_t      m_Count;
    ValidatorInit* get_VI() const; // array of { Address, Weight } pairs
};
```

Reward accumulation and validator status transitions work identically to `pbft_dpos`, but there is no user-facing staking interface. The validator set is immutable after deployment (apart from status transitions initiated by the node).

---

## I_PBFT Interface

Both contracts implement the common `I_PBFT` interface, which defines the on-chain state the node reads directly:

```cpp
// Per-validator state (Key: tag=2 | Address)
struct I_PBFT::State::Validator {
    uint64_t m_Weight;     // voting weight (equals bonded stake in dPoS, static in STAT)
    Status   m_Status;     // Active | Jailed | Suspended | Tombed
    uint8_t  m_NumSlashed; // slash count (saturates at 255)
    Height   m_hSuspend;   // block height when last suspended
};

// Global state (Key: tag=1)
struct I_PBFT::State::Global {
    Amount m_RewardPending; // fees waiting to be distributed
};
```

The node mirrors this state in memory — it does not call the contract on every block to re-read it. Contract state changes are detected via the normal BVM contract execution path and synced into the node's in-memory validator set.

---

## Security Model

**Finality:** PBFT provides deterministic finality once a QC is included in the next block. A block carrying a valid QC for its predecessor is considered final — there is no probabilistic waiting as in PoW.

**Liveness:** If fewer than 1/3 of stake-weighted validators are unresponsive, PBFT halts rather than producing an invalid block. View-change rounds add `m_RoundUp_ms` per retry, backing off gracefully under partial failures.

**Slashing:** Byzantine behavior (equivocation — signing two conflicting QCs for the same slot) results in a 10% stake burn. The slash penalty escalates with `m_NumSlashed` and leads to permanent `Tombed` status after repeated violations.

**Whitelist guard:** `m_NumRequired > 0` prevents a purely anonymous stake takeover: even if an attacker accumulates >2/3 of staked weight, a quorum is invalid unless it also includes the required number of pre-approved validator identities.

**vs. pure PoW:** PoW offers probabilistic finality (50+ confirmations recommended for large transfers). PBFT offers single-block finality for transactions in a finalized block, but introduces liveness dependence on the validator committee.

---

## Tip trimming: empty PBFT slots (“cannibalization”)

In PBFT mode the node still stores chain states in `NodeDB` with the usual tip tables (`Tips`, `TipsReachable`). **Tip trimming** here means removing a **superseded empty tip row** after the chain cursor moves to a successor that **reuses the same block height and parent link** instead of appending a new height — behaviour the code calls **cannibalization** of the previous empty block.

### Empty tip flag

For PBFT headers, `TestBlock` (validator path) requires the `Empty` flag on `Block::Pbft::HdrData` to match the body: an empty block at height greater than 1 must set `HdrData::Flags::Empty`; non-empty blocks must clear it.

When applying blocks, the processor notes that zero-offset empty blocks are valid in PBFT, but **only the last block on the chain may be empty**, because intermediate empties would be merged away by design.

### How the next header reuses the empty tip

When the current tip is marked `Empty`, `GenerateNewBlock` does **not** advance `m_Number` or `m_Prev` for the new header. It points the new block at the **same** `(m_Prev, m_Number)` as the empty parent, subtracts the parent’s PoW difficulty from accumulated chain work, then adds a larger header difficulty computed from a span that counts **both** slots (`dh` includes the parent empty slot via `Difficulty2Span` on the parent difficulty). The new block is therefore the canonical representation of **multiple consecutive empty leader rounds** collapsed into one chain position.

`Validator::MakeFullHdr` applies the same rules when building the committed header after a quorum is reached.

### When the DB row is deleted (`TryGoUp`)

`NodeProcessor::TryGoUp` repeatedly selects the highest–chain-work **functional** tip (`EnumFunctionalTips`) and advances the cursor with `TryGoTo`. At entry it remembers whether the cursor was on an empty PBFT tip. After a successful move, if the cursor row changed and the **previous** position was that empty tip, the processor calls `DeleteState` on the old row only — the comment in code is explicit: **cannibalized empty forked blocks** are not kept.

So **tip trimming** is not a periodic background prune of arbitrary forks; it is the **targeted removal of the orphaned empty state** once a non-empty (or next) block has absorbed that slot into a single header.

### Relation to general pruning

`TryGoUp` still calls `PruneOld()` after cursor movement; that path removes **old inferior tips** by height vs `m_Horizon.m_Branching` and raises fossil / TXO horizons — the same mechanism as non-PBFT nodes. The PBFT-specific behaviour above is **in addition** to that and is tied to the `Empty` header flag and cannibalized headers.

---

## See Also

- [Consensus-Hard-Forks](Consensus-Hard-Forks.md) — Fork heights and activation rules per network
- [Consensus-BeamHash](Consensus-BeamHash.md) — BeamHash III PoW algorithm used in non-Warp mode
- [BVM-Internals](../bvm/BVM-Internals.md) — BVM execution environment that hosts the dPoS contract
- [BVM-Shader-Development](../bvm/BVM-Shader-Development.md) — SDK for writing BVM contracts
