---
label: Other Operations
icon: cpu
order: 10
---

## Size of a Repository

You can get the size of a repository using `size` getter. It returns a future of the number of entities in the repository.

```dart
int size = await productRepository.size;
```

## Clearing a Repository

You can clear all the entities from a repository using `clear()` method. It removes all the entities from the repository and index entries from the indexes. It does not drop the repository.

```dart
await productRepository.clear();
```

## Dropping a Repository

You can drop a repository using `drop()` method. It removes all the entities from the repository and index entries from the indexes. It also drops all the indexes associated with the repository. It also removes the repository from the database.

```dart
await productRepository.drop();
```

You can call `isDropped` getter to check if the repository is dropped or not.

```dart
bool isDropped = productRepository.isDropped;
```

!!!warning
Any further operation on a dropped repository will throw `NitriteIOException`.
!!!

## Closing a Repository

You can close a repository using `close()` method. Any further operation on a closed repository will throw `NitriteIOException`.

```dart
await productRepository.close();
```

After closing a repository, you must re-open it via `Nitrite.getRepository()` method to perform any operation on it.

You can call `isOpen` getter to check if the repository is closed or not.

```dart
bool isOpen = productRepository.isOpen;
```

## Event Listener

You can register an event listener on a repository to get notified on entity changes. The event listener is a function of type `CollectionEventListener`. It will receive `CollectionEventInfo` whenever an entity is inserted, updated or removed from the repository.

```dart
productRepository.subscribe((eventInfo) {
  // do something with eventInfo
});
```

You can unsubscribe the event listener using `unsubscribe()` method.

```dart
productRepository.unsubscribe(listener);
```

### CollectionEventInfo

More on `CollectionEventInfo` can be found [here](../collection/other.md#collectioneventinfo).

## Attributes

Attributes is a metadata information associated with a repository. 

You can get/set attributes on a repository. The attributes are stored in the database and can be retrieved later. The attributes are stored in a special map named `$nitrite_meta_map`.

```dart
// get the attributes
var attributes = await productRepository.getAttributes();
attributes.set('key', 'value');

// save the attributes
await productRepository.setAttributes(attributes);

// get the attributes
attributes = await productRepository.getAttributes();

// get the value of a key
var value = attributes.get['key'];
```

## Processors

Processor can be used to process the underlying documents of a repository. More on processors can be found [here](../collection/other.md#processors).

### Registering a Processor

You can register a processor on a repository using `addProcessor()` method. It takes a `Processor` as input parameter.

```dart
// create a processor
var processor = MyProcessor();

// process existing entities
await processor.process(productRepository);

// register the processor
productRepository.addProcessor(processor);
```

### Available Processors

Nitrite provides a `StringFieldEncryptionProcessor` which can be used to encrypt a field before writing it into a repository and decrypt a field after reading it from a repository.

More on this is described in [Support](../support/encryption.md#field-level-encryption).