# Secure Bulletin Board System (SBBS)

SBBS is the encrypted, asynchronous messaging layer that Beam wallets use to exchange transaction parameters and coordinate interactive signing. It is implemented as a service running inside the node (server side) and consumed by wallets (client side) via the fly-client protocol.

The key property of SBBS: messages are encrypted end-to-end using the recipient's public key. The relaying nodes never see plaintext, and no persistent connection between the two wallets is required.

---

## Overview

Each Beam wallet address encodes a 33-byte secp256k1 public key (`m_Pk`) and a 2-byte channel number (`m_Channel`). When wallet A wants to send a message to wallet B, it:

1. Encrypts the message payload under B's public key.
2. Posts the ciphertext to the SBBS channel derived from B's public key.
3. Any node that has BBS enabled stores and relays the message.
4. Wallet B, subscribed to its channel, receives the ciphertext and decrypts it with its private key.

---

## WalletID and BBS Address

```cpp
// wallet/core/common.h
#pragma pack(push, 1)
struct WalletID {
    uintBig_t<2> m_Channel;   // BBS channel number (0–1023 for wallet channels)
    ECC::Point   m_Pk;        // Recipient's BBS public key (33 bytes compressed)
    // Total: 35 bytes, encoded as hex
};
#pragma pack(pop)
```

The channel number is not stored independently — it is derived from the public key at address creation time:

```cpp
// wallet/core/common.cpp
void WalletID::SetChannelFromPk()
{
    BbsChannel ch;
    m_Pk.ExportWord<0>(ch);             // high-order word of the point's X coordinate
    ch %= proto::Bbs::s_MaxWalletChannels;  // modulo 1024
    m_Channel = ch;
}
```

A `WalletID` is considered valid only if `m_Pk` encodes a non-zero secp256k1 point and `m_Channel < 1024` (`proto::Bbs::s_MaxWalletChannels`).

### Channel Allocation

| Range | Purpose |
|-------|---------|
| 0 – 1023 | Wallet-to-wallet SBBS messages |
| 1024 (`s_SwapOffersChannel`) | Atomic swap offer book |
| 1027 (`s_BroadcastChannel`) | Node broadcast / announcements |
| 1028 (`s_DexOffersChannel`) | DEX order book |

The design intent (from source comments): at peak load ~1K transactions per block, with a 12-hour message lifetime, the system may simultaneously hold ~1M distinct transaction messages. Sharding into 1024 channels gives ~1K messages per channel — enough to reduce per-subscriber traffic while maintaining a meaningful anonymity set per channel.

---

## Encryption

BBS messages are encrypted with an ECDH-derived key and an AES-256 stream cipher authenticated by a Blake2b HMAC.

### Key Derivation (Diffie-Hellman)

```cpp
// core/proto.cpp — InitViaDiffieHellman
bool InitViaDiffieHellman(
    const ECC::Scalar::Native& myPrivate,
    const PeerID&              remotePublic,
    AES::Encoder&              enc,
    ECC::Hash::Mac&            hmac,
    AES::StreamCipher*         pCipherOut,
    AES::StreamCipher*         pCipherIn)
{
    // ECDH: shared point = remotePublic * myPrivate
    ECC::Point::Native ptSecret = p * myPrivate;

    // Hash the shared point to 32 bytes → symmetric key material
    ECC::Hash::Processor() << ptSecret >> hvSecret;

    enc.Init(hvSecret.m_pData);          // AES-256 key
    hmac.Reset(hvSecret.m_pData, 32);    // Blake2b-HMAC key

    // IV for sender stream: Hash(hvSecret || remotePublic)
    // IV for receiver stream: Hash(hvSecret || myPublic)
}
```

### Encrypt (`proto::Bbs::Encrypt`)

```cpp
bool Bbs::Encrypt(
    ByteBuffer& res,
    const PeerID& publicAddr,   // recipient's BBS public key
    ECC::Scalar::Native& nonce, // ephemeral sender private key (randomized per message)
    const void* p, uint32_t n)
{
    PeerID myPublic;
    myPublic.FromSk(nonce);   // ephemeral sender public key

    // Derive shared secret via ECDH
    InitViaDiffieHellman(nonce, publicAddr, enc, hmac, &cOut, NULL);

    // Compute HMAC over plaintext
    hmac.Write(p, n);
    hmac >> hvMac;   // 32-byte authentication tag

    // Layout: [ myPublic (33 B) | AES_stream(hvMac (32 B) | plaintext) ]
    res.resize(33 + 32 + n);
    memcpy(res.data(),      myPublic, 33);
    memcpy(res.data() + 33, hvMac,   32);
    memcpy(res.data() + 65, p,        n);
    cOut.XCrypt(enc, res.data() + 33, 32 + n);  // encrypt MAC + payload together
}
```

Wire layout of the `m_Message` field inside `BbsMsg`:

```
[ sender_ephemeral_pubkey (33 bytes) | AES256_stream( blake2b_mac (32 bytes) | plaintext ) ]
```

### Decrypt (`proto::Bbs::Decrypt`)

The receiver:

1. Parses the first 33 bytes as the sender's ephemeral public key.
2. Performs ECDH using its own private BBS key and that ephemeral public key.
3. Derives the same AES key and HMAC key.
4. Decrypts the remainder (MAC + payload).
5. Recomputes the HMAC over the decrypted payload.
6. Returns `true` only if the stored MAC equals the recomputed MAC — this is authenticated decryption.

Because the channel holds messages for many addresses, a wallet must attempt decryption for every message on its channel, succeeding only for messages addressed to its own `m_Pk`.

---

## Wire Protocol Messages

BBS messages are a subset of the node P2P protocol (`core/proto.h`). Peers that support BBS set the `LoginFlags::Bbs` bit in their handshake.

### `BbsMsg` (opcode `0x3f`)

```cpp
#define BeamNodeMsg_BbsMsg(macro) \
    macro(BbsChannel,       Channel)    \
    macro(Timestamp,        TimePosted) \
    macro(ByteBuffer,       Message)    \
    macro(Bbs::NonceType,   Nonce)      // 4-byte PoW nonce
```

Used both to post a new message and to relay it between nodes.

### `BbsHaveMsg` (opcode `0x39`) / `BbsGetMsg` (opcode `0x3a`)

```cpp
#define BeamNodeMsg_BbsHaveMsg(macro)  macro(BbsMsgID, Key)
#define BeamNodeMsg_BbsGetMsg(macro)   macro(BbsMsgID, Key)
```

Gossip pair: a node that stores a new message sends `BbsHaveMsg` to its peers. A peer that does not have that key responds with `BbsGetMsg`; the sender replies with the full `BbsMsg`.

### `BbsSubscribe` (opcode `0x3b`)

```cpp
#define BeamNodeMsg_BbsSubscribe(macro) \
    macro(BbsChannel, Channel)          \
    macro(Timestamp,  TimeFrom)         \
    macro(bool,       On)
```

Client → Node subscription management. The node delivers all stored messages for `Channel` with `TimePosted >= TimeFrom`, then forwards future messages as they arrive.

### `BbsResetSync` (opcode `0x3e`)

```cpp
#define BeamNodeMsg_BbsResetSync(macro) macro(Timestamp, TimeFrom)
```

Sent by a reconnecting wallet to request re-delivery of all BBS messages since `TimeFrom`. The wallet persists per-channel timestamps in `TimestampHolder` across sessions to minimize re-delivery.

### `BbsPickChannel` / `BbsPickChannelRes`

Client asks for a recommended channel. The node returns the least-populated channel with fewer than 100 active listeners (checked hourly). This is advisory — the wallet may use any channel derived from its key.

---

## Anti-Spam Proof of Work

Each `BbsMsg` must include a valid 4-byte `Nonce` such that the message hash satisfies a PoW condition:

```cpp
// core/proto.cpp
void Bbs::get_Hash(ECC::Hash::Value& hv, const BbsMsg& msg)
{
    ECC::Hash::Processor hp;
    hp << "bbs.msg" << msg.m_Channel << Blob(msg.m_Message)
       << msg.m_TimePosted << msg.m_Nonce
       >> hv;
}

bool Bbs::IsHashValid(const ECC::Hash::Value& hv)
{
    uint32_t nHigh;
    hv.ExportWord<0>(nHigh);
    return nHigh < (1 << 10);  // upper 22 bits must be zero; ~1-in-4M probability
}
```

Nodes with protocol version ≥ 1 verify the PoW before storing or relaying a message. The `BbsMsgID` (used for deduplication) is this hash.

---

## Node Relay Mechanics

### Message Lifetime (TTL)

```cpp
// node/node.h
struct Bbs {
    uint32_t m_MessageTimeout_s = 3600 * 12;  // 12 hours default
    uint32_t m_CleanupPeriod_ms = 3600 * 1000; // cleanup every hour
    NodeDB::BbsTotals m_Limit {
        .m_Count = 20'000'000,  // max stored messages
        .m_Size  = 5ULL << 30,  // max 5 GB total
    };
};
```

A `BbsMsg` is rejected if:

- `m_TimePosted > now + MaxAhead_s` (clock skew guard, max ~2 h)
- `m_TimePosted + m_MessageTimeout_s < now` (message is expired)
- The PoW hash is invalid (anti-spam)
- `m_Message.size() > s_MaxMsgSize` (1 MB hard cap)

Cleanup runs hourly. When the node is at the storage limit (`m_Totals >= m_Limit`) it evicts oldest messages first.

### Deduplication

The `BbsMsgID` (Blake2b hash of channel + payload + timestamp + nonce) serves as the deduplication key. Before storing, the node checks `NodeDB::BbsFind(key)`. Duplicate `BbsHaveMsg` notifications trigger a `BbsGetMsg` request only if the key is not already in the local DB or wait-list.

### Subscription Delivery

When a client sends `BbsSubscribe(channel, timeFrom, on=true)`:

1. Node creates a `Node::Bbs::Subscription` entry indexed by channel.
2. Any stored message on that channel with `TimePosted >= timeFrom` is sent immediately.
3. Future `BbsMsg` arrivals on that channel are forwarded to all active subscribers.

---

## Client-Side Architecture

### Class Hierarchy

```
WalletNetworkViaBbs
├── BaseMessageEndpoint        (address registration, decrypt-on-receive)
│     ├── AddOwnAddress()      → derive BBS key, subscribe to channel
│     ├── Send()               → Bbs::Encrypt() + submit BbsMsg
│     └── ProcessMessage()     → try Bbs::Decrypt() for each registered key
└── BbsProcessor               (fly-client send queue, channel sub/unsub)
      ├── Send()               → queues WalletRequestBbsMsg
      ├── SubscribeChannel()   → sends BbsSubscribe to node
      └── OnMsg()              → dispatched to BaseMessageEndpoint
```

### Address Registration

```cpp
void BaseMessageEndpoint::AddOwnAddress(const WalletAddress& address)
{
    // Private BBS key comes from wallet KDF (Key::Type::Bbs, index)
    // Public key = G * sk_bbs
    // Channel = ExportWord<0>(pubkey) % 1024

    Addr* pAddr = CreateAddr(address.m_BbsAddr, handler);
    // If this is the first address on this channel → SubscribeChannel()
}
```

The wallet may register multiple addresses. Each address maps to exactly one channel; a channel may have multiple registered addresses. The wallet subscribes to a channel exactly once regardless of how many of its addresses share it (`IsSingleChannelUser` guard).

### Timestamp Persistence

`TimestampHolder` maintains an in-memory map of `BbsChannel → Timestamp` and flushes it to `WalletDB` (variable key `"BbsTimestamps"`) on a timer. On reconnection it sends `BbsResetSync` with the persisted timestamp, avoiding re-processing of already-seen messages.

---

## Privacy Properties and Limitations

| Property | Status |
|----------|--------|
| Message content confidentiality | Strong — ECDH + AES-256 + Blake2b HMAC; node cannot decrypt |
| Sender anonymity from recipient | Strong — ephemeral sender key per message; recipient sees only a fresh public key |
| Receiver anonymity from node | Partial — node sees the channel subscription; ~1024 wallets share a channel, so the channel leaks a coarse group membership |
| Sender anonymity from node | Partial — node sees the source IP and the channel of the posted message |
| Traffic analysis | Node can correlate subscription → channel → address group; TOR transport mitigates IP leakage |
| Message availability | Wallets offline > 12 h may miss messages (TTL expiry); senders may need to retry |
| Replay protection | `BbsMsgID` deduplication prevents exact replay; PoW nonce is committed to in the hash |
| Forward secrecy | None — if the recipient's static BBS private key is compromised, past messages stored by nodes can be decrypted |

### Comparison to Direct Transport

The [Wallet Architecture](Wallet-Architecture) page describes how newer address types (max-privacy, public-offline) bypass SBBS entirely, using the node as a one-way relay or requiring no online round-trip. SBBS is used for:

- Classic regular addresses (interactive signing with two live parties)
- Swap offer discovery (`s_SwapOffersChannel`)
- DEX order propagation (`s_DexOffersChannel`)
- Node broadcast messages (`s_BroadcastChannel`)

---

## Related Pages

- [Wallet Architecture](Wallet-Architecture) — overall wallet engine, how SBBS fits into the network layer
- [Node P2P Protocol](Node-P2P-Protocol) — full message framing and peer management
- [Node Fly Client Protocol](Node-Fly-Client-Protocol) — how wallets connect to nodes to post and receive BBS messages
- [Addresses in Beam](Addresses-in-Beam) — address types and when SBBS is used vs. direct transport
