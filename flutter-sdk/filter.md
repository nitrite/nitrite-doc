---
label: Filters
icon: filter
order: 15
---

Filters are used to specify the criteria used to select documents from a collection or repository. It provides a way to specify conditions that the documents must meet to be included in the result set. Filters are used in conjunction with the `find` method. The `find` method returns all documents in a collection that match the specified filtering criteria.

Each filtering criteria is based on a field in the document. If the field is indexed, the find operation takes advantage of the index to speed up the operation. If the field is not indexed, the find operation scans the entire collection to find matching documents.

## Fluent API

Nitrite provides a fluent API to create filters via `FluentFilter` class. It provides a global method `where` to create a filter. The `where` method takes a field name as input parameter and returns a `FluentFilter` object. The `FluentFilter` object provides various methods to create a filter. Here is an example:

```dart
var clause = where('name');

// filter for equality
var filter = clause.eq('John');

var cursor = collection.find(filter: filter);
```

The above code will create a filter for the field `name` and then filter for equality. The `find` method will return all documents where the field `name` is equal to `John`. 

For spatial filters, use the `SpatialFluentFilter` class for the fluent API. Similarly it also provides a global method `where` to create a filter. The `where` method takes a field name as input parameter and returns a `SpatialFluentFilter` object. The `SpatialFluentFilter` object provides various methods to create a filter. Here is an example:

```dart
import 'package:nitrite_spatial/nitrite_spatial.dart';

var clause = where('location');

// filter for within
var filter = clause.within(polygon);

var cursor = collection.find(filter: filter);
```

More on spatial module can be found [here](modules/spatial.md).

## Usage

Filters are used with the `find` method of `NitriteCollection` or `ObjectRepository` to retrieve documents or entities that match the filter. Here's an example:

```dart
// find all documents where the field 'name' equals 'John'
var cursor = collection.find(filter: where('name').eq('John'));
```

In this example, the `where` and `eq` methods are used to create an equality filter that matches documents where the `name` field equals "John". The filter is then passed to the find method to retrieve the matching documents.

### String Extensions

Nitrite provides a `String` extension to create filters. It creates a filter for the field name specified in the string.

```dart
// find all documents where the field 'name' equals 'John'
var cursor = collection.find(filter: 'name'.eq('John'));
```

### Operator Overloading

Nitrite provides operator overloading to create filters. It creates a filter for the field name specified in the string.

```dart
// find all documents where the field 'age' is greater than 18
var cursor = collection.find(filter: where('age') > 18);

// or using string extension
var cursor = collection.find(filter: 'age' > 18);
```

There are various operators available for creating filters. Here is the list:

- `>`: Greater than filter
- `>=`: Greater than or equal to filter
- `<`: Less than filter
- `<=`: Less than or equal to filter
- `~`: Not filter
- `&`: And filter
- `|`: Or filter

These operators are also valid on the `String` extension of filter. More on this is discussed in the following sections.

## Types of Filters

Nitrite filters can be grouped into the following categories:

- Comparison filters
- Logical filters
- Array filters
- Evaluation filters
- Spatial filters

## Comparison Filters

Comparison filters are used to compare a field with a value. The following comparison filters are available:

- Equality (eq): Matches all documents where the value of the field equals the specified value.
- Inequality (notEq): Matches all documents where the value of the field is not equal to the specified value.
- Greater than (gt): Matches all documents where the value of the field is greater than the specified value.
- Greater than or equal to (gte): Matches all documents where the value of the field is greater than or equal to the specified value.
- Less than (lt): Matches all documents where the value of the field is less than the specified value.
- Less than or equal to (lte): Matches all documents where the value of the field is less than or equal to the specified value.
- Within (within): Matches all documents where the value of the field equals any value in the specified array.
- Not in (notIn): Matches all documents where the value of the field does not equal any value in the specified array.
- Between (between): Matches all documents where the value of the field is between the specified values.
- All (all): Matches all documents in the collection.

### Equality Filter

The equality filter is used to match documents where the value of a field equals the specified value. The following example shows how to use the equality filter:

```dart
// find all documents where the field 'name' equals 'John'
collection.find(filter: where('name').eq('John'));

// or using string extension
collection.find(filter: 'name'.eq('John'));
```

In this example, the `where()` and `eq()` methods are used to create an equality filter that matches documents where the `name` field equals "John". The filter is then passed to the find method to retrieve the matching documents.

### Inequality Filter

The inequality filter is used to match documents where the value of a field is not equal to the specified value. The following example shows how to use the inequality filter:

```dart
// find all documents where the field 'name' is not equal to 'John'
collection.find(filter: where('name').notEq('John'));

// or using string extension
collection.find(filter: 'name'.notEq('John'));
```

In this example, the `where()` and `notEq()` methods are used to create an inequality filter that matches documents where the `name` field does not equal "John". The filter is then passed to the find method to retrieve the matching documents.

### Greater Than Filter

The greater than filter is used to match documents where the value of a field is greater than the specified value. The following example shows how to use the greater than filter:

```dart
// find all documents where the field 'age' is greater than 18
collection.find(filter: where('age').gt(18));

// or using string extension
collection.find(filter: 'age'.gt(18));

// or using operator overloading
collection.find(filter: where('age') > 18);

// or using operator overloading with string extension
collection.find(filter: 'age' > 18);
```

In this example, the `where()` and `gt()` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

### Greater Than or Equal To Filter

The greater than or equal to filter is used to match documents where the value of a field is greater than or equal to the specified value. The following example shows how to use the greater than or equal to filter:

```dart
// find all documents where the field 'age' is greater than or equal to 18
collection.find(filter: where('age').gte(18));

// or using string extension
collection.find(filter: 'age'.gte(18));

// or using operator overloading
collection.find(filter: where('age') >= 18);

// or using operator overloading with string extension
collection.find(filter: 'age' >= 18);
```

In this example, the `where()` and `gte()` methods are used to create a greater than or equal to filter that matches documents where the `age` field is greater than or equal to 18. The filter is then passed to the find method to retrieve the matching documents.

### Less Than Filter

The less than filter is used to match documents where the value of a field is less than the specified value. The following example shows how to use the less than filter:
    
```dart
// find all documents where the field 'age' is less than 18
collection.find(filter: where('age').lt(18));

// or using string extension
collection.find(filter: 'age'.lt(18));

// or using operator overloading
collection.find(filter: where('age') < 18);

// or using operator overloading with string extension
collection.find(filter: 'age' < 18);
```

In this example, the `where()` and `lt()` methods are used to create a less than filter that matches documents where the `age` field is less than 18. The filter is then passed to the find method to retrieve the matching documents.

### Less Than or Equal To Filter

The less than or equal to filter is used to match documents where the value of a field is less than or equal to the specified value. The following example shows how to use the less than or equal to filter:

```dart
// find all documents where the field 'age' is less than or equal to 18
collection.find(filter: where('age').lte(18));

// or using string extension
collection.find(filter: 'age'.lte(18));

// or using operator overloading
collection.find(filter: where('age') <= 18);

// or using operator overloading with string extension
collection.find(filter: 'age' <= 18);
```

In this example, the `where()` and `lte()` methods are used to create a less than or equal to filter that matches documents where the `age` field is less than or equal to 18. The filter is then passed to the find method to retrieve the matching documents.

### Within Filter

The within filter is used to match documents where the value of a field equals any value in the specified array. The following example shows how to use the in filter:

```dart
// find all documents where the field 'name' equals 'John' or 'Jane'
collection.find(filter: where('name').within(['John', 'Jane']));

// or using string extension
collection.find(filter: 'name'.within(['John', 'Jane']));
```

In this example, the `where()` and `within()` methods are used to create an in filter that matches documents where the `name` field equals "John" or "Jane". The filter is then passed to the find method to retrieve the matching documents.

### Not In Filter

The not in filter is used to match documents where the value of a field does not equal any value in the specified array. The following example shows how to use the not in filter:
    
```dart
// find all documents where the field 'name' does not equal 'John' or 'Jane'
collection.find(filter: where('name').notIn(['John', 'Jane']));

// or using string extension
collection.find(filter: 'name'.notIn(['John', 'Jane']));
```

In this example, the `where()` and `notIn()` methods are used to create a not in filter that matches documents where the `name` field does not equal "John" or "Jane". The filter is then passed to the find method to retrieve the matching documents.

### Between Filter

The between filter is used to match documents where the value of a field is between the specified values. The following example shows how to use the between filter:

```dart
// find all documents where the field 'age' is between 18 and 30
collection.find(filter: where('age').between(18, 30));

// or using string extension
collection.find(filter: 'age'.between(18, 30));
```

In this example, the `where()` and `between()` methods are used to create a between filter that matches documents where the `age` field is between 18 and 30. The filter is then passed to the find method to retrieve the matching documents.

There two other optional parameters available in the `between` method. The first parameter is `lowerInclusive` which is a boolean value to indicate whether the lower bound is inclusive or not. The default value is `true`. The second parameter is `upperInclusive` which is a boolean value to indicate whether the upper bound is inclusive or not. The default value is `true`.

```dart
// find all documents where the field 'age' is between 18 and 30
// both lower and upper bounds are inclusive
collection.find(filter: where('age').between(18, 30));

// find all documents where the field 'age' is between 18 and 30
// lower bound is exclusive and upper bound is inclusive
collection.find(filter: where('age').between(18, 30, lowerInclusive: false, upperInclusive: true));

// find all documents where the field 'age' is between 18 and 30
// both lower and upper bounds are exclusive
collection.find(filter: where('age').between(18, 30, lowerInclusive: false, upperInclusive: false));

// or using string extension
// find all documents where the field 'age' is between 18 and 30
// both lower and upper bounds are inclusive
collection.find(filter: 'age'.between(18, 30));

// find all documents where the field 'age' is between 18 and 30
// lower bound is exclusive and upper bound is inclusive
collection.find(filter: 'age'.between(18, 30, lowerInclusive: false, upperInclusive: true));

// find all documents where the field 'age' is between 18 and 30
// both lower and upper bounds are exclusive
collection.find(filter: 'age'.between(18, 30, lowerInclusive: false, upperInclusive: false));
```

### All Filter

The all filter is used to match all documents in the collection. The following example shows how to use the all filter:

```dart
// find all documents in the collection
collection.find(filter: all);
```

In this example, the `all` constant is used to create an all filter that matches all documents in the collection. The filter is then passed to the find method to retrieve the matching documents.

## Logical Filters

Logical filters are used to combine multiple filters into a single filter. The following logical filters are supported:

- And (and): Matches all documents that satisfy all the specified filters.
- Or (or): Matches all documents that satisfy at least one of the specified filters.
- Not (not): Matches all documents that do not satisfy the specified filter.

### And Filter

The and filter is used to match documents that satisfy all the specified filters. The following example shows how to use the and filter:

```dart
// find all documents where the field 'name' equals 'John' and 'age' is greater than 18
collection.find(filter: where('name').eq('John').and('age').gt(18));

// or using string extension and operator overloading
collection.find(filter: 'name'.eq('John') & 'age'.gt(18));
```

In this example, the `where()` and `eq()` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `where()` and `gt()` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The two filters are then combined using the `and()` method to create an and filter that matches documents where the `name` field equals "John" and the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

The `and()` method can be used to combine any number of filters. The following example shows how to combine three filters:

```dart
// find all documents where the field 'name' equals 'John' and 'age' is greater than 18 and less than 30
collection.find(filter: where('name').eq('John').and('age').gt(18).and('age').lt(30));
```

Operators can also be used to combine filters. The following example shows how to combine three filters using operators:

```dart
// find all documents where the field 'name' equals 'John' and 'age' is greater than 18 and less than 30
collection.find(filter: 'name'.eq('John') & (('age' > 18) & ('age' < 30)));
```

The global `and()` method can also be used to combine filters. The following example shows how to combine three filters using the global `and()` method:

```dart
// find all documents where the field 'name' equals 'John' and 'age' is greater than 18 and less than 30
collection.find(filter: and(where('name').eq('John'), where('age').gt(18), where('age').lt(30)));
```

In this example, the `and()` method is used to combine three filters into a single filter. The filter is then passed to the find method to retrieve the matching documents.

### Or Filter

The or filter is used to match documents that satisfy at least one of the specified filters. The following example shows how to use the or filter:

```dart
// find all documents where the field 'name' equals 'John' or 'age' is greater than 18
collection.find(filter: where('name').eq('John').or('age').gt(18));

// or using string extension and operator overloading
collection.find(filter: 'name'.eq('John') | 'age'.gt(18));
```

In this example, the `where()` and `eq()` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `where()` and `gt()` methods are used to create a greater than filter that matches documents where the `age` field is greater than 18. The two filters are then combined using the `or()` method to create an or filter that matches documents where the `name` field equals "John" or the `age` field is greater than 18. The filter is then passed to the find method to retrieve the matching documents.

The `or()` method can be used to combine any number of filters. The following example shows how to combine three filters:

```dart
// find all documents where the field 'name' equals 'John' or 'age' is greater than 18 or less than 30
collection.find(filter: where('name').eq('John').or('age').gt(18).or('age').lt(30));
```

Operators can also be used to combine filters. The following example shows how to combine three filters using operators:

```dart
// find all documents where the field 'name' equals 'John' or 'age' is greater than 18 or less than 30
collection.find(filter: 'name'.eq('John') | (('age' > 18) | ('age' < 30)));
```

The global `or()` method can also be used to combine filters. The following example shows how to combine three filters using the global `or()` method:

```dart
// find all documents where the field 'name' equals 'John' or 'age' is greater than 18 or less than 30
collection.find(filter: or(where('name').eq('John'), where('age').gt(18), where('age').lt(30)));
```

In this example, the `or()` method is used to combine three filters into a single filter. The filter is then passed to the find method to retrieve the matching documents.

### Not Filter

The not filter is used to match documents that do not satisfy the specified filter. The following example shows how to use the not filter:

```dart
// find all documents where the field 'name' does not equal 'John'
collection.find(filter: where('name').eq('John').not());

// or using operator overloading
collection.find(filter: ~('name'.eq('John')));
```

In this example, the `where()` and `eq()` methods are used to create an equality filter that matches documents where the `name` field equals "John". The `not()` method is then used to create a not filter that matches documents where the `name` field does not equal "John". The filter is then passed to the find method to retrieve the matching documents.

The `not()` method can be used to negate any filter. The following example shows how to negate a greater than filter:

```dart
// find all documents where the field 'age' is not greater than 18
collection.find(filter: where('age').gt(18).not());

// or using operator overloading
collection.find(filter: ~('age' > 18));
```

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

```dart
// find all documents where the field 'addresses' contains at least one element where the field 'city' equals 'New York'
collection.find(filter: where('addresses').elemMatch(where('city').eq('New York'))));
```

In this example, the `where()` and `elemMatch()` methods are used to create an element match filter that matches documents where the `addresses` field contains at least one element that matches the filter where the `city` field equals "New York". The filter is then passed to the find method to retrieve the matching documents.

The `elemMatch` filter can be used to search an array of elements for a specific value. The following example shows how to use the `elemMatch` filter to find all documents where the `data` field contains at least one element that is greater than 2 or less than equal to 5:

```dart
// find all documents where the field 'data' contains at least one element that is greater than 2 or less than equal to 5
collection.find(filter: where('data').elemMatch($.gt(2).or($.lte(5))));
```

The global `$` variable is used to create a filter for the current element in the array. The `$.gt(2).or($.lte(5))` filter is used to match documents where the `data` field contains at least one element that is greater than 2 or less than equal to 5. The filter is then passed to the find method to retrieve the matching documents.

## Evaluation Filters

Evaluation filters are used to match documents based on evaluating the value of any field in a document. The following evaluation filters are supported:

- Text (text): Matches all documents which contain a specified full-text search expression.
- Regex (regex): Matches all documents where values contain a specified regular expression.

### Text Filter

The text filter is used to match documents which contain a specified full-text search expression. The following example shows how to use the text filter:

```dart
// find all documents where the field 'name' contains the word 'John'
collection.find(filter: where('name').text('John'));
```

In this example, the `where()` and `text()` methods are used to create a text filter that matches documents where the `name` field contains the word "John". The filter is then passed to the find method to retrieve the matching documents.

The `text` filter supports glob patterns. The following example shows how to use the text filter with glob patterns:

```dart
// find all documents where the field 'name' starts with the word 'Jo'
collection.find(filter: where('name').text('Jo*'));

// find all documents where the field 'name' ends with the word 'hn'
collection.find(filter: where('name').text('*hn'));

// find all documents where the field 'name' contains the word 'oh'
collection.find(filter: where('name').text('*oh*'));
```

!!!info
The text filter can only be used with a field that has a full-text index.
!!!

### Regex Filter

The regex filter is used to match documents where values contain a specified regular expression. The following example shows how to use the regex filter:

```dart
// find all documents where the field 'name' contains the word 'John'
collection.find(filter: where('name').regex('John.*'));
```

In this example, the `where()` and `regex()` methods are used to create a regex filter that matches documents where the `name` field matches the regular expression "John.*". The filter is then passed to the find method to retrieve the matching documents.

!!!primary
This filter scans the entire collection to find matching documents. It cannot take advantage of an index.
!!!

## Spatial Filters

Spatial filters are used to match documents based on the values in a spatial field. The following spatial filters are supported:

- Near (near): Matches all documents where the spatial field is within the specified distance from the specified point.
- Within (within): Matches all documents where the spatial field is within the specified shape.
- Intersects (intersects): Matches all documents where the spatial field intersects the specified shape.

To use spatial filters, you need create a spatial index on the field, which needs the `nitrite-spatial` module to be loaded. More on this can be found [here](modules/spatial.md).

!!!info
Spatial filters can only be used with a field that has a spatial index.
!!!

### Near Filter

The near filter is used to match documents where the value of the spatial field is near the specified point. The following example shows how to use the near filter:

```dart
import 'package:nitrite_spatial/nitrite_spatial.dart';

// find all documents where the field 'location' is within 1000 meters of the point (40.730610, -73.935242)
var reader = WKTReader();
var point = reader.read('POINT(40.730610 -73.935242)') as Point;

collection.find(filter: where('location').near(Center.fromPoint(point), 1000));

// or using Center.fromCoordinates()
collection.find(filter: where('location').near(Center.fromCoordinate(point.getCoordinate()!), 1000));
```

In this example, the `where()` and `near()` methods are used to create a near filter that matches documents where the `location` field is within 1000 meters of the point (40.730610, -73.935242). The filter is then passed to the find method to retrieve the matching documents.

### Within Filter

The within filter is used to match documents where the value of the spatial field is within the specified shape. The following example shows how to use the within filter:

```dart
import 'package:nitrite_spatial/nitrite_spatial.dart';

// find all documents where the field 'location' is within the polygon ((0 0, 0 3, 3 3, 3 0, 0 0))
var reader = WKTReader();
var polygon = reader.read('POLYGON((0 0, 0 3, 3 3, 3 0, 0 0))') as Polygon;

collection.find(filter: where('location').within(polygon));
```

In this example, the `where()` and `within()` methods are used to create a within filter that matches documents where the `location` field is within the polygon ((0 0, 0 3, 3 3, 3 0, 0 0)). The filter is then passed to the find method to retrieve the matching documents.

### Intersects Filter

The intersects filter is used to match documents where the value of the spatial field intersects the specified shape. The following example shows how to use the intersects filter:

```dart
import 'package:nitrite_spatial/nitrite_spatial.dart';

// find all documents where the field 'location' intersects the polygon ((0 0, 0 3, 3 3, 3 0, 0 0))
var reader = WKTReader();
var polygon = reader.read('POLYGON((0 0, 0 3, 3 3, 3 0, 0 0))') as Polygon;

collection.find(filter: where('location').intersects(polygon));
```

In this example, the `where()` and `intersects()` methods are used to create an intersects filter that matches documents where the `location` field intersects the polygon ((0 0, 0 3, 3 3, 3 0, 0 0)). The filter is then passed to the find method to retrieve the matching documents.