---
label: Read Operations
icon: search
order: 13
---

## Find Operations

You can search entities in a repository using `find()` method. There are several overloaded version of `find()` method using a filter or find options or both. You can also search an entity using it's id.

### Filters

Nitrite uses filters to find entities in a repository. A filter is a simple expression which evaluates to `true` or `false`. More information about filters can be found [here](../filter.md).

### Find All Entities

You can find all entities in a repository by calling `find()`. It returns a `Cursor` object.

```java
Cursor<Product> cursor = productRepository.find();
```

### Finding an Entity Using Id

You can find an entity using it's id by calling `getById()`. It takes an id as input parameter. It returns an entity if the entity is found. Otherwise, it returns `null`.

```java
Product product = productRepository.getById(new ProductId(1));
```

### Finding an Entity Using a Filter

You can find an entity using a filter. It takes a `Filter` object as input parameter. It returns a `Cursor` object.

If there is an index on the field specified in the filter, this operation will use the index to find the entity. Otherwise, it will scan the entire repository to find the entity.

```java
Cursor<Product> cursor = productRepository.find(where("productName").eq("Apple iPhone 6"));
```

### Finding an Entity Using a Filter and Options

You can find an entity using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `FindOptions` object as the second input parameter. It returns a `Cursor` object.

#### FindOptions

More information about `FindOptions` can be found [here](../collection/read.md#findoptions).


## Cursor

`Cursor` represents a result set of a find operation. It provides methods to iterate over the result of a find operation and retrieve the entities. It also provides methods like projection, join etc. to get the desired result.

### Iterating over Entities

The `Cursor` extends `Iterable` interface. So, you can iterate over the entities using `for-each` loop.

```java
Cursor<Product> cursor = productRepository.find();

for (Product product : cursor) {
    // do something with the product
}
```

!!!info
A `Cursor` is a lazy iterable. It will not load all the entities in memory at once. It will load the entities in memory as needed. So, it is memory efficient.
!!!

### Getting the Entities

You can get the entities from the cursor at once using `toList()` and `toSet()` method. It returns a `List` and `Set` of entities respectively.

```java
Cursor<Product> cursor = productRepository.find();

List<Product> products = cursor.toList();
Set<Product> productSet = cursor.toSet();
```

### Getting the First Entity

You can get the first entity from the cursor using `firstOrNull()` method. It returns the first entity if the cursor is not empty. Otherwise, it returns `null`.

```java
Cursor<Product> cursor = productRepository.find();

Product product = cursor.firstOrNull();
```

### Getting the Size of the Cursor

You can get the size of the cursor using `size()` method. It returns the number of entities in the cursor.

```java
Cursor<Product> cursor = productRepository.find();

long size = cursor.size();
```

### Projection

You can project the cursor using `project()` method. It takes another entity as input parameter. It returns a `Cursor` object of the projected entity.

The projected entity must contain only the fields that needs to be projected.

Let's say you have an entity like this:

```java
public class User {
    private String firstName;
    private String lastName;
    private Address address;
}

public class Address {
    private String street;
    private String city;
}
```

And you want to project only `lastName` and `address.street` fields. Then you can define a new entity like this:

```java
public class UserProjection {
    private String lastName;
    private StreetAddress address;
}

public class StreetAddress {
    private String street;
}
```

And then you can project the cursor like this:

```java
Cursor<User> cursor = userRepository.find(where("firstName").eq("John"));

RecordStream<UserProjection> projectedStream = cursor.project(UserProjection.class);
```

### Join

You can join two cursors using `join()` method. It takes another cursor as the first input parameter. It takes a `Lookup` as the second input parameter and type of the join as the third input parameter. It returns a `RecordStream` object.

The join operation is similar to SQL left outer join operation. It takes two cursors and a lookup object. The lookup object contains the join condition.

Let's say you have two entities like this:

```java
public class User {
    private String userId;
    private String firstName;
    private String lastName;
}

public class Order {
    private String orderId;
    private String userId;
    private String productName;
}
```

And you want to join these two entities using `userId` field. Then you can define a new entity like this:

```java
public class UserOrder {
    private String userId;
    private String firstName;
    private String lastName;
    private List<Order> orders;
}
```

And then you can join the cursors like this:

```java
Cursor<User> userCursor = userRepository.find();
Cursor<Order> orderCursor = orderRepository.find();

Lookup lookup = new Lookup();
lookup.setLocalField("userId");
lookup.setForeignField("userId");
lookup.setTargetField("orders");

RecordStream<UserOrder> joinedStream = userCursor.join(orderCursor, lookup, UserOrder.class);
```

### FindPlan

You can get the `FindPlan` of a cursor using `getFindPlan()` method. It returns a `FindPlan` object.

More information about `FindPlan` can be found [here](../collection/read.md#findplan).

```java
Cursor<Product> cursor = productRepository.find(where("productName").eq("Apple iPhone 6"));

FindPlan findPlan = cursor.getFindPlan();
```