---
label: Schema Migration
icon: iterations
order: 10
---

Nitrite for Rust supports versioned schema evolution through `Migration` and `InstructionSet`. Migrations are registered on the builder and run when the stored schema version differs from the version you request with `schema_version()`.

## Define a migration

Use `Migration::new(from_version, to_version, |instruction| { ... })` to define the transition.

```rust
use nitrite::common::Value;
use nitrite::migration::Migration;

let migration = Migration::new(1, 2, |instruction| {
    instruction
        .for_collection("users")
        .rename_field("last_name", "family_name")
        .add_field("status", Some(Value::String("active".to_string())), None)
        .create_index("NON_UNIQUE", &["status"]);

    Ok(())
});
```

Register the migration when opening the database:

```rust
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .schema_version(2)
    .add_migration(migration)
    .open_or_create(None, None)
    .expect("failed to open migrated database");
```

## Migration scopes

`InstructionSet` exposes three builders, each grounded to a specific migration scope.

### Database instructions

`for_database()` is used for database-wide changes such as:

- `add_user(username, password)`
- `change_password(username, old_password, new_password)`
- `drop_collection(name)`
- `drop_repository(entity_name, key)`
- `custom_instruction(...)`

### Collection instructions

`for_collection(name)` works on raw document collections and supports:

- `rename(new_name)`
- `add_field(field_name, default_value, generator)`
- `rename_field(old_name, new_name)`
- `delete_field(field_name)`
- `drop_index(field_names)`
- `drop_all_indices()`
- `create_index(index_type, field_names)`

### Repository instructions

`for_repository(entity_name, key)` targets typed repositories and adds repository-specific operations:

- `rename_repository(new_entity_name, new_key)`
- `add_field(field_name, default_value, generator)`
- `rename_field(old_name, new_name)`
- `delete_field(field_name)`
- `change_data_type(field_name, converter)`
- `change_id_field(old_field_names, new_field_names)`
- `drop_index(field_names)`
- `drop_all_indices()`
- `create_index(index_type, field_names)`

## Generated values during migration

Collection and repository `add_field()` calls can use either a static `Value` or a generator closure that receives the current `Document`.

```rust
use nitrite::common::Value;
use nitrite::migration::Migration;

let migration = Migration::new(2, 3, |instruction| {
    instruction
        .for_repository("User", None)
        .add_field("display_name", None, Some(|doc| {
            let first = match doc.get("first_name")? {
                Value::String(value) => value,
                _ => String::new(),
            };
            let last = match doc.get("family_name")? {
                Value::String(value) => value,
                _ => String::new(),
            };

            Ok(Value::String(format!("{} {}", first, last).trim().to_string()))
        }));

    Ok(())
});
```

## Choosing index types

Migration builders take index types as string identifiers. For the core engine, the relevant values are:

- `UNIQUE`
- `NON_UNIQUE`
- `FULL_TEXT`

Specialized modules may add their own index type strings. For example, the Tantivy FTS and spatial modules each provide their own index options and matching filters.

## Practical guidance

- advance `schema_version()` only when you have a matching migration path
- keep each migration focused on one version transition
- prefer collection instructions for document-shape cleanup and repository instructions for entity-centric changes
- test migrations against a copy of real data before promoting them into production flows