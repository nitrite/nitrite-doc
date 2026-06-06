---
label: Indexing
icon: list-unordered
order: 14
---

Indexes accelerate collection queries at the cost of additional write-time maintenance. The core Rust SDK supports unique, non-unique, and core full-text indexes through `IndexOptions` helpers.

## Create indexes

```rust
use nitrite::index::{full_text_index, non_unique_index, unique_index};

let collection = db.collection("users").expect("collection open failed");

collection.create_index(vec!["email"], &unique_index())
    .expect("unique index failed");
collection.create_index(vec!["department"], &non_unique_index())
    .expect("non-unique index failed");
collection.create_index(vec!["bio"], &full_text_index())
    .expect("full-text index failed");
```

## Compound indexes

Pass more than one field name to create a compound index.

```rust
use nitrite::index::non_unique_index;

collection.create_index(vec!["last_name", "first_name"], &non_unique_index())
    .expect("compound index failed");
```

Compound indexes are useful when your most selective queries combine the same fields repeatedly.

## Write performance and low-cardinality fields

Since `0.4.0`, non-unique indexes (simple and compound) store one composite `(value…, id)` row per entry instead of a single growing array of ids per value. Each insert and removal is an O(1) point write, and per-insert memory is flat.

This matters most for **low-cardinality** fields — ones where many documents share the same value, such as a status, category, account, or folder id. Previously, bulk-loading such a field was O(n²) in time and O(n) in per-insert memory because every insert rewrote an ever-growing id array; it is now linear. You can index these fields freely without paying a quadratic write penalty:

```rust
use nitrite::index::non_unique_index;

// Low-cardinality, high-volume fields — cheap to keep indexed in 0.4.
collection.create_index(vec!["account_id"], &non_unique_index())
    .expect("account index failed");
collection.create_index(vec!["folder_id"], &non_unique_index())
    .expect("folder index failed");
```

Equality lookups on a non-unique field are prefix range scans (O(matches)), and range/sorted scans are exact — see [Read Operations](read.md) and [Filters](../filter.md).

!!!warning On-disk format changed in `0.4.0`
The non-unique/compound index layout (and the Fjall key encoding) changed in `0.4.0`. Indexes built by `0.3.x` are not readable by `0.4.x`; rebuild them on upgrade — see [Schema Migration](../migration.md#upgrading-from-03x-to-04x). Indexes are derived data, so `rebuild_index(...)` (or recreating the database) is all that is required.
!!!

## Inspect and maintain indexes

Persistent collections expose several maintenance methods:

- `has_index(field_names)`
- `list_indexes()`
- `is_indexing(field_names)`
- `rebuild_index(field_names)`
- `drop_index(field_names)`
- `drop_all_indexes()`

```rust
let has_email_index = collection.has_index(vec!["email"]).expect("has_index failed");
let descriptors = collection.list_indexes().expect("list_indexes failed");

if has_email_index {
    collection.rebuild_index(vec!["email"]).expect("rebuild failed");
}

println!("index count: {}", descriptors.len());
```

## Choosing an index type

Use the core types like this:

- `unique_index()` for fields such as email addresses or external IDs
- `non_unique_index()` for category, status, and other repeated values
- `full_text_index()` for lightweight text-searchable fields inside the core engine

Specialized modules add their own index types:

- `nitrite_spatial::spatial_index()` for spatial fields
- `nitrite_tantivy_fts::fts_index()` for Tantivy-backed full-text search

Those module-specific indexes are covered in the modules section.