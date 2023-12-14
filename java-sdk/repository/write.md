---
label: Write Operations
icon: pencil
order: 14
---

## Inserting Entities

Entities can be inserted into a repository using `insert()` method. It takes one or multiple Java entities as input parameter. It returns a `WriteResult` object.

If the entity has a `NitriteId` field, then the field value would be populated with a new `NitriteId` before inserting into the repository.

If any of the field is already indexed in the repository, then the index will be updated accordingly.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Insert` event.
!!!

### Inserting a Single Entity

```java
Product product = new Product();
product.setProductId(new ProductId());
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

WriteResult result = productRepository.insert(product);
```

### Inserting Multiple Entities

```java
Product product1 = new Product();
product1.setProductId(new ProductId(1));
product1.setProductName("Apple iPhone 6");
product1.setPrice(600.0);

Product product2 = new Product();
product2.setProductId(new ProductId(2));
product2.setProductName("Apple iPhone 6 Plus");
product2.setPrice(700.0);

WriteResult result = productRepository.insert(product1, product2);
```

### Error Scenarios

- If the entity is `null`, then it will throw a `ValidationException`.
- If a field of the entity is unique indexed and it's value violates the index constraint, then it will throw a `UniqueConstraintException`.

### WriteResult

More information about `WriteResult` can be found [here](../collection/write.md#writeresult).

## Updating Entities

Entities can be updated in a repository using `update()` method. There are several overloaded methods available for updating entities. All of them returns a `WriteResult` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Update` event.
!!!

### Updating a Single Entity

You can update a single entity using `update()` method. It takes a Java entity as input parameter. It returns a `WriteResult` object.

```java
Product product = new Product();
product.setProductId(new ProductId(1));
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

WriteResult result = productRepository.insert(product);

product.setPrice(700.0);
result = productRepository.update(product);
```

!!!warning Warning
The entity must have a valid id field marked with `@Id` annotation. In case an `EntityDecorator` is used, the `getIdField()` method must return a valid non-null `EntityId` object. Otherwise, it will throw a `NotIdentifiableException`.
!!!

### Upserting a Single Entity

You can upsert a single entity using `update()` method. It takes a Java entity as first input parameter. It takes a `boolean` value as second input parameter. If the second input parameter is `true`, then it will insert the entity if it does not exist in the repository. Otherwise, it will update the entity if it exists in the repository. It returns a `WriteResult` object.

```java
Product product = new Product();
product.setProductId(new ProductId(1));
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

WriteResult result = productRepository.update(product, true);
```

### Updating Using a Filter

You can update an entity using a filter. It takes a `Filter` object as first input parameter. It takes a Java entity as second input parameter. It returns a `WriteResult` object. The entity must not be `null`.

If the filter result matches multiple entities, then all the entities will be updated.

```java
Product product = new Product();
product.setProductId(new ProductId(1));
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

WriteResult result = productRepository.update(where("productName").eq("Apple iPhone 6"), product);
```

### Updating Using a Filter and Options

You can update an entity using a filter and options. It takes a `Filter` object as first input parameter. It takes a Java entity as second input parameter. It takes a `UpdateOptions` object as third input parameter. It returns a `WriteResult` object. The entity must not be `null`.

#### UpdateOptions

More information about `UpdateOptions` can be found [here](../collection/write.md#updateoptions).

```java
Product product = new Product();
product.setProductId(new ProductId(1));
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

UpdateOptions updateOptions = new UpdateOptions();
updateOptions.insertIfAbsent(true);

WriteResult result = productRepository.update(where("productName").eq("Apple iPhone 6"), product, updateOptions);
```

### Updating Using a Filter and Document

You can update multiple entities using a filter and document. It takes a `Filter` object as first input parameter. It takes a `Document` object as second input parameter. It returns a `WriteResult` object. The document must not be `null` or empty. 

If the filter result matches multiple entities, then all the entities will be updated. The document must contain only the fields that needs to be updated.

```java
Document update = Document.createDocument("price", 700.0);

WriteResult result = productRepository.update(where("productName").eq("Apple iPhone 6"), update);
```

!!!warning
The document should not contain `_id` field.
!!!

### Updating Using a Filter, Document and Options

You can update multiple entities using a filter, document and options. It takes a `Filter` object as first input parameter. It takes a `Document` object as second input parameter. It takes a boolean value as third input parameter. If the third input parameter is `true`, then it will only update the first entity matched by the filter. Otherwise, it will update all the entities matched by the filter. It returns a `WriteResult` object. The document must not be `null` or empty.

The document must contain only the fields that needs to be updated.

```java
Document update = Document.createDocument("price", 700.0);

WriteResult result = productRepository.update(where("productName").eq("Apple iPhone 6"), update, false);
```

!!!warning
The document should not contain `_id` field.
!!!

## Removing Entities

Entities can be removed from a repository using `remove()` method. There are several overloaded methods available for removing entities. All of them returns a `WriteResult` object.

!!!info
This operation will notify all the registered `CollectionEventListener` with `EventType.Remove` event.
!!!

### Removing a Single Entity

You can remove a single entity using `remove()` method. It takes a Java entity as input parameter. It returns a `WriteResult` object.

The entity must have a valid id field marked with `@Id` annotation. In case an `EntityDecorator` is used, the `getIdField()` method must return a valid non-null `EntityId` object. Otherwise, it will throw a `NotIdentifiableException`.

```java
Product product = new Product();
product.setProductId(new ProductId(1));
product.setProductName("Apple iPhone 6");
product.setPrice(600.0);

WriteResult result = productRepository.insert(product);

result = productRepository.remove(product);
```

### Removing Using a Filter

You can remove an entity using a filter. It takes a `Filter` object as input parameter. It returns a `WriteResult` object.

If the filter result matches multiple entities, then all the entities will be removed.

```java
WriteResult result = productRepository.remove(where("productName").eq("Apple iPhone 6"));
```

### Removing Using a Filter and Options

You can remove an entity using a filter and options. It takes a `Filter` object as first input parameter. It takes a `boolean` value as second input parameter. If the second input parameter is `true`, then it will remove only the first entity matched by the filter. Otherwise, it will remove all the entities matched by the filter. It returns a `WriteResult` object.

```java
WriteResult result = productRepository.remove(where("productName").eq("Apple iPhone 6"), false);
```
