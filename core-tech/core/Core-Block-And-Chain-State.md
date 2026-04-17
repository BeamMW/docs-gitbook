# Core Block and Chain State

Reference documentation for Beam's block structure, system-state model, difficulty encoding, and the FlyClient-based chain-work proof. Source: `core/block_crypt.h`, `core/block_crypt.cpp`, `core/difficulty.h`, `core/chainwork.cpp`.

---

## Terminology

| Term | Definition |
|---|---|
| **System state** | The complete, valid state of the chain at a given height — fully defined by the live UTXO set, kernel set, and the inherited-states MMR. |
| **Block** | The transition between two consecutive system states. It contains a transaction (inputs, outputs, kernels) and a block header. |
| **Block number** | `Block::Number` — a `uint64_t` counter. Height 0 is the genesis (empty) state; height 1 is after the first block. |

The codebase avoids the phrase "block hash" in favour of "system state hash" because the hash belongs to the resulting state, not the block itself.

---

## Block Structures

### `Block::PoW`

The proof-of-work attached to each block header. For Beam this is a BeamHash (Equihash-based) solution.

```cpp
struct Block::PoW {
    static const uint32_t N = 150;
    static const uint32_t K = 5;
    static const uint32_t nNumIndices    = 1 << K;          // 32
    static const uint32_t nBitsPerIndex  = N / (K + 1) + 1; // 26
    static const uint32_t nSolutionBits  = nNumIndices * nBitsPerIndex; // 832
    static const uint32_t nSolutionBytes = nSolutionBits >> 3;         // 104

    std::array<uint8_t, nSolutionBytes> m_Indices; // sorted Equihash solution
    uintBig_t<8>                        m_Nonce;   // 8-byte nonce
    Difficulty                          m_Difficulty;
};
```

`m_Difficulty` is set before mining begins and cannot be adjusted post-hoc; the entire `PoW` struct (including the solution) is included in the system-state hash. This is a deliberate design choice: it makes it impossible to produce a valid chain of headers without actually mining them, which is a prerequisite for the FlyClient security argument.

The total PoW payload is 104 (indices) + 8 (nonce) + 4 (difficulty) = **116 bytes**.

For post-Fork-6 PBFT blocks the same memory layout (`sizeof(HdrData) == sizeof(PoW)`) is reused for the PBFT quorum certificate (`Block::Pbft::HdrData`), allowing unified serialization.

---

## SystemState Structures

### `Block::SystemState::ID`

A compact identifier for a known state: the state hash and its block number.

```cpp
struct Block::SystemState::ID {
    Merkle::Hash m_Hash;
    Block::Number m_Number;
};
```

`m_Number` is redundant (the hash alone is unique) but included for practical convenience when performing lookups.

### `Block::SystemState::Sequence::Prefix`

The three fields of a full state header that can be derived from the preceding header when headers are transmitted as a contiguous sequence:

```cpp
struct Sequence::Prefix {
    Block::Number    m_Number;    // block number (monotonically increasing)
    Merkle::Hash     m_Prev;      // hash of the previous system state
    Difficulty::Raw  m_ChainWork; // cumulative chainwork including this block
};
```

When sending a run of consecutive headers, `m_Prev`, `m_Number`, and `m_ChainWork` are omitted from all but the first header to save bandwidth. The receiving side reconstructs them from the preceding `Prefix`.

### `Block::SystemState::Sequence::Element`

The fields unique to each state in a sequence:

```cpp
struct Sequence::Element {
    Merkle::Hash m_Kernels;    // Before Fork3: kernel root; after Fork3: UTXO root
    Merkle::Hash m_Definition; // System Definition Hash (see below)
    Timestamp    m_TimeStamp;
    PoW          m_PoW;        // proof-of-work (or PBFT QC after Fork6)
};
```

### `Block::SystemState::Full`

Inherits both `Sequence::Prefix` and `Sequence::Element`, combining them into a complete standalone header:

```cpp
struct Full : public Sequence::Prefix, public Sequence::Element {
    void get_HashForPoW(Merkle::Hash&) const; // hash of everything except PoW solution
    void get_Hash(Merkle::Hash&) const;       // hash of all fields including PoW
    Height get_Height() const;
    bool IsValidPoW() const;
    bool IsValid() const;  // IsSane() && IsValidPoW()
};
```

`get_HashForPoW` is the input fed to the Equihash solver. `get_Hash` produces the system-state hash that appears as `m_Prev` in the next header.

**Proof verification methods** on `Full`:

| Method | What it proves |
|---|---|
| `IsValidProofState(id, hard_proof)` | The state `id` is an ancestor of `this` (uses the inherited-states DMMR) |
| `IsValidProofKernel(id, proof)` | A kernel with the given ID exists in `this` state |
| `IsValidProofUtxo(commitment, proof)` | A UTXO with the given commitment exists in `this` state |
| `IsValidProofShieldedOutp/Inp(desc, proof)` | A shielded output/input exists in the shielded pool |
| `IsValidProofAsset(asset, proof)` | A Confidential Asset is registered in `this` state |
| `IsValidProofContract(key, val, proof)` | A contract variable has a given value in `this` state |
| `IsValidProofLog(hash, proof)` | A contract log entry exists in `this` state |

All these use the `m_Definition` field as the trusted root; no full node is needed.

---

## System Definition Hash

`m_Definition` is the single hash that commits to the entire live state. Its formula has evolved across forks:

| Era | Formula |
|---|---|
| Before Fork2 | `Definition = Hash( History \| Utxos )` |
| Fork2 – Fork3 | `Definition = Hash( History \| Hash( Utxos \| Hash( Shielded \| Assets ) ) )` |
| Fork3 onward | `Definition = Hash( History \| Live )` where `Live = Hash( KL \| CSA )`, `KL = Hash( Kernels \| Logs )`, `CSA = Hash( Contracts \| Hash( Shielded \| Assets ) )` |

`History` is the root of the inherited-states DMMR. `Utxos`, `Kernels`, `Shielded`, `Assets`, `Contracts`, `Logs` are roots of their respective Radix hash trees.

The `SystemState::Evaluator` class implements this formula. It is subclassed by both the full-node block processor (which supplies the live tree roots) and by proof verifiers (which receive the hashes from proofs).

```cpp
struct SystemState::Evaluator : public Merkle::IEvaluator {
    Height m_Height;

    bool get_Definition(Merkle::Hash&); // computes the full Definition hash
    void GenerateProof();               // proof-generation variant

    virtual bool get_History(Merkle::Hash&);
    virtual bool get_Utxos(Merkle::Hash&);
    virtual bool get_Kernels(Merkle::Hash&);
    virtual bool get_Shielded(Merkle::Hash&);
    virtual bool get_Assets(Merkle::Hash&);
    virtual bool get_Contracts(Merkle::Hash&);
    virtual bool get_Logs(Merkle::Hash&);
};
```

The `m_Height` field is used to branch on the appropriate formula for the active fork.

---

## Difficulty Encoding

### `Difficulty` (compact form)

`Difficulty` stores a 32-bit packed value (`m_Packed`) representing a floating-point difficulty target.

```cpp
struct Difficulty {
    uint32_t m_Packed;
    static const uint32_t s_MantissaBits = 24;
    // exponent occupies the remaining 8 bits (s_MaxOrder = 231)

    typedef ECC::uintBig Raw;  // 256-bit big-endian cumulative work

    void Unpack(Raw&) const;         // expand to 256-bit work value
    void Unpack(uint32_t& order, uint32_t& mantissa) const;
    void Pack(uint32_t order, uint32_t mantissa);
    bool IsTargetReached(const ECC::uintBig& hash) const;
    void Calculate(const Raw& wrk, uint32_t dh, uint32_t dtTrg_s, uint32_t dtSrc_s);
    double ToFloat() const;
};
```

The encoding is: `m_Packed = (exponent << 24) | mantissa_24bit`. The target hash value a miner must beat is approximately `2^(256 - exponent) / mantissa`. The special value `s_Inf` represents maximum difficulty (only the zero hash passes).

`Difficulty::Raw` is a 256-bit type (`ECC::uintBig`) that holds cumulative chainwork. Arithmetic operators (`+`, `-`, `+=`, `-=`) that take a `Difficulty` unpack it and apply it to a `Raw` accumulator.

### Retargeting

The `Rules::DA` (difficulty adjustment) parameters control retargeting:

```cpp
struct Rules::DA {
    uint32_t Target_ms;      // target block time in milliseconds
    uint32_t WindowWork;     // number of blocks in the work window
    uint32_t WindowMedian0;  // inner window for median timestamp
    uint32_t WindowMedian1;  // outer window for median timestamp
    Difficulty Difficulty0;  // genesis difficulty
    struct { uint32_t M, N; } Damp; // damp fraction M/N toward expected dt
};
```

`Difficulty::Calculate(wrk, dh, dtTrg_s, dtSrc_s)` computes the next difficulty from the total work done over `dh` blocks in `dtSrc_s` seconds, targeting `dtTrg_s` seconds. The damp factor smooths sudden swings.

---

## ChainWorkProof (FlyClient Protocol)

`Block::ChainWorkProof` implements a probabilistic proof of total chainwork, based on the [FlyClient](https://eprint.iacr.org/2019/226) protocol by Luu, Bünz, and Zamani.

### Problem

A light client cannot download all headers. The ChainWorkProof convinces a verifier that the claimed chain has accumulated at least a given amount of cumulative work, with negligible probability of forgery.

### Security Parameters

- Assumed attacker fraction: < 2/3 of total hashrate (40% of overall power).
- Target forgery probability: ≈ 2^−60 (≈ 10^−18).
- Minimum samples per suffix: N = ceil(60 × ln 2 / (ln 3 − ln 2)) = **103**.

### Sampling Strategy

The prover works backwards from the chain tip:

1. Compute `range = 1/103` of the remaining chainwork from tip to lower bound.
2. Pick a uniformly random point `d` within that range (using a Fiat-Shamir oracle seeded from the tip hash).
3. Find the block whose cumulative chainwork interval contains `d`.
4. Include that block's header and a Merkle proof for it in the proof.
5. Cut off the range below `d` and repeat until the lower bound is reached.

The proof is generated in reverse so it can be **cropped** without rebuilding — the full proof is computed once, and a truncated version is sent to each client depending on how much chain state it already knows.

### Proof Structure

```cpp
struct Block::ChainWorkProof {
    struct {
        SystemState::Sequence::Prefix   m_Prefix;   // oldest header in contiguous run
        std::vector<Sequence::Element>  m_vElements;// recent headers (tip first)
    } m_Heading;

    std::vector<SystemState::Full> m_vArbitraryStates; // sampled non-contiguous headers
    Merkle::MultiProof             m_Proof;            // merged Merkle proofs
    Merkle::Hash                   m_hvRootLive;       // live-state root from tip
    Difficulty::Raw                m_LowerBound;       // minimum chainwork to prove
};
```

`m_Heading` encodes the most recent run of consecutive headers compactly (sharing the `Prefix`). Earlier sampled headers are in `m_vArbitraryStates`. All Merkle proofs are merged into the single `m_Proof` via `MultiProof`.

`IsValid()` drives the verifier: it replicates the Fiat-Shamir sampling and checks that each sampled block's proof verifies against the tip's `m_Definition`.

`Crop(src, lower_bound)` produces a shorter proof covering only the chainwork above `lower_bound`, enabling incremental syncing.

---

## ChainNavigator

`ChainNavigator` (`core/navigator.h`) is a memory-mapped branching log used by the node database to support chain reorganisations. It maintains a tree of **tags** and a doubly-linked list of **patches** attached to each tag.

```cpp
struct TagInfo {
    Height   m_Height;
    TagType  m_Tag;      // Merkle::Hash
};

struct TagMarker {
    TagInfo m_Diff;      // incremental change applied at this tag
    Links   m_Links;     // next/prev siblings
    Links   m_Patches;   // head/tail of patch list
    Offset  m_Child0;    // first child tag
    Offset  m_Parent;    // parent tag
};
```

- `MoveFwd(tag)` / `MoveBwd()` — advance or retreat along the current chain branch, applying or reverting patches.
- `CreateTag(info)` — fork the current position into a new branch.
- `DeleteTag(offset)` — remove a branch; its patches are re-parented to children.
- `Commit(patch)` — attach a new patch to the current tag.

The navigator provides the foundation for the node's ability to roll back blocks during a reorg and reapply them on a different fork, without keeping full copies of intermediate state.

---

## History Compression (Macroblocks)

Beam supports Mimblewimble-style history compression. All blocks between two system states can be merged into a single **macroblock** — a large transaction where spent outputs and the inputs that consumed them have been removed (cut-through).

Authenticity of the compressed history is verified entirely through `m_Definition`: the node applies the macroblock as if it were a normal block and confirms that the resulting system-state hash matches the expected `m_Definition` committed in the tip header. No additional Merkle proofs or witnesses are needed.

The node generates macroblocks incrementally using a cascade-merge (repeatedly halving and merging), but exposes only a single recent macroblock to syncing peers. Generation runs asynchronously and does not block normal block processing.

**Historical design note:** An earlier design called *cascade-merge* intended to expose a hierarchy of macroblocks to reduce download overhead for clients with varying history depths. This was abandoned because it imposed awkward download choices on the client. The current approach (a single macroblock generated roughly once a day) is simpler and sufficient.

---

## Horizon-Based History Pruning and Sparse Synchronization

### Horizons

A *horizon* denotes a relative distance in blocks from the current chain tip. Subtracting it from the current height gives the corresponding absolute height. Beam defines three horizons:

1. **Max-rollback distance** — fixed consensus parameter; 1 440 on mainnet (≈ 1 day). Defines the maximum depth of a chain reorganization. Blocks below this height are considered stable.
2. **Hi-Horizon** — how long a spent TXO is kept **fully** after spending. Below the corresponding height the node discards the bulletproof but retains the commitment (*Reduced TXO*).
3. **Lo-Horizon** — how long a *reduced* spent TXO is kept. Below the corresponding height the TXO is erased entirely.

The required ordering is:
```
Max-rollback-distance ≤ Hi-Horizon ≤ Lo-Horizon
```
or equivalently in heights (where Hi-Height = tip − Hi-Horizon):
```
Max-rollback-Height ≥ Hi-Height ≥ Lo-Height
```

Reduced TXOs retain only the commitment (~5% of a full UTXO size). Dropping the bulletproof has a dramatic effect on storage and bandwidth but means the TXO cannot be independently trusted as valid.

### Sparse Blocks

Beam supports *sparse blocks* — on-the-fly filtered versions of original blocks used during jump synchronization. When a node wants to jump from height `H0` to a new state, it requests sparse blocks specifying:

- Block identifier (height + hash)
- `H0` — the requesting node's current height
- `Lo-Height` and `Hi-Height` the node wants to achieve *after* the jump

The serving node generates sparse blocks by filtering inputs and outputs:

| Element | Rule |
|---|---|
| **Input** | Include if `SpendHeight > Lo-Height` OR `CreateHeight ≤ H0`; otherwise exclude |
| **Output** | Include full if `SpendHeight > Hi-Height`; include reduced if `SpendHeight > Lo-Height`; otherwise exclude |

Kernels are always included (they are not subject to cut-through).

### Pros and Cons

**Advantage:** Unlike earlier cut-through-from-genesis designs, sparse blocks support *arbitrary height jumps*. A node that has been offline for weeks downloads only the minimum information needed to reach the current state without downloading the full historical chain.

**Advantage:** The serving node requires no special preparation or extra storage to generate sparse blocks; with appropriate data structures the cost is nearly the same as retrieving the original blocks.

**Cost:** The serving node must retain reduced TXOs for a reasonable duration (the `Lo-Horizon` window). Reduced TXOs are dramatically smaller than full TXOs, so keeping a half-year backlog has negligible storage impact.

### Sparse Block Verification

During jump sync, verification is staged:

- **Per block:** Kernel validity; kernel MMR commitment vs. header; all inputs reference existing UTXOs; all non-reduced outputs have valid bulletproofs; UTXO set transitions are applied.
- **Per block ≥ Lo-Height (in addition):** UTXO set commitment matches header (important for DoS hardening — without this an attacker could inject fake inputs for later sparse block generation); overall arithmetic (sum of outputs − inputs = block subsidy).
- **Final check (after all sparse blocks):** No reduced UTXOs remain in the live UTXO set.

### DoS Attack Analysis

**Lo-Horizon problem:** Because TXOs below Lo-Height are erased entirely, blocks in that range cannot be verified individually. An attacker can modify inputs/outputs in those blocks without detection until all blocks down to Lo-Height have been downloaded. Mitigation: if the first sync attempt fails, the node retries without Lo-Horizon (downloading all commitments), making the attack infeasible because the verifier checks UTXO set commitment after every block.

**Hi-Horizon problem:** An attacker might include reduced (commitment-only) outputs in blocks where the node expects full outputs. The node cannot detect this until all blocks down to Hi-Height are downloaded. Mitigation: once detected, the node can identify the specific problematic blocks and their source peers, and re-download only those blocks without re-fetching kernels.

**Final strategy:** The node first attempts sync with Lo-Horizon. If it fails, it restarts without Lo-Horizon. During sync the node is in an *unreliable state* (proofs cannot be generated, tip not reported). Once all sparse blocks are downloaded and verified, the node switches to standard operation.

### Node Configurations

| Configuration | Hi-Horizon | Lo-Horizon | Description |
|---|---|---|---|
| **Archiving node** | ∞ | ∞ | Never deletes history; always performs comprehensive sync; can generate any sparse block on request |
| **Standard node** | 1 440 (= Max-rollback) | 1 440 × 180 (≈ half year) | Keeps recent history plus a half-year reduced-TXO backlog; supports sparse blocks for other standard nodes unless they were offline more than half a year |

### Cut-Through Mode Activation

Cut-through (jump sync) mode activates automatically when:

1. The node is not already in cut-through mode.
2. There is a *proven* state (all headers from genesis verified) with height at least `current height + Hi-Horizon × 1.5`.

When activated:
- `Target-Hi-Height = ProvenTip.Height − Hi-Horizon`
- `Target-Lo-Height = ProvenTip.Height − Lo-Horizon`

`Target-Lo-Height` is fixed for the duration of the sync. `Target-Hi-Height` can increase as the node sees a higher proven tip.

Once all sparse blocks are downloaded and pass verification, the node enters standard operation. If verification fails, all sparse blocks are discarded and the process restarts.

---

## UTXO and Kernel Proof Queries

### UTXO Proofs

The client requests a Merkle proof for a UTXO by specifying:
- **Commitment** — the EC point identifying the UTXO.
- **MaturityMin** (optional) — minimum maturity to query (default 0). Used to page through results when duplicate UTXOs exist.

The node response is an **array** of `(Maturity, Count, MerkleProof)` tuples. The client reconstructs the `UtxoLeaf.Hash` for each entry using:

```
UtxoLeaf.Hash = Hash(key_bits_zero_padded || m_Count)
```

where the 321-bit key is `Commitment || Maturity` zero-padded to the next byte. The Merkle proof is then verified against the UTXO tree root in `m_Definition`.

**Empty array** — the UTXO does not exist.

**Array size limit** — responses are capped at **20 elements**. If more `(Commitment, Maturity)` combinations exist (which can happen with intentionally duplicate UTXOs), the client must repeat the query with a higher `MaturityMin`.

For regular UTXOs (no intentional duplicates) the array contains at most one element.

### Kernel Proofs

The client sends a kernel ID and receives a single Merkle proof if the kernel exists in the current state. The kernel ID is the leaf hash in the `RadixHashOnlyTree` used for the kernel set.

### Why Kernel Proofs Matter

A missing UTXO proof (empty array) does **not** prove the UTXO was spent — it is impossible to prove non-existence of an element in the Merkle tree without presenting adjacent elements (a disproof). An empty result could mean the UTXO was spent, never existed, or is simply outside the queried maturity range.

A **kernel proof**, by contrast, proves with certainty that a specific transaction occurred and was included in a block. If the kernel ID corresponding to the spending transaction is present, the UTXO was definitely spent. Kernel proofs are therefore the definitive way to verify transaction inclusion, regardless of whether the UTXO outputs are received or spent.
