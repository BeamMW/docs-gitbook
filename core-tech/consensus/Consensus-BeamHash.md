# Consensus: BeamHash PoW Algorithm

Beam uses a family of Equihash-based proof-of-work algorithms collectively called **BeamHash**. The algorithm has evolved across three generations (BeamHash I → II → III), each activated at a specific fork height to preserve ASIC-resistance during the network's early life.

---

## Algorithm Generations

| Generation | Equihash variant | Fork activation | Status |
|---|---|---|---|
| BeamHash I | `EquihashR<150,5,0>` | Genesis (fork 0) | Legacy, mainnet heights < 321,321 |
| BeamHash II | `EquihashR<150,5,3>` | Fork 1 | Legacy, mainnet heights 321,321–777,776 |
| BeamHash III | `BeamHash_III` | Fork 2 | Active on mainnet (height ≥ 777,777) |

The three variants share the same Equihash parameters **N = 150, K = 5** (giving 32 indices of 26 bits each, encoded into 104 bytes of solution data), but differ in the data-path mixing applied before the memory-hard phase. The mixing changes were introduced to invalidate purpose-built ASIC designs while keeping GPU solvers viable.

---

## Block::PoW Structure

Every block header carries a `Block::PoW` struct (`core/block_crypt.h`):

```cpp
struct Block::PoW {
    static const uint32_t N = 150;
    static const uint32_t K = 5;

    static const uint32_t nNumIndices    = 1 << K;          // 32
    static const uint32_t nBitsPerIndex  = N / (K + 1) + 1; // 26
    static const uint32_t nSolutionBits  = nNumIndices * nBitsPerIndex; // 832 bits
    static const uint32_t nSolutionBytes = nSolutionBits >> 3;          // 104 bytes

    std::array<uint8_t, nSolutionBytes> m_Indices; // packed Equihash solution
    NonceType m_Nonce;   // 8-byte nonce; incremented when nonce space is exhausted
    Difficulty m_Difficulty;

    bool IsValid(const void* pInput, uint32_t nSizeInput, Height) const;
    bool Solve(const void* pInput, uint32_t nSizeInput, Height, const Cancel& = …);
};
```

The total PoW blob serialised into a block header is **112 bytes** (104 solution + 8 nonce).

---

## Input and State Initialisation

The Blake2b state is initialised per `Block::PoW::Helper::Reset()`:

```
H = Blake2b_init(personalisation from current PoW scheme)
H.update(pInput, nSizeInput)   // 32-byte block header hash
H.update(nonce.m_pData, 8)     // 8-byte nonce
```

`pInput` is the 32-byte `Merkle::Hash` of the block header (without the PoW fields themselves). Each variant calls its own `InitialiseState` to set the personalisation string before the common data is fed in.

---

## Solution Validation

A solution is valid if both checks pass:

1. **Equihash correctness**: the packed indices, when expanded, satisfy the Equihash XOR/tree constraints for the variant active at the block's height.
2. **Difficulty target**: `Blake2b(m_Indices) ≤ target`, where the target is derived from `m_Difficulty`.

```cpp
bool Block::PoW::IsValid(…, Height h) const {
    Helper hlp;
    hlp.Reset(pInput, nSizeInput, m_Nonce, h);      // init Blake2b state
    hlp.getCurrentPoW(h)->IsValidSolution(hlp.m_Blake, indices) &&
    hlp.TestDifficulty(&m_Indices.front(), …, m_Difficulty);
}
```

`getCurrentPoW(h)` selects the variant by checking `IsPastFork_<2>(h)` (BeamHash III) then `IsPastFork_<1>(h)` (BeamHash II), falling back to BeamHash I.

---

## Difficulty Encoding

Difficulty is stored as a 32-bit packed value (`Difficulty::m_Packed`) that encodes a floating-point number with a **24-bit mantissa** and an 8-bit exponent (order):

```
m_Packed = mantissa | (order << s_MantissaBits)   // s_MantissaBits = 24
```

The **target** for a solution hash `hv` is:

```
valid iff:  hv × mantissa  fits within  (256 + s_MantissaBits − order)  bits
```

This avoids computing an explicit 256-bit target value during batch validation. A `Difficulty` can be converted to a `double` via `ToFloat()` for logging and display.

The initial mainnet difficulty at genesis is `2^22 ≈ 4,194,304`, chosen to correspond to roughly 10,000 mid-range GPUs at launch.

---

## Why Difficulty Changes

In a Proof-of-Work blockchain, difficulty is a dynamic parameter that tracks fluctuations in total network hash power. As more miners join the network, blocks are found faster; as miners leave, blocks slow down. Without adjustment, this would cause unpredictable issuance rates and settlement times.

Beam's target block time is **60 seconds**. This underpins both the currency issuance schedule and the transaction confirmation experience for users. Difficulty retargeting ensures that regardless of how much hash power the network has, blocks continue to arrive approximately once per minute on average.

## Difficulty Retargeting

Beam retargets difficulty with every block, independently on every node. The algorithm in `Rules::DA`:

| Parameter | Mainnet value | Meaning |
|---|---|---|
| `DA.Target_ms` | 60,000 ms | Target inter-block interval |
| `DA.WindowWork` | 120 blocks | Difficulty-sum window (~2 hours at normal operation) |
| `DA.WindowMedian0` | 25 blocks | Timestamp median window for current tip |
| `DA.WindowMedian1` | 7 blocks | Timestamp median window for window boundaries |
| `DA.Damp.M` / `DA.Damp.N` | 1 / 3 | Damp factor; dampens adjustment toward target |

### Retargeting formula

1. Identify the **Window End** block: the median-timestamp block among the last `WindowMedian1` blocks.
2. Identify the **Window Start** block: the median-timestamp block among the `WindowMedian1` blocks starting at `WindowWork` positions before the current tip.
3. Compute **ΔWork** = sum of all block difficulties between Window Start and Window End.
4. Compute **Δt** = timestamp difference between Window End and Window Start. Δt is clamped to [1 hour, 4 hours] to bound extreme swings.
5. Apply damp factor: `ΔtEffective = (Δt × M + Target_s × (N − M)) / N`
6. New difficulty: `NewDifficulty = ΔWork / ΔtEffective × Target_s`

The damp factor (`M=1, N=3`) blends one-third of the actual measured rate with two-thirds of the target rate, preventing oscillation under sudden hash-rate spikes or drops.

`Difficulty::Calculate(ref, dh, dtTrg_s, dtSrc_s)` in `core/difficulty.cpp` implements step 6 using a floating-point approximation (`BigFloat`) to avoid 256-bit division.

---

## Cross-references

- Fork activation heights per network: [Consensus Hard Forks](Consensus-Hard-Forks.md)
- Mining operational modes (stratum, OpenCL, integrated): [Node Mining Modes](../node/Node-Mining-Modes.md)
