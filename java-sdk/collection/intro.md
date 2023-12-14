---
label: Introduction
icon: info
order: 17
---

`NitriteCollection` represents a named document collection stored in a Nitrite database. It persists documents in a Nitrite database. It is similar to a table in relational database or a collection in MongoDB.

Each document in a collection is associated with a unique `NitriteId`. It exposes a set of methods to perform CRUD operations on documents. It also supports indexing and querying. It also supports event based notification on document changes.

`NitriteCollection` is thread-safe and supports concurrent read and write operations.

## Creating a Collection

A `NitriteCollection` can be created using `Nitrite` class. You need to call `getCollection()` method on `Nitrite` class to get an instance of a `NitriteCollection`.

If the collection does not exist, then it will be created automatically. If a collection with the same name already exists, then it will return the existing collection. 

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

NitriteCollection collection = db.getCollection("myCollection");
```

## Limitations on Collection Name

A collection name cannot be `null` or empty string. It cannot contains any of the following characters:

- `|` (pipe)
- `:` (colon)
- `+` (plus)

The name also cannot be any of the following reserved words:

- $nitrite_users
- $nitrite_index_meta
- $nitrite_index
- $nitrite_meta_map
- $nitrite_store_info
- $nitrite_catalog