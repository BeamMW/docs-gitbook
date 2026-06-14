# Node P2P Protocol

This page documents the node-to-node wire protocol used by Beam full nodes: message framing, the connection handshake, all message types, peer selection, and the Dandelion++ transaction-privacy relay.

**Source references:** `core/proto.h`, `core/peer_manager.h/.cpp`, `p2p/protocol.h`, `p2p/protocol_base.h`, `p2p/msg_serializer.h`, `node/txpool.h`, `node/node.h`

---

## Message Framing

Every message on the wire is prefixed by an 8-byte header defined in `p2p/protocol_base.h`:

```cpp
struct MsgHeader {
    static constexpr size_t SIZE = 8;  // always 8 bytes

    uint8_t  V0, V1, V2;   // protocol version / magic bytes
    MsgType  type;          // 1-byte message type code
    uint32_t size;          // little-endian body length in bytes
};
```

Encoding is **little-endian**. The version bytes act as a combined magic number and protocol version. Any message whose V0/V1/V2 do not match the node's configured version is immediately rejected with a `version_error`.

The `p2p/Protocol` class owns a **dispatch table** indexed by `MsgType` (a `uint8_t`, so up to 256 types). Each entry records the registered callback, size bounds (`minSize`/`maxSize`), and a pointer to the handler object. Unknown or out-of-range types produce `msg_type_error`; bodies outside the declared size bounds produce `msg_size_error`.

### Serialization

Message bodies are serialized using the [yas](https://github.com/niXman/yas) binary archive (`SERIALIZE_OPTIONS`). Each struct exposes a `serialize(Archive& ar)` method that reads or writes all fields in declaration order. The `MsgSerializer` / `MsgSerializeOstream` classes accumulate fragments until `finalize()` is called, at which point the body length is written back into the header placeholder.

---

## Encryption: ProtocolPlus

`proto::ProtocolPlus` (defined in `core/proto.h`) extends the base `Protocol` with a **per-connection AES stream cipher and HMAC**:

```
                      Handshake
     Plaintext ──────────────────► Outgoing  ──► Duplex
     (before SChannel)  (encrypt out only)   (encrypt both directions)
```

Fields involved:
| Field | Role |
|---|---|
| `m_MyNonce` / `m_RemoteNonce` | ECDH nonce pair for key agreement |
| `m_Enc` | `AES::Encoder` for key derivation |
| `m_CipherIn` / `m_CipherOut` | `AES::StreamCipher` for the two directions |
| `m_HMac` | 8-byte HMAC appended to every encrypted message |

Once the `SChannelReady` message is exchanged, all subsequent traffic is encrypted. The HMAC is verified on every received message before dispatch.

---

## Connection Lifecycle

`proto::NodeConnection` manages a single TCP connection end-to-end. The typical lifecycle is:

### 1. TCP Connect / Accept

```cpp
void NodeConnection::Connect(const io::Address& addr, ...);
void NodeConnection::Accept(io::TcpStream::Ptr&& newStream);
```

After the TCP socket is established, both sides immediately start the secure-channel handshake.

### 2. Secure Channel Handshake

| Step | Initiator → Responder | Message |
|---|---|---|
| 1 | → | `SChannelInitiate { NoncePub }` — ECDH ephemeral public key |
| 2 | ← | `SChannelReady` — responder has computed the shared secret; encryption starts |
| 3 | → | `SChannelReady` — initiator confirms; both sides now in `Duplex` mode |

`SecureConnect()` triggers step 1. `GenerateSChannelNonce()` must be overridden by subclasses to provide the ephemeral key.

### 3. Authentication

After the secure channel is established, a peer may prove its identity:

```cpp
void NodeConnection::ProveID(ECC::Scalar::Native&, uint8_t nIDType);
```

This sends an `Authentication { ID, IDType, Sig }` message. The signature is over the session's shared nonce material. Three identity types are defined:

| Code | Meaning |
|---|---|
| `'N'` | Regular node |
| `'O'` | Owner (wallet connected to its own node) |
| `'V'` | Viewer |

### 4. Login

After authentication, each side sends a `Login` message that announces its capabilities:

```cpp
struct Login {
    std::vector<ECC::Hash::Value> Cfgs;  // hash of Rules/consensus config
    uint32_t                      Flags;
};
```

#### Login Flags

| Flag | Value | Meaning |
|---|---|---|
| `SpreadingTransactions` | `0x1` | Node is relaying txs; peer should send new ones |
| `Bbs` | `0x2` | Node is relaying BBS (SBBS) messages |
| `SendPeers` | `0x4` | Request periodic peer-list updates |
| `MiningFinalization` | `0x8` | Request `GetBlockFinalization` flow (online mining mode) |
| `WantDependentState` | `0x10000` | Request `DependentContextChanged` notifications |

#### Protocol Extension Level

Bits 4–15 of `Flags` encode a protocol extension version (`LoginFlags::Extension`). The current range is **Minimum = 8, Maximum = 11**. Each level gates support for specific message types:

| Level | Capabilities added |
|---|---|
| 8 | Contract vars/logs, flexible header requests, newer ShieldedList, Status codes |
| 9 | Dependent transaction messages |
| 10 | `GetAssetsListAt` |
| 11 | `GetProofKernel3` |

The `Cfgs` field carries the Blake2b hash of the node's `Rules` struct. A peer with an incompatible configuration hash triggers a `NodeProcessingException::Type::Incompatible` disconnect.

### 5. Disconnection

`Bye { Reason }` is sent before a clean disconnect. Reason codes:

| Code | Meaning |
|---|---|
| `'s'` | Node is stopping |
| `'b'` | Peer is banned |
| `'L'` | Loopback connection detected |
| `'d'` | Duplicate connection |
| `'t'` | Timeout |
| `'p'` | Probed (probe-only connection) |
| `'o'` | Other |

The `DisconnectReason` enum also tracks unclean disconnects: `Io`, `Protocol`, `ProcessingExc`, `Bye`, `Drown` (outbound buffer overflow).

---

## Message Reference

All messages are declared in `core/proto.h` via the `BeamNodeMsgsAll` macro. The byte codes are stable across protocol versions.

### Session / Control Messages

| Code | Message | Fields | Direction |
|---|---|---|---|
| `0x01` | `Bye` | `Reason: uint8_t` | both |
| `0x02` | `Ping` | *(empty)* | both |
| `0x03` | `Pong` | *(empty)* | both |
| `0x04` | `SChannelInitiate` | `NoncePub: PeerID` | both |
| `0x05` | `SChannelReady` | *(empty)* | both |
| `0x06` | `Authentication` | `ID: PeerID`, `IDType: uint8_t`, `Sig: ECC::Signature` | both |
| `0x07` | `PeerInfoSelf` | `Port: uint16_t` | both |
| `0x08` | `PeerInfo` | `ID: PeerID`, `LastAddr: io::Address` | both |
| `0x09` | `GetExternalAddr` | *(empty)* | → |
| `0x0a` | `ExternalAddr` | `Value: uint32_t` (IP as int) | ← |
| `0x0b` | `GetTime` | *(empty)* | → |
| `0x0c` | `Time` | `Value: Timestamp` | ← |
| `0x0d` | `DataMissing` | *(empty)* | ← |
| `0x0f` | `Login` | `Cfgs: vector<Hash>`, `Flags: uint32_t` | both |
| `0x44` | `Status` | `Value: uint8_t`, `ExtraInfo: string` | ← |

`Status` is returned after `NewTransaction` with a `TxStatus` code (see [Transaction status codes](#transaction-status-codes) below).

### Blockchain Sync Messages

| Code | Message | Fields |
|---|---|---|
| `0x10` | `NewTip` | `Description: Block::SystemState::Full` |
| `0x11` | `GetHdr` | `ID: Block::SystemState::ID` |
| `0x12` | `Hdr` | `Description: Block::SystemState::Full` |
| `0x13` | `GetHdrPack` | `Top: SystemState::ID`, `Count: uint32_t` (max 2048) |
| `0x14` | `HdrPack` | `Prefix: Sequence::Prefix`, `vElements: vector<Sequence::Element>` |
| `0x15` | `GetBody` | `ID: Block::SystemState::ID` |
| `0x16` | `Body` | `Body: BodyBuffers` |
| `0x26` | `GetBodyPack` | `Top`, `FlagP`, `FlagE`, `CountExtra`, `Block0`, `HorizonLo1`, `HorizonHi1` |
| `0x27` | `BodyPack` | `Bodies: vector<BodyBuffers>` |
| `0x33` | `EnumHdrs` | `Height: HeightRange` |

`BodyBuffers` splits a block body into two byte arrays:
- `m_Perishable` — inputs and outputs (pruneable at the horizon)
- `m_Eternal` — kernels (retained indefinitely)

Body request `FlagP` / `FlagE` control which part is returned: `Full=0` (both), `None=1` (neither), `Recovery1=2` (outputs only, for archive nodes).

### Proof Messages

These implement the fly-client SPV proof protocol (see [Node-Fly-Client-Protocol](Node-Fly-Client-Protocol.md)):

| Request | Response | What it proves |
|---|---|---|
| `GetProofState (0x17)` | `ProofState (0x18)` | Block header inclusion in DMMR |
| `GetCommonState (0x22)` | `ProofCommonState (0x23)` | Binary-search for chain fork point |
| `GetProofKernel (0x19)` | `ProofKernel (0x1a)` | Kernel inclusion (hash only) |
| `GetProofKernel2 (0x24)` | `ProofKernel2 (0x25)` | Kernel inclusion + kernel body |
| `GetProofKernel3 (0x2b)` | *(reuses ProofKernel2)* | Kernel by height position |
| `GetProofUtxo (0x1b)` | `ProofUtxo (0x1c)` | UTXO inclusion in Radix tree |
| `GetProofChainWork (0x1d)` | `ProofChainWork (0x1e)` | Chain-work proof (FlyClient) |
| `GetProofShieldedOutp (0x28)` | `ProofShieldedOutp (0x29)` | Shielded output in DMMR |
| `GetProofShieldedInp (0x20)` | `ProofShieldedInp (0x21)` | Shielded nullifier inclusion |
| `GetProofAsset (0x35)` | `ProofAsset (0x36)` | Confidential Asset registration |

`GetProofUtxo` accepts a `MaturityMin` parameter for paginated UTXO proof retrieval when the result set is large.

### Contract / BVM Query Messages

| Request | Response | Purpose |
|---|---|---|
| `ContractVarsEnum (0x1f)` | `ContractVars (0x2d)` | Paginated key-value contract state scan |
| `GetContractVar (0x38)` | `ContractVar (0x3c)` | Single contract variable with Merkle proof |
| `ContractLogsEnum (0x40)` | `ContractLogs (0x41)` | Paginated contract log scan |
| `GetContractLogProof (0x42)` | `ContractLogProof (0x43)` | Merkle proof for a log entry |

All range queries return `bMore: bool` indicating whether the result was truncated.

### Owner-Channel Messages

These messages are only sent over connections authenticated with `IDType::Owner` or when the `MiningFinalization` login flag is set:

| Code | Message | Purpose |
|---|---|---|
| `0x2c` | `GetEvents { HeightMin }` | Request wallet UTXO/shielded events |
| `0x34` | `Events { Events: ByteBuffer }` | Packed event list (up to 1024 per message) |
| `0x37` | `EventsSerif { Value, Height }` | Checkpoint hash for event stream integrity |
| `0x2e` | `GetBlockFinalization { Height, Fees }` | Request coinbase UTXO for online mining mode |
| `0x2f` | `BlockFinalization { Value: Transaction::Ptr }` | Signed coinbase transaction from wallet |

The Events stream carries three event types: `Utxo` (coin created/spent), `Shielded` (shielded coin), and `AssetCtl` (asset emission change).

### Transaction Broadcast Messages

| Code | Message | Fields |
|---|---|---|
| `0x30` | `NewTransaction0` | `Transaction::Ptr`, `Fluff: bool` (legacy) |
| `0x49` | `NewTransaction` | `Transaction::Ptr`, `Context: unique_ptr<Hash>`, `Fluff: bool` |
| `0x31` | `HaveTransaction` | `ID: Transaction::KeyType` |
| `0x32` | `GetTransaction` | `ID: Transaction::KeyType` |
| `0x4a` | `SetDependentContext` | `Context: unique_ptr<Hash>` |
| `0x4b` | `DependentContextChanged` | `vCtxs: vector<Hash>`, `PrefixDepth: uint32_t` |

`HaveTransaction` / `GetTransaction` implement a pull-based relay: a node announces it has a transaction; interested peers request it. The `Fluff: bool` field in `NewTransaction` allows the sender to bypass the Dandelion++ stem phase and broadcast directly.

### Transaction Status Codes

Returned in the `Status` message (`Value` field) after `NewTransaction`:

| Code | Constant | Meaning |
|---|---|---|
| `0x00` | `Unspecified` | No status (legacy compatibility) |
| `0x01` | `Ok` | Accepted |
| `0x02` | `TooSmall` | Missing required elements (input + kernel, or output + kernel) |
| `0x03` | `Obscured` | Overlap with another tx; dropped to avoid collision |
| `0x10` | `Invalid` | Context-free validation failed |
| `0x11` | `InvalidContext` | Invalid in chain context (timelock, relative lock height) |
| `0x12` | `LowFee` | Fee below minimum |
| `0x13` | `LimitExceeded` | Block limit exceeded (tx too large, too many shielded I/O) |
| `0x14` | `InvalidInput` | Non-existent or non-matured input referenced |
| `0x30`–`0x3e` | `ContractFail` range | Contract execution failure (code encodes failure type) |
| `0x3f` | `ContractFailNode` | Non-existent contract, duplicate create, or destructor left garbage |
| `0x48` | `DependentNoParent` | Dependent tx references unknown parent context |
| `0x49` | `DependentNotBest` | Tx valid but loses to a competing dependent tx |
| `0x4a` | `DependentNoNewCtx` | Duplicate new context; kernel not marked as dependent |

### BBS / SBBS Messages

| Code | Message | Fields |
|---|---|---|
| `0x3f` | `BbsMsg` | `Channel: BbsChannel`, `TimePosted: Timestamp`, `Message: ByteBuffer`, `Nonce: uintBig_t<4>` |
| `0x39` | `BbsHaveMsg` | `Key: BbsMsgID` |
| `0x3a` | `BbsGetMsg` | `Key: BbsMsgID` |
| `0x3b` | `BbsSubscribe` | `Channel: BbsChannel`, `TimeFrom: Timestamp`, `On: bool` |
| `0x3e` | `BbsResetSync` | `TimeFrom: Timestamp` |

Max BBS message body: 1 MiB. Wallet channels are sharded across up to 1024 channels (`s_MaxWalletChannels`). Special channels: swap offers (`s_SwapOffersChannel`), broadcast (`s_BroadcastChannel`), DEX offers (`s_DexOffersChannel`). The `Nonce` field enables proof-of-work spam filtering. See [Wallet-SBBS](../wallet/Wallet-SBBS.md) for the wallet-side encryption details.

### Statistics Messages

| Code | Message | Purpose |
|---|---|---|
| `0x45` | `GetStateSummary` / `0x46 StateSummary` | Node TXO counts, shielded pool size, asset summary |
| `0x47` | `GetShieldedOutputsAt` / `0x48 ShieldedOutputsAt` | Shielded output count at a given height |
| `0x4c` | `GetAssetsListAt` / `0x4d AssetsListAt` | Paginated active asset list at height |

`StateSummary` fields: `TxoLo` (0 = archive node), `Kernels`, `Txos`, `Utxos`, `ShieldedOuts`, `ShieldedIns`, `AssetsMax`, `AssetsActive`.

### PBFT / Warp Messages

Used for the Beam Warp dPoS consensus layer (see [Consensus-Beam-Warp-dPoS](../consensus/Consensus-Beam-Warp-dPoS.md)):

| Code | Message | Fields |
|---|---|---|
| `0x51` | `PbftRoundStart` | `iRound`, `Address`, `NoncePub`, `Signature`, `IsCommitted` |
| `0x52` | `PbftProposal` | `iRound`, `Signature`, block header + body |
| `0x53` | `PbftVote` | `iRound`, `Signature`, `iKind`, `Address` |
| `0x54` | `PbftStamp` | `ValidatorSet`, `hvVsNext` (next validator set hash) |
| `0x55` | `PbftSigRequest` | `iRound`, `Mask`, `Signature` |
| `0x56` | `PbftSig` | `iRound`, `Address`, `Signature: ECC::Scalar` |
| `0x57` | `PbftPeerAssessment` | `From`, `Height`, `Signature`, `Reputation` map |

---

## Peer Manager

`PeerManager` (`core/peer_manager.h`) maintains a rated set of known peers and decides which to connect to.

### Rating System

Peer quality is tracked on a **logarithmic bandwidth scale**:

```
Rating = A × log(Bps / norm)    where A = 172, norm = 255 Bps
```

| Constant | Value | Meaning |
|---|---|---|
| `Rating::Initial` | 1024 | Default rating for unknown peers (~100 KBps) |
| `Rating::PenaltyNetworkErr` | 128 | Deducted on quick-disconnect error |
| `Starvation_s_ToRatio` | 1 | Rating bonus per second since last use |

Two parallel ratings are maintained:
- **Raw rating** — based purely on behavior (speed, reliability)
- **Adjusted rating** — raw + starvation bonus (elapsed time since last data request)

### Peer Selection

`PeerManager::Update()` runs periodically and selects two overlapping groups:
1. **Group 1** — top `m_DesiredHighest` (default **5**) peers by raw rating → stable, best-quality connections
2. **Group 2** — top `m_DesiredTotal` (default **10**) peers by adjusted rating → rotates in underused peers

Peers not selected in either group are disconnected, subject to a minimum connection time of `m_TimeoutDisconnect_ms` (default **2 minutes**) to prevent thrashing.

### Banning

A peer is banned (rating set to **0**) on:
- Any protocol violation
- Receiving an invalid block from the peer
- Incompatible consensus configuration

Banned peers cannot be reconnected for `m_TimeoutBan_ms` (default **10 minutes**). After the ban expires the peer is restored to rating **1** and becomes eligible again.

### Configuration

```cpp
struct PeerManager::Cfg {
    uint32_t m_DesiredHighest      = 5;               // top-rated group size
    uint32_t m_DesiredTotal        = 10;              // total active peers
    uint32_t m_TimeoutDisconnect_ms = 1000 * 60 * 2; // min connection time before penalty
    uint32_t m_TimeoutReconnect_ms  = 1000;           // min wait before reconnect attempt
    uint32_t m_TimeoutBan_ms        = 1000 * 60 * 10;// ban duration
    uint32_t m_TimeoutAddrChange_s  = 60 * 60 * 2;   // how often to update peer address
    uint32_t m_TimeoutRecommend_s   = 60 * 60 * 10;  // peer gossip interval
};
```

---

## Dandelion++ Transaction Relay

Beam implements Dandelion++ to obscure the network origin of transactions. The implementation lives in `node/node.h` (`Node::Config::Dandelion`, `Node::Dandelion`) and `node/txpool.h` (`TxPool::Stem`, `TxPool::Fluff`).

### Phases

```
NewTransaction received
        │
        ▼
   [Stem phase]  ──────────────────────────────────► single peer relay
        │         (10% fluff probability per hop)
        │  stem timeout or fluff probability triggered
        ▼
   [Fluff phase] ─────────────────────────────────► broadcast to all peers
```

**Stem phase (`TxPool::Stem`):** The transaction is held in the stem pool and forwarded to exactly one randomly selected peer. At each hop, there is a configurable probability (`m_FluffProbability ≈ 10%`) that the transaction transitions to the fluff phase instead.

**Fluff phase (`TxPool::Fluff`):** The transaction is added to the standard mempool and broadcast to all connected peers via `NewTransaction` / `HaveTransaction` gossip.

**Auto-fluff:** If a stem transaction is not included in a block within `m_dhStemConfirm + 1` blocks (default **3 blocks**), it is automatically promoted to the fluff phase. This prevents transactions from being lost if the stem path breaks.

### Aggregation

While in the stem phase, Beam extends Dandelion++ with **Mimblewimble transaction aggregation**:

- Transactions in the stem pool are merged with other pending stem transactions (cut-through applied)
- Aggregation continues until `m_OutputsMin` (default **5**) outputs are reached, or `m_AggregationTime_ms` (default **10 seconds**) elapses
- Aggregation is capped at `m_OutputsMax` (default **40**) outputs

This breaks the graph between individual senders even on the network level.

### Decoy UTXOs

To improve privacy, the node creates **dummy outputs** with the miner key during the stem phase. These are spent later (randomly, between `m_DummyLifetimeLo = 720` and `m_DummyLifetimeHi = 10080` blocks), making the transaction graph harder to analyze. Decoy creation requires `m_Keys.m_pMiner` to be set; setting `m_DummyLifetimeHi = 0` disables decoys entirely.

**When decoys are added.** Dummy outputs are not added to every transaction — they are the *padding* for an aggregation that did not fill up. When the aggregation timer fires (`Dandelion::OnTimedOut`) and the stem transaction is still below `m_OutputsMin`, the node calls `AddDummyOutputs()` to top it up to `m_OutputsMin` before fluffing. The number added is therefore `m_OutputsMin − (current outputs)`, further capped by the transaction's remaining fee reserve: each dummy output costs a standard output fee, and the node stops once the reserve can no longer cover one (so a low-fee transaction may get fewer than the gap). Each dummy is recorded in `NodeDB::Dummies` with a scheduled spend height sampled from `[m_DummyLifetimeLo, m_DummyLifetimeHi]`.

**When decoys are spent.** Spending is *opportunistic*, not scheduled. At the start of each stem aggregation, `AddDummyInputs()` pulls in the dummy with the lowest scheduled spend height that has come due (`GetLowestDummy() ≤ tip`) and adds it as an input. A dummy whose height has passed is therefore *eligible* to be spent but waits for the node's next outgoing/relayed transaction to carry it — so on a node that isn't actively relaying, due dummies accumulate in `NodeDB::Dummies` until the next aggregation consumes them. Spending a dummy deletes it from the table (the kernel-less output is later cut through), which is why dummies leave no permanent chain footprint.

### Configuration

```cpp
struct Node::Config::Dandelion {
    uint16_t m_FluffProbability  = 0x1999; // ≈10% (normalized to 0xFFFF)
    uint32_t m_dhStemConfirm     = 2;      // auto-fluff after N+1 blocks without mining
    uint32_t m_AggregationTime_ms = 10000; // aggregation window
    uint32_t m_OutputsMin        = 5;      // outputs required before fluffing
    uint32_t m_OutputsMax        = 40;     // max aggregation size
    uint32_t m_DummyLifetimeLo   = 720;    // min blocks before dummy is spent
    uint32_t m_DummyLifetimeHi   = 10080;  // max blocks (set 0 to disable)
};
```

---

## Related Pages

- [Node-Architecture](Node-Architecture.md) — block processing, mempool integration
- [Node-Fly-Client-Protocol](Node-Fly-Client-Protocol.md) — SPV proof types requested over this protocol
- [Core-Block-And-Chain-State](../core/Core-Block-And-Chain-State.md) — `SystemState`, DMMR, chain work
- [Wallet-SBBS](../wallet/Wallet-SBBS.md) — BBS channel encryption
- [Consensus-Beam-Warp-dPoS](../consensus/Consensus-Hard-Forks.md) — PBFT messages context
