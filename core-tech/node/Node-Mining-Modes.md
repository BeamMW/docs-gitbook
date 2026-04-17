# Node Mining Modes

Beam nodes can produce blocks in several operational modes. All modes share the same `IExternalPOW` abstraction (`pow/external_pow.h`), which decouples the node's block-production logic from the solver implementation.

---

## `IExternalPOW` Interface

Every mining backend implements this interface:

```cpp
class IExternalPOW {
public:
    enum BlockFoundResultCode {
        solution_accepted,
        solution_rejected,
        solution_expired
    };

    using BlockFound      = std::function<BlockFoundResult()>;
    using CancelCallback  = std::function<bool()>;

    // New mining job from the node
    virtual void new_job(
        const std::string& jobID,
        const Merkle::Hash& input,   // 32-byte block header hash
        const Block::PoW& pow,       // difficulty + nonce seed
        const Height& height,
        const BlockFound& callback,
        const CancelCallback& cancelCallback) = 0;

    virtual void get_last_found_block(std::string& jobID, Height&, Block::PoW&) = 0;
    virtual void stop_current() = 0;
    virtual void stop() = 0;
};
```

The node calls `new_job` whenever it assembles a new candidate block. When a solver finds a valid solution it invokes the `BlockFound` callback, and the node submits the block.

---

## Mode 1 — Integrated CPU Mining

**How to enable:** pass `--mine` to the beam-node binary.

The node creates a `create_local_solver(fakeSolver)` instance. `fakeSolver = false` runs the real `Block::PoW::Solve` loop; `fakeSolver = true` skips actual solving and returns a dummy solution (used in unit tests and local testnets).

The solve loop in `pow/beamHash.cpp`:

```cpp
while (true) {
    hlp.Reset(pInput, nSizeInput, m_Nonce, h);           // init Blake2b state
    if (hlp.getCurrentPoW(h)->OptimisedSolve(…)) break;  // Equihash search
    if (fnCancel(true)) return false;                     // abort requested
    m_Nonce.Inc();                                        // try next nonce
}
```

When CPU AVX extensions are available, the solver uses a vectorised code path; see [AVX](AVX.md) for the build-time flag.

---

## Mode 2 — Local OpenCL Mining (GPU)

**How to enable:** the node (or a separate process) calls `IExternalPOW::create_opencl_solver(devices)`.

`devices` is a vector of OpenCL device indices. The implementation (`pow/opencl_pow.cpp`) creates an `OpenCLMiner` which:

1. Spins up a `WorkProvider` thread that feeds jobs to the OpenCL host (`beamMiner::clHost`).
2. Spins up a miner thread that calls `_ClHost.startMining()`.
3. Handles solutions via `WorkProvider::handleSolution`, packing the compressed indices and nonce back into `Block::PoW`.

The initial nonce seed is randomised with `ECC::GenRandom` at construction, so multiple OpenCL instances running simultaneously explore different nonce spaces.

A separate, smaller thread validates difficulty against the packed `Difficulty` value before dispatching the `BlockFound` callback to the node.

See [Supported nVidia Cards (OpenCL)](Supported-nVidia-cards-for-mining-using-OpenCL-miner.md) for the tested GPU list.

---

## Mode 3 — Stratum Pool Mining

Beam uses a JSON-RPC line-delimited protocol over TCP (optionally TLS) between pool miners and the node. The five message types are:

| Direction | Method | Purpose |
|---|---|---|
| Miner → Node | `login` | Authenticate with an API key |
| Node → Miner | `job` | Distribute a new mining job |
| Node → Miner | `cancel` | Revoke a previously issued job |
| Miner → Node | `solution` | Submit a found solution |
| Node → Miner | `result` | Acknowledge login or solution |

### Job message

```json
{
  "jsonrpc": "2.0",
  "method": "job",
  "id": "212",
  "input": "636b90cc…",      // 32-byte block header hash (hex)
  "difficulty": 3441671469,  // packed Difficulty value
  "height": 1500000
}
```

`input` is the 32-byte `Merkle::Hash` passed directly to the Equihash solver. `difficulty` is `Block::PoW::m_Difficulty.m_Packed` as a `uint32_t`.

### Solution message

```json
{
  "jsonrpc": "2.0",
  "method": "solution",
  "id": "212",
  "nonce": "0bb11009afc29dbe",  // 8-byte nonce (hex)
  "output": "a32a1e04…"         // 104-byte solution indices (hex)
}
```

### Result codes

| Code | Name | Meaning |
|---|---|---|
| 0 | `no_error` | Login successful |
| 1 | `solution_accepted` | Valid solution, block submitted |
| 2 | `solution_rejected` | Solution failed difficulty or Equihash check |
| 3 | `solution_expired` | Job was superseded before solution arrived |
| −32000 | `message_corrupted` | JSON parse error |
| −32003 | `login_failed` | API key rejected |

### Nonce prefix

On login success the node may return a `nonceprefix` field (0–6 hex bytes). The miner must fix that prefix at the start of every nonce it tries, partitioning the nonce space across connected miners:

```json
{
  "method": "result",
  "id": "login",
  "code": 0,
  "description": "Login successful",
  "nonceprefix": "ab4e3a"
}
```

The prefix length is configured on the stratum server with `noncePrefixDigits` (0–6 hex nibbles). The `Server` class assigns a unique prefix to each TCP connection via `gen_nonceprefix(connId)`.

### Fork height hints

The `Result` struct carries optional `forkheight` and `forkheight2` fields so miners can prepare for upcoming hard forks without polling the node separately.

### Access control

The `Server::AccessControl` class reads an API-key file at startup and re-reads it on each refresh tick. Connections that fail the key check receive a `login_failed` result and are closed.

---

## Mode 4 — Stratum Client (External Pool Miner)

`pow/miner_client.cpp` provides a standalone binary (`miner_client`) that connects to any Beam stratum server and drives a local solver:

```
miner_client --server <host:port> --key <api_key> [--no-tls] [--fake]
```

The client (`StratumClient`) maintains a persistent TCP connection and auto-reconnects after 1,000 ms on disconnect. On connection it sends a `login` message, then passes each incoming `job` to a local `IExternalPOW` instance (CPU solver by default). When a solution is found it sends a `solution` message.

---

## Coinbase Key Management and Online vs. Offline Mining

A node must possess a key to create the coinbase UTXO for each block it mines. Beam supports two key management strategies:

**Offline mining (default):** the node uses its dedicated **miner key** (derived from the master key but separate from spend keys) to create coinbase UTXOs without wallet involvement. This is the default and recommended mode. The wallet can be offline; when it reconnects it will detect the newly mined coins through the standard coin-discovery process.

**Online mining (opt-in):** if a wallet with the matching **owner key** is connected to the node, the node can delegate coinbase UTXO creation to the wallet. This keeps the miner key out of the node process entirely. It is disabled by default because a stalled wallet connection caused some mining pools to miss blocks (the node sent coinbase-creation requests but received no response, wasting solved blocks).

The miner key is derived from the master seed and scopes coinbase outputs separately from regular wallet coins, limiting exposure if the mining node is compromised.

---

## Job Latency and Mempool Packing

The node regenerates a new mining job when:
- A new block is received (immediate), or
- A new transaction enters the mempool's fluff phase (rate-limited by `miner_job_latency`, default 1,000 ms).

Miners should not discard incoming jobs prematurely. Since the Equihash solver switches jobs at zero cost (nonce increment is the only state), accepting every job maximises fee revenue and is required for correct operation of [Hi-Frequency Transactions](Hi-Frequency-transactions).

---

## Cross-references

- BeamHash algorithm and difficulty encoding: [Consensus BeamHash](Consensus-BeamHash.md)
- Fork activation heights: [Consensus Hard Forks](Consensus-Hard-Forks.md)
- Stratum wire protocol reference: [Beam Mining API (Stratum)](Beam-mining-protocol-API-(Stratum).md)
- AVX CPU optimisation: [AVX](AVX.md)
