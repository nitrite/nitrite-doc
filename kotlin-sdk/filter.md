---
label: Filters
icon: filter
order: 12
---

Filters are used to specify the criteria used to select documents from a collection or repository. It provides a way to specify conditions that the documents must meet to be included in the result set. Filters are used in conjunction with the `find` method. The `find` method returns all documents in a collection that match the specified filtering criteria.

More details on filters can be found [here](../java-sdk/filter.md).

## Kotlin API for Filters

Potassium Nitrite provides a set of infix functions to make filter usage more like operators. It also provides a set of extension functions for `KProperty` so that you can use property name instead of string in case of entity class.

## Comparison Operators

Comparison operators are used to compare two expressions. More details on comparison operators can be found [here](../java-sdk/filter.md#comparison-filters).

### Equal To

To check if a field value is equal to a value, you can use the `eq` function.

```kotlin
val filter = ("firstName" eq "John")
```

Similarly, you can use the `eq` function with `KProperty`.

```kotlin
val filter = (Employee::firstName eq "John")
```

More details on `eq` function can be found [here](../java-sdk/filter.md#equality-filter).

### Not Equal To

To check if a field value is not equal to a value, you can use the `notEq` function.

```kotlin
val filter = ("firstName" notEq "John")
```

Similarly, you can use the `notEq` function with `KProperty`.

```kotlin
val filter = (Employee::firstName notEq "John")
```

More details on `notEq` function can be found [here](../java-sdk/filter.md#inequality-filter).

### Greater Than

To check if a field value is greater than a value, you can use the `gt` function.

```kotlin
val filter = ("age" gt 18)
```

Similarly, you can use the `gt` function with `KProperty`.

```kotlin
val filter = (Employee::age gt 18)
```

More details on `gt` function can be found [here](../java-sdk/filter.md#greater-than-filter).

### Greater Than or Equal To

To check if a field value is greater than or equal to a value, you can use the `gte` function.

```kotlin
val filter = ("age" gte 18)
```

Similarly, you can use the `gte` function with `KProperty`.

```kotlin
val filter = (Employee::age gte 18)
```

More details on `gte` function can be found [here](../java-sdk/filter.md#greater-than-or-equal-to-filter).

### Less Than

To check if a field value is less than a value, you can use the `lt` function.

```kotlin
val filter = ("age" lt 18)
```

Similarly, you can use the `lt` function with `KProperty`.

```kotlin
val filter = (Employee::age lt 18)
```

More details on `lt` function can be found [here](../java-sdk/filter.md#less-than-filter).

### Less Than or Equal To

To check if a field value is less than or equal to a value, you can use the `lte` function.

```kotlin
val filter = ("age" lte 18)
```

Similarly, you can use the `lte` function with `KProperty`.

```kotlin
val filter = (Employee::age lte 18)
```

More details on `lte` function can be found [here](../java-sdk/filter.md#less-than-or-equal-to-filter).

### Within

To check if a field value is in a list of values, you can use the `within` function.

```kotlin
val filter = ("firstName" within arrayOf("John", "Jane"))
```

Similarly, you can use the `within` function with `KProperty`.

```kotlin
val filter = (Employee::firstName within listOf("John", "Jane"))
```

You can also use it with range.

```kotlin
val filter = ("age" within (18..65))
```

More details on `within` function can be found [here](../java-sdk/filter.md#in-filter).

### Not Within

To check if a field value is not in a list of values, you can use the `notWithin` function.

```kotlin
val filter = ("firstName" notWithin arrayOf("John", "Jane"))
```

Similarly, you can use the `notWithin` function with `KProperty`.

```kotlin
val filter = (Employee::firstName notWithin listOf("John", "Jane"))
```

You can also use it with range.

```kotlin
val filter = ("age" notWithin (18..65))
```

More details on `notWithin` function can be found [here](../java-sdk/filter.md#not-in-filter).

### Between

To check if a field value is between two values, you can use the `between` function. More details on `between` function can be found [here](../java-sdk/filter.md#between-filter).

```kotlin
val filter = ("age".between(18, 65))

// or
val filter = ("age".between (18, 65, true, true))
```

## Logical Filters

Logical operators are used to combine two or more expressions. More details on logical operators can be found [here](../java-sdk/filter.md#logical-filters).

### And

To combine two or more expressions with `AND` operator, you can use the `and` function.

```kotlin
val filter = (("firstName" eq "John") and ("lastName" eq "Doe"))
```

Similarly, you can use the `and` function with `KProperty`.

```kotlin
val filter = ((Employee::firstName eq "John") and (Employee::lastName eq "Doe"))
```

More details on `and` function can be found [here](../java-sdk/filter.md#and-filter).

### Or

To combine two or more expressions with `OR` operator, you can use the `or` function.

```kotlin

val filter = (("firstName" eq "John") or ("lastName" eq "Doe"))
```

Similarly, you can use the `or` function with `KProperty`.

```kotlin
val filter = ((Employee::firstName eq "John") or (Employee::lastName eq "Doe"))
```

More details on `or` function can be found [here](../java-sdk/filter.md#or-filter).

## Array Filters

Array filters are used to match documents based on the values in an array field. More details on array filters can be found [here](../java-sdk/filter.md#array-filters).

### Element Match

To check if an array field contains at least one element that matches the specified filter, you can use the `elemMatch` function.

```kotlin
val filter = ("phone" elemMatch ("$" eq "212-555-1234"))
```

Similarly, you can use the `elemMatch` function with `KProperty`.

```kotlin
val filter = (Employee::phone elemMatch ("$" eq "212-555-1234"))
```

More details on `elemMatch` function can be found [here](../java-sdk/filter.md#element-match-filter).

## Evaluation Filters

Evaluation filters are used to match documents based on evaluating the value of any field in a document. More details on evaluation filters can be found [here](../java-sdk/filter.md#evaluation-filters).

### Regex

To check if a field value matches a regular expression, you can use the `regex` function.

```kotlin
val filter = ("firstName" regex "^J.*")
```

Similarly, you can use the `regex` function with `KProperty`.

```kotlin
val filter = (Employee::firstName regex "^J.*")
```

More details on `regex` function can be found [here](../java-sdk/filter.md#regex-filter).

### Text

The text filter is used to match documents which contain a specified full-text search expression.

```kotlin
val filter = ("notes" text "John*")
```

Similarly, you can use the `text` function with `KProperty`.

```kotlin
val filter = (Employee::notes text "John*")
```

More details on `text` function can be found [here](../java-sdk/filter.md#text-filter).

## Spatial Filters

Spatial filters are used to match documents based on their geometric shape. More details on spatial filters can be found [here](../java-sdk/filter.md#spatial-filters).

### Near

To check if a field value is near to a point, you can use the `near` function.

```kotlin
val filter = ("location".near(reader.read("POINT (490 490)") as Point, 20.0))
```

Similarly, you can use the `near` function with `KProperty`.

```kotlin
val filter = (Employee::location.near(reader.read("POINT (490 490)") as Point, 20.0))
```

More details on `near` function can be found [here](../java-sdk/filter.md#near-filter).

### Within

To check if a field value is within a specified shape, you can use the `within` function.

```kotlin
val filter = ("location" within reader.read("POLYGON ((0 0, 0 1000, 1000 1000, 1000 0, 0 0))") as Polygon)
```

Similarly, you can use the `within` function with `KProperty`.

```kotlin
val filter = (Employee::location within reader.read("POLYGON ((0 0, 0 1000, 1000 1000, 1000 0, 0 0))") as Polygon)
```

More details on `within` function can be found [here](../java-sdk/filter.md#within-filter).

### Intersects

The intersects filter is used to match documents where the value of the spatial field intersects the specified shape.

```kotlin
val filter = ("location" intersects reader.read("POLYGON ((0 0, 0 1000, 1000 1000, 1000 0, 0 0))") as Polygon)
```

Similarly, you can use the `intersects` function with `KProperty`.

```kotlin
val filter = (Employee::location intersects reader.read("POLYGON ((0 0, 0 1000, 1000 1000, 1000 0, 0 0))") as Polygon)
```

More details on `intersects` function can be found [here](../java-sdk/filter.md#intersects-filter).