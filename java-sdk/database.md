---
icon: database
label: Nitrite Database
order: 19
---

## Creating a Database

Nitrite database can be created in-memory or on-disk. By default, Nitrite database is created in-memory. To create a database on-disk, you need to add a storage module dependency to your project.
More details about storage modules can be found [here](modules/store-modules/store-modules.md).

To create a database, you need to use `NitriteBuilder` class. To get an instance of `NitriteBuilder`, you need to call `builder()` method on `Nitrite` class.

```java
NitriteBuilder builder = Nitrite.builder();
```

### In-memory database

The below code snippet shows how to create a database in-memory.

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();
```

### On-disk database

The below code snippet shows how to create a database on-disk using H2's MVStore.

#### MVStore backed database

```java
MVStoreModule storeModule = MVStoreModule.withConfig()
    .filePath("/path/to/my.db")
    .build();

Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

More details about MVStore configuration can be found [here](modules/store-modules/mvstore/).

#### RocksDB backed database

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

### Open or create a database

To open or create a database, you need to call `openOrCreate()` method on `NitriteBuilder` instance. This method returns a `Nitrite` instance.

If no `StoreModule` is configured, then Nitrite will create an in-memory database. If a `StoreModule` is configured, then Nitrite will create a file-based database.

If the database file does not exist, then Nitrite will create a new database file. If the database file already exists, then Nitrite will open the existing database file.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

### Securing a database

To secure a database, you need to call `openOrCreate()` method with username and password on `NitriteBuilder` instance. This method returns a `Nitrite` instance with the given username and password.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate("user", "password");
```

!!!warning Warning
If you are using a file-based database, then you need to use the same username and password to open the database again. Otherwise, you will get a `NitriteSecurityException`.

Both username and password must be provided or both must be null.
!!!

### Load a Module

Nitrite database is modular in nature. It provides various modules to extend its functionality. To load a module, you need to call `loadModule()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

#### Loading a storage module

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .openOrCreate();
```

#### Loading a Jackson based mapper module

```java
Nitrite db = Nitrite.builder()
    .loadModule(new JacksonMapperModule())
    .openOrCreate();
```

More on the Nitrite's module system can be found [here](/java-sdk/modules/module-system/).

### Add Migration steps

Nitrite database supports schema migration. To configure a migration step, you need to call `addMigrations()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

```java
Migration migration = new Migration(Constants.INITIAL_SCHEMA_VERSION, 2) {
    @Override
    public void migrate(InstructionSet instruction) {
        instruction.forDatabase()
            .addPassword("test-user", "test-password");

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

### Current Schema version

To configure the current schema version, you need to call `schemaVersion()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .schemaVersion(2)
    .openOrCreate();
```

!!!primary Important
By default, the initial schema version is set to `1`.
!!!

### Field separator character

To configure the field separator character, you need to call `fieldSeparator()` method on `NitriteBuilder` instance. This method returns the same `NitriteBuilder` instance.

It is used to separate field names in a nested document. For example, if a document has a field <b>address</b> which is a nested document, then the field <b>street</b> of the nested document can be accessed using <b>address.street</b> syntax.

```java
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .fieldSeparator('.')
    .openOrCreate();
```

!!!primary Important
The default field separator character is set to `.`.
!!!
