---
label: Indexing
icon: list-ordered
order: 12
---

Indexing is a way to optimize the performance of a database by minimizing the number of disk accesses required when a query is processed. It is a data structure technique which is used to quickly locate and access the data in a database.

Nitrite supports indexing on a repository. It supports indexing on a single field or multiple fields. It also supports full-text indexing.

Indexes for an entity can be defined using various annotations like `@Id`, `@Index`, `@Indices` etc. More information about annotations can be found [here](entity.md#annotations). It can also be managed using various methods of `ObjectRepository` interface.

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

### Creating a Unique Index

You can create a unique index on a single field or multiple fields. It takes the name of the fields on which the index will be created as input parameter.

#### Using Annotations

You can create a unique index on a single field or multiple fields using annotations. You can use `@Id` annotation to create a unique index on a single field. You can use `@Index` annotation to create a unique index on multiple fields.

```java
@Indices(
    @Index(value = "productName", type = IndexType.UNIQUE)
    @Index(value = {"category", "manufacturer"}, type = IndexType.UNIQUE)
)
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private String manufacturer;
    // getter/setter
}
```

#### Using ObjectRepository

You can create a unique index on a single field or multiple fields using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter.

```java
productRepository.createIndex("productName");
productRepository.createIndex("category", "manufacturer");
```

### Creating a Non-unique Index

You can create a non-unique index on a single field or multiple fields by passing the index type as `IndexType.NON_UNIQUE`.

#### Using Annotations

You can create a non-unique index on a single field or multiple fields using annotations. You can use `@Index` annotation to create a non-unique index on multiple fields.

```java
@Indices(
    @Index(value = "productName", type = IndexType.NON_UNIQUE)
    @Index(value = {"category", "price"}, type = IndexType.NON_UNIQUE)
)
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private String manufacturer;
    // getter/setter
}
```

#### Using ObjectRepository

You can create a non-unique index on a single field or multiple fields using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter and the index type as `IndexType.NON_UNIQUE`.

```java
productRepository.createIndex(IndexOptions.indexOptions(IndexType.NON_UNIQUE), "productName");
productRepository.createIndex(IndexOptions.indexOptions(IndexType.NON_UNIQUE), "category", "price");
```

### Creating a Full-text Index

You can create a full-text index on a single field by passing the index type as `IndexType.FULL_TEXT`.

#### Using Annotations

You can create a full-text index on a single field using annotations. You can use `@Index` annotation to create a full-text index on a single field.

```java
@Indices(
    @Index(value = "description", type = IndexType.FULL_TEXT)
)
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private String manufacturer;
    private String description;
    // getter/setter
}
```

#### Using ObjectRepository

You can create a full-text index on a single field using `createIndex()` method. It takes the name of the fields on which the index will be created as input parameter and the index type as `IndexType.FULL_TEXT`.

```java
productRepository.createIndex(IndexOptions.indexOptions(IndexType.FULL_TEXT), "description");
```

!!!primary
Full-text index is not supported on multiple fields.
!!!

### Creating Index on Array Field

Nitrite supports creating index on array field. It will create index on each element of the array. For example, if you have an entity like this:

```java
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private String manufacturer;
    private String[] tags;
    // getter/setter
}
```

You can create index on `tags` field like this:

```java
productRepository.createIndex(IndexOptions.indexOptions(IndexType.NON_UNIQUE), "tags");
```

### Creating Index on Embedded Field

Nitrite supports creating index on embedded field. For example, if you have entities like this:

```java
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private Manufacturer manufacturer;
    // getter/setter
}

public class Manufacturer {
    private String name;
    private String address;
    // getter/setter
}
```

You can create index on name of `Manufacturer` entity via annotation like this:

```java
@Indices(
    @Index(value = "manufacturer.name", type = IndexType.UNIQUE)
)
public class Product {
    @Id
    private String productId;
    private String productName;
    private Double price;
    private String category;
    private Manufacturer manufacturer;
    // getter/setter
}
```

Or you can create index on name of `Manufacturer` entity via `ObjectRepository` like this:

```java
productRepository.createIndex("manufacturer.name");
```

!!!primary
You cannot create index on nested field if the parent field is an array.
!!!

## Rebuilding an Index

You can rebuild an index on a repository using `rebuildIndex()` method. It takes the name of the fields on which the index will be rebuilt as input parameter.

```java
// rebuild an index on a single field
productRepository.rebuildIndex("productName");

// rebuild an index on multiple fields
productRepository.rebuildIndex("category", "manufacturer.name");
```

## Dropping an Index

You can drop an index on a repository using `dropIndex()` method. It takes the name of the fields on which the index will be dropped as input parameter.

```java
// drop an index on a single field
productRepository.dropIndex("productName");

// drop an index on multiple fields
productRepository.dropIndex("category", "manufacturer.name");
```

## Dropping All Indexes

You can drop all indexes on a repository using `dropAllIndices()` method.

```java
productRepository.dropAllIndices();
```

## Getting All Indexes

You can get all indexes on a repository using `listIndices()` method. It returns a `Collection` of `IndexDescriptor` object.

```java
Collection<IndexDescriptor> indexes = productRepository.listIndices();
```

### IndexDescriptor

`IndexDescriptor` is a simple class which contains the following information about an index:

- `collectionName`: The name of the collection on which the index is created.
- `indexType`: The type of the index.
- `fields`: A `Fields` object containing the name of the fields on which the index is created.

## Checking If an Index Exists

You can check if an index exists on a repository using `hasIndex()` method. It takes the name of the fields on which the index will be dropped as input parameter.

```java
// check if an index exists on a single field
boolean exists = productRepository.hasIndex("productName");

// check if an index exists on multiple fields
boolean exists = productRepository.hasIndex("category", "manufacturer.name");
```

## Error Scenarios

The following error scenarios are possible while creating an index:

- If another index of any type is already created on the repository on the same field(s), then it will throw `IndexingException`.
- If a unique index is created on a field and the field contains duplicate value, then it will throw `UniqueConstraintException`.
- If a full-text index is created on multiple fields, then it will throw `IndexingException`.
- If a full-text index is created on a field which is not a `String`, then it will throw `IndexingException`.
- If you try to drop an index which does not exist, then it will throw `IndexingException`.
- If you try to rebuild an index which does not exist, then it will throw `IndexingException`.

