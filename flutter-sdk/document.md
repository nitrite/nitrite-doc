---
label: Document
icon: project-roadmap
order: 18
---

Document is the basic unit of data in Nitrite database. It is a JSON like field-value pairs. The field is always a `String` and value can be anything including `null`. Document is schema-less, which means you can store any kind of data in a document.

Nitrite document supports nested document. That means, a value of a field can be another document. This allows you to create complex data structure.

## Document Structure

Nitrite document has the following structure.

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "address": {
        "street": "123 Main Street",
        "city": "New York",
        "state": "NY",
        "zip": "10001"
    },
    "phone": [
        "212-555-1234",
        "646-555-4567"
    ]
}
```

## Document Field

Document field is always a `String`. It can be any valid string. The field cannot be `null` or empty string.

Below fields are reserved and cannot be used as key in a document.

- *_id* : The unique identifier of a document. This field is auto-generated by Nitrite database during insertion.
- *_revision* : The revision number of a document. This field is auto-generated by Nitrite database during insertion and update.
- *_source* : The source collection name of a document.
- *_modified* : The last modified timestamp of a document. This field is auto-generated by Nitrite database during insertion and update.

## Document Value

Document value can be any valid Dart object. It can be `null` or any of the below types.

- `String`
- `int`
- `double`
- `bool`
- `List`
- `Map`
- `Set`
- `DateTime`
- `NitriteId`
- `Document`

## Document Identifier

It is a unique identifier of a document. It is auto-generated by Nitrite database during insertion. It is a `NitriteId` and is stored as a `String` in the document.

### NitriteId

NitriteId is a unique identifier across the Nitrite database. Each document in a nitrite collection is associated with a `NitriteId`.

During insertion if a unique `String` value representing a integer is supplied in the `_id` field of the document, then the value of the `_id` field will be used to generate the `NitriteId`. Otherwise, a new `NitriteId` will be generated and will be used in the `_id` field of the document.

The value of the `NitriteId` is a `String` representation of a 64-bit integer. The id generation is based on Twitter Snowflake algorithm. The id is composed of:

- 41 bits for time in milliseconds
- 10 bits for a machine id
- 12 bits for a sequence number
- 1 unused sign bit

The id is not guaranteed to be monotonically increasing. The id is sortable and the timestamp is stored in the id itself.

#### Retrieving a NitriteId from a Document

The id can be retrieved from a document using `Document.id` getter. If the document does not have an id, then it will create a new `NitriteId` and will set it in the document and will return the id. If the document already has an id, then it will return the id.

```dart
var document = createDocument("firstName", "John");

var id = document.id;
```

## Field Separator

To access a field of a nested document, or an element of an array field, you need to use the field separator.

Field separator is configurable. You can change the field separator character by calling `NitriteBuilder.fieldSeparator()` method.

```dart
var db = await nitriteBuilder()
    .fieldSeparator('.')
    .openOrCreate();
```

!!!info
Nitrite uses `.` as field separator by default.
!!!

### Nested Document

To specify or access a field of a nested document, you need to use the field separator character. That means, if a document has a field <b>address</b> which is a nested document, then the field <b>street</b> of the nested document can be accessed using <b>address.street</b> syntax.

### Array Field

To specify or access an element of an array field, you need to use the index of the element. For example, if a document has a field <b>phone</b> which is an array, then the first element of the array can be accessed using <b>phone.0</b> syntax.

## Creating a Document

To create a document, you need to use `createDocument()` function.

### Creating an Empty Document

```dart
var document = emptyDocument();
```

### Creating a Document with Initial Field-value Pair

```dart
var document = createDocument("firstName", "John");
```

### Creating a Document with a Map

```dart
var map = {
    "firstName": "John",
    "lastName": "Doe",
    "address": {
        "street": "123 Main Street",
        "city": "New York",
        "state": "NY",
        "zip": "10001"
    },
    "phone": [
        "212-555-1234",
        "646-555-4567"
    ]
};

var document = documentFromMap(map);
```

## Updating a Document

To update a document, you need to use `Document.put()` method. This method takes two parameters, the field name and the value. If the field already exists in the document, then the value will be updated. If the field does not exist, then it will be created.

You can also use the `[]` operator to update a document.

```dart
var document = createDocument("firstName", "John");

document.put("lastName", "Doe");
document["address"] = createDocument("street", "123 Main Street");
```

## Retrieving a Value from Document

To retrieve a value from document, you need to use `Document.get()` method. This method takes one parameter, the field name. If the field exists in the document, then the value will be returned. If the field does not exist, then it will return `null`.

You can also use the `[]` operator to retrieve a value from a document.

```dart
var document = createDocument("firstName", "John");

var firstName = document.get("firstName");
var firstName = document["firstName"];
```

To retrieve a value from a nested document, use the field separator character.

```dart
var document = createDocument("firstName", "John")
    .put("address", createDocument("street", "123 Main Street"));

var street = document.get<String>("address.street");
```

To retrieve an element from an array field, use the index of the element.

```dart
var document = createDocument("firstName", "John")
    .put("phone", ["212-555-1234", "646-555-4567"]);

var phone = document.get<String>("phone.0");
```

## Removing a Field from Document

To remove a field from document, you need to use `Document.remove()` method. This method takes one parameter, the field name. If the field exists in the document, then it will be removed. If the field does not exist, then it will do nothing.

```dart
var document = createDocument("firstName", "John");

document.remove("firstName");
```

To remove a field from a nested document, use the field separator character.

```dart
var document = createDocument("firstName", "John")
    .put("address", createDocument("street", "123 Main Street"));

document.remove("address.street");
```

To remove an element from an array field, use the index of the element.

```dart
var document = createDocument("firstName", "John")
    .put("phone", ["212-555-1234", "646-555-4567"]);

document.remove("phone.0");
```

## Checking If a Field Exists in Document

To check if a field exists in a document, you need to use `Document.containsKey()` method. This method takes one parameter, the field name. If the field exists in the document, then it will return `true`. If the field does not exist, then it will return `false`.

```dart
var document = createDocument("firstName", "John");

var exists = document.containsKey("firstName");
```

To check if a field exists in a nested document, use the field separator character.

```dart
var document = createDocument("firstName", "John")
    .put("address", createDocument("street", "123 Main Street"));

var exists = document.containsKey("address.street");
```

!!!warning
It cannot check if an element exists in an array field.
!!!