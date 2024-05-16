---
icon: database
label: Nitrite Database
order: 19
---

Nitrite database is a serverless, embedded, and self-contained Java NoSQL database. It is an open-source project that provides a simple API for persistent data storage. Nitrite database is designed to be lightweight, fast, and easy to use.

## Creating a Database

Nitrite database can be created in-memory or on-disk. By default, Nitrite database is created in-memory. To create a database on-disk, you need to add a storage module dependency to your project. More details about storage modules can be found [here](modules/store-modules/store-modules.md).

To create a database, you need to use `NitriteBuilder` class. To get an instance of `NitriteBuilder`, you need to call `builder()` method on `Nitrite` class.

```java
NitriteBuilder builder = Nitrite.builder();
```

### In-memory Database

If you don't load any on-disk storage module, then Nitrite will create an in-memory database. The below code snippet shows how to create a database in-memory.

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();
```

### On-disk Database

The below code snippet shows how to create a database on-disk.

#### MVStore Backed Database

```java
MVStoreModule storeModule = MVStoreModule.withConfig()
    .filePath("/path/to/my.db")
    .build();

Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

More details about MVStore configuration can be found [here](modules/store-modules/mvstore/).

#### RocksDB Backed Database

```java
RocksDBModule storeModule = RocksDBModule.withConfig()
    .filePath("/path/to/my.db")
    .build();

Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

More details about RocksDB configuration can be found [here](modules/store-modules/rocksdb/).

## NitriteBuilder

`NitriteBuilder` provides a fluent API to configure and create a Nitrite database instance.

### Open or Create a Database

To open or create a database, you need to call `openOrCreate()` method on `NitriteBuilder` instance. This method returns a `Nitrite` instance.

If no `StoreModule` is configured, then Nitrite will create an in-memory database. If a `StoreModule` is configured, then Nitrite will create a file-based database. If the database file does not exist, then Nitrite will create a new database file. If the database file already exists, then Nitrite will open the existing database file.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

### Securing a Database

To secure a database, you need to call `openOrCreate()` method with username and password on `NitriteBuilder` instance. This method returns a `Nitrite` instance with the given username and password.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate("user", "password");
```

!!!warning
If you are using a file-based database, then you need to use the same username and password to open the database again. Otherwise, you will get a `NitriteSecurityException`.

Both username and password must be provided or both must be null.
!!!

### Registering an EntityConverter

Nitrite database uses a mapper to map Java entities to Nitrite documents and vice-versa. By default, Nitrite uses `SimpleNitriteMapper` as its mapper. This mapper uses `EntityConverter`s to map Java entities to Nitrite documents and vice-versa. To register an `EntityConverter`, you need to call `registerEntityConverter()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .registerEntityConverter(new ProductConverter())
    .registerEntityConverter(new ProductIdConverter())
    .registerEntityConverter(new ManufacturerConverter())
    .openOrCreate();
```

More on `EntityConverter` can be found [here](repository/mapper.md#entityconverter).

### Loading a Module

Nitrite database is modular in nature. It provides various modules to extend its functionality. To load a module, you need to call `loadModule()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

#### Loading a Storage Module

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

#### Loading a Jackson Based Mapper Module

```java
Nitrite db = Nitrite.builder()
    .loadModule(new JacksonMapperModule())
    .openOrCreate();
```

More on the Nitrite's module system can be found [here](/java-sdk/modules/module-system/).

### Adding Migration Steps

Nitrite database supports schema migration. To configure a migration step, you need to call `addMigrations()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

```java
Migration migration = new Migration(Constants.INITIAL_SCHEMA_VERSION, 2) {
    @Override
    public void migrate(InstructionSet instruction) {
        instruction.forDatabase()
            .addUser("test-user", "test-password");

        instruction.forRepository(OldClass.class, "demo1")
            .renameRepository("new", null)
            .changeDataType("empId", (TypeConverter<String, Long>) Long::parseLong)
            .changeIdField(Fields.withNames("uuid"), Fields.withNames("empId"))
            .deleteField("uuid")
            .renameField("lastName", "familyName")
            .addField("fullName", document -> document.get("firstName", String.class) + " "
                + document.get("familyName", String.class))
            .dropIndex("firstName")
            .dropIndex("literature.text")
            .changeDataType("literature.ratings", (TypeConverter<Float, Integer>) Math::round);
    }
};

Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .addMigrations(migration)
    .openOrCreate();
```

More on the schema migration can be found [here](migration.md).

### Current Schema Version

To configure the current schema version, you need to call `schemaVersion()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .schemaVersion(2)
    .openOrCreate();
```

!!!info
By default, the initial schema version is set to `1`.
!!!

### Field Separator Character

To configure the field separator character, you need to call `fieldSeparator()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

It is used to separate field names in a nested document. For example, if a document has a field <b>address</b> which is a nested document, then the field <b>street</b> of the nested document can be accessed using <b>address.street</b> syntax.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .fieldSeparator('.')
    .openOrCreate();
```

!!!info
The default field separator character is set to `.`.
!!!

### Disable Repository Type Validation

To disable repository type validation, you need to call `disableRepositoryTypeValidation()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

Repository type validation is a feature in Nitrite that ensures the type of the objects stored in the repository can be converted to and from the document. By default, the repository type validation is enabled. If you disable it,
and if you try to store an object that cannot be converted to a document, Nitrite will throw an exception during the operation.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .disableRepositoryTypeValidation()
    .openOrCreate();
```