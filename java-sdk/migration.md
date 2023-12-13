---
label: Schema Migration
icon: versions
order: 10
---

A migration is a set of changes to the schema that can be applied to a database. Migrations are used to keep the database schema up to date with the codebase. It contains a queue if instructions that are executed in order to update the database schema from one version to the next. 

!!!info Important
The migration is executed only once. If you want to execute the migration again, you need to change the schema version of the database.
!!!

For example, if you have a database with the schema version 1 and you want to update it to version 2, you need to apply the migration steps from version 1 and version 2.

```java
// Create a migration

Migration migration = new Migration(1, 2) {
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

// Apply the migration
Nitrite db = Nitrite.builder()
    .loadModule(storeModule)
    .schemaVersion(1)
    .addMigrations(migration)
    .openOrCreate("test-user", "test-password");

```

The migration instructions are executed in the order they are added to the migration. The order of the instructions is important. For example, if you want to rename a field, you need to add the `renameField` instruction before the `deleteField` instruction. Otherwise, the `deleteField` instruction will fail.

!!!warning Important
Once a migration is applied to a database, it cannot be reverted. If you want to revert a migration, you need to create a new migration that will revert the changes.
!!!

Nitrite provides a set of instructions that can be used to update the database schema. These instructions can be grouped into three categories:

- Database Instruction
- Collection Instruction
- Repository Instruction


## Database Instruction

Database instructions are used perform operations on the database. The following instructions are available:

### Add Password

Adds an instruction to set a user authentication to the database.The user will be used to open the database.

```java
instruction.forDatabase()
    .addUser("test-user", "test-password");
```

### Change Password

Adds an instruction to change the password for the user authentication to the database.

```java
instruction.forDatabase()
    .changePassword("test-user", "test-password", "new-password");
```

### Drop Collection

Adds an instruction to drop a `NitriteCollection` from the database.

```java
instruction.forDatabase()
    .dropCollection("collection-name");
```

### Drop Repository

Adds an instruction to drop a `ObjectRepository` from the database. There are several ways to drop a repository:

#### Drop by Class

```java
instruction.forDatabase()
    .dropRepository(OldClass.class);
```

#### Drop by Class and Key Name

```java
instruction.forDatabase()
    .dropRepository(OldClass.class, "demo1");
```

#### Drop by EntityDecorator

```java
instruction.forDatabase()
    .dropRepository(new OldClassEntityDecorator<OldClass>());
```

#### Drop by EntityDecorator and Key Name

```java
instruction.forDatabase()
    .dropRepository(new OldClassEntityDecorator<OldClass>(), "demo1");
```

#### Drop by Repository Type Name

```java
instruction.forDatabase()
    .dropRepository("com.example.OldClass");
```

#### Drop by Repository Type Name and Key Name

```java
instruction.forDatabase()
    .dropRepository("com.example.OldClass", "demo1");
```

### Custom Instruction

Adds a custom instruction to perform a user defined operation on the database.

```java
instruction.forDatabase()
    .customInstruction(nitrite -> {
        // do something with the database
    });
```

## Collection Instruction

Collection instructions are used perform operations on a `NitriteCollection`. The following instructions are available:

### Rename Collection

Adds an instruction to rename a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .rename("new-collection-name");
```

### Add Field

Adds an instruction to add new field to the documents of a `NitriteCollection`.

#### Add Field with Null Value

```java
instruction.forCollection("collection-name")
    .addField("new-field");
```

The new field will be added to all the documents of the collection. The value of the new field will be `null`.

#### Add Field with Default Value

```java
instruction.forCollection("collection-name")
    .addField("new-field", "new-value");
```

The new field will be added to all the documents of the collection. The value of the new field will be `new-value`.

#### Add Field with Generator

```java
instruction.forCollection("collection-name")
    .addField("new-field", document -> {
        // do something with the document
        return "new-value";
    });
```

The new field will be added to all the documents of the collection. The value of the new field will be the value returned by the `Generator`.

#### Generator

A `Generator` is a functional interface that takes a `Document` as input and returns a value. The value returned by the `Generator` function will be used as the value of the new field.

```java
Generator<String> generator = document -> {
    // do something with the document
    return "new-value";
};
```

### Rename Field

Adds an instruction to rename a field of the documents of a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .renameField("old-field", "new-field");
```

### Delete Field

Adds an instruction to delete a field from the documents of a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .deleteField("field-name");
```

### Drop Index

Adds an instruction to drop an index from a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .dropIndex("field-name-1", "field-name-2");
```

The drop index instruction can be used to drop a single field index or a compound index.

### Drop All Indexes

Adds an instruction to drop all the indexes from a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .dropAllIndices();
```

### Create Index

Adds an instruction to create an index on a `NitriteCollection`.

```java
instruction.forCollection("collection-name")
    .createIndex(IndexType.NON_UNIQUE, "field-name-1", "field-name-2");
```

The create index instruction can be used to create a single field index or a compound index.

## Repository Instruction

Repository instructions are used perform operations on a `ObjectRepository`. The following instructions are available:

### Rename Repository

Adds an instruction to rename a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .renameRepository("com.example.NewClass", null);
```

### Add Field

Adds an instruction to add new field to the entity of a `ObjectRepository`.

#### Add Field with Null Value

```java
instruction.forRepository(OldClass.class, "demo1")
    .addField("new-field");
```

The new field will be added to all the entities of the repository. The value of the new field will be `null`.

#### Add Field with Default Value

```java
instruction.forRepository(OldClass.class, "demo1")
    .addField("new-field", 0);
```

The new field will be added to all the entities of the repository. The value of the new field will be `0`.

#### Add Field with Generator Function

```java
instruction.forRepository(OldClass.class, "demo1")
    .addField("new-field", document -> {
        // do something with the document
        return "new-value";
    });
```

The new field will be added to all the entities of the repository. The value of the new field will be the value returned by the `Generator` function.

More information about the `Generator` can be found [here](#generator).

### Rename Field

Adds an instruction to rename a field of the entity of a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .renameField("old-field", "new-field");
```

### Delete Field

Adds an instruction to delete a field from the entity of a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .deleteField("field-name");
```

### Change Data Type

Adds an instruction to change the data type of a field of the entity of a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .changeDataType("field-name", (TypeConverter<String, Long>) Long::parseLong);
```

#### Type Converter

A `TypeConverter` is a functional interface that takes a value of one type as input and returns a value of another type. The value returned by the `TypeConverter` function will be used as the value of the new field.

```java
// convert String to Long
TypeConverter<String, Long> converter = Long::parseLong;

// convert Float to Integer
TypeConverter<Float, Integer> converter = Math::round;
```

### Change Id Field

Adds an instruction to change the id field of the entity of a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .changeIdField(Fields.withNames("uuid"), Fields.withNames("empId"));
```

### Drop Index

Adds an instruction to drop an index from a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .dropIndex("field-name-1", "field-name-2");
```

The drop index instruction can be used to drop a single field index or a compound index.

### Drop All Indexes

Adds an instruction to drop all the indexes from a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .dropAllIndices();
```

### Create Index

Adds an instruction to create an index on a `ObjectRepository`.

```java
instruction.forRepository(OldClass.class, "demo1")
    .createIndex(IndexType.NON_UNIQUE, "field-name-1", "field-name-2");
```

The create index instruction can be used to create a single field index or a compound index.









