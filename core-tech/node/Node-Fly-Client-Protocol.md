# Node Fly Client Protocol

The **fly client** is Beam's lightweight node protocol. It allows wallets and lite nodes to interact with the blockchain — submitting transactions, querying proofs, scanning for owned coins, and receiving SBBS messages — without downloading or validating the full block history. All data received from a full node is cryptographically verified against the current chain tip before being trusted.

The implementation lives in `core/fly_client.h` and `core/fly_client.cpp`.

---

## Architecture

Three layers compose the fly client stack:

```
FlyClient (abstract)          ← wallet or lite node implements this
    └─ INetwork (abstract)    ← pluggable transport
         └─ NetworkStd        ← standard TCP implementation
              └─ Connection   ← one TCP connection to a full node
```

### `FlyClient` (abstract base)

The consumer of the protocol. Wallets (`Wallet` class) derive from `FlyClient` and override its virtual callbacks:

| Method | Purpose |
|---|---|
| `OnNewTip()` | A new best chain tip has been confirmed and added to local history |
| `OnTipUnchanged()` | Connected to a node whose tip matches our known tip — no sync needed |
| `OnRolledBack()` | Some local headers were invalidated; rolled back from history |
| `get_History()` | Returns the client's `Block::SystemState::IHistory` store |
| `get_Kdf()` | Optional: provides master KDF for owned-node authentication |
| `get_OwnerKdf()` | Optional: provides owner public KDF (viewer authentication) |
| `OnOwnedNode(id, up)` | Called when ownership of a node is confirmed or lost |
| `OnEventsSerif(hash, height)` | Wallet-event checkpoint notification |

### `INetwork` (abstract)

The network abstraction. Callers post typed `Request` objects and receive results via an `IHandler` callback. The interface also provides:

- `Connect()` / `Disconnect()` — lifecycle
- `PostRequest(req, handler)` — enqueue a request
- `BbsSubscribe(channel, since, receiver)` — subscribe to a BBS channel
- `DependentSubscribe(bool)` — opt into dependent-state notifications

### `NetworkStd`

The production implementation. Holds a `ConnectionList` of open TCP `Connection` objects (one per configured node address). Configuration lives in `NetworkStd::Config`:

| Field | Default | Meaning |
|---|---|---|
| `m_vNodes` | — | Peer node addresses |
| `m_PollPeriod_ms` | 0 | 0 = persistent connection; >0 = poll interval |
| `m_ReconnectTimeout_ms` | 5 000 | Delay before reconnecting after a drop |
| `m_CloseConnectionDelay_ms` | 1 000 | After last request done in poll mode, wait briefly for BBS then close |
| `m_PreferOnlineMining` | true | Advertise `MiningFinalization` capability |
| `m_UseProxy` | false | Route through SOCKS proxy |

Requests are dispatched to whichever `Connection` is currently at the known tip (`IsAtTip()`). If multiple connections exist, the one with the most dependent-state context is preferred.

---

## Tip Synchronization

On every new connection and whenever the node sends a `NewTip` message, the fly client must determine whether its local header history is consistent with the node's current tip.

### Simple case

If the new tip's block number immediately follows the fly client's current tip, the new state is appended directly to history and `OnNewTip()` is fired.

### Full sync (reorg or cold start)

When there is a gap or a conflicting chain:

```
SearchBelow(height, n)
  → Send GetCommonState([IDs of n local states below height])
  ← Receive ProofCommonState(id, hardProof)
  → verify proof against current tip
  → binary-search for the fork point
  → RequestChainworkProof()
  → Send GetProofChainWork(lowerBound)
  ← Receive ProofChainWork(proof)
  → verify proof (IsValid())
  → PostChainworkProof(): prune invalidated local headers, add new ones
  → OnNewTip()
```

**`GetCommonState`** sends up to `n` locally known state IDs. The node responds with whichever of those IDs it can prove is part of its chain (via a `Merkle::HardProof`). If none match, `SearchBelow` recurses with twice as many candidates.

**Chain work proof** (`Block::ChainWorkProof`) proves that the node's claimed tip is part of a valid chain with at least `m_LowerBound` cumulative work. The proof contains:

| Field | Contents |
|---|---|
| `m_Heading` | A consecutive run of recent block headers (sequence prefix + elements) |
| `m_vArbitraryStates` | Sampled headers from earlier in the chain |
| `m_Proof` | `Merkle::MultiProof` linking all sampled states |
| `m_hvRootLive` | Hash bridging the history MMR to the definition MMR |
| `m_LowerBound` | Minimum chainwork the proof certifies |

The fly client calls `ChainWorkProof::IsValid()` which walks all embedded states, verifies each state's PoW, and checks the Merkle multi-proof for consistency. If valid, the returned tip must equal the `m_Tip` received in the `NewTip` message.

**Owned connections skip chain work proof.** When the fly client proves ownership of an owner key (see [Authentication](#authentication) below), the node is treated as trusted and `PostChainworkProof()` is called immediately without a round-trip.

**PBFT mode.** In networks using PBFT consensus (`Rules::Consensus::Pbft`), chain work proofs are irrelevant (block finality is determined by BFT votes, not accumulated work). The fly client still syncs headers but skips the chain work proof.

---

## Request Types

All requests derive from `FlyClient::Request`. The `Request::IHandler::OnComplete(Request&)` callback fires when the node responds. Each concrete request type pairs an outgoing message (`m_Msg`) with an incoming result (`m_Res`).

### Proof requests

| Request type | Out message | In message | What it verifies |
|---|---|---|---|
| `RequestUtxo` | `GetProofUtxo` (commitment, maturity) | `ProofUtxo` (vector of `Input::Proof`) | Each proof via `IsValidProofUtxo` against tip |
| `RequestKernel` | `GetProofKernel` (kernel ID hash) | `ProofKernel` (`TxKernel::LongProof`) | `IsValidProofKernel` against tip; state added to history |
| `RequestKernel2` | `GetProofKernel2` (ID, fetch flag) | `ProofKernel2` (Merkle proof + height + optional kernel) | Merkle proof against tip |
| `RequestKernel3` | `GetProofKernel3` (HeightPos) | `ProofKernel2` | — |
| `RequestAsset` | `GetProofAsset` (asset ID or owner) | `ProofAsset` (Asset::Full + Merkle proof) | `IsValidProofAsset` against tip |
| `RequestProofShieldedOutp` | `GetProofShieldedOutp` (serial pub) | `ProofShieldedOutp` | `IsValidProofShieldedOutp` against tip |
| `RequestProofShieldedInp` | `GetProofShieldedInp` (spend key) | `ProofShieldedInp` | `IsValidProofShieldedInp` against tip |

A proof response being empty (zero-length `Proof`) means the queried object does not exist in the node's view of the chain.

### Data requests

| Request type | Out message | In message | Notes |
|---|---|---|---|
| `RequestTransaction` | `NewTransaction` | `Status` | Only sent to nodes advertising `SpreadingTransactions`; requires ext ≥ 9 for context-dependent txs |
| `RequestEvents` | `GetEvents` (height) | `Events` (blob) | Only sent to **owned** connections |
| `RequestShieldedList` | `GetShieldedList` (id, count) | `ShieldedList` | — |
| `RequestStateSummary` | `GetStateSummary` | `StateSummary` | Node's TXO / UTXO / shielded counts |
| `RequestEnumHdrs` | `EnumHdrs` | `HdrPack` | Batch header download; headers are PoW-verified in parallel |
| `RequestBodyPack` / `RequestBody` | `GetBodyPack` | `BodyPack` / `Body` | Full block body download |
| `RequestContractVar` | `GetContractVar` | `ContractVar` | Single contract KV lookup |
| `RequestContractVars` | `ContractVarsEnum` | `ContractVars` | Enumeration of contract KV store |
| `RequestContractLogs` | `ContractLogsEnum` | `ContractLogs` | Contract log enumeration |
| `RequestContractLogProof` | `GetContractLogProof` | `ContractLogProof` | Proof of log entry |
| `RequestShieldedOutputsAt` | `GetShieldedOutputsAt` (height) | `ShieldedOutputsAt` | Shielded output count at height |
| `RequestAssetsListAt` | `GetAssetsListAt` (height, id) | `AssetsListAt` | Asset list at height |
| `RequestBbsMsg` | — | — | Outbound BBS message; mined by `BbsMiner` threads before sending |

### `RequestEnsureSync`

A pseudo-request with no wire message. It completes immediately once the connection is at tip. The `m_IsDependent` flag variant additionally waits for the dependent-state context to be received from the node.

---

## Authentication

After the TLS-style secure channel is established, the node sends an `Authentication` message with `IDType::Node` and its public node ID, signed by its node key.

The fly client may then prove ownership:

- **Owner key** (`IDType::Owner`): the client holds the master KDF and calls `ProveKdfObscured`. The node recognises this and flags the connection as `Owned`. Events (wallet scans) are only delivered over owned connections.
- **Viewer key** (`IDType::Viewer`): the client holds only the owner public KDF and calls `ProvePKdfObscured`. The node verifies via `IsPKdfObscured` and also flags the connection as `Owned`.

On a confirmed owned connection, `FlyClient::OnOwnedNode(nodeID, true)` is called. If the node loses ownership (e.g., key mismatch after reconnect), `OnOwnedNode(nodeID, false)` fires.

### Online mining finalization

If `m_PreferOnlineMining` is set, the fly client advertises `LoginFlags::MiningFinalization` during the handshake. The node may then call back with `GetBlockFinalization(height, fees)`, asking the wallet to construct and return a coinbase transaction for block assembly. The fly client responds with `BlockFinalization` containing the signed transaction. See [Node-Mining-Modes](Node-Mining-Modes) for the trade-offs between online and offline mining.

---

## BBS Integration

BBS (Secure Bulletin Board System) channels are subscribed through the network layer:

```cpp
network->BbsSubscribe(channel, timestampFrom, &receiver);
```

Internally, the active connection sends `BbsSubscribe` to the node for each subscribed channel. Incoming `BbsMsg` messages are routed to the corresponding `IBbsReceiver`.

Outbound BBS messages use `RequestBbsMsg`. Before sending, the `BbsMiner` worker pool computes the required proof-of-work nonce on background threads, then delivers the message once mined. In `FakePoW` networks (testing), the PoW step is skipped.

---

## `IHistory` — Header Store

Every `FlyClient` must supply a `Block::SystemState::IHistory` via `get_History()`. This is a sliding window of confirmed block headers used for:

- Determining whether the client is at tip (`get_Tip()`)
- Providing candidate state IDs during `SearchBelow`
- Storing states from proof responses (kernel proofs add states to history)
- Rolling back on reorg (`DeleteFrom(height)`)

The minimal in-memory implementation is `Block::SystemState::HistoryMap`. Wallets use their own DB-backed implementation (`WalletDB`).

---

## Connection Flags

Each `Connection` tracks a bitmask of state flags:

| Flag | Set when |
|---|---|
| `Flags::Node` | Node authentication message received |
| `Flags::Owned` | Owner/viewer key proof accepted by node |
| `Flags::ReportedConnected` | `OnNodeConnected(true)` already fired |
| `Flags::DependentPending` | Waiting for dependent-state context from node |

---

## Security Model and Limitations

**What is verified cryptographically:**
- Every UTXO, kernel, asset, shielded, and contract proof is checked against the `m_Tip` received from the node using the MMR-based `IsValidProof*` methods.
- Chain work proofs are cryptographically verified: sampled states' PoW is recomputed and the Merkle multi-proof is checked.
- Header packs (`EnumHdrs`) are PoW-verified in a parallel task pool before being accepted.

**What is trusted:**
- The fly client trusts that the node with the highest cumulative chain work (as proven by `ChainWorkProof`) is on the honest chain. A set of colluding adversarial miners with more than 50% of hash power could present a valid but dishonest chain work proof.
- Owned connections: once ownership is proven, the chain work proof step is skipped entirely. The client trusts the owned node fully for sync.
- PBFT finality: in Warp/dPoS networks, the client trusts PBFT signatures (`PbftStamp`) as confirmation.

**Eclipse attack surface:**
If all configured node addresses are controlled by an attacker, the fly client will sync to the attacker's chain (provided it has a valid chain work proof). Users should configure multiple independent node addresses.

**No full block validation:**
The fly client never downloads full block bodies (unless `RequestBody` is explicitly issued). It cannot detect invalid transactions or double-spends except via UTXO/kernel inclusion proofs. Full validation requires running a complete node.

---

## Polling vs Persistent Connection

| Mode | `PollPeriod_ms` | Behaviour |
|---|---|---|
| Persistent | 0 (default) | Connection stays open indefinitely; recommended for wallets that need real-time BBS/event delivery |
| Polling | > 0 | Connects, fetches pending requests, waits briefly for BBS (`CloseConnectionDelay_ms`), then disconnects. Reconnects after `max(Target_ms, PollPeriod_ms)`. |

---

## Related Pages

- [Core-Block-And-Chain-State](Core-Block-And-Chain-State) — `SystemState`, DMMR, chain work
- [Core-Merkle-Structures](Core-Merkle-Structures) — MMR proof structures used in verification
- [Node-P2P-Protocol](Node-P2P-Protocol) — full-node wire protocol that fly client messages ride on
- [Node-Mining-Modes](Node-Mining-Modes) — online vs offline mining, `MiningFinalization` detail
- [Wallet-SBBS](Wallet-SBBS) — BBS channel derivation and encryption
