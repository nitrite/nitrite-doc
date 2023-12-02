---
label: Introduction
icon: info
order: 17
---

`ObjectRepository` provides a simple and type-safe API for storing and retrieving Java objects in a Nitrite database. It is built on top of `NitriteCollection` and provides a similar API for CRUD operations. It also supports indexing and querying. It also supports event based notification on object changes.

`ObjectRepository` is thread-safe and supports concurrent read and write operations.

## Creating a repository

A `ObjectRepository` can be created using `Nitrite` class. You need to call `getRepository()` method on `Nitrite` class to get an instance of a `ObjectRepository`. If the repository does not exist, then it will be created automatically. If a repository with the same name already exists, then it will return the existing repository.

There are several overloaded methods available to create a repository. You can pass a class type or an `EntityDecorator` along with an optional string key to create a repository.

### Creating a repository with class type

You can create a repository by passing a class type to `getRepository()` method.

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

ObjectRepository<Employee> repository = db.getRepository(Employee.class);
```

### Creating a repository with class type and key

You can create a keyed repository by passing a class type and a key to `getRepository()` method.

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

ObjectRepository<Employee> repository = db.getRepository(Employee.class, "myKey");
```

One typical use case of this keyed repository is to create a repository for each user in a multi-user application. The key can be the user name or user id. This will ensure that each user will have a separate repository for storing objects.

### Creating a repository with EntityDecorator

A `ObjectRepository` can be created using `EntityDecorator`. This is useful when you cannot modify the object class to add annotations.

More details about `EntityDecorator` can be found [here](entity.md).

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

ObjectRepository<Employee> repository = db.getRepository(new EmployeeDecorator());
```

### Creating a repository with EntityDecorator and key

A keyed `ObjectRepository` can be created using `EntityDecorator` and a key. This is useful when you cannot modify the object class to add annotations.

More details about `EntityDecorator` can be found [here](entity.md).

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

ObjectRepository<Employee> repository = db.getRepository(new EmployeeDecorator(), "myKey");
```