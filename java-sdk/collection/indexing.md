---
label: Indexing
icon: list-ordered
order: 14
---

Indexing is a way to optimize the performance of a database by minimizing the number of disk accesses required when a query is processed. It is a data structure technique which is used to quickly locate and access the data in a database.

Nitrite supports indexing on a collection. It supports indexing on a single field or multiple fields. It also supports full-text indexing.

## Index types

Nitrite supports the following types of index:

- Unique Index
- Non-Unique Index
- Full-text Index

### Unique Index

A unique index ensures that the indexed field contains unique value. It does not allow duplicate value in the indexed field. It also ensures that the indexed field is not `null`.

### Non-Unique Index

A non-unique index does not ensure that the indexed field contains unique value. It allows duplicate value in the indexed field.

### Full-text Index

A full-text index is used to search text content in a document. It is useful when you want to search text content in a document. It is also useful when you want to search text content in a document in a language other than English.

!!!primary Note
- Document's `_id` field is always indexed.
- Indexing on non-comparable value is not supported.
!!!

## Creating an index

You can create an index on a collection using `createIndex()` method. There are several overloaded version of `createIndex()` method. You can create an index on a single field or multiple fields.

### Creating a unique index

You can create a unique index on a single field or multiple fields. It takes the name of the fields on which the index will be created as input parameter.

```java
// create a unique index on a single field
collection.createIndex("firstName");

// create a unique index on multiple fields
collection.createIndex("firstName", "lastName");
```

### Creating a non-unique index

You can create a non-unique index on a single field or multiple fields by passing the index type as `IndexType.NON_UNIQUE` in `IndexOptions` object and the name of the fields on which the index will be created as input parameters.

```java
// create a non-unique index on a single field
collection.createIndex(IndexOptions.indexOptions(IndexType.UNIQUE), "firstName");

// create a non-unique index on multiple fields
collection.createIndex(IndexOptions.indexOptions(IndexType.NON_UNIQUE), "firstName", "lastName");
```

### Creating a full-text index

You can create a non-unique index on a single field or multiple fields by passing the index type as `IndexType.FULL_TEXT` in `IndexOptions` object and the name of the fields on which the index will be created as input parameters.

```java
// create a full-text index on a single field
collection.createIndex(IndexOptions.indexOptions(IndexType.FULL_TEXT), "firstName");

```

!!!warning
Full-text index is not supported on multiple fields.
!!!

### Creating index on array field

Nitrite supports creating index on array field. It will create index on each element of the array. For example, if you have a document like this:

```java
Document document = Document.createDocument("firstName", "John")
    .put("lastName", "Doe")
    .put("age", 30)
    .put("address", "123 Street")
    .put("phones", new String[]{"1234567890", "0987654321"});
```

You can create index on `phones` field like this:

```java
// create unique index on array field
collection.createIndex("phones");

// create non-unique index on array field
collection.createIndex(IndexOptions.indexOptions(IndexType.NON_UNIQUE), "phones");
```

### Creating index on nested field

You can create index on nested field. For example, if you have a document like this:

```java
Document document = Document.createDocument("firstName", "John")
    .put("lastName", "Doe")
    .put("age", 30)
    .put("phones", new String[]{"1234567890", "0987654321"})
    .put("address", Document.createDocument("street", "123 Street")
        .put("city", "New York")
        .put("state", "NY")
        .put("zip", "10021"));
```

You can create a unique index on `street` field like this:

```java
// create unique index on nested field
collection.createIndex("address.street");
```

!!!warning
You cannot create index on nested field if the parent field is an array.
!!!

## Rebuilding an index

You can rebuild an index on a collection using `rebuildIndex()` method. It takes the name of the fields on which the index will be rebuilt as input parameter.

```java
// rebuild an index on a single field
collection.rebuildIndex("firstName");

// rebuild an index on multiple fields
collection.rebuildIndex("firstName", "lastName");
```

## Dropping an index

You can drop an index on a collection using `dropIndex()` method. It takes the name of the fields on which the index will be dropped as input parameter.

```java
// drop an index on a single field
collection.dropIndex("firstName");

// drop an index on multiple fields
collection.dropIndex("firstName", "lastName");
```

## Dropping all indexes

You can drop all indexes on a collection using `dropAllIndices()` method.

```java
collection.dropAllIndices();
```

## Getting all indexes

You can get all indexes on a collection using `listIndices()` method. It returns a `Collection` of `IndexDescriptor` object.

```java
Collection<IndexDescriptor> indexes = collection.listIndices();
```

### IndexDescriptor

`IndexDescriptor` is a simple class which contains the following information about an index:

- `collectionName`: The name of the collection on which the index is created.
- `indexType`: The type of the index.
- `fields`: A `Fields` object containing the name of the fields on which the index is created.

## Checking if an index exists

You can check if an index exists on a collection using `hasIndex()` method. It takes the name of the fields on which the index will be checked as input parameter.

```java
// check if an index exists on a single field
boolean exists = collection.hasIndex("firstName");

// check if an index exists on multiple fields
boolean exists = collection.hasIndex("firstName", "lastName");
```

## Error scenarios

The following error scenarios are possible while creating an index:

- If another index of any type is already created on the collection on the same field(s), then it will throw `IndexingException`.
- If a unique index is created on a field and the field contains duplicate value, then it will throw `UniqueConstraintException`.
- If a full-text index is created on multiple fields, then it will throw `IndexingException`.
- If a full-text index is created on a field which is not a `String`, then it will throw `IndexingException`.
- If you try to drop an index which does not exist, then it will throw `IndexingException`.
- If you try to rebuild an index which does not exist, then it will throw `IndexingException`.

