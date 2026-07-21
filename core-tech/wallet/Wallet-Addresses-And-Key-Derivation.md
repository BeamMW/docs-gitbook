# Wallet Addresses and Key Derivation

Beam has no addresses on-chain. Every UTXO is an opaque commitment; ownership is determined solely by key knowledge. Addresses exist only as an off-chain communication primitive — they encode enough information for two wallets to negotiate and build a transaction without exposing identity to the public ledger.

---

## Key Hierarchy

All wallet keys derive from the **Master Key**, which is initialized from the wallet's 12-word seed phrase via a BIP39-style derivation.

```
Master Key
├── Owner KDF       — scan-only: recognizes owned UTXOs on chain
│   └── Endpoint keys     (Key::Type::EndPoint)
│   └── Asset ownership keys
├── SBBS KDF        — communication: SBBS message encryption/decryption
│   └── BBS per-address keys  (Key::Type::Bbs)
└── Coin Keys       — spending: blinding factors for UTXOs
```

**Owner KDF** is safe to share with watch-only nodes (owner key mode). It cannot spend coins but can identify which UTXOs belong to the wallet.

**SBBS KDF** is a separate derivation used only for the Secure Bulletin Board System messaging layer. It is intentionally separated so that SBBS traffic analysis cannot be linked to spending keys.

---

## SBBS Key Derivation

Each wallet address has a numeric derivation index `ownID` (`uint64_t`). From it:

```cpp
// Derive SBBS private key and public PeerID
ECC::Hash::Value hv;
Key::ID(ownID, Key::Type::Bbs).get_Hash(hv);    // Key::Type::Bbs = FOURCC("BbsM")
pKdfSbbs->DeriveKey(sk, hv);                    // sk = private scalar
pid.FromSk(sk);                                 // pid = compressed pubkey X-coord

// Derive WalletID (legacy format)
wid.m_Pk = pid;
wid.SetChannelFromPk();   // channel = first word of pid mod Bbs::s_MaxWalletChannels
```

`IWalletDB::get_SbbsWalletID()` / `get_SbbsPeerID()` in `wallet_db.cpp` implement this. See also [Wallet SBBS](Wallet-SBBS.md).

---

## Endpoint Derivation

The **Endpoint** (also called *Identity* or *SecureID*) is the receiver/sender's persistent cryptographic identity used in payment proofs and hardware wallet signing. It derives from the **Owner KDF**, not the SBBS KDF:

```cpp
// wallet_db.cpp: IWalletDB::get_Endpoint()
ECC::Hash::Value hv;
Key::ID(ownID, Key::Type::EndPoint).get_Hash(hv);  // Key::Type::EndPoint = FOURCC("tRid")
ECC::Point::Native pt;
get_OwnerKdf()->DerivePKeyG(pt, hv);               // multiply generator by derived scalar
pid = ECC::Point(pt).m_X;                         // Endpoint = X-coord of resulting point
```

The same `ownID` index produces both an SBBS key pair (via SBBS KDF) and an Endpoint (via Owner KDF). They are different keys on different curves but share the same index.

---

## WalletAddress

The in-memory and database representation of a wallet address:

```cpp
struct WalletAddress {
    WalletID    m_BbsAddr;      // SBBS address: {4-byte channel, 32-byte pubkey}
    std::string m_label;        // user-assigned label
    std::string m_category;
    Timestamp   m_createTime;
    uint64_t    m_duration;     // 0 = never expires; AddressExpiration24h = 86400s
    uint64_t    m_OwnID;        // derivation index; 0 for contact addresses
    PeerID      m_Endpoint;     // identity pubkey
    std::string m_Token;        // public-facing token string
};
```

`m_OwnID != 0` marks an owned (self-generated) address. For contact addresses saved from received tokens, `m_OwnID == 0` and `m_BbsAddr` / `m_Endpoint` are taken from the received token.

---

## Legacy Address Format (Old-Style)

The original address type, introduced with Beam's SBBS system. It encodes a `WalletID` directly as a hex string.

```
WalletID {
    m_Channel : uintBig<4>   // BBS channel number (derived from public key)
    m_Pk      : PeerID       // 32-byte compressed pubkey X-coord
}
```

**Encoding:** hex, 64–67 characters.

**Detection:** if `ParseParameters()` can decode the string as hex and the resulting buffer validates as a `WalletID`, it is treated as a legacy address.

**Channel derivation:**
```cpp
void WalletID::SetChannelFromPk() {
    BbsChannel ch;
    m_Pk.ExportWord<0>(ch);                   // first 32-bit word of the pubkey
    ch %= proto::Bbs::s_MaxWalletChannels;
    m_Channel = ch;
}
```

Legacy addresses encode only the SBBS side — no Endpoint, no transaction type metadata. They are incompatible with hardware wallets.

---

## New-Style Address / Token Format

All modern address types (Regular, Offline, MaxPrivacy, PublicOffline) are encoded as **tokens**: a serialized bag of `TxParameterID` key–value pairs, Base58-encoded.

### TxToken wire format

```cpp
struct TxToken {
    uint8_t               m_Flags;       // 0x80 (TokenFlag)
    optional<TxID>        m_TxID;        // present if token carries a specific tx ID
    PackedTxParameters    m_Parameters;  // vector<pair<TxParameterID, ByteBuffer>>
};
```

**Serialization:** Beam's binary serializer (`Serializer`) → `vector<uint8_t>` → `EncodeToBase58()`.

**Parsing** (`ParseParameters()` in `common.cpp`):
1. Try hex-decode. If valid → check for legacy `WalletID`.
2. Try Base58-decode.
3. If `buffer[0] & 0x80` and buffer length > 33 → deserialize as `TxToken`.
4. Otherwise → try legacy `WalletID` from raw bytes.

### Base58 alphabet and encoding

Beam uses the standard Bitcoin Base58 alphabet (`123456789ABCDEFGHJKLMNPQRSTUVWXYZabcdefghijkmnopqrstuvwxyz`). The implementation is a straightforward big-number base conversion without a checksum appended (unlike Bitcoin's Base58Check). Token validity is instead verified by deserializing and type-checking each parameter's value.

---

## Address Types

### Regular (new-style, interactive)

Used for standard Mimblewimble interactive transactions. Both wallets must be online simultaneously.

**Token parameters:**
| Parameter | Value |
|---|---|
| `TransactionType` | `TxType::Simple` |
| `PeerAddr` | `WalletID` — SBBS address |
| `PeerEndpoint` | `PeerID` — Endpoint |
| `IsPermanentPeerID` | `bool` |
| `Amount` / `AssetID` | optional, for payment tokens |

**Generation** (`GenerateRegularNewToken`): sets the SBBS address and Endpoint. Both come from the `WalletAddress` struct.

### Legacy (old-style)

Just a `WalletID` hex string. No Endpoint. Not usable with hardware wallets. Kept for backward compatibility with exchanges that validate address format by regex.

### Offline

Uses Lelantus-MW shielded transactions (`TxType::PushTransaction`). The sender does not need the receiver to be online; the receiver can be offline when funds arrive.

**Token parameters:**
| Parameter | Value |
|---|---|
| `TransactionType` | `TxType::PushTransaction` |
| `PeerAddr` | `WalletID` — SBBS address (for requesting more vouchers) |
| `PeerEndpoint` | `PeerID` — Endpoint |
| `ShieldedVoucherList` | `vector<ShieldedTxo::Voucher>` — pre-signed tickets |
| `IsPermanentPeerID` | `bool` |
| `Amount` / `AssetID` | optional |

**Generation** (`GenerateOfflineToken`): calls `GenerateVoucherList()` via KeyKeeper to produce N vouchers (default: 10). Each voucher is a shielded ticket pre-signed by the receiver's Endpoint key.

**Privacy note:** the sender knows the ticket internals and can detect when the receiver spends the corresponding shielded coin. Third-party observers cannot link the spend to the deposit, but the original sender can. See [Wallet SBBS](Wallet-SBBS.md) and [Addresses in Beam](Wallet-Addresses-And-Key-Derivation.md) for the detailed privacy analysis.

### Max Privacy

Uses a single Lelantus-MW voucher. The sender cannot detect when the receiver spends the coin, providing full anonymity from the sender's perspective.

**Token parameters:**
| Parameter | Value |
|---|---|
| `TransactionType` | `TxType::PushTransaction` |
| `PeerEndpoint` | `PeerID` — Endpoint |
| `Voucher` | `ShieldedTxo::Voucher` — single pre-signed ticket |
| `MaxPrivacyMinAnonimitySet` | `uint8_t` = 64 (minimum anonymity set) |
| `Amount` / `AssetID` | optional |

**Generation** (`GenerateMaxPrivacyToken`): generates exactly 1 voucher. Each Max Privacy token is **single-use**. When the voucher is consumed the sender must request new vouchers via SBBS before sending again.

**SBBS for replenishment:** if the token includes a SBBS address (via optional `PeerAddr`), the sender's wallet can automatically request more vouchers when the supply runs low, without user intervention.

### Public Offline

A stateless, reusable address derived from a deterministic public generator rather than pre-signed tickets. Suitable for donation addresses and situations where the same address must be shared with many senders indefinitely.

**Token parameters:**
| Parameter | Value |
|---|---|
| `TransactionType` | `TxType::PushTransaction` |
| `PublicAddreessGen` | `ShieldedTxo::PublicGen` — public generator |
| `PublicAddressGenSig` | `ECC::Signature` — generator signed by Endpoint key |
| `PeerEndpoint` | `PeerID` — Endpoint |

**Generation** (`GeneratePublicToken`): calls `IPrivateKeyKeeper2::Method::CreateOfflineAddr` with the address's `ownID`. The KeyKeeper produces a `PublicGen` and a signature over it.

**Privacy note:** each sender independently generates a ticket from the public generator. The receiver (who holds the corresponding private generator) can see all ticket parameters. Privacy is weaker than Offline: any two senders using the same Public Offline address could in principle correlate their sends by the generator.

---

## Address Type Detection

`GetAddressTypeImpl()` (`common.cpp:226`) classifies a parsed token:

```
Has Voucher + PeerEndpoint          → MaxPrivacy
Has ShieldedVoucherList + PeerAddr  → Offline  
Has PublicAddreessGen               → PublicOffline
TransactionType == Simple           → Regular
Has PeerAddr (no TxType set)        → Regular (legacy-compatible)
TransactionType == AtomicSwap       → AtomicSwap
Otherwise                           → Unknown
```

The API method `validate_address` returns the detected type as a string: `regular`, `offline`, `max_privacy`, `public_offline`, or `regular_new`.

---

## Payment Tokens

Any address token can optionally embed a requested payment amount and asset:

```cpp
// common parameters present in all token types when amount is specified
TxParameterID::Amount   → Amount (uint64_t, in Groth)
TxParameterID::AssetID  → Asset::ID (uint32_t; 0 = BEAM)
```

`GenerateCommonAddressPart()` sets these before the type-specific parameters. When a wallet parses such a token, it pre-fills the send amount and asset fields.

---

## Address Expiration

```cpp
static constexpr uint64_t AddressExpirationNever = 0;
static constexpr uint64_t AddressExpiration24h   = 24 * 60 * 60;
static constexpr uint64_t AddressExpirationAuto  = 24 * 60 * 60 * 61; // ~2 months
```

`m_duration` seconds from `m_createTime`. A wallet with an expired address stops decrypting incoming SBBS messages for that address's key. The Endpoint key and coin keys remain valid indefinitely — expiry only disables SBBS reception.

---

## Summary Table

| Type | Encoding | Interactive | Privacy vs sender | Reusable |
|---|---|---|---|---|
| Legacy | hex (64-67 chars) | yes (SBBS) | low | yes |
| Regular | Base58 | yes (SBBS) | low | yes |
| Offline | Base58 | no | partial (sender can link) | yes (up to voucher count) |
| Max Privacy | Base58 | no | full | single-use per voucher |
| Public Offline | Base58 | no | low (generator visible) | unlimited |

---

## Related Pages

- [Wallet Architecture](Wallet-Architecture.md) — `IWalletDB` interface and key storage
- [Wallet SBBS](Wallet-SBBS.md) — BBS channel subscription and message encryption
- [Wallet Key Keeper](Wallet-Key-Keeper.md) — `IPrivateKeyKeeper2` interface and voucher generation
- [Wallet Database Schema](Wallet-Database-Schema.md) — `addresses` table schema

---

## API Integration

### Address Validation

`validate_address` returns the detected address type and (for offline addresses) the number of remaining payments:

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "result": {
        "is_valid": true,
        "is_mine": false,
        "type": "offline",
        "payments": 3
    }
}
```

Possible `type` values: `regular`, `offline`, `max_privacy`, `public_offline`, `regular_new`.

The `payments` field is present only for `offline` addresses and indicates how many vouchers remain.

If your integration only supports standard interactive transactions, accept only `type == "regular"` or `type == "regular_new"`.

**Address regex** (accepts both hex legacy and Base58 new-style):
```
/[0-9a-zA-Z]{1,1000}/
```

### Address Creation

To create a specific address type via `create_address`, pass the `"type"` parameter:

```json
{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "create_address",
    "params": {
        "type": "max_privacy",
        "expiration": "auto",
        "comment": "John Smith"
    }
}
```

Valid `type` values: `regular`, `offline`, `max_privacy`, `public_offline`, `regular_new`. Defaults to `regular` if omitted.

---

## Address System Design Notes and Proposals

*The following section records historical design decisions and proposals from the Beam development team. It describes intended future directions, not current behavior.*

### Why Coins Don't Belong to Addresses

Beam coins are not tied to addresses. Each coin is recognized by the wallet's **Owner Key** and spendable by its **Master Key**. Addresses are only used between wallets to communicate and negotiate transactions; once a transaction is included in a block, the addresses involved are irrelevant.

This means losing an address (e.g., after wallet restore) does **not** mean losing coins. Coins are always recoverable by scanning the chain with the Owner Key.

SBBS address rotation (generating new addresses regularly) is not necessary for security — all SBBS messages are opaque and cannot be linked to a specific address by external observers. The main reason to have multiple addresses is if the user wants to present different identities to different counterparties (e.g., separate exchange accounts).

### It's All About Endpoints

The most important identifier in the Beam address system is the **Endpoint** — the cryptographic identity of the sender or receiver. Current address formats bundle the Endpoint with communication metadata (SBBS address, transaction type, vouchers) in a single opaque string. A clearer design would make the Endpoint the primary, human-visible identity.

**Proposed human-readable address format:**

```
beam_Bk1azc8VtaYU1f6t7jiRGkxJDiAVui6Y5WvohjoU1yFA_bbs274b78587e1c9643e7472e221be1634b8efe06f747175d3d8c98ce1ef665b056d4a
```

The `beam_` prefix (a common practice in other networks) is followed by a human-readable Endpoint, then the SBBS address.

**Address tampering resistance:** Beam addresses are generally safe to modify accidentally — if the SBBS part is corrupted, communication fails but no funds are lost; if the Endpoint is corrupted, the transaction negotiation fails. Funds are safe as long as the Endpoint is verified.

### Proposed Address Book Refactor

The current address book groups entries by address type. A contact-centric design would instead group by **Endpoint**:

- A list of known Endpoints, each annotated with a user-assigned name.
- Per-Endpoint metadata: SBBS address (if known), whether an Offline address is available, and how many Max Privacy vouchers remain (with an optional "request more" button).
- For owned (self) Endpoints: the internal key index (allowing re-generation after wallet restore), and an option to generate any address type for that Endpoint.

### Proposed Send Screen Improvements

**"To" field:** Instead of accepting an arbitrary address string and then determining the transaction type from it, the wallet should:
1. Extract the Endpoint from the provided address or token.
2. Look up the Endpoint in the address book.
3. Offer all transaction types for which the wallet has sufficient information about that Endpoint (online, offline, max-privacy) — not just what the single token encodes.
4. Accept an Endpoint or contact name directly, without requiring a full token.

**"From" field (currently absent):** Allow the sender to optionally specify an owned Endpoint as the sender identity. Options: any active address, a previously used (now expired) address, or "Anonymous" (ephemeral Endpoint, current default behavior). This allows the receiver to see who sent the funds.

### Proposed Transaction Details Simplification

Current transaction details expose many low-level parameters (SBBS addresses, raw signatures) that are not meaningful to most users. The proposed design shows:

- **For fund transfers:** Endpoint of sender and receiver (with address book lookup), transaction type (Online / Offline / Max Privacy), amount and asset, payment proof.
- **For contract calls and swaps:** List of sent/received asset types and amounts only (Endpoints and payment proof are not relevant).
