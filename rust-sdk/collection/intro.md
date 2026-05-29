A `NitriteCollection` stores schemaless `Document` values under a logical name. Collections are the most direct way to use Nitrite: you work with documents, filters, indexes, processors, and events without adding an entity layer.

## Get or create a collection

`Nitrite::collection(name)` returns a named collection and creates it if it does not already exist.

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

let collection = db.collection("users").expect("collection open failed");
```

Collection names must be valid and cannot collide with repository-backed collection names.

## What collections are good at

Collections are a good fit when:

- the stored shape is flexible
- you are ingesting heterogeneous records
- you want to work with `Document` directly
- you are writing utilities, scripts, or migrations where a typed entity model adds little value

If your domain model is stable and strongly typed, use [Object Repository](../repository/intro.md) instead.

## Documents, IDs, and metadata

Every collection stores `Document` values. Nitrite manages a few reserved fields automatically, including the internal `_id` field, revision metadata, and modification timestamps.

```rust
use nitrite::doc;

let collection = db.collection("users").expect("collection open failed");
collection.insert(doc! {
    "name": "Alice",
    "email": "alice@example.com"
}).expect("insert failed");
```

The inserted document receives a `NitriteId` if it did not already have one.

## Core collection capabilities

`NitriteCollection` combines two kinds of behavior:

- document CRUD through methods such as `insert`, `update`, `remove`, `find`, and `get_by_id`
- persistent-collection capabilities such as indexing, processors, attributes, and event subscriptions

The remaining pages in this section cover those capabilities in detail:

- [Read Operations](read.md)
- [Write Operations](write.md)
- [Indexing](indexing.md)
- [Other Operations](other.md)
