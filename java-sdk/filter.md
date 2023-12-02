---
label: Filters
icon: filter
order: 15
---

Filters are used to specify the criteria used to select documents from a collection or repository. It provides a way to specify conditions that the documents must meet to be included in the result set. Filters are used in conjunction with the `find` method. The `find` method returns all documents in a collection that match the specified filtering criteria.

Each filtering criteria is based on a field in the document. If the field is indexed, the find operation takes advantage of the index to speed up the operation. If the field is not indexed, the find operation scans the entire collection to find matching documents.

## Usage

Filters are used with the `find` method of `NitriteCollection` or `ObjectRepository` to retrieve documents that match the filter. Here's an example:

```java
// find all documents where the field 'name' equals 'John'
DocumentCursor cursor = collection.find(where("name").eq("John"));
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The filter is then passed to the find method to retrieve the matching documents.

## Types of Filters

Nitrite filters can be grouped into the following categories:

- Comparison filters
- Logical filters
- Array filters
- Evaluation filters
- Geospatial filters


## Comparison Filters

Comparison filters are used to compare a field with a value. The following comparison filters are supported:

- Equality (eq): Matches all documents where the value of the field equals the specified value.
- Inequality (notEq): Matches all documents where the value of the field is not equal to the specified value.
- Greater than (gt): Matches all documents where the value of the field is greater than the specified value.
- Greater than or equal to (gte): Matches all documents where the value of the field is greater than or equal to the specified value.
- Less than (lt): Matches all documents where the value of the field is less than the specified value.
- Less than or equal to (lte): Matches all documents where the value of the field is less than or equal to the specified value.
- In (in): Matches all documents where the value of the field equals any value in the specified array.
- Not in (notIn): Matches all documents where the value of the field does not equal any value in the specified array.
- Between (between): Matches all documents where the value of the field is between the specified values.
- All (all): Matches all documents in the collection.

### Equality Filter

The equality filter is used to match documents where the value of a field equals the specified value. The following example shows how to use the equality filter:

```java
// find all documents where the field 'name' equals 'John'
collection.find(where("name").eq("John"));
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The filter is then passed to the find method to retrieve the matching documents.

### Inequality Filter

The inequality filter is used to match documents where the value of a field is not equal to the specified value. The following example shows how to use the inequality filter:

```java
// find all documents where the field 'name' does not equal 'John'
collection.find(where("name").notEq("John"));
```

In this example, the `where` and `notEq` methods are used to create an inequality filter that matches documents where the `name` field does not equal "John". The filter is then passed to the find method to retrieve the matching documents.

### Greater Than Filter

The greater than filter is used to match documents where the value of a field is greater than the specified value. The following example shows how to use the greater than filter:

```java
// find all documents where the field 'age' is greater than 18
collection.find(where("age").gt(18));
```

In this example, the `where` and `gt` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

### Greater Than or Equal To Filter

The greater than or equal to filter is used to match documents where the value of a field is greater than or equal to the specified value. The following example shows how to use the greater than or equal to filter:

```java
// find all documents where the field 'age' is greater than or equal to 18
collection.find(where("age").gte(18));
```

In this example, the `where` and `gte` methods are used to create a greater than or equal to filter that matches documents where the `age` field is greater than or equal to 18. The filter is then passed to the find method to retrieve the matching documents.

### Less Than Filter

The less than filter is used to match documents where the value of a field is less than the specified value. The following example shows how to use the less than filter:

```java
// find all documents where the field 'age' is less than 18
collection.find(where("age").lt(18));
```

In this example, the `where` and `lt` methods are used to create a less than filter that matches documents where the `age` field is less than 18. The filter is then passed to the find method to retrieve the matching documents.

### Less Than or Equal To Filter

The less than or equal to filter is used to match documents where the value of a field is less than or equal to the specified value. The following example shows how to use the less than or equal to filter:

```java
// find all documents where the field 'age' is less than or equal to 18
collection.find(where("age").lte(18));
```

In this example, the `where` and `lte` methods are used to create a less than or equal to filter that matches documents where the `age` field is less than or equal to 18. The filter is then passed to the find method to retrieve the matching documents.

### In Filter

The in filter is used to match documents where the value of a field equals any value in the specified array. The following example shows how to use the in filter:

```java
// find all documents where the field 'name' equals 'John' or 'Jane'
collection.find(where("name").in("John", "Jane"));
```

In this example, the `where` and `in` methods are used to create an in filter that matches documents where the `name` field equals "John" or "Jane". The filter is then passed to the find method to retrieve the matching documents.

### Not In Filter

The not in filter is used to match documents where the value of a field does not equal any value in the specified array. The following example shows how to use the not in filter:

```java
// find all documents where the field 'name' does not equal 'John' or 'Jane'
collection.find(where("name").notIn("John", "Jane"));
```

In this example, the `where` and `notIn` methods are used to create a not in filter that matches documents where the `name` field does not equal "John" or "Jane". The filter is then passed to the find method to retrieve the matching documents.

### Between Filter

The between filter is used to match documents where the value of a field is between the specified values. The following example shows how to use the between filter:

```java
// find all documents where the field 'age' is between 18 and 30
collection.find(where("age").between(18, 30));
```

In this example, the `where` and `between` methods are used to create a between filter that matches documents where the `age` field is between 18 and 30. The filter is then passed to the find method to retrieve the matching documents.

There are three overloaded versions of the between filter. The first version takes two parameters, the lower and upper bounds of the range. The second version takes an additional boolean parameter that indicates whether both the lower and upper bounds are inclusive or exclusive. The third version takes two additional boolean parameters that indicate whether the lower and upper bounds are inclusive or exclusive. The following example shows how to use the second version of the between filter:

```java
// find all documents where the field 'age' is between 18 and 30
// both the lower and upper bounds are inclusive
collection.find(where("age").between(18, 30, true));
```

In this example, the `where` and `between` methods are used to create a between filter that matches documents where the `age` field is between 18 and 30. Both the lower and upper bounds are inclusive. The filter is then passed to the find method to retrieve the matching documents.

The following example shows how to use the third version of the between filter:

```java
// find all documents where the field 'age' is between 18 and 30
// the lower bound is inclusive and the upper bound is exclusive
collection.find(where("age").between(18, 30, true, false));
```

In this example, the `where` and `between` methods are used to create a between filter that matches documents where the `age` field is between 18 and 30. The lower bound is inclusive and the upper bound is exclusive. The filter is then passed to the find method to retrieve the matching documents.

### All Filter

The all filter is used to match all documents in the collection. The following example shows how to use the all filter:

```java
// find all documents in the collection
collection.find(Filter.ALL);
```

In this example, the `Filter.ALL` constant is used to create an all filter that matches all documents in the collection. The filter is then passed to the find method to retrieve the matching documents.



## Logical Filters

Logical filters are used to combine multiple filters into a single filter. The following logical filters are supported:

- And (and): Matches all documents that satisfy all the specified filters.
- Or (or): Matches all documents that satisfy at least one of the specified filters.
- Not (not): Matches all documents that do not satisfy the specified filter.


### And Filter

The and filter is used to match documents that satisfy all the specified filters. The following example shows how to use the and filter:

```java
// find all documents where the field 'name' equals 'John' and the field 'age' is greater than 18
collection.find(where("name").eq("John").and(where("age").gt(18)));
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `where` and `gt` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The two filters are then combined using the `and` method to create an and filter that matches documents where the `name` field equals "John" and the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

The `and` method can be used to combine any number of filters. The following example shows how to combine three filters:

```java
// find all documents where the field 'name' equals 'John' and the field 'age' is greater than 18 and less than 30
collection.find(where("name").eq("John").and(where("age").gt(18)).and(where("age").lt(30)));
```

The static `Filter.and` method can also be used to combine multiple filters into a single filter. The following example shows how to combine two filters into a single filter:

```java
// find all documents where the field 'name' equals 'John' and the field 'age' is greater than 18
// and the field 'age' is less than 30
collection.find(and(where("name").eq("John"), where("age").gt(18),where("age").lt(30)));
```

In this example, the `and` method is used to combine three filters into a single filter. The filter is then passed to the find method to retrieve the matching documents.

### Or Filter

The or filter is used to match documents that satisfy at least one of the specified filters. The following example shows how to use the or filter:

```java
// find all documents where the field 'name' equals 'John' or the field 'age' is greater than 18
collection.find(where("name").eq("John").or(where("age").gt(18)));
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `where` and `gt` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The two filters are then combined using the `or` method to create an or filter that matches documents where the `name` field equals "John" or the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

The `or` method can be used to combine any number of filters. The following example shows how to combine three filters:

```java
// find all documents where the field 'name' equals 'John' or the field 'age' is greater than 18 or less than 30
collection.find(where("name").eq("John").or(where("age").gt(18)).or(where("age").lt(30)));
```

The static `Filter.or` method can also be used to combine multiple filters into a single filter. The following example shows how to combine two filters into a single filter:

```java
// find all documents where the field 'name' equals 'John' or the field 'age' is greater than 18
// or the field 'age' is less than 30
collection.find(or(where("name").eq("John"), where("age").gt(18),where("age").lt(30)));
```

In this example, the `or` method is used to combine three filters into a single filter. The filter is then passed to the find method to retrieve the matching documents.

### Not Filter

The not filter is used to match documents that do not satisfy the specified filter. The following example shows how to use the not filter:

```java
// find all documents where the field 'name' does not equal 'John'
collection.find(where("name").eq("John").not());
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `not` method is then used to create a not filter that matches documents where the `name` field does not equal "John". The filter is then passed to the find method to retrieve the matching documents.


## Array Filters

Array filters are used to match documents based on the values in an array field. The following array filters are supported:

- Element match (elemMatch): Matches all documents where the array field contains at least one element that matches the specified filter.

### Element Match Filter

The element match filter is used to match documents where the array field contains at least one element that matches the specified filter. The following example shows how to use the element match filter:

Let's say we have a collection that contains documents with the following structure:

```json
{
  "name": "John",
  "addresses": [
    {
      "city": "New York",
      "state": "NY"
    },
    {
      "city": "Los Angeles",
      "state": "CA"
    }
  ]
}
```

The following example shows how to use the element match filter to find all documents where the `addresses` field contains at least one element that matches the filter where the `city` field equals "New York":

```java
// find all documents where the field 'addresses' contains at least one element that matches the filter
// where the field 'city' equals 'New York'
collection.find(where("addresses").elemMatch(where("city").eq("New York")));
```

In this example, the `where` and `elemMatch` methods are used to create an element match filter that matches documents where the `addresses` field contains at least one element that matches the filter where the `city` field equals "New York". The filter is then passed to the find method to retrieve the matching documents.

The `elemMatch` filter can be used to search an array of elements for a specific value. The following example shows how to use the `elemMatch` filter to find all documents where the `data` field contains at least one element that is greater than 2 or less than equal to 5:

```java
// find all documents where the field 'data' contains at least one element that is greater than 2 or less than equal to 5
collection.find(where("data").elemMatch($.gt(2).or($.lte(5))));
```

The static `FluentFilter.$` is used to refer to the current element in the array. The `$.gt(2).or($.lte(5))` filter is used to match documents where the `data` field contains at least one element that is greater than 2 or less than equal to 5. The filter is then passed to the find method to retrieve the matching documents.


## Evaluation Filters

Evaluation filters are used to match documents based on evaluating the value of any field in a document. The following evaluation filters are supported:

- Text (text): Matches all documents which contain a specified full-text search expression.
- Regex (regex): Matches all documents where values contain a specified regular expression.

### Text Filter

The text filter is used to match documents which contain a specified full-text search expression. The following example shows how to use the text filter:

```java
// find all documents where the field 'name' contains the word 'John'
collection.find(where("name").text("John"));
```

In this example, the `where` and `text` methods are used to create a text filter that matches documents where the `name` field contains the word "John". The filter is then passed to the find method to retrieve the matching documents.

!!!primary Important
The text filter can only be used with a field that has a full-text index.
!!!

### Regex Filter

The regex filter is used to match documents where values contain a specified regular expression. The following example shows how to use the regex filter:

```java
// find all documents where the field 'name' matches the regular expression 'John.*'
collection.find(where("name").regex("John.*"));
```

In this example, the `where` and `regex` methods are used to create a regex filter that matches documents where the `name` field matches the regular expression "John.*". The filter is then passed to the find method to retrieve the matching documents.

!!!primary Important
This filter scans the entire collection to find matching documents. It cannot take advantage of an index.
!!!

## Geospatial Filters

Geospatial filters are used to match documents based on the values in a geospatial field. The following geospatial filters are supported:

- Near (near): Matches all documents where the geospatial field is within the specified distance from the specified point.
- Within (within): Matches all documents where the geospatial field is within the specified shape.
- Intersects (intersects): Matches all documents where the geospatial field intersects the specified shape.

To use geospatial filters, you need create a geospatial index on the field, which needs the `nitrite-spatial` module to be loaded. More on this can be found [here](modules/spatial.md).

### Near Filter

The near filter is used to match documents where the value of the geospatial field is near the specified point. The following example shows how to use the near filter:

```java
// find all documents where the field 'location' is within 1000 meters of the point (40.730610, -73.935242)
WKTReader reader = new WKTReader();
Point search = (Point) reader.read("POINT (40.730610, -73.935242)");
Coordinate coordinate = search.getCoordinate();

collection.find(where("location").near(coordinate, 1000));
```

In this example, the `where` and `near` methods are used to create a near filter that matches documents where the `location` field is within 1000 meters of the point (40.730610, -73.935242). The filter is then passed to the find method to retrieve the matching documents.


### Within Filter

The within filter is used to match documents where the value of the geospatial field is within the specified shape. The following example shows how to use the within filter:

```java
// find all documents where the field 'location' is within the polygon ((0 0, 0 3, 3 3, 3 0, 0 0))
WKTReader reader = new WKTReader();
Geometry search = reader.read("POLYGON ((0 0, 0 3, 3 3, 3 0, 0 0))");

collection.find(where("location").within(search));
```

In this example, the `where` and `within` methods are used to create a within filter that matches documents where the `location` field is within the polygon ((0 0, 0 3, 3 3, 3 0, 0 0)). The filter is then passed to the find method to retrieve the matching documents.

### Intersects Filter

The intersects filter is used to match documents where the value of the geospatial field intersects the specified shape. The following example shows how to use the intersects filter:

```java
// find all documents where the field 'location' intersects the polygon ((0 0, 0 3, 3 3, 3 0, 0 0))
WKTReader reader = new WKTReader();
Geometry search = reader.read("POLYGON ((0 0, 0 3, 3 3, 3 0, 0 0))");

collection.find(where("location").intersects(search));
```

In this example, the `where` and `intersects` methods are used to create an intersects filter that matches documents where the `location` field intersects the polygon ((0 0, 0 3, 3 3, 3 0, 0 0)). The filter is then passed to the find method to retrieve the matching documents.

## Fluent API

Nitrite provides a fluent API to create filters via `FluentFilter` class. It provides a static `where` method that creates a new `FluentFilter` instance with the specified field name. Here's an example:

```java
// create a filter for the field 'name'
FluentFilter filter = FluentFilter.where("name").eq("John");
DocumentCursor cursor = collection.find(filter);
```

In this example, the `FluentFilter.where` method is used to create a filter that matches documents where the `name` field equals "John". The filter is then passed to the find method to retrieve the matching documents.

For geospatial filters, use the `SpatialFluentFilter` class for the fluent API. It provides a static `where` method that creates a new `SpatialFluentFilter` instance with the specified field name. Here's an example:

```java
// create a filter for the field 'location'
SpatialFilter filter = SpatialFluentFilter.where("location").near(coordinate, 1000);
DocumentCursor cursor = collection.find(filter);
```
