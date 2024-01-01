---
label: Module System
icon: gear
order: 4
---

Nitrite is a modular database engine. The core functionality of Nitrite is provided by the `nitrite` module. The `nitrite` module is the only mandatory module for Nitrite. All other modules are optional and can be added to the project as per the requirement.

Nitrite's module system is built on top of `NitritePlugin` and `NitriteModule` interfaces. A module is a collection of plugins. While opening a database, all the modules must be loaded. Each module then load and initialize all the plugins it contains.

The `nitrite` module provides default implementation of all the interfaces. For example, the `nitrite` module provides in-memory storage implementation. If you want to use on-disk storage, you need to add a storage module to your project.

The main advantage of Nitrite's module system is that it allows you to extend the functionality of Nitrite without adding all the dependencies at once in your project. You only need to add dependencies that you want to use. This helps to keep the application size small. 

You can write your own module and plugin and add it to the project. The module system also allows you to replace the default implementation of any plugin. For example, if you want to use a different storage engine, you can write your own storage module and add it to the project.

## NitriteModule

The `NitriteModule` interface is the base interface for all the modules. It encapsulates a set of plugins. A module must be loaded before opening a database. 

```dart
var module = MyModule();

var db = await Nitrite.builder()
    .loadModule(module)
    .openOrCreate();
```

To create a module, you need to implement the `NitriteModule` interface. The `NitriteModule` interface has a single getter `plugins` which returns a set of plugins. 

```dart
class MyModule implements NitriteModule {
  @override
  Set<NitritePlugin> get plugins => {MyPlugin()};
}
```

### Dynamic Module

A dynamic module can be created using the global `module()` method. A dynamic module is a module which is not loaded from any package, instead it is created dynamically. 

```dart
var module = module([MyPlugin()]);

var db = await Nitrite.builder()
    .loadModule(module)
    .openOrCreate();
```

A dynamic module is useful when you want to create a module from a set of plugins at runtime.

### Available Modules

The following modules are available from Nitrite.

- [Storage Module](store-modules/store-modules.md)
- [Spatial Module](spatial.md)

## NitritePlugin

The `NitritePlugin` interface is the base interface for all the plugins. A plugin is a single unit of functionality. A plugin must be loaded via a `NitriteModule` before opening a database. 

```dart
var plugin = MyPlugin();
var module = MyModule(plugin);
var db = await Nitrite.builder()
    .loadModule(module)
    .openOrCreate();
```

### Initializing Plugin

A plugin can be initialized by implementing the `initialize()` method of the `NitritePlugin` interface. The `initialize()` method is called by the module when the plugin is loaded. 

```dart
class MyPlugin implements NitritePlugin {
  @override
  Future<void> initialize(NitriteConfig nitriteConfig) async {
    // do something
  }
}
```

### Closing Plugin

A plugin can be closed by implementing the `close()` method of the `NitritePlugin` interface. The `close()` method is called by the module when the database is closed. 

```dart
class MyPlugin implements NitritePlugin {
  @override
  Future<void> close() async {
    // do something
  }
}
```

### Available Plugins

Following are the available plugins in Nitrite which can be used to extend the functionality of Nitrite:

- `NitriteStore` - A plugin to provide storage functionality.
- `NitriteMapper` - A plugin to provide object mapping functionality.
- `NitriteIndexer` - A plugin to provide indexing functionality.
- `EntityConverter` - A plugin to provide entity conversion functionality.

You can implement any of the above plugins to extend the functionality of Nitrite.

## NitriteStore

The `NitriteStore` interface is the base interface for all the storage plugins. A storage plugin is responsible for storing and retrieving data from the underlying storage. The `NitriteStore` interface extends the `NitritePlugin` interface. 

```dart
class MyStore implements NitriteStore {
    // implement the methods  
}
```

A reference implementation of the `NitriteStore` interface can be found [here](store-modules/custom.md).

## NitriteMapper

The `NitriteMapper` interface is the base interface for all the object mapping plugins. A mapper plugin is responsible for mapping an object to a document and vice-versa. The `NitriteMapper` interface extends the `NitritePlugin` interface. 

```dart
class MyMapper extends NitriteMapper {
    // implement the methods
}
```

## NitriteIndexer

The `NitriteIndexer` interface is the base interface for all the indexing plugins. An indexer plugin is responsible for indexing a document. The `NitriteIndexer` interface extends the `NitritePlugin` interface. 

```dart
class CustomIndexer implements NitriteIndexer {
    static final String customIndex = 'custom';

    @override
    String get indexType => customIndex;

    // implement the methods
}
```

While creating a new index, Nitrite uses the `indexType` getter to determine the type of the index. The `indexType` getter must return a unique string for each type of index. So when you create a new index, you need to pass the same string to the `IndexOptions.indexType`.

```dart
var index = await collection.createIndex(['myField'], indexOptions(CustomIndexer.customIndex);
```

## EntityConverter

An `EntityConverter` is a plugin which is responsible for converting an object to a document and vice-versa. The `EntityConverter` interface extends the `NitritePlugin` interface. 

If Nitrite is using the default `SimpleNitriteMapper`, then all `EntityConverter`s from all the loaded modules are automatically registered with the mapper. If you are using a custom mapper, then you need to register the `EntityConverter` with the mapper manually.

One of the available `EntityConverter` in Nitrite is `GeometryConverter` from the spatial module. It is responsible for converting a `Geometry` object of JTS to a document and vice-versa. Once you load the spatial module, the `GeometryConverter` is automatically registered with the mapper. You can find more information about `GeometryConverter` [here](spatial.md#geometryconverter).

If you want to load multiple `EntityConverter`s, while opening the database, you can also create a dynamic module and load it instead of using the `NitriteBuilder.registerEntityConverter()` method multiple times.

```dart
var module = module([MyConverter(), MyOtherConverter()]);

var db = await Nitrite.builder()
    .loadModule(module)
    .openOrCreate();
```