# Getting Started in Rust

Nitrite for Rust is organized as a small crate ecosystem. The core `nitrite` crate provides collections, repositories, indexes, filters, transactions, and migrations. Companion crates add derive macros, persistent storage, spatial indexing, and full-text search.

## Add dependencies

The core setup is the `nitrite` crate plus `nitrite-derive` if you want typed repositories.

```toml
[dependencies]
nitrite = "0.4"
nitrite-derive = "0.4"
```

Nitrite's published crates currently use Rust edition 2021. Optional capabilities are split into separate crates:

```toml
[dependencies]
nitrite = "0.4"
nitrite-derive = "0.4"
nitrite-fjall-adapter = "0.4"
nitrite-spatial = "0.4"
nitrite-tantivy-fts = "0.4"
```

Use only the crates you need:

- `nitrite` for the core document database API.
- `nitrite-derive` for `Convertible` and `NitriteEntity` derive macros.
- `nitrite-fjall-adapter` for persistent Fjall-backed storage.
- `nitrite-spatial` for spatial indexes and geospatial filters.
- `nitrite-tantivy-fts` for Tantivy-backed full-text search.

## Open a database

If you do not load a storage module, Nitrite starts with the built-in in-memory store.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");
```

The builder entry point is `Nitrite::builder()`. The returned `NitriteBuilder` lets you configure:

- a custom field separator for nested document paths
- the target schema version
- migrations
- additional modules such as Fjall, spatial indexing, or full-text search

## Work with a collection

Collections store schemaless `Document` values. The `doc!` macro is the shortest way to build them.

```rust
use nitrite::doc;
use nitrite::filter::field;
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

let collection = db.collection("users").expect("missing collection");

collection.insert(doc! {
    "name": "John Doe",
    "age": 30,
    "active": true
}).expect("insert failed");

let mut cursor = collection
    .find(field("name").eq("John Doe"))
    .expect("query failed");

if let Some(Ok(document)) = cursor.first() {
    println!("matched document: {:?}", document);
}
```

Collections are the best fit when:

- the document shape is flexible or evolves frequently
- you want to work directly with `Document` values
- you are building tooling, migrations, or mixed-schema ingestion flows

## Work with a typed repository

Repositories wrap a backing collection and map documents to Rust types. A repository type must implement both `Convertible` and `NitriteEntity`. In practice that usually means deriving both macros and providing an entity ID.

```rust
use nitrite::filter::field;
use nitrite::nitrite::Nitrite;
use nitrite::repository::ObjectRepository;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct User {
    id: i64,
    name: String,
    email: String,
}

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

let repo: ObjectRepository<User> = db.repository().expect("repository open failed");

repo.insert(User {
    id: 1,
    name: "Alice".to_string(),
    email: "alice@example.com".to_string(),
}).expect("insert failed");

let user = repo.get_by_id(&1).expect("read failed");
let mut users = repo.find(field("name").eq("Alice")).expect("query failed");

if let Some(Ok(found)) = users.first() {
    println!("found user: {}", found.email);
}

assert!(user.is_some());
```

Use a repository when:

- you want compile-time type safety around stored entities
- your domain type has a stable identifier and index definition
- you want projection, joins, or keyed repositories for a typed model

## Add persistence or optional modules

The core crate loads an in-memory store automatically. File-based persistence and advanced indexes are added as modules.

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

Spatial and full-text modules are loaded through the same builder API. See [Module System](modules/module-system.md), [Storage Modules](modules/store-modules/store-modules.md), [Spatial Module](modules/spatial.md), and [Full-Text Search](modules/full-text-search.md).

## Next steps

- Use [Nitrite Database](database.md) for builder options, database lifecycle, and persistence choices.
- Use [Document](document.md) for document structure, nested fields, and reserved fields.
- Use [Nitrite Collection](collection/intro.md) for schemaless collection operations.
- Use [Object Repository](repository/intro.md) for entity mapping and typed CRUD.
- Use [Transaction](transaction.md) and [Schema Migration](migration.md) when you need coordinated writes and controlled schema evolution.
