---
label: Indexing
icon: list-ordered
order: 14
---

Indexing is a way to optimize the performance of a database by minimizing the number of disk accesses required when a query is processed. It is a data structure technique which is used to quickly locate and access the data in a database.

Nitrite supports indexing on a collection. It supports indexing on a single field or multiple fields. It also supports full-text indexing.

## Index Types

Nitrite supports the following types of index out of the box:

- Unique Index
- Non-Unique Index
- Full-text Index

### Unique Index

A unique index ensures that the indexed field contains unique value. It does not allow duplicate value in the indexed field. It also ensures that the indexed field is not `null`.

### Non-unique Index

A non-unique index does not ensure that the indexed field contains unique value. It allows duplicate value in the indexed field.

### Full-text Index

A full-text index is used to search text content in a document. It is useful when you want to search text content in a document. It is also useful when you want to search text content in a document in a language other than English.

!!!info
- Document's `_id` field is always indexed.
- Indexing on non-comparable value is not supported.
!!!

### Custom Index

You can also create your own custom index. You need to implement `NitriteIndexer` interface to create your own custom index. `NitriteIndexer` is a [`NitritePlugin`](../modules/module-system.md#nitriteplugin), so you need to register it using `loadModule()` method while opening a database. During index creation you need to pass the type of the custom index in `IndexOptions` object.

One of such custom index implementation can be found in spatial module. It provides spatial indexing on a collection. More on spatial indexing can be found [here](../modules/spatial.md#spatial-index).

## Creating an Index

You can create an index on a collection using `createIndex()` method. There are several overloaded version of `createIndex()` method. You can create an index on a single field or multiple fields.

!!!warning
You cannot create a new index on a field or a set of fields which is already indexed with a different type of index.

For example, if you have a unique index on `firstName` field, then you cannot create a non-unique index on `firstName` field. But you can create a unique index on `firstName` and `lastName` fields.
Similarly, if you have a unique index on `firstName` and `lastName` fields, then you cannot create a non-unique index on `firstName` and `lastName` fields. But you can create two indexes on `firstName` and `lastName` fields separately.
!!!

### Creating a Unique Index

You can create a unique index on a single field or multiple fields. It takes the name of the fields on which the index will be created as input parameter.

```dart
// create a unique index on a single field
await collection.createIndex(["firstName"]);

// create a unique index on multiple fields
await collection.createIndex(["firstName", "lastName"]);
```

### Creating a Non-unique Index

You can create a non-unique index on a single field or multiple fields by passing the index type as `IndexType.nonUnique` in `IndexOptions` object and the name of the fields on which the index will be created as input parameters.

```dart
// create a non-unique index on a single field
await collection.createIndex(["firstName"], indexOptions(IndexType.nonUnique));

// create a non-unique index on multiple fields
await collection.createIndex(["firstName", "lastName"], indexOptions(IndexType.nonUnique));
```

### Creating a Full-text Index

You can create a full-text index on a single field by passing the index type as `IndexType.fullText` in `IndexOptions` object and the name of the fields on which the index will be created as input parameters.

```dart
// create a full-text index on a single field
await collection.createIndex(["firstName"], indexOptions(IndexType.fullText));
```

!!!warning
Full-text index is not supported on multiple fields.
!!!

### Creating Index on Array Field

Nitrite supports creating index on array field. It will create index on each element of the array. For example, if you have a document like this:

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "age": 30,
    "address": "123 Street",
    "phones": ["1234567890", "0987654321"]
}
```

You can create index on `phones` field like this:

```dart
// create unique index on array field
await collection.createIndex(["phones"]);

// create non-unique index on array field
await collection.createIndex(["phones"], indexOptions(IndexType.nonUnique));
```

### Creating Index on Nested Field

You can create index on nested field. For example, if you have a document like this:

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "age": 30,
    "phones": ["1234567890", "0987654321"],
    "address": {
        "street": "123 Street",
        "city": "New York",
        "state": "NY",
        "zip": "10021"
    }
}
```

You can create index on `street` field like this:

```dart
// create unique index on nested field
await collection.createIndex(["address.street"]);
```

!!!primary
You cannot create index on nested field if the parent field is an array.
!!!

## Rebuilding an Index

You can rebuild an index on a collection using `rebuildIndex()` method. It takes the name of the fields on which the index will be rebuilt as input parameter.

```dart
// rebuild index on a single field
await collection.rebuildIndex(["firstName"]);

// rebuild index on multiple fields
await collection.rebuildIndex(["firstName", "lastName"]);
```

## Dropping an Index

You can drop an index on a collection using `dropIndex()` method. It takes the name of the fields on which the index will be dropped as input parameter.

```dart
// drop index on a single field
await collection.dropIndex(["firstName"]);

// drop index on multiple fields
await collection.dropIndex(["firstName", "lastName"]);
```

## Dropping All Indexes

You can drop all indexes on a collection using `dropAllIndices()` method.

```dart
await collection.dropAllIndices();
```

## Getting All Indexes

You can get all indexes on a collection using `listIndices()` method. It returns a `Future<Collection>` of `IndexDescriptor` object.

```dart
Collection<IndexDescriptor> indexes = await collection.listIndices();
```

### IndexDescriptor

`IndexDescriptor` is a simple class which contains the following information about an index:

- `collectionName`: The name of the collection on which the index is created.
- `indexType`: The type of the index.
- `fields`: A `Fields` object containing the name of the fields on which the index is created.

## Checking If an Index Exists

You can check if an index exists on a collection using `hasIndex()` method. It takes the name of the fields on which the index will be checked as input parameter.

```dart
// check if an index exists on a single field
bool exists = await collection.hasIndex(["firstName"]);

// check if an index exists on multiple fields
bool exists = await collection.hasIndex(["firstName", "lastName"]);
```

## Error Scenarios

The following error scenarios are possible while creating an index:

- If another index of any type is already created on the collection on the same field(s), then it will throw `IndexingException`.
- If a unique index is created on a field and the field contains duplicate value, then it will throw `UniqueConstraintException`.
- If a full-text index is created on multiple fields, then it will throw `IndexingException`.
- If a full-text index is created on a field which is not a `String`, then it will throw `IndexingException`.
- If you try to drop an index which does not exist, then it will throw `IndexingException`.
- If you try to rebuild an index which does not exist, then it will throw `IndexingException`.