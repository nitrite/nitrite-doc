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

```java
DocumentCursor cursor = collection.find();
```

### Finding a Document Using NitriteId

You can find a document using it's `NitriteId` by calling `getById()`. It takes a `NitriteId` as input parameter. It returns a `Document` object if the document is found. Otherwise, it returns `null`.

```java
Document doc = collection.getById(someNitriteId);
```

### Finding a Document Using a Filter

You can find a document using a filter. It takes a `Filter` object as input parameter. It returns a `DocumentCursor` object. 

If there is an index on the field specified in the filter, this operation will use the index to find the document. Otherwise, it will scan the entire collection to find the document.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));
```

### Finding a Document Using a Filter and Options

You can find a document using a filter and options. It takes a `Filter` object as the first input parameter. It takes a `FindOptions` object as the second input parameter. It returns a `DocumentCursor` object.

#### FindOptions

`FindOptions` is a class that contains several options for find operation. It has the following options:

- `limit`: It specifies the maximum number of documents to be returned by the find operation.
- `skip`: It specifies the number of documents to be skipped from the beginning of the result set.
- `orderBy`: It specifies a collection of fields to be sorted by, along with sort order for each field. The sort order can be `SortOrder.Ascending` or `SortOrder.Descending`.
- `collator`: It specifies a collator to be used for sorting. If this option is not specified, then the default collator will be used.
- `distinct`: It specifies if the find operation should return distinct documents. If this option is `true`, then it will return only the distinct documents. Otherwise, it will return all the documents matched by the filter.

```java
FindOptions findOptions = new FindOptions();
findOptions.skip(10).limit(10).thenOrderBy("firstName", SortOrder.Descending).withDistinct(true);

DocumentCursor cursor = collection.find(where("firstName").eq("John"), findOptions);
```

## DocumentCursor

`DocumentCursor` represents a result set of a find operation. It provides methods to iterate over the result of a find operation and retrieve the documents. It also provides methods like projection, join etc. to get the desired result.

### Iterating over Documents

The `DocumentCursor` extends `Iterable` interface. So, you can iterate over the documents using `for-each` loop.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

for (Document doc : cursor) {
    // do something with the document
}
```

!!!info
A `DocumentCursor` is a lazy iterable. It does not load all the documents in memory at once. It loads the documents in memory as needed. So, it is memory efficient.
!!!


### Getting the Documents

You can get the documents at once using `toList()` and `toSet()` methods. It returns a `List` and `Set` of documents respectively.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

List<Document> documents = cursor.toList();
Set<Document> documents = cursor.toSet();
```

### Getting the First Document

You can get the first document using `firstOrNull()` method. It returns the first document if the cursor has any document. Otherwise, it returns `null`.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

Document doc = cursor.firstOrNull();
```

### Getting the Size of the Result Set

You can get the size of the result set using `size()` method. It returns the number of documents in the result set.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

long size = cursor.size();
```

### Projection

You can project the result set using `project()` method. It takes a `Document` as the only input parameter. It returns a `RecordStream<Document>` object.

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

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

Document projection = Document.createDocument("lastName", null).put("address", createDocument("street", null));

RecordStream<Document> projectedStream = cursor.project(projection);
```

The result set will contain only `lastName` and `address.street` fields.

```json
{
    "lastName": "Doe",
    "address": {
        "street": "123 Main Street"
    }
}
```

### Join

You can join two cursors using `join()` method. It takes a `DocumentCursor` as the first input parameter. It takes a `Lookup` as the second input parameter. It returns a `RecordStream` object.

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

Now, you want to join these two collections on `userId` field. Then you can do it like this:

```java
DocumentCursor users = db.getCollection("users").find();
DocumentCursor orders = db.getCollection("orders").find();

Lookup lookup = new Lookup();
lookup.setLocalField("userId");
lookup.setForeignField("userId");
lookup.setTargetField("orders");

RecordStream<Document> joinedStream = users.join(orders, lookup);
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
- `collator`: It specifies a collator to be used for sorting.
- `subPlans`: It contains the sub plans for finding a document using `or` filter.

You can get the execution plan of a find operation using `getFindPlan()` method. It returns a `FindPlan` object.

```java
DocumentCursor cursor = collection.find(where("firstName").eq("John"));

FindPlan findPlan = cursor.getFindPlan();
```
