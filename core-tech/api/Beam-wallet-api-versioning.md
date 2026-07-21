# Beam Wallet API Versioning

## Why versioned subfolders?

Each subfolder (`v6_0/`, `v6_1/`, `v7_0/`, …) holds the methods introduced in that API version. All versions live in the same binary — a client connecting to the `wallet-api` server requests a specific version (e.g. `--api-version 7.3`) and gets back an object that only knows about methods up to that version. Keeping each version in its own folder makes it easy to see exactly when a method was introduced or changed, and prevents older-version files from growing endlessly as new methods accumulate.

---

## How the versioning system works

### Inheritance chain

The API classes form a strict linear hierarchy:

```
ApiBase  (base/api_base.h)
  └── V6Api          (v6_0/)
        └── V61Api   (v6_1/)
              └── V70Api   (v7_0/)
                    └── V71Api   (v7_1/)
                          └── V72Api   (v7_2/)
                                └── V73Api   (v7_3/)
                                      └── V74Api   (v7_4/)
```

Each child constructor calls its parent constructor first, then registers its own methods:

```cpp
V74Api::V74Api(IWalletApiHandler& handler, ...)
    : V73Api(handler, avMajor, avMinor, init)   // parent registers v6.0–v7.3 methods
{
    V7_4_API_METHODS(BEAM_API_REG_METHOD)        // add/overwrite v7.4 methods
}
```

### Method registry

`ApiBase` holds a `std::unordered_map<std::string, Method> _methods`. The `BEAM_API_REG_METHOD` macro calls `regMethod(name, method)`, which is simply:

```cpp
void regMethod(const std::string& name, Method method)
{
    _methods[name] = std::move(method);   // map assignment — last write wins
}
```

When a JSON-RPC request arrives, the dispatcher looks up `_methods[req.method]` and runs it. If the key is not present the client receives a `-32601 Procedure not found` error.

### Version selection (server side)

The server instantiates exactly one API version object for the lifetime of a connection. There is no per-request version field. Version is chosen at server start:

```bash
wallet-api --api-version 7.3   # creates a V73Api — only knows methods up to v7.3
wallet-api                     # uses ApiVerCurrent (= 7.4 today)
```

`IWalletApi::CreateInstance()` in `i_wallet_api.cpp` maps the requested version string/number to the appropriate class and returns it. `ApiVerMin` (currently `6.0`) and `ApiVerMax` (currently `7.4`) are defined in `i_wallet_api.h`.

Clients can discover the running version by calling the `get_version` method (available since v6.1).

---

## The three files every version needs

| File | Purpose |
|---|---|
| `vX_Y_api_defs.h` | Struct definitions for request/response; the `VX_Y_API_METHODS(macro)` macro |
| `vX_Y_api.h` / `vX_Y_api.cpp` | Class declaration + constructor body that calls `VX_Y_API_METHODS(BEAM_API_REG_METHOD)` |
| `vX_Y_api_parse.cpp` | `onParseXxx` — JSON → struct; `getResponse` — struct → JSON |
| `vX_Y_api_handle.cpp` | `onHandleXxx` — business logic, calls `doResponse` |

All of these are compiled into the single `wallet_api` static library (see `CMakeLists.txt`). When adding a new version, add its four `.cpp` files to the `SOURCES` list in `CMakeLists.txt` and update `ApiVersions` + `ApiVerCurrent` in `i_wallet_api.h`.

---

## How to modify an existing endpoint

The right approach depends on whether the change is backward-compatible.

### Case 1 — Backward-compatible change (edit in place)

**When:** Adding optional parameters, loosening validation, changing internal logic without altering the wire format.

**Do:** Edit the struct and parse/handle files in the version where the method was originally defined.

**Example (what we just did):** `sign_message` lived in `v7_0/`. Its original signature required `key_material`. We added an optional `address` field and made `key_material` optional too. Old clients that still send `key_material` continue to work; new clients can send `address` instead.

```
Edit directly:
  wallet/api/v7_0/v7_0_api_defs.h    ← update struct fields
  wallet/api/v7_0/v7_0_api_parse.cpp ← handle the new optional param
  wallet/api/v7_0/v7_0_api_handle.cpp ← branch on which field is present
```

No new folder, no new version class needed. Every version from 7.0 upward inherits the improved method transparently.

### Case 2 — Breaking change, same method name (override in a new version)

**When:** Removing or renaming a required parameter, restructuring the response, or changing semantics in a way that would break existing callers.

**Do:** Leave the original version's files untouched. Create a new version (e.g. `v7_5/`) and re-define the method there under the same JSON name but with a new C++ struct name.

Because the child constructor runs after the parent, `BEAM_API_REG_METHOD` in `V75Api` will call `regMethod("method_name", …)` which overwrites the entry in `_methods` left by the parent. Clients on v7.0–v7.4 keep getting the old behaviour because they never instantiate `V75Api`.

```cpp
// v7_5_api_defs.h
#define V7_5_API_METHODS(macro) \
    macro(SignMessageV75, "sign_message", API_READ_ACCESS, API_SYNC, APPS_ALLOWED)

struct SignMessageV75 {
    // completely redesigned — no keyMaterial at all
    std::string address;
    std::string message;
    struct Response { std::string signature; std::string pubkey; };
};
```

```
Create new files:
  wallet/api/v7_5/v7_5_api_defs.h
  wallet/api/v7_5/v7_5_api.h          ← class V75Api : public V74Api
  wallet/api/v7_5/v7_5_api.cpp        ← V7_5_API_METHODS(BEAM_API_REG_METHOD)
  wallet/api/v7_5/v7_5_api_parse.cpp
  wallet/api/v7_5/v7_5_api_handle.cpp
```

This is exactly the same mechanism `v6_1/` already uses: both `invoke_contract` and `wallet_status` were first defined in `v6_0/` then overridden in `v6_1/` with new structs (`InvokeContractV61`, `WalletStatusV61`) but the same JSON method names.

### Case 3 — New method name

**When:** Entirely new functionality, or you want both old and new endpoints to coexist.

**Do:** Add the new struct and macro entry to the version where it should first appear (`v7_5/` if it's a v7.5 addition). Nothing special — it simply won't exist in `_methods` for any version below that.

---

## Working with a minimum supported version

The "minimum API version" refers to the oldest version the server is *willing to serve* — controlled by `ApiVerMin` in `i_wallet_api.h` and enforced in `IWalletApi::CreateInstance()`. It has nothing to do with where methods live in the file tree.

**Scenario:** The project currently ships `--api-version 7.5` by default but must keep accepting `--api-version 7.3` connections from older clients.

- Methods added in `v7_4/` or `v7_5/` are simply absent when a client connects on 7.3 — they get `-32601` if they try to call them.
- Methods that exist in `v7_0/` (like `sign_message`) are inherited all the way up, so all versions from 7.0 onward get them.
- A backward-compatible edit to an older file (Case 1) benefits every version simultaneously.
- A breaking override (Case 2) only applies to v7.5+ connections; 7.3 clients still hit the old code path.

**Rule of thumb:**
- If you can keep old callers working → edit the original file.
- If you must break the wire contract → create a new version folder and override there.
- Never edit a version file to deliberately break callers on that version; old version files are append-only once released.

---

## Quick reference: adding a new version (v7.5 example)

1. Create `wallet/api/v7_5/` with the four standard files.
2. In `v7_5_api.h`: `class V75Api : public V74Api { … V7_5_API_METHODS(…) };`
3. In `v7_5_api.cpp`: constructor calls parent then `V7_5_API_METHODS(BEAM_API_REG_METHOD)`.
4. In `i_wallet_api.h`: add `macro(7, 5)` to `ApiVersions` and update `ApiVerCurrent = ApiVer7_5`.
5. In `i_wallet_api.cpp`: add a `case ApiVer7_5: return std::make_unique<V75Api>(…)` branch.
6. In `CMakeLists.txt`: append `v7_5/v7_5_api.cpp`, `v7_5/v7_5_api_handle.cpp`, `v7_5/v7_5_api_parse.cpp` to `SOURCES`.
