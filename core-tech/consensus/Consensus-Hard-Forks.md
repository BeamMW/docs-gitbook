# Consensus Hard Forks

Beam's consensus rules evolve through numbered hard forks. Every full node and miner must upgrade before a fork height is reached; nodes that do not upgrade will follow a different chain and become unable to sync with the canonical tip.

This page documents the `Rules` struct that encodes consensus parameters, the fork height table for each network, and the specific consensus changes introduced at each fork boundary.

---

## The `Rules` Struct

All consensus-sensitive parameters live in a single `struct Rules` (`core/block_crypt.h`). At startup each binary loads exactly one `Rules` instance, selects the correct network, and calls `SetNetworkParams()` to populate fork heights and parameter tables.

```cpp
struct Rules {
    HeightHash pForks[7];       // indexed 0–6; pForks[i].m_Height is the activation height
    Network    m_Network;       // mainnet | masternet | testnet | dappnet | dappnet2 | warp_dev3
    Consensus  m_Consensus;     // PoW | FakePoW | Pbft

    struct { Amount DepositForList2; Amount DepositForList5; ... } CA;
    struct { bool Enabled; Sigma::Cfg m_ProofMax; ... } Shielded;
    struct { Pbft::Whitelist m_Whitelist; ... } m_Pbft;

    Height MaxKernelValidityDH; // enforced past Fork 2
    size_t MaxBodySize;
    // ...
};
```

`Rules::get()` returns a `thread_local` pointer to the active instance. Test code uses an RAII guard to override it for a single scope:

```cpp
struct Rules::Scope {
    Scope(const Rules& r);  // pushes r as the active instance
    ~Scope();               // restores the previous instance
};
```

### Key query methods

| Method | Meaning |
|---|---|
| `IsPastFork(h, i)` | Returns `true` if height `h` has passed fork `i` |
| `IsPastFork_<i>(h)` | Compile-time-indexed form; `static_assert` on `i` |
| `TestForkAtLeast_<i>(h)` | Throws `Fail_Fork(i)` if `h` has not yet passed fork `i` |
| `DisableForksFrom(i)` | Sets forks `i…6` to `MaxHeight` (disables them) |

### Test helpers

```cpp
Rules::Fail_Fork(iFork);  // throws; used by TestForkAtLeast_ in validators
rules.DisableForksFrom(3); // pin consensus at Fork 2 era for unit tests
```

---

## Network Fork Height Table

`pForks[0]` is always height 0 (genesis). Heights listed as `—` are set to `MaxHeight`, meaning that fork is not yet active on that network.

| Fork | Mainnet | Masternet | Testnet |
|---|---|---|---|
| 0 (Genesis) | 0 | 0 | 0 |
| 1 | 321,321 | 30 | 270,910 |
| 2 | 777,777 | 30 | 690,000 |
| 3 | 1,280,000 | 1,500 | 1,135,300 |
| 4 | 1,820,000 | 516,700 | 1,670,000 |
| 5 | 1,920,000 | 676,330 | 1,780,000 |
| 6 | — | — | — |

`dappnet` and `dappnet2` are developer test networks with either fake PoW or PBFT consensus and compress early forks to block 30–100 to reduce iteration time. `warp_dev3` is the Beam Warp PBFT testbed; it enables all forks from block 0.

---

## Fork-by-Fork Consensus Changes

### Fork 0 — Genesis

Establishes the initial Beam consensus:

- **PoW algorithm:** BeamHash I — Equihash-R(150, 5, 0) with a Blake2b seed. The `(150, 5)` parameters match BTG's recommendation (~1 GB average working set). The final `0` is the nonce personalization used by BeamHash I.
- **Block body:** Inputs, outputs (Pedersen commitments with Bulletproof range proofs), and standard kernels (`TxKernelStd`).
- **No nested kernels**, no relative time-locks, no asset proofs, no shielded pool.
- Emission: 80 BEAM per block, halving schedule governed by `Emission.Drop0` / `Emission.Drop1`.
- Initial difficulty: `2^22 ≈ 4.19M` on mainnet; `2^8 = 256` for test networks.

### Fork 1 — Height 321,321 (mainnet)

**PoW algorithm change:** BeamHash I → **BeamHash II**, Equihash-R(150, 5, **3**). The personalization nonce changes from 0 to 3, making old BeamHash I miners produce invalid proofs.

**Kernel extensions enabled:**

- `TxKernel::m_CanEmbed = true` — a kernel may be nested inside another kernel. Validation enforces that a child kernel's height range contains the parent's range.
- `TxKernelStd::m_pRelativeLock` — a standard kernel may carry a relative time-lock referencing an earlier kernel. The lock must be resolved before the kernel becomes valid.

**Output oracle change:** Past Fork 1, the output Pedersen commitment and any asset proof are included in the oracle used for Bulletproof recovery. This strengthens key-recovery security.

### Fork 2 — Height 777,777 (mainnet)  *(Eager Electron 5.0)*

**PoW algorithm change:** BeamHash II → **BeamHash III**. BeamHash III replaces the Equihash construction entirely with a GPU-friendly memory-hard function (see [Beam Equihash specification](Beam-Equihash-specification)).

**Confidential Assets (CA) activated:**

- `TxKernelAssetCreate`, `TxKernelAssetEmit`, `TxKernelAssetDestroy` become valid transaction kernel types. All require `TestForkAtLeast_<2>` and `CA.Enabled = true`.
- Listing a new asset requires locking a deposit of **3,000 BEAM** (`CA.DepositForList2`).
- `Rules::IsEnabledCA(hScheme)` returns `IsPastFork_<2>(hScheme) && CA.Enabled`.

**Shielded pool (Lelantus-MW) activated:**

- `Shielded.Enabled = true`. Push and pull transactions (`TxKernelShieldedOutput` / `TxKernelShieldedInput`) become valid.
- Anonymity set parameters: `m_ProofMax = {4,8}` (64 K elements), `m_ProofMin = {4,5}` (1 K elements).
- Per-block limits: 20 shielded inputs, 30 shielded outputs.

**Kernel validity window capped:**

`MaxKernelValidityDH` is enforced: a kernel whose height range spans more than `1440 × 30 = 43,200` blocks has its effective maximum height silently clamped. This prevents kernels with arbitrarily long lifetimes from blocking UTXO cut-through.

**Nested kernel ordering relaxed:**

Prior to Fork 2, nested kernels within a parent had to be sorted by kernel ID (a historical enforcement). After Fork 2, sort order is not enforced. Code guarded by `!r.IsPastFork_<2>(hScheme)` will be unreachable once Fork 2 is past everywhere.

**Parent-nested commitment semantics changed:**

Before Fork 2, a parent kernel's excess commitment was supposed to absorb the nested kernels' excess. After Fork 2, nested excess is accumulated separately into `exc`, matching the correct Mimblewimble balance equation.

### Fork 3 — Height 1,280,000 (mainnet)  *(Fierce Fermion 6.0)*

**Smart contracts / BVM activated:**

`TxKernelContractControl` (and its subtypes `ContractCreate`, `ContractInvoke`, `ContractDestroy`) require `TestForkAtLeast_<3>`. The BVM WASM interpreter begins executing shaders embedded in contract kernels. See [Programming Beam](Programming-Beam) for shader development.

**Live state Merkle root restructured:**

The `Block::SystemState` live hash changes structure at each fork boundary:

| Era | Live hash composition |
|---|---|
| Before Fork 2 | `Hash(UTXOs)` |
| Fork 2 – Fork 3 | `Hash(UTXOs \| Hash(Shielded \| Assets))` |
| Past Fork 3 | `Hash(Contracts \| Hash(Hash(Kernels \| Logs) \| Hash(Shielded \| Assets)))` |

The `get_Live`, `get_CSA`, `get_KL`, and `get_SA` evaluator methods in `block_crypt.cpp` switch on `IsPastFork_<3>` and `IsPastFork_<2>` to select the correct composition.

**Kernel log sub-tree added:**

Past Fork 3, block headers commit to a `Kernels` sub-tree (kernels of this block) and a `Logs` sub-tree. Before Fork 3, `m_Kernels` held only block-local kernels; after Fork 3 it participates in the `KL` (Kernels + Logs) combined hash.

**Address types expanded:**

The 6.0 wallet introduces new Base58-encoded address types (max-privacy, offline, public offline). These are wallet-layer changes that do not affect consensus directly, but validators must understand them for fee calculation — offline and max-privacy transactions include at least one shielded output, raising the minimum fee to 1,100,000 Groth.

### Fork 4 — Height 1,820,000 (mainnet)

**Dependent contract context hashing:**

`TxKernelContractControl` carries a `m_Dependent` flag. When set, the kernel's signing hash must include the parent contract context hash `*pParentCtx`. This dependency is only included in the hash past Fork 4:

```cpp
if (m_Dependent) {
    if (Rules::get().IsPastFork_<4>(m_Height.m_Min))
        hp << *pParentCtx;
}
```

This allows contract invocations to be cryptographically bound to a specific parent contract state, preventing replay across unrelated contexts.

### Fork 5 — Height 1,920,000 (mainnet)

**CA deposit mechanism redesigned:**

The fixed 3,000 BEAM listing deposit is replaced by a variable deposit. Past Fork 5:

- `CA.DepositForList5 = 10 BEAM` becomes the default deposit for new asset registrations.
- `TxKernelAssetDestroy::IsCustomDeposit()` returns `true` (via `IsPastFork_<5>`), meaning the actual deposit to return is stored in `m_Deposit` within the kernel rather than read from `Rules::CA`.
- `Rules::get_DepositForCA(hScheme)` returns `DepositForList5` past Fork 5, `DepositForList2` before it.

Assets registered before Fork 5 retain their 3,000 BEAM deposit, which is returned when the asset is destroyed.

**Shielded output validation:**

`TxKernelShieldedOutput::IsValid()` uses `IsPastFork_<5>` internally. This guards new validation rules for shielded outputs that were not enforced in earlier eras.

### Fork 6 — Not yet active on mainnet

**EVM integration:**

`TxKernelEvmInvoke` is a new kernel subtype that embeds an Ethereum-compatible transaction into a Beam block. Its `TestValid()` asserts `TestForkAtLeast_<6>`. The `Evm.Groth2Wei` parameter controls the Groth-to-Wei conversion ratio; it is currently set to `0` (EVM disabled) in all shipped networks.

When activated, EVM execution will share the BVM charge budget (100 M charge units per block) with WASM contract kernels. Gas is drawn at a 1:1 ratio from the block charge limit; the full fee is paid to the miner, and no fee is burned.

---

## Encoding in the Chain Checksum

Each fork's hash (`pForks[i].m_Hash`) is derived during `UpdateChecksum()` by hashing the fork height and the accumulated consensus parameters into a Blake2b oracle. The checksum is exchanged during the node handshake (`proto::Login`) so nodes on different forks reject each other immediately.

```
oracle << pForks[i].m_Height << ... >> pForks[i].m_Hash
```

---

## Practical Checklist for Pools and Exchanges

When a fork is approaching:

1. **Update node and wallet binaries** before the fork height. The new binary is backward-compatible with the old chain until the fork activates.
2. **Back up `node.db` and `wallet.db`** before the first run of the new binary. The schema may be migrated on first startup.
3. **Mining pools:** Ensure miners support the new PoW algorithm (if the fork includes a BeamHash change) and can switch automatically at the fork block. Stale shares using the old algorithm are invalid from fork height onward.
4. **CA-aware exchanges (Fork 2+):** Enable or explicitly disable CA support (`CA.Enabled`). By default CA is enabled; pools should leave it disabled and filter non-BEAM asset outputs.
5. **Fee schedules (Fork 3+):** Max-privacy and offline transactions include at least one shielded output. Account for the higher minimum fee (1,100,000 Groth base + 1,000,000 Groth per shielded output).
6. **Address validation regex (Fork 3+):** Both legacy hex addresses (64 chars) and new Base58 addresses are valid. Use `/[0-9a-zA-Z]{64,500}/` to accept both.

For network-specific upgrade announcements, see:
- [Upgrade Guide: Eager Electron 5.0](Beam-Eager-Electron-5.0-Upgrade-Guide-for-pools-and-exchanges) (Fork 2)
- [Upgrade Guide: Fierce Fermion 6.0](Beam-Fierce-Fermion-6.0-Upgrade-Guide-for-pools-and-exchanges) (Fork 3)
