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
