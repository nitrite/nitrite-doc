---
label: Schema Migration
icon: versions
order: 10
---

A migration is a set of changes to the schema that can be applied to a database. Migrations are used to keep the database schema up to date with the codebase. It contains a queue of instructions that are executed in order to update the database schema from one version to the next. 

!!!primary
The migration is executed only once for a specific schema version.ni If you want to execute the migration again, you need to change the schema version of the database.
!!!

For example, if you have a database with the schema version 1 and you want to update it to version 2, you need to apply the migration steps from version 1 and version 2.

```dart
// create a migration

var migration = Migration(1, 2, (instructionSet) {
   instructionSet
        .forDatabase()
        .addUser('test-user', 'test-password');

   instructionSet
        .forRepository<OldClass>(key: 'demo1')
        .renameRepository('new')
        .changeDataType('empId', (value) => int.parse(value))
        .changeIdField(Fields.withNames(['uuid']), Fields.withNames(['empId']))
        .deleteField('uuid')
        .renameField('lastName', 'familyName')
        .addField('fullName', generator: (document) => '${document['firstName']} ${document['familyName']}')
        .dropIndex(['firstName'])
        .dropIndex(['literature.text'])
        .changeDataType('literature.ratings', (value) => (value as double).round());
    },
);

// apply the migration
var db = Nitrite.builder()
    .loadModule(storeModule)
    .schemaVersion(1)
    .addMigration(migration)
    .openOrCreate('test-user', 'test-password');
``` 

The migration instructions are executed in the order they are added to the migration. The order of the instructions is important. For example, if you want to rename a field, you need to add the `renameField` instruction before the `deleteField` instruction. Otherwise, the `deleteField` instruction will fail.

!!!warning :zap: Warning :zap:
Once a migration is applied to a database, it cannot be reverted. If you want to revert a migration, you need to create a new migration that will revert the changes.
!!!

Nitrite provides a set of instructions that can be used to update the database schema. These instructions can be grouped into three categories:

- Database Instruction
- Collection Instruction
- Repository Instruction

## Database Instruction

Database instructions are used to perform operations on the database. The following instructions are available:

### Add User

Adds an instruction to set a user authentication to the database. The user will be used to open the database.

```dart
instruction.forDatabase()
    .addUser("test-user", "test-password");
```

!!!danger :zap: Important :zap:
Nitrite database supports only one user authentication per database. If you try to add a new user when there is already a user authentication present, the migration will fail throwing a `NitriteSecurityException`.
!!!

### Change Password

Adds an instruction to change the password for the user authentication to the database.

```dart
instruction.forDatabase()
    .changePassword("test-user", "test-password", "new-password");
```

!!!danger :zap: Important :zap:
The user authentication must be present in the database. If you try to change the password for a user that does not exist, the migration will fail throwing a `NitriteSecurityException`.

If you try to change the password for a user with a wrong password, the migration will also fail throwing a `NitriteSecurityException`.
!!!

### Drop Collection

Adds an instruction to drop a `NitriteCollection` from the database.

```dart
instruction.forDatabase()
    .dropCollection("collection-name");
```

### Drop Repository

Adds an instruction to drop a `ObjectRepository` from the database. There are several ways to drop a repository:

#### Drop by Type

```dart
instruction.forDatabase()
    .dropRepository<OldClass>();
```

#### Drop by Type and Key Name

```dart
instruction.forDatabase()
    .dropRepository<OldClass>(key: "demo1");
```

#### Drop by EntityDecorator
    
```dart
instruction.forDatabase()
    .dropRepository(entityDecorator: OldClassEntityDecorator());
```

#### Drop by EntityDecorator and Key Name

```dart
instruction.forDatabase()
    .dropRepository(entityDecorator: OldClassEntityDecorator(),  key: "demo1");
```

### Drop by Entity Name

```dart
instruction.forDatabase()
    .dropRepository(entityName: "OldClass");
```

### Drop by Entity Name and Key Name

```dart
instruction.forDatabase()
    .dropRepository(entityName: "OldClass", key: "demo1");
```

### Custom Instruction

Adds a custom instruction to the migration. The custom instruction is a callback function that takes a `Nitrite` instance as an argument. The callback function can be used to perform any operation on the database.

```dart
instruction.forDatabase()
    .customInstruction((db) {
        // perform any operation on the database
    });
```

## Collection Instruction

Collection instructions are used to perform operations on a `NitriteCollection`. The following instructions are available:

### Rename Collection

Adds an instruction to rename a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .rename("new-collection-name");
```

### Add Field

Adds an instruction to add new field to the documents of a `NitriteCollection`.

#### Add Field with Null Value

```dart
instruction.forCollection("collection-name")
    .addField("new-field");
```

The new field will be added to all the documents of the collection. The value of the new field will be `null`.

#### Add Field with Default Value

```dart
instruction.forCollection("collection-name")
    .addField("new-field", defaultValue: "new-value");
```

The new field will be added to all the documents of the collection. The value of the new field will be `new-value`.

#### Add Field with Generator

```dart
instruction.forCollection("collection-name")
    .addField("new-field", generator: (document) => "new-value");
```

The new field will be added to all the documents of the collection. The value of the new field will be the value returned by the `Generator`.

#### Generator

A `Generator` is a function that takes a `Document` as input and returns a value. The value returned by the `Generator` will be used as the value of the new field.

```dart
Generator generator = (document) => "new-value";
```

### Rename Field

Adds an instruction to rename a field of the documents of a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .renameField("old-field", "new-field");
```

### Delete Field

Adds an instruction to delete a field from the documents of a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .deleteField("field-name");
```

### Drop Index

Adds an instruction to drop an index from a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .dropIndex(["field-name-1", "field-name-2"]);
```

The drop index instruction can be used to drop a single field index or a compound index.

### Drop All Indexes

Adds an instruction to drop all the indexes from a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .dropAllIndices();
```

### Create Index

Adds an instruction to create an index on a `NitriteCollection`.

```dart
instruction.forCollection("collection-name")
    .createIndex(IndexType.nonUnique, ["field-name-1", "field-name-2"]);
```

The create index instruction can be used to create a single field index or a compound index.

## Repository Instruction

Repository instructions are used to perform operations on a `ObjectRepository`. The following instructions are available:

### Rename Repository

Adds an instruction to rename a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .renameRepository("newName");
```

### Add Field

Adds an instruction to add new field to the entity of a `ObjectRepository`.

#### Add Field with Null Value

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .addField("new-field");
```

The new field will be added to all the entities of the repository. The value of the new field will be `null`.

#### Add Field with Default Value

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .addField("new-field", defaultValue: "new-value");
```

The new field will be added to all the entities of the repository. The value of the new field will be `new-value`.

#### Add Field with Generator Function

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .addField("new-field", generator: (entity) => "new-value");
```

The new field will be added to all the entities of the repository. The value of the new field will be the value returned by the `Generator` function.

More information about the `Generator` can be found [here](#generator).

### Rename Field

Adds an instruction to rename a field of the entity of a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .renameField("old-field", "new-field");
```

### Delete Field

Adds an instruction to delete a field from the entity of a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .deleteField("field-name");
```

### Change Data Type

Adds an instruction to change the data type of a field of the entity of a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .changeDataType("field-name", (value) => int.parse(value));
```

#### Type Converter

A `TypeConverter` is a function that takes a value of one type as input and returns a value of another type. The value returned by the `TypeConverter` will be used as the value of the new field.

```dart
// convert String to int
TypeConverter typeConverter = (value) => int.parse(value);

// convert double to int
TypeConverter typeConverter = (value) => (value as double).round();
```

### Change Id Field

Adds an instruction to change the id field of the entity of a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .changeIdField(Fields.withNames(["uuid"]), Fields.withNames(["empId"]));
```

### Drop Index

Adds an instruction to drop an index from a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .dropIndex(["field-name-1", "field-name-2"]);
```

The drop index instruction can be used to drop a single field index or a compound index.

### Drop All Indexes

Adds an instruction to drop all the indexes from a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .dropAllIndices();
```

### Create Index

Adds an instruction to create an index on a `ObjectRepository`.

```dart
instruction.forRepository<OldClass>(key: "demo1")
    .createIndex(IndexType.nonUnique, ["field-name-1", "field-name-2"]);
```

The create index instruction can be used to create a single field index or a compound index.
