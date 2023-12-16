---
label: Document
icon: project-roadmap
order: 18
---

Document is the basic unit of data in Nitrite database. It is a JSON like field-value pairs. The field is always a `String` and value can be anything including `null`. Document is schema-less, which means you can store any kind of data in a document.

Nitrite document supports nested document. That means, a value of a field can be another document. This allows you to create complex data structure.

More details on document can be found [here](../java-sdk/document.md).

## Kotlin API for Document

Potassium Nitrite provides a set of extension functions to make document manipulation easier. 

### Create a Document

To create a document, you can use the `documentOf` function. It takes a vararg of `Pair<String, Any?>` and returns a `Document`.

```kotlin
val document = documentOf("firstName" to "John", "lastName" to "Doe")
```

To create a nested document, you can use the `documentOf` function recursively.

```kotlin
val document = documentOf(
    "firstName" to "John",
    "lastName" to "Doe",
    "address" to documentOf(
        "street" to "123 Main Street",
        "city" to "New York",
        "state" to "NY",
        "zip" to "10001"
    ),
    "phone" to listOf("212-555-1234", "646-555-4567")
)
```

To create an empty document, you can use the `emptyDocument` function.

```kotlin
val document = emptyDocument()
```

### Get a Field Value

To get a field value from a document, you can use the index operator `[]` on the document. It takes a `String` as the field name and returns the value of the field.

```kotlin
val firstName = document["firstName"]
```

### Checking If a Document is Empty

To check if a document is empty, you can use the `isEmpty` function. It takes no argument and returns a `Boolean`.

```kotlin
val isEmpty = document.isEmpty()
```

Conversely, to check if a document is not empty, you can use the `isNotEmpty` function. It takes no argument and returns a `Boolean`.

```kotlin
val isNotEmpty = document.isNotEmpty()
```