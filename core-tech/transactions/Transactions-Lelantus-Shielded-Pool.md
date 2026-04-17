# Lelantus-MW: Shielded Pool

Beam's shielded pool implements a variant of the [Lelantus](https://eprint.iacr.org/2019/373) zero-knowledge protocol, adapted to the Mimblewimble framework (Lelantus-MW). It provides a privacy layer on top of standard MW transactions: instead of cutting through directly visible inputs and outputs, coins pass through an opaque accumulator pool. The spending transaction proves membership in a set of commitments without revealing which element is being spent, breaking the transaction graph at the point of entry and exit.

The shielded pool is enabled at [Fork 2](Consensus-Hard-Forks) (mainnet height 321,321).

---

## How Lelantus-MW Breaks the Transaction Graph

A standard MW transaction has a direct link between inputs and outputs because the values and blinding factors must balance. Lelantus-MW severs this link:

1. **Push (shield)** — a standard UTXO is consumed and a `ShieldedTxo` (shielded output) is created. The shielded output has its own commitment and range proof, and a ticket that encodes the receiver's identity in encrypted form.

2. **Shielded pool accumulator** — the node keeps an ordered list of every shielded output ever created (never pruned, unlike regular UTXOs). Each entry is identified by a sequential `TxoID`.

3. **Pull (unshield)** — the spender constructs a Lelantus spend proof over a sliding window of the pool (the _anonymity set_). The proof asserts "I know one element in this window whose blinding factor and spend key I know" without identifying which element. The kernel carries `m_WindowEnd` (the exclusive upper bound of the window), and the shielded state MMR hash at that point is mixed into the proof.

The result: chain analysis cannot link the push transaction to the pull transaction.

---

## ShieldedTxo Structure

```cpp
struct ShieldedTxo {
    ECC::Point                   m_Commitment;  // Pedersen commitment: k·G + v·H(assetID)
    ECC::RangeProof::Confidential m_RangeProof; // Bulletproof for value ∈ [0, 2^64)
    Asset::Proof::Ptr             m_pAsset;     // Present when assetID != BEAM
    Ticket                        m_Ticket;     // Encrypted identity material
};
```

### Ticket

```cpp
struct ShieldedTxo::Ticket {
    ECC::Point                  m_SerialPub;  // Blinded serial: kG·G + kJ·J
    ECC::SignatureGeneralized<2> m_Signature; // Schnorr-J1 over (G, J) generators
};
```

The ticket carries the serial public key `kG·G + kJ·J`, a double-blinded commitment using the standard generator `G` and the alternate `J`. The signature is a generalized Schnorr proof of knowledge of both scalars `(kG, kJ)`.

### ShieldedTxo ID (wallet-side)

```cpp
struct ShieldedTxo::ID {
    BaseKey  m_Key;    // kSerG (serial scalar), key index, IsCreatedByViewer flag
    User     m_User;   // Sender PeerID + two 32-byte message fields
    Amount   m_Value;
    Asset::ID m_AssetID;
};
```

`m_User.m_pMessage` stores structured metadata encoded as field scalars; the first 16 bytes are interpreted as `PackedMessage` (TxID, max-privacy minimum anonymity set, receiver own-ID).

---

## Key Derivation for Shielded Transactions

The shielded system uses two key hierarchies derived from the wallet owner key:

```
OwnerKey  ──[HashTxt("Own.Gen") || nIdx]──►  Gen KDF  (m_pGen)
          ──[HashTxt("Own.Ser") || nIdx]──►  Ser PKDF (m_pSer)
```

**`Viewer`** holds both (`IKdf m_pGen` + `IPKdf m_pSer`) and can:
- Scan the chain for incoming shielded outputs (`TicketParams::Recover`)
- Derive the full spend parameters needed to extract coins

**`PublicGen`** holds only the public halves (`IPKdf m_pGen` + `IPKdf m_pSer`). A sender with only the receiver's `PublicGen` can create a shielded output for the receiver without knowing the receiver's private keys.

### Ticket generation (`TicketParams`)

1. Derive `kG` from nonce via `gen.DerivePKey`.
2. Compute `SerialPreimage` from `kG`, then `SpendPk = ser.DerivePKeyG(SerialPreimage)`.
3. Derive `kJ = Lelantus::SpendKey::ToSerial(SpendPk)`.
4. Compute the shared secret via ECDH: `sharedPt = kG · gen(DH(SerialPub)) + kJ · gen_J(DH(SerialPub))`.
5. Sign `(kG, kJ)` as a generalized Schnorr signature over `m_SerialPub`.

The receiver's spend key `spendSk` is `gen.DeriveKey(SerialPreimage)` and is only derivable by the holder of `m_pGen`'s private counterpart.

### Voucher

```cpp
struct ShieldedTxo::Voucher {
    Ticket              m_Ticket;       // Pre-generated, receiver-owned ticket
    ECC::Hash::Value    m_SharedSecret; // ECDH shared secret (for output decryption)
    ECC::Signature      m_Signature;   // Signed by receiver wallet address key
};
```

A `Voucher` is a pre-generated, single-use ticket signed by the receiver. The sender uses it to construct the shielded output without an interactive round-trip. Wallets maintain a pool of vouchers per address (stored in `ShieldedVoucherList`). On receiving a push transaction, the receiver locates its coin by running `TicketParams::Recover` on every chain-visible ticket.

---

## Push Transaction (Shield)

A push transaction moves funds from the transparent MW layer into the shielded pool.

**On-chain structure:**
- One `TxKernelShieldedOutput` kernel (Fork 2 required) containing the complete `ShieldedTxo`.
- Standard `Input`s consuming regular UTXOs to fund the operation.
- Optional change `Output`s.

**Transaction flow (`PushTransaction`):**

1. `PushTxBuilder` selects input coins and builds the balance equation:
   ```
   ∑(inputs) = value + fee
   ```
2. `SignSendShielded` invokes the key keeper to sign the shielded output:
   - Generates the ticket (from a sender-held voucher or self-generated).
   - Generates `OutputParams`: commits to `(value, assetID, k)` in `m_Commitment`, creates bulletproof.
3. The transaction is broadcast and registered.
4. After confirmation, the wallet requests `ProofShieldedOutp` from the node to obtain the assigned `TxoID`, then saves the `ShieldedCoin` record.

**Fee:** `m_ShieldedOutputTotal` (a flat surcharge over the standard kernel fee; post-Fork 3 this is `Coin / 100` per shielded output).

---

## Pull Transaction (Unshield)

A pull transaction extracts funds from the shielded pool back to the transparent layer.

**On-chain structure:**
- One `TxKernelShieldedInput` kernel (Fork 2 required):
  ```cpp
  struct TxKernelShieldedInput : TxKernelNonStd {
      TxoID          m_WindowEnd;   // exclusive upper bound of the anonymity window
      Lelantus::Proof m_SpendProof; // zero-knowledge membership + spend proof
      Asset::Proof::Ptr m_pAsset;
  };
  ```
- Standard `Output`s receiving the unshielded funds.

**Transaction flow (`PullTransaction`):**

1. The wallet selects an `Available` `ShieldedCoin` by `ShieldedOutputId`.
2. `IPrivateKeyKeeper2::ShieldedInput` is constructed with the coin's `ID` and per-input fee.
3. The key keeper generates the Lelantus proof (see below).
4. States: `Initial` → `Registration` → `KernelConfirmation`.

**Fee:** `m_ShieldedInputTotal` (higher than standard kernel fee; post-Fork 3 this is `Coin / 100` per shielded input, due to the cost of verifying the Lelantus proof).

---

## Lelantus Spend Proof

The proof asserts: "I know an index `L` in the anonymity window, and a blinding factor and spend key for the commitment at index `L`."

### Sigma sub-proof (set membership)

`Sigma::Cfg` parameterises the set:

| Parameter | Meaning | Typical value |
|---|---|---|
| `n` | Base (arity) | 4 |
| `M` | Depth | 5 – 8 |
| `N = n^M` | Anonymity set size | 1,024 – 65,536 |

`Shielded.m_ProofMax = {4, 8}` → up to 65,536 elements; `m_ProofMin = {4, 5}` → at least 1,024. `MaxWindowBacklog = 65,536`.

The Sigma proof has two parts:

- **Part 1** (`m_A, m_B, m_C, m_D` + `m_vG[M]`): commitments to the index decomposition.
- **Part 2** (`m_zA, m_zC, m_zR` + `m_vF[M*(n-1)]`): blinded response scalars.

Verification confirms a linear combination of all pool commitments weighted by proof-derived coefficients equals the target element's commitment, without revealing `L`.

### Lelantus extension

`Lelantus::Proof` extends `Sigma::Proof` with:

```cpp
struct Lelantus::Proof : Sigma::Proof {
    Cfg                          m_Cfg;
    ECC::Point                   m_Commitment; // the shielded coin's commitment
    ECC::Point                   m_SpendPk;    // G · spendSk
    ECC::SignatureGeneralized<2> m_Signature;  // proves knowledge of (blindingFactor, value, spendSk)
};
```

The `m_Signature` is a 2-of-2 Schnorr signature over `(G, H)` with secrets `(k, v)` and a single-key signature using `spendSk`. The "bias" point used in the Sigma proof equals:

```
bias = commitment + J · SpendKey::ToSerial(SpendPk)
```

This ties the Sigma set-membership proof to a specific spend key, preventing one proof from being reused to spend a different coin.

### Shielded state binding

Before signing, the kernel mixes `m_hvShieldedState` (the shielded MMR root at `m_WindowEnd`) into the oracle transcript. This anchors the proof to a specific chain state, preventing replays against a different pool snapshot.

### Prover witness

```cpp
struct Lelantus::Prover::Witness {
    uint32_t           m_L;         // index of spent coin in window [0, WindowEnd)
    ECC::Scalar::Native m_R;        // Sigma blinding randomness
    Amount             m_V;         // coin value
    ECC::Scalar::Native m_R_Output; // full blinding factor of coin commitment
    ECC::Scalar::Native m_R_Adj;    // adjusted blinding factor (assets)
    ECC::Scalar::Native m_SpendSk;  // private spend key
};
```

---

## Unlink Transaction (Re-randomisation)

The unlink transaction (`UnlinkFundsTransaction`) improves anonymity by replacing a coin already in the pool with a fresh one, increasing the separation between the original push and the eventual pull.

**State machine:**

```
Initial
  │  CreateInsertTransaction() → run PushTransaction (sub-tx 2)
  ▼
Insertion  [PushTransaction completes]
  │
  ▼
Unlinking  [wait until CheckAnonymitySet() → 100% progress]
  │         i.e. enough coins have entered the pool to fill the window
  ▼
BeforeExtraction
  │  CreateExtractTransaction() → run PullTransaction (sub-tx 3)
  ▼
Extraction  [PullTransaction completes]
  │
  ▼
Completed
```

`CheckAnonymitySet` reads `ShieldedCoin::UnlinkStatus.m_Progress` and only proceeds when it reaches 100 (the full anonymity window around the new coin has been populated by other coins). The mandatory wait guarantees that the unlinked coin is indistinguishable from the surrounding set.

If the user cancels during `Unlinking`, the transaction transitions to `BeforeExtraction`, immediately pulling the coin out rather than waiting.

---

## Anonymity Set and Privacy Guarantees

| Property | Detail |
|---|---|
| Anonymity set size | 1,024 – 65,536 (configurable per proof, bounded by `m_ProofMax`) |
| Set composition | All shielded outputs in a sliding window ending at `m_WindowEnd` |
| Nullifier | `SpendPk = G · spendSk`; nodes track spent `SpendPk` values to prevent double-spend |
| Linkability | The `SpendPk` is published on-chain but not linkable to the original push ticket without the private spend key |
| Unlink wait | Full anonymity set (`m_Progress == 100`) required before extraction in `UnlinkFundsTransaction` |
| Asset privacy | Confidential assets in the shielded pool carry an additional `Asset::Proof` in the kernel; asset type is hidden |

**Practical limitation:** the anonymity set is bounded by the total number of shielded outputs ever created. On a chain with few shielded outputs, the effective set is smaller than `m_ProofMax`. The wallet displays a "min anonymity set" setting (encoded in `m_User.m_pMessage` as `m_MaxPrivacyMinAnonymitySet`) to let the user express a minimum acceptable anonymity level.

---

## Shielded Pool Limits

```cpp
// max shielded ins/outs per block (block_crypt.h)
uint32_t m_MaxShieldedIns  // Rules::get() consensus parameter
uint32_t m_MaxShieldedOuts
```

These caps prevent individual blocks from being flooded with expensive Lelantus proofs (each requires batch verification over the anonymity window).

---

## Related Pages

- [Core Cryptographic Primitives](Core-Cryptographic-Primitives) — Sigma / Bulletproof / Schnorr primitives
- [Core Transaction Elements](Core-transaction-elements) — `TxKernelShieldedInput`, `TxKernelShieldedOutput` kernel subtypes
- [Consensus Hard Forks](Consensus-Hard-Forks) — Fork 2 activation height for the shielded pool
- [Transactions Confidential Assets](Transactions-Confidential-Assets) — how asset proofs appear inside shielded outputs
- [Wallet Addresses And Key Derivation](Wallet-Addresses-And-Key-Derivation) — max-privacy address type that uses shielded pool

---

## Shielded Output Coloring and DH Encoding

Lelantus-MW allows non-interactive payments: the sender creates a shielded output for the receiver without interaction. To make outputs recognizable only by the intended receiver, a *coloring* scheme embeds metadata (the *Coin ID*) into the bulletproof.

### Standard Bulletproof Coloring

The scheme embeds up to 255 bits of metadata in a deterministic way by manipulating the bulletproof nonces derived from a shared *coloring seed*.

During bulletproof construction:
- `α` — nonce generated from the commitment and coloring seed
- `ρ` — nonce generated from the commitment and coloring seed
- `x` — Fiat-Shamir challenge (from the public transcript)
- Revealed: `μ = α + β + ρ*x`

The payer converts the 24-byte Coin ID into a scalar `β` and adds it to the nonce `α` before publishing `μ`.

**Recognition (receiver side):**
1. Regenerate `α`, `ρ` from the commitment and the shared coloring seed.
2. Compute the challenge `x` from the public bulletproof transcript.
3. Recover `β = μ − α − ρ*x`.
4. Decode the Coin ID from `β`; reject if the trailing bytes are non-zero or the format is invalid.
5. Re-derive the UTXO commitment from the master secret and extracted Coin ID; verify it matches.

This scheme requires that both payer and payee share the same coloring seed.

### Advanced Coloring (Diffie-Hellman Encoding)

Standard coloring is not anonymous with respect to anyone who holds the coloring seed. The advanced scheme adds a Diffie-Hellman layer so that only the payee (not the payer, and not third parties who may share the seed) can identify the payment.

The payee creates an *encoding key* pair. The *encoding pubkey* is shared with the payer along with the coloring seed.

The double-blinded bulletproof `T1` commitment is normally:
```
T1 = n1*G + n2*J
```
where `n1`, `n2` are nonces derived from the coloring seed.

The payer modifies it by adding a random nonce `n3`:
```
T1 = (n1 + n3)*G + n2*J
```
`n3` is random — it cannot be recovered from the coloring seed — but `n3*G` is exposed via the modified `T1`.

**Shared secret derivation:**
- Payer: `S = n3 * encoding_pubkey`
- Payee: `S = (n3*G) * encoding_private_key`

Both compute the same point `S`. Its X-coordinate is converted to a scalar `γ`, which is added to the embedded metadata `β` before encoding. The payee subtracts `γ` after recovering `n3*G` from the modified `T1`.

**Payee address:** The encoding pubkey fully controls the Y-coordinate parity (the payee can negate the private key to force even Y), so only the 32-byte X-coordinate is needed in the payee address. The coloring seed can be derived deterministically from the encoding pubkey.

**Multiple addresses:** A payee can generate arbitrary numbers of addresses, each with a different encoding key. However, the payee must scan all shielded outputs against each generated address; there is no unified recognition path without scanning all keys.

### Shielded Inputs Recognition

Shielded inputs are identified by the published `SpendPk`. The payee tracks `SpendPk` values for all shielded outputs it has detected, and recognizes a spend when it sees a shielded input with the same `SpendPk`.

---

## CLI Usage

### Address Generation

Generate an offline address with 10 embedded payment vouchers:
```
./beam-wallet get_address --offline_count=10
```

Generate a max-privacy address (single voucher, fully anonymous from sender):
```
./beam-wallet get_address --max_privacy
```

Generate a public offline address (reusable donation address):
```
./beam-wallet get_address --public_offline
```

### Sending via Shielded Addresses

Send using any address type (offline, max-privacy, or public offline) via the `-r` flag:
```
./beam_wallet send -r <token> -n <node_address> -a <amount> -f <fee>
```

To force an offline (one-sided) payment when the address is an offline address:
```
./beam_wallet send -r <offline_token> --offline -n <node_address> -a <amount> -f <fee>
```

Without `--offline`, the wallet attempts a regular interactive transaction even if the token is an offline address. With `--offline`, the payment uses one of the embedded vouchers.

**v6.0 change:** the `get_address --offline` switch was renamed to `--offline_count`. The `--offline` flag on the `send` command is unchanged.

### Offline Payment Walkthrough

**Receiver:**
1. Generate a token with the desired number of vouchers:
   ```
   ./beam-wallet get_address --offline_count=3
   ```
2. Send the token to the payer and go offline.

**Sender:**
```
./beam_wallet send -r <token> --offline -n <node_address> -a <amount> -f <fee>
```

**Receiver (later):**
1. Check for incoming shielded UTXOs:
   ```
   ./beam_wallet info
   ```
   Shielded coins appear with type `shld`:
   ```
   |    ID |           BEAM |          GROTH | Maturity | Status  | Type |
      14724               34                0   95849     [Spent]   shld
      14725                7                0   95873     [Spent]   shld
   ```

2. Check transaction history for received entries:
   ```
   ./beam_wallet info --tx_history
   ```
   ```
   2020.11.14 12:52:22  incoming  5  received offline
   2020.10.30 18:13:22  incoming  1  received max privacy
   2020.10.30 11:15:22  incoming  2  received public offline
   ```

---

## Transaction Graph Obfuscation

Standard MW provides no inherent protection against transaction graph analysis when transactions are broadcast individually to the network. An attacker running a single malicious node can observe the original transaction graph before cut-through occurs in a block.

### The Problem

When a transaction is broadcast, every node that receives it sees the exact set of inputs and outputs before they are merged with other transactions in a block. An attacker who receives a transaction graph can trace relationships between users even though values and identities are hidden.

This matters because MW is not address-based — user identities are not exposed — but *relations* between users are. If any user in a chain is de-anonymized (by external means), the attacker learns the entire transaction chain with certainty.

### In-Node Obfuscation (Dandelion Extension)

Beam nodes obfuscate the transaction graph during the Dandelion++ stem phase — before transactions are broadcast to the entire network — using two complementary techniques:

**Non-interactive merge:** During the stem phase, a node may hold a transaction for a short timeout and attempt to merge it with another pending transaction instead of forwarding it immediately. The merged transaction grows like a snowball, obscuring which inputs originated from which senders.

Fairness constraints:
- Nodes should only merge transactions with comparable fee-per-byte ratios. Otherwise high-fee transactions subsidize low-fee ones.
- A node could append its own zero-fee transaction to another's — users can detect this after the fact and ban the node.
- To limit information leakage, nodes should merge only transactions of comparable size (rather than incrementally accumulating small ones).
- DoS protection: nodes verify there are no conflicts (all inputs reference existing UTXOs, no double-spends) before merging.

**Dummy UTXOs:** Any node can append one or more dummy outputs encoding zero value to a transaction. The dummy UTXO looks indistinguishable from a real UTXO. After a random timer (in terms of block count), the node spends the dummy UTXO in a later unrelated transaction, creating background activity indistinguishable from real activity.

The dummy UTXO approach does not create permanent chain bloat: kernels are not created for dummies, and all dummy UTXOs are eventually spent and cut through.

**Effect:** Even merging just two transactions creates ≈ 50% uncertainty in the input-output relation. Over 10 hops this yields ≈ 10⁻³ probability of correctly tracing the relation. A combination of both techniques is used in practice.

### Privacy Comparison

| Technique | Third-party privacy | Sender-side linkability |
|---|---|---|
| Standard MW (no obfuscation) | Inputs and outputs visible before block | Full linkage possible |
| Dandelion++ stem merge | Inputs merged before broadcast | Reduced to 1/group probability |
| Lelantus shielded pool | Inputs/outputs fully unlinkable | None (with max-privacy address) |

Transaction graph obfuscation complements the Lelantus shielded pool: it hardens the transparent MW layer, while Lelantus provides cryptographic unlinkability for users who explicitly use shielded transactions.
