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

## Creating an Index

You can create an index on a collection using `createIndex()` method. There are several overloaded version of `createIndex()` method. You can create an index on a single field or multiple fields.

### Creating a Unique Index

