# Hi-Frequency Transactions (HFTX)

Hi-Frequency Transactions (HFTX) are **contract-driven, dependent transactions** that are cryptographically bound to a specific position in a block's transaction tree. Unlike ordinary Beam transactions — which float in the mempool for hours and can be included in any future block — HFTX must be included at an exact position relative to a parent context, or they are rejected. This makes HFTX suitable for DeFi flows where transaction ordering and the observable contract state matter.

Related pages: [Core-Transaction-Elements](Core-transaction-elements), [Consensus-Hard-Forks](Consensus-Hard-Forks), [Transactions-Creation-Protocol](Transactions-Creation-Protocol)

---

## Motivation

Standard Beam transactions are order-independent: the mempool is a flat set and miners choose transactions by fee/size ratio. This works well for simple transfers but breaks down in DeFi scenarios:

- A user queries an AMM contract for a price quote, then builds a swap transaction. By the time the swap is mined, earlier transactions may have shifted the pool price — the user is exposed to front-running and price impact.
- Two users build transactions that would both be valid in isolation but conflict when executed in the same order — for example, two claims that both drain a faucet.

HFTX solves these problems by introducing **dependent transactions**: each HFTX commits to a *parent context hash* that encodes the exact state of the contract-execution tree up to and including its predecessor. If the predecessor changes, the HFTX's signature is invalid and it is dropped.

---

## Architecture Overview

The dependent transaction mechanism has three layers:

| Layer | Component | Responsibility |
|---|---|---|
| Consensus | `TxKernelContractControl::m_Dependent` | Makes kernel ID commit to parent context |
| Node | `TxPool::Dependent` | Stores dependent txs in a tree; exposes the "best branch" |
| Wallet | `ContractTransaction` (state `RebuildHft`) | Monitors the tree; rebuilds and resubmits when the context changes |

---

## Contract Kernel Hierarchy

All contract operations use kernels that derive from `TxKernelContractControl` (`core/block_crypt.h:1437`). There are three concrete subtypes:

```cpp
struct TxKernelContractControl : public TxKernelNonStd {
    ECC::Point     m_Commitment;  // blinding factor + all funds consumed/emitted
    ECC::Signature m_Signature;   // aggregated multi-sig over all required keys
    ByteBuffer     m_Args;        // serialised method arguments
    bool           m_Dependent;   // if true, kernel ID commits to parent context
};

struct TxKernelContractCreate : public TxKernelContractControl {
    ByteBuffer m_Data;  // WASM bytecode of the new contract shader
};

struct TxKernelContractInvoke : public TxKernelContractControl {
    ContractID m_Cid;    // 32-byte contract identifier (hash of creation kernel)
    uint32_t   m_iMethod; // method index (0 = constructor, used only in Create)
};
```

### Kernel ID Derivation

The kernel ID is the hash produced by `HashSelfForID`, which serialises `m_Signature`. The *message* that is signed (from `HashSelfForMsg`) commits to:

- `m_Commitment`, `m_Args` (common to all contract kernels)
- For `ContractCreate`: the WASM data size and content
- For `ContractInvoke`: the contract ID and method index

When `m_Dependent = true` and the block height is past **Fork 4** (mainnet 1,820,000), the signing hash additionally includes the **parent context hash** `*pParentCtx`. This means the kernel's signature — and therefore its ID — changes if anything in the preceding execution chain changes:

```cpp
// block_crypt.cpp
void TxKernelContractControl::Prepare(ECC::Hash::Processor& hp, const Merkle::Hash* pParentCtx) const {
    hp << get_Msg();
    if (m_Dependent) {
        assert(pParentCtx);
        if (Rules::get().IsPastFork_<4>(m_Height.m_Min))
            hp << *pParentCtx;
    }
}
```

The parent context hash itself is computed by `DependentContext::get_Ancestor`:

```cpp
static void get_Ancestor(Merkle::Hash& hvRes,
                          const Merkle::Hash& hvParent,
                          const Merkle::Hash& hvTx)
{
    ECC::Hash::Processor() << "dep.tx" << hvParent << hvTx >> hvRes;
}
```

Each transaction in a branch extends the running context hash by hashing the previous context with the new transaction's kernel IDs. A dependent transaction submitted to the node carries the `m_ParentCtx` field set to the `(height, hash)` of the chain tip at which the parent context was computed.

### Fork Activation

Contract kernels require **Fork 3** (`TestForkAtLeast_<3>`, mainnet height 1,280,000). Dependent signing (the parent-context inclusion in the signing hash) is active past **Fork 4** (mainnet height 1,820,000). Contract kernels are rejected by consensus before these fork heights.

---

## InvokeData: From App Shader to Kernel

The wallet never directly constructs contract kernels. Instead, the app shader — a WASM program executed in the wallet's BVM sandbox — produces `ContractInvokeData` (`bvm/invoke_data.h`) which the wallet then turns into a kernel.

### ContractInvokeEntry

Each contract call is represented as a `ContractInvokeEntry`:

```cpp
struct ContractInvokeEntry {
    ContractID  m_Cid;       // contract to call (zero = Create)
    uint32_t    m_iMethod;   // method index
    ByteBuffer  m_Data;      // WASM shader bytecode (Create only)
    ByteBuffer  m_Args;      // serialised method arguments
    std::vector<ECC::Hash::Value> m_vSig; // required signing keys
    uint32_t    m_Charge;    // BVM charge budget consumed by this call
    HeightHash  m_ParentCtx; // dependent context (if Flags::Dependent set)
    FundsMap    m_Spend;     // net asset flows: positive = wallet pays, negative = wallet receives
    std::string m_sComment;  // human-readable description

    struct Flags {
        static const uint8_t Adv         = 1;    // advanced fields present
        static const uint8_t Dependent   = 2;    // tx is dependent (HFTX)
        static const uint8_t Multisigned = 8;    // multi-party signing required
        static const uint8_t HasCommitment = 0x10;
        static const uint8_t SaveAppInvoke  = 0x20;
        static const uint8_t SaveSpendMax   = 0x40;
    };
};
```

The `m_iMethod = 0` sentinel indicates a `ContractCreate` operation; any non-zero value maps to a `ContractInvoke`.

### ContractInvokeDataBase

Multiple calls may be batched into a single transaction by placing several `ContractInvokeEntry` objects into the `m_vec` vector of `ContractInvokeDataBase`:

```cpp
struct ContractInvokeDataBase {
    std::vector<ContractInvokeEntry> m_vec;  // one entry per kernel
    std::vector<ECC::Point>          m_vPeers;    // for multi-party signing
    FundsMap                         m_SpendExtra; // extra UTXO flows
    ECC::Hash::Value                 m_hvKey;
    bool                             m_IsSender = true;
};
```

Each entry in `m_vec` produces exactly one contract kernel. A transaction with three entries in `m_vec` therefore carries three `TxKernelContractControl`-derived kernels in one Beam transaction. All kernels in the batch pay a combined fee; the minimum fee is computed per entry (see [Fee Model](#fee-model)) and summed.

`ContractInvokeData` (the richer variant saved to the wallet database) extends `ContractInvokeDataBase` with the app-shader bytecode (`m_AppInvoke`) so the wallet can re-run the shader to rebuild the HFTX when the context changes.

---

## Dependent Transaction Pool

The node stores ordinary transactions in a flat pool ordered by fee/size. Dependent transactions live in a **separate tree** (`TxPool::Dependent`, `node/txpool.h:179`):

```cpp
struct Dependent {
    struct Element {
        Transaction::Ptr m_pValue;
        Element*         m_pParent;   // parent in the dependency tree

        // cumulative totals from root to this element
        Amount   m_Fee;
        uint32_t m_BvmCharge;
        uint32_t m_Size;
        uint32_t m_Depth;
    };

    Element* m_pBest;  // head of the branch with the highest cumulative fee
};
```

Each `Element` points to its parent, forming a tree rooted at the current chain tip. The node continuously tracks `m_pBest` — the leaf of the heaviest (by cumulative fee) branch. When a miner assembles a block it includes all transactions from the root to `m_pBest` in order, followed by transactions from the ordinary pool.

### Rejection Codes

When the node rejects a dependent transaction, it returns one of three status codes (from `core/proto.h:691`):

| Code | Meaning |
|---|---|
| `DependentNoParent` | The kernel's declared parent context was not found in the dependent tree |
| `DependentNotBest` | The transaction is valid but a competing branch has higher cumulative fee |
| `DependentNoNewCtx` | The new context hash is a duplicate — likely `m_Dependent` was not set on the kernel |

---

## ContractTransaction Wallet State Machine

The wallet manages HFTX lifecycle through the `ContractTransaction` class (`wallet/core/contract_transaction.h`). The state machine has these states:

| State | Description |
|---|---|
| `Initial` | Parameters received; building the transaction for the first time |
| `GeneratingCoins` | Allocating UTXOs for the fee / change |
| `Registration` | Transaction submitted to the node; awaiting confirmation |
| `OutputsConfirmation` | Kernels confirmed; verifying output UTXOs |
| `Negotiating` | Multi-party signing in progress |
| `RebuildHft` | Dependent context changed; re-running app shader to rebuild the transaction |

### HFT Rebuild Loop

When the dependent transaction tree changes (a competing branch has overtaken the current one), the node notifies the wallet via `OnDependentStateChanged()`. The wallet then:

1. Checks whether any previously submitted variant is still pending (`IsHftPending`).
2. If the context has changed, re-executes the app shader (`AppShaderExec::StartRun`) with `m_EnforceDependent = true` and the new parent context.
3. Builds a fresh transaction from the new `ContractInvokeData`, sets the correct `m_ParentCtx`, and resubmits.
4. Continues until a variant is confirmed in a block, or the transaction expires and `RetryHft` can no longer proceed.

This loop provides **pseudo-confirmation**: even before a block is mined, the wallet knows whether its transaction is in the leading branch.

---

## Fee Model

Contract kernel fees have two components:

**1. Structure fee** — covers the kernel's contribution to block size, computed from `TxStats`:

```
fee_structure = FeeSettings::Calculate(stats)
              + FeeSettings::CalculateForBvm(stats, m_Charge)
```

`stats.m_Contract++` is set for every contract kernel, and `m_ContractSizeExtra` accounts for the byte length of `m_Args` (and `m_Data` for creates).

**2. Charge fee** — covers BVM execution time. The block is budgeted at **100 million charge units** (`BlockCharge = 100 * 1000 * 1000` in `bvm2_cost.h`). Representative operation costs:

| Operation | Charge units |
|---|---|
| WASM CPU cycle | 5 |
| Heap alloc per byte | 2 |
| Load variable (base) | 5,000 |
| Load variable per byte | 50 |
| Save variable (base) | 20,000 |
| Save variable per byte | 100 |
| `AddSig` | 10,000 |
| `FundsLock` | 2,000 |
| `AssetEmit` | 5,000 |

The minimum fee for a contract entry (`ContractInvokeEntry::get_FeeMin`) is:

```
fee_min = max(Calculate(stats), DefaultStdFee) + CalculateForBvm(stats, m_Charge)
```

For a batched transaction with N entries, the total minimum fee is the sum of each entry's `get_FeeMin`.

---

## HFTX vs. Simple Transactions

| Criterion | Simple Transaction | HFTX |
|---|---|---|
| Ordering guarantee | None (mempool reordering) | Exact position in block |
| Front-running resistance | None | Strong (dependent context) |
| Round-trips required | 1–2 (interactive signing) | 0 (app-shader unilateral) |
| Rebuild on reorg | Manual retry | Automatic (`RebuildHft` state) |
| Fee | Fixed at creation | Minimum recalculated each rebuild |
| Applicable to | BEAM / CA transfers | Contract method calls |
| Wallet must be online | No (for offline / max-privacy) | Yes (to react to context changes) |

**Use HFTX when:**
- The transaction calls a contract that reads or writes on-chain state that may change between blocks (AMM swaps, auctions, order matching).
- Front-running protection is required.
- Multiple users' transactions need to be ordered in the same block with predictable effects.

**Use simple transactions when:**
- Transferring BEAM or a CA between wallets with no contract logic.
- The receiver is offline and cannot participate in interactive signing (use offline / public-offline address types).

---

## DApp API Relationship

App shaders that produce dependent `ContractInvokeData` can be invoked from the wallet API via `invoke_contract`. Methods marked with the DAPPs-allowed badge in the API docs may build HFTX internally when the app shader sets `Flags::Dependent` on one or more entries. The wallet API returns a transaction ID immediately; the caller should poll `tx_status` to track the `RebuildHft` rebuild cycles and eventual confirmation.

See [Beam-wallet-api-versioning](Beam-wallet-api-versioning) for how wallet API versions expose contract invocation methods.
