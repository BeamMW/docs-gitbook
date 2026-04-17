# Confidential Assets and Simple Transactions

Beam supports two classes of on-chain value: native **BEAM** (asset ID 0) and user-issued **Confidential Assets** (CA, asset ID ≥ 1). Both are transferred using the standard interactive signing protocol (`SimpleTransaction`). Asset lifecycle operations — registration, issuance, consumption, and unregistration — each use a dedicated single-party transaction type backed by specialized kernel subtypes.

All fees are always paid in BEAM/Groth regardless of which asset is being transacted.

Related pages: [Core Transaction Elements](Core-transaction-elements) | [Wallet Architecture](Wallet-Architecture) | [Asset Descriptor v1.0](Asset-Descriptor-v1.0) | [Consensus Hard Forks](Consensus-Hard-Forks)

---

## Simple Transaction — Interactive Signing Protocol

`SimpleTransaction` (`TxType::Simple`) handles both plain BEAM transfers and CA transfers. The only difference for CA transfers is a non-zero `m_AssetID` field.

### Parties and roles

| Party | Role |
|---|---|
| Sender | Pays fee, selects inputs, initiates the exchange |
| Receiver | Creates output UTXOs, contributes partial signature |

A **split transaction** (same sender and receiver wallet) bypasses peer communication entirely and uses `SimpleTxBuilder` directly.

### State machine

```
Initial → [Invitation] → [PeerConfirmation] → [InvitationConfirmation]
        → Registration → KernelConfirmation → OutputsConfirmation
```

States in brackets exist only for mutual (two-party) transactions. Self-transactions skip directly to Registration after coin selection.

### Round structure (`MutualTxBuilder`)

**Round 1 — Sender invitation**

The sender calls `MutualTxBuilder::SignTxSender(initial=true)`, which:
1. Selects inputs to cover `amount + fee` from the BEAM balance (or asset balance for the asset portion).
2. Constructs the kernel height range and lifetime.
3. Sends `SetTxParameter` to the receiver with: `Amount`, `AssetID`, `Fee`, `MinHeight`, `Lifetime`, `IsSender=false`, and optional endpoint IDs (`MyEndpoint`/`PeerEndpoint`).

**Round 2 — Receiver response**

The receiver calls `MutualTxBuilder::SignTxReceiver()`, which:
1. Creates output UTXOs for the received amount (one per entry in `AmountList`, defaulting to a single output).
2. Signs its half of the kernel (nonce + excess contribution).
3. Sends back: `PeerPublicExcess`, `PeerPublicNonce`, `PeerSignature`, `PeerOffset`, and optionally `PeerInputs`/`PeerOutputs`.

**Round 3 — Sender finalization**

The sender calls `MutualTxBuilder::SignTxSender(initial=false)`:
1. Loads the receiver's partial signature via `LoadPeerPart`.
2. Adds the peer's offset.
3. Verifies and aggregates into the final kernel signature.
4. Calls `FinalyzeTx()` → normalizes, verifies, and sets `Status::FullTx`.
5. Submits the transaction via `GetGateway().register_tx(...)`.

### Safety: `IsInSafety`

A transaction is in safety (cannot be rolled back locally) once it reaches `State::KernelConfirmation`, meaning the kernel ID proof has been requested from the node.

### Key parameters settable by peer

The set of `TxParameterID` values the receiver is permitted to set externally includes:
- `Amount`, `AssetID`, `Fee`, `MinHeight`, `MaxHeight`, `Lifetime`
- `PeerPublicExcess`, `PeerPublicNonce`, `PeerSignature`, `PeerInputs`, `PeerOutputs`, `PeerOffset`
- `PaymentConfirmation`, `FailureReason`

All other parameters are ignored from the peer to prevent manipulation.

### Endpoint-based authentication

When either party uses a remote key keeper, or when the peer sends an endpoint identifier, both parties exchange `MyEndpoint`/`PeerEndpoint` (a `PeerID` derived from the wallet key). This ties the transaction to a specific key pair and enables payment proof.

---

## Asset Kernel Hierarchy

All asset lifecycle operations use kernels derived from `TxKernelAssetControl`:

```cpp
struct TxKernelAssetControl : public TxKernelNonStd {
    PeerID m_Owner;  // asset owner's public key (derived from master key + metadata)
};

struct TxKernelAssetCreate : public TxKernelAssetControl { // subtype 5
    Asset::Metadata m_MetaData;
};

struct TxKernelAssetEmit : public TxKernelAssetControl {   // subtype 2
    Asset::ID      m_AssetID;
    AmountSigned   m_Value;   // positive = issue, negative = consume
};

struct TxKernelAssetDestroy : public TxKernelAssetControl { // subtype 6
    Asset::ID m_AssetID;
    Amount    m_Deposit;  // set explicitly after Fork 5
};
```

The owner ID (`m_Owner`) is a pseudo-random public key derived from the master KDF and the asset metadata string. Only the wallet that holds the corresponding private key can produce valid asset control kernels.

---

## Asset Registration (`TxType::AssetReg`)

**Class:** `AssetRegisterTransaction`  
**Kernel:** `TxKernelAssetCreate`

### State machine

```
Initial → Making → Registration → KernelConfirmation
        → AssetConfirmation → AssetCheck → Finalizing
```

### Balance effect

```
BEAM balance -= (deposit + fee)
```

The deposit is locked in the chain until the asset is unregistered. Its amount depends on the fork:

| Network / Fork | Deposit |
|---|---|
| Mainnet, Fork 2–4 | 3 000 BEAM |
| Mainnet, Fork 5+ | 10 BEAM |
| Testnet | 1 000 BEAM (pre-fork5) |

The deposit value is returned by `Rules::get_DepositForCA(hScheme)`.

### Execution flow

1. Coin selection: lock `deposit + fee` from BEAM inputs.
2. Build `TxKernelAssetCreate` with the full `Asset::Metadata` blob and call `FinalyzeTx()`.
3. Submit to node; wait for `KernelProofHeight`.
4. After kernel confirmed, call `ConfirmAsset()` to verify the assigned asset ID.
5. Store `AssetID` in the transaction parameters and mark complete.

The node assigns the lowest available asset ID. The asset is immediately locked for 1 440 blocks after registration.

---

## Asset Issuance and Consumption (`TxType::AssetIssue` / `TxType::AssetConsume`)

**Class:** `AssetIssueTransaction`  
**Kernel:** `TxKernelAssetEmit`

Both operations share the same class; the `_issue` flag distinguishes them.

### State machine

```
Initial → [AssetConfirmation] → Making → KernelConfirmation
```

`AssetConfirmation` is skipped if the wallet already has fresh asset info cached.

### Balance effects

**Issue:**
```
asset[assetID] balance += amount
BEAM balance          -= fee
```

**Consume:**
```
asset[assetID] balance -= amount
BEAM balance          -= fee
```

### Kernel value encoding

```cpp
pKrn->m_Value = amount;          // issue: positive
pKrn->m_Value = -amount;         // consume: negative
```

The node validates that emission does not underflow zero on consume.

### Constraints

- Only the asset owner (holder of the private key matching `m_Owner`) can issue or consume.
- Issue and consume operations are not restricted during the lock period.
- Maximum emission: 2<sup>128</sup> − 1 units (tracked as `AmountBig` in the chain state).
- Maximum per-operation amount: 2<sup>64</sup> − 1 units.

---

## Asset Unregistration (`TxType::AssetUnreg`)

**Class:** `AssetUnregisterTransaction`  
**Kernel:** `TxKernelAssetDestroy`

### State machine

```
Initial → AssetConfirmation → Registration → KernelConfirmation → Finalizing
```

### Preconditions (checked before coin selection)

| Condition | Failure reason |
|---|---|
| Asset info available locally | `NoAssetInfo` |
| Total emission == 0 | `AssetInUse` |
| Lock period elapsed | `AssetLocked` |

`CanRollback(height)` returns true if the asset's last-zero-crossing height is within `MaxRollback` of the given height, preventing destruction while a rollback could resurrect emission.

### Balance effect

```
BEAM balance += (deposit − fee)
```

After Fork 5 the deposit amount is encoded in the kernel itself (`TxKernelAssetDestroy::m_Deposit`). Before Fork 5, the node uses the fixed `DepositForList2` constant.

### Post-unregistration

The asset ID is freed and can be reassigned to a new asset. Any UTXOs denominated in the old asset ID become unspendable orphans on the chain — wallets should consume all coins before unregistering.

---

## Asset Info Query (`TxType::AssetInfo`)

**Class:** `AssetInfoTransaction`

This is a wallet-local operation that queries the connected node for the latest asset state. No transaction is broadcast to the network.

### State machine

```
Initial → AssetConfirmation → AssetCheck → Finalizing
```

### Stored result: `WalletAsset`

```cpp
class WalletAsset : public Asset::Full {
    Height  m_RefreshHeight;  // block at which info was fetched
    int32_t m_IsOwned;        // non-zero if this wallet owns the asset
};
```

`Asset::Full` carries: `m_ID`, `m_Owner`, `m_Value` (total emission as `AmountBig`), `m_LockHeight`, and the raw metadata blob.

**Important:** All fields are valid only at `m_RefreshHeight`. In subsequent blocks emission may change and the asset may be unregistered. Callers should re-query if the cached height is too stale.

---

## Sending and Receiving CA

Asset transfers use `SimpleTransaction` with a non-zero `AssetID` parameter. The signing protocol is identical to BEAM transfers.

### Lock period restriction

An asset becomes **locked** for `CA.LockPeriod` (1 440 blocks, ≈ 24 hours) each time its total emission transitions through zero:

- Emission goes from 0 → non-zero (after any issue that breaks zero).
- Emission goes from non-zero → 0 (after a consume that reaches zero).

During the lock period:
- Non-owner wallets **reject** incoming asset transactions (`AssetLocked` error).
- Unregistration is **blocked**.
- Issue and consume by the owner are **permitted**.
- Lelantus shield/unshield are **unrestricted**.

The lock height is tracked in `Asset::Full::m_LockHeight`. Wallets check `WalletAsset::CanRollback()` before proceeding.

### Privacy properties

- The asset ID is hidden inside the UTXO commitment — external observers cannot distinguish BEAM outputs from CA outputs.
- The `TxKernelAssetEmit` and `TxKernelAssetCreate`/`Destroy` kernels are visible on-chain and reveal the asset ID and the owner's public key.
- Send/receive transactions do not expose the asset ID (standard Mimblewimble blinding).

---

## Asset Metadata Standard

Asset metadata is an immutable byte buffer stored in `Asset::Metadata` on-chain. It is provided at registration and cannot be updated.

### Format

```
STD:key1=value1;key2=value2;...
```

- Encoding: UTF-8.
- Maximum size: 16 384 bytes.
- The `STD:` prefix is mandatory; no trailing semicolon required.

### Mandatory fields (schema version 1)

| Key | Description |
|---|---|
| `SCH_VER` | Schema version; currently must be `1` |
| `N` | Human-readable asset name (e.g., `Beam Coin`) |
| `SN` | Short name / ticker (≤ 6 characters, e.g., `BEAM`) |
| `UN` | Unit name (e.g., `Beam`) |
| `NTHUN` | Smallest unit name (e.g., `Groth`) |

### Optional standard fields

| Key | Description |
|---|---|
| `NTH_RATIO` | Smallest-unit-to-unit ratio; default `100000000` if omitted |
| `OPT_SHORT_DESC` | One-liner description (≤ 128 characters) |
| `OPT_LONG_DESC` | Paragraph description (≤ 1024 characters) |
| `OPT_SITE_URL` | Asset website URL |
| `OPT_PDF_URL` | White paper or description document URL |
| `OPT_FAVICON_URL` | Favicon URL |
| `OPT_LOGO_URL` | Logo in SVG format URL |
| `OPT_COLOR` | UI display color in hex (`#RRGGBB`) |

The wallet parses metadata via `WalletAssetMeta`, which exposes typed getters (`GetName()`, `GetShortName()`, etc.) and `isStd()` / `isStd_v6_0()` / `isStd_v5_0()` validators.

Full specification: [Asset Descriptor v1.0](Asset-Descriptor-v1.0).

---

## Fee Rules Summary

| Operation | BEAM fee | Additional cost |
|---|---|---|
| Register asset | Standard tx fee | Deposit (10 BEAM post-fork5, 3 000 BEAM pre-fork5) |
| Issue / Consume | Standard tx fee | None |
| Unregister asset | Standard tx fee | Deposit returned to wallet |
| Send / Receive CA | Standard tx fee | None |
| Query asset info | None | None (no transaction) |

All fees and deposits are denominated in BEAM Groth (1 BEAM = 100 000 000 Groth). It is not possible to pay any fee using a Confidential Asset.

---

## Transaction Type Summary

| `TxType` | Class | Kernel subtype | Who can initiate |
|---|---|---|---|
| `Simple` | `SimpleTransaction` | `TxKernelStd` (1) | Any wallet (sender) |
| `AssetReg` | `AssetRegisterTransaction` | `TxKernelAssetCreate` (5) | Any wallet |
| `AssetIssue` | `AssetIssueTransaction` | `TxKernelAssetEmit` (2) | Asset owner only |
| `AssetConsume` | `AssetIssueTransaction` (flag=false) | `TxKernelAssetEmit` (2) | Asset owner only |
| `AssetUnreg` | `AssetUnregisterTransaction` | `TxKernelAssetDestroy` (6) | Asset owner only |
| `AssetInfo` | `AssetInfoTransaction` | *(node query, no kernel)* | Any wallet |

---

## API Integration

To enable CA support in the Wallet API, start with `--enable_assets` (or add `enable_assets=true` to the config file). Without this flag, CA transactions are rejected.

### Updated API methods (with CA support)

Pass `"assets": true` to include CA data in responses:

- **`wallet_status`** with `"assets": true` — returns a `totals` array with one entry per asset ID (0 = BEAM, 1+ = CAs), each with `available`, `maturing`, `receiving`, `sending` fields.
- **`tx_send`** / **`tx_split`** — add `"asset_id": <N>` to the params to send or split a Confidential Asset.
- **`get_utxo`** — add `"assets": true` and optionally `"filter": { "asset_id": 1 }` to list asset UTXOs.
- **`tx_list`** — add `"assets": true` to include asset transactions.

### New API methods

| Method | Purpose |
|---|---|
| `tx_asset_issue` | Mint (issue) asset coins; owner-only |
| `tx_asset_consume` | Burn (consume) asset coins; owner-only |
| `tx_asset_info` | Trigger an async node query to refresh local asset info |
| `get_asset_info` | Read locally cached asset info by `asset_id` |
| `calc_change` | Calculate change and explicit fee for a given amount and asset |

**`tx_asset_issue` request:**
```json
{
    "jsonrpc": "2.0", "id": 2,
    "method": "tx_asset_issue",
    "params": { "value": 6, "asset_id": 1 }
}
```

**`calc_change` request:**
```json
{
    "jsonrpc": "2.0", "id": 4,
    "method": "calc_change",
    "params": { "amount": 1234, "asset_id": 2, "fee": 10000, "is_push_transaction": true }
}
```

**`get_asset_info` response:**
```json
{
    "result": {
        "asset_id": 1,
        "emission": 2000000000,
        "emission_str": "2000000000",
        "isOwned": 1,
        "lockHeight": 39,
        "metadata": "STD:N=NAME;SN=SNM;UN=UNIT;NTHUN=NTHUNIT",
        "ownerId": "0ae08a49e018e98177774294107dc033790b87538e54a20e99c6b98f1dbd39ce",
        "refreshHeight": 927
    }
}
```

Note: `emission` and `emission_str` both appear because the maximum emission (2¹²⁸ − 1) exceeds JavaScript's `Number.MAX_SAFE_INTEGER`. Use `emission_str` for large values. All fields are valid only at `refreshHeight`; call `tx_asset_info` to refresh.

---

## CLI Reference

All CLI operations require the `--enable_assets` flag. Asset registration and unregistration are available **via CLI only** — there are no API equivalents for those operations.

### Enable Assets

Add `--enable_assets` to any command that creates, sends, or receives CA transactions. Without this flag, CA transactions are rejected with error `AssetsDisabled (43)`.

### Register an Asset

```
./beam-wallet asset_reg --pass <pwd> -n <node> \
  --asset_meta "STD:SCH_VER=1;N=My Token;SN=MTK;UN=Token;NTHUN=Grothtoken" \
  --fee 100 --enable_assets
```

A fixed registration deposit (10 BEAM post-Fork 5, 3 000 BEAM pre-Fork 5) is automatically deducted in addition to the transaction fee. The asset is immediately locked for 1 440 blocks after registration.

### Issue Asset Coins

```
./beam-wallet issue --pass <pwd> --asset_id 1 -n <node> --amount 10 --fee 100 --enable_assets
```

The owner can also reference the asset by metadata instead of ID:
```
./beam-wallet issue --pass <pwd> --asset_meta "STD:..." -n <node> --amount 10 --fee 100 --enable_assets
```

### Consume (Burn) Asset Coins

```
./beam-wallet consume --pass <pwd> --asset_id 1 -n <node> --amount 10 --fee 100 --enable_assets
```

### Unregister an Asset

Requires emission == 0 and lock period elapsed.

```
./beam-wallet asset_unreg --pass <pwd> -n <node> --asset_id 1 --fee 100 --enable_assets
```

The registration deposit is returned upon successful unregistration.

### Query Asset Info

```
./beam-wallet asset_info --pass <pwd> -n <node> --asset_id 1 --enable_assets
```

`asset_info` always fetches the latest data from the node. To view locally cached info, use the `info` command with `--assets` or `--asset_id`.

### View Asset UTXOs and Transactions

```
# All assets
./beam-wallet info --pass <pwd> --assets
./beam-wallet info --pass <pwd> --assets --tx_history

# Specific asset
./beam-wallet info --pass <pwd> --asset_id 1
./beam-wallet info --pass <pwd> --asset_id 1 --tx_history

# Shielded (Lelantus) asset UTXOs
./beam-wallet info --pass <pwd> --asset_id 1 --shielded_utxos
./beam-wallet info --pass <pwd> --asset_id 1 --shielded_tx_history
```

The `--enable_assets` flag is not required for display commands.

### Send and Receive Assets

```
./beam-wallet -n <node> --pass <pwd> send \
  -r <receiver_address> --amount 5 --asset_id 1 --enable_assets
```

Both sender and receiver must have `--enable_assets` active, otherwise the transaction fails with `AssetsDisabled (43)`.

**Lock period:** Assets cannot be sent to or received by non-owners during the lock period (`AssetLocked (34)` error). The lock period does not apply to Lelantus (shielded) transactions.

### Notes

- Asset ID 0 is reserved for native BEAM.
- Fees and deposits are always paid in BEAM/Groth, never in assets.
- Asset info refreshes automatically during issue/consume/unreg operations and on the first receive. After wallet restore, run `asset_info` manually for each asset.
- Transactions referencing an asset by metadata that has no valid asset ID will appear as `orphaned` in the transaction list.
