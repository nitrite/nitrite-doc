---
icon: database
label: Nitrite Database
order: 19
---

Nitrite database is a serverless, embedded, and self-contained Java NoSQL database. It is an open-source project that provides a simple API for persistent data storage. Nitrite database is designed to be lightweight, fast, and easy to use.

## Creating a Database

Nitrite database can be created in-memory or on-disk. By default, Nitrite database is created in-memory. To create a database on-disk, you need to add a storage module dependency to your project. More details about storage modules can be found [here](../java-sdk/modules/store-modules/store-modules.md).

To create a database, you need to use `nitrite()` function. It is a builder function that returns an instance of `Nitrite`.

```kotlin
var db = nitrite {
    // database configuration
}
```

### In-memory Database

If you don't load any on-disk storage module, then Nitrite will create an in-memory database. The below code snippet shows how to create a database in-memory.

```kotlin
var db = nitrite()
```

### On-disk Database

The below code snippet shows how to create a database on-disk.

#### MVStore Backed Database

```kotlin
val storeModule = MVStoreModule.withConfig()
    .filePath("/path/to/my.db")
    .build()

var db = nitrite {
    loadModule(storeModule)
}
```

More details about MVStore configuration can be found [here](../java-sdk/modules/store-modules/mvstore.md).

#### RocksDB Backed Database

```kotlin
val storeModule = RocksDBModule.withConfig()
    .filePath("/path/to/my.db")
    .build()

var db = nitrite {
    loadModule(storeModule)
}
```

More details about RocksDB configuration can be found [here](../java-sdk/modules/store-modules/rocksdb.md).

### Securing a Database

To secure a database, you need to pass a username and password to the `nitrite()` function. The below code snippet shows how to secure a database.

```kotlin
var db = nitrite("user", "password") {
    // database configuration
}
```

!!!warning
If you are using a file-based database, then you need to use the same username and password to open the database again. Otherwise, you will get a `NitriteSecurityException`.

Both username and password must be provided or both must be null.
!!!

### Registering an EntityConverter

Nitrite database uses a mapper to map Kotlin/Java entities to Nitrite documents and vice-versa. By default, Nitrite uses `SimpleNitriteMapper` as its mapper. This mapper uses `EntityConverter`s to map entities to Nitrite documents and vice-versa. To register an `EntityConverter`, you need to call `registerEntityConverter()` method on `Builder` instance. 

```kotlin
var db = nitrite {
    registerEntityConverter(MyEntityConverter())
}
```

More details about `EntityConverter` can be found [here](../java-sdk/repository/mapper.md#entityconverter).

### Loading a Nitrite Module

Nitrite database is modular in nature. It provides a set of modules to extend its functionality. To load a module, you need to call `loadModule()` method on `Builder` instance.

```kotlin
var db = nitrite {
    loadModule(MyModule())
}
```

More details about Nitrite modules can be found [here](../java-sdk/modules/module-system.md).

#### Loading a Storage Module

```kotlin
var db = nitrite {
    loadModule(MVStoreModule.withConfig()
        .filePath("/path/to/my.db")
        .build())
}
```

More details about storage modules can be found [here](../java-sdk/modules/store-modules/store-modules.md).

### Adding Migration Steps

Nitrite database supports migration from one version to another. To add a migration step, you need to call `addMigration()` method on `Builder` instance.

```kotlin
var db = nitrite {
    addMigration(object : Migration(Constants.INITIAL_SCHEMA_VERSION, 2) {
        override fun migrate(instruction: InstructionSet) {
            instruction.forDatabase()
                .dropCollection("test")
        }
    })
    loadModule(MVStoreModule(fileName))
    schemaVersion = 2
}
```

More details about schema migration can be found [here](../java-sdk/migration.md).

### Current Schema Version

To configure the current schema version, you need to set the `schemaVersion` property on `Builder` instance.

```kotlin
var db = nitrite {
    schemaVersion = 2
}
```

!!!info
By default, the initial schema version is set to `1`.
!!!

### Field Separator Character

Nitrite database uses `.` as the field separator character. It is used to separate nested fields in a document. For example, if you have a document like below:

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "address": {
        "street": "123 Main Street",
        "city": "New York",
        "state": "NY",
        "zip": "10001"
    }
}
```

Then you can access the `firstName` field like below:

```kotlin
val firstName = document["firstName"]
```

And you can access the `street` field like below:

```kotlin
val street = document["address.street"]
```

To configure the field separator character, you need to set the `fieldSeparator` property on `Builder` instance.

```kotlin
var db = nitrite {
    fieldSeparator = '.'
}
```

!!!info
By default, the field separator character is set to `.`.
!!!

### Disable Repository Type Validation

Nitrite database has a feature called repository type validation. It ensures the type of the objects stored in the repository can be converted to and from the document. By default, the repository type validation is enabled. If you disable it, and if you try to store an object that cannot be converted to a document, Nitrite will throw an exception during the operation.

To disable repository type validation, you need to set the `enableRepositoryValidation` property to `false` on `Builder` instance.

```kotlin
var db = nitrite {
    enableRepositoryValidation = false
}
```