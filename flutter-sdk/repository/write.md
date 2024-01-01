---
label: Write Operations
icon: pencil
order: 13
---

## Inserting Entities

Entities can be inserted into a repository using `insert()` or `insertMay()` method. It takes one or multiple Dart entities as input parameter respectively. It returns a future of `WriteResult` object.

If the entity has a `NitriteId` field, then the field value would be populated with a new `NitriteId` before inserting into the repository.

If any of the field is already indexed in the repository, then the index will be updated accordingly.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.insert` event.
!!!

### Inserting a Single Entity

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 600.0;

var result = await productRepository.insert(product);
```

### Inserting Multiple Entities

```dart
var product1 = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 600.0;

var product2 = Product()
    ..productId = ProductId(2)
    ..productName = 'Apple iPhone 6 Plus'
    ..price = 700.0;

var result = await productRepository.insertMany([product1, product2]);
```

### Error Scenarios

- If a field of the entity is unique indexed and it's value violates the index constraint, then it will throw a `UniqueConstraintException`.

### WriteResult

More information about `WriteResult` can be found [here](../collection/write.md#writeresult).

## Updating Entities

Entities can be updated using any of the following methods:

- `updateOne()` - updates a single entity
- `update()` - updates multiple entities using a filter and update entity
- `updateDocument()` - updates multiple entities using a filter and update document

All update methods returns a `Future<WriteResult>` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.update` event.
!!!

### Updating a Single Entity

You can update a single entity using `updateOne()` method. It takes a Dart entity as input parameter. It returns a `Future<WriteResult>` object. The entity must have a valid id field.

```dart

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 600.0;

var result = await productRepository.insert(product);

product.price = 700.0;

result = await productRepository.updateOne(product);
```


!!!warning Warning
The entity must have a valid id field marked with `@Id` annotation. In case an `EntityDecorator` is used, the `idField` getter must return a valid non-null `EntityId` object. Otherwise, it will throw a `NotIdentifiableException`.
!!!

### Upserting a Single Entity

You can also upsert a single entity using `updateOne()` method. Along with the `entity` parameter, you need to pass a named parameter `insertIfAbsent` with value `true`. If the entity does not exist in the repository, then it will be inserted. Otherwise, it will be updated. The default value of `insertIfAbsent` is `false`.

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 600.0;

var result = await productRepository.updateOne(product, insertIfAbsent: true);
```

### Updating Using a Filter

You can update multiple entities using a filter. It takes a `Filter` object as first input parameter. It takes a Dart entity as second input parameter. It returns a `Future<WriteResult>` object.

If the filter result matches multiple entities, then all the entities will be updated.

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 650.0;

var result = await productRepository.update(where('productName').eq('Apple iPhone 6'), product);
```

### Updating Using a Filter and Options

You can update multiple entities using a filter and options. It takes a `Filter` object as first input parameter. It takes a Dart entity as second input parameter. It takes a `UpdateOptions` object as third input parameter. It returns a `Future<WriteResult>` object.

#### UpdateOptions

More information about `UpdateOptions` can be found [here](../collection/write.md#updateoptions).

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 650.0;

var updateOptions = UpdateOptions();
updateOptions.insertIfAbsent = true;

var result = await productRepository.update(where('productName').eq('Apple iPhone 6'), product, updateOptions);
```

### Updating Using a Filter and Document

You can update multiple entities using a filter. It takes a `Filter` object as first input parameter. It takes a `Document` object as second input parameter. It returns a `Future<WriteResult>` object.

If the filter result matches multiple entities, then all the entities will be updated. The document must contain only the fields that needs to be updated.

```dart
var update = createDocument('price', 700);

var result = await productRepository.updateDocument(where('productName').eq('Apple iPhone 6'), update);
```

!!!warning
The document should not contain `_id` field.
!!!

### Updating Using a Filter, Document and Options

You can update multiple entities using a filter, document and options. It takes a `Filter` object as first input parameter. It takes a `Document` object as second input parameter. It takes a boolean value as third input parameter. If the third input parameter is `true`, then it will only update the first entity matched by the filter. Otherwise, it will update all the entities matched by the filter. It returns a `Future<WriteResult>` object.

The document must contain only the fields that needs to be updated.

```dart
var update = createDocument('price', 700);

var result = await productRepository.updateDocument(where('productName').eq('Apple iPhone 6'), update, justOnce: true);
```

!!!warning
The document should not contain `_id` field.
!!!

## Removing Entities

Entities can be removed using any of the following methods:

- `removeOne()` - removes a single entity
- `remove()` - removes multiple entities using a filter

All remove methods returns a `Future<WriteResult>` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.remove` event.
!!!

### Removing a Single Entity

You can remove a single entity using `removeOne()` method. It takes a Dart entity as input parameter. It returns a `Future<WriteResult>` object.

The entity must have a valid id field marked with `@Id` annotation. In case an `EntityDecorator` is used, the `idField` getter must return a valid non-null `EntityId` object. Otherwise, it will throw a `NotIdentifiableException`.

```dart
var product = Product()
  ..productId = ProductId(1)
  ..productName = 'Apple iPhone 6'
  ..price = 600.0;

var result = await productRepository.insert(product);

result = await productRepository.removeOne(product);
```

### Removing Using a Filter

You can remove multiple entities using a filter. It takes a `Filter` object as input parameter. It returns a `Future<WriteResult>` object.

If the filter result matches multiple entities, then all the entities will be removed.

```dart
var result = await productRepository.remove(where('productName').eq('Apple iPhone 6'));
```

### Removing Using a Filter and Options

You can remove multiple entities using a filter and options. It takes a `Filter` object as first input parameter. It takes a `boolean` value as second input parameter. If the second input parameter is `true`, then it will remove only the first entity matched by the filter. Otherwise, it will remove all the entities matched by the filter. It returns a `Future<WriteResult>` object.

```dart
var result = await productRepository.remove(where('productName').eq('Apple iPhone 6'), justOne: true);
```