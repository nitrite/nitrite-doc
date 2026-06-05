---
label: Usage
icon: code
order: 3
---

Unlike the Java and Flutter SDKs, the Rust SDK does not have a separate `nitrite-support` package. Instead, functionality is split into focused crates.

## Core installation

For most applications, start with the core crate and the derive macros.

```bash
cargo add nitrite
cargo add nitrite-derive
```

Equivalent `Cargo.toml` entries look like this:

```toml
[dependencies]
nitrite = "0.3"
nitrite-derive = "0.3"
```

## Optional crates

Add only the modules you need.

```bash
cargo add nitrite-fjall-adapter
cargo add nitrite-spatial
cargo add nitrite-tantivy-fts
```

```toml
[dependencies]
nitrite = "0.3"
nitrite-derive = "0.3"
nitrite-fjall-adapter = "0.3"
nitrite-spatial = "0.3"
nitrite-tantivy-fts = "0.3"
```

## One naming detail to remember

Cargo dependency names use hyphens, while Rust import paths use underscores.

For example:

- dependency: `nitrite-fjall-adapter`
- import: `use nitrite_fjall_adapter::FjallModule;`

The same pattern applies to `nitrite-spatial` -> `nitrite_spatial` and `nitrite-tantivy-fts` -> `nitrite_tantivy_fts`.

## Versioning guidance

Keep the Nitrite crates on the same release line unless you have a specific compatibility reason not to. The current source tree uses `0.3.x` across the core, derive, Fjall, spatial, and Tantivy FTS crates.