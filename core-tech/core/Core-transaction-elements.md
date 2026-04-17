# Core Transaction Elements

This page documents the fundamental data structures that compose a Beam transaction: inputs, outputs, and kernels. It also covers the balance invariant that all valid transactions and blocks must satisfy, the validation rules applied by the node, and the serialization model.

> **Related pages:** [Core Cryptographic Primitives](Core-Cryptographic-Primitives) · [UTXO Set, Horizons and Cut-Through](UTXO-set,-horizons-and-cut-through) · [Transaction Creation Protocol](Transaction-creation-protocol) · [Confidential Assets](Confidential-assets) · [Lelantus MW (Shielded Pool)](Lelantus-MW)

---

## Primitive Types

| Type | Underlying | Description |
|---|---|---|
| `Height` | `uint64_t` | Block height. `MaxHeight` = `UINT64_MAX`. |
| `HeightRange` | `{m_Min, m_Max}` | Inclusive range. `m_Min > m_Max` is a valid empty range. |
| `Timestamp` | `uint64_t` | Unix timestamp in seconds. |
| `Amount` | `uint64_t` | Value in Groth (1 BEAM = 100,000,000 Groth). |
| `AmountBig::Type` | `uintBig_t<16>` | 128-bit accumulator for sums of many UTXOs. |
| `AmountSigned` | `int64_t` | Signed value used by asset emit/burn kernels. |
| `Asset::ID` | `uint32_t` | Asset identifier, 1-based. `0` is reserved for BEAM itself. |
| `ContractID` | `ECC::uintBig` | 256-bit contract address, derived from WASM bytecode hash. |
| `TxoID` | `uint64_t` | Sequential index of a shielded output in the shielded pool. |

---

## TxElement (Base)

Both `Input` and `Output` extend `TxElement`:

```cpp
struct TxElement {
    ECC::Point m_Commitment;  // Pedersen commitment to the UTXO
};
```

The commitment encodes both value and blinding factor: `C = v·H + k·G`, where `H` and `G` are fixed generators, `v` is the amount in Groth, and `k` is the secret blinding factor. See [Core Cryptographic Primitives](Core-Cryptographic-Primitives) for details.

---

## Input

```cpp
struct Input : public TxElement {
    // m_Commitment — commitment that must exist unspent in the UTXO set

    struct State {
        Height m_Maturity;    // block height at which this UTXO became spendable
        Input::Count m_Count; // number of duplicate commitments in the set
    };
};
```

In a standard transaction the `Input` carries only the commitment. The node looks up the UTXO in the radix tree to verify it exists and obtain its maturity.

**Macroblock note:** In compressed history blocks (macroblocks) the maturity is stored inline in the `Input::State` struct to avoid the radix-tree lookup during initial sync. This field is illegal in normal transactions.

---

## Output

```cpp
struct Output : public TxElement {
    bool   m_Coinbase;   // true iff this output was created by the miner (block reward)
    Height m_Incubation; // extra blocks of immaturity on top of system rules

    // Exactly one of these must be set:
    std::unique_ptr<ECC::RangeProof::Confidential> m_pConfidential; // Bulletproof
    std::unique_ptr<ECC::RangeProof::Public>       m_pPublic;       // explicit value + Schnorr sig
    Asset::Proof::Ptr                               m_pAsset;        // Sigma proof for CA outputs
};
```

### Range Proofs

| Proof type | When used | Amount visible |
|---|---|---|
| `Confidential` (Bulletproof) | All non-coinbase outputs (system-enforced) | No |
| `Public` | Coinbase outputs; test mode | Yes (`m_Value` field explicit) |

The public range proof consists of an explicit `Amount m_Value` and an `ECC::Signature` over the blinding factor, proving the creator holds the secret key.

**`m_Incubation` binding:** The incubation value is hashed into the range proof to prevent tampering. An attacker cannot increase the lock-up period after the proof is computed.

### Coinbase Rules
- `m_pPublic` is mandatory for coinbase outputs — the miner's reward amount must be visible.
- The sum of all coinbase amounts in a block must equal the emission for that block height (defined by `Rules::get_Emission(height)`).
- Coinbase outputs cannot appear in transactions; only in blocks.
- Under PBFT consensus (`Rules::Consensus::Pbft`) coinbase outputs are disabled entirely.

### Asset Outputs
Outputs for [Confidential Assets](Confidential-assets) carry an additional `m_pAsset` (`Asset::Proof`) — a Sigma proof that ties the commitment to a specific registered asset generator instead of the default BEAM generator `H`.

---

## TxKernel (Base)

A kernel is the primary authorization and fee-bearing element of a Beam transaction. Unlike UTXOs, kernels are never spent — they accumulate permanently in the kernel MMR.

```cpp
struct TxKernel {
    Amount      m_Fee;        // transaction fee in Groth (can be 0)
    HeightRange m_Height;     // timelock: valid only in [m_Min, m_Max]
    bool        m_CanEmbed;   // if true, this kernel may be nested (Fork1+)

    std::vector<Ptr> m_vNested; // nested (embedded) kernels

    // not serialized — computed on demand:
    mutable Lazy<Merkle::Hash> m_Lazy_ID;
};
```

All elements in `m_vNested` are included in the signing commitment, making the parent kernel immutable with respect to its children.

### Kernel ID

The kernel ID is the canonical 256-bit identifier used in the kernel MMR and for deduplication. No two kernels with the same ID may exist in the chain.

**TxKernelStd ID formula:**
```
ID = Blake2b(
    m_Fee | m_Height.m_Min | m_Height.m_Max   // HashBase fields
    | m_Commitment | 0 | flags                // excess key + flags byte
    | [HashLock image if flag bit 0 set]
    | [RelativeLock.m_ID | RelativeLock.m_LockHeight if flag bit 1 set]
    | [each nested kernel's ID]
)
```
`flags` encodes `(hashLock ? 1 : 0) | (relativeLock ? 2 : 0) | (canEmbed ? 4 : 0)`.

Including `m_Commitment` (the public excess key) in the hash makes the excess immutable after signing — any modification to the excess would invalidate both the ID and the signature.

The only forbidden ID value is `0`; if the hash produces zero, the implementation mutates the result.

**Non-standard kernel ID (`TxKernelNonStd` subclasses):**
Non-standard kernels use a two-layer derivation to separate the message-to-sign from the full ID:

```
Msg = Blake2b(
    HashBase | invalid_point_marker | subtype_enum
    | [nested kernel IDs]
    | HashSelfForMsg()    // subtype-specific fields signed by the counterparty
)
ID = Blake2b(Msg | HashSelfForID())   // adds signature bytes
```

This allows the message to be computed before the signature exists, which is required for interactive signing protocols.

### Effective Height Range

Past Fork2, the node caps a kernel's validity window:

```
effective_max = min(m_Height.m_Max, m_Height.m_Min + MaxKernelValidityDH)
```

`MaxKernelValidityDH` is a consensus parameter. Kernels with excessively wide windows have their effective expiry silently reduced, preventing indefinitely-valid kernels from bloating the kernel set.

### Nested Kernels

Nesting (`m_CanEmbed = true`, requires Fork1+) allows one kernel to be embedded inside another. The parent's ID covers all nested kernel IDs, making it impossible to remove a nested kernel without invalidating the parent.

**Use case — shielded outputs:** A `TxKernelShieldedOutput` is always nested inside a `TxKernelStd`. The sender cannot prove payment unless the shielded kernel is present, so nesting makes the two atomic. A receiver who sees the transaction in the pool cannot surgically strip the shielded kernel and replace it with a plain UTXO.

Nesting depth is limited to `TxKernel::s_MaxRecursionDepth = 2`.

---

## Kernel Type Hierarchy

The complete set of kernel subtypes:

| ID | Name | Description |
|---|---|---|
| 1 | `Std` | Standard transfer kernel |
| 2 | `AssetEmit` | Emit or burn Confidential Asset tokens |
| 3 | `ShieldedOutput` | Create a shielded (Lelantus) output |
| 4 | `ShieldedInput` | Spend a shielded (Lelantus) input |
| 5 | `AssetCreate` | Register a new Confidential Asset |
| 6 | `AssetDestroy` | Unregister a Confidential Asset |
| 7 | `ContractCreate` | Deploy a new BVM smart contract |
| 8 | `ContractInvoke` | Invoke a method on an existing BVM contract |
| 9 | `EvmInvoke` | Execute an EVM-compatible transaction |

### TxKernelStd

The standard kernel for BEAM transfers.

```cpp
struct TxKernelStd : public TxKernel {
    ECC::Point     m_Commitment;     // public excess key: k·G
    ECC::Signature m_Signature;      // Schnorr signature over the kernel ID

    struct HashLock {
        ECC::Hash::Value m_Value;    // hash image (commitment to preimage)
    };
    struct RelativeLock {
        Merkle::Hash m_ID;           // kernel ID this lock depends on
        Height       m_LockHeight;   // height delta from that kernel's inclusion
    };

    std::unique_ptr<HashLock>     m_pHashLock;
    std::unique_ptr<RelativeLock> m_pRelativeLock;
};
```

**`HashLock`:** The kernel is only valid when submitted with the preimage of `m_Value`. Used in atomic swap HTLC constructions (see [Atomic Swap](Atomic-swap)).

**`RelativeLock`:** This kernel is not valid unless the referenced kernel is already in the chain at some height `h`, and `currentHeight ≥ h + m_LockHeight`.

### TxKernelAssetControl (base for asset kernels)

```cpp
struct TxKernelAssetControl : public TxKernelNonStd {
    PeerID m_Owner;                           // asset owner's compressed public key
    ECC::Point m_Commitment;
    ECC::SignatureGeneralized<1> m_Signature; // multi-part Schnorr signature
};
```

**`TxKernelAssetEmit`** — adds `m_AssetID` and `m_Value` (`AmountSigned`): positive = issuance, negative = burn.

**`TxKernelAssetCreate`** — adds `m_MetaData` (`Asset::Metadata`, up to 16 KB). The metadata hash is included in the signing message, making it immutable after asset creation. See [Confidential Assets](Confidential-assets).

**`TxKernelAssetDestroy`** — adds `m_AssetID` and `m_Deposit` for the optional refundable deposit return.

### TxKernelShieldedOutput

```cpp
struct TxKernelShieldedOutput : public TxKernelNonStd {
    ShieldedTxo m_Txo; // commitment + Bulletproof + Ticket (blinded serial number)
};
```

Must be nested inside a `TxKernelStd` to prevent the receiver from surgically removing it from the transaction pool. See [Lelantus MW](Lelantus-MW) for the full shielded protocol.

### TxKernelShieldedInput

```cpp
struct TxKernelShieldedInput : public TxKernelNonStd {
    TxoID           m_WindowEnd;   // 1 past the last shielded output in anonymity set
    Lelantus::Proof m_SpendProof;  // one-out-of-many spend proof
    Asset::Proof::Ptr m_pAsset;    // CA asset proof (if spending a CA shielded coin)
};
```

The anonymity set is the range `[0, m_WindowEnd)` in the shielded pool. The spend proof does not reveal which element is being spent. See [Lelantus MW](Lelantus-MW) for proof construction and anonymity set sizing.

### TxKernelContractControl (base for BVM kernels)

```cpp
struct TxKernelContractControl : public TxKernelNonStd {
    ECC::Point   m_Commitment;  // blinding factor + net funds moved by contract
    ECC::Signature m_Signature; // aggregated multi-sig: blinding key + contract keys
    ByteBuffer   m_Args;        // serialized method arguments
    bool         m_Dependent;   // if true, valid only in a specific transaction context
};
```

`m_Commitment` aggregates all value movements performed by the contract call (funds locked, unlocked, or emitted), ensuring the balance equation holds across the entire transaction including contract-side effects.

**`TxKernelContractCreate`** — adds `m_Data` (`ByteBuffer`): the compiled WASM bytecode. The contract ID (`ContractID`) is derived from `Blake2b(m_Data)`.

**`TxKernelContractInvoke`** — adds `m_Cid` (256-bit contract ID) and `m_iMethod` (zero-based method index).

**`TxKernelEvmInvoke`** — adds `m_From`, `m_To` (20-byte Ethereum-style addresses), `m_Nonce` (`uint64_t`), `m_CallValue` (value in wei as a 256-bit word), and `m_Subsidy` (Beam-side fund transfer in Groth, signed).

---

## Transaction

```cpp
struct Transaction : public TxBase, public TxVectors::Full {
    // From TxBase:
    ECC::Scalar m_Offset;  // blinding factor offset (sum of all per-party offsets)

    // From TxVectors::Full:
    std::vector<Input::Ptr>    m_vInputs;   // sorted ascending
    std::vector<Output::Ptr>   m_vOutputs;  // sorted ascending
    std::vector<TxKernel::Ptr> m_vKernels;  // sorted ascending
};
```

All three vectors must be in canonical sort order before validation. The `Normalize()` / `NormalizeP()` / `NormalizeE()` methods enforce this and also perform cut-through (removing outputs that are simultaneously consumed as inputs).

The distinction between `Perishable` (inputs + outputs) and `Eternal` (kernels) reflects Beam's pruning model: inputs and outputs can be cut through once both appear in the same chain segment, but kernels are retained permanently in the kernel MMR to prove the transaction history is valid.

### Balance Equation

The core Mimblewimble invariant: a transaction is balanced if and only if the following sum equals the elliptic curve identity:

```
Σ = Σ(output.Commitment)
  - Σ(input.Commitment)
  + Σ(kernel.m_Commitment)     ← TxKernelStd excess keys
  + m_Offset · G
  + Σ(kernel.m_Fee) · H        ← fees treated as burned outputs
= 0
```

This proves no value was created or destroyed. The fees are treated as implicit outputs in a transaction context; the miner collects them via an explicit coinbase output in the block.

**Block balance:** Fees must already be reflected in explicit UTXO outputs (collected by the miner), and the block subsidy is the only legitimate source of new coins:

```
Σ = Σ(output.Commitment) - Σ(input.Commitment)
  + Σ(kernel.m_Commitment) + m_Offset · G
  - subsidy · H                ← single valid source of new value
= 0
```

---

## Fee Model

`Transaction::FeeSettings` (height-dependent, retrieved via `FeeSettings::get(height)`) defines the minimum fee structure:

| Setting | Covers |
|---|---|
| `m_Output` | Fee per UTXO output created |
| `m_Kernel` | Fee per kernel (nested kernels counted individually) |
| `m_ShieldedInputTotal` | Fee for one shielded input (includes implicit kernel overhead) |
| `m_ShieldedOutputTotal` | Fee for one shielded output |
| `m_Default` | Minimum fee for a standard BEAM transfer |
| `m_Bvm.m_ChargeUnitPrice` | Price per BVM charge unit consumed by contract execution |
| `m_Bvm.m_Minimum` | Minimum fee for any contract kernel |
| `m_Bvm.m_ExtraBytePrice` | Price per byte of `m_Args` / `m_Data` above the free byte threshold |

---

## Context-Free Validation

The node validates both transactions and blocks through `TxBase::Context::ValidateAndSummarizeStrict()`. Key checks in order:

1. **Fork gating** — The context's height range is constrained to a single fork era. Mixed-fork transactions are rejected.

2. **Sorted order** — Inputs, outputs, and kernels must be in canonical ascending order (`Fail_Order()` on violation). Within an iteration, the code also checks that no output matches an input commitment (cut-through in the pool is rejected as `"dup out"`).

3. **Output range proofs** — Each output's Bulletproof or public signature is verified via `Output::IsValid()`. Missing range proofs fail with `"Missing rangeproof"`. Unsigned outputs are only permitted in macroblock mode.

4. **Kernel validation** — `TxKernel::TestValid()` is called for each kernel. For `TxKernelStd` this verifies the Schnorr signature against the computed kernel ID. The effective height range is intersected into the validation context; if the intersection becomes empty, `"Height mismatch"` is thrown.

5. **Coinbase accounting** — Coinbase outputs are only allowed in blocks (not raw transactions). Their sum must equal `rules.get_Emission(height)`. PBFT blocks reject all coinbase outputs.

6. **Sigma check** — `TestSigma()` asserts `m_Sigma == 0` after accumulating all commitments plus the offset and fees. Any non-zero result means the balance equation failed.

---

## Serialization

Beam uses a compact binary serialization library (`yas`) accessed via `serialize(Archive& ar)` template methods. Elements are serialized in field declaration order with no type tags — the schema is entirely compile-time.

The `TxBase::IReader` / `IWriter` streaming interfaces allow iterating over transaction elements without loading the entire data set into memory. This is used during block merging and initial block download:

```cpp
struct TxBase::IReader {
    const Input*    m_pUtxoIn;
    const Output*   m_pUtxoOut;
    const TxKernel* m_pKernel;

    virtual void NextUtxoIn()  = 0;
    virtual void NextUtxoOut() = 0;
    virtual void NextKernel()  = 0;
};
```

`TxVectors::Reader` implements `IReader` for in-memory `Transaction` objects. `IWriter::Combine()` performs a streaming merge-sort of two readers and deletes cut-through pairs on the fly.

`block_rw.h` adds higher-level file-based constructs: `RecoveryInfo` (chain recovery export with UTXO proofs), `KeyString` (wallet key export in encrypted form), and `BlobEncoder` (AES-CBC envelope for key files).

---

## Summary of Invariants

| Property | Rule |
|---|---|
| Element ordering | All vectors sorted before validation |
| Kernel uniqueness | No duplicate kernel IDs anywhere in chain history |
| Balance | `Σ = 0` for both transactions and blocks |
| Coinbase | Only in PoW blocks; amount = block emission; requires public range proof |
| Kernel nesting depth | Maximum depth 2 |
| Fork gates | `m_CanEmbed` requires Fork1; shielded + CA kernels require Fork2; contract kernels require Fork3 |
| Kernel lifetime cap | `effective_max = min(m_Height.m_Max, m_Height.m_Min + MaxKernelValidityDH)` past Fork2 |
