---
label: Full-Text Search
icon: search
order: 1
---

The `nitrite-tantivy-fts` crate adds Tantivy-backed full-text indexing and a dedicated full-text query DSL.

## Add the dependency

```toml
[dependencies]
nitrite = "0.3"
nitrite-tantivy-fts = "0.3"
```

## Load the module

The simplest configuration uses `TantivyFtsModule::default()`.

```rust
use nitrite::nitrite::Nitrite;
use nitrite_tantivy_fts::TantivyFtsModule;

let db = Nitrite::builder()
    .load_module(TantivyFtsModule::default())
    .open_or_create(None, None)
    .expect("failed to open database");
```

You can also build a tuned module configuration.

```rust
use nitrite_tantivy_fts::TantivyFtsModule;

let module = TantivyFtsModule::with_config()
    .index_writer_heap_size(100 * 1024 * 1024)
    .num_threads(4)
    .build();
```

## Create an FTS index

```rust
use nitrite_tantivy_fts::fts_index;

let collection = db.collection("articles").expect("collection open failed");
collection.create_index(vec!["content"], &fts_index())
    .expect("fts index failed");
```

## Query with `fts_field`

```rust
use nitrite_tantivy_fts::fts_field;

let mut matches = collection.find(
    fts_field("content").matches("+rust +database")
).expect("fts query failed");

let mut exact_phrase = collection.find(
    fts_field("content").phrase("embedded database")
).expect("fts phrase query failed");

let _ = matches.next();
let _ = exact_phrase.next();
```

`matches()` uses Tantivy's query parser, so it supports multi-term searches, required terms with `+`, excluded terms with `-`, and boolean operators. `contains()` and `text()` are aliases for `matches()`.

## Batch commit behavior

In the current 0.3 line, full-text writes and deletes are buffered and committed on the next search or on `db.close()` instead of once per document. Searches still observe their own writes because Nitrite commits the dirty batch and reloads the Tantivy reader before executing the query.

For bulk indexing this is much faster than committing every document individually. A clean close flushes any remaining FTS batch; an unclean crash can lose only the uncommitted derived index batch, which Nitrite can rebuild.

## When to use this module

The core crate already supports `field("...").text(...)` filters and the core `full_text_index()` helper. Use the Tantivy module when you want a dedicated search index and query parser rather than the core text-matching path.