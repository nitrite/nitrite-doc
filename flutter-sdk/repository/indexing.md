---
label: Indexing
icon: list-ordered
order: 11
---

Indexing is a way to optimize the performance of a database by minimizing the number of disk accesses required when a query is processed. It is a data structure technique which is used to quickly locate and access the data in a database.

Nitrite supports indexing on a repository. It supports indexing on a single field or multiple fields. It also supports full-text indexing.

Indexes for an entity can be defined using various annotations like `@Id`, `@Index` etc. More information about annotations can be found [here](entity.md#annotations). It can also be managed using various methods of `ObjectRepository` interface.

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

A full-text index is used to search text content in an entity. It is useful when you want to search text content in an entity. It is also useful when you want to search text content in an entity in a language other than English.

!!!primary
Indexing on non-comparable value is not supported.
!!!

### Custom Index

You can also create your own custom index. More information about custom index can be found [here](../collection/indexing.md#custom-index).

## Creating an Index

You can define indexes for an entity using annotations. You can also create indexes using `ObjectRepository` interface.

!!!warning
You cannot create a new index on a field or a set of fields which is already indexed with a different type of index.

For example, if you have a unique index on `firstName` field, then you cannot create a non-unique index on `firstName` field. But you can create a unique index on `firstName` and `lastName` fields.
Similarly, if you have a unique index on `firstName` and `lastName` fields, then you cannot create a non-unique index on `firstName` and `lastName` fields. But you can create two indexes on `firstName` and `lastName` fields separately.
!!!

### Creating a Unique Index

You can create a unique index on a single field or multiple fields. It takes the name of the fields on which the index will be created as input parameter.

#### Using Annotations

You can create a unique index on a single field or multiple fields using annotations. You can use `@Id` annotation to create a unique index on a single field. You can use `@Index` annotation to create a unique index on multiple fields.

```dart
@Entity(indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['description'], type: IndexType.fullText),
  Index(fields: ['price', 'publisher']),
])
class Book {
  @Id()
  int id;
  String title;
  String description;
  String publisher;
  List<String> tags;
  double price;

  Book(this.title, this.description, this.publisher, this.tags, this.price);
}
```

#### Using ObjectRepository

You can create a unique index on a single field or multiple fields using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter.

```dart
await bookRepository.createIndex(['tags']);
await bookRepository.createIndex(['price', 'publisher']);
```

### Creating a Non-unique Index

You can create a non-unique index on a single field or multiple fields by passing the index type as `IndexType.nonUnique`.

#### Using Annotations

You can create a non-unique index on a single field or multiple fields using annotations. You can use `@Index` annotation to create a non-unique index on multiple fields.

```dart
@Entity(indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['price', 'publisher'], type: IndexType.nonUnique),
])
class Book {
  @Id()
  int id;
  String title;
  String description;
  String publisher;
  List<String> tags;
  double price;

  Book(this.title, this.description, this.publisher, this.tags, this.price);
}
```

#### Using ObjectRepository

You can create a non-unique index on a single field or multiple fields using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter and the index type as `IndexType.nonUnique`.

```dart
await bookRepository.createIndex(['tags'], indexOptions(IndexType.nonUnique));
await bookRepository.createIndex(['price', 'publisher'], indexOptions(IndexType.nonUnique));
```

### Creating a Full-text Index

You can create a full-text index on a single field or multiple fields by passing the index type as `IndexType.fullText`.

#### Using Annotations

You can create a full-text index on a single field using annotations. You can use `@Index` annotation to create a full-text index on a single field.

```dart
@Entity(indices: [
  Index(fields: ['description'], type: IndexType.fullText),
])
class Book {
  @Id()
  int id;
  String title;
  String description;
  String publisher;
  List<String> tags;
  double price;

  Book(this.title, this.description, this.publisher, this.tags, this.price);
}
```

#### Using ObjectRepository

You can create a full-text index on a single field using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter and the index type as `IndexType.fullText`.

```dart
await bookRepository.createIndex(['description'], indexOptions(IndexType.fullText));
```

!!!primary
Full-text index is not supported on multiple fields.
!!!

### Creating Index on Iterable Field

Nitrite supports creating index on iterable field. It will create index on each element of the iterable. For example, if you have an entity like this:

```dart
@Entity(indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
])
class Book {
  @Id()
  int id;
  String title;
  String description;
  String publisher;
  List<String> tags;
  double price;

  Book(this.title, this.description, this.publisher, this.tags, this.price);
}
```

Then it will create index on each element of the `tags` list. You can also create index on `tags` field like this:

```dart
await bookRepository.createIndex(['tags'], indexOptions(IndexType.nonUnique));
```

### Creating Index on Embedded Field

Nitrite supports creating index on embedded field. For example, if you have entities like this:

```dart
@Entity(indices: [
  Index(fields: ['address.city'], type: IndexType.nonUnique),
])
class Person {
  @Id()
  int id;
  String name;
  Address address;

  Person(this.name, this.address);
}

class Address {
  String city;
  String state;
  String country;

  Address(this.city, this.state, this.country);
}
```

Then it will create index on `city` field of `Address` entity. You can also create index on `city` field of `Address` entity like this:

```dart
await personRepository.createIndex(['address.city'], indexOptions(IndexType.nonUnique));
```

!!!primary
You cannot create index on nested field if the parent field is an array.
!!!

## Rebuilding an Index

You can rebuild an index using `rebuildIndex()` method. It takes the name of the fields on which the index will be rebuilt as input parameter.

```dart
// rebuild index on a single field
await bookRepository.rebuildIndex(['tags']);

// rebuild index on multiple fields
await bookRepository.rebuildIndex(['price', 'publisher']);
```

## Dropping an Index

You can drop an index using `dropIndex()` method. It takes the name of the fields on which the index will be dropped as input parameter.

```dart
// drop index on a single field
await bookRepository.dropIndex(['tags']);

// drop index on multiple fields
await bookRepository.dropIndex(['price', 'publisher']);
```

## Dropping All Indexes

You can drop all indexes using `dropAllIndices()` method.

```dart
await bookRepository.dropAllIndices();
```

## Getting All Indexes

You can get all indexes using `listIndices()` method. It returns a `Future<Collection>` of `IndexDescriptor` object.

```dart
Collection<IndexDescriptor> indexes = await bookRepository.listIndices();
```

### IndexDescriptor

`IndexDescriptor` is a class which contains information about an index. It contains the following information:

- `collectionName`: The name of the collection on which the index is created.
- `indexType`: The type of the index.
- `fields`: A `Fields` object containing the name of the fields on which the index is created.

## Checking If an Index Exists

You can check if an index exists using `hasIndex()` method. It takes the name of the fields on which the index will be checked as input parameter.

```dart
// check if an index exists on a single field
bool exists = await bookRepository.hasIndex(['tags']);

// check if an index exists on multiple fields
bool exists = await bookRepository.hasIndex(['price', 'publisher']);
```

## Error Scenarios

The following error scenarios are possible while creating an index:

- If another index of any type is already created on the repository on the same field(s), then it will throw `IndexingException`.
- If a unique index is created on a field and the field contains duplicate value, then it will throw `UniqueConstraintException`.
- If a full-text index is created on multiple fields, then it will throw `IndexingException`.
- If a full-text index is created on a field which is not a `String`, then it will throw `IndexingException`.
- If you try to drop an index which does not exist, then it will throw `IndexingException`.
- If you try to rebuild an index which does not exist, then it will throw `IndexingException`.
