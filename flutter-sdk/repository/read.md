---
label: Read Operations
icon: search
order: 12
---

## Find Operations

You can search entities in a repository using `find()` method. There are several ways of using `find()` method using a filter or find options or both. You can also search an entity using it's id.

### Filters

Nitrite uses filters to find entities in a repository. A filter is a simple expression which evaluates to `true` or `false`. More information about filters can be found [here](../filter.md).

### Find All Entities

You can find all entities in a repository by calling `find()`. It returns a `Cursor` object.

```dart
Cursor<Product> cursor = productRepository.find();
```

### Finding an Entity Using Id

You can find an entity using it's id by calling `getById()`. It takes an id as input parameter. It returns an entity if the entity is found. Otherwise, it returns `null`.

```dart
var product = await productRepository.getById(ProductId(1));
```

### Finding an Entity Using a Filter

You can find an entity using a filter. It takes a `Filter` object as input parameter. It returns a `Cursor` object.

If there is an index on the field specified in the filter, this operation will use the index to find the entity. Otherwise, it will scan the entire repository to find the entity.

```dart
Cursor<Product> cursor = productRepository.find(filter: where('productName').eq('Apple iPhone 6'));
```

### Finding an Entity Using a Filter and Options

You can find an entity using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `FindOptions` object as the second input parameter. It returns a `Cursor` object.

#### FindOptions

More information about `FindOptions` can be found [here](../collection/read.md#findoptions).

## Cursor

`Cursor` represents a result set of a find operation. It provides methods to iterate over the result of a find operation and retrieve the entities. It also provides methods like projection, join etc. to get the desired result.

### Iterating over Entities

The `Cursor` extends `Stream` of entities. So, you can iterate over the entities using await for loop.

```dart
Cursor<Product> cursor = productRepository.find();

await for (var product in cursor) {
  // do something with product
}
```

!!!info
A `Cursor` is a stream of entities. It does not load all the entities in memory at once. It loads the entity in memory as needed. So, it is memory efficient.
!!!

### Getting the Entities

You can get the entities from a `Cursor` using `toList()` method. It returns a `Future<List<T>>` object.

```dart
Cursor<Product> cursor = productRepository.find();

var products = await cursor.toList();
```

### Getting the First Entity

You can get the first entity from a `Cursor` using `first` getter. It returns a `Future<T?>` object.

```dart
Cursor<Product> cursor = productRepository.find();

var product = await cursor.first;
```

### Getting the Size of the Cursor

You can get the size of the cursor using `length` getter. It returns a `Future<int>` object.

```dart
Cursor<Product> cursor = productRepository.find();

var size = await cursor.length;
```

### Projection

You can project a field or a set of fields from a `Cursor` using `project()` method. It takes another entity type as input parameter. It returns a `Stream` of projected entities.

The projected entity must contain only the fields that needs to be projected.

Let's say you have an entity like this:

```dart
@Entity()
@Convertable()
class User {
  String? firstName;
  String? lastName;
  Address? address;

  User({this.firstName, this.lastName, this.address});
}

@Convertable()
class Address {
  String? city;
  String? state;
  String? country;

  Address({this.city, this.state, this.country});
}
```

Then you can project the `lastName` and `address.street` fields. Then you define a new entity like this:

```dart
@Entity()
@Convertable()
class UserProjection {
  String? lastName;
  StreetAddress? address;

  UserProjection({this.lastName, this.address});
}

@Convertable()
class StreetAddress {
  String? street;

  StreetAddress({this.street});
}
```

And then you can project the cursor like this:

```dart
Cursor<User> cursor = userRepository.find(filter: where('firstName').eq('John'));

var projectedStream = cursor.project<UserProjection>();
```

### Join

You can join two cursors using `leftJoin()` method. It takes another cursor as the first input parameter. It takes a `Lookup` as the second input parameter. The first type parameter is the type of the foreign entity. The second type parameter is the type of the joined entity. It returns a `Stream` of joined entities.

The join operation is similar to SQL left outer join operation. It takes two cursors and a lookup object. The lookup object contains the join condition.

Let's say you have two entities like this:

```dart
@Entity()
@Convertable()
class User {
  String? userId;
  String? firstName;
  String? lastName;

  User({this.userId, this.firstName, this.lastName});
}

@Entity()
@Convertable()
class Order {
  String? orderId;
  String? userId;
  String? productName;

  Order({this.orderId, this.userId, this.productName});
}
```

And you want to join these two entities using `userId` field. Then you can define a new entity like this:

```dart
@Entity()
@Convertable()
class UserOrder {
  String? userId;
  String? firstName;
  String? lastName;
  List<Order>? orders;

  UserOrder({this.userId, this.firstName, this.lastName, this.orders});
}
```

And then you can join the cursors like this:

```dart
Cursor<User> userCursor = userRepository.find();
Cursor<Order> orderCursor = orderRepository.find();

var lookup = Lookup()
  ..localField = 'userId'
  ..foreignField = 'userId'
  ..alias = 'orders';

var joinedStream = userCursor.leftJoin<Order, UserOrder>(orderCursor, lookup);
```

### FindPlan

You can get the `FindPlan` of a cursor using `findPlan` getter. It returns a future of `FindPlan` object.

More information about `FindPlan` can be found [here](../collection/read.md#findplan).

```dart
Cursor<Product> cursor = productRepository.find(filter: where('productName').eq('Apple iPhone 6'));

var findPlan = await cursor.findPlan;
```
