---
label: Custom Storage Modules
icon: check-circle
order: 1
---

The beauty of Nitrite is that it is highly extensible. You can write your own storage module and use it with Nitrite. Nitrite provides a simple interface `StoreModule` to write your own storage module.

## StoreModule Interface

To write a custom storage module, you need to implement the `StoreModule` interface. The interface has only one method `getStore()` which returns an implementation of `NitriteStore` interface.

Optionally, you can also implement the `StoreConfig` interface to provide configuration options for your storage module.

### NitriteStore Interface

The `NitriteStore` is the storage abstraction layer for Nitrite. It provides the basic operations required by Nitrite to store and retrieve data. A `NitriteStore` is responsible for managing `NitriteMap` and `NitriteRTree` instances which are the main building blocks of database collections.

### StoreConfig Interface

The `StoreConfig` interface provides configuration options for your storage module. You can use this interface to provide configuration options for your storage module.

### NitriteMap Interface

The `NitriteMap` interface is the abstraction layer for Nitrite's key-value store. It provides the basic operations required by a collection to store and retrieve data. A `NitriteMap` is responsible for managing `NitriteId` and `Document` instances which are the main unit of data storage in Nitrite.

### NitriteRTree Interface

The `NitriteRTree` interface is the abstraction layer for Nitrite's R-Tree. It provides the basic operations required by a spatial index to store and retrieve data.

## Example

Let's write a simple storage module which stores data in a `Map` object. The module will be a simple in-memory storage module. The module will be configured with a `Map` object and will use it to store data.

### Store Module

The following code snippet shows the implementation of the storage module.

```dart
import 'package:nitrite/nitrite.dart';

class InMemoryStoreModule extends StoreModule {
  InMemoryConfig _storeConfig = InMemoryConfig();

  static InMemoryModuleBuilder withConfig() => InMemoryModuleBuilder._();

  @override
  NitriteStore<Config> getStore<Config extends StoreConfig>() {
    var store = InMemoryStore(_storeConfig);
    return store as NitriteStore<Config>;
  }

  @override
  Set<NitritePlugin> get plugins => <NitritePlugin>{getStore()};
}
```

### Store Config

The following code snippet shows the implementation of the store configuration.

```dart
class InMemoryConfig extends StoreConfig {
  Set<StoreEventListener> _eventListeners = <StoreEventListener>{};

  @override
  void addStoreEventListener(StoreEventListener listener) {
    _eventListeners.add(listener);
  }

  @override
  String? get filePath => null;

  @override
  bool get isReadOnly => false;

  @override
  Set<StoreEventListener> get eventListeners =>
      Set.unmodifiable(_eventListeners);
}
```

And the builder class for the configuration.

```dart
class InMemoryModuleBuilder {
  final Set<StoreEventListener> _eventListeners = <StoreEventListener>{};
  final InMemoryConfig _dbConfig = InMemoryConfig();

  InMemoryModuleBuilder._();

  /// Adds a [StoreEventListener] to the in-memory module builder.
  InMemoryModuleBuilder addStoreEventListener(StoreEventListener listener) {
    _eventListeners.add(listener);
    return this;
  }

  /// Builds an in-memory store module.
  InMemoryStoreModule build() {
    var module = InMemoryStoreModule();
    _dbConfig._eventListeners = _eventListeners;
    module._storeConfig = _dbConfig;
    return module;
  }
}
```

### Nitrite Store

The following code snippet shows the implementation of the `NitriteStore` interface.

```dart
class InMemoryStore extends AbstractNitriteStore<InMemoryConfig> {
  final Map<String, dynamic> _nitriteMapRegistry;
  final Map<String, dynamic> _nitriteRTreeMapRegistry;
  bool _closed = false;

  InMemoryStore(super._storeConfig)
      : _nitriteMapRegistry = <String, dynamic>{},
        _nitriteRTreeMapRegistry = <String, dynamic>{};

  @override
  bool get isClosed => _closed;

  @override
  Future<bool> get hasUnsavedChanges async => false;

  @override
  bool get isReadOnly => false;

  @override
  String get storeVersion => 'InMemory/$nitriteVersion';

  @override
  Future<void> openOrCreate() async {
    _initEventBus();
    alert(StoreEvents.opened);
  }

  @override
  Future<void> close() async {
    // close all maps and rtree
    close(MapEntry entry) => entry.value.close();
    _closed = true;

    // to avoid concurrent modification exception
    var tempMap = Map.from(_nitriteMapRegistry);
    tempMap.entries.forEach(close);

    tempMap = Map.from(_nitriteRTreeMapRegistry);
    tempMap.entries.forEach(close);

    _nitriteMapRegistry.clear();
    _nitriteRTreeMapRegistry.clear();
    await super.close();
  }

  @override
  Future<void> commit() async {
    alert(StoreEvents.commit);
  }

  @override
  Future<bool> hasMap(String mapName) async {
    return _nitriteMapRegistry.containsKey(mapName) ||
        _nitriteRTreeMapRegistry.containsKey(mapName);
  }

  @override
  Future<NitriteMap<Key, Value>> openMap<Key, Value>(String mapName) async {
    if (_nitriteMapRegistry.containsKey(mapName)) {
      var nitriteMap = _nitriteMapRegistry[mapName]!;
      if (nitriteMap.isClosed) {
        _nitriteMapRegistry.remove(mapName);
      } else {
        return nitriteMap as InMemoryMap<Key, Value>;
      }
    }

    var nitriteMap = InMemoryMap<Key, Value>(mapName, this);
    _nitriteMapRegistry[mapName] = nitriteMap;
    return nitriteMap;
  }

  @override
  Future<NitriteRTree<Key, Value>> openRTree<Key extends BoundingBox, Value>(
      String rTreeName) async {
    if (_nitriteRTreeMapRegistry.containsKey(rTreeName)) {
      return _nitriteRTreeMapRegistry[rTreeName]! as InMemoryRTree<Key, Value>;
    }

    var nitriteRTree = InMemoryRTree<Key, Value>(rTreeName, this);
    _nitriteRTreeMapRegistry[rTreeName] = nitriteRTree;
    return nitriteRTree;
  }

  @override
  Future<void> closeMap(String mapName) async {
    _nitriteMapRegistry.remove(mapName);
  }

  @override
  Future<void> closeRTree(String rTreeName) async {
    _nitriteRTreeMapRegistry.remove(rTreeName);
  }

  @override
  Future<void> removeMap(String mapName) async {
    if (_nitriteMapRegistry.containsKey(mapName)) {
      var map = _nitriteMapRegistry[mapName]!;

      if (!map.isClosed && !map.isDropped) {
        await map.close();
      }

      _nitriteMapRegistry.remove(mapName);
      var catalog = await getCatalog();
      await catalog.remove(mapName);
    }
  }

  @override
  Future<void> removeRTree(String rTreeName) async {
    if (_nitriteRTreeMapRegistry.containsKey(rTreeName)) {
      var rTree = _nitriteRTreeMapRegistry[rTreeName]!;
      await rTree.close();
      _nitriteRTreeMapRegistry.remove(rTreeName);
      var catalog = await getCatalog();
      await catalog.remove(rTreeName);
    }
  }

  void _initEventBus() {
    if (storeConfig!.eventListeners.isNotEmpty) {
      for (var listener in storeConfig!.eventListeners) {
        subscribe(listener);
      }
    }
  }
}
```

### Nitrite Map

The following code snippet shows the implementation of the `NitriteMap` interface.

```dart
class InMemoryMap<Key, Value> extends NitriteMap<Key, Value> {
  final SplayTreeMap<Key, Value> _backingMap;
  final NitriteStore _nitriteStore;
  final String _mapName;

  bool _droppedFlag = false;
  bool _closedFlag = false;

  InMemoryMap(this._mapName, this._nitriteStore)
      : _backingMap = SplayTreeMap<Key, Value>(_comp);

  @override
  String get name => _mapName;

  @override
  Future<bool> containsKey(Key key) async {
    _checkOpened();
    return _backingMap.containsKey(key);
  }

  @override
  Future<Value?> operator [](Key key) async {
    _checkOpened();
    return _backingMap[key];
  }

  @override
  NitriteStore<Config> getStore<Config extends StoreConfig>() {
    return _nitriteStore as NitriteStore<Config>;
  }

  @override
  Stream<Value> values() {
    _checkOpened();
    return Stream.fromIterable(_backingMap.values);
  }

  @override
  Future<Value?> remove(Key key) async {
    _checkOpened();
    var val = _backingMap.remove(key);
    await updateLastModifiedTime();
    return val;
  }

  @override
  Stream<Key> keys() {
    _checkOpened();
    return Stream.fromIterable(_backingMap.keys);
  }

  @override
  Future<void> put(Key key, Value value) {
    _checkOpened();
    _backingMap[key] = value;
    return updateLastModifiedTime();
  }

  @override
  Future<int> size() async {
    _checkOpened();
    return _backingMap.length;
  }

  @override
  Future<Value?> putIfAbsent(Key key, Value value) async {
    _checkOpened();
    if (await containsKey(key)) {
      return _backingMap[key];
    } else {
      _backingMap[key] = value;
      await updateLastModifiedTime();
      return null;
    }
  }

  @override
  Stream<(Key, Value)> entries() {
    _checkOpened();
    return Stream.fromIterable(
        _backingMap.entries.map((e) => (e.key, e.value)));
  }

  @override
  Stream<(Key, Value)> reversedEntries() {
    _checkOpened();
    return Stream.fromIterable(
        _backingMap.reversedEntries.map((e) => (e.key, e.value)));
  }

  @override
  Future<Key?> higherKey(Key key) async {
    _checkOpened();
    if (key == null) return null;
    return _backingMap.higherKey(key);
  }

  @override
  Future<Key?> ceilingKey(Key key) async {
    _checkOpened();
    if (key == null) return null;
    return _backingMap.ceilingKey(key);
  }

  @override
  Future<Key?> lowerKey(Key key) async {
    _checkOpened();
    if (key == null) return null;
    return _backingMap.lowerKey(key);
  }

  @override
  Future<Key?> floorKey(Key key) async {
    _checkOpened();
    if (key == null) return null;
    return _backingMap.floorKey(key);
  }

  @override
  Future<bool> isEmpty() async {
    _checkOpened();
    return _backingMap.isEmpty;
  }

  @override
  Future<void> drop() async {
    if (!_droppedFlag) {
      _backingMap.clear();
      await getStore().removeMap(_mapName);
      _droppedFlag = true;
      _closedFlag = true;
    }
  }

  @override
  bool get isDropped => _droppedFlag;

  @override
  Future<void> close() async {
    _closedFlag = true;
    await _nitriteStore.closeMap(_mapName);
  }

  @override
  bool get isClosed => _closedFlag;

  @override
  Future<void> clear() async {
    _checkOpened();
    _backingMap.clear();
    await _nitriteStore.closeMap(_mapName);
    await updateLastModifiedTime();
  }

  static int _comp(k1, k2) {
    if (k1 is Comparable && k2 is Comparable) {
      return compare(k1, k2);
    }
    return Comparable.compare(k1, k2);
  }

  void _checkOpened() {
    if (_closedFlag) {
      throw InvalidOperationException('Map $_mapName is closed');
    }
    if (_droppedFlag) {
      throw InvalidOperationException('Map $_mapName is dropped');
    }
  }

  @override
  Future<void> initialize() async {}
}
```

### Nitrite RTree

The following code snippet shows the implementation of the `NitriteRTree` interface.

```dart
class InMemoryRTree<Key extends BoundingBox, Value>
    extends NitriteRTree<Key, Value> {
  final Map<SpatialKey, Key?> _backingMap = <SpatialKey, Key?>{};
  final NitriteStore _nitriteStore;
  final String _mapName;

  bool _droppedFlag = false;
  bool _closedFlag = false;

  InMemoryRTree(this._mapName, this._nitriteStore);

  @override
  Future<int> size() async {
    _checkOpened();
    return _backingMap.length;
  }

  @override
  Future<void> add(Key? key, NitriteId? value) async {
    _checkOpened();
    if (value != null) {
      var spatialKey = _getKey(key, int.parse(value.idValue));
      _backingMap[spatialKey] = key;
    }
  }

  @override
  Future<void> remove(Key? key, NitriteId? value) async {
    _checkOpened();
    if (value != null) {
      var spatialKey = _getKey(key, int.parse(value.idValue));
      _backingMap.remove(spatialKey);
    }
  }

  @override
  Stream<NitriteId> findIntersectingKeys(Key? key) async* {
    _checkOpened();
    var spatialKey = _getKey(key, 0);

    for (var sk in _backingMap.keys) {
      if (_isOverlap(sk, spatialKey)) {
        yield NitriteId.createId(sk.id.toString());
      }
    }
  }

  @override
  Stream<NitriteId> findContainedKeys(Key? key) async* {
    _checkOpened();
    var spatialKey = _getKey(key, 0);

    for (var sk in _backingMap.keys) {
      if (_isInside(sk, spatialKey)) {
        yield NitriteId.createId(sk.id.toString());
      }
    }
  }

  @override
  Future<void> clear() async {
    _checkOpened();
    _backingMap.clear();
    await _nitriteStore.closeRTree(_mapName);
  }

  @override
  Future<void> close() async {
    _closedFlag = true;
    await _nitriteStore.closeRTree(_mapName);
  }

  @override
  Future<void> drop() async {
    _checkOpened();
    _droppedFlag = true;
    _backingMap.clear();
    await _nitriteStore.removeRTree(_mapName);
  }

  SpatialKey _getKey(Key? key, int id) {
    if (key == null || key == BoundingBox.empty) {
      return SpatialKey(id, []);
    }
    return SpatialKey(id, [key.minX, key.maxX, key.minY, key.maxY]);
  }

  bool _isOverlap(SpatialKey a, SpatialKey b) {
    if (a.isNull() || b.isNull()) {
      return false;
    }

    for (var i = 0; i < 2; i++) {
      if (a.max(i) < b.min(i) || a.min(i) > b.max(i)) {
        return false;
      }
    }

    return true;
  }

  bool _isInside(SpatialKey a, SpatialKey b) {
    if (a.isNull() || b.isNull()) {
      return false;
    }

    for (var i = 0; i < 2; i++) {
      if (a.min(i) <= b.min(i) || a.max(i) >= b.max(i)) {
        return false;
      }
    }

    return true;
  }

  void _checkOpened() {
    if (_closedFlag) {
      throw NitriteException('RTreeMap is closed');
    }
    if (_droppedFlag) {
      throw NitriteException('RTreeMap is dropped');
    }
  }

  @override
  Future<void> initialize() async {}
}
```
