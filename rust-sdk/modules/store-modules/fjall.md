---
label: Fjall Module
icon: database
order: 2
---

`nitrite-fjall-adapter` is the persistent store module in the current Rust Nitrite ecosystem. It uses Fjall as an embedded LSM-tree backend.

## Add the dependency

```toml
[dependencies]
nitrite = "1.0"
nitrite-fjall-adapter = "1.0"
```

Since `1.0.0` the adapter is built on **Fjall 3**. Fjall 3 changes the on-disk format, so a
database written by `0.10.x` or earlier cannot be opened — see
[Schema Migration](../../migration.md#upgrading-from-010x-to-100).

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

The adapter defaults to `Durability::Periodic`. Commits are buffered to the OS and fsynced by a background timer within the configured `fsync_frequency()` window, which is `1000` ms by default. That keeps acknowledged writes safe across a process crash while trading a bounded power-loss window for much better throughput than fsyncing every commit.

Fjall 3 removed its own background fsync timer, so since `1.0.0` the adapter runs that timer itself. The behaviour and the `fsync_frequency()` knob are unchanged; `fsync_frequency(0)` still disables the timer, and a `Periodic` write is then only fsynced on a clean close.

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
- `compaction_strategy(...)`
- `kv_separated(...)`
- `staleness_threshold(...)`

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

### What changed in `1.0.0`

- `compaction_strategy(...)` takes `nitrite_fjall_adapter::Strategy` — `Leveled` (the default) or
  `Fifo` — instead of `fjall::compaction::Strategy`. Fjall 3 replaced that enum with a boxed
  trait object and dropped size-tiered compaction, so `SizeTiered` is gone.
- `space_amp_factor(...)` was **removed**. It only ever fed Fjall 2's explicit
  `gc_with_space_amp_target` pass; Fjall 3 folds blob reclamation into ordinary compaction and has
  no space-amplification target to aim at.
- `staleness_threshold(...)` is still there and still the ratio that drives blob reclamation, but
  it is now applied when the keyspace is created rather than per garbage-collection call. It only
  has an effect with `kv_separated(true)`.
- `flush_workers(...)` and `compaction_workers(...)` are both still accepted, but Fjall 3 has a
  single shared worker pool, so the adapter sizes it at the larger of the two.
- `max_journaling_size(...)` now has a floor of 64 MiB (Fjall 2's was 24 MiB); Fjall panics below
  it. The default is 512 MiB, so this only matters if you were setting it explicitly.

## On-disk key ordering

Fjall is an LSM-tree that keeps keys sorted by their raw bytes. Since `0.4.0`, the adapter serializes keys with an **order-preserving codec** so that byte order matches Nitrite's value ordering. This makes indexed range filters (`between`, `gte`/`lte`) and sorted index scans exact for every comparable type — including negative and large integers (for example nanosecond timestamps order correctly, beyond `f64` precision) — instead of the byte-order artifacts the previous encoding produced.

This is a transparent change to your code, but it does change the on-disk format.

!!!warning Upgrading from `0.3.x`
A database written by `0.3.x` cannot be opened by `0.4.x` as-is: both the key encoding and the non-unique/compound index layout changed. Rebuild the database (or drop and re-create its indexes) on upgrade. See [Schema Migration](../../migration.md#upgrading-from-03x-to-04x).
!!!

!!!warning Upgrading from `0.10.x`
`1.0.0` moves to Fjall 3, whose own on-disk format is not readable by Fjall 2 and vice versa. This is a storage-engine change, not a Nitrite one — no application API changed beyond the tuning knobs listed above. See [Schema Migration](../../migration.md#upgrading-from-010x-to-100).
!!!