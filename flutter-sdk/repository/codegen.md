---
label: Code Generator
icon: file-code
order: 16
---

Nitrite provides Dart build system builders to extract metadata and generate code for entity classes which is used by `ObjectRepository`. The builders generate code when they find a class annotated with some Nitrite specific annotations. The generated code is written to a file that is named after the source file, but with `.no2.dart` extension.

Flutter doesn't support reflection, so the generated code is used to extract metadata and generate code for entity classes.

There are two generators available:

- **Entity Generator** - Extracts metadata from classes annotated with `@Entity` and generates code for entity classes.
- **Converter Generator** - Scans the classes annotated with `@Convertable` and generates the code to convert the Dart classes to Nitrite documents and vice-versa.

Both of these generators are available in the `nitrite_generator` package.

## Setup

Add the following package as your dev dependency:

```bash
dart pub add --dev build_runner
dart pub add --dev nitrite_generator
```

Once you have added the annotations to your code, you can run the generator using the `build_runner` command-line tool:

```bash
dart run build_runner build
```

## Annotations for Entity Generator

The following annotations are used by the entity generator:

- `@Entity` - Marks a class as an entity class.
- `@Id` - Marks a field as the primary key of the entity.
- `@Index` - Provides index information for entity fields.

More on each of these annotations is [here](entity.md#annotations).


## Annotations for Converter Generator

The following annotations are used by the converter generator:

- `@Convertable` - Generates the `EntityConverter` for the class.
- `@DocumentKey` - Specifies the name of the field in the document.
- `@IgnoredKey` - Marks a field as ignored while converting to document.

More on each of these annotations is [here](mapper.md#code-generation).


## Examples

Consider a book entity class and it's generated code:

+++ book.dart

```dart
import 'package:nitrite/nitrite.dart';

part 'book.no2.dart';

@Convertable(className: 'MyBookConverter')
@Entity(name: 'books', indices: [
  Index(fields: ['tags'], type: IndexType.nonUnique),
  Index(fields: ['description'], type: IndexType.fullText),
  Index(fields: ['price', 'publisher']),
])
class Book with _$BookEntityMixin {
  @Id(fieldName: 'book_id', embeddedFields: ['isbn', 'book_name'])
  @DocumentKey(alias: 'book_id')
  BookId? bookId;

  String? publisher;

  double? price;

  List<String> tags = [];

  String? description;

  Book(
      [this.bookId,
      this.publisher,
      this.price,
      this.tags = const [],
      this.description]);
}

@Convertable()
class BookId {
  String? isbn;

  @DocumentKey(alias: "book_name")
  String? name;

  @IgnoredKey()
  String? author;
}
```
+++ book.no2.dart

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
        "book_id",
        false,
        ["isbn", "book_name"],
      );
}

// **************************************************************************
// ConverterGenerator
// **************************************************************************

class MyBookConverter extends EntityConverter<Book> {
  @override
  Book fromDocument(
    Document document,
    NitriteMapper nitriteMapper,
  ) {
    var entity = Book();
    entity.bookId =
        nitriteMapper.tryConvert<BookId, Document>(document['book_id']);
    entity.publisher = document['publisher'];
    entity.price = document['price'];
    entity.tags = EntityConverter.toList(document['tags'], nitriteMapper);
    entity.description = document['description'];
    return entity;
  }

  @override
  Document toDocument(
    Book entity,
    NitriteMapper nitriteMapper,
  ) {
    var document = emptyDocument();
    document.put(
        'book_id', nitriteMapper.tryConvert<Document, BookId>(entity.bookId));
    document.put('publisher', entity.publisher);
    document.put('price', entity.price);
    document.put('tags', EntityConverter.fromList(entity.tags, nitriteMapper));
    document.put('description', entity.description);
    return document;
  }
}

class BookIdConverter extends EntityConverter<BookId> {
  @override
  BookId fromDocument(
    Document document,
    NitriteMapper nitriteMapper,
  ) {
    var entity = BookId();
    entity.isbn = document['isbn'];
    entity.name = document['book_name'];
    return entity;
  }

  @override
  Document toDocument(
    BookId entity,
    NitriteMapper nitriteMapper,
  ) {
    var document = emptyDocument();
    document.put('isbn', entity.isbn);
    document.put('book_name', entity.name);
    return document;
  }
}
```
+++

## Limitations

There are certain limitations of each of the generators as described below:

### Entity Generator

- `@Entity` annotation can only be used on a class.
- `@Id` annotation can only be used on a field and once per class.
- `@Id` field should be one of the following data types:
  - `num`
  - `int`
  - `double`
  - `String`
  - `Runes`
  - `bool`
  - `DateTime`
  - `Duration`
  - `NitriteId`
  - A class in case of embedded id.
- `@Id` field should not be static or synthetic.


### Converter Generator

- `@Convertable` annotation can only be used on a class and enum and not on an abstract class or mixin.
- `@Convertable` can only be used on a class which has at least one public constructor which is either a default constructor or one with all the parameters named or optional.
- A class with a constructor having all positional optional parameters should not have any final field.
- A class with a constructor having all named parameters (optional/required) should have all the fields' names same as the parameter names.
- There are certain limitations on the fields and its data types:
  - `dynamic` type is not supported.
  - Nested collection is not supported.
  - Function types are not supported.
  - `Symbol` type is not supported.
  - `Never` type is not supported.
  - `Void` type is not supported.
  - Any field with user-defined class type should be nullable.
- `@DocumentKey`, `@IgnoredKey` annotation can only be used on a field or property.
- `@DocumentKey` annotation can not be used on a private/static/synthetic field.
- `@IgnoredKey` annotation can not be used on a field with the same name as a non-null required named parameter in the constructor.
- A getter must be defined for corresponding setter or vice-versa.


!!!warning
If any of the above limitations are not met, then the generator will throw an exception and the build will fail.
!!!