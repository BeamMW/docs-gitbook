# BVM Internals

The Beam Virtual Machine (BVM) is a sandboxed WASM interpreter embedded in the Beam node and wallet. It executes *shaders* — custom programs that implement on-chain contract logic and wallet-side application interfaces. BVM was introduced at Hard Fork 2 and has evolved through subsequent forks.

Related pages: [Transactions-Hi-Frequency](../transactions/Transactions-Hi-Frequency.md) · [Core-transaction-elements](../core/Core-transaction-elements.md) · [Consensus-Hard-Forks](../consensus/Consensus-Hard-Forks.md)

---

## Architecture Overview

The BVM is split across two C++ namespaces:

| Namespace | Role |
|---|---|
| `beam::Wasm` | Low-level WASM binary parser and stack-machine interpreter (`wasm_interpreter.h`) |
| `beam::bvm2` | BVM layer on top of WASM: storage, funds, signatures, host-function dispatch (`bvm2.h`) |

Shader bytecode is **compiled** from standard WASM into an internal linearized format by `Processor::Compiler::Compile()` before execution. The compiler resolves host-function bindings (imports) and strips unreachable code.

---

## Contract vs. App Shader (Execution Contexts)

The BVM has two execution contexts, selected at compile time and enforced at runtime:

```cpp
enum struct Kind {
    Contract,  // node-side: deterministic, consensus-critical
    Manager,   // wallet-side: read-only chain access, kernel generation
};
```

### Contract Shader (`ProcessorContract`)

Runs on every validating node when a `TxKernelContractControl` kernel is processed. Must be **deterministic** — identical inputs always produce identical state transitions on all nodes.

Constraints:
- No random number generation, no current-time access
- Floating-point operations not supported (FPU behavior differs across hardware)
- Memory is always initialized to the same state at entry
- Read/write access restricted to the contract's own key-value store
- Execution budget enforced via the charge counter (`m_Charge`)

### App Shader (`ProcessorManager`)

Runs inside the wallet process. Constructs transactions and presents information to the user. Not consensus-critical.

Capabilities beyond contract shaders:
- Read any contract's variables and logs (`Vars_Enum`, `Logs_Enum`)
- Access block headers and Merkle proofs (`VarGetProof`, `LogGetProof`)
- Derive and use user public keys (`DerivePk`, `get_Pk`)
- Generate random numbers (`GenerateRandom`)
- Emit JSON output for the wallet UI (`Doc*` functions)
- Build contract invocation kernels (`GenerateKernel`, `GenerateKernelAdvanced`)
- Communicate with other wallets via SBBS (`Comm_*`)

---

## WASM Subset and Memory Model

### Supported Types

`i32`, `i64` (primary). `f32`/`f64` are parsed but **not** executed in contract context — floating-point is prohibited for determinism reasons.

### Linear Memory Regions

The WASM address space is partitioned by the two most-significant bits of a 32-bit pointer:

| Tag bits | Region | Max size |
|---|---|---|
| `00` | Data (heap) | 1 MB (`Limits::HeapSize`) |
| `01` | Global variables | — |
| `10` | Operand/alias stack | 64 KB (`Limits::StackSize`) |

Boundary violations trap immediately.

### Stack Machine

`Wasm::Processor` maintains:
- **Operand stack** — `Word` (uint32) entries, with 64-bit values occupying two slots
- **Alias stack** — grows from the top of the stack region downward; used for function locals and `StackAlloc` calls
- **Call stack** tracked via `OnCall`/`OnRet` overrides

### Reader Modes

The WASM binary reader operates in one of four modes:

| Mode | Used when |
|---|---|
| `AutoWorkAround` | Compilation (resolves conflicting flags automatically) |
| `Restrict` | Transaction verification (fails on any non-standard construct) |
| `Emulate_x86` | Emulates historical x86/amd64 edge-case behavior |
| `Standard` | Strict WASM compliance — activated after Hard Fork 4 |

### Heap Allocator

`Processor::Heap` manages the data region with a two-index free-list (`MapSize` by block size, `MapPos` by address). The heap supports `Alloc`, `Free`, and `OnGrow`. Each far-call frame (`FarCalls::Frame`) carries its own heap snapshot.

---

## Processor Class Hierarchy

```
Wasm::Processor               — binary interpreter, stack machine
  └── bvm2::Processor         — adds heap, hash/scalar/point objects, storage interface
        ├── ProcessorContract — contract execution: storage R/W, funds, sigs, far-calls
        └── ProcessorManager  — app shader: chain reads, kernel generation, comms
```

### Key Limits

```cpp
struct Limits {
    static const uint32_t FarCallDepth = 32;
    static const uint32_t VarKeySize   = 256;   // bytes
    static const uint32_t StackSize    = 0x10000;  // 64 KB
    static const uint32_t HeapSize     = 0x100000; // 1 MB
    static const uint32_t HashObjects  = 8;   // concurrent hash state objects
    static const uint32_t SecScalars   = 16;
    static const uint32_t SecPoints    = 16;
};
```

---

## Host Function Opcodes

Host functions are imported into WASM under the BVM module name and dispatched via `InvokeExt(uint32_t opcode)`. The opcode table is defined in `bvm2_opcodes.h` using X-macros.

### Common Functions (both contexts)

| Opcode | Signature | Description |
|---|---|---|
| `0x05` | `Write(pData, nData, iStream)` | Output to stream 0 (stdout) or 1 (stderr) |
| `0x10` | `Memcpy(pDst, pSrc, size) → void*` | Memory copy |
| `0x11` | `Memset(pDst, val, size) → void*` | Memory fill |
| `0x12` | `Memcmp(p1, p2, size) → i32` | Memory compare |
| `0x13` | `Memis0(p, size) → u8` | Test all-zero |
| `0x14` | `Strlen(sz) → u32` | C-string length |
| `0x15` | `Strcmp(sz1, sz2) → i32` | C-string compare |
| `0x18` | `StackAlloc(size) → void*` | Alias-stack allocation |
| `0x19` | `StackFree(size)` | Alias-stack release |
| `0x1A` | `Heap_Alloc(size) → void*` | Heap allocation |
| `0x1B` | `Heap_Free(pPtr)` | Heap release |
| `0x20` | `LoadVar(pKey, nKey, pVal, nVal, nType) → u32` | Load contract variable |
| `0x21` | `SaveVar(pKey, nKey, pVal, nVal, nType) → u32` | Save contract variable |
| `0x27` | `LoadVarEx(pKey, &nKey, nKeyBufSize, pVal, &nVal, nType, nSearchFlag)` | Iterative load with key search |
| `0x28` | `Halt()` | Abort execution |
| `0x2A` | `get_AssetInfo(aid, &res, pMetadata, nMetadata) → u32` | Query asset metadata |
| `0x2B` | `HashWrite(pHash, p, size)` | Feed bytes to hash object |
| `0x2D` | `HashGetValue(pHash, pDst, size)` | Finalize hash |
| `0x2E` | `HashFree(pHash)` | Destroy hash object |
| `0x2F` | `HashClone(pHash) → HashObj*` | Clone hash state |
| `0x40` | `get_Height() → Height` | Current block height |
| `0x41` | `get_HdrInfo(&hdr)` | Block header (compact) |
| `0x42` | `get_HdrFull(&hdr)` | Block header (full, with PoW) |
| `0x43` | `get_RulesCfg(h, &res) → Height` | Consensus rules hash at height |
| `0x44` | `get_ForkHeight(iFork) → Height` | Activation height of fork N |
| `0x48` | `HashCreateSha256() → HashObj*` | Create SHA-256 context |
| `0x49` | `HashCreateBlake2b(pPersonal, nPersonal, nResultSize) → HashObj*` | Create Blake2b context |
| `0x4A` | `HashCreateKeccak(nBits) → HashObj*` | Create Keccak context |
| `0x80`–`0x88` | `Secp_Scalar_*` | secp256k1 scalar arithmetic |
| `0x90`–`0x9B` | `Secp_Point_*` | secp256k1 point arithmetic (import/export/add/mul/neg) |
| `0xB0` | `VerifyBeamHashIII(pInp, nInp, pNonce, nNonce, pSol, nSol) → u8` | Verify BeamHash III PoW solution |

### Contract-Only Functions

| Opcode | Signature | Description |
|---|---|---|
| `0x22` | `EmitLog(pKey, nKey, pVal, nVal, nType) → u32` | Append to the contract event log |
| `0x23` | `CallFar(cid, iMethod, pArgs, nArgs, nFlags)` | Cross-contract call |
| `0x24` | `get_CallDepth() → u32` | Current far-call nesting depth |
| `0x25` | `get_CallerCid(iCaller, &cid)` | CID of a caller frame |
| `0x26` | `UpdateShader(pVal, nVal)` | Replace the running contract's shader bytecode |
| `0x29` | `AddSig(pubKey)` | Demand a Schnorr signature on `pubKey` |
| `0x30` | `FundsLock(aid, amount)` | Lock funds into the contract |
| `0x31` | `FundsUnlock(aid, amount)` | Release funds from the contract |
| `0x32` | `RefAdd(cid) → u8` | Increment reference count on another contract |
| `0x33` | `RefRelease(cid) → u8` | Decrement reference count |
| `0x38` | `AssetCreate(pMeta, nMeta) → AssetID` | Register a new confidential asset |
| `0x39` | `AssetEmit(aid, amount, bEmit) → u8` | Mint or burn asset supply |
| `0x3A` | `AssetDestroy(aid) → u8` | Unregister an asset |

### App-Shader-Only Functions

| Opcode | Group | Description |
|---|---|---|
| `0x50` | Context | `SelectContext(bDependent, nCharge)` — set block context for kernel |
| `0x51`–`0x53` | Vars | `Vars_Enum` / `Vars_MoveNext` / `Vars_Close` — iterate contract variables |
| `0x54` | Proof | `VarGetProof` — variable value + Merkle inclusion proof |
| `0x55`–`0x57` | Logs | `Logs_Enum` / `Logs_MoveNext` / `Logs_Close` — iterate log entries |
| `0x58` | Proof | `LogGetProof` — log entry + Merkle inclusion proof |
| `0x5A` | Key | `DerivePk(pubKey, pID, nID)` — derive user public key |
| `0x5B`–`0x5D` | Assets | `Assets_Enum` / `Assets_MoveNext` / `Assets_Close` |
| `0x60`–`0x6C` | JSON | `DocAddGroup/Text/Num32/Num64/Array/Blob`, `DocGet*` — JSON output/input |
| `0x70` | Kernel | `GenerateKernel(pCid, iMethod, pArg, nArg, pFunds, nFunds, pSig, nSig, szComment, nCharge)` |
| `0x78`/`0x79` | API | `GetApiVersion` / `SetApiVersion` |
| `0xA0` | Random | `GenerateRandom(pBuf, nSize)` |
| `0xA1`–`0xA4` | Nonce | `get_SlotImage`, `SlotInit`, `get_Pk`, `get_BlindSk` — nonce slot management |
| `0xA5` | Kernel | `GenerateKernelAdvanced` — kernel with custom height range, multisig |
| `0xA9` | Multisig | `SetMultisignedTx` — mark kernel as requiring multiple signers |
| `0xB1`–`0xB4` | Comms | `Comm_Send`, `Comm_Read`, `Comm_WaitMsg`, `Comm_Listen` — SBBS messaging |
| `0xC0`/`0xC1` | Spend | `GetMaxSpend` / `SetMaxSpend` — declare maximum spend amounts |

---

## Charge Model (Gas)

Every contract execution begins with a charge counter:

```cpp
uint32_t m_Charge = Limits::BlockCharge;  // 100,000,000 units
```

Each host-function call and WASM instruction deducts from this budget. When it reaches zero, execution aborts with `ErrorSubType::NoCharge`. The budget is derived from a target block capacity — `ChargeFor<N>::V = BlockCharge / N` gives the charge for an operation that can run at most N times per block:

| Operation | Budget target | Charge per call |
|---|---|---|
| WASM cycle | 20M cycles/block | ~5 units |
| Memory op (per byte) | 50M bytes/block | ~2 units |
| `HeapOp` | 1M ops | ~100 units |
| `LoadVar` (base) | 20K loads | ~5,000 units |
| `LoadVar` (per byte) | 2M bytes | ~50 units |
| `SaveVar` (base) | 5K saves | ~20,000 units |
| `SaveVar` (per byte) | 1M bytes | ~100 units |
| `EmitLog` (base) | 20K logs | ~5,000 units |
| `EmitLog` (per byte) | 1M bytes | ~100 units |
| `UpdateShader` | 1K updates | ~100,000 units |
| `CallFar` | 10K calls | ~10,000 units |
| `AddSig` | 10K sigs | ~10,000 units |
| `AssetCreate/Destroy` | 1K ops | ~100,000 units |
| `AssetEmit` | 20K ops | ~5,000 units |
| `FundsLock` | 50K ops | ~2,000 units |
| `HashOp` | 1M hashes | ~100 units |
| `Secp_Point_Multiply` | 2K mults | ~50,000 units |

The caller specifies `nCharge` in `GenerateKernel` — this becomes the charge allocated to the kernel in the transaction. The node verifies the actual consumed charge does not exceed what was declared.

---

## Contract Storage

### Key Structure

All contract variables share a flat key-value store scoped per contract. The storage key is prefixed by the runtime before any `LoadVar`/`SaveVar` call:

```
key = ContractID (32 bytes) || tag (1 byte) || user_key (≤ 256 bytes)
```

**Built-in tag values** (`KeyTag`):

| Tag | Value | Meaning |
|---|---|---|
| `Internal` | `0` | General contract state |
| `InternalStealth` | `8` | Internal stealth variant |
| `LockedAmount` | `1` | Per-asset locked balance |
| `Refs` | `2` | Reference counts to other contracts |
| `OwnedAsset` | `3` | Assets registered by this contract |
| `ShaderChange` | `4` | Event emitted when shader code changes (post-HF4) |

### Variable Size Limits

| Era | Max value size |
|---|---|
| Pre-HF4 | 8 KB (`VarSize_0 = 0x2000`) |
| Post-HF4 | 1 MB (`VarSize_4 = 0x100000`) |

### Iterative Access

`LoadVarEx` supports prefix scan and nearest-key queries via `nSearchFlag`:

| Flag | Meaning |
|---|---|
| `KeySearchFlags::Exact` | Exact key match only |
| `KeySearchFlags::Bigger` | First key ≥ supplied key |

### Logs vs. Variables

Variables (`SaveVar`) are mutable contract state persisted in the UTXO set. Logs (`EmitLog`) are **immutable** append-only events anchored at a `HeightPos` (block height + tx position). Logs cannot be overwritten or deleted, but are provable via `LogGetProof`.

### Storage Interface

```cpp
struct Storage::IBase {
    virtual void LoadVar(const Blob&, Blob& res) = 0;
    virtual void LoadVarEx(Blob& key, Blob& res, bool bExact, bool bBigger) = 0;
    virtual uint32_t SaveVar(const Blob&, const Blob& val) = 0;
    virtual uint32_t OnLog(const Blob&, const Blob& val) = 0;
};
```

The node's `ProcessorContract` subclass wires these to the `NodeDB`.

---

## Cross-Contract Calls (Far Calls)

`CallFar` invokes a public method on another contract at the given `ContractID`. Each call pushes a `FarCalls::Frame` containing the callee's bytecode, a fresh heap snapshot, and the return address:

```cpp
struct FarCalls::Frame {
    ContractID m_Cid;
    ByteBuffer m_Body;       // callee shader bytecode
    Wasm::Word m_FarRetAddr;
    Heap       m_Heap;       // isolated heap for callee
    Blob       m_Args;
    uint32_t   m_Flags;
};
```

**Call flags** (`CallFarFlags`):

| Flag | Effect |
|---|---|
| `InheritContext` | Callee shares caller's storage context (analogous to `delegatecall`) |
| `SelfBlock` | Prevent recursive call back to the calling contract |
| `SelfLockRO` | Allow recursion but block all state modifications by self |
| `GlobalLockRO` | Block all modifications by any callee in the subtree (`staticcall` equivalent) |

Maximum call depth: **32** frames (`Limits::FarCallDepth`).

---

## Contract Identity

```cpp
// ShaderID = Blake2b(shader_bytecode)
void get_ShaderID(ShaderID&, const Blob& data);

// ContractID = Blake2b(ShaderID || constructor_args)
void get_Cid(ContractID&, const Blob& data, const Blob& args);
void get_CidViaSid(ContractID&, const ShaderID&, const Blob& args);
```

The same shader bytecode deployed with different constructor arguments produces different `ContractID`s. Two contracts with identical `ContractID`s are guaranteed to have identical shader code and identical constructor arguments.

---

## InvokeData: Serializing a Contract Call

The wallet builds invocations as `ContractInvokeData` objects before signing and broadcasting:

```cpp
struct ContractInvokeEntry {
    ContractID  m_Cid;       // target contract (zero for Create)
    uint32_t    m_iMethod;   // 0 = Create, N = method N, Destructor = last method
    ByteBuffer  m_Args;      // serialized method arguments
    ByteBuffer  m_Data;      // shader bytecode (Create only)
    FundsMap    m_Spend;     // expected fund flows (aid → signed amount)
    vector<ECC::Hash::Value> m_vSig;  // public keys that must sign
    uint32_t    m_Charge;    // gas budget for this invocation
    string      m_sComment;  // human-readable label
    uint32_t    m_Flags;
};

struct ContractInvokeData : ContractInvokeDataBase {
    AppInvokeData m_AppInvoke;  // app shader + contract shader + args + privilege level
    FundsMap      m_SpendMax;   // upper bound on funds the app may spend
};
```

**Flags on `ContractInvokeEntry`:**

| Flag | Meaning |
|---|---|
| `Adv` | Entry uses the `Advanced` sub-struct (height range, fee, explicit signature) |
| `Dependent` | Kernel depends on a specific parent block (`m_ParentCtx`) |
| `Multisigned` | Requires multi-party Schnorr signature |
| `HasCommitment` | Explicit commitment point is included |
| `SaveAppInvoke` | Persist app shader invocation data for replay |
| `SaveSpendMax` | Persist max-spend map |

`ContractInvokeEntry::Generate()` converts the entry into a `TxKernelContractControl` kernel and signs it using the provided `Key::IKdf`. For multisig flows, `GenerateAdv` collects partial signatures from peers via SBBS.

---

## Contract Lifecycle

### Creation

A `ContractCreate` kernel carries the shader bytecode and constructor args. The node:
1. Computes `ShaderID = Hash(bytecode)` and `ContractID = Hash(ShaderID || args)`
2. Verifies no contract with that ID already exists
3. Charges the minimum deployment fee
4. Calls public method 0 (constructor) via `ProcessorContract`
5. On success, stores the bytecode under the `SidCid` synthetic key

### Method Invocation

A `ContractInvoke` kernel carries `ContractID + method_index + args`. The node:
1. Loads the shader bytecode for the CID
2. Instantiates `ProcessorContract`, wires storage to `NodeDB`
3. Calls `CallMethod(iMethod)` 
4. Verifies the committed fund flows match what the shader actually locked/unlocked
5. Verifies all `AddSig`-requested public keys appear in the kernel's multi-signature

### Destruction

Destructor method must:
- Delete all custom variables the contract stored
- Ensure no locked funds remain
- Ensure no assets created by this contract remain active
- Ensure no other contracts hold a reference (via `RefAdd`) to this contract

---

## API Versioning

The BVM host-function set is versioned:

```cpp
struct ApiVersion {
    static const uint32_t Current = 2;
};
```

App shaders call `GetApiVersion()` to query which version the wallet supports, and `SetApiVersion(nVer)` to pin themselves to a specific set of available functions. This allows older app shaders to run on newer wallets without accidentally using unavailable opcodes.

---

## Determinism and Security

- **No non-determinism in contract context**: randomness, current wall-clock time, and external network calls are unavailable.
- **Strict WASM mode post-HF4**: `Reader::Mode::Standard` enforces full spec compliance; no x86 workarounds.
- **Memory isolation**: each far-call frame has its own heap. Callee cannot access caller's heap except through explicit argument passing.
- **No cross-contract state reads**: a contract can only `LoadVar`/`SaveVar` its own keys. Reading another contract's variables requires an app shader.
- **Audit caveat**: the BVM enforces execution correctness, not contract intent. A shader may be internally correct but economically malicious. Users should only interact with audited, open-source contracts whose bytecode has been reproduced from published source.
