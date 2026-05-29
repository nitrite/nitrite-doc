Repository writes follow the same lifecycle as collection writes, but accept typed entities instead of raw `Document` values.

## Insert entities

```rust
repo.insert(User {
    id: 1,
    name: "Alice".to_string(),
    email: "alice@example.com".to_string(),
}).expect("insert failed");

repo.insert_many(vec![
    User {
        id: 2,
        name: "Bob".to_string(),
        email: "bob@example.com".to_string(),
    },
    User {
        id: 3,
        name: "Carol".to_string(),
        email: "carol@example.com".to_string(),
    },
]).expect("batch insert failed");
```

## Update by filter

Use `update()` for the default behavior or `update_with_options()` when you want insert-if-absent or just-once semantics.

```rust
use nitrite::collection::just_once;
use nitrite::filter::field;

let options = just_once();
let updated = User {
    id: 1,
    name: "Alice".to_string(),
    email: "alice@new-domain.example".to_string(),
};

repo.update_with_options(
    field("email").eq("alice@example.com"),
    updated,
    &options,
).expect("update failed");
```

## Update by entity ID

`update_one()` derives the unique lookup filter from the entity's configured ID metadata.

```rust
repo.update_one(User {
    id: 1,
    name: "Alice".to_string(),
    email: "alice@example.com".to_string(),
}, false).expect("update_one failed");
```

If you already have the internal `NitriteId`, `update_by_nitrite_id()` is the most direct update path.

```rust
use nitrite::filter::all;

let mut cursor = repo.find(all()).expect("query failed");

for row in cursor.iter_with_id() {
    let (id, mut user) = row.expect("cursor row failed");
    user.name = user.name.to_uppercase();
    repo.update_by_nitrite_id(&id, user, false)
        .expect("update_by_nitrite_id failed");
}
```

## Partial raw-document updates

If you need to patch repository data with a raw `Document`, use `update_document()`.

```rust
use nitrite::doc;
use nitrite::filter::field;

let patch = doc! { "status": "inactive" };
repo.update_document(field("id").eq(1), &patch, true)
    .expect("update_document failed");
```

This is useful when the update payload does not naturally map to a full entity instance.

## Remove entities

Use `remove_one()` for a specific entity or `remove()` for filtered deletion.

```rust
use nitrite::filter::field;

repo.remove(field("status").eq("inactive"), false)
    .expect("remove failed");

repo.remove_one(User {
    id: 3,
    name: String::new(),
    email: String::new(),
}).expect("remove_one failed");
```

`remove_one()` uses the entity's ID metadata. The non-ID fields in the example above are ignored for the lookup.
