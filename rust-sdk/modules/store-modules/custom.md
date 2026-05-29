---
label: Custom Storage Modules
icon: check-circle
order: 1
---

Nitrite's storage layer is public and extensible. If Fjall is not the right persistence strategy for your application, you can build a custom store module.

## Storage extension points

At the module boundary, custom storage is expressed through `StoreModule`.

```rust
use nitrite::errors::NitriteResult;
use nitrite::store::{NitriteStore, StoreModule};
use nitrite::NitriteModule;

pub trait StoreModule: NitriteModule {
    fn get_store(&self) -> NitriteResult<NitriteStore>;
}
```

In practice, a custom storage integration has two layers:

- a store provider implementing `NitriteStoreProvider`
- a module implementing `NitriteModule` and `StoreModule`

## What a store provider is responsible for

`NitriteStoreProvider` is the low-level contract. A custom store must handle:

- opening or creating the physical store
- returning collection and repository registries
- opening maps
- committing and compacting persisted state
- publishing store events
- closing cleanly

Because `NitriteStoreProvider` extends `NitritePluginProvider`, it also participates in plugin initialization and shutdown.

## Registering the store through a module

The module wrapper is responsible for registering the custom store with the plugin registrar.

```rust
use nitrite::common::{NitriteModule, NitritePlugin, PluginRegistrar};
use nitrite::errors::NitriteResult;
use nitrite::store::{NitriteStore, StoreModule};

#[derive(Clone)]
struct MyStoreModule {
    store: NitriteStore,
}

impl NitriteModule for MyStoreModule {
    fn plugins(&self) -> NitriteResult<Vec<NitritePlugin>> {
        Ok(vec![self.store.as_plugin()])
    }

    fn load(&self, plugin_registrar: &PluginRegistrar) -> NitriteResult<()> {
        plugin_registrar.register_store_plugin(self.store.clone())
    }
}

impl StoreModule for MyStoreModule {
    fn get_store(&self) -> NitriteResult<NitriteStore> {
        Ok(self.store.clone())
    }
}
```

The `store` field in that example is typically constructed with `NitriteStore::new(my_provider)` where `my_provider` is your `NitriteStoreProvider` implementation.

## When to build a custom store

Build a custom store only when you need behavior the current ecosystem does not provide, such as:

- a different persistence engine
- a specialized durability model
- platform-specific storage constraints
- custom observability or operational hooks at the storage layer