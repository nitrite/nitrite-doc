---
label: Introduction
icon: container
order: 17
---

An `ObjectRepository<T>` is Nitrite's typed persistence API. It stores Rust values as documents internally, but exposes typed CRUD and query operations at the call site.

## Repository requirements

Repository entities must implement:

- `Default`
- `Convertible<Output = T>`
- `NitriteEntity`
- `Send + Sync`

In most applications, you satisfy those requirements with the derive macros from `nitrite-derive`.

```rust
use nitrite::nitrite::Nitrite;
use nitrite::repository::ObjectRepository;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct User {
    id: i64,
    name: String,
    email: String,
}

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

let repo: ObjectRepository<User> = db.repository().expect("repository open failed");
```

## Keyed repositories

The Rust SDK also supports keyed repositories. They let you keep multiple repository namespaces for the same entity type.

```rust
let live: ObjectRepository<User> = db.repository().expect("repository open failed");
let archived: ObjectRepository<User> = db.keyed_repository("archived")
    .expect("keyed repository open failed");
```

This is useful for patterns such as tenant partitioning, soft archives, or environment-specific datasets that still share the same entity definition.

## Repositories are backed by collections

Every repository is backed by a Nitrite collection. That means repositories inherit collection capabilities such as:

- indexes
- processors
- attributes
- event listeners
- lifecycle and maintenance operations

If you need to drop down to raw document access, use `document_collection()` on the repository.

## When to choose a repository

Use a repository when:

- your data has a stable Rust type
- you want type-safe reads and writes
- you want entity IDs and indexes declared next to the model
- you need typed cursor helpers such as projection

Use a collection instead when the data shape is still fluid or when you intentionally want raw document access.