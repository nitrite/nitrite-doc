---
label: Other Operations
icon: cpu
order: 11
---

## Size of a repository

You can get the size of a repository using `size()` method. It returns the number of entities in the repository.

```java
long size = productRepository.size();
```

## Clearing a repository

You can clear all the entities from a repository using `clear()` method. It removes all the entities from the repository and index entries from the indexes. It does not drop the repository.

```java
productRepository.clear();
```

## Dropping a repository

You can drop a repository using `drop()` method. It removes all the entities from the repository and index entries from the indexes. It also drops all the indexes associated with the repository. It also removes the repository from the database.

```java
productRepository.drop();
```

You can call `isDropped()` method to check if the repository is dropped or not.

```java
boolean isDropped = productRepository.isDropped();
```

!!!warning Warning
Any further operation on a dropped repository will throw `NitriteIOException`.
!!!

## Closing a repository

You can close a repository using `close()` method. Any further operation on a closed repository will throw `NitriteIOException`.

```java
productRepository.close();
```

After closing a repository, you must re-open it via `Nitrite.getRepository()` method to perform any operation on it.

You can call `isOpen()` method to check if the repository is closed or not.

```java
boolean isOpen = productRepository.isOpen();
```

## Event Listener

You can register an event listener on a repository to get notified on entity changes. The event listener must implement `CollectionEventListener` interface. It will receive `CollectionEventInfo` whenever an entity is inserted, updated or removed from the repository.

```java
productRepository.subscribe(eventInfo -> {
    // do something with the eventInfo
});
```

You can also remove an event listener from a collection.

```java
productRepository.unsubscribe(listener);
```

### CollectionEventInfo

More on `CollectionEventInfo` can be found [here](../collection/other.md#collectioneventinfo).

## Attributes

Attributes is a metadata information associated with a repository. 

You can get/set attributes on a repository. The attributes are stored in the database and can be retrieved later. The attributes are stored in a special map named `$nitrite_meta_map`.

```java
// get the attributes
Attributes attributes = productRepository.getAttributes();
attributes.set("key", "value");

// save the attributes
productRepository.setAttributes(attributes);

// get the attributes
attributes = productRepository.getAttributes();

// get the value of a key
String value = attributes.get("key");
```

## Processors

Processor can be used to process the underlying documents of a repository. More on processors can be found [here](../collection/other.md#processors).

### Registering a processor

You can register a processor on a repository using `addProcessor()` method. It takes a `Processor` as input parameter.

```java
// create a processor
MyProcessor processor = new MyProcessor();

// process existing entities
processor.process(productRepository);

// register the processor
productRepository.addProcessor(processor);
```

### Available processors

Nitrite provides a `StringFieldEncryptionProcessor` which can be used to encrypt a field before writing it into a repository and decrypt a field after reading it from a repository.

More on this is described in [Support](../support/encryption.md).