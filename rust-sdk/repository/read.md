---
label: Read Operations
icon: search
order: 13
---

Repository reads are the typed equivalent of collection reads. The result cursor converts each matching document back into your entity type lazily. In the current 0.3 line, the underlying `find()` cursor is streaming, so `reset()` reruns the query instead of replaying a cached snapshot.

## Read by ID

Use `get_by_id()` when you already know the repository ID value.

```rust
let maybe_user = repo.get_by_id(&1).expect("lookup failed");
assert!(maybe_user.is_some());
```

The ID type must match the entity's configured `NitriteEntity::Id` type.

## Query with filters

`find()` accepts the same core `Filter` values that collections use.

```rust
use nitrite::filter::field;

let mut users = repo.find(field("name").eq("Alice"))
    .expect("query failed");

if let Some(Ok(user)) = users.first() {
    println!("{}", user.email);
}
```

## Sorting and pagination

`find_with_options()` uses `FindOptions` exactly like collection queries do.

```rust
use nitrite::collection::FindOptions;
use nitrite::common::SortOrder;
use nitrite::filter::all;

let options = FindOptions::new()
    .sort_by("name".to_string(), SortOrder::Ascending)
    .skip(20)
    .limit(20);

let mut page = repo.find_with_options(all(), &options)
    .expect("query failed");

println!("page size: {}", page.size());
```

As with collection cursors, fully index-covered repository queries can answer `size()` without materializing every entity.

## Cursor helpers

`ObjectCursor<T>` adds a few repository-specific conveniences:

- `first()` to fetch the first entity
- `size()` to count the result set, with a fast path for fully index-covered queries
- `reset()` to restart iteration; streaming cursors rerun the query, while rewindable cursors replay cached rows
- `iter_with_id()` to stream `(NitriteId, T)` pairs
- `project::<P>()` to map each result to a projection type

```rust
use nitrite::filter::all;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
struct UserSummary {
    name: String,
    email: String,
}

let mut cursor = repo.find(all()).expect("query failed");
let mut projected = cursor.project::<UserSummary>()
    .expect("projection failed");

if let Some(Ok(summary)) = projected.next() {
    println!("{} <{}>", summary.name, summary.email);
}
```

If you need cursor composition across two result sets, `ObjectCursor<T>` also exposes `join()` with a `Lookup`.

## Drop to raw documents when needed

If a read flow needs raw documents instead of typed entities, use `document_collection()` and switch to the collection cursor API.

```rust
let collection = repo.document_collection();
let mut docs = collection.find(nitrite::filter::all())
    .expect("query failed");
let _ = docs.next();
```