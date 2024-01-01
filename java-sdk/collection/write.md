---
label: Write Operations
icon: pencil
order: 16
---

## Inserting Documents

Documents can be inserted into a collection using `insert()` method. It takes one or multiple `Document` objects as input parameter. It returns a `WriteResult` object.

If the document has a `NitriteId` already in it's `_id` field, then it will be used as a unique key to identify the document in the collection. Otherwise, a new `NitriteId` will be generated and inserted into the document.

If any of the field is already indexed in the collection, then the index will be updated accordingly.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Insert` event.
!!!

### Inserting a Single Document

```java
Document doc = Document.createDocument("firstName", "John")
    .put("lastName", "Doe")
    .put("age", 30)
    .put("data", new byte[] { 1, 2, 3, 4, 5 });

WriteResult result = collection.insert(doc);
```

### Inserting Multiple Documents

```java
Document doc1 = Document.createDocument("firstName", "John")
    .put("lastName", "Doe")
    .put("age", 30)
    .put("data", new byte[] { 1, 2, 3, 4, 5 });

Document doc2 = Document.createDocument("firstName", "Jane")
    .put("lastName", "Doe")
    .put("age", 25)
    .put("data", new byte[] { 1, 2, 3, 4, 5 });

WriteResult result = collection.insert(doc1, doc2);
```

### Error Scenarios

- If the document is `null`, then it will throw a `ValidationException`.
- If the document contains invalid value in it's `_id` field, then it will throw a `InvalidIdException`.
- If there is another document with the same `_id` value in the collection, then it will throw a `UniqueConstraintException`.
- If a field of the document is unique indexed and it's value violates the index constraint, then it will throw a `UniqueConstraintException`.

### WriteResult

`WriteResult` contains the result of a write operation. It contains the following information:

- Number of documents affected by the write operation. You can get this value using `getAffectedCount()` method.
- List of `NitriteId` of the documents affected by the write operation. The `WriteResults` implements `Iterable<NitriteId>` interface. So you can iterate over the `WriteResult` to get the `NitriteId` of the documents affected by the write operation.

## Updating Documents

Documents can be updated in a collection using `update()` method. There are several overloaded version of `update()` method. You can update a single document or multiple documents at a time. You can also update a document using a filter.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Update` event.
!!!

### Updating a Single Document

You can update a single document using `update()` method. It takes a `Document` object as input parameter. It returns a `WriteResult` object. The document must contain a valid `NitriteId` in it's `_id` field. The document must not be `null`.

```java
Document doc = Document.createDocument("_id", existingDoc.getId())
    .put("age", 30);

WriteResult result = collection.update(doc);
```

!!!warning
If the document does not contain a valid `NitriteId` in it's `_id` field, then it will throw a `NotIdentifiableException`.
!!!

### Upserting a Single Document

You can upsert a single document using `update()` method. It takes a `Document` object as the first input parameter. It takes a `boolean` value as the second input parameter. If the second input parameter is `true`, then it will insert the document if it does not exist in the collection. Otherwise, it will update the document if it exists in the collection. It returns a `WriteResult` object. The document must not be `null`.

```java
Document doc = Document.createDocument("_id", existingDoc.getId())
    .put("lastName", "Doe")
    .put("age", 30)
    .put("data", new byte[] { 1, 2, 3, 4, 5 });

WriteResult result = collection.update(doc, true);
```

### Updating Using a Filter

You can update multiple documents using a filter. It takes a `Filter` object as the first input parameter. It takes a `Document` object as the second input parameter. It returns a `WriteResult` object. The document must not be `null` or empty.

If the filter result matches multiple documents, then all the documents will be updated.

```java
Document update = Document.createDocument("_id", existingDoc.getId())
    .put("firstName", "Jane");

WriteResult result = collection.update(where("firstName").eq("John"), update);
```

### Updating Using a Filter and Options

You can update multiple documents using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `Document` object as the second input parameter. It takes a `UpdateOptions` object as the third input parameter. It returns a `WriteResult` object. The document must not be `null` or empty.

#### UpdateOptions

`UpdateOptions` is a class that contains several options for update operation. It has the following options:

- `insertIfAbsent`: If this option is `true`, then it will insert the document if it does not exist in the collection. Otherwise, it will update the document if it exists in the collection.
- `justOnce`: If this option is `true`, then it will update only the first document matched by the filter. Otherwise, it will update all the documents matched by the filter.

```java
Document update = Document.createDocument("_id", existingDoc.getId())
    .put("firstName", "Jane");

UpdateOptions updateOptions = new UpdateOptions();
updateOptions.insertIfAbsent(true);
updateOptions.justOnce(true);

WriteResult result = collection.update(where("firstName").eq("John"), update, updateOptions);
```

## Removing Documents

Documents can be removed from a collection using `remove()` method. There are several overloaded version of `remove()` method. You can remove a single document or multiple documents at a time using a filter.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Remove` event.
!!!

### Removing a Single Document

You can remove a single document using `remove()` method. It takes a `Document` object as input parameter. It returns a `WriteResult` object. The document must contain a valid `NitriteId` in it's `_id` field. The document must not be `null`.

```java
Document doc = Document.createDocument("_id", existingDoc.getId());

WriteResult result = collection.remove(doc);
```

!!!warning
If the document does not contain a valid `NitriteId` in it's `_id` field, then it will throw a `NotIdentifiableException`.
!!!

### Removing Using a Filter

You can remove multiple documents using a filter. It takes a `Filter` object as the input parameter. It returns a `WriteResult` object.

If the filter result matches multiple documents, then all the documents will be removed.

```java
WriteResult result = collection.remove(where("firstName").eq("John"));
```

### Removing Using a Filter and Options

You can remove multiple documents using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `boolean` value as the second input parameter. If the second input parameter is `true`, then it will remove only the first document matched by the filter. Otherwise, it will remove all the documents matched by the filter. It returns a `WriteResult` object.

```java
WriteResult result = collection.remove(where("firstName").eq("John"), true);
```