Collections return `DocumentCursor` values for query results. In the current 0.3 line, the common `find()` cursor is streaming: it yields documents lazily and `reset()` rebuilds the query instead of replaying a cached snapshot. For a stable collection the observable results are the same; if the underlying data changes before `reset()`, the cursor reflects the current state.

## Query with filters

Use `find(filter)` for the common case.

```rust
use nitrite::filter::field;

let collection = db.collection("users").expect("collection open failed");
let mut cursor = collection
    .find(field("active").eq(true))
    .expect("query failed");

if let Some(Ok(document)) = cursor.first() {
    println!("first active user: {:?}", document);
}
```

To scan everything, use `nitrite::filter::all()`.

## Sorting, pagination, and distinct queries

`find_with_options()` accepts a `FindOptions` value.

```rust
use nitrite::collection::FindOptions;
use nitrite::common::SortOrder;
use nitrite::filter::field;

let options = FindOptions::new()
    .sort_by("age".to_string(), SortOrder::Descending)
    .skip(10)
    .limit(20);

let mut cursor = collection
    .find_with_options(field("active").eq(true), &options)
    .expect("query failed");

println!("page size: {}", cursor.size());
```

When a query is fully covered by an index, `size()` can answer from the index metadata without materializing every matching document. Indexed bounded range filters also use bounded scans in the current 0.3 line, so narrow range queries stay narrow instead of scanning from one side and post-filtering.

There are also convenience constructors:

- `order_by(field, sort_order)`
- `skip_by(count)`
- `limit_to(count)`
- `distinct()`

## Read by internal ID

If you already have a `NitriteId`, `get_by_id()` is the most direct path.

```rust
let maybe_document = collection.get_by_id(&id).expect("lookup failed");
```

This is an $O(1)$ lookup against the document's internal identifier.

## Cursor helpers

`DocumentCursor` exposes a few helpers that are useful in tooling and update flows:

- `first()` to fetch the first matching document
- `size()` to count the result set, with a fast path for fully index-covered queries
- `reset()` to restart iteration; streaming cursors rerun the query, while rewindable cursors replay cached rows
- `iter_with_id()` to stream `(NitriteId, Document)` pairs

```rust
let mut cursor = collection.find(nitrite::filter::all()).expect("query failed");

for row in cursor.iter_with_id() {
    let (id, document) = row.expect("cursor row failed");
    println!("{} -> {:?}", id, document);
}
```

`iter_with_id()` is especially useful when you plan to feed the IDs into `update_by_id()` later.

## Joins and projections

Document cursors can join documents as a post-query step.

```rust
use nitrite::common::Lookup;
use nitrite::doc;

let mut users = db.collection("users").expect("collection open failed")
    .find(nitrite::filter::all())
    .expect("query failed");

let mut teams = db.collection("teams").expect("collection open failed")
    .find(nitrite::filter::all())
    .expect("query failed");

let lookup = Lookup::new("team_id", "id", "team");
let mut joined = users.join(&mut teams, &lookup).expect("join failed");
let _ = joined.next();
```

Use joins sparingly for read-model shaping. For projection-oriented typed read pipelines, the repository cursor API is usually a better fit.
