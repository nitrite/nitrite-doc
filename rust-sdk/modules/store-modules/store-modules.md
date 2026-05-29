Nitrite for Rust always has a store backend. If you do not register one explicitly, the plugin manager loads the built-in in-memory store. If you need persistence, add a store module.

The supported store paths in the current Rust codebase are:

- built-in in-memory storage with no extra crate
- persistent Fjall-backed storage via `nitrite-fjall-adapter`
- custom stores built on the public store traits

- [Fjall Module](fjall.md)
- [Custom Storage Modules](custom.md)

## In-memory by default

```rust
let db = nitrite::nitrite::Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");
```

This path requires no additional dependency and is ideal for tests and ephemeral workloads.

## Add persistence when needed

As soon as you need a file-backed store, register a store module through `load_module()` before opening the database.
