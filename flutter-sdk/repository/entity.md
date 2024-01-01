---
label: Entity
icon: diamond
order: 15
---

An entity is a Dart object that can be stored in a collection. An entity can be a simple class with a few fields or a complex class with nested objects. Every entity in an `ObjectRepository` is actually converted to a `Document` before storing in the underlying `NitriteCollection`. When an entity is converted to a `Document`, the fields of the entity are mapped to the fields of the `Document`. While retrieving an entity from the `ObjectRepository`, the `Document` is converted back to the entity.

Nitrite uses a `NitriteMapper` implementation to convert an entity to a `Document` and vice versa. By default, Nitrite uses an `EntityConverter` based implementation of `NitriteMapper` to convert an entity to a `Document`. You can also provide your own implementation of `NitriteMapper` to convert an entity to a `Document`.

More on `NitriteMapper` can be found [here](mapper.md).

## Annotations

Nitrite uses annotations to define an entity. There are several annotations available to define an entity. These annotations are:

- `@Entity`
- `@Id`
- `@Index`

Flutter does not support reflection. So, Nitrite uses code generators to process these annotations and generates the metadata for an entity. The entity generator generates a mixin for an entity class in a separate file that is named after the source file, but with `.no2.dart` extension.

The name of the mixin is like - `_$` + `ClassName` + `EntityMixin`. For example, if the entity class name is `Person`, the mixin name will be _$PersonEntityMixin. The generated mixin also implements `NitriteEntity` interface which defines few getters to access the metadata of the entity. You need to add this mixin to your entity class to make it a Nitrite entity.

The entity generator is a separate package and more on the entity generator can be found [here](codegen.md).

### @Entity

The `@Entity` annotation marks a class as an entity class. The annotation has the following properties:

- `name` - The name of the collection to which the entity belongs. If not specified, the name of the class is used as the collection name.
- `indices` - An array of `@Index` annotations. The indices are created on the fields specified in the `@Index` annotation.

+++ No Parameter
```dart
import 'package:nitrite/nitrite.dart';

@Entity()
class Book {
  @Id()
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++ With Parameter
```dart
import 'package:nitrite/nitrite.dart';

@Entity(name: 'books', indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['description'], type: IndexType.fullText),
  Index(fields: ['price', 'publisher']),
])
class Book {
  @Id()
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

   Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++

The above code generates the following code in `book.no2.dart`:

+++ No Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [];
  @override
  EntityId get entityId => EntityId(
        "bookId",
        false,
      );
}
```
+++ With Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "books";
  @override
  List<EntityIndex> get entityIndexes => const [
        EntityIndex(["tags"], IndexType.nonUnique),
        EntityIndex(["description"], IndexType.fullText),
        EntityIndex(["price", "publisher"], IndexType.unique),
      ];
  @override
  EntityId get entityId => EntityId(
        "bookId",
        false,
      );
}
```
+++

Now you can use the generated mixin in your entity class:

+++ No Parameter
```dart
import 'package:nitrite/nitrite.dart';

part 'book.no2.dart';

@Entity()
class Book with _$BookEntityMixin {
  @Id()
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++ With Parameter
```dart
import 'package:nitrite/nitrite.dart';

part 'book.no2.dart';

@Entity(name: 'books', indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['description'], type: IndexType.fullText),
  Index(fields: ['price', 'publisher']),
])
class Book with _$BookEntityMixin {
  @Id()
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++

### @Id

The `@Id` annotation marks a field as the unique identifier of the entity. It is a field level annotation and takes optional `fieldName` parameter. The `fieldName` parameter is used to specify the name of the field in the document. If the `fieldName` parameter is not specified, then the name of the field in the class will be used as the name of the field in the document. 

+++ No Parameter
```dart
import 'package:nitrite/nitrite.dart';

@Entity()
class Book with _$BookEntityMixin {
  @Id()
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++ With Parameter
```dart
import 'package:nitrite/nitrite.dart';

@Entity()
class Book with _$BookEntityMixin {
  @Id(fieldName: 'book_id')
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```
+++

The above code generates the following code in `book.no2.dart`:

+++ No Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [];
  @override
  EntityId get entityId => EntityId(
        "bookId",
        false,
      );
}
```
+++ With Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [];
  @override
  EntityId get entityId => EntityId(
        "book_id",
        false,
      );
}
```
+++

#### Embedded Id

Nitrite also supports embedded id. The embedded id is used when the entity has a composite primary key. 
The `@Id` annotation can also be used to mark a field as an embedded id. In case of embedded id, the field should be marked with `@Id` annotation and the `fieldName` and `embeddedFields` parameters should be specified. The `fieldName` parameter is used to specify the name of the field in the document. The `embeddedFields` parameter is used to specify the name of the fields in the embedded object.

```dart
import 'package:nitrite/nitrite.dart';

@Entity()
class Book with _$BookEntityMixin {
  @Id(fieldName: 'book_id', embeddedFields: ['isbn', 'title'])
  BookId? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}

class BookId {
  String? isbn;
  String? title;
  String? author;
}
```

The above code generates the following code in `book.no2.dart`:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [];
  @override
  EntityId get entityId => EntityId(
        "book_id",
        false,
        embeddedFields: ["isbn", "title"],
      );
}
```

The above `@Id` annotation will create a unique compound index on the fields - `book_id.isbn` and `book_id.title`.

#### Data Type

The data type of the field marked with `@Id` annotation should be one of the following:

- `num`
- `int`
- `double`
- `String`
- `Runes`
- `bool`
- `DateTime`
- `Duration`
- `NitriteId`

In case of embedded id, the data type of the field marked with `@Id` annotation should be a class.

### @Index

`@Index` annotation is used to declare an index on an entity field. It is a class level annotation and takes mandatory `fields` parameter and optional `type` parameter. The `fields` parameter is used to specify the names of the fields in the document to be indexed. The `type` parameter is used to specify the type of the index. If the `type` parameter is not specified, then the type of the index will be `IndexType.unique`.

```dart
import 'package:nitrite/nitrite.dart';

@Entity(indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['description'], type: IndexType.fullText),
  Index(fields: ['price', 'publisher']),
])
class Book with _$BookEntityMixin {
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  String? description;

   Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    this.description,
  ]);
}
```

The above code generates the following code in `book.no2.dart`:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [
        EntityIndex(["tags"], IndexType.nonUnique),
        EntityIndex(["description"], IndexType.fullText),
        EntityIndex(["price", "publisher"], IndexType.unique),
      ];
  @override
  EntityId get entityId => EntityId(
        "bookId",
        false,
      );
}
```

The above `@Index` annotations will create the following indices:

- `tags` - Non-unique simple index
- `description` - Full-text index
- `price` and `publisher` - Unique compound index


`@Index` annotation can be used on super class as well. When the entity class extends a super class, the `@Index` annotations of the super class are also processed by the entity generator.

```dart
import 'package:nitrite/nitrite.dart';

@Entity(indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['price', 'publisher']),
])
class Book extends BaseEntity with _$BookEntityMixin {
  String? bookId;
  String? publisher;
  double? price;
  List<String> tags = [];
  
  Book([
    this.bookId,
    this.publisher,
    this.price,
    this.tags = const [],
    super.description,
  ]);
}

@Index(fields: ['description'], type: IndexType.fullText)
abstract class BaseEntity {
  String? description;

  BaseEntity([this.description]);
}
```

The above code generates the following code in `book.no2.dart`:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// NitriteEntityGenerator
// **************************************************************************

mixin _$BookEntityMixin implements NitriteEntity {
  @override
  String get entityName => "Book";
  @override
  List<EntityIndex> get entityIndexes => const [
        EntityIndex(["tags"], IndexType.nonUnique),
        EntityIndex(["price", "publisher"], IndexType.unique),
        EntityIndex(["description"], IndexType.fullText),
      ];
  @override
  EntityId? get entityId => null;
}
```

## EntityDecorator

`EntityDecorator` is used to add metadata to an entity. If you cannot modify the entity class to add annotations, then you can use `EntityDecorator` to add metadata to an entity. You can use `EntityDecorator` to add id, indices and collection name for an entity. 

While creating or opening a repository, you can pass an instance of `EntityDecorator` to the `getRepository()` method on `Nitrite` class. Nitrite will extract the metadata from the `EntityDecorator` instance and use it to create the repository.

```dart
ObjectRepository<Product> repository = db.getRepository<Product>(entityDecorator: ProductDecorator());
```

To write an `EntityDecorator` for an entity, you need to implement the `EntityDecorator` interface. Let's take an example of a `Product` entity.

```dart
class Product {
    ProductId? productId;
    Manufacturer? manufacturer;
    String? productName;
    double? price;

    Product([
        this.productId,
        this.manufacturer,
        this.productName,
        this.price,
    ]);
}

class ProductId {
    String? uniqueId;
    String? productCode;

    ProductId([
        this.uniqueId,
        this.productCode,
    ]);
}

class Manufacturer {
    String? name;
    String? uniqueId;

    Manufacturer([
        this.name,
        this.uniqueId,
    ]);
}
```

The `Product` entity has an embedded id `ProductId` and two indices on a nested object `Manufacturer`. To write an `EntityDecorator` for the `Product` entity with collection name 'products', you need to implement the `EntityDecorator` interface as follows.

```dart
class ProductDecorator extends EntityDecorator<Product> {
  @override
  String get entityName => "products";

  @override
  EntityId? get idField => EntityId(
        "productId",
        false,
        embeddedFields: ["uniqueId", "productCode"],
      );

  @override
  List<EntityIndex> get indexFields => const [
        EntityIndex(["manufacturer.name"], IndexType.nonUnique),
        EntityIndex(["productName", "manufacturer.uniqueId"]),
      ];
}
```

which is equivalent to the following entity class when code generator is used:

```dart
@Entity(name: 'products', indices: [
  Index(fields: ['manufacturer.name'], type: IndexType.nonUnique),
  Index(fields: ['productName', 'manufacturer.uniqueId']),
])
class Product with _$ProductEntityMixin {
  @Id(fieldName: 'productId', embeddedFields: ['uniqueId', 'productCode'])
  ProductId? productId;
  Manufacturer? manufacturer;
  String? productName;
  double? price;

  Product([
    this.productId,
    this.manufacturer,
    this.productName,
    this.price,
  ]);
}

class ProductId {
  String? uniqueId;
  String? productCode;

  ProductId([
    this.uniqueId,
    this.productCode,
  ]);
}

class Manufacturer {
  String? name;
  String? uniqueId;

  Manufacturer([
    this.name,
    this.uniqueId,
  ]);
}
```

### EntityId

`EntityId` is used to specify the id field of an entity. It takes mandatory `fieldName` parameter and optional `embeddedFields` parameter in case of embedded id. The `fieldName` parameter is used to specify the name of the field in the document. The `embeddedFields` parameter is used to specify the name of the fields in the embedded object.

```dart
@override
EntityId? get idField =>
    EntityId('productId', false, ['uniqueId', 'productCode']);
```

Here `productId` is the name of the field in the document and `uniqueId` and `productCode` are the names of the fields in the embedded object. This will create a unique compound index on the fields - `productId.uniqueId` and `productId.productCode`.

### EntityIndex

`EntityIndex` is used to specify the indices on an entity. It takes mandatory `type` parameter and `fields` parameter. The `type` parameter is used to specify the type of the index. The `fields` parameter is used to specify the name of the fields in the document to be indexed.

```dart
@override
List<EntityIndex> get indexFields => const [
    EntityIndex(['manufacturer.name'], IndexType.nonUnique),
    EntityIndex(['productName', 'manufacturer.uniqueId']),
    ];
```

Here `manufacturer.name` is the name of the field in the document and `productName` and `manufacturer.uniqueId` are the names of the fields in the document. This will create a non-unique index on the field `manufacturer.name` and a unique compound index on the fields - `productName` and `manufacturer.uniqueId`.


### Entity Name

Entity name is used to specify the name of the collection in which the entity will be stored. It is a string value and can be specified using `entityName` getter.

```dart
@override
String get entityName => "products";
```

Here `products` is the name of the `NitriteCollection` in which the product document will be stored.

