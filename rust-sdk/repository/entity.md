---
label: Entity
icon: id-badge
order: 16
---

`NitriteEntity` supplies repository metadata: the entity name, the repository ID definition, and any declared indexes. In day-to-day Rust code this is usually derived, not implemented by hand.

## A basic entity

```rust
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct User {
    id: i64,
    name: String,
    email: String,
}
```

This gives Nitrite enough metadata to:

- name the repository
- build an ID filter for `get_by_id()` and `update_one()`
- attach any declared indexes

## Custom entity names

You can override the repository entity name with `name = "..."`.

```rust
#[derive(Default, Convertible, NitriteEntity)]
#[entity(name = "users", id(field = "id"))]
struct User {
    id: i64,
    name: String,
}
```

## Embedded and compound IDs

Entity IDs can be defined on nested value objects through `embedded_fields`.

```rust
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible)]
struct BookId {
    author: String,
    isbn: String,
}

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "book_id", embedded_fields = "author, isbn"))]
struct Book {
    book_id: BookId,
    title: String,
}
```

When Nitrite builds ID filters for this entity, it addresses the encoded nested fields under the configured field separator.

## Declaring indexes on the entity

Indexes can live beside the entity definition.

```rust
use nitrite::collection::NitriteId;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(
    id(field = "id"),
    index(type = "unique", fields = "isbn"),
    index(type = "non-unique", fields = "author, year")
)]
struct Book {
    id: NitriteId,
    title: String,
    author: String,
    isbn: String,
    year: i32,
}
```

This keeps repository metadata close to the domain type and avoids repeating index definitions at every call site.

## `NitriteId` support

If you want Nitrite to manage entity identifiers automatically, use `NitriteId` as the ID field type.

```rust
use nitrite::collection::NitriteId;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct Order {
    id: NitriteId,
    total_cents: i64,
}
```

The runtime trait behind the derive macro exposes this metadata through `entity_name()`, `entity_indexes()`, and `entity_id()`.