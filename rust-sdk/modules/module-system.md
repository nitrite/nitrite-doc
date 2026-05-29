---
label: Module System
icon: workflow
order: 4
---

Nitrite for Rust is modular by design. Modules are how storage engines and specialized indexers are added to the core engine.

## Core concepts

The module system is centered on three public types:

- `NitriteModule` for module registration
- `NitritePluginProvider` for plugin lifecycle (`initialize()`, `close()`, `as_plugin()`)
- `PluginRegistrar` for registering stores and indexers during module loading

At runtime, the plugin manager keeps registries of store and indexer plugins.

## What the core engine loads automatically

When no custom module overrides them, the core plugin manager auto-loads:

- the built-in in-memory store
- the unique indexer
- the non-unique indexer
- the core full-text indexer

That is why a plain `Nitrite::builder().open_or_create(None, None)` works without any extra setup.

## Loading modules

Modules are added through `NitriteBuilder::load_module()`.

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::FjallModule;
use nitrite_spatial::SpatialModule;
use nitrite_tantivy_fts::TantivyFtsModule;

let db = Nitrite::builder()
    .load_module(
        FjallModule::with_config()
            .db_path("./data/nitrite")
            .build(),
    )
    .load_module(SpatialModule)
    .load_module(TantivyFtsModule::default())
    .open_or_create(None, None)
    .expect("failed to open database");
```

## How custom modules register functionality

A custom module implements `NitriteModule` and uses `PluginRegistrar` inside `load()`.

```rust
use nitrite::common::{NitriteModule, NitritePlugin, PluginRegistrar};
use nitrite::errors::NitriteResult;

struct MyModule;

impl NitriteModule for MyModule {
    fn plugins(&self) -> NitriteResult<Vec<NitritePlugin>> {
        Ok(vec![])
    }

    fn load(&self, _plugin_registrar: &PluginRegistrar) -> NitriteResult<()> {
        // Register a store or indexer plugin here.
        Ok(())
    }
}
```

For storage modules, see [Storage Modules](store-modules/store-modules.md) and [Custom Storage Modules](store-modules/custom.md).

## Choosing between core and optional modules

Use the core engine alone when you need:

- in-memory storage
- unique or non-unique indexing
- core document and repository APIs

Add modules when you need:

- on-disk persistence with Fjall
- spatial indexes and geospatial filters
- Tantivy-backed full-text search