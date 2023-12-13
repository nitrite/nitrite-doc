---
label: MVStore Modules
icon: check-circle
order: 3
---

Nitrite provides a persistent storage module based on [MVStore](https://www.h2database.com/html/mvstore.html). MVStore is a pure Java key-value store database. It is a single file based database. It is very fast and lightweight.

## Adding MVStore Module

Add MVStore module to your project:

### Maven

Add the MVStore dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dizitart</groupId>
    <artifactId>nitrite-mvstore-adapter</artifactId>
</dependency>
```

### Gradle

Add the MVStore dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation 'org.dizitart:nitrite-mvstore-adapter'
}
```

## Using MVStore Module

To use MVStore as your persistent storage, you need to load the `MVStoreModule` while opening the database. You can also configure the module as per your requirement.

Once the module is loaded, you can use the database as usual. All the data will be persisted in the file you specified.

```java
// configure the module
MVStoreModule storeModule = MVStoreModule.withConfig()
            .filePath(filePath)
            .compress(true)
            .build();

Nitrite db = Nitrite.builder()
            .loadModule(storeModule)
            .openOrCreate();
```

## Configuring MVStore Module

You can configure the MVStore module as per your requirement. The following configuration options are available:

- `filePath` - the file path where the database will be stored.
- `autoCommitBufferSize` - the size of the buffer used for auto-commit. When the buffer is full, the changes are automatically committed to the database. The default buffer size is 1024.
- `encryptionKey` - the encryption key used to encrypt the database. If not specified, the database will not be encrypted.
- `readOnly` - if set to `true`, the database will be opened in read-only mode. The default value is `false`.
- `compress` - if set to `true`, the database will be compressed. The default value is `false`.
- `compressHigh` - if set to `true`, the database will be compressed using high compression. The default value is `false`. It may result in slower read and write performance but smaller file size on disk.
- `autoCommit` - if set to `true`, all changes will be committed immediately. If set to false, changes will be buffered until the buffer is full or `Nitrite.commit()` is called. The default value is `true`.
- `recoveryMode` - if set to `true`, the database will be opened in recovery mode. The default value is `false`.
- `cacheSize` - the cache size in MB. The default value is 16 MB.
- `cacheConcurrency` - the read cache concurrency used by MVStore. Default is 16 segments.
- `pageSplitSize` - the amount of memory a MVStore page should contain at most, in bytes, before it is split. The default is 16 KB.
- `fileStore` - the file store used by MVStore.

Nitrite provides a builder pattern to configure the module. You can use the `MVStoreModule.withConfig()` method to get the builder instance. The builder has a `build()` method which returns the configured module.

```java
MVStoreModule storeModule = MVStoreModule.withConfig()
            .filePath(filePath)
            .compress(true)
            .build();
```

## Upgrading from 3.x

If you are upgrading from Nitrite 3.x, you need to use the MVStore module as your storage module if you want to automatically upgrade your database. Any other storage module will not be able to upgrade your database automatically; you need to write your own upgrade logic.

Nitrite 3.x uses MVStore version 1.4.x. Nitrite 4.x uses MVStore version 2.2.x. The MVStore version 2.2.x is not backward compatible with version 1.4.x. So, Nitrite 4.x will try to upgrade your database automatically when you open it for the first time on a [!badge variant="warning" text="best effort basis without any guarantee"]. If the upgrade fails, you need to write your own upgrade logic. However, it is recommended to take a backup of your database before upgrading.
