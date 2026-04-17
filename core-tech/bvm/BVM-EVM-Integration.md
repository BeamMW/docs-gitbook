# BVM — EVM Integration

Beam embeds an Ethereum Virtual Machine (EVM) interpreter alongside the native WASM-based BVM. The two runtimes are independent: the WASM BVM executes Beam shader contracts; the EVM interpreter is used for Ethereum bytecode execution and, most practically, for on-chain Ethash PoW verification via the `Ethash` shader.

---

## Why EVM Is Embedded

The primary motivation is **Ethereum interoperability**:

1. **Ethash on-chain verification** — the `Ethash` shader verifies Ethereum block PoW headers inside Beam smart contracts, enabling trustless Ethereum-side light client verification and cross-chain bridges without relying on external oracles.
2. **EVM contract execution** — Ethereum bytecode can run inside the BVM environment, sharing Beam's block state and storage model, which provides a path toward EVM-compatible DApp deployment on Beam.

---

## Architecture

### `beam::Evm::Processor`

The EVM interpreter is implemented in `bvm/evm.h` and `bvm/evm.cpp` as an abstract base class `beam::Evm::Processor`. Concrete subclasses supply the block-state backend.

```cpp
struct Processor {
    // Must override — supply block environment
    virtual Height get_Height() = 0;
    virtual bool get_BlockHeader(BlockHeader&, Height) = 0;

    // Must override — supply persistent account/slot storage
    virtual Account& LoadAccount(const Address&) = 0;
    virtual Account::Slot& LoadSlot(Account&, const Word&) = 0;

    // Optional — chain ID (default: 0)
    virtual void get_ChainID(Word&);

    void Call(const Address& to, const Args&, bool isDeploy);
    void RunOnce();   // execute one instruction
};
```

Subclasses must implement four virtual methods. Everything else — opcode dispatch, gas accounting, stack/memory management, revert — is handled by the base class.

### Execution State

Each EVM call frame is represented by `Processor::Frame`, which holds:

| Field | Description |
|---|---|
| `Stack` | Up to 1024 256-bit words (`Word`) |
| `Memory` | Dynamically-sized byte array; gas is charged per expansion |
| `Code` | Pointer into bytecode, with instruction pointer `m_Ip` |
| `Args` | Call data buffer and value transferred |
| `m_Gas` | Remaining gas for this frame |
| `m_pAccount` | The account being executed |
| `m_Type` | `Normal`, `CreateContract`, or `CallRetStatus` |

Nested calls push a new `Frame` onto `Processor::m_lstFrames`. Gas flows up on return; unused gas is returned to the caller's frame.

### Account Model

`Processor::Account` tracks per-address state:

```cpp
struct Account {
    Variable<Word>  m_Balance;   // 256-bit balance
    Variable<bool>  m_Exists;    // whether account exists
    Variable<Blob>  m_Code;      // deployed bytecode
    Slot::Map       m_Slots;     // key-value storage (256-bit key → 256-bit value)
};
```

`Variable<T>` carries a modification counter; the `UndoOp` list on each frame records how to reverse changes. On revert, the undo list is replayed in reverse order. On success, the undo list is spliced into the parent frame's list.

### Block Header Mapping

`Processor::BlockHeader` exposes the fields EVM opcodes query:

| EVM opcode | Field |
|---|---|
| `BLOCKHASH` | `m_Hash` |
| `DIFFICULTY` | `m_Difficulty` |
| `GASLIMIT` | `m_GasLimit` |
| `COINBASE` | `m_Coinbase` |
| `TIMESTAMP` | `m_Timestamp` |
| `NUMBER` | `get_Height()` |
| `CHAINID` | `get_ChainID()` (virtual, default 0) |

The host provides these values by implementing `get_BlockHeader(bh, h)`.

---

## Supported Opcodes

The interpreter dispatches through three macro-generated tables:

**Binary (two-operand, result replaces top):** `ADD`, `MUL`, `SUB`, `DIV`, `SDIV`, `MOD`, `SMOD`, `EXP`, `SIGNEXTEND`, `LT`, `GT`, `SLT`, `SGT`, `EQ`, `AND`, `OR`, `XOR`, `BYTE`, `SHL`, `SHR`, `SAR`, `SHA3`

**Unary:** `ISZERO`, `NOT`

**Custom (complex handling):** `STOP`, `ADDMOD`, `MULMOD`, `ADDRESS`, `BALANCE`, `ORIGIN`, `CALLER`, `CALLVALUE`, `CALLDATALOAD`, `CALLDATASIZE`, `CALLDATACOPY`, `CODESIZE`, `CODECOPY`, `EXTCODESIZE`, `EXTCODECOPY`, `RETURNDATASIZE`, `RETURNDATACOPY`, `BLOCKHASH`, `COINBASE`, `TIMESTAMP`, `NUMBER`, `DIFFICULTY`, `GASLIMIT`, `CHAINID`, `SELFBALANCE`, `POP`, `MLOAD`, `MSTORE`, `MSTORE8`, `SLOAD`, `SSTORE`, `JUMP`, `JUMPI`, `PC`, `GAS`, `JUMPDEST`, `PUSH0`, `CALL`, `RETURN`, `CREATE2`, `STATICCALL`, `REVERT`, `SELFDESTRUCT`

**Generated ranges:** `PUSH1`–`PUSH32`, `DUP1`–`DUP16`, `SWAP1`–`SWAP16`, `LOG0`–`LOG4`

### Not Yet Implemented

The following opcodes are noted in comments as missing or stub-only:

| Opcode | Code | Notes |
|---|---|---|
| `GASPRICE` | `0x3a` | Not implemented |
| `EXTCODEHASH` | `0x3f` | Not implemented |
| `BASEFEE` | `0x48` | Not implemented |
| `MSIZE` | `0x59` | Not implemented |
| `CREATE` | `0xf0` | Only `CREATE2` is supported |
| `CALLCODE` | `0xf2` | Not implemented |
| `DELEGATECALL` | `0xf4` | Not implemented |
| `INVALID` | `0xfe` | Not implemented |

Ethereum precompile contracts (addresses `0x01`–`0x09`: `ecrecover`, `SHA256`, `RIPEMD-160`, etc.) are also not mapped.

---

## Gas Model

Gas accounting follows Ethereum's Yellow Paper with some simplifications:

- **Static gas**: each opcode has a fixed charge encoded in the opcode table macros (e.g., `ADD` = 3, `MUL` = 5, `SLOAD` = 800, `BALANCE` = 700).
- **Dynamic gas**: charged on top for memory expansion and for `EXP` (10 + 50 × exponent byte length).
- **Memory cost**: `3a + floor(a² / 512)` where `a` is the number of 32-byte words touched.
- **`SSTORE` dynamic gas**: the complex Ethereum SSTORE gas schedule (dirty/clean/reset logic) is present in comments but currently not applied — SSTORE only pays static gas.
- **Gas return**: unused gas in a completed sub-call is returned to the parent frame (`fPrev.m_Gas += m_Gas`).

Gas exhaustion (`DrainGas` fails) triggers an exception caught by `RunOnce`, which then unwinds the failed frame via `OnFrameDone(..., false, false)`.

---

## Contract Creation

Two creation patterns are supported:

**`AddressForContract(res, from, nonce)`** — standard Ethereum CREATE address derivation. RLP-encodes `[from, nonce]`, then takes Keccak-256 and truncates to 20 bytes.

**`CREATE2`** — deterministic address from `keccak(0xFF ++ sender ++ salt ++ bytecode)`. Truncated to 20-byte `Address`.

On deploy (`isDeploy = true`), the init bytecode is run; the bytes it `RETURN`s become the account's stored code. Address collision (account already exists) fails the frame silently.

---

## Method Selector

`Processor::Method` provides selector computation matching Ethereum ABI:

```cpp
struct Method {
    uintBigFor<uint32_t>::Type m_Selector;  // 4-byte Keccak prefix
    void SetSelector(const char* szSignature);   // e.g. "transfer(address,uint256)"
};
```

---

## Ethash Service

The **Ethash service** (`bvm/ethash_service/`) is a standalone process that pre-computes Ethereum epoch data and serves Merkle proofs for on-chain PoW verification by the `Ethash` shader.

### Epoch Data Generation

Ethereum's Ethash PoW uses a large dataset that changes every ~30,000 blocks (one epoch). The service pre-computes and stores this data offline:

```
ethash_service --generate [--epoch N] [--path EthEpoch/]
```

For each epoch `N`, three files are produced:

| File | Contents |
|---|---|
| `N.cache` | Ethash light cache (used to regenerate dataset items) |
| `N.tre3` | Merkle tree over dataset items, pruned at height 3 |
| `N.tre5` | Further pruned at height 5 (smaller, used for proofs) |

After all epochs, a `Super.tre` is generated — a super-tree over all epoch roots — with a **hard-coded root** `s_SuperRoot` embedded in the `Ethash` shader.

The service supports up to `nEpochsTotal = 1024` epochs.

### Proof Generation

A JSON-RPC server mode exposes one method:

**`get_proof`**

| Field | Description |
|---|---|
| `epoch` | Epoch number |
| `seed` | 64-byte Ethash seed hash (hex) |

Response:

| Field | Description |
|---|---|
| `dataset_count` | Total number of dataset items for this epoch |
| `proof` | Hex-encoded proof blob: 64 × `Hash1024` solution elements + Merkle proof nodes |

The proof blob is passed to the `Ethash` shader's `VerifyHdr` function for on-chain verification.

### On-Chain Verification (`Ethash` Shader)

The `Ethash` shader (`bvm/Shaders/Ethash.h`) implements the full verification pipeline inside a BVM contract:

1. **InterpretPath** — runs the Ethash mix function with the 64 solution dataset items, producing a `mix_hash` and 64 dataset element indices.
2. **Multi-proof verification** — verifies that the 64 items are in the correct positions in the dataset Merkle tree.
3. **Epoch root promotion** — elevates the epoch's Merkle root into the super-tree using the proof remainder, checking against `s_SuperRoot`.
4. **Final hash** — `keccak256(seed ++ mix_hash)` produces the block PoW result.
5. **Difficulty check** — `final_hash × difficulty < 2^256` using 256-bit arithmetic.

This allows a Beam shader to trustlessly verify an Ethereum block header's PoW without trusting any oracle — only the pre-committed `s_SuperRoot` constant.

---

## Integration with BVM

The EVM interpreter and the BVM WASM runtime are **independent execution environments**. They share:

- The same `block_crypt.h` types: `Amount`, `Height`, `Timestamp`, `ContractID`
- Keccak-256 hash function (`core/keccak.h`)
- RLP encoding (`Shaders/Eth.h`)

The EVM `Processor` is not exposed as a BVM host function. Instead, contracts that need Ethereum verification call the `Ethash` shader (a standard BVM contract), which performs the Ethash computation entirely in shader code using the proof blob supplied by the off-chain Ethash service.

---

## Current Limitations

| Area | Status |
|---|---|
| Missing opcodes | `DELEGATECALL`, `CALLCODE`, `CREATE`, `GASPRICE`, `EXTCODEHASH`, `BASEFEE`, `MSIZE` not implemented |
| Precompiles | No Ethereum precompile contracts (ecrecover, SHA256, etc.) |
| SSTORE gas | Complex dirty/clean/reset gas schedule not applied |
| ChainID | Returns 0 unless subclass overrides `get_ChainID` |
| EVM ↔ BVM interop | No direct cross-call between EVM contracts and WASM shaders |
| Ethash epochs | Capped at 1024 epochs; `s_SuperRoot` is a hard-coded constant and cannot be updated without a new shader deployment |

---

## Related Pages

- [BVM Internals](BVM-Internals.md) — WASM execution model, host functions, gas/charge model
- [BVM Shader Development](BVM-Shader-Development.md) — writing contract and app shaders
- [Consensus — BeamHash](../consensus/Consensus-BeamHash.md) — Beam's own PoW algorithm
