# BEAM IPFS Support

Beam embeds a full [IPFS](https://ipfs.tech/) (InterPlanetary File System) node into the wallet process, enabling decentralized content-addressed storage for DApp user interfaces and other on-chain assets.

## Why IPFS Is Embedded

Beam DApps are implemented as [shader pairs](bvm/BVM-Shader-Development.md): a contract shader running inside the BVM and an app shader that runs in the wallet and builds the UI. The UI assets — HTML, JavaScript, images — must be delivered to the wallet client without relying on a centralized server. IPFS provides content-addressed immutable storage, so a CID embedded in the shader's metadata permanently identifies the exact UI bundle. When a user opens a DApp the wallet fetches the bundle by CID from the IPFS network.

Secondary use cases include NFT metadata storage and application-level content pinning from DApp shaders via the wallet API.

## Client Modes

| Client type | IPFS node | Write operations | Read operations |
|---|---|---|---|
| Desktop wallet | Full local node (optional, starts on DApp launch by default) | `ipfs_add`, `ipfs_pin`, `ipfs_unpin`, `ipfs_gc` | `ipfs_get`, `ipfs_hash` |
| `wallet-api` CLI | Full local node (opt-in: `--enable_ipfs=true`) | All methods | All methods |
| Mobile / WASM clients | No local node | Not available (fail) | Via HTTP to BEAM-managed IPFS nodes |

The compile-time guard `BEAM_IPFS_SUPPORT` controls whether any of this code is included; builds without it expose none of the IPFS symbols.

## Architecture

### Threading Model

`IPFSService` (`wallet/ipfs/ipfs.h`) is the public interface. The underlying `asio_ipfs::node` runs on a dedicated **service thread** with its own `boost::asio::io_context`. All IPFS operations are non-blocking from the caller's perspective.

```cpp
struct IPFSService {
    struct Handler {
        // Called from the IPFS service thread — must dispatch back to the
        // wallet's reactor thread, not execute work inline.
        virtual void AnyThread_pushToClient(std::function<void()>&&) = 0;
        virtual void AnyThread_onStatus(const std::string& error, uint32_t peercnt) = 0;
    };

    static Ptr AnyThread_create(HandlerPtr);

    // Calling thread becomes the service thread for this call
    virtual void ServiceThread_start(asio_ipfs::config) = 0;
    virtual void ServiceThread_stop() = 0;

    virtual bool   AnyThread_running() const = 0;
    virtual std::string AnyThread_id() const = 0;

    virtual void AnyThread_add   (data, bool pin, timeout, res_cb, err_cb) = 0;
    virtual void AnyThread_hash  (data, timeout, res_cb, err_cb) = 0;
    virtual void AnyThread_get   (hash, timeout, res_cb, err_cb) = 0;
    virtual void AnyThread_pin   (hash, timeout, res_cb, err_cb) = 0;
    virtual void AnyThread_unpin (hash, res_cb, err_cb) = 0;
    virtual void AnyThread_gc    (timeout, res_cb, err_cb) = 0;
};
```

Every method name prefixed `AnyThread_` is safe to call from any thread. Callbacks always arrive in the wallet's reactor thread via `Handler::AnyThread_pushToClient`, which posts through `PostToReactorThread` — an `io::AsyncEvent`-based cross-thread queue (`wallet/ipfs/ipfs_async.h`).

### Startup Sequence

`ServiceThread_start` runs the `asio_ipfs::node::build` coroutine **synchronously** in the calling thread until the node is fully initialized (bootstrap connection established, swarm key accepted, peer ID derived). Only then does the dedicated service thread begin its event loop. This simplifies the startup flow at the cost of a potentially multi-second blocking call on first launch.

## Configuration (`asio_ipfs::config`)

| Field | Desktop default | Server (`wallet-api`) default | CLI flag | `settings.ini` key |
|---|---|---|---|---|
| `repo_root` | `<wallet-data>/ipfs-repo` | same | `--ipfs_repo` | `[ipfsnode] ipfs_repo` |
| `storage_max` | `2GB` | `20GB` | `--ipfs_storage_max` | `[ipfsnode] ipfs_storage_max` |
| `low_water` | `100` | `100` | `--ipfs_low_water` | `[ipfsnode] ipfs_low_water` |
| `high_water` | `200` | `200` | `--ipfs_high_water` | `[ipfsnode] ipfs_high_water` |
| `grace_period` | `20s` | `20s` | `--ipfs_grace_period` | `[ipfsnode] ipfs_grace_period` |
| `swarm_port` | `10100` | `10100` | `--ipfs_swarm_port` | `[ipfsnode] ipfs_swarm_port` |
| `api_address` | *(disabled)* | `/ip4/127.0.0.1/tcp/6100` | `--ipfs_api_addr` | `[ipfsnode] ipfs_api_addr` |
| `gateway_address` | *(disabled)* | `/ip4/127.0.0.1/tcp/6200` | `--ipfs_gateway_addr` | `[ipfsnode] ipfs_gateway_addr` |
| `routing_type` | `dht` | `dhtserver` | `--ipfs_routing_type` | `[ipfsnode] ipfs_routing_type` |
| `auto_relay` | `true` | `false` | `--ipfs_auto_relay` | `[ipfsnode] ipfs_auto_relay` |
| `relay_hop` | `false` | `false` | `--ipfs_relay_hop` | `[ipfsnode] ipfs_relay_hop` |
| `autonat` | `true` | `true` | `--ipfs_autonat` | `[ipfsnode] ipfs_autonat` |
| `autonat_limit` | `30` | `30` | `--ipfs_autonat_limit` | `[ipfsnode] ipfs_autonat_limit` |
| `autonat_peer_limit` | `3` | `3` | `--ipfs_autonat_peer_limit` | `[ipfsnode] ipfs_autonat_peer_limit` |
| `run_gc` | `true` | `false` | `--ipfs_run_gc` | `[ipfsnode] ipfs_run_gc` |
| `bootstrap` | network defaults | network defaults | `--ipfs_bootstrap` (space-separated multiaddr list) | `[ipfsnode] ipfs_bootstrap` |
| `peering` | network defaults | network defaults | — | — |
| `swarm_key` | network default | network default | `--ipfs_swarm_key` | `[ipfsnode] ipfs_swarm_key` |

### Network-Specific Bootstrap and Swarm Keys

Beam operates a **private IPFS swarm** — nodes with a wrong or missing swarm key cannot join. Bootstrap nodes and swarm keys are injected automatically based on `Rules::get().m_Network`:

| Network | Bootstrap peers | Swarm key prefix |
|---|---|---|
| `mainnet` | `eu-node01..04.mainnet.beam.mw:38041` | `1fabcf9e…` |
| `testnet` | `eu-node01..03.testnet.beam.mw:38041` | `1191aea7…` |
| `masternet` | `3.19.32.148:38041` | `18502580…` |
| `dappnet` | `3.16.160.95:38041` | `bf2f2063…` |

All swarm keys use the PSK v1 format: `/key/swarm/psk/1.0.0/\n/base16/\n<hex>`.

If a custom `swarm_key` or `bootstrap` list is provided in config, the network defaults are ignored entirely for that field.

### `config.lock` — Preventing Config Overwrite

On every startup Beam forcibly overwrites the BEAM-specific settings in `<repo>/config`. To suppress this (e.g., to preserve manual edits), create the file `<repo>/config.lock`. When this file is present, all BEAM-side config knobs — CLI flags, `wallet_api.cfg`, desktop settings — are ignored and the existing `config` file is used as-is.

## Async API

All six service methods follow the same pattern: submit a coroutine to the IPFS `io_context`, optionally attach a `boost::asio::steady_timer` for timeout enforcement, then marshal the result back to the wallet's reactor thread via `Handler::AnyThread_pushToClient`.

| Method | Direction | Notes |
|---|---|---|
| `AnyThread_add(data, pin, timeout, res, err)` | local → IPFS network | Stores bytes; returns CID string. `pin=true` prevents GC from evicting the block. |
| `AnyThread_hash(data, timeout, res, err)` | local only | Computes CID without storing (`calc_cid` internally). No network I/O. |
| `AnyThread_get(hash, timeout, res, err)` | IPFS network → local | Fetches content by CID; returns raw bytes. |
| `AnyThread_pin(hash, timeout, res, err)` | local bookkeeping | Marks a CID as pinned so GC will not remove it. |
| `AnyThread_unpin(hash, res, err)` | local bookkeeping | Removes pin; timeout not applicable (local). |
| `AnyThread_gc(timeout, res, err)` | local | Removes all un-pinned blocks from the local store. |

Timeout is in milliseconds; pass `0` to disable timeout enforcement (unpin always uses `0`).

### Wallet API Methods

These operations are also exposed as JSON-RPC methods in the wallet API (since v6.3), all tagged `APPS_ALLOWED` so DApp shaders can call them:

| JSON-RPC method | Access level | Async |
|---|---|---|
| `ipfs_add` | write | yes |
| `ipfs_hash` | read | yes |
| `ipfs_get` | write | yes |
| `ipfs_pin` | write | yes |
| `ipfs_unpin` | write | yes |
| `ipfs_gc` | write | yes |

See the [Wallet API v7.0](api/Beam-wallet-protocol-API-v7.0.md) documentation for parameter schemas.

## Interaction with Shader Invocation

When a DApp is opened in the desktop wallet, the wallet:

1. Reads the shader's embedded metadata to find the app shader's IPFS CID.
2. Calls `AnyThread_get(cid, ...)` to fetch the UI bundle from IPFS.
3. Starts a local IPFS node on demand (if not already running) via `IWThread_startIPFSNode`.
4. Launches a sandboxed WebView pointing at the fetched bundle.

The app shader running inside the WebView can call `ipfs_add`, `ipfs_get`, etc., through the DApp API (`wallet/client/apps_api/apps_api.h`). The DApp API checks whether an IPFS node is available (`hasIPFSNode`) at DApp launch time and logs a warning if it is not; read operations from mobile/WASM clients fall back to HTTP calls to BEAM-managed gateway nodes.

## Repository Layout

The IPFS repo lives at `<wallet-data>/ipfs-repo` by default and is a standard go-ipfs repository. The standard IPFS CLI can inspect it:

```bash
export IPFS_PATH=/path/to/beam/ipfs-repo
export LIBP2P_FORCE_PNET=1   # required for private swarm
ipfs swarm peers
ipfs pin ls
```

Ensure the Beam wallet is not running when using the IPFS CLI against the same repo — both processes cannot hold the repo lock simultaneously.

**Desktop note:** The IPFS API (`Addresses.API`) is disabled in the desktop client's default repo config. Set `ipfs_node_api_port` in `settings.ini` or add `Addresses.API` manually to `ipfs-repo/config` before starting, otherwise `ipfs daemon` will abort.

### SystemD Unit (wallet-api)

```ini
[Unit]
Description=Beam Wallet API with IPFS
After=network.target

[Service]
Type=exec
Restart=on-failure
WorkingDirectory=/home/beam/wallet-api
ExecStart=/home/beam/wallet-api/wallet-api --enable_ipfs=true ...

[Install]
WantedBy=multi-user.target
```

## Current Limitations

- **WebUI** is disabled and not planned.
- **FUSE mounts** (`/ipfs`, `/ipns`) are disabled and not planned.
- **Remote MFS-root pinning** is not supported.
- **Sockets-based activation** for `io.ipfs.api` / `io.ipfs.gateway` is not supported.
- The IPFS startup is **synchronous** (blocks the calling thread until the node is connected); async startup is a noted TODO in the implementation.
- Connect timeout lowering is also listed as a TODO; the initial peer connection may take several seconds on a cold start.
