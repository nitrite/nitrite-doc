---
label: Potassium Nitrite Module
icon: briefcase
order: 8
---

Nitrite is a modular database engine, so is Potassium Nitrite. All Nitrite modules are compatible with Potassium Nitrite as well. More details about Nitrite modules can be found [here](../java-sdk/modules/module-system.md).

Potassium Nitrite provides a single module `KNO2Module` to load all plugins it provides. You can load the module while opening the database.

```kotlin
val db = nitrite {
    loadModule(KNO2Module())
}
```

This module loads the following plugins:

- [KNO2JacksonMapper](repository.md#jackson-mapper) - A Jackson based mapper for Potassium Nitrite.
- [SpatialIndexer](../java-sdk/modules/spatial.md#spatial-index) - A spatial index plugin for Nitrite.

Visit the respective pages to know more about the plugins.