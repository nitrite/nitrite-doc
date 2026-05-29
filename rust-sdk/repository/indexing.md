Repositories can be indexed in two ways:

- declare indexes on the entity with `#[entity(index(...))]`
- create and maintain indexes dynamically through the repository handle

## Entity-declared indexes

The derive macro is the cleanest option when the index shape is part of the domain model.

```rust
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(
    id(field = "id"),
    index(type = "unique", fields = "isbn"),
    index(type = "non-unique", fields = "author, year")
)]
struct Book {
    id: i64,
    title: String,
    author: String,
    isbn: String,
    year: i32,
}
```

## Runtime index management

Repositories also implement `PersistentCollection`, so you can manage indexes through the repository handle.

```rust
use nitrite::index::{full_text_index, non_unique_index, unique_index};

repo.create_index(vec!["email"], &unique_index())
    .expect("unique index failed");
repo.create_index(vec!["department"], &non_unique_index())
    .expect("non-unique index failed");
repo.create_index(vec!["bio"], &full_text_index())
    .expect("full-text index failed");
```

Compound indexes work the same way as they do on collections.

```rust
repo.create_index(vec!["last_name", "first_name"], &non_unique_index())
    .expect("compound index failed");
```

## Inspect and maintain repository indexes

The same maintenance operations available on collections are available on repositories:

- `has_index(field_names)`
- `list_indexes()`
- `is_indexing(field_names)`
- `rebuild_index(field_names)`
- `drop_index(field_names)`
- `drop_all_indexes()`

Use entity-declared indexes for the stable baseline and runtime index calls when indexing needs to remain operationally configurable.
