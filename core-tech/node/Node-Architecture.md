# Node Architecture

This page documents the internals of the Beam full node: how blocks are applied and rolled back, how the transaction pool operates, how the database is organized, and how initial synchronization works. For the P2P wire protocol and peer management see [Node-P2P-Protocol](Node-P2P-Protocol.md). For the UTXO horizon and cut-through mechanics see [Core Block and Chain State](../core/Core-Block-And-Chain-State.md#horizon-based-history-pruning-and-sparse-synchronization).

---

## High-Level Component Map

```
┌──────────────────────────────────────────────┐
│                    Node                      │
│  ┌────────────┐  ┌─────────┐  ┌───────────┐  │
│  │NodeProcessor│ │ TxPool  │  │  PeerMan  │  │
│  │  (chain)   │  │(mempool)│  │  (P2P)    │  │
│  └─────┬──────┘  └────┬────┘  └─────┬─────┘  │
│        │              │             │        │
│  ┌─────▼──────┐  ┌────▼────┐        │        │
│  │  NodeDB    │  │Dandelion│        │        │
│  │ (SQLite)   │  │(stem)   │        │        │
│  └─────┬──────┘  └─────────┘        │        │
│        │                            │        │
│  ┌─────▼──────┐                     │        │
│  │   Mapped   │  (memory-mapped     │        │
│  │ UTXO tree  │   radix trees)      │        │
│  │ Contract   │                     │        │
│  │   tree     │                     │        │
│  └────────────┘                     │        │
└──────────────────────────────────────────────┘
```

The `Node` struct owns a `NodeProcessor` (chain state machine), `TxPool::Fluff` (mempool), `TxPool::Dependent` (contract tx chains), a `Dandelion` (stem-phase router, extends `TxPool::Stem`), and a `PeerMan` (peer rating and connection manager).

---

## NodeProcessor

`NodeProcessor` (`node/processor.h`) is the chain state machine. It owns two persistent stores — `NodeDB` (SQLite) and `Mapped` (memory-mapped files) — and drives the canonical chain forward or backward.

### Cursor and Extra

```cpp
struct Cursor {
    Block::SystemState::Full m_Full;   // full header of best tip
    HeightHash               m_hh;     // height + hash
    uint64_t                 m_Row;    // DB row of tip
    Merkle::Hash             m_History;
    Merkle::Hash             m_HistoryNext;
    Difficulty               m_DifficultyNext;
    StateExtra::Full         m_StateExtra; // total offset, CSA hash, logs hash
    Merkle::Hash             m_hvKernels;
} m_Cursor;

struct Extra {
    TxoID        m_TxosTreasury; // TXOs created by treasury (0 until treasury applied)
    TxoID        m_Txos;         // total TXOs ever created
    Block::Number m_Fossil;      // below this: original blocks erased
    Block::Number m_TxoLo;       // below this: TXOs fully erased
    Block::Number m_TxoHi;       // below this: TXOs compacted (commitment only)
    TxoID        m_ShieldedOutputs;
} m_Extra;
```

### Block Application

Incoming data enters through:

| Method | Purpose |
|--------|---------|
| `OnState(Full, PeerID)` | Receive and store a block header |
| `OnBlock(StateID, bbP, bbE, PeerID)` | Receive a block body (perishable + eternal parts) |
| `OnTreasury(Blob)` | Apply the genesis treasury (first run only) |

`DataStatus::Enum` is the return value:

| Value | Meaning |
|-------|---------|
| `Accepted` | New and usable |
| `Rejected` | Duplicate or irrelevant |
| `Invalid` | Cryptographically invalid |
| `Unreachable` | Below the lo-horizon |

After receiving new data, the node calls `TryGoUp()`, which walks the highest-chainwork branch and applies blocks via `HandleBlockInternal()`. Block body data is split into two blobs:

- **Perishable** (`bbP`) — outputs with bulletproofs; discarded below `TxoHi`
- **Eternal** (`bbE`) — kernels and inputs; kept indefinitely

### Rollback

```cpp
void RollbackTo(Block::Number);          // internal, used by fast-sync failure
void ManualRollbackTo(Block::Number);    // CLI-initiated
```

Rollback data (`pRB` stored per state in `NodeDB`) records which UTXOs to restore. `RollbackTo` walks backward through `m_RecentStates`, unspends inputs, removes outputs, and updates the UTXO radix tree. The artificial rollback cap is `m_Cfg.m_RollbackLimit.m_Max` (default 60 blocks); deeper rollbacks require a timeout (`m_TimeoutSinceTip_s = 3600 s`) since the last known tip.

### Horizon Pruning

`NodeProcessor::Horizon` encodes three cut-off boundaries:

```cpp
struct Horizon {
    Height m_Branching;          // branches behind this are pruned

    struct m_Schwarzschild {
        Height Lo;               // spent TXOs fully erased below this
        Height Hi;               // spent TXOs compacted (naked) below this
    };

    m_Schwarzschild m_Sync;      // used during fast sync
    m_Schwarzschild m_Local;     // used for local storage

    void SetInfinite();          // full archive node
    void SetStdFastSync();       // Hi = minimum, Lo = ~180 days
};
```

`PruneOld()` / `RaiseFossil()` / `RaiseTxoLo()` / `RaiseTxoHi()` advance these boundaries periodically:

- Below `TxoHi`: output commitment kept, bulletproof dropped (`TxoNaked` form, ~32 bytes vs ~700 bytes)
- Below `TxoLo`: output entry fully deleted from the TXO table
- Below `Fossil`: original block bodies deleted from `NodeDB`

See [Core Block and Chain State — Horizons](../core/Core-Block-And-Chain-State.md#horizon-based-history-pruning-and-sparse-synchronization) for the full semantics.

### Mapped Files (UTXO and Contract Radix Trees)

`NodeProcessor::Mapped` wraps two memory-mapped radix trees:

| Tree | Type | Purpose |
|------|------|---------|
| `m_Utxo` | `UtxoTree` | Set of all unspent TXO commitments |
| `m_Contract` | `RadixHashOnlyTree` | Set of all BVM contract key-value hashes |

Both trees are backed by a `.map` file alongside the SQLite database. On startup, the node checks `MappingStamp` (stored in `NodeDB::Params`) to detect whether the map is consistent with the DB; if not, it rebuilds the trees from the TXO table. `CommitMappingAndDB()` atomically flushes both the SQLite transaction and the map file.

---

## NodeDB Schema

`NodeDB` (`node/db.h`) is a SQLite database (one file, default `node.db`). All tables are accessed through prepared statements enumerated in `Query::Enum`. The major logical tables:

### States

Every known block header (not just the active chain) gets a row. Columns include:

| Field | Description |
|-------|-------------|
| `Number` | Block number (height) |
| `HashPrev` | Previous block hash |
| `ChainWork` | Cumulative proof-of-work |
| `Flags` | `Functional` \| `Reachable` \| `Active` |
| `Body (P, E, RB)` | Block body: perishable, eternal, rollback blobs |
| `Inputs` | Serialized `StateInput[]` array (commitment + TxoID) |
| `Txos` | TxoID at state end (total outputs created through this block) |
| `Extra` | `StateExtra::Full` (total blinding offset, CSA hash, logs hash) |
| `Peer` | PeerID that delivered this block |

State lifecycle flags:

```cpp
struct StateFlags {
    static const uint32_t Functional = 0x1; // block body is present
    static const uint32_t Reachable  = 0x2; // all ancestors up to genesis are functional
    static const uint32_t Active     = 0x4; // part of the current best chain
};
```

### Params

Key/value store for node-wide scalars:

| ParamID | Contents |
|---------|---------|
| `CursorRow` / `CursorNumber` | Active chain tip |
| `NumberFossil` | Fossil horizon height |
| `NumberTxoLo` / `NumberTxoHi` | TXO horizon heights |
| `SyncData` | Fast-sync state blob (`SyncData` struct) |
| `MappingStamp` | Hash used to validate the memory-mapped trees |
| `Treasury` | Raw treasury blob (first run only) |
| `MyID` | Node's own PeerID |
| `AidMax` | Highest active Confidential Asset ID |
| `PbftCid` / `PbftStamp` | PBFT contract ID and epoch stamp |

### TXOs

The `Txos` table stores every output ever created (indexed by `TxoID`, a monotonically increasing 64-bit counter). Each row holds the serialized output blob and the spend height (null if unspent). Below `TxoHi`, the blob is replaced with the naked (commitment-only) form; below `TxoLo`, the row is deleted.

### Kernels

Kernel blobs indexed by `(height, blob-hash)`. Used for proof-of-kernel-inclusion queries from lite clients.

### BBS (Secure Bulletin Board)

SBBS messages stored with `(Key, Channel, TimePosted, Message, Nonce)`. TTL defaults to 12 hours (`m_MessageTimeout_s`). Cleaned up periodically; storage capped at `m_Limit.m_Count` messages / `m_Limit.m_Size` bytes (default 20 M messages / 5 GB).

### Peers

Known peer addresses persisted across restarts: `(PeerID, Rating, Address, LastSeen)`. Flushed every `m_Timeout.m_PeersDbFlush_ms` (1 minute).

### Events and Accounts

Owner-key-scanned wallet events (`Events` table: per-account, per-height, keyed blob). Multiple accounts can be registered (`Accounts` table: each with an owner `Key::IPKdf`, serif, and `TxoHi` watermark).

### Contract Storage and Logs

BVM contract key-value state (`ContractData` table) and event logs (`ContractLog` table), both keyed by contract ID. In rich-info mode (`RichContractInfo` param set), additional parsed information is stored in `KrnInfo`.

### Streams

Five append-only byte streams used by the MMR implementations:

| StreamType | Contents |
|-----------|---------|
| `StatesMmr` | States MMR leaves |
| `Shielded` | Shielded output commitments |
| `ShieldedMmr` | Shielded MMR leaves |
| `AssetsMmr` | Assets MMR leaves |
| `ShieldedState` | Shielded state hashes |

Each stream is stored as fixed-size records in a SQLite `Streams` table, with a simple LRU cache optimized for sequential append and root calculation.

---

## Transaction Pool

The node operates two transaction pool instances on `Node`:

```cpp
TxPool::Fluff     m_TxPool;      // public mempool
TxPool::Dependent m_TxDependent; // contract tx dependency chains
```

And a Dandelion stem router:

```cpp
struct Dandelion : public TxPool::Stem { … } m_Dandelion;
```

### TxPool::Fluff (Main Mempool)

Transactions in the fluff pool are stored with three concurrent indices:

| Index | Type | Key |
|-------|------|-----|
| `m_setTxs` | `TxSet` | Transaction kernel hash (`Transaction::KeyType`) |
| `m_setProfit` | `ProfitSet` | Fee / (size + BvmCharge) ratio — descending |
| `m_lstWaitFluff` / `m_lstOutdated` | `HistList` | Expiry height |

**States:**

| State | Meaning |
|-------|---------|
| `PreFluffed` | Received from stem phase, not yet broadcast |
| `Fluffed` | Publicly broadcast, eligible for block inclusion |
| `Outdated` | Past valid height range, excluded from new blocks |

**Fee-replacement:** a new transaction replaces an existing one only if it has a strictly higher profit score (`Profit::operator<`). There is no explicit RBF flag; the profit ordering ensures higher-fee transactions displace lower-fee ones for the same kernel slot.

**Cleanup:** `DeleteOutdated()` is called on each new block. Outdated transactions are evicted from `m_setProfit` but kept in `m_lstOutdated` until the pool size limit (`m_MaxPoolTransactions = 100,000`) requires reclamation.

### Dandelion++ (TxPool::Stem)

Dandelion++ ([paper](https://arxiv.org/abs/1805.11060)) is Beam's transaction privacy protocol. New transactions received from wallets enter the **stem phase** first.

**Stem phase parameters (from `Node::Config::Dandelion`):**

| Parameter | Default | Meaning |
|-----------|---------|---------|
| `m_FluffProbability` | `0x1999` (≈10%) | Per-hop probability of early fluffing |
| `m_dhStemConfirm` | 2 blocks | If not mined within this many blocks: auto-fluff |
| `m_AggregationTime_ms` | 10,000 ms | Window to aggregate multiple stem txs |
| `m_OutputsMin` | 5 | Minimum outputs before aggregation is accepted |
| `m_OutputsMax` | 40 | Maximum outputs before aggregation is forced |

**Stem lifecycle:**

1. Transaction arrives at `OnTransactionStem()`.
2. Node attempts `TryMerge()` — combines the new tx with an existing stem element (Mimblewimble cut-through, aggregating kernels and offsets).
3. After `m_AggregationTime_ms` the aggregated stem tx is forwarded to a single randomly selected peer (not broadcast).
4. At each hop, with probability `m_FluffProbability`, the transaction enters fluff phase instead of being forwarded further.
5. If the transaction is not confirmed within `m_dhStemConfirm` blocks, it is auto-fluffed (`SetState(x, Fluffed)`).

**Dummy outputs (decoys):** when the node has a miner key, it creates dummy outputs (stored in `NodeDB::Dummies`) and spends them as inputs to stem transactions via `AddDummyInputs()`. This enlarges the anonymity set even for single-output transactions. Dummy lifetime: `[m_DummyLifetimeLo, m_DummyLifetimeHi]` blocks (defaults: 720–10,080, i.e. ~12 h to ~7 days).

### TxPool::Dependent

Contract transactions can form dependency chains (a transaction depends on the output of a prior transaction in the same block). `TxPool::Dependent` tracks these as a tree:

```cpp
struct Element {
    Transaction::Ptr m_pValue;
    Element*         m_pParent;   // null for chain root
    Amount           m_Fee;       // cumulative fee (chain)
    uint32_t         m_BvmCharge; // cumulative BVM gas
    uint32_t         m_Size;      // cumulative size
    uint32_t         m_Depth;
    Merkle::Hash     m_Context;   // context hash linking to parent
    bool             m_Fluff;     // promote to fluff pool?
};

Element* m_pBest; // highest-fee chain tip
```

`m_pBest` is the tip of the highest cumulative-fee chain, which is what `GenerateNewBlock` prefers to include.

---

## Block Production

`NodeProcessor::GenerateNewBlock(BlockContext&)` assembles a new candidate block.

```cpp
struct BlockContext {
    TxPool::Fluff&                    m_TxPool;
    const TxPool::Dependent::Element* m_pParent; // optional contract chain

    Key::Index  m_SubIdx;  // miner sub-key index
    Key::IKdf&  m_Coin;    // key for coinbase output
    Key::IPKdf& m_Tag;     // owner key for tagging

    enum Mode { Assemble, Finalize, SinglePass } m_Mode;
    // ...
};
```

**Assembly sequence:**

1. **Treasury (first block only):** if `m_Extra.m_TxosTreasury == 0`, the treasury blob is applied, creating the initial set of UTXOs.
2. **Mempool sweep:** iterate `TxPool::Fluff::ProfitSet` from highest profit to lowest. For each transaction:
   - Validate context (`ValidateTxContextEx`): check inputs are unspent, shielded inputs not double-spent, BVM charge fits.
   - If the contract dependency chain (`m_pParent`) is set, include the chain's transactions first.
   - Apply cut-through within the block: matching input/output commitments cancel.
   - Stop when block size budget is exhausted.
3. **Coinbase:** create a single coinbase output using `m_Coin` KDF at sub-index `m_SubIdx`. The coinbase kernel carries the block reward + fees.
4. **Header fields:** set `Height`, `Prev` (previous state hash from `m_Cursor.m_History`), `Definition` (Merkle root of UTXO + kernel + log trees), `TimeStamp`, `PoW` (left zeroed for the external solver).

In **online mining mode** (`m_PreferOnlineMining = true` and owner key present), the node requests the coinbase UTXO from a connected wallet instead of creating it locally. If no wallet responds, the node falls back to offline coinbase creation. See [Node Mining Modes](Node-Mining-Modes.md) for the full discussion.

---

## Initial Block Download and Fast Sync

### Normal Sync

In normal (archive) sync, blocks are downloaded one by one via `OnBlock()` and applied in order through `TryGoUp()`. The node tracks pending download tasks in a `TaskSet` / `TaskList`, assigning header and block requests to peers by `TryAssignTask(Task&, Peer&)`. Timeouts: 5 s for headers (`m_GetState_ms`), 30 s for blocks (`m_GetBlock_ms`).

### Fast Sync

When a peer's chainwork is significantly higher, `NodeProcessor::EnumCongestions()` detects that the node is far behind and enters **fast-sync mode**.

**Fast-sync state (`SyncData`):**

```cpp
struct SyncData {
    NodeDB::StateID m_Target; // target block to sync to
    Block::Number   m_n0;     // starting point (current tip at fast-sync entry)
    Block::Number   m_TxoLo;  // below this: sparse blocks only
    ECC::Point      m_Sigma;  // accumulated Pedersen sum for verification
};
```

**Protocol:**

1. A target block is selected (`m_Target`): `m_Horizon.m_Sync.Hi` blocks behind the peer's tip.
2. `m_TxoLo` is set to `target − m_Horizon.m_Sync.Lo` (e.g. ~180 days of blocks before target).
3. Blocks below `m_TxoLo` are downloaded as **sparse blocks**: no bulletproofs, inputs replaced by commitments only. Validation of individual outputs is deferred.
4. As sparse blocks arrive, `m_Sigma` accumulates the Pedersen sum of all inputs and outputs (via `MultiblockContext`).
5. Once all sparse blocks are downloaded (up to `m_TxoLo`), the node verifies the aggregate `m_Sigma` matches the committed state (`SyncData::m_Sigma` checked against the UTXO-set Merkle root).
6. Full blocks (with bulletproofs) are downloaded for `[m_TxoLo, m_Target]`.
7. After reaching `m_Target`, `OnFastSyncOver()` finalizes: runs `TryGoUp()` on the remaining full blocks, verifies the final definition hash.

**Failure handling:** any validation failure at either the lo-boundary or the final target calls `OnFastSyncFailed(bool bDeleteBlocks)`, which rolls the chain back to `m_n0` and clears `SyncData`. The node then retries with different peers.

`IsFastSync()` returns true while `m_SyncData.m_Target.m_Row != 0`.

---

## Observer Interface

`Node::IObserver` provides callbacks to the application layer:

| Callback | Trigger |
|---------|---------|
| `OnSyncProgress()` | `m_SyncStatus` changed (headers or blocks downloaded) |
| `OnStateChanged()` | New active tip applied |
| `OnRolledBack()` | Chain rolled back |
| `InitializeUtxosProgress(done, total)` | UTXO tree rebuild in progress |
| `OnSyncError(Error)` | Fatal sync error (e.g. `TimeDiffToLarge`) |

`Node::SyncStatus` tracks progress in weighted units (`s_WeightHdr = 1`, `s_WeightBlock = 8`) so that block download appears heavier than header download in progress bars.

---

## Key Configuration Parameters

| Config field | Default | Effect |
|---|---|---|
| `m_MaxConcurrentBlocksRequest` | 18 | Simultaneous block download tasks |
| `m_MaxPoolTransactions` | 100,000 | Fluff pool capacity |
| `m_MaxDeferredTransactions` | 100,000 | Deferred (pre-validation) tx queue |
| `m_MiningThreads` | 0 | Internal PoW solver threads (0 = disabled) |
| `m_VerificationThreads` | 0 | Block validation threads (negative = auto) |
| `m_RollbackLimit.m_Max` | 60 | Max automatic rollback depth |
| `m_Bbs.m_MessageTimeout_s` | 43,200 | SBBS message TTL (12 h) |
| `m_PreferOnlineMining` | true | Request coinbase from wallet; fall back to offline |

---

## Related Pages

- [Core Block and Chain State — Horizons](../core/Core-Block-And-Chain-State.md#horizon-based-history-pruning-and-sparse-synchronization) — detailed cut-through and horizon semantics
- [Core Block and Chain State](../core/Core-Block-And-Chain-State.md) — block header structure and system state
- [Node Mining Modes](Node-Mining-Modes.md) — integrated, OpenCL, stratum, and online/offline coinbase modes
- [Consensus BeamHash](../consensus/Consensus-BeamHash.md) — PoW algorithm and difficulty
- [Consensus Hard Forks](../consensus/Consensus-Hard-Forks.md) — fork-gated validation changes
