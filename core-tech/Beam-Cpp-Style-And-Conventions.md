# Beam C++ Style and Conventions

This page documents the C++ idioms, patterns, and conventions used throughout the Beam codebase. Its audience is contributors adding new code to `core/`, `node/`, `wallet/`, `bvm/`, or `utility/`.

---

## C++ Standard and Compiler Support

The baseline standard is **C++17** (`CMAKE_CXX_STANDARD 17`), enforced as required. An optional C++20 mode exists via the CMake flag `-DBEAM_CPP_20_STANDARD=ON`, but the vast majority of the codebase targets C++17. Avoid C++20-only features in new code unless explicitly building under that flag.

Supported compilers: GCC, Clang, and MSVC. Platform guards use `#ifdef WIN32`, `#if defined(__clang__)`, `#if defined(__GNUC__)`.

---

## Naming Conventions

### Types and Classes

All types use **PascalCase**: `NodeProcessor`, `TxKernel`, `WalletDB`, `HeightRange`, `Serializer`.

Nested types follow the same rule: `NodeProcessor::Mapped::Utxo`, `Reactor::Object`, `Exc::Checkpoint`.

### Data Members

All data members carry an `m_` prefix:

```cpp
Height m_Height = 0;
uint32_t m_Pos = 0;
ECC::Hash::Value m_Hash;
```

Pointer members additionally use a `p` suffix on the prefix: `m_pData`, `m_pExternal`, `m_pNext`. This signals non-owning raw pointer semantics.

Static data members use `s_`: `s_pInstance`, `s_pTop`.

### Free Functions and Methods

Both free functions and member functions use **PascalCase**: `HeightAdd`, `ZeroObject`, `GetTime_ms`, `ExportNnz`, `FromPubKey`. Utility helpers outside the `beam` namespace (in `utility/helpers.h`) may use `lower_snake_case` (`format_timestamp`, `local_timestamp_msec`).

### Macros

All macros use `UPPER_SNAKE_CASE` with the `BEAM_` prefix for public-API macros: `BEAM_LOG_ERROR`, `BEAM_VERIFY`, `BEAM_LOG_INFO`. Internal codegen macros omit the prefix but still use upper snake case: `COMPARISON_VIA_CMP`, `IMPLEMENT_GET_PARENT_OBJ`, `SERIALIZE`, `BIND_THIS_MEMFN`.

---

## Memory Ownership

### The `Ptr` typedef

Every class that participates in shared lifetime declares a `Ptr` member type alias:

```cpp
// Shared ownership — services, long-lived objects
using Ptr = std::shared_ptr<Wallet>;          // wallet/core/wallet.h
using Ptr = std::shared_ptr<IWalletDB>;       // wallet/core/wallet_db.h
using Ptr = std::shared_ptr<Reactor>;         // utility/io/reactor.h

// Exclusive ownership — data objects with clear single owner
typedef std::unique_ptr<Input> Ptr;           // core/block_crypt.h
typedef std::unique_ptr<TxKernel> Ptr;
using Ptr = std::unique_ptr<Timer>;           // utility/io/timer.h
```

Callers use `ClassName::Ptr` rather than spelling out the smart pointer type, so ownership semantics are expressed once at the declaration site.

### Guidelines

| Scenario | Type |
|---|---|
| Object with a single, clear owner (kernel, input, output) | `std::unique_ptr<T>` |
| Object shared between subsystems (wallet, reactor, DB) | `std::shared_ptr<T>` |
| Non-owning reference / observer | raw pointer (`T*`) or raw reference (`T&`) |
| Optional pointer, not owned | raw pointer initialized to `nullptr` |

`beam::Ptr<>` does not appear as a standalone alias in the codebase; ownership is expressed through the per-class `Ptr` typedef described above.

---

## Error Handling

### `beam::Exc` — Recoverable Exceptions

The primary exception type is `beam::Exc`, derived from `std::runtime_error`:

```cpp
struct Exc : public std::runtime_error {
    uint32_t m_Type;

    static void Fail();                // throws with empty message
    static void Fail(const char*);     // throws with message
    static void Test(bool b) { if (!b) Fail(); }
};
```

`Exc::Test(cond)` is the preferred assertion-to-exception idiom: it reads like a guard clause and avoids `if (!cond) throw`.

### `Exc::Checkpoint` — Contextual Breadcrumbs

`Exc::Checkpoint` is a thread-local linked list of context objects that are dumped when an exception is thrown. Concrete implementations provide a `Dump(std::ostream&)` override. `CheckpointTxt` is the lightweight string variant:

```cpp
Exc::CheckpointTxt cp("verifying kernel");
// ... code that may throw ...
// On exception, "verifying kernel" appears in the error output
```

The logger integrates with checkpoints via `BEAM_LOG_MESSAGE << FlushAllCheckpoints{}`.

### `beam::CorruptionException` — Non-Recoverable Errors

`CorruptionException` is intentionally not derived from `std::exception`. It signals database or block corruption that cannot be recovered in-process and should not be caught at intermediate call sites. It propagates to the top-level handler, which shuts down the node cleanly.

### Network Errors: `proto::NodeConnection::OnDisconnect`

Network layer errors do not throw; they invoke the virtual callback:

```cpp
virtual void OnDisconnect(const DisconnectReason&) {}
```

Subclasses override `OnDisconnect` to clean up connection state. The `DisconnectReason` carries an error code and a human-readable description. This pattern avoids exception propagation across async boundaries.

---

## Serialization

### YAS Binary Archive

Beam uses the **YAS** (Yet Another Serialization) library for binary serialization, configured as:

```cpp
constexpr int SERIALIZE_OPTIONS =
    yas::binary | yas::no_header | yas::elittle | yas::compacted;
```

- `binary` — raw binary format, not JSON or text
- `no_header` — no version header in the byte stream
- `elittle` — explicit little-endian for all integers
- `compacted` — variable-length encoding for integers

### The `SERIALIZE(...)` Macro

Structs declare serialization inline using the `SERIALIZE` macro from `utility/serialize_fwd.h`:

```cpp
struct HeightPos {
    Height m_Height = 0;
    uint32_t m_Pos = 0;

    template<typename Archive> void serialize(Archive& ar) const {
        ar & m_Height & m_Pos;
    }
    template<typename Archive> void serialize(Archive& ar) {
        ar & m_Height & m_Pos;
    }
};
// Equivalent — generated by:
SERIALIZE(m_Height, m_Pos)
```

The macro generates both const and non-const overloads. Use the macro for new structs; spell it out only when custom logic is needed inside `serialize`.

### `Serializer` and `Deserializer`

```cpp
// Write
beam::Serializer ser;
ser & myObject;
auto [ptr, size] = ser.buffer();

// Read
beam::Deserializer deser;
deser.reset(ptr, size);
deser & myObject;
```

For fixed-size on-stack buffers use `StaticBufferSerializer<N>` (default 100 KB). For size estimation without storing data, use `SerializerSizeCounter`.

---

## IO and Reactor Model

### `io::Reactor`

All async I/O runs through `io::Reactor`, a thin wrapper around **libuv**. A single reactor per thread owns the event loop:

```cpp
auto reactor = io::Reactor::create();   // returns Reactor::Ptr
{
    io::Reactor::Scope scope(*reactor); // registers as thread-local current reactor
    reactor->run();                     // blocks until reactor->stop() is called
}
```

`Reactor::Scope` is an RAII guard that sets the thread-local current reactor. `Reactor::get_Current()` retrieves it from anywhere on that thread.

### Timers

```cpp
io::Timer::Ptr timer = io::Timer::create(*reactor);
timer->start(intervalMs, /*isPeriodic=*/true, [&]() {
    // callback fires every intervalMs
});
```

`Timer::Ptr` is `std::unique_ptr<Timer>`. Destroying the `Ptr` cancels the timer. The callback is stored as `std::function<void()>`.

### TCP Connections

TCP connect/accept use `std::function` callbacks passed at connection time:

```cpp
reactor->tcp_connect(address, tag,
    [](uint64_t tag, std::unique_ptr<TcpStream>&& stream, io::ErrorCode ec) {
        if (ec != 0) { /* handle error */ return; }
        // stream is ready
    },
    timeoutMs
);
```

All network objects (`TcpStream`, `TcpServer`, `SslStream`) inherit from `Reactor::Object`, which owns a `uv_handle_t*` and a `Reactor::Ptr` back-reference.

### `GracefulIntHandler`

`Reactor::GracefulIntHandler` catches `SIGINT`/`SIGTERM` (or `Ctrl+C` on Windows) and calls `reactor->stop()`. Instantiate one per process to get clean shutdown behaviour:

```cpp
io::Reactor::GracefulIntHandler intHandler(*reactor);
reactor->run();
```

---

## RAII Scope Guards

The `Scope` pattern recurs throughout Beam to set thread-local or global context for the lifetime of a block:

| Class | What it sets |
|---|---|
| `io::Reactor::Scope` | Current reactor on this thread |
| `Executor::Scope` | Current parallel executor on this thread |
| `Rules::Scope` | Consensus parameters (used in tests to override fork heights) |
| `NodeDB::Transaction` | Database transaction |

All follow the same pattern: constructor records the previous value, destructor restores it.

---

## Common Macros

### `IMPLEMENT_GET_PARENT_OBJ`

Used when an inner struct needs access to its enclosing object without storing an explicit back-pointer. It computes the parent pointer arithmetically from `this` using offsetof-style arithmetic:

```cpp
struct DB : public NodeDB {
    void OnModified() override { get_ParentObj().OnModified(); }
    IMPLEMENT_GET_PARENT_OBJ(NodeProcessor, m_DB)
} m_DB;
```

The generated `get_ParentObj()` returns a reference to the enclosing `NodeProcessor`. An `assert` in the macro validates the pointer computation in debug builds.

### `COMPARISON_VIA_CMP`

Generates all six comparison operators from a single `int cmp(const T&) const` method:

```cpp
struct Blob {
    int cmp(const Blob&) const;
    COMPARISON_VIA_CMP
};
```

### `BIND_THIS_MEMFN(M)`

Wraps a member function pointer into a `std::function` lambda bound to `this`:

```cpp
reactor->tcp_connect(addr, tag, BIND_THIS_MEMFN(OnConnected), timeout);
// equivalent to:
// std::bind_memfn(this, &ClassName::OnConnected)
```

### `BEAM_VERIFY(x)`

Like `assert`, but preserves the expression in release builds as a void cast (side-effect safe, no check). Use for expressions that must be evaluated in release but whose failure is non-fatal in production.

---

## X-Macro Pattern for Message Types

The node-to-node protocol (`core/proto.h`) uses **X-macros** to declare all message types once and expand them into struct definitions, handler declarations, and dispatch tables:

```cpp
// Declaration: each message lists its fields
#define BeamNodeMsg_NewTip(macro) \
    macro(Block::SystemState::Full, Description)

// Expansion: generates struct NewTip { Block::SystemState::Full Description; };
// and virtual void OnMsg(NewTip&&) {} in NodeConnection
```

The same macro set is used to generate serialization code and the handler virtual methods. Adding a new message type requires only a new `BeamNodeMsg_Foo(macro)` definition and adding `Foo` to the `BeamNodeMsgsAll` list.

---

## Logging

All logging goes through the `beam::Logger` singleton, accessed via macros:

```cpp
BEAM_LOG_INFO() << "processing block " << height;
BEAM_LOG_WARNING() << "unexpected state: " << TRACE(value);
BEAM_LOG_ERROR() << "failed to apply tx: " << errorCode;
```

The macros check `Logger::will_log(level)` before constructing the `LogMessage` object, so disabled log levels cost nothing. In release builds (`NDEBUG`), `BEAM_LOG_DEBUG()` and `BEAM_LOG_VERBOSE()` expand to a no-op `LogMessageStub` that is optimized away entirely.

`TRACE(var)` is a shorthand that emits `" var=<value>"` for quick variable printing.

Logger instances are created as `std::shared_ptr<Logger>` RAII objects; when all references are released the logger is flushed and torn down.

---

## CMake Module Structure

### Adding a New Library

The standard pattern, illustrated by `core/CMakeLists.txt`:

```cmake
set(MY_LIB_SRC
    file1.cpp
    file2.cpp
)
add_library(mylib STATIC ${MY_LIB_SRC})
target_link_libraries(mylib
    PUBLIC
        utility          # exposed in headers
        Boost::boost
    PRIVATE
        secp256k1        # implementation detail only
)
```

- **`PUBLIC` dependencies** propagate to consumers of `mylib`.
- **`PRIVATE` dependencies** are hidden; consumers do not need them.

### Test Subdirectories

Unit tests live in `unittest/` subdirectories and are guarded by a CMake option:

```cmake
if(BEAM_TESTS_ENABLED)
    add_subdirectory(unittest)
endif()
```

This keeps test binaries out of production builds. Test targets are plain executables registered with `add_test()` via the `AddTest` module included at the top level.

### Beam Interface Target

A top-level `INTERFACE` target named `beam` carries the global include path and feature requirements:

```cmake
add_library(beam INTERFACE IMPORTED GLOBAL)
target_include_directories(beam INTERFACE
    ${PROJECT_SOURCE_DIR}
    ${PROJECT_SOURCE_DIR}/3rdparty)
target_compile_features(beam INTERFACE cxx_std_17)
```

All library targets link against `beam` to pick up the include path and standard requirement.

---

## Unit Test Conventions

Test files live in `unittest/` inside the relevant subsystem directory:

- `core/unittest/ecc_test.cpp`, `core/unittest/storage_test.cpp`
- `node/unittests/`

There is no single test framework dependency. Tests are standalone executables that `assert()` or call a lightweight `BEAM_VERIFY` helper. Each test file includes directly from its parent subsystem using relative paths (`../ecc_native.h`).

Tests that need to manipulate consensus parameters use `Rules::Scope` to override fork heights, then restore the original state on exit.
