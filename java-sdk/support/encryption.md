---
label: Encryption
icon: key
order: 2
---

Nitrite provides encryption support via `nitrite-support` module. You can encrypt your data using a password. Nitrite provides an `Encryptor` interface to encrypt and decrypt data. Nitrite also provides an [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) based implementation of `Encryptor` interface, which uses AES/GCM/NoPadding algorithm to encrypt and decrypt data.


## Using Encryption

To use encryption, you can either use the `AES` implementation of `Encryptor` interface or you can create your own implementation of `Encryptor` interface.

### Using AES Encryptor

To use AES based encryption you need to use the `AESEncryptor` class. The class takes a password as a parameter in its constructor.

```java
// create an AES encryptor with a password
Encryptor encryptor = new AESEncryptor("my-password");
```

There is another constructor also which takes a password along with the algorithm, tag length, IV length, and salt length. You can use the default constructor if you don't want to provide these parameters.
    
```java
// create an AES encryptor with a password and algorithm parameters

// algo: AES/CBC/PKCS5Padding
// tag length: 128
// IV length: 12
// salt length: 16

Encryptor encryptor = new AESEncryptor("my-password", "AES/CBC/PKCS5Padding", 128, 12, 16);
```

You can use the encryptor to encrypt and decrypt data.

```java
// encrypt a string
String encrypted = encryptor.encrypt("my-data".getBytes(StandardCharsets.UTF_8));

// decrypt the string
String decrypted = encryptor.decrypt(encrypted);
```

### Using Custom Encryptor

If you want to use your own encryption algorithm, you can do so by implementing the `Encryptor` interface. The interface has two methods `encrypt` and `decrypt`. You can implement these methods to encrypt and decrypt data.

```java
public class MyEncryptor implements Encryptor {
    @Override
    public String encrypt(byte[] plainText) {
        // implement your encryption logic
    }

    @Override
    public String decrypt(String encryptedText) {
        // implement your decryption logic
    }
}
```

## Field Level Encryption

Nitrite provides field level encryption support. You can encrypt a field of a document using a password. Nitrite provides an `StringFieldEncryptionProcessor` utility class to encrypt and decrypt a field of a document. It uses the `Encryptor` interface to encrypt and decrypt data.

### Using Field Encryption

To use field encryption, you need to create an instance of `StringFieldEncryptionProcessor` class. The class takes either an `Encryptor` instance or a password as a parameter in its constructor.

If you provide the password, the class will create an instance of `AESEncryptor` class using the password. If you provide an `Encryptor` instance, the class will use the provided `Encryptor` instance.

```java
// create an instance of StringFieldEncryptionProcessor using a password
StringFieldEncryptionProcessor processor = new StringFieldEncryptionProcessor("my-password");

// add fields to encrypt
processor.addFields("field1", "field2");

// use the processor
collection.addProcessor(processor);

// use the collection
collection.insert(doc);
```

You can also use the `AESEncryptor` class to create an instance of `StringFieldEncryptionProcessor` class.

```java
// create an instance of AESEncryptor
Encryptor encryptor = new AESEncryptor("my-password");

// create an instance of StringFieldEncryptionProcessor using the encryptor
StringFieldEncryptionProcessor processor = new StringFieldEncryptionProcessor(encryptor);

// add fields to encrypt
processor.addFields("field1", "field2");

// use the processor
collection.addProcessor(processor);

// use the collection
collection.insert(doc);
```

!!!info Note
The encrypted data is a base64 encoded string of the encrypted bytes.
!!!
