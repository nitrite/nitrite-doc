---
label: Write Operations
icon: pencil
order: 16
---

Collections support inserts, batch inserts, document updates, and removals. All write operations return a `WriteResult` containing the affected `NitriteId` values.

## Insert documents

Use `insert()` for a single document and `insert_many()` for a batch.

```rust
use nitrite::doc;

let collection = db.collection("users").expect("collection open failed");

collection.insert(doc! {
    "name": "Alice",
    "email": "alice@example.com"
}).expect("insert failed");

collection.insert_many(vec![
    doc! { "name": "Bob" },
    doc! { "name": "Carol" },
]).expect("batch insert failed");
```

## Update matching documents

Use `update()` for the default behavior or `update_with_options()` for more control.

```rust
use nitrite::collection::just_once;
use nitrite::doc;
use nitrite::filter::field;

let patch = doc! { "status": "inactive" };
let options = just_once();

collection.update_with_options(
    field("email").eq("alice@example.com"),
    &patch,
    &options,
).expect("update failed");
```

`UpdateOptions` controls two behaviors:

- `insert_if_absent` to insert a new document when no match exists
- `just_once` to stop after the first matching document

The helper constructors are `insert_if_absent()` and `just_once()`.

## Update one document by identity or ID

If you already have the full document, `update_one()` uses the document's `_id`. If you already know the `NitriteId`, `update_by_id()` is the most direct update path.

```rust
let mut document = collection
    .find(nitrite::filter::field("name").eq("Alice"))
    .expect("query failed")
    .first()
    .transpose()
    .expect("cursor failed")
    .expect("document missing");

let id = document.id().expect("missing id");
document.put("status", "active").expect("put failed");

collection.update_by_id(&id, &document, false).expect("update by id failed");
```

`update_by_id()` is an $O(1)$ lookup by internal identifier and is the most efficient update route when you already have the ID.

## Remove documents

Collections support both filtered removal and single-document removal.

```rust
use nitrite::filter::field;

collection.remove(field("status").eq("inactive"), false)
    .expect("remove failed");

collection.remove_one(&document).expect("remove one failed");
```

The `just_once` flag on `remove()` behaves the same way as it does for updates: `true` removes the first match only.

## Practical write patterns

Two common collection write patterns are:

- use `find(...).iter_with_id()` and then `update_by_id()` when you need efficient bulk rewrites
- use `update_with_options(..., &insert_if_absent())` when you want collection-level upsert behavior

If you want write operations against strongly typed entities instead of raw documents, move to [Object Repository write operations](../repository/write.md).