---
label: RocksDB Module
icon: check-circle
order: 2
---

Nitrite provides a persistent storage module based on [RocksDB](https://rocksdb.org/). RocksDB is a persistent key-value store for fast storage. It is based on a log-structured merge-tree (LSM tree) data structure. It is very fast and lightweight. Nitrite uses the JNI wrapper of RocksDB. 

!!!info
RocksDB's column family feature is used to store the data of each collection.
!!!

## Adding RocksDB Module

Add RocksDB module to your project:

### Maven

Add the RocksDB dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dizitart</groupId>
    <artifactId>nitrite-rocksdb-adapter</artifactId>
</dependency>
```

### Gradle

Add the RocksDB dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation 'org.dizitart:nitrite-rocksdb-adapter'
}
```

## Using RocksDB Module

To use RocksDB as your persistent storage, you need to load the `RocksDBModule` while opening the database. You can also configure the module as per your requirement.

Once the module is loaded, you can use the database as usual. All the data will be persisted in the file you specified.

```java
// configure the module
RocksDBModule storeModule = RocksDBModule.withConfig()
            .filePath(filePath)
            .build();

Nitrite db = Nitrite.builder()
            .loadModule(storeModule)
            .openOrCreate();
```

## Configuring RocksDB Module

You can configure the RocksDB module as per your requirement. The following configuration options are available:

- `filePath` - the file path where the database will be stored.
- `options` - the RocksDB's `Options` object. You can configure RocksDB as per your requirement. The default value is `null`.
- `dbOptions` - the RocksDB's `DBOptions` object. You can configure RocksDB as per your requirement. The default value is `null`.
- `columnFamilyOptions` - the RocksDB's `ColumnFamilyOptions` object. You can configure RocksDB as per your requirement. The default value is `null`.
- `objectFormatter` - the `ObjectFormatter` to be used to serialize/deserialize objects. The default implementation is `KryoObjectFormatter` which is based on [Kryo](https://github.com/EsotericSoftware/kryo) serializer.

## Object Serialization

RocksDB is a key-value store database. It stores data in byte array format. To store objects in RocksDB, Nitrite uses a serializer. Nitrite uses `ObjectFormatter` to serialize/deserialize objects. The default implementation is `KryoObjectFormatter` which is based on [Kryo](https://github.com/EsotericSoftware/kryo) serializer. You can also provide your own implementation of `ObjectFormatter` to serialize/deserialize objects.

### Implementing ObjectFormatter

To implement your own `ObjectFormatter`, you need to implement the `ObjectFormatter` interface.

```java
public class MyObjectFormatter implements ObjectFormatter {
    @Override
    public <T> byte[] encode(T object) {
        // your implementation
    }

    @Override
    public <T> byte[] encodeKey(T object) {
        // your implementation
    }

    @Override
    public <T> T decode(byte[] bytes, Class<T> type) {
        // your implementation
    }

    @Override
    public <T> T decodeKey(byte[] bytes, Class<T> type) {
        // your implementation
    }
}
```

Once you have implemented the `ObjectFormatter`, you can use it while configuring the RocksDB module.

```java
// configure the module
RocksDBModule storeModule = RocksDBModule.withConfig()
            .filePath(filePath)
            .objectFormatter(new MyObjectFormatter())
            .build();

Nitrite db = Nitrite.builder()
            .loadModule(storeModule)
            .openOrCreate();
```

Normally, you don't need to implement your own `ObjectFormatter`. The default implementation is good enough for most of the use cases.





