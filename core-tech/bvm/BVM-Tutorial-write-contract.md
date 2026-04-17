How to write a contract shader

1) File layout
- Create `contract.h` with public argument structs and method indices. Example:

```0:0:bvm/Shaders/vault/contract.h
```

Typical header pattern:

```markdown
// contract.h (sketch)
namespace MyContract {
#pragma pack (push, 1)
struct MethodCreate { static const uint32_t s_iMethod = 0; /* ctor args */ };
struct MethodDestroy { static const uint32_t s_iMethod = 1; };
struct Deposit { static const uint32_t s_iMethod = 2; AssetID m_Aid; Amount m_Amount; };
struct Withdraw { static const uint32_t s_iMethod = 3; AssetID m_Aid; Amount m_Amount; PubKey m_Account; };
#pragma pack (pop)
}
```

2) Implement entry points
- Implement in `contract.cpp` and include `../common.h` and your header. Export with `BEAM_EXPORT`.
- Reserve 0 and 1 for `Ctor` and `Dtor` even if empty.

```cpp
#include "../common.h"
#include "contract.h"

BEAM_EXPORT void Ctor(const MyContract::MethodCreate& r) {
    // initialize state, set refs
}

BEAM_EXPORT void Dtor(void*) {
    // cleanup and release refs
}

BEAM_EXPORT void Method_2(const MyContract::Deposit& r) {
    // read-modify-write state
    // Env::SaveVar_T / LoadVar_T / EmitLog_T
    Env::FundsLock(r.m_Aid, r.m_Amount);
}

BEAM_EXPORT void Method_3(const MyContract::Withdraw& r) {
    // authorization
    Env::AddSig(r.m_Account);
    Env::FundsUnlock(r.m_Aid, r.m_Amount);
}
```

3) Storage keys and state
- Define compact keys for variables and use typed helpers:
  - `Env::LoadVar_T(key, val)`, `Env::SaveVar_T(key, val)`, `Env::DelVar_T(key)`.
- Emit logs for auditability with `Env::EmitLog_T(key, val)`.

4) Funds and signature safety
- Only `FundsUnlock` what you previously locked (VM enforces total balance).
- Require signatures for withdrawals with `Env::AddSig(pubkey)`.
- Use `Strict::Add/Sub` to avoid overflow/underflow on balances.

5) Cross-contract calls (optional)
- Use `Env::CallFar_T(cid, args)` to invoke another contract. Maintain references with `Env::RefAdd` in `Ctor` and `Env::RefRelease` in `Dtor`.

6) Assets (optional)
- Create and manage assets with `Env::AssetCreate`, `Env::AssetEmit`, `Env::AssetDestroy` when needed.

7) Build
- compile to WASM (see `Shaders/Readme.txt`). Example flags:
  - `clang -O3 --target=wasm32 -Wl,--export-dynamic,--no-entry,--allow-undefined -nostdlib -o out.wasm sources...`

Good examples to study
- Minimal: `playground/contract.cpp` (Ctor/Dtor/Method stub).
- Storage + funds + auth: `vault/contract.cpp`.
- Assets and cross-calls: `mirrorcoin/contract.cpp`, `dummy/contract.cpp`.

Reference
- BVM functions for shaders: https://github.com/BeamMW/shader-sdk/wiki/BVM-functions-for-shaders


