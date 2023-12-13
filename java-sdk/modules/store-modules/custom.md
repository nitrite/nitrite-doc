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

```java
public class InMemoryStoreModule implements StoreModule {

    @Setter(AccessLevel.PACKAGE)
    private InMemoryConfig storeConfig;

    public InMemoryStoreModule() {
        this.storeConfig = new InMemoryConfig();
    }

    public static InMemoryModuleBuilder withConfig() {
        return new InMemoryModuleBuilder();
    }

    @Override
    public NitriteStore<?> getStore() {
        InMemoryStore store = new InMemoryStore();
        store.setStoreConfig(storeConfig);
        return store;
    }

    @Override
    public Set<NitritePlugin> plugins() {
        return setOf(getStore());
    }
}
```

### Store Config

The following code snippet shows the implementation of the store configuration.

```java
@Accessors(fluent = true)
public class InMemoryConfig implements StoreConfig {
    @Getter
    @Setter(AccessLevel.PACKAGE)
    private Set<StoreEventListener> eventListeners;

    public InMemoryConfig() {
        this.eventListeners = new HashSet<>();
    }

    @Override
    public final String filePath() {
        return null;
    }

    @Override
    public Boolean isReadOnly() {
        return false;
    }

    @Override
    public void addStoreEventListener(StoreEventListener listener) {
        eventListeners.add(listener);
    }
}
```

And the builder class for the configuration.

```java
@Getter
@Accessors(fluent = true)
public class InMemoryModuleBuilder {
    private final Set<StoreEventListener> eventListeners;
    private final InMemoryConfig dbConfig;

    public InMemoryModuleBuilder() {
        dbConfig = new InMemoryConfig();
        eventListeners = new HashSet<>();
    }

    public InMemoryModuleBuilder addStoreEventListener(StoreEventListener listener) {
        eventListeners.add(listener);
        return this;
    }

    public InMemoryStoreModule build() {
        InMemoryStoreModule module = new InMemoryStoreModule();
        dbConfig.eventListeners(eventListeners());
        module.setStoreConfig(dbConfig);
        return module;
    }
}
```

### Nitrite Store

The following code snippet shows the implementation of the `NitriteStore` interface.

```java
public final class InMemoryStore extends AbstractNitriteStore<InMemoryConfig> {
    private final Map<String, NitriteMap<?, ?>> nitriteMapRegistry;
    private final Map<String, NitriteRTree<?, ?>> nitriteRTreeMapRegistry;
    private volatile boolean closed = false;

    public InMemoryStore() {
        super();
        this.nitriteMapRegistry = new ConcurrentHashMap<>();
        this.nitriteRTreeMapRegistry = new ConcurrentHashMap<>();
    }

    @Override
    public void openOrCreate() {
        initEventBus();
        alert(StoreEvents.Opened);
    }

    @Override
    public boolean isClosed() {
        return closed;
    }

    @Override
    public boolean hasUnsavedChanges() {
        return false;
    }

    @Override
    public boolean isReadOnly() {
        return false;
    }

    @Override
    public void commit() {
        alert(StoreEvents.Commit);
    }

    @Override
    public void close() {
        closed = true;
        Consumer<Map.Entry<?, ?>> closeConsumer = entry -> {
            if (entry.getValue() instanceof AutoCloseable) {
                try {
                    ((AutoCloseable) entry.getValue()).close();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            }
        };

        nitriteMapRegistry.entrySet().forEach(closeConsumer);
        nitriteRTreeMapRegistry.entrySet().forEach(closeConsumer);

        nitriteMapRegistry.clear();
        nitriteRTreeMapRegistry.clear();
        super.close();
    }

    @Override
    public boolean hasMap(String mapName) {
        return nitriteMapRegistry.containsKey(mapName) || nitriteRTreeMapRegistry.containsKey(mapName);
    }

    @Override
    @SuppressWarnings("unchecked")
    public <Key, Value> NitriteMap<Key, Value> openMap(String mapName, Class<?> keyType, Class<?> valueType) {
        if (nitriteMapRegistry.containsKey(mapName)) {
            NitriteMap<Key, Value> nitriteMap = (NitriteMap<Key, Value>) nitriteMapRegistry.get(mapName);
            if (nitriteMap.isClosed()) {
                nitriteMapRegistry.remove(mapName);
            } else {
                return nitriteMap;
            }
        }

        NitriteMap<Key, Value> nitriteMap = new InMemoryMap<>(mapName, this);
        nitriteMapRegistry.put(mapName, nitriteMap);
        return nitriteMap;
    }

    @Override
    @SuppressWarnings("unchecked")
    public <Key extends BoundingBox, Value> NitriteRTree<Key, Value> openRTree(String rTreeName,
                                                                               Class<?> keyType,
                                                                               Class<?> valueType) {
        if (nitriteRTreeMapRegistry.containsKey(rTreeName)) {
            return (InMemoryRTree<Key, Value>) nitriteRTreeMapRegistry.get(rTreeName);
        }

        NitriteRTree<Key, Value> rTree = new InMemoryRTree<>(rTreeName, this);
        nitriteRTreeMapRegistry.put(rTreeName, rTree);

        return rTree;
    }

    @Override
    public void closeMap(String mapName) {
        nitriteMapRegistry.remove(mapName);
    }

    @Override
    public void closeRTree(String rTreeName) {
        nitriteRTreeMapRegistry.remove(rTreeName);
    }

    @Override
    public void removeMap(String mapName) {
        if (nitriteMapRegistry.containsKey(mapName)) {
            NitriteMap<?, ?> nitriteMap = nitriteMapRegistry.get(mapName);
            if (!nitriteMap.isClosed() && !nitriteMap.isDropped()) {
                nitriteMap.clear();
                nitriteMap.close();
            }
            nitriteMapRegistry.remove(mapName);
            getCatalog().remove(mapName);
        }
    }

    @Override
    public void removeRTree(String rTreeName) {
        if (nitriteRTreeMapRegistry.containsKey(rTreeName)) {
            NitriteRTree<?, ?> rTree = nitriteRTreeMapRegistry.get(rTreeName);
            rTree.close();
            nitriteRTreeMapRegistry.remove(rTreeName);
            getCatalog().remove(rTreeName);
        }
    }

    @Override
    public String getStoreVersion() {
        return "InMemory/" + NITRITE_VERSION;
    }

    private void initEventBus() {
        if (getStoreConfig().eventListeners() != null) {
            for (StoreEventListener eventListener : getStoreConfig().eventListeners()) {
                subscribe(eventListener);
            }
        }
    }
}
```

### Nitrite Map

The following code snippet shows the implementation of the `NitriteMap` interface.

```java
public class InMemoryMap<Key, Value> implements NitriteMap<Key, Value> {
    private final NavigableMap<Key, Value> backingMap;
    private final NitriteStore<?> nitriteStore;
    private final String mapName;
    private final AtomicBoolean droppedFlag;
    private final AtomicBoolean closedFlag;

    public InMemoryMap(String mapName, NitriteStore<?> nitriteStore) {
        this.mapName = mapName;
        this.nitriteStore = nitriteStore;
        this.backingMap = new ConcurrentSkipListMap<>((o1, o2) ->
            Comparables.compare((Comparable<?>) o1, (Comparable<?>) o2));

        this.closedFlag = new AtomicBoolean(false);
        this.droppedFlag = new AtomicBoolean(false);
    }

    @Override
    public boolean containsKey(Key key) {
        checkOpened();
        return backingMap.containsKey(key);
    }

    @Override
    public Value get(Key key) {
        checkOpened();
        return backingMap.get(key);
    }

    @Override
    public NitriteStore<?> getStore() {
        return nitriteStore;
    }

    @Override
    public String getName() {
        return mapName;
    }

    @Override
    public RecordStream<Value> values() {
        checkOpened();
        return RecordStream.fromIterable(backingMap.values());
    }

    @Override
    public Value remove(Key key) {
        checkOpened();
        Value value = backingMap.remove(key);
        updateLastModifiedTime();
        return value;
    }

    @Override
    public RecordStream<Key> keys() {
        checkOpened();
        return RecordStream.fromIterable(backingMap.keySet());
    }

    @Override
    public void put(Key key, Value value) {
        checkOpened();
        notNull(value, "value cannot be null");
        backingMap.put(key, value);
        updateLastModifiedTime();
    }

    @Override
    public long size() {
        checkOpened();
        return backingMap.size();
    }

    @Override
    public Value putIfAbsent(Key key, Value value) {
        checkOpened();
        notNull(value, "value cannot be null");

        Value v = get(key);
        if (v == null) {
            put(key, value);
            updateLastModifiedTime();
        }
        return v;
    }

    @Override
    public RecordStream<Pair<Key, Value>> entries() {
        checkOpened();
        return getStream(backingMap);
    }

    @Override
    public RecordStream<Pair<Key, Value>> reversedEntries() {
        checkOpened();
        return getStream(backingMap.descendingMap());
    }

    @Override
    public Key higherKey(Key key) {
        checkOpened();
        if (key == null) {
            return null;
        }
        return backingMap.higherKey(key);
    }

    @Override
    public Key ceilingKey(Key key) {
        checkOpened();
        if (key == null) {
            return null;
        }
        return backingMap.ceilingKey(key);
    }

    @Override
    public Key lowerKey(Key key) {
        checkOpened();
        if (key == null) {
            return null;
        }
        return backingMap.lowerKey(key);
    }

    @Override
    public Key floorKey(Key key) {
        checkOpened();
        if (key == null) {
            return null;
        }
        return backingMap.floorKey(key);
    }

    @Override
    public boolean isEmpty() {
        checkOpened();
        return backingMap.isEmpty();
    }

    @Override
    public void drop() {
        if (!droppedFlag.get()) {
            backingMap.clear();
            getStore().removeMap(mapName);
            droppedFlag.compareAndSet(false, true);
            closedFlag.compareAndSet(false, true);
        }
    }

    @Override
    public boolean isDropped() {
        return droppedFlag.get();
    }

    @Override
    public void close() {
        closedFlag.compareAndSet(false, true);
        getStore().closeMap(mapName);
    }

    @Override
    public boolean isClosed() {
        return closedFlag.get();
    }

    @Override
    public void clear() {
        checkOpened();
        backingMap.clear();
        getStore().closeMap(mapName);
        updateLastModifiedTime();
    }

    private RecordStream<Pair<Key, Value>> getStream(NavigableMap<Key, Value> primaryMap) {
        return RecordStream.fromIterable(() -> new Iterator<Pair<Key, Value>>() {
            private final Iterator<Map.Entry<Key, Value>> entryIterator =
                primaryMap.entrySet().iterator();
            @Override
            public boolean hasNext() {
                return entryIterator.hasNext();
            }

            @Override
            public Pair<Key, Value> next() {
                Map.Entry<Key, Value> entry = entryIterator.next();
                return new Pair<>(entry.getKey(), entry.getValue());
            }
        });
    }

    private void checkOpened() {
        if (closedFlag.get()) {
            throw new InvalidOperationException("Map " + mapName + " is closed");
        }

        if (droppedFlag.get()) {
            throw new InvalidOperationException("Map " + mapName + " is dropped");
        }
    }
}
```

### Nitrite RTree

The following code snippet shows the implementation of the `NitriteRTree` interface.

```java
public class InMemoryRTree<Key extends BoundingBox, Value> implements NitriteRTree<Key, Value> {
    private final Map<SpatialKey, Key> backingMap;
    private final AtomicBoolean droppedFlag;
    private final AtomicBoolean closedFlag;
    private final String mapName;
    private final NitriteStore<?> nitriteStore;

    public InMemoryRTree(String mapName, NitriteStore<?> nitriteStore) {
        this.backingMap = new ConcurrentHashMap<>();
        this.closedFlag = new AtomicBoolean(false);
        this.droppedFlag = new AtomicBoolean(false);
        this.mapName = mapName;
        this.nitriteStore = nitriteStore;
    }

    @Override
    public void add(Key key, NitriteId nitriteId) {
        checkOpened();
        if (nitriteId != null && nitriteId.getIdValue() != null) {
            SpatialKey spatialKey = getKey(key, Long.parseLong(nitriteId.getIdValue()));
            backingMap.put(spatialKey, key);
        }
    }

    @Override
    public void remove(Key key, NitriteId nitriteId) {
        checkOpened();
        if (nitriteId != null && nitriteId.getIdValue() != null) {
            SpatialKey spatialKey = getKey(key, Long.parseLong(nitriteId.getIdValue()));
            backingMap.remove(spatialKey);
        }
    }

    @Override
    public RecordStream<NitriteId> findIntersectingKeys(Key key) {
        checkOpened();
        SpatialKey spatialKey = getKey(key, 0L);
        Set<NitriteId> set = new HashSet<>();

        for (SpatialKey sk : backingMap.keySet()) {
            if (isOverlap(sk, spatialKey)) {
                set.add(NitriteId.createId(Long.toString(sk.getId())));
            }
        }

        return RecordStream.fromIterable(set);
    }

    @Override
    public RecordStream<NitriteId> findContainedKeys(Key key) {
        checkOpened();
        SpatialKey spatialKey = getKey(key, 0L);
        Set<NitriteId> set = new HashSet<>();

        for (SpatialKey sk : backingMap.keySet()) {
            if (isInside(sk, spatialKey)) {
                set.add(NitriteId.createId(Long.toString(sk.getId())));
            }
        }

        return RecordStream.fromIterable(set);
    }

    @Override
    public long size() {
        checkOpened();
        return backingMap.size();
    }

    @Override
    public void close() {
        closedFlag.compareAndSet(false, true);
    }

    @Override
    public void clear() {
        checkOpened();
        backingMap.clear();
    }

    @Override
    public void drop() {
        checkOpened();
        droppedFlag.compareAndSet(false, true);
        backingMap.clear();
        nitriteStore.removeRTree(mapName);
    }

    private SpatialKey getKey(Key key, long id) {
        return new SpatialKey(id, key.getMinX(),
            key.getMaxX(), key.getMinY(), key.getMaxY());
    }

    private boolean isOverlap(SpatialKey a, SpatialKey b) {
        if (a.isNull() || b.isNull()) {
            return false;
        }
        for (int i = 0; i < 2; i++) {
            if (a.max(i) < b.min(i) || a.min(i) > b.max(i)) {
                return false;
            }
        }
        return true;
    }

    private boolean isInside(SpatialKey a, SpatialKey b) {
        if (a.isNull() || b.isNull()) {
            return false;
        }
        for (int i = 0; i < 2; i++) {
            if (a.min(i) <= b.min(i) || a.max(i) >= b.max(i)) {
                return false;
            }
        }
        return true;
    }

    private void checkOpened() {
        if (closedFlag.get()) {
            throw new InvalidOperationException("RTreeMap is closed");
        }

        if (droppedFlag.get()) {
            throw new InvalidOperationException("RTreeMap is dropped");
        }
    }
}
```