# BVM Shader Development

Beam smart contracts are implemented as *shaders* — WASM binaries that run inside the [BVM](BVM-Internals). Each contract is split across two compilation units: a **contract shader** that executes on every validating node, and an **app shader** that executes inside the wallet to build transactions and query state.

Related pages: [BVM-Internals](BVM-Internals) · [Transactions-Hi-Frequency](Transactions-Hi-Frequency) · [BVM-functions-for-shaders](BVM-functions-for-shaders)

---

## File Layout

A typical shader project has three files:

| File | Compiled to | Runs in |
|---|---|---|
| `contract.h` | (header only) | shared declarations |
| `contract.cpp` | `contract.wasm` | node (consensus-critical) |
| `app.cpp` | `app.wasm` | wallet (`ManagerStd`) |

`contract.h` holds the shared data structures — method argument structs, state layout keys, the hardcoded `ShaderID` — that both the contract and app shaders reference.

### Typical `contract.h` Skeleton

```cpp
#pragma once

namespace MyContract {
    // Precomputed SHA-256 of contract.wasm bytes
    static const ShaderID s_SID = { 0xAA, 0xBB, ... };

#pragma pack(push, 1)
    struct Params {           // constructor args (method 0)
        static const uint32_t s_iMethod = 0;
        Amount m_InitialSupply;
    };
    struct Deposit {          // method 2
        static const uint32_t s_iMethod = 2;
        AssetID m_Aid;
        Amount  m_Amount;
    };
    struct Key {              // composite storage key
        PubKey  m_Account;
        AssetID m_Aid;
    };
#pragma pack(pop)
}
```

All on-chain structs use `#pragma pack(push, 1)` — no padding — so serialization is always identical across compilers.

---

## Build Pipeline

Shaders are compiled with **Clang targeting wasm32**:

```bat
clang -O3 --target=wasm32 -std=c++17 -fno-rtti \
  -Wl,--export-dynamic,--no-entry,--allow-undefined \
  -nostdlib contract.cpp --output contract.wasm
```

The convenience script at `bvm/Shaders/make_shader.bat` (or `make_shader.sh`) wraps this command:

```bat
make_shader.bat faucet\contract   # produces faucet/contract.wasm
make_shader.bat faucet\app        # produces faucet/app.wasm
```

Both `contract.wasm` and `app.wasm` must be deployed when creating a new contract instance. Only `app.wasm` is needed to invoke an existing deployed contract.

### ShaderID Derivation

The `ShaderID` stored in `contract.h` is `SHA-256("bvm.shader.id" || len(wasm) || wasm_bytes)`. It uniquely identifies the contract bytecode; the `ContractID` additionally depends on the constructor arguments:

```
ShaderID = SHA-256("bvm.shader.id" || len || bytecode)
ContractID = SHA-256("bvm.cid" || ShaderID || len(args) || args)
```

---

## Contract-Side SDK

Contract shaders include `common.h` from the Shader SDK. All host functions are declared in the `Env::` namespace (generated from `bvm2_opcodes.h` via X-macros in `common.h`).

### Primitive Types

```cpp
typedef uint64_t Height;     // blockchain height
typedef uint64_t Amount;     // value in Groth (1 BEAM = 100,000,000 Groth)
typedef uint32_t AssetID;    // asset identifier; 0 = native BEAM
typedef uint64_t Timestamp;  // UTC seconds
typedef Opaque<32> ContractID;
typedef Opaque<32> ShaderID;
typedef Opaque<32> HashValue;
typedef Secp_point_data PubKey; // 33 bytes: 32-byte X + 1-byte parity
```

`AssetID = 0` always refers to native BEAM.

### Contract Entry Points

A contract exports three entry points:

```cpp
BEAM_EXPORT void Ctor(const MyContract::Params& r);   // method 0 — constructor
BEAM_EXPORT void Dtor(void*);                          // method 1 — destructor
BEAM_EXPORT void Method_2(const MyContract::Deposit& r); // method N
```

`Ctor` is called once when the contract is deployed. `Dtor` is called when the contract is destroyed (all funds must be zero at that point). Numbered methods correspond to `s_iMethod` constants in `contract.h`.

### Storage: Key–Value Vars

Contract state is stored in a per-contract key–value store. The BVM enforces that a contract can only read and write its own keys.

```cpp
// Save a typed value under a typed key
template <typename TKey, typename TVal>
bool Env::SaveVar_T(const TKey& key, const TVal& val);

// Load a typed value; returns true if found and size matches
template <typename TKey, typename TVal>
bool Env::LoadVar_T(const TKey& key, TVal& val);

// Delete a key
template <typename TKey>
bool Env::DelVar_T(const TKey& key);
```

Keys are namespaced automatically: the BVM prepends the `ContractID` and a `KeyTag` byte before each key when writing to global storage. `KeyTag::Internal` (value 0) is the default for contract-defined keys.

#### Key Prefix Layout

```cpp
struct Env::KeyPrefix {
    ContractID m_Cid;
    uint8_t    m_Tag; // KeyTag::Internal = 0, KeyTag::LockedAmount = 3, etc.
};
template <typename T>
struct Env::Key_T {
    Env::KeyPrefix m_Prefix;
    T              m_KeyInContract;
};
```

App shaders enumerate vars using the full prefixed key format.

### Funds Management

```cpp
void Env::FundsLock(AssetID aid, Amount amount);   // move funds from wallet into contract
void Env::FundsUnlock(AssetID aid, Amount amount); // move funds from contract back to wallet
```

`FundsLock` is the Beam equivalent of a payable function — it transfers funds **into** the contract. `FundsUnlock` transfers funds **out**. Both calls must balance across the transaction; an unbalanced transaction is rejected by the node.

Faucet deposit/withdraw example:

```cpp
BEAM_EXPORT void Method_2(const Faucet::Deposit& r)
{
    Env::FundsLock(r.m_Aid, r.m_Amount);
}

BEAM_EXPORT void Method_3(const Faucet::Withdraw& r)
{
    // ... validate account limits ...
    Env::FundsUnlock(r.m_Key.m_Aid, r.m_Amount);
    Env::AddSig(r.m_Key.m_Account); // require owner's signature
}
```

### Signature Authorization

```cpp
void Env::AddSig(const PubKey& pubKey);
```

`AddSig` declares that the transaction **must** include a valid Schnorr signature for `pubKey`. The BVM collects all declared keys during contract execution; the host verifies them against the kernel's aggregate signature after execution completes.

A contract can call `AddSig` multiple times for explicit multi-party authorization.

### Logs

```cpp
template <typename TKey, typename TVal>
uint32_t Env::EmitLog_T(const TKey& key, const TVal& val);
```

Logs are append-only and indexed by `(Height, Pos)`. They are not accessible to the contract itself at execution time but are queryable by app shaders. Logs are used for events (transfers, state changes) that wallet UIs need to scan.

### Cross-Contract Calls

```cpp
void Env::CallFar(const ContractID& cid, uint32_t iMethod,
                  void* pArgs, uint32_t nArgs, uint8_t bInheritContext);
```

Calls another deployed contract. `bInheritContext = 1` lets the callee use the caller's fund balance (used by upgradable contract patterns). Depth limit applies; `get_CallDepth()` and `get_CallerCid()` let a contract inspect the call stack.

```cpp
uint8_t Env::RefAdd(const ContractID& cid);     // increment reference count
uint8_t Env::RefRelease(const ContractID& cid); // decrement; contract may be GC'd
```

`RefAdd`/`RefRelease` prevent another contract from being destroyed while a dependent contract holds a reference to it (used by AMM → DAO vault dependency).

### Asset Management

```cpp
AssetID Env::AssetCreate(const void* pMeta, uint32_t nMeta); // returns new AssetID, or 0 on failure
uint8_t Env::AssetEmit(AssetID aid, Amount amount, uint8_t bEmit); // mint (1) or burn (0)
uint8_t Env::AssetDestroy(AssetID aid);
```

Asset metadata follows the `Asset-Descriptor-v1.0` standard. Assets created inside a contract are owned by that contract — only that contract can emit or destroy them. See [Transactions-Confidential-Assets](Transactions-Confidential-Assets).

---

## App-Shader SDK

App shaders include `common.h` and `app_common_impl.h`. They run in `ProcessorManager` inside the wallet, not on-chain.

### Method Dispatch Pattern

App shaders expose a **role/action** dispatch scheme via a macro-generated table. Method 0 returns the schema (role tree), and Method 1 routes calls:

```cpp
BEAM_EXPORT void Method_0()  // schema query — returns JSON role/action tree
{
    Env::DocGroup root("");
    { Env::DocGroup gr("roles");
        // ... list roles and their parameters ...
    }
}

BEAM_EXPORT void Method_1()  // execute — reads role/action from DocGet, dispatches
{
    char szRole[0x10], szAction[0x10];
    Env::DocGetText("role", szRole, sizeof(szRole));
    Env::DocGetText("action", szAction, sizeof(szAction));
    // ... dispatch via macro table ...
}
```

The wallet invokes Method 0 to discover what actions are available (populating the UI), then invokes Method 1 with `role` and `action` parameters to execute a specific action.

### JSON Output (Doc API)

App shaders communicate results back to the wallet as structured JSON via the `Doc*` host functions:

```cpp
void Env::DocAddGroup(const char* szID);  // opens a JSON object
void Env::DocCloseGroup();
void Env::DocAddArray(const char* szID);  // opens a JSON array
void Env::DocCloseArray();
void Env::DocAddText(const char* szID, const char* val);
void Env::DocAddNum32(const char* szID, uint32_t val);
void Env::DocAddNum64(const char* szID, uint64_t val);
void Env::DocAddBlob(const char* szID, const void* pBlob, uint32_t nBlob);
```

RAII wrappers `Env::DocGroup` and `Env::DocArray` close their scope automatically in destructors:

```cpp
{
    Env::DocGroup root("");        // opens root object
    Env::DocAddNum("height", h);
    {
        Env::DocArray gr("items"); // opens "items" array
        // ...
    }                              // closes array
}                                  // closes root object
```

### Reading Parameters

```cpp
uint32_t Env::DocGetText(const char* szID, char* szRes, uint32_t nLen);
uint8_t  Env::DocGetNum32(const char* szID, uint32_t* pOut);
uint8_t  Env::DocGetNum64(const char* szID, uint64_t* pOut);
uint32_t Env::DocGetBlob(const char* szID, void* pOut, uint32_t nLen);
```

Typed wrappers via `Env::DocGet(szID, val)` dispatch to the correct underlying call.

### Variable and Log Enumeration

App shaders read on-chain contract state using `VarReader`:

```cpp
// Read a single variable
Env::VarReader::Read_T(key, val);

// Iterate a key range
Env::VarReaderEx<false> r(k0, k1);
while (r.MoveNext_T(key, val)) { ... }
```

Log enumeration uses `Env::LogReader`, which additionally returns the `HeightPos` of each entry.

### Transaction Building

The key app-shader function is:

```cpp
void Env::GenerateKernel(
    const ContractID* pCid,   // null = deploy new contract
    uint32_t          iMethod,
    const void*       pArg,   // method argument struct
    uint32_t          nArg,
    const FundsChange* pFunds, uint32_t nFunds, // fund movements
    const SigRequest*  pSig,   uint32_t nSig,   // keys to sign with
    const char*       szComment,
    uint32_t          nCharge  // gas budget (charge units)
);
```

`FundsChange` specifies per-asset fund movement:

```cpp
struct FundsChange {
    Amount  m_Amount;
    AssetID m_Aid;
    uint8_t m_Consume; // 1 = from wallet into contract; 0 = from contract to wallet
};
```

`SigRequest` declares a key the wallet should sign with:

```cpp
struct SigRequest {
    const void* m_pID; // key derivation seed
    uint32_t    m_nID;
};
```

The wallet does not sign immediately — it collects all `GenerateKernel` calls from the app, then assembles and signs the final transaction after the app shader returns.

#### Faucet Withdrawal Example

```cpp
ON_METHOD(my_account, withdraw)
{
    Faucet::Withdraw arg;
    arg.m_Amount = amount;
    arg.m_Key.m_Aid = aid;
    DeriveMyPk(arg.m_Key.m_Account, cid); // derive user's contract-specific pubkey

    FundsChange fc;
    fc.m_Amount = amount;
    fc.m_Aid = aid;
    fc.m_Consume = 0; // funds flow from contract to wallet

    SigRequest sig;
    sig.m_pID = &cid;
    sig.m_nID = sizeof(cid);

    Env::GenerateKernel(&cid, Faucet::Withdraw::s_iMethod,
        &arg, sizeof(arg),
        &fc, 1,
        &sig, 1,
        "withdraw from Faucet", 0);
}
```

### Key Derivation

App shaders derive user-specific public keys for contracts using:

```cpp
void Env::DerivePk(PubKey& pubKey, const void* pID, uint32_t nID);
```

The typical pattern uses the `ContractID` as the key seed, producing a per-user, per-contract public key that the contract then uses as the account identifier. The corresponding private key never leaves the wallet.

---

## Authorization Patterns

### Simple: Single-Owner (`AddSig` + `SigRequest`)

The most common pattern. The contract calls `Env::AddSig(userPk)` to require the user's signature, and the app includes a matching `SigRequest` in `GenerateKernel`. This is secure, compact, and requires no elevated privilege:

```cpp
// contract side
Env::AddSig(r.m_Key.m_Account);

// app side
SigRequest sig;
sig.m_pID = &cid;
sig.m_nID = sizeof(cid);
Env::GenerateKernel(&cid, iMethod, &arg, sizeof(arg), &fc, 1, &sig, 1, "...", 0);
```

### Role-Based Key Derivation

For admin/governance keys, app shaders use a string seed instead of `ContractID`:

```cpp
static const char g_szAdminSeed[] = "upgr3-dao-vault";

struct AdminKeyID : public Env::KeyID {
    AdminKeyID() : Env::KeyID(&g_szAdminSeed, sizeof(g_szAdminSeed)) {}
};
```

Calling `AdminKeyID().get_Pk(pk)` derives the admin's public key; the contract stores this key at construction and checks it on privileged methods.

### Explicit Multi-Signature

For M-of-N authorization, the contract calls `AddSig` multiple times:

```cpp
// contract side (dao-vault pattern)
BEAM_EXPORT void Method_4(const Method::Withdraw& r)
{
    Upgradable3::Settings stg;
    stg.Load();
    stg.TestAdminSigs(r.m_ApproveMask); // checks N of M approvers have signed
    Env::FundsUnlock(r.m_Aid, r.m_Amount);
}
```

The app must collect nonces from all co-signers and coordinate a multi-round signing ceremony. The `Upgradable3::Manager::MultiSigRitual` helper in `upgradable3/app_common_impl.h` encapsulates this flow.

### Advanced Signatures (Elevated Privilege)

For custom signature schemes (ring signatures, seamless multi-sig), the app can use:

```cpp
void Env::GenerateKernelAdvanced(
    const ContractID* pCid, uint32_t iMethod, ...
    Height hMin, Height hMax,
    const PubKey& ptFullBlind,  // total blinding factor image
    const PubKey& ptFullNonce,  // total nonce image
    const Secp_scalar_data& skForeignSig,
    uint32_t iSlotBlind, uint32_t iSlotNonce,
    Secp_scalar_data* pChallenges // [out] derived challenges
);
```

This requires **privilege level 2** (elevated). The nonce slot API (`SlotInit`, `get_SlotImage`, `get_BlindSk`) provides access to ephemeral nonces without exposing raw secret keys.

#### Nonce Slots

The BVM maintains a table of nonce slots. Each slot holds an ephemeral nonce that was generated using system randomness (optionally strengthened by app-supplied seed data). Apps can:

- Generate a unique nonce (written into a slot, never exposed directly).
- Retrieve the nonce's image (public point) with any generator.
- Retrieve a *blinded key*: the sum `skBlind = sk + challenge * nonce` for a chosen `KeyID`, challenge, and slot. Once retrieved, the slot's nonce is **immediately wiped**.

This provides the building blocks for any signature scheme (Schnorr, ring, Groth Sigma) without ever exposing raw private keys to the app.

#### Risk of Signature Hijacking

When using native signatures (`SigRequest` / `AddSig`), the response scalar is automatically bound to the kernel's blinding factor. A third party cannot reuse the signature in a different transaction because the kernel blinding factor is unknown to them.

Custom signatures (built from blinded keys) are **not** automatically bound to the kernel blinding factor. An attacker who observes the transaction can extract the custom signature and replay it in their own transaction invoking the same contract method with the same arguments.

**Mitigation:** Use a hybrid of native and custom signatures.

1. Derive an ephemeral key `k_eph`. Include its pubkey `P_eph` in the contract method arguments.
2. Initialize the Oracle (challenge derivation) from `P_eph`.
3. Include `P_eph` in a **native** `AddSig` declaration.

This binds `P_eph`'s signature to the kernel blinding factor (via the native path), which in turn anchors the Oracle challenges — and therefore the custom signature — to this specific transaction. Replaying the custom signature in a different kernel is infeasible because `P_eph`'s native signature would not verify.

#### Multi-Signed Transaction Protocol (Native Signatures)

When multiple wallets must co-sign a contract invocation, all kernel parameters must be fixed in advance so that all parties derive the same challenges. The flow:

1. **Decide auxiliary kernel parameters:** `hMin`, `hMax`, fee, fund balance.
2. **Each co-signer:** generate a nonce for their key and send the nonce image.
3. **Sum all nonces** (including the BVM's internal kernel-blinding nonce).
4. **First call to `Env::GenerateKernelAdvanced`:** derive challenges from the total nonce sum.
5. **Each co-signer:** compute blinded key = `sk + challenge * nonce` and send it.
6. **Sum all blinded keys** → multi-signature.
7. **Second call to `Env::GenerateKernelAdvanced`:** finalize the transaction.

**Real examples:**
- **`vault` app** — seamless multi-sig: the vault `PubKey` is the sum of two users' keys. The contract does not know it is multi-owned; `AddSig` verifies the combined key.
- **`upgradable2` contract** — explicit multi-sig: the contract calls `AddSig` for each required key; all must be present in the kernel's aggregate signature.

**App privilege levels:**

| Level | Access | Risk |
|---|---|---|
| 0 | Read-only chain state, no transactions | Safe |
| 1 (default) | Public keys (contract scope), transaction building | User approval required; risk of deanonymization and misdirected funds |
| 2 | Blinded keys, `GenerateKernelAdvanced` | Can sign arbitrary transactions; leaked signatures usable later |
| 3 | SBBS inter-wallet communication | Leaked signatures communicable directly to attacker |

Levels 2 and 3 must be explicitly enabled by the user. At any level above 0, there is a risk of deanonymization and funds theft from malicious apps. Apps have no access to UTXO keys, shielded output keys, or the owner key — exposure is bounded to contract-scoped keys.

---

## Common Patterns from Real Shaders

### Faucet: Basic Deposit/Withdraw

**Contract** (`faucet/contract.cpp`):
- State: one `Params` record (backlog period, max withdraw), one `AccountData` record per `(PubKey, AssetID)`.
- Deposit: `FundsLock` + save updated account balance.
- Withdraw: `FundsUnlock` + `AddSig(account)` — enforces per-account rate limiting.

**App** (`faucet/app.cpp`):
- Uses the role/action macro pattern.
- `DeriveMyPk(pubKey, cid)` obtains the user's per-contract key.
- `view_accounts` enumerates all accounts by ranging over the contract's key space.

### DAO Vault: Admin-Gated Treasury

**Contract** (`dao-vault/contract.cpp`):
- Holds a multi-signer `Upgradable3::Settings` struct at construction.
- Deposit: free (`FundsLock`).
- Withdraw: requires M-of-N admin approvers (`TestAdminSigs(approveMask)`).
- Upgradable: `Upgradable3` pattern allows replacing the contract bytecode with admin approval.

**App** (`dao-vault/app.cpp`):
- `MultiSigRitual` handles the multi-round signing for withdrawal.
- Exposes `my_admin_key` action to let each admin identify their key.

### AMM: Automated Market Maker

**Contract** (`amm/contract.cpp`):
- State: one `Settings` (DAO vault reference), one `Pool` per `(Aid1, Aid2, FeeKind)`.
- Each pool holds a liquidity token (`AssetID`) created via `AssetCreate`.
- `PoolCreate`: creates a new pool, mints a liquidity-tracking asset.
- `Trade`: swaps one asset for another using constant-product formula, splits fees between pool and DAO vault.
- `AddLiquidity` / `RemoveLiquidity`: mints/burns liquidity tokens.
- Holds a `RefAdd` reference to the DAO vault contract.

Fee tiers (stored as `FeeSettings::m_Kind`):

| Kind | Fee |
|---|---|
| 0 | 0.05% (low volatility) |
| 1 | 0.30% (mid volatility) |
| 2 | 1.00% (high volatility) |

30% of collected fees are forwarded to the DAO vault; 70% stay in the pool.

---

## `ManagerStd`: Wallet-Side Execution

`ManagerStd` (in `bvm/ManagerStd.h`) is the concrete implementation of `ProcessorManager` that the wallet uses to run app shaders.

```cpp
class ManagerStd : public ProcessorManager {
public:
    proto::FlyClient::INetwork::Ptr m_pNetwork; // for var/log/header queries
    Block::SystemState::IHistory*   m_pHist;    // chain state history

    ByteBuffer m_BodyManager;   // app.wasm bytes (always required)
    ByteBuffer m_BodyContract;  // contract.wasm (required for deploy only)

    std::ostringstream m_Out;   // JSON output stream

    void Reset();
    void StartRun(uint32_t iMethod); // starts async execution
};
```

### Execution Flow

1. Wallet loads `app.wasm` into `m_BodyManager` and calls `StartRun(iMethod)`.
2. The WASM interpreter runs synchronously until the shader calls a blocking host function (a network request for vars/logs/headers).
3. The pending request is issued via `m_pNetwork` (fly-client protocol); execution suspends.
4. When the response arrives, `OnUnfreezed()` resumes execution from the suspension point.
5. When `Method_1` returns, the wallet collects all `GenerateKernel` calls accumulated in the output queue, builds the transaction, obtains user approval, signs, and broadcasts.

The `SelectContext(bDependent, nChargeNeeded)` host function lets the app shader select which block context to use for reads — independent (latest confirmed) or dependent (potentially unconfirmed, used for chained transactions).

### Calling Methods 0 and 1

- **Method 0** (schema query): called once when the wallet loads the shader, used to discover roles/actions for the UI. No transaction is built.
- **Method 1** (execute): called with a JSON parameter blob containing `role` and `action`. The shader reads these via `DocGetText`, dispatches, calls `GenerateKernel`, and writes result JSON to `m_Out`.

---

## Utility Headers

| Header | Contents |
|---|---|
| `Shaders/Math.h` | `BitUtils::clz`, `Power::Log`, `Strict::Add` (overflow-safe add), integer math helpers |
| `Shaders/Float.h` | `MultiPrecision::Float` — software floating point for price calculations in AMM |
| `Shaders/BeamHeader.h` | `BlockHeader::Full` — on-chain block header type with `IsValid()` / `get_Hash()` |
| `Shaders/BeamHashIII.h` | PoW solution verification callable from contract shaders |
| `app_common_impl.h` | `WalkerContracts`, `WalkerFunds`, `Utils::Shader` — enumerate deployed contracts, query locked funds, load/hash WASM |

`Strict::Add(a, b)` halts execution (`Env::Halt()`) on overflow, which is the standard way to guard against arithmetic overflow in contract code.
