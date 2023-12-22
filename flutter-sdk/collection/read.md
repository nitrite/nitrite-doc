---
label: Read Operations
icon: search
order: 15
---

## Find Operations

You can search documents in a collection using `find()` method. There are several overloaded version of `find()` method using a filter or find options or both. You can also search a document using it's `NitriteId`.

### Filters

Nitrite uses filters to find documents in a collection. A filter is a simple expression which evaluates to `true` or `false`. More information about filters can be found [here](../filter.md).

### Find All Documents

You can find all documents in a collection by calling `find()`. It returns a `DocumentCursor` object.

```dart
var cursor = collection.find();
```

### Finding a Document Using NitriteId

You can find a document using it's `NitriteId` by calling `getById()`. It takes a `NitriteId` as input parameter. It returns a `Future<Document?>` object if the document is found. Otherwise, it returns `null`.

```dart
var doc = await collection.getById(someNitriteId);
```

### Finding a Document Using a Filter

You can find a document using a filter. It takes a `Filter` object as input parameter. It returns a `DocumentCursor` object.

If there is an index on the field specified in the filter, this operation will use the index to find the document. Otherwise, it will scan the entire collection to find the document.

```dart
var cursor = collection.find(where("firstName").eq("John"));
```

### Finding a Document Using a Filter and Options

You can find a document using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `FindOptions` object as the second input parameter. It returns a `DocumentCursor` object.

#### FindOptions

`FindOptions` is a class that contains several options for find operation. It has the following options:

- `limit`: It specifies the maximum number of documents to be returned by the find operation.
- `skip`: It specifies the number of documents to be skipped from the beginning of the result set.
- `orderBy`: It specifies a collection of fields to be sorted by, along with sort order for each field. The sort order can be `SortOrder.ascending` or `SortOrder.descending`.
- `distinct`: It specifies if the find operation should return distinct documents. If this option is `true`, then it will return only the distinct documents. Otherwise, it will return all the documents matched by the filter.

```dart
var findOptions = FindOptions();
findOptions.setSkip(10)
    ..setLimit(10)
    ..thenOrderBy("firstName", SortOrder.descending)
    ..withDistinct(true);

var cursor = collection.find(filter: where("firstName").eq("John"), findOptions: findOptions);
```

## DocumentCursor

`DocumentCursor` represents a result set of a find operation. It provides methods to iterate over the result of a find operation and retrieve the documents. It also provides methods like projection, join etc. to get the desired result.

### Iterating Over Documents

The `DocumentCursor` extends `Stream<Document>`. So, you can iterate over the documents using await for loop.

```dart
var cursor = collection.find(where("firstName").eq("John"));

await for (var doc in cursor) {
  // do something with the document
}
```

!!!info
A `DocumentCursor` is a stream of document. It does not load all the documents in memory at once. It loads the documents in memory as needed. So, it is memory efficient.
!!!

### Getting the Documents

You can get the documents from a `DocumentCursor` using `toList()` method. It returns a `Future<List<Document>>` object.

```dart
var cursor = collection.find(where("firstName").eq("John"));

var docs = await cursor.toList();
```

### Getting the First Document

You can get the first document from a `DocumentCursor` using `first` getter. It returns a `Future<Document?>` object.

```dart
var cursor = collection.find(where("firstName").eq("John"));

var doc = await cursor.first;
```

### Getting the Size of the Result Set

You can get the size of the result set from a `DocumentCursor` using `length` getter. It returns a `Future<int>` object.

```dart
var cursor = collection.find(where("firstName").eq("John"));

var size = await cursor.length;
```

### Projection

You can project a field or a set of fields from a `DocumentCursor` using `project()` method. It takes a `Document` as input parameter. It returns a `Stream<Document>` object.

The document must contain only the fields that needs to be projected. The field values must be `null` or a nested document. The condition holds true for nested documents as well.

Let's say you have a document like this:

```json
{
    "firstName": "John",
    "lastName": "Doe",
    "address": {
        "street": "123 Main Street",
        "city": "New York"
    }
}
```

And you want to project only `lastName` and `address.street` fields. Then you can do it like this:

```dart
var cursor = collection.find(where("firstName").eq("John"));

var projection = createDocument("lastName", null)
    ..put("address", createDocument("street", null));

var projectedCursor = cursor.project(projection);
```

The result set will contain only `lastName` and `address.street` fields.

```json
[
    {
        "lastName": "Doe",
        "address": {
            "street": "123 Main Street"
        }
    }
]
```

### Join

You can join two `DocumentCursor` using `leftJoin()` method. It takes another `DocumentCursor` as input parameter along with a `Lookup` object. It returns a `Stream<Document>` object.

The join operation is similar to SQL left outer join operation. It takes two cursors and a lookup object. The lookup object contains the join condition.

Let's say you have two collections `users` and `orders`. The `users` collection contains the following documents:

```json
{
    "userId": 1,
    "firstName": "John",
    "lastName": "Doe"
}
```

```json
{
    "userId": 2,
    "firstName": "Jane",
    "lastName": "Doe"
}
```

And the `orders` collection contains the following documents:

```json
{
    "orderId": 1,
    "userId": 1,
    "orderDate": "2018-01-01"
}
```

```json
{
    "orderId": 2,
    "userId": 1,
    "orderDate": "2018-01-02"
}
```

```json
{
    "orderId": 3,
    "userId": 2,
    "orderDate": "2018-01-03"
}
```

```json
{
    "orderId": 4,
    "userId": 2,
    "orderDate": "2018-01-04"
}
```

And you want to join these two collections on `userId` field. Then you can do it like this:

```dart
var userCollection = await db.collection("users");
var orderCollection = await db.collection("orders");

var users = userCollection.find();
var orders = orderCollection.find();

var lookup = Lookup();
lookup.setLocalField("userId");
lookup.setForeignField("userId");
lookup.setTargetField("orders");

var joinedCursor = users.leftJoin(orders, lookup);
```

The result set will contain the following documents:

```json
{
    "userId": 1,
    "firstName": "John",
    "lastName": "Doe",
    "orders": [
        {
            "orderId": 1,
            "userId": 1,
            "orderDate": "2018-01-01"
        },
        {
            "orderId": 2,
            "userId": 1,
            "orderDate": "2018-01-02"
        }
    ]
}
```

```json
{
    "userId": 2,
    "firstName": "Jane",
    "lastName": "Doe",
    "orders": [
        {
            "orderId": 3,
            "userId": 2,
            "orderDate": "2018-01-03"
        },
        {
            "orderId": 4,
            "userId": 2,
            "orderDate": "2018-01-04"
        }
    ]
}
```

### FindPlan

`FindPlan` is a class that contains the execution plan of a find operation. It has the following properties:

- `byIdFilter`: It contains the filter for finding a document using `NitriteId`.
- `indexScanFilter`: It contains the filter for finding a document using index.
- `collectionScanFilter`: It contains the filter for finding a document using full scan.
- `indexDescriptor`: It contains the index descriptor for finding a document using index.
- `indexScanOrder`: It contains the sort order for finding a document using index.
- `blockingSortOrder`: It contains the sort order for finding a document using full scan.
- `skip`: It contains the number of documents to be skipped from the beginning of the result set.
- `limit`: It contains the maximum number of documents to be returned by the find operation.
- `distinct`: It specifies if the find operation returns distinct documents. 
- `subPlans`: It contains the sub plans for finding a document using `or` filter.

You can get the execution plan of a find operation using `findPlan` getter. It returns a `Future<FindPlan>` object.

```dart
var cursor = collection.find(where("firstName").eq("John"));

var findPlan = await cursor.findPlan;
```



