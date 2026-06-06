`nitrite-fjall-adapter` is the persistent store module in the current Rust Nitrite ecosystem. It uses Fjall as an embedded LSM-tree backend.

## Add the dependency

```toml
[dependencies]
nitrite = "0.4"
nitrite-fjall-adapter = "0.4"
```

## Basic configuration

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::FjallModule;

let module = FjallModule::with_config()
    .db_path("./data/nitrite")
    .build();

let db = Nitrite::builder()
    .load_module(module)
    .open_or_create(None, None)
    .expect("failed to open database");
```

`db_path()` is the essential setting because it tells Fjall where to store the database files.

## Durability modes

In the current 0.4 line, Fjall defaults to `Durability::Periodic`. Commits are buffered to the OS and fsynced by a background timer within the configured `fsync_frequency()` window, which is `1000` ms by default. That keeps acknowledged writes safe across a process crash while trading a bounded power-loss window for much better throughput than fsyncing every commit.

If you want every commit fsynced before it returns, opt into `Durability::OnCommit` explicitly:

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::{Durability, FjallModule};

let module = FjallModule::with_config()
    .db_path("./data/nitrite")
    .durability(Durability::OnCommit)
    .build();

let db = Nitrite::builder()
    .load_module(module)
    .open_or_create(None, None)
    .expect("failed to open database");
```

## Presets

The builder ships with three presets for common operating modes.

```rust
use nitrite_fjall_adapter::FjallModule;

let production = FjallModule::with_config()
    .production_preset()
    .db_path("./data/prod")
    .build();

let throughput = FjallModule::with_config()
    .high_throughput_preset()
    .db_path("./data/bulk")
    .build();

let low_memory = FjallModule::with_config()
    .low_memory_preset()
    .db_path("./data/dev")
    .build();
```

Use them as starting points, then override specific settings if needed.

## Tuning knobs

The builder exposes a broad set of Fjall tuning parameters. The most relevant operational controls are:

- `block_cache_capacity(...)`
- `blob_cache_capacity(...)`
- `max_write_buffer_size(...)`
- `max_memtable_size(...)`
- `flush_workers(...)`
- `compaction_workers(...)`
- `manual_journal_persist(...)`
- `fsync_frequency(...)`
- `durability(...)`
- `compression_type(...)`
- `kv_separated(...)`

Example:

```rust
use nitrite_fjall_adapter::FjallModule;

let module = FjallModule::with_config()
    .db_path("./data/nitrite")
    .block_cache_capacity(256 * 1024 * 1024)
    .blob_cache_capacity(64 * 1024 * 1024)
    .max_write_buffer_size(128 * 1024 * 1024)
    .flush_workers(4)
    .compaction_workers(2)
    .build();
```

For most applications, start with `production_preset()` and tune only after measuring your workload. If you stay on `Durability::Periodic`, `fsync_frequency(...)` controls the background fsync interval.

## On-disk key ordering

Fjall is an LSM-tree that keeps keys sorted by their raw bytes. Since `0.4.0`, the adapter serializes keys with an **order-preserving codec** so that byte order matches Nitrite's value ordering. This makes indexed range filters (`between`, `gte`/`lte`) and sorted index scans exact for every comparable type — including negative and large integers (for example nanosecond timestamps order correctly, beyond `f64` precision) — instead of the byte-order artifacts the previous encoding produced.

This is a transparent change to your code, but it does change the on-disk format.

!!!warning Upgrading from `0.3.x`
A database written by `0.3.x` cannot be opened by `0.4.x` as-is: both the key encoding and the non-unique/compound index layout changed. Rebuild the database (or drop and re-create its indexes) on upgrade. See [Schema Migration](../../migration.md#upgrading-from-03x-to-04x).
!!!
