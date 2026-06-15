# Wallet Key Keeper

The Key Keeper is the security boundary inside the Beam wallet. All private key material lives on one side of this boundary; transaction construction and network I/O live on the other. The boundary is expressed as a single C++ interface, `IPrivateKeyKeeper2`, which has four concrete implementations targeting different threat models.

---

## `IPrivateKeyKeeper2` Interface

Defined in `wallet/core/private_key_keeper.h`.

```cpp
struct IPrivateKeyKeeper2 {
    // Status codes returned by every method
    struct Status {
        static const Type Success      = 0;
        static const Type InProgress   = -1;   // async, not yet done
        static const Type Unspecified  = 1;
        static const Type UserAbort    = 2;
        static const Type NotImplemented = 3;
    };

    // Nonce slot identifier (hardware wallets have a fixed number of slots)
    struct Slot { typedef uint32_t Type; static const Type Invalid; };

    // KDF flavour requested from the device
    enum struct KdfType { Root, Sbbs };
};
```

Every method defined by the interface exists in two forms, expanded from the `KEY_KEEPER_METHODS` macro:

```cpp
virtual Status::Type InvokeSync(Method::X&);     // blocking
virtual void         InvokeAsync(Method::X&,     // non-blocking
                                 const Handler::Ptr&) = 0;
```

`InvokeSync` has a default implementation that blocks until the matching `InvokeAsync` completes. Most callers use the async form.

### Method Table

| Method | Direction | Description |
|---|---|---|
| `get_Kdf` | Out → host | Return root or SBBS KDF (public half always; private half only in trusted mode) |
| `get_NumSlots` | Out → host | Number of nonce slots available on the device |
| `get_Commitment` | Out → host | Pedersen commitment for a given `CoinID` |
| `CreateOutput` | Out → host | Full UTXO output (commitment + [bulletproof](../HW-wallet-design.md)) for a `CoinID` |
| `CreateInputShielded` | Out → host | Lelantus spend proof for a shielded coin |
| `CreateVoucherShielded` | Out → host | One or more shielded vouchers for offline/max-privacy sends |
| `CreateOfflineAddr` | Out → host | Public generator for offline address + ownership signature |
| `SignReceiver` | Out → host | Sign the receive side of an interactive transaction |
| `SignSender` | Out → host | Sign the send side; requires a nonce slot, two-round protocol |
| `SignSendShielded` | Out → host | Sign a shielded push (shield) transaction |
| `SignSplit` | Out → host | Sign a self-transfer (fee-only spend) |
| `DisplayEndpoint` | → device | Show an endpoint on the hardware device screen |

### Signing Methods in Detail

**`SignSender`** is the only method that uses a nonce slot and requires two rounds:

1. First call: `m_UserAgreement` is all-zeros. The device returns the kernel commitment and public nonce using the nonce stored in `m_Slot`.
2. Second call: `m_UserAgreement` contains the receiver's agreement hash. The device verifies the receiver's contribution, shows the spend details to the user, and if approved completes the signature. It then **regenerates** the slot value to prevent nonce reuse.

**`SignReceiver`** is single-round: the device confirms the incoming value, signs the kernel receiver contribution and the payment proof.

**`SignSplit`** is single-round: no funds leave the wallet, so no user confirmation is required beyond the fee amount.

### Trustless vs. Trusted Mode

`LocalPrivateKeyKeeper2::IsTrustless()` controls the split:

| Capability | Trusted (software wallet) | Trustless (hardware wallet) |
|---|---|---|
| `get_Kdf` returns private KDF | Yes | No (public KDF only) |
| `CreateOutput` without height limit | Yes | No (Fork1 scheme required) |
| `m_NonConventional` transactions | Yes | No |
| User confirmation for spends | No | Yes, on-device display |

---

## Class Hierarchy

```
IPrivateKeyKeeper2
├── PrivateKeyKeeper_WithMarshaller        (async event loop plumbing)
│   ├── PrivateKeyKeeper_AsyncNotify       (wraps sync methods as async completions)
│   │   └── LocalPrivateKeyKeeper2         (software key keeper — in-process KDF)
│   │       └── LocalPrivateKeyKeeperStd   (production software keeper, ~10M nonce slots)
│   ├── RemoteKeyKeeper                    (binary protocol over a transport)
│   │   └── HidKeyKeeper                   (USB/HID transport — used by Ledger)
│   └── TrezorKeyKeeperProxy               (Trezor bridge protocol)
└── ThreadedPrivateKeyKeeper               (runs any keeper on a dedicated thread)
```

---

## Local Key Keeper (`LocalPrivateKeyKeeperStd`)

The default software key keeper, used by the desktop and CLI wallets. It holds a reference to a `Key::IKdf` (the master KDF derived from the user's seed phrase) and performs all operations in the calling process.

**Nonce slots.** `LocalPrivateKeyKeeperStd` uses a `State` map of slot index → nonce preimage. The default slot count is `s_DefNumSlots = 10 * 1024 * 1024`, practically unlimited for software use.

```cpp
class LocalPrivateKeyKeeperStd : public LocalPrivateKeyKeeper2 {
    static const Slot::Type s_DefNumSlots = 10 * 1024 * 1024;
    struct State {
        ECC::Hash::Value m_hvLast;
        std::map<Slot::Type, ECC::Hash::Value> m_Used; // slot → preimage
    } m_State;
};
```

Nonces are derived deterministically from the slot's preimage combined with other parameters; after use the preimage is regenerated from a hash chain to prevent reuse.

**Thread safety.** `LocalPrivateKeyKeeperStd` is not internally thread-safe. Wrap it in `ThreadedPrivateKeyKeeper` when calling from multiple threads.

---

## Remote Key Keeper (`RemoteKeyKeeper` / `HidKeyKeeper`)

`RemoteKeyKeeper` serializes each `IPrivateKeyKeeper2` method into a packed binary request, sends it over an abstract transport (`SendRequestAsync`), and deserializes the response. The wire format uses little-endian packed structs defined in `hw_crypto/keykeeper.h`, mirroring the C structs used inside the hardware device firmware.

The protocol structs follow a request/response pair model:

```c
// Example from the hw_crypto layer (simplified):
struct SignSender_Out { uint8_t m_OpCode; TxCommonIn m_Tx; TxMutualIn m_Mut; ... };
struct SignSender_In  { TxCommonOut m_Tx; TxSig m_Sig; ...  };
```

`RemoteKeyKeeper` caches the owner public KDF and the slot count after the first successful query, serving subsequent `InvokeSync(get_Kdf)` and `InvokeSync(get_NumSlots)` calls from cache without a round-trip.

`HidKeyKeeper` provides the USB/HID transport:

- Runs a dedicated background thread (`m_Thread`) that issues blocking USB reads/writes.
- Communicates results back to the reactor thread via an `io::AsyncEvent`.
- Manages connection state (`Disconnected` / `Connected` / `Stalled`) and reports errors through the `IEvents` callback.
- Supports auto-detection: leaving `m_sPath` empty causes it to scan for a recognized device.
- Used for **Ledger** hardware wallets (Nano S, Nano S+).

```cpp
class HidKeyKeeper : public RemoteKeyKeeper {
    void SendRequestAsync(void* pBuf, uint32_t nRequest,
                          uint32_t nResponse,
                          const Handler::Ptr& pHandler) override;
    // ...
    enum struct DevState { Disconnected, Connected, Stalled };
};
```

---

## Trezor Key Keeper (`TrezorKeyKeeperProxy`)

Trezor communication uses the **Trezor bridge** (a system daemon) via the `client.hpp` / `device_manager.hpp` third-party library rather than raw HID.

```cpp
class TrezorKeyKeeperProxy : public PrivateKeyKeeper_WithMarshaller {
    std::shared_ptr<Client> m_Client;            // trezor-bridge connection
    std::shared_ptr<DeviceManager> m_DeviceManager;
    Cache m_Cache;                               // owner PKdf + child KDFs (LRU)
    HWWallet::IHandler::Ptr m_UIHandler;         // for showing "confirm on device" UI
};
```

The cache stores the owner `Key::IPKdf` and up to a configurable number of child PKdfs in an LRU set. This avoids repeated device round-trips for commitment generation during transaction building.

### `HWWallet` Discovery Layer

```cpp
class HWWallet {
    std::vector<std::string> getDevices() const;          // enumerate connected devices
    IPrivateKeyKeeper2::Ptr  getKeyKeeper(const std::string& device,
                                          IHandler::Ptr uiHandler = {});
};
```

`getDevices()` enumerates via `client.enumerate()`. `getKeyKeeper()` returns a fresh `TrezorKeyKeeperProxy` for the named device. The `IHandler` interface lets the host UI show / hide a "waiting for device confirmation" overlay.

---

## Ledger Firmware Loader (`LedgerFw`)

Beyond runtime signing, `keykeeper/ledger_loader.h` contains the `LedgerFw::Loader` used to install or update the Beam app on a Ledger device over USB. It implements the Ledger secure-channel (`EstablishSChannel`) using ECDH key exchange + AES-CBC, and handles flash zone parsing from `.hex` files.

This is a build-time / setup tool, not part of the normal signing flow.

---

## WASM Key Keeper (Browser Wallets)

`keykeeper/wasm_key_keeper.cpp` is compiled with Emscripten to produce a WASM module used by the browser-based Beam wallet. It wraps `LocalPrivateKeyKeeperStd` (the same software keeper as the desktop wallet) and exposes a JavaScript-friendly binding layer.

Key exported functions:

| JS function | Description |
|---|---|
| `GetOwnerKey(pass)` | Export the owner public KDF, encrypted with `pass` |
| `GetEndpoint([keyID])` | Return endpoint identity public key |
| `GetSbbsAddress(ownID)` | Derive SBBS address for an endpoint index |
| `GetSbbsAddressPrivate(ownID)` | Return the private SBBS key (used for BBS decryption in the browser) |
| `GetSendToken(addr, identity, amount)` | Encode a send token for address + identity |

The WASM keeper runs entirely inside the browser sandbox; the seed phrase never leaves the page. The keeper serializes transaction signing requests to JSON (via `nlohmann/json`) and returns signed results back to the JavaScript wallet engine.

---

## `ThreadedPrivateKeyKeeper`

A thin wrapper that runs any `IPrivateKeyKeeper2` implementation on a dedicated background thread:

```cpp
class ThreadedPrivateKeyKeeper : public PrivateKeyKeeper_WithMarshaller {
    IPrivateKeyKeeper2::Ptr m_pKeyKeeper; // the underlying keeper
    MyThread m_Thread;
    // queue-based dispatch: in-queue (requests), out-queue (completions)
};
```

All `InvokeAsync` calls post a typed `Task` to the in-queue; the background thread runs the underlying keeper's `InvokeSync`; the result is posted to the out-queue and signalled back via `io::AsyncEvent`. This allows blocking HID or Trezor calls to execute without stalling the wallet's async reactor.

---

## Security Properties Summary

| Property | Local | Remote/HW |
|---|---|---|
| Private keys ever leave the keeper | In-process only | Never |
| User can view balance without spending | Always | Always (owner PKdf exported freely) |
| Spending requires explicit user action | No (automated) | Yes (on-device button press) |
| Nonce reuse prevented | Hash-chain regeneration | Slot regeneration after signature |
| Host compromise → funds at risk | Yes | No (spend requires device approval) |

For the full security rationale and the interactive signing protocol design, see [HW Wallet Design](../HW-wallet-design.md).
