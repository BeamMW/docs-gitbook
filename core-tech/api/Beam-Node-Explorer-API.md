# Beam Node Explorer HTTP API

Endpoint names are declared in [`explorer/server.h`](https://github.com/BeamMW/beam/blob/master/explorer/server.h) (`ExplorerNodeDirs` macro); routing and query parameters are implemented in [`explorer/server.cpp`](https://github.com/BeamMW/beam/blob/master/explorer/server.cpp).

## How to run `explorer-node`

```
Node explorer options:
  -h [ --help ]                      list of all options
  --peer arg (=172.104.249.212:8101) peer address
  --port arg (=10000)                port to start the local node on
  --api_port arg (=8888)             port to start the local api server on
```

Example: `./explorer-node --peer eu-node01.mainnet.beam.mw:8100 --api_port=8080`  
The first sync can take a long time.

## Request URL shape

- Paths are **root-relative**: the first path segment must be one of the directory names below (e.g. `/status`, `/block`, `/contract`).
- Query string: `name=value` pairs separated by `&`.

Public hosts may use a prefix, e.g. `https://explorer.example/api/status` instead of `http://127.0.0.1:8888/status`.

## Response format modifiers (all endpoints)

Append to any URL:

| Query flag | Effect |
|------------|--------|
| `htm` | JSON is converted to HTML. |
| `exp_am` | “Explicit type” JSON (`type` / `value` cells, amounts as objects). |
| *(neither)* | Default **legacy** JSON (illustrated below for `/block`). |

---

## Endpoints

From `ExplorerNodeDirs` in `server.h`: `status`, `block`, `blocks`, `hdrs`, `peers`, `swap_offers`, `swap_totals`, `asset_swaps`, `asset_swaps_totals`, `contracts`, `contract`, `asset`, `assets`.

Examples use host `http://127.0.0.1:8888`. Responses are **illustrative**; field sets vary by chain (PoW vs PBFT), build flags (e.g. atomic / asset swap), and data volume. Large arrays are often truncated with `/* … */`.

---

### `GET /status`

**Request**

```http
GET /status HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/status"
```

**Response** `200` — JSON object (legacy):

```json
{
  "chainwork": "252,586,924,895,111",
  "hash": "4b19d7e306a2d7a3c1c5460d8b33262c68a042359b5083409e9d745ac2c7e109",
  "height": 3798950,
  "low_horizon": 0,
  "peers_count": 61,
  "shielded_outputs_per_24h": 10,
  "shielded_outputs_total": 23932,
  "shielded_possible_ready_in_hours": "78644.000000",
  "timestamp": 1775085513,
}
```

---

### `GET /block`

Single block. **`kernel`** (hex, 32-byte hash = 64 hex digits) is checked first; otherwise **`height`** and optional **`adj`**. There is **no** `hash=` query in current `server.cpp`.

#### By height

**Request**

```http
GET /block?height=20516 HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/block?height=20516"
```

**Response** `200` — legacy JSON (PoW example; `inputs` / `outputs` / `kernels` abbreviated):

```json
{
  "found": true,
  "timestamp": 1550157362,
  "height": 20516,
  "h": 20516,
  "hash": "c2a7315b63b1de6106a185c1c79219001ef5e3a07c217db227b079bbb9dd9b64",
  "prev": "4b9e35b467b416e0d307dd94bd2fdce6e720b6b3a029dca822ccab3ac57c6d22",
  "inputs": [],
  "outputs": [
    {
      "coinbase": true,
      "commitment": "0x75329071d041e7828a57cbf2f63fb8db21543f35c1c2291d5c26c20d9b11465a",
      "incubation": 0,
      "maturity": 20756
    }
  ],
  "kernels": [
    {
      "excess": "0x60413b5a09858312403190721938463ca22d7d87981a024873ddfa204a399eec",
      "fee": 0,
      "id": "d72684dba6255b2fe8631be9df764ee3c984cb0c9f386a8cf71f566acebd197d",
      "maxHeight": 18446744073709552000,
      "minHeight": 20516
    }
  ],
  "subsidy": 8000000000,
  "difficulty": 157.9972152709961,
  "chainwork": "0x384fdc718a20",
  "rate_btc": "0.00001551",
  "rate_usd": "0.231282"
}
```

#### By kernel

**Request**

```http
GET /block?kernel=d72684dba6255b2fe8631be9df764ee3c984cb0c9f386a8cf71f566acebd197d HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200` — same shape as height lookup when the kernel is found.

#### Not found

**Request**

```http
GET /block?height=999999999 HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200`:

```json
{
  "found": false,
  "height": 999999999
}
```

#### Optional `adj`

**Request**

```http
GET /block?height=20516&adj=-1 HTTP/1.1
Host: 127.0.0.1:8888
```

Used when the exact height is not on the active chain; see `Adapter::get_block` in `explorer/adapter.cpp`. Response shape matches a successful block when a nearby block is resolved.

---

### `GET /blocks`

**Request** — `height` must be **> 0**; `n` is clamped to **1…1500**.

```http
GET /blocks?height=20402&n=3 HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/blocks?height=20402&n=3"
```

**Response** `200` — JSON **array** of block objects (same family as `/block`; second and third entries abbreviated):

```json
[
  {
    "found": true,
    "timestamp": 1550150788,
    "height": 20402,
    "h": 20402,
    "hash": "2107b173f174972c8ec7543ab731bf1905d24d101f45b42d5f0cd64853e4c38e",
    "prev": "09cf95acda1e9c3e8b1a45873fd9ef1d744d6645d16bd6b8c5e9ae8dfe2d0b1a",
    "inputs": [],
    "outputs": [],
    "kernels": [],
    "subsidy": 8000000000,
    "difficulty": 161.94417613232422,
    "chainwork": "0x380b69b2d420",
    "rate_btc": "0.00001551",
    "rate_usd": "0.231282"
  },
  {
    "found": true,
    "height": 20401,
    "h": 20401,
    "timestamp": 1550150719
  },
  {
    "found": true,
    "height": 20400,
    "h": 20400,
    "timestamp": 1550150667
  }
]
```

---

### `GET /hdrs`

Columnar header / totals. Defaults for `cols` are chosen in `server.cpp` when `cols` is omitted.

**Request**

```http
GET /hdrs?hMax=20531&nMax=2&dh=1 HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/hdrs?hMax=20531&nMax=2&dh=1"
```

**Response** `200` — table wrapper; first row = column headers, following rows = data. Column names depend on `cols` or defaults (`Hash`, `Timestamp`, `Difficulty`, …). `more.hMax` appears when the server truncated rows:

```json
{
  "type": "table",
  "value": [
    [
      { "type": "th", "value": "Height" },
      { "type": "th", "value": "Hash" },
      { "type": "th", "value": "Timestamp" }
    ],
    [
      { "type": "height", "value": 20531 },
      { "type": "hash", "value": "7353b5e4ad29a2ffa5f7952749d1eb04acedd82215b1f4f01d75107165f4622b" },
      { "type": "time", "value": 1550158283 }
    ],
    [
      { "type": "height", "value": 20530 },
      { "type": "hash", "value": "…" },
      { "type": "time", "value": 1550158200 }
    ]
  ],
  "more": {
    "hMax": 20529
  }
}
```

---

### `GET /peers`

**Request**

```http
GET /peers HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/peers"
```

**Response** `200`:

```json
[
  "172.104.249.212:8101",
  "192.168.1.10:10005"
]
```

---

### `GET /swap_offers`

Only when built with atomic-swap support (`BEAM_ATOMIC_SWAP_SUPPORT`). Shape from `get_swap_offers` in `adapter.cpp`.

**Request**

```http
GET /swap_offers HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200` — JSON array:

```json
[
  {
    "status": 0,
    "status_string": "pending",
    "txId": "1b726d0adffe45c993b801c8bb46184e",
    "beam_amount": "3",
    "swap_amount": "3",
    "swap_currency": "2",
    "time_created": "2020.11.06 18:31:54",
    "min_height": 252406,
    "height_expired": 253126,
    "is_beam_side": true
  }
]
```

(`swap_currency` is an enum encoded as in your build; do not assume a three-letter ticker string.)

---

### `GET /swap_totals`

**Request**

```http
GET /swap_totals HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200`:

```json
{
  "total_swaps_count": 2,
  "beams_offered": "15",
  "bitcoin_offered": "0",
  "litecoin_offered": "10",
  "qtum_offered": "0",
  "dogecoin_offered": "0",
  "dash_offered": "0",
  "ethereum_offered": "0",
  "dai_offered": "0",
  "usdt_offered": "0",
  "wbtc_offered": "0"
}
```

---

### `GET /asset_swaps`

Only when built with asset-swap support (`BEAM_ASSET_SWAP_SUPPORT`). Live asset-swap offers, which the node receives over SBBS broadcast. Shape from `get_asset_swaps` in `adapter.cpp`.

Offers are reported in **maker terms**: `send_*` is what the maker offers, `receive_*` is what the maker wants. Amounts are formatted decimal strings (`PrintableAmount`, may include thousands separators); `send_asset_id` / `receive_asset_id` are numeric asset IDs (`0` = BEAM). `create_time` / `expire_time` are raw unix seconds (formatted client-side). Expired offers are omitted.

**Request**

```http
GET /asset_swaps HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/asset_swaps"
```

**Response** `200` — JSON array:

```json
[
  {
    "id": "5f8d3c2b1a094e6f7d8c9b0a1e2f3d4c",
    "send_asset_id": 0,
    "send_currency": "BEAM",
    "send_amount": "4,862.63068319",
    "receive_asset_id": 7,
    "receive_currency": "USDT",
    "receive_amount": "500",
    "create_time": 1781784005,
    "expire_time": 1781785805
  }
]
```

(`id` is the 32-hex-char order UUID. Empty array `[]` when there are no live offers.)

---

### `GET /asset_swaps_totals`

Only when built with asset-swap support (`BEAM_ASSET_SWAP_SUPPORT`). Aggregate of the live offers, grouped per asset, plus the total offer count. Shape from `get_asset_swaps_totals` in `adapter.cpp`.

`total_offers_count` is the number of non-expired offers. Each `assets` entry sums, per asset ID, `amount_offered` (the asset on makers' send side) and `amount_wanted` (the asset on makers' receive side); both are formatted decimal strings.

**Request**

```http
GET /asset_swaps_totals HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/asset_swaps_totals"
```

**Response** `200`:

```json
{
  "total_offers_count": 12,
  "assets": [
    { "asset_id": 0, "currency": "BEAM", "amount_offered": "5,000", "amount_wanted": "0" },
    { "asset_id": 7, "currency": "USDT", "amount_offered": "0", "amount_wanted": "2,500" }
  ]
}
```

---

### `GET /contracts`

**Request**

```http
GET /contracts HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200` — table of deployed contracts; top-level `h` is current chain height (`add_current_height`):

```json
{
  "type": "table",
  "value": [
    [
      { "type": "th", "value": "Cid" },
      { "type": "th", "value": "Kind" },
      { "type": "th", "value": "Deploy Height" },
      { "type": "th", "value": "Locked Funds" },
      { "type": "th", "value": "Owned Assets" }
    ],
    [
      { "type": "cid", "value": "0066b12078623df132b691001b25d7eb94b207b42c018020c9e58152e21ecd25" },
      { "type": "string", "value": "DaoVault v0" },
      3798000,
      "",
      ""
    ]
  ],
  "h": 3799000
}
```

---

### `GET /contract`

**Request** — `id` is **required** (64 hex chars for a 32-byte contract ID). Optional: `hMin`, `hMax`, `nMaxTxs`, `state`, `assets_owned`, `funds_locked`, `ver_info` (see `server.cpp`).

```http
GET /contract?id=0066b12078623df132b691001b25d7eb94b207b42c018020c9e58152e21ecd25&hMax=3676621 HTTP/1.1
Host: 127.0.0.1:8888
```

```bash
curl -sS "http://127.0.0.1:8888/contract?id=0066b12078623df132b691001b25d7eb94b207b42c018020c9e58152e21ecd25&hMax=3676621"
```

**Response** `200` — large object with multiple sections; **`Calls history`** is what tools such as beammm parse for contract calls (structure simplified):

```json
{
  "Calls history": {
    "type": "table",
    "more": { "hMax": 3676621 },
    "value": [
      [
        { "type": "th", "value": "Height" },
        { "type": "th", "value": "Cid" },
        { "type": "th", "value": "Kind" },
        { "type": "th", "value": "Method" },
        { "type": "th", "value": "Arguments" },
        { "type": "th", "value": "Funds" },
        { "type": "th", "value": "Keys" }
      ],
      {
        "type": "group",
        "value": [
          [
            3798790,
            "",
            "",
            "Trade",
            {
              "Aid1": { "type": "aid", "value": 0 },
              "Aid2": { "type": "aid", "value": 47 },
              "Volatility": "High"
            },
            {
              "type": "table",
              "value": [
                [
                  { "type": "aid", "value": 0 },
                  { "type": "amount", "value": "-15181633533" }
                ],
                [
                  { "type": "aid", "value": 47 },
                  { "type": "amount", "value": "+245918323" }
                ]
              ]
            },
            "",
            ""
          ]
        ]
      }
    ]
  },
  "h": 3799000
}
```

**Error** — missing `id` → `500` with body like `Internal error: id missing`.

---

### `GET /asset`

**Request** — non-zero `id` required (`adapter.cpp` throws if `!aid`).

```http
GET /asset?id=47&hMin=0&hMax=-1&nMaxOps=100 HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200` — object with at least **`Asset history`** and **`Asset distribution`** (table shapes; rows vary):

```json
{
  "Asset history": {
    "type": "table",
    "value": [
      [
        { "type": "th", "value": "Height" },
        { "type": "th", "value": "Event" },
        { "type": "th", "value": "Amount" },
        { "type": "th", "value": "Total Amount" },
        { "type": "th", "value": "Extra" }
      ],
      [
        100000,
        "Mint",
        { "type": "amount", "value": "+1000000000" },
        { "type": "amount", "value": "1000000000" },
        ""
      ]
    ]
  },
  "Asset distribution": {
    "type": "table",
    "value": []
  }
}
```

---

### `GET /assets`

**Request** — optional `height` (default: chain tip in adapter).

```http
GET /assets?height=20531 HTTP/1.1
Host: 127.0.0.1:8888
```

**Response** `200` — table of assets at that height; includes `h` when querying current tip:

```json
{
  "type": "table",
  "value": [
    [
      { "type": "th", "value": "Aid" },
      { "type": "th", "value": "Owner" },
      { "type": "th", "value": "Deposit" },
      { "type": "th", "value": "Supply" },
      { "type": "th", "value": "Lock height" },
      { "type": "th", "value": "Metadata" }
    ],
    [
      { "type": "aid", "value": 0 },
      "",
      { "type": "amount", "value": "0" },
      { "type": "amount", "value": "110000000000" },
      0,
      ""
    ],
    [
      { "type": "aid", "value": 47 },
      { "type": "pubkey", "value": "…" },
      { "type": "amount", "value": "1000000" },
      { "type": "amount", "value": "1000000000000" },
      0,
      "USDC"
    ]
  ],
  "h": 20531
}
```

---

## Errors and access control

**403** — IP not whitelisted (if configured): plain text body, e.g. `Forbidden`.

**404** — path does not match a known directory: `Not Found`.

**500** — handler exception, e.g. `Internal error: id missing` for `/contract` without `id`.

## CORS

Successful responses include `Access-Control-Allow-Origin: *` and `Access-Control-Allow-Headers: *` (`Server::send` in `server.cpp`).
