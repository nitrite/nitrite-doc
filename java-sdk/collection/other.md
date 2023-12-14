---
label: Other Operations
icon: cpu
order: 13
---

## Size of a Collection

You can get the size of a collection using `size()` method. It returns the number of documents in the collection.

```java
long size = collection.size();
```

## Clearing a Collection

You can clear all the documents from a collection using `clear()` method. It removes all the documents from the collection and index entries from the indexes. It does not drop the collection.

```java
collection.clear();
```

## Dropping a Collection

You can drop a collection using `drop()` method. It removes all the documents from the collection and index entries from the indexes. It also drops all the indexes associated with the collection. It also removes the collection from the database.

```java
collection.drop();
```

You can call `isDropped()` method to check if the collection is dropped or not.

```java
boolean isDropped = collection.isDropped();
```

!!!warning
Any further operation on a dropped collection will throw `NitriteIOException`.
!!!

## Closing a Collection

You can close a collection using `close()` method. Any further operation on a closed collection will throw `NitriteIOException`.

```java
collection.close();
```

After closing a collection, you must re-open it via `Nitrite.getCollection()` method to perform any operation on it.

You can call `isOpen()` method to check if the collection is closed or not.

```java
boolean isOpen = collection.isOpen();
```

## Event Listener

You can register an event listener on a collection to get notified on document changes. The event listener must implement `CollectionEventListener` interface. It will receive `CollectionEventInfo` whenever a document is inserted, updated or removed from the collection.

```java
collection.subscribe(eventInfo -> {
    // do something
});
```

You can also remove an event listener from a collection.

```java
collection.unsubscribe(listener);
```

### CollectionEventInfo

`CollectionEventInfo` contains the following information:

- `item` - the document which is inserted, updated or removed.
- `originator` - the name of the collection on which the event is fired.
- `eventType` - the type of the event. It can be any of the following:
    - `EventType.Insert`
    - `EventType.Update`
    - `EventType.Remove`
    - `EventType.IndexStart`
    - `EventType.IndexEnd`
- `timestamp` - the timestamp of the event.

## Attributes

Attributes is a metadata information associated with a collection. 

You can get/set attributes on a collection. The attributes are stored in the database and can be retrieved later. The attributes are stored in a special map named `$nitrite_meta_map`.

```java
// get the attributes
Attributes attributes = collection.getAttributes();
attributes.set("key", "value");

// save the attributes
collection.setAttributes(attributes);

// get the attributes
attributes = collection.getAttributes();

// get the value of a key
String value = attributes.get("key");
```

## Processors

Processors are used to process documents before writing them into or after reading them from a collection. 

Processors are useful when you want to add some additional information in a document or transform any field before inserting or updating it in a collection. Processors are also useful when you want to validate a document before inserting or updating it in a collection.

Processors are registered on a collection. A collection can have multiple processors. Processors are executed in the order they are registered.

### Registering a Processor

You can register a processor on a collection using `addProcessor()` method. It takes a `Processor` as input parameter.

```java
public class MyProcessor implements Processor {
    @Override
    public Document processBeforeWrite(Document document) {
        // encrypt the secret value
        document.put("secretValue", encrypt(document.get("secretValue")));
        return document;
    }

    @Override
    public Document processAfterRead(Document document) {
        // decrypt the secret value
        document.put("secretValue", decrypt(document.get("secretValue")));
        return document;
    }
}

// create a processor
MyProcessor processor = new MyProcessor();

// process existing documents, if the collection is not empty
processor.process(collection);

// register the processor
collection.addProcessor(processor);
```

### Available Processors

Nitrite provides a `StringFieldEncryptionProcessor` which can be used to encrypt a field before writing it into a collection and decrypt a field after reading it from a collection.

More on this is described in [Support](../support/encryption.md#field-level-encryption).


