---
label: Encryption
icon: key
order: 1
---

Nitrite provides field level encryption support via `nitrite-support` module. You can encrypt your data using with or without a password.

## Field Level Encryption

Nitrite provides a `StringFieldEncryptionProcessor` utility class to encrypt and decrypt a field of a document. It is a `Processor` implementation. It uses [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) to encrypt and decrypt data.

While initializing the `StringFieldEncryptionProcessor`, you can either provide a password or not. If you provide a password, the processor will use the password to encrypt and decrypt data. If you do not provide a password, the processor will use a random key to encrypt and decrypt data.

```dart
// create a processor with a password
var processor = StringFieldEncryptionProcessor("my-password");

// or, create a processor without a password
var processor = StringFieldEncryptionProcessor();

// add fields to encrypt
processor.addFields(["field1", "field2"]);

// use the processor
collection.addProcessor(processor);

// use the collection
await collection.insert(doc);
```