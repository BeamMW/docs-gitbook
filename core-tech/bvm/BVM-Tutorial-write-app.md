How to write an app shader

1) Purpose
- App shaders are off-chain entry points that:
  - Parse input parameters from the wallet/CLI via `Env::DocGet*`
  - Produce structured output via `Env::DocAdd*`
  - Initiate contract transactions via `Env::GenerateKernel` or `Env::GenerateKernelAdvanced`

2) Structure
- Include `../common.h` and `../app_common_impl.h`. Export `Method_N` handlers.
- Often `Method_0` describes available roles/actions and parameters (introspection), and `Method_1` executes based on `role`/`action` request.

3) Reading params and writing output
```cpp
BEAM_EXPORT void Method_1() {
    Env::DocGroup root("");
    char role[16], action[16];
    if (!Env::DocGetText("role", role, sizeof(role))) { Env::DocAddText("error", "Role not specified"); return; }
    if (!Env::DocGetText("action", action, sizeof(action))) { Env::DocAddText("error", "Action not specified"); return; }
    // Dispatch by role/action...
}
```
- Use helpers: `Env::DocAddNum32/64`, `Env::DocAddText`, `Env::DocAddBlob`, groups/arrays via `Env::DocGroup`, `Env::DocArray`.
- Enumerate contract state via `Env::VarReader`/`Env::LogReader` and dump to docs (see `vault/app.cpp`).

4) Generating transactions
- Simple flow with `GenerateKernel`:
  - Provide `ContractID*`, method index, args pointer/size, `FundsChange*` array, optional signatures and comment.
  - Example deposit:
```cpp
FundsChange fc; fc.m_Aid = aid; fc.m_Amount = amount; fc.m_Consume = 1; // lock
Env::GenerateKernel(&cid, MyContract::Deposit::s_iMethod, &arg, sizeof(arg), &fc, 1, nullptr, 0, "deposit", 0);
```
- Advanced flow with `GenerateKernelAdvanced` for multisig/co-signing:
  - Provide time window, kernel blind and full nonce pubkeys, obtain challenge, complete signature, and submit (see `vault/app.cpp` `MultiSigProto`).

5) Keys and identities
- Derive per-context keys with `Env::DerivePk` or `Env::KeyID`. For multisig, combine keys/points and share public data via communication APIs if needed.

6) Proofs and state queries
- Get Merkle proofs for variables via `Env::VarGetProof` to return to the UI/client.

7) Examples
- Minimal: `playground/app.cpp` (discover and optionally deploy, then call a method).
- Rich UI-style app: `vault/app.cpp` (roles/actions, deposit/withdraw, logs, proofs, multisig negotiation).

Reference
- BVM functions for shaders: https://github.com/BeamMW/shader-sdk/wiki/BVM-functions-for-shaders


