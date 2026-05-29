---
icon: database
label: Nitrite Database
order: 19
---

Nitrite is an embedded, serverless database. In the Rust SDK, the database handle is the `Nitrite` type and the entry point for configuration is `Nitrite::builder()`.

## Creating a database

### In-memory database

If you do not register a store module, Nitrite auto-loads its built-in in-memory store.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");
```

This mode is useful for tests, short-lived tools, and workloads where persistence is not required.

### Fjall-backed database

Persistent storage is currently provided through the `nitrite-fjall-adapter` crate.

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::FjallModule;

let storage = FjallModule::with_config()
    .db_path("./data/nitrite")
    .build();

let db = Nitrite::builder()
    .load_module(storage)
    .open_or_create(None, None)
    .expect("failed to open database");
```

More on the persistent store options is available in [Storage Modules](modules/store-modules/store-modules.md) and [Fjall Module](modules/store-modules/fjall.md).

## NitriteBuilder

`NitriteBuilder` is the Rust SDK's configuration surface. It lets you build a database instance incrementally and applies validation when `open_or_create()` is called.

### Open or create

`open_or_create(username, password)` accepts optional credentials.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(Some("app-user"), Some("secret-passphrase"))
    .expect("failed to open database");
```

The credential rules are strict:

- either provide both username and password
- or provide neither
- if credentials are provided and the database has no users yet, Nitrite creates the first user during authentication

### Field separator

Nested document paths use `.` by default. You can change that with `field_separator()`.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .field_separator("::")
    .open_or_create(None, None)
    .expect("failed to open database");
```

This affects document field addressing for operations such as `Document::get()` and filter paths. See [Document](document.md) for nested field examples.

### Loading modules

Modules extend the core engine with store implementations and specialized indexers.

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::FjallModule;
use nitrite_spatial::SpatialModule;

let db = Nitrite::builder()
    .load_module(
        FjallModule::with_config()
            .db_path("./data/nitrite")
            .build(),
    )
    .load_module(SpatialModule)
    .open_or_create(None, None)
    .expect("failed to open database");
```

### Schema version and migrations

The builder tracks a schema version and accepts registered migrations.

```rust
use nitrite::common::Value;
use nitrite::migration::Migration;
use nitrite::nitrite::Nitrite;

let migration = Migration::new(1, 2, |instruction| {
    instruction
        .for_collection("users")
        .add_field("status", Some(Value::String("active".to_string())), None)
        .create_index("NON_UNIQUE", &["status"]);
    Ok(())
});

let db = Nitrite::builder()
    .schema_version(2)
    .add_migration(migration)
    .open_or_create(None, None)
    .expect("failed to open migrated database");
```

See [Schema Migration](migration.md) for the available database, collection, and repository instruction builders.

## Working with database handles

Once the database is open, the `Nitrite` handle is the main gateway to collections and repositories.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

let collection = db.collection("users").expect("collection open failed");
let repo = db.repository::<User>().expect("repository open failed");
let keyed = db.keyed_repository::<User>("tenant-a").expect("keyed repository open failed");

db.commit().expect("commit failed");
db.close().expect("close failed");
```

Useful database-level operations include:

- `collection(name)` to get or create a named document collection
- `repository::<T>()` and `keyed_repository::<T>(key)` for typed access
- `has_collection(name)`, `has_repository::<T>()`, and `has_keyed_repository::<T>(key)` for discovery
- `destroy_collection(name)`, `destroy_repository::<T>()`, and `destroy_keyed_repository::<T>(key)` for cleanup
- `commit()` to flush pending changes to the underlying store
- `close()` to shut the database down cleanly

In the repository examples above, `User` is any type that implements `Default`, `Convertible`, and `NitriteEntity`.