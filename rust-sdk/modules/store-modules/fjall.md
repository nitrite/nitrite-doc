`nitrite-fjall-adapter` is the persistent store module in the current Rust Nitrite ecosystem. It uses Fjall as an embedded LSM-tree backend.

## Add the dependency

```toml
[dependencies]
nitrite = "0.2"
nitrite-fjall-adapter = "0.2"
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

For most applications, start with `production_preset()` and tune only after measuring your workload.
