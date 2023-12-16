---
label: Object Repository
icon: container
order: 16
---

`ObjectRepository` provides a simple and type-safe API for storing and retrieving Kotlin/Java objects in a Nitrite database. It is built on top of `NitriteCollection` and provides a similar API for CRUD operations. It also supports indexing and querying. It also supports event based notification on object changes.

More details on repository can be found [here](../java-sdk/repository/intro.md).

## Kotlin API for ObjectRepository

Potassium Nitrite provides some higher-order functions to make repository manipulation easier.

### Creating a Repository with Class Type

You can create a `ObjectRepository` by passing a class type to `getRepository()` method.

```kotlin
val repository = db.getRepository<Employee>() {
    // do something with repository
    insert(Employee("John", "Doe"))
    createIndex("firstName")
}
```

### Creating a Repository with Class Type and Key

You can create a keyed `ObjectRepository` by passing a class type and a key to `getRepository()` method.

```kotlin
val repository = db.getRepository<Employee>("myKey") {
    // do something with repository
    insert(Employee("John", "Doe"))
    createIndex("firstName")
}
```

One typical use case of this keyed repository is to create a repository for each user in a multi-user application. The key can be the user name or user id. This will ensure that each user will have a separate repository for storing objects.

### Creating a Repository with EntityDecorator

A `ObjectRepository` can be created using `EntityDecorator`. This is useful when you cannot modify the object class to add annotations.

More details about `EntityDecorator` can be found [here](../java-sdk/repository/entity.md#entitydecorator).

```kotlin
val repository = db.getRepository(EmployeeDecorator()) {
    // do something with repository
    insert(Employee("John", "Doe"))
    createIndex("firstName")
}
```

### Creating a Repository with EntityDecorator and Key

A keyed `ObjectRepository` can be created using `EntityDecorator` and a key. This is useful when you cannot modify the object class to add annotations.

More details about `EntityDecorator` can be found [here](../java-sdk/repository/entity.md#entitydecorator).

```kotlin
val repository = db.getRepository(EmployeeDecorator(), "myKey") {
    // do something with repository
    insert(Employee("John", "Doe"))
    createIndex("firstName")
}
```

## NitriteMapper

Nitrite provides a mapper interface called `NitriteMapper`. It is used to map Java objects to Nitrite documents and vice-versa. More details on `NitriteMapper` can be found [here](../java-sdk/repository/mapper.md).

Apart from the `SimpleNitriteMapper`, Potassium Nitrite also provides two additional mappers:

- `KNO2JacksonMapper`
- `KotlinXSerializationMapper`

### Jackson Mapper

Potassium Nitrite extends the `JacksonMapper` to provide `KNO2JacksonMapper`. This class overrides the `getObjectMapper()` method to register some Kotlin specific Jackson modules as follows:

- KotlinModule
- JavaTimeModule
- Jdk8Module
- [GeometryModule](../java-sdk/modules/spatial.md#using-spatial-modules-with-jackson)

It also disables the `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` feature. The constructor of `KNO2JacksonMapper` takes a vararg of `com.fasterxml.jackson.databind.Module` to register additional Jackson modules.

To use the `KNO2JacksonMapper` instead of the default `SimpleNitriteMapper` you need to load it while opening the database.

```kotlin
val db = nitrite {
    loadModule(module(KNO2JacksonMapper()))
}
```

### KotlinX Serialization Mapper

Potassium Nitrite also provides a KotlinX Serialization based mapper called `KotlinXSerializationMapper`. KotlinX Serialization is used to convert Java/Kotlin objects to `Document` and vice-versa. 

To use the `KotlinXSerializationMapper` instead of the default `SimpleNitriteMapper` you need to load it while opening the database.

```kotlin
val db = nitrite {
    loadModule(module(KotlinXSerializationMapper()))
}
```

