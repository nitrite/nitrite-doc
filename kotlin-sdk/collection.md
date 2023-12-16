---
label: Nitrite Collection
icon: stack
order: 17
---

`NitriteCollection` represents a named document collection stored in a Nitrite database. It persists documents in a Nitrite database. It is similar to a table in relational database or a collection in MongoDB.

More details on collection can be found [here](../java-sdk/collection/intro.md).

## Kotlin API for NitriteCollection

Potassium Nitrite provides a higher-order function to make collection manipulation easier.

```kotlin
db.getCollection("myCollection") {
    // do something with collection
    insert(documentOf("firstName" to "John", "lastName" to "Doe"))
    createIndex("firstName")
}
```
