# `rethdb-reader`

A dylib to be accessed via FFI in `op-service`'s `sources` package for reading information
directly from the `reth` database.

## Developing

**Building**

To build the dylib, you must first have the [Rust Toolchain][rust-toolchain] installed.

```sh
cargo build --release
```

**Docs**

Documentation is available via rustdoc.

```sh
cargo doc --open
```

**Linting**

```sh
cargo +nightly fmt -- && cargo +nightly clippy --all --all-features -- -D warnings
```

**Generating `testdata`**

The testdata block and `reth` database can be generated by running the `testdata` make directive:

```sh
ETH_RPC_URL="<your_L1_RPC_URL>" make testdata
```

**Generating the C header**

To generate the C header, first install `cbindgen` via `cargo install cbindgen --force`. Then, run the generation script:

```sh
./headgen.sh
```

### C Header

The C header below is generated by `cbindgen`, and it is the interface that consumers of the dylib use to call its exported
functions. Currently, the only exported functions pertain to reading fully hydrated block receipts from the database.

```c
#include <stdarg.h>
#include <stdbool.h>
#include <stdint.h>
#include <stdlib.h>

/**
 * A [ReceiptsResult] is a wrapper around a JSON string containing serialized [TransactionReceipt]s
 * as well as an error status that is compatible with FFI.
 *
 * # Safety
 * - When the `error` field is false, the `data` pointer is guaranteed to be valid.
 * - When the `error` field is true, the `data` pointer is guaranteed to be null.
 */
typedef struct ReceiptsResult {
  uint32_t *data;
  uintptr_t data_len;
  bool error;
} ReceiptsResult;

/**
 * A [OpenDBResult] is a wrapper of DB instance [BlockchainProvider]
 * as well as an error status that is compatible with FFI.
 *
 * # Safety
 * - When the `error` field is false, the `data` pointer is guaranteed to be valid.
 * - When the `error` field is true, the `data` pointer is guaranteed to be null.
 */
typedef struct OpenDBResult {
  const void *data;
  bool error;
} OpenDBResult;

/**
 * Read the receipts for a blockhash from the RETH database directly.
 *
 * # Safety
 * - All possible nil pointer dereferences are checked, and the function will return a failing
 *   [ReceiptsResult] if any are found.
 */
struct ReceiptsResult rdb_read_receipts(const uint8_t *block_hash,
                                        uintptr_t block_hash_len,
                                        const void *db_instance);

/**
 * Free a string that was allocated in Rust and passed to C.
 *
 * # Safety
 * - All possible nil pointer dereferences are checked.
 */
void rdb_free_string(char *string);

/**
 * Open a DB instance and return.
 *
 * # Safety
 * - All possible nil pointer dereferences are checked, and the function will return a failing
 *   [OpenDBResult] if any are found.
 */
struct OpenDBResult open_db_read_only(const char *db_path);
```

[rust-toolchain]: https://rustup.rs/
