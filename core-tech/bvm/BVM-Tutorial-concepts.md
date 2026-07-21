Core concepts for Beam shaders

1) Contract vs App shaders
- Contract shaders: on-chain logic. Export `Ctor`, `Dtor`, and `Method_N` entry points with `BEAM_EXPORT` and a single aggregated argument struct per method. See `vault/contract.cpp` and `playground/contract.cpp`.
- App shaders: off-chain drivers. Read request params via `Env::DocGet*`, emit JSON-ish output via `Env::DocAdd*`, construct transactions using `Env::GenerateKernel`/`Env::GenerateKernelAdvanced`. See `playground/app.cpp`, `vault/app.cpp`.

2) Entry points and ABI
- Reserved indices: `Ctor` = 0, `Dtor` = 1. Public methods are `Method_2`, `Method_3`, ... The exact index is declared in the arguments header `struct` via `static const uint32_t s_iMethod = N` and used by apps to invoke.
- Each exported method has signature like `BEAM_EXPORT void Method_2(const Args& r)` with a single pointer/reference parameter and no return type. Inputs/outputs are passed through that struct and via VM effects (storage, funds, logs).

3) Storage and logs
- Key-value storage is per-contract and typed helpers exist:
  - `Env::LoadVar_T(key, value)` / `Env::SaveVar_T(key, value)` / `Env::DelVar_T(key)` to read/write/delete.
  - `Env::EmitLog_T(key, value)` to append a log (readable by apps via `Env::LogReader`).
- Keys usually namespace by wrapping with `Env::Key_T<YourKey>` or composite structs. Example: `vault/contract.cpp` stores account balances keyed by `{PubKey, AssetID}`.

4) Funds balance, locking/unlocking, and invariants
- `Env::FundsLock(assetId, amount)` moves funds from the transaction to the contract (deposit). `Env::FundsUnlock(assetId, amount)` releases funds from the contract back to the transaction (withdraw). The VM enforces that a contract cannot unlock more than it has locked overall.
- Contract methods should make storage changes before funds operations (or carefully handle reverts). If a funds op fails, the whole call reverts.

5) Signatures and authorization
- Require a signature for sensitive actions with `Env::AddSig(pubkey)`; the node will enforce that the final transaction includes a signature by the owner of that key.
- For multisig, app shaders perform a nonce/key negotiation and then call `GenerateKernelAdvanced` to obtain challenges and complete the signature flow (see `vault/app.cpp` around `MultiSigProto`).

6) Assets
- Contracts can manage assets: `Env::AssetCreate`, `Env::AssetEmit`, `Env::AssetDestroy` are used by contracts such as `mirrorcoin/contract.cpp`. Emission typically accompanies `FundsLock` for minting flows.

7) Cross-contract calls
- Use `Env::CallFar` or helper `Env::CallFar_T(cid, args)` to invoke another contract method synchronously inside a contract (see `dummy/contract.cpp`). References can be tracked via `Env::RefAdd`/`Env::RefRelease` in `Ctor`/`Dtor` to ensure existence.

8) App I/O and JSON
- Use `Env::DocAddText/Num32/Num64/Blob`, `Env::DocAddGroup/Array` to construct structured responses. Helpers `Env::DocGroup` and `Env::DocArray` scope groups/arrays.
- Parse inputs with `Env::DocGetText/Num32/Num64/Blob` and sugar like `Env::DocGet("cid", ContractID&)` provided in `common.h`.

9) Transactions from apps
- `Env::GenerateKernel` submits a kernel with optional contract id, method index, args blob, list of `FundsChange` (lock/unlock), optional signatures and comment.
- `Env::GenerateKernelAdvanced` provides challenge pre-sign flow with explicit blinding and nonce pubkeys, enabling advanced multisig and co-signing.

10) Enumerating state and proofs
- `Env::VarReader` to enumerate contract variables by key prefix ranges. `Env::LogReader` to enumerate logs.
- `Env::VarGetProof` returns a Merkle proof and value for a given key for SPV-style verification (see `vault/app.cpp`).

References
- Beam wiki: BVM functions for shaders: https://github.com/BeamMW/shader-sdk/wiki/BVM-functions-for-shaders
- Examples: `vault/contract.cpp`, `vault/app.cpp`, `playground/app.cpp`, `dummy/contract.cpp`, `mirrorcoin/contract.cpp`.


