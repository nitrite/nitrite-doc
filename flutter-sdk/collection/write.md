---
label: Write Operations
icon: pencil
order: 16
---

## Inserting Documents

Documents can be inserted into a collection using `insert()` or `insertMany()` methods. It takes one or multiple `Document` objects as input parameter respectively. It returns a `Future<WriteResult>` object.

If the document has a `NitriteId` already in it's `_id` field, then it will be used as a unique key to identify the document in the collection. Otherwise, a new `NitriteId` will be generated and inserted into the document.

If any of the field is already indexed in the collection, then the index will be updated accordingly.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.insert` event.
!!!

### Inserting a Single Document

```dart
var doc = createDocument("firstName", "John")
    ..put("lastName", "Doe")
    ..put("age", 30)
    ..put("data", Uint8List.fromList([1, 2, 3, 4, 5]));

var result = await collection.insert(doc);
```

### Inserting Multiple Documents

```dart
var doc1 = createDocument("firstName", "John")
    ..put("lastName", "Doe")
    ..put("age", 30)
    ..put("data", Uint8List.fromList([1, 2, 3, 4, 5]));

var doc2 = createDocument("firstName", "Jane")
    ..put("lastName", "Doe")
    ..put("age", 25)
    ..put("data", Uint8List.fromList([1, 2, 3, 4, 5]));

var result = await collection.insertMany([doc1, doc2]);
```

### Error Scenarios

- If the document contains invalid value in it's `_id` field, then it will throw a `InvalidIdException`.
- If there is another document with the same `_id` value in the collection, then it will throw a `UniqueConstraintException`.
- If a field of the document is unique indexed and it's value violates the index constraint, then it will throw a `UniqueConstraintException`.

### WriteResult

`WriteResult` contains the result of a write operation. It contains the following information:

- Number of documents affected by the write operation. You can get this value using `getAffectedCount()` method.
- List of `NitriteId` of the documents affected by the write operation. The `WriteResults` implements `Iterable<NitriteId>` interface. So you can iterate over the `WriteResult` to get the `NitriteId` of the documents affected by the write operation.

## Updating Documents

Documents can be updated using any of the following methods:

- `updateOne()` - Updates a single document.
- `update()` - Update the filtered documents in the collection.

All update methods returns a `Future<WriteResult>` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.update` event.
!!!

### Updating a Single Document

You can update a single document using `updateOne()` method. It takes a `Document` object as input parameter. It returns a `Future<WriteResult>` object. The document must contain a valid `NitriteId` in it's `_id` field. 

```dart
var doc = createDocument("_id", existingDoc.id)
    ..put("age", 30);

var result = await collection.updateOne(doc);
```

!!!warning
If the document does not contain a valid `NitriteId` in it's `_id` field, then it will throw a `NotIdentifiableException`.
!!!

### Upserting a Single Document

You can also upsert a single document using `updateOne()` method. Along with the `document` parameter, you need to pass a named parameter `insertIfAbsent` with value `true`. If the document does not exist in the collection, then it will be inserted. Otherwise, it will be updated. The default value of `insertIfAbsent` is `false`.

```dart
var doc = createDocument("_id", existingDoc.id)
    ..put("lastName", "Doe")
    ..put("age", 30);

var result = await collection.updateOne(doc, insertIfAbsent: true);
```

### Updating Using a Filter

You can update a document using a filter. It takes a `Filter` object as the first input parameter. It takes a `Document` object as the second input parameter. It returns a `Future<WriteResult>` object.

```dart
var update = createDocument("_id", existingDoc.id)
    ..put("lastName", "Doe")
    ..put("age", 30)
    ..put("data", Uint8List.fromList([1, 2, 3, 4, 5]));

var result = await collection.update(where("firstName").eq("John"), update);
```

### Updating Using a Filter and Options

You can update a document using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `Document` object as the second input parameter. It takes a `UpdateOptions` object as the optional third input parameter. It returns a `Future<WriteResult>` object.

#### UpdateOptions

`UpdateOptions` is a class that contains several options for update operation. It has the following options:

- `insertIfAbsent`: If this option is `true`, then it will insert the document if it does not exist in the collection. Otherwise, it will update the document if it exists in the collection.
- `justOnce`: If this option is `true`, then it will update only the first document matched by the filter. Otherwise, it will update all the documents matched by the filter.

```dart
var update = createDocument("_id", existingDoc.id)
    ..put("lastName", "Doe")
    ..put("age", 30)
    ..put("data", Uint8List.fromList([1, 2, 3, 4, 5]));

var updateOptions = UpdateOptions();
updateOptions.insertIfAbsent = true;
updateOptions.justOnce = true;

var result = await collection.update(where("firstName").eq("John"), update, updateOptions);
```

## Removing Documents

Documents can be removed from a collection using any of the following methods:

- `removeOne()` - Removes a single document.
- `remove()` - Removes the filtered documents from the collection.

All remove methods returns a `Future<WriteResult>` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.remove` event.
!!!

### Removing a Single Document

You can remove a single document using `removeOne()` method. It takes a `Document` object as input parameter. It returns a `Future<WriteResult>` object. The document must contain a valid `NitriteId` in it's `_id` field. 

```dart
var doc = createDocument("_id", existingDoc.id);

var result = await collection.removeOne(doc);
```

!!!warning
If the document does not contain a valid `NitriteId` in it's `_id` field, then it will throw a `NotIdentifiableException`.
!!!

### Removing Using a Filter

You can remove a document using a filter. It takes a `Filter` object as the input parameter. It returns a `Future<WriteResult>` object.

```dart
var result = await collection.remove(where("firstName").eq("John"));
```

### Removing Using a Filter and Options

You can remove a document using a filter and options. It takes a `Filter` object as the first input parameter. It takes a named parameter `justOne` with value `true` or `false`. If the value is `true`, then it will remove only the first document matched by the filter. Otherwise, it will remove all the documents matched by the filter. The default value of `justOne` is `false`. It returns a `Future<WriteResult>` object.

```dart
var result = await collection.remove(where("firstName").eq("John"), justOne: true);
```