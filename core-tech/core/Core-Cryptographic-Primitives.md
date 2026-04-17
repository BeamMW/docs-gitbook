# Core Cryptographic Primitives

Reference documentation for the low-level cryptographic building blocks used throughout Beam. All constructs are defined in `core/ecc.h`, `core/ecc_native.h`, and their implementations.

---

## Elliptic Curve Group

Beam uses the **secp256k1** curve (the same as Bitcoin), but extends its use significantly:

- Standard secp256k1 libraries expose two generators (`G` and `H`). Beam requires **131 generators** — `G`, `H`, `J`, and a full inner product proof table — and implements its own multi-scalar multiplication for efficiency.
- All curve arithmetic is in the `ECC` namespace.

### Scalar

`ECC::Scalar` is the serializable form of a scalar field element: a 256-bit big-endian integer stored as `uintBig`. Valid range is `[0, group_order)`.

```cpp
struct Scalar {
    static const uintBig s_Order; // secp256k1 group order
    uintBig m_Value;              // big-endian, 32 bytes
    bool IsValid() const;         // checks m_Value < s_Order
};
```

`Scalar::Native` wraps `secp256k1_scalar` (the in-memory, reduced form). It supports full arithmetic (`+`, `-`, `*`, inverse) and is **securely erased** on destruction via `SecureErase`.

### Point

`ECC::Point` is the serializable form of a curve point: an x-coordinate (`uintBig`, 32 bytes) plus a 1-bit Y-parity flag (`uint8_t m_Y`).

```cpp
struct Point {
    static const uintBig s_FieldOrder; // field prime (slightly larger than group order)
    uintBig  m_X;  // 32-byte x-coordinate
    uint8_t  m_Y;  // parity of y-coordinate: 0 = even, 1 = odd
};
```

`Point::Native` wraps `secp256k1_gej` (Jacobian projective coordinates) and is the working form for arithmetic. Conversion to/from `Point` (affine form) involves a field inversion.

**Compact storage in composite proofs:** In bulletproofs and similar structures, multiple Y-parity bits are packed and stored separately to save space.

`Point::Storage` is a fully affine `(x, y)` form for platform-independent serialization.

### Execution Mode

```cpp
struct Mode {
    enum Enum { Secure, Fast };
    class Scope { /* RAII, reverts on destruction */ };
};
```

`Mode::Secure` (the default) ensures constant-time operations and side-channel resistance. `Mode::Fast` trades security for speed, suitable for verification-only paths where no secret data is involved.

---

## Generators

The global `ECC::Context` (singleton, initialized once via `InitializeContext()`) holds all prepared generator tables:

| Name | Type | Use |
|---|---|---|
| `G` | `Generator::Obscured` | Blinding factor component of Pedersen commitments and signatures |
| `H` | `Generator::Simple` | Value component (BEAM amounts) of Pedersen commitments |
| `J` | `Generator::Obscured` | ElGamal / switch commitments (serial numbers) |
| `m_Ipp.m_pGen_[2][64]` | `MultiMac::Prepared` | Inner product proof generators (two interleaved vectors of 64) |

**Asset generators** for Confidential Assets are derived on demand:
```
H_i = Hash("B.Asset.Gen.V1" || i)   for i > 0
H_0 = H  (standard BEAM generator)
```
The blinded generator `H' = k*G + H_i` is what appears in an asset commitment; the asset proof demonstrates it is a valid linear combination of `G` and a legitimate `H_i`.

Generator types:
- `Generator::Obscured` — adds a random scalar offset at init time, preventing the precomputed table from revealing the generator directly. Slower but mandatory for generators multiplied by secret keys.
- `Generator::Simple` — straightforward Comb-method table. Used for `H` (value is always public once revealed).

---

## Pedersen Commitments

A Pedersen commitment to amount `v` with blinding factor `k` is:

```
C = k*G + v*H
```

The `ECC::Commitment` class is a lightweight expression object that computes this when assigned to a `Point::Native`:

```cpp
class Commitment {
    const Scalar::Native& k;
    const Amount& val;
public:
    Commitment(const Scalar::Native& k_, const Amount& val_);
    void Assign(Point::Native& res, bool bSet) const; // res = k*G + val*H
};
```

For Confidential Assets, `H` is replaced by a blinded asset generator `H'`:

```cpp
namespace Tag {
    bool IsCustom(const Point::Native* pHGen);            // false if pHGen==nullptr (BEAM)
    void AddValue(Point::Native&, const Point::Native* pHGen, Amount);
}
```

**Commitment binding:** The commitment is perfectly hiding (given uniform random `k`) and computationally binding under the discrete log assumption.

**Balance equation:** In a valid Beam transaction, the sum of output commitments minus sum of input commitments equals the kernel excess: `sum(C_out) - sum(C_in) = excess*G + fee*H`.

---

## Range Proofs

Beam uses two range proof types depending on output type:

### Confidential (Bulletproof)

`RangeProof::Confidential` is a non-interactive zero-knowledge proof that a committed value `v` satisfies `v ∈ [1, 2^64)` (minimum value is 1 Groth, defined by `RangeProof::s_MinimumValue`).

Structure:
```cpp
struct Confidential {
    struct Part1 { Point m_A; Point m_S; } m_Part1;  // commitment to bit-vectors
    struct Part2 { Point m_T1; Point m_T2; } m_Part2; // polynomial commitments
    struct Part3 { Scalar m_TauX; } m_Part3;           // blinding aggregation
    Scalar m_Mu;
    Scalar m_tDot;
    InnerProduct m_P_Tag; // inner product argument
};
```

Construction: `Create(sk, Params::Create, Oracle&)` — single-pass signing.

Verification: `IsValid(commitment, Oracle&)` — standalone, or `IsValid(commitment, Oracle&, BatchContext&)` for batch verification.

**Recovery:** If the prover embeds the `Key::ID` and value in the proof (using the `m_Seed` derived from the commitment and master secret), the owner can call `Recover(Oracle&, Params::Recover&)` to extract the amount and key ID. Up to 2 extra scalars (`m_pExtra`) and a user blob (`m_Blob`) can also be embedded.

**Multi-sig bulletproofs:** Two parties can co-sign a range proof via the `MultiSig` / `CoSign` protocol in three phases (`Step2`, `Finalize`) without revealing their blinding factors to each other.

### InnerProduct (Proof Substructure)

`ECC::InnerProduct` is the core inner product argument, defined over `nDim = 64` dimensions with `nCycles = 6` halving rounds:

```cpp
struct InnerProduct {
    static const uint32_t nDim = 64;   // dimension = bit-width of Amount
    static const uint32_t nCycles = 6; // log2(64)
    Point  m_pLR[6][2];  // L, R commitments per round
    Scalar m_pCondensed[2]; // final 1-element vectors
};
```

Batch verification accumulates all `Point::Native` multiplications into a single `MultiMac` evaluation, amortizing expensive group operations across many proofs.

### Public (Coinbase / Transparent)

`RangeProof::Public` is used for coinbase outputs where the value is publicly known:

```cpp
struct Public {
    Signature m_Signature;
    Amount    m_Value;
    struct Recovery {
        Key::ID::Packed m_Kid;
        Hash::Value     m_Checksum;
    } m_Recovery; // XOR-encrypted with seed
};
```

The value is signed and the `Key::ID` is embedded in encrypted form so the owner can recover it, but the value itself is visible on-chain.

---

## Schnorr Signatures

Beam uses a generalized Schnorr scheme built on `SignatureBase`:

```cpp
struct SignatureBase {
    Point m_NoncePub; // public nonce N = nk * G

    // Core operations:
    void Sign(const Config&, const Hash::Value& msg, Scalar* pK,
              const Scalar::Native* pSk, Scalar::Native* pRes);
    bool IsValid(const Config&, const Hash::Value& msg,
                 const Scalar* pK, const Point::Native* pPk) const;
};
```

The standard form is `(N, k)` where `k = -(nk + e*sk)` and `e` is the Oracle challenge derived from `(N, msg)`. Verification checks `k*G + e*Pk + N == 0`.

### Standard Signature

`ECC::Signature` (single generator `G`, single key):

```cpp
struct Signature : SignatureBase {
    Scalar m_k;
    void Sign(const Hash::Value& msg, const Scalar::Native& sk);
    bool IsValid(const Hash::Value& msg, const Point::Native& pk) const;
};
```

### Generalized Signature

`SignatureGeneralized<nG>` carries `nG` scalar responses, one per generator, all sharing a single `m_NoncePub`. Multiple independent challenges are derived from the Oracle:

```cpp
template <uint32_t nG>
struct SignatureGeneralized : SignatureBase {
    Scalar m_pK[nG];
};
```

Pre-configured combinations used in Beam (from `Context::Sig`):
- `m_CfgG1` — standard (G, 1 key)
- `m_CfgGJ1` — G + J generators, 1 key (shielded serial number)
- `m_CfgG2` — G, 2 keys (multisig)
- `m_CfgGH2` — G + H, 2 keys (Lelantus spend proof)

### Multi-Party Signing Protocol

For interactive multi-sig, each party must:
1. Generate a **unique** nonce with external randomness (not just from secret key and message — shared challenges across different rituals would leak the private key).
2. Exchange public nonces and sum them to form `m_NoncePub`.
3. Each party calls `SignPartial`, contributing their partial `k_i`.
4. Final signature is the sum of partial signatures.

---

## Oracle (Challenge Generator)

`ECC::Oracle` is the Fiat-Shamir transcript accumulator:

```cpp
class Oracle {
    Hash::Processor m_hp; // SHA-256 based
public:
    void Reset();
    template <typename T> Oracle& operator<<(const T& t); // feed data
    void operator>>(Scalar::Native&);  // extract challenge
    void operator>>(Hash::Value&);
};
```

Extracting a challenge finalizes the current hash state, produces a challenge, and immediately re-feeds the result back into the hash — so the next challenge incorporates all prior transcript data including previous challenges.

If the extracted value does not satisfy a constraint (e.g., must be non-zero or a valid x-coordinate), the Oracle cycles until a valid value is found (accept/reject, with negligible probability of looping).

---

## Nonce Generator and KDF

`ECC::NonceGenerator` produces deterministic secret scalars:

```cpp
class NonceGenerator {
    // Initialized with a salt string and secret data
    // Produces scalars via HMAC-SHA-256 + secp256k1 nonce function
    NonceGenerator& operator<<(const T& t); // mix in transcript
    NonceGenerator& operator>>(Scalar::Native&); // draw nonce
};
```

**Key Derivation Function:** `Key::IKdf` and `Key::IPKdf` derive child keys from a master secret. The `Key::ID` struct identifies a key:

```cpp
struct Key::ID {
    uint64_t m_Idx;    // sequential index
    Type     m_Type;   // FourCC tag (see table below)
    Index    m_SubIdx; // child KDF index
};
```

Common key types:

| FourCC | Constant | Use |
|---|---|---|
| `fees` | `Type::Comission` | Mining fee outputs |
| `mine` | `Type::Coinbase` | Block reward outputs |
| `norm` | `Type::Regular` | Regular UTXOs |
| `chng` | `Type::Change` | Change outputs |
| `kern` | `Type::Kernel` | Test kernels |
| `BbsM` | `Type::Bbs` | SBBS message keys |
| `Asst` | `Type::Asset` | Confidential asset ownership |

`IKdf::DeriveKey(Scalar::Native& out, const Key::ID&)` hashes the Key::ID to produce a `Hash::Value`, then feeds it through the NonceGenerator to produce a deterministic scalar.

---

## Hash Functions

| Function | Location | Role in Beam |
|---|---|---|
| **SHA-256** | `Hash::Processor` (`secp256k1_sha256` internally) | Primary hash for all ECC operations: Oracle challenges, nonce generation, commitment hashing, KDF |
| **HMAC-SHA-256** | `Hash::Mac` | NonceGenerator — produces cryptographic nonces from secret data and transcript |
| **AES-256-CTR** | `core/aes.h` | Symmetric encryption for SBBS messages and secure communication channels |
| **Keccak-256/512** | `core/keccak.h` | Ethereum bridge: address derivation, EVM transaction hashing |
| **Blake2b** | `3rdparty/crypto/blake/` | BeamHash PoW algorithm (Equihash seed hashing) — see [Consensus-BeamHash](Consensus-BeamHash) |

### SHA-256 Hash Processor

```cpp
class Hash::Processor {
    // Feeds typed values in a platform-independent, unambiguous way:
    // - bool → 1 byte (0 or 1)
    // - strings → bytes including null terminator
    // - integers → variable-length with terminator mark
    // - Scalar, Point, uintBig → their serialized forms
    template <typename T> Hash::Processor& operator<<(const T&);
    void operator>>(Hash::Value&); // finalize and reset
};
```

### AES-256

Used in CTR mode for SBBS. The stream cipher maintains a 16-byte counter block and a buffer:

```cpp
struct AES::StreamCipher {
    uintBig_t<16> m_Counter;  // CTR mode counter
    uint8_t m_pBuf[16];       // generated keystream block
    void XCrypt(const Encoder&, uint8_t* pBuf, uint32_t nSize);
};
```

### Keccak

`beam::KeccakProcessor<nBits>` is a streaming wrapper over the Keccak sponge construction. Instantiated as `KeccakProcessor<256>` (Keccak-256) or `KeccakProcessor<512>` for Ethereum compatibility:

```cpp
template <uint32_t nBits_>
struct KeccakProcessor : KeccakProcessorBase {
    void Write(const uint8_t* pSrc, uint32_t nSrc);
    void Read(uint8_t* pRes);
    void operator>>(uintBig_t<nBytes>&);
};
```

---

## uintBig — Fixed-Width Big Integer

`beam::uintBig_t<N>` is an `N`-byte big-endian byte array providing:
- Basic arithmetic: increment, XOR, complement
- Comparison (`cmp`)
- Hex serialization / deserialization
- Bit-order and range utilities (`_GetOrder`, `_Accept` for modular reduction)

The most common instantiation is `ECC::uintBig` = `uintBig_t<32>` (256 bits), used for scalars, point x-coordinates, and hash values.

```cpp
static const uint32_t ECC::nBytes = 32;
typedef beam::uintBig_t<32> ECC::uintBig;
```

**Serialization invariant:** The byte array is stored and transmitted as-is (big-endian), making it platform-independent. A `Scalar` is only valid if its `uintBig` value is strictly less than `Scalar::s_Order`.

**Not for performance-critical paths.** `uintBig_t` is explicitly documented as a non-optimized utility type; all performance-sensitive scalar arithmetic uses `Scalar::Native` (the secp256k1 reduced representation).

---

## Security Properties Summary

| Property | Guarantee |
|---|---|
| Commitment hiding | Perfect hiding (uniform random blinding factor) |
| Commitment binding | Computational — under discrete log hardness of secp256k1 |
| Range proofs | Completeness, soundness, and zero-knowledge under DL hardness |
| Signature unforgeability | Existential unforgeability under DL hardness (Schnorr security) |
| Nonce determinism | RFC 6979 variant — same (key, message) always produces the same nonce |
| Side-channel resistance | `Mode::Secure` (default) enforces constant-time field/group ops |
| Secret key erasure | `Scalar::Native` destructor calls `SecureErase`; `NoLeak<T>` wrapper for sensitive stack values |

---

---

## Biased Sigma Protocol

The Biased Sigma protocol proves knowledge of the opening of one element out of a set of N, after a known *Bias* point has been methodically subtracted from all elements. It is used as the set-membership sub-proof inside Lelantus spend proofs.

**Inputs:**
- `Bias` — an EC point (the element being proved minus its serial number)
- Set of N elements (EC points)

**Key properties compared to the standard Groth Sigma protocol:**

1. **Oracle-bound transcript:** The proof operates on an `Oracle` (Fiat-Shamir), making it inseparable from the enclosing transcript (the full spend proof). It cannot be detached or replayed in a different context.
2. **Batch-friendly:** The Bias is not literally subtracted from each set element. Instead, a cumulative coefficient for the Bias is tracked and applied once at the end, allowing multi-exponentiation over the unchanged set to be deferred and accumulated across multiple proofs.
3. **Padding:** If the set has fewer than N elements, it is padded with points at infinity (zero). This is safe because all valid uses of the Bias Sigma protocol prove something about the *Bias*, not the zero element. An attacker cannot use a zero-padded position to forge a proof.

### Asset Generator Proof

Proves that a blinded generator `H'` satisfies `H' = k*G + H_i` where `H_i` is a legitimate asset generator:
```
H_i = Hash("B.Asset.Gen.V1" || i)   for i > 0
H_0 = H   (standard BEAM generator)
```

The proof is a Biased Sigma proof in terms of the `G` generator only, with `Bias := H'`. The prover selects the window containing `H_i` and specifies the first element of the window; the window size is fixed by consensus.

**Open design question:** `H'` is currently *not* exposed to the Oracle used in the Biased Sigma sub-proof. This means an attacker could theoretically craft a Sigma proof for one `H'` and then substitute a different `H'` that satisfies the equation. However, such a substituted `H'` is a random EC point with no exploitable relation to any standard generator, so it cannot be used to conceal negative values or serial numbers. Whether `H'` should be exposed to the Oracle for defense-in-depth is an open question.

### Lelantus Spend Proof Transcript

The full Lelantus spend proof combines a generalized Schnorr signature with the Biased Sigma set-membership proof. The transcript order is:

```
oracle ← Sigma parameters (n, M)
oracle ← Commitment
oracle ← SpendPk
oracle ← N  (public nonce of the generalized Schnorr multi-signature)
oracle → challenge for Commitment (e_C)
oracle → challenge for SpendPk   (e_S)
         ← Schnorr multi-signature responses: (k_G, k_H)
oracle ← Sigma protocol Part 1: (A, B, C, D, G-vector)
oracle → challenge for Sigma protocol (x)
         ← Sigma protocol Part 2: (a, c, r, f-vector)
```

**Technical compression:** `Commitment` appears twice in the verification equation — once in the Schnorr signature and once as part of the Bias. Rather than using it twice, its coefficient is accumulated and the point is used once in the final multi-exponentiation.

**Schnorr multi-signature:** The two individual signature proofs (validity of `Commitment` in terms of `(G, H)` and knowledge of the SpendPk pre-image) are compressed into a single `SignatureGeneralized<2>` with secrets `(k, v)` and a single public nonce `N`. This saves one group element compared to two independent signatures.

---

*See also:* [Core-Transaction-Elements](Core-transaction-elements) for how commitments and signatures compose into transactions. [Transactions-Lelantus-Shielded-Pool](Transactions-Lelantus-Shielded-Pool) for how the Biased Sigma protocol is used in the full Lelantus-MW spend proof.
