---
label: NitriteMapper
icon: workflow
order: 14
---

`NitriteMapper` is a simple and lightweight object mapper which can be used to map Dart objects to Nitrite documents and vice-versa. Nitrite uses a `NitriteMapper` implementation to map Dart entities to Nitrite documents and vice-versa while storing and retrieving objects from an `ObjectRepository`.

## SimpleNitriteMapper

Nitrite provides a default `NitriteMapper` implementation called `SimpleNitriteMapper`. This is a simple and lightweight mapper which uses `EntityConverter` to map a Dart object to Nitrite documents and vice-versa. This mapper is suitable for most of the use cases.

### EntityConverter

`EntityConverter` is a simple interface which provides methods to convert a Dart object to Nitrite document and vice-versa. For each Dart class, you need to provide an implementation of `EntityConverter` and register it with `SimpleNitriteMapper`. `SimpleNitriteMapper` will use this converter to map the Dart object to Nitrite document and vice-versa.

You can write your own implementation of `EntityConverter` or you can use the code generator to generate the implementation for you. More on this is discussed [here](#code-generation).


Let's take an example of `Product` class.

```dart
class Product {
    ProductId? productId;
    Manufacturer? manufacturer;
    String? productName;
    double? price;
    
    Product({this.productId, this.manufacturer, this.productName, this.price});
}

class ProductId {
    String? uniqueId;
    String? productCode;
    
    ProductId({this.uniqueId, this.productCode});
}

class Manufacturer {
    String? name;
    String? address;
    int? uniqueId;
    
    Manufacturer({this.name, this.address, this.uniqueId});
}
```

To map to Nitrite document, we need to provide an implementation of `EntityConverter` for each class. Let's take a look at the converter for `Product` class.

```dart
class ProductConverter extends EntityConverter<Product> {
    @override
    Document toDocument(Product entity, NitriteMapper nitriteMapper) {
        Document productId = nitriteMapper.tryConvert<Document, ProductId>(entity.productId);
        Document manufacturer = nitriteMapper.tryConvert<Document, Manufacturer>(entity.manufacturer);

        return emptyDocument()
            .put("productId", productId)
            .put("manufacturer", manufacturer)
            .put("productName", entity.productName)
            .put("price", entity.price);
    }

    @override
    Product fromDocument(Document document, NitriteMapper nitriteMapper) {
        var product = Product();
        product.productId = nitriteMapper.tryConvert<ProductId, Document>(document["productId"]);
        product.manufacturer = nitriteMapper.tryConvert<Manufacturer, Document>(document["manufacturer"]);
        product.productName = document["productName"];
        product.price = document["price"];

        return product;
    }
}
```

Similarly, we need to provide converters for `ProductId` and `Manufacturer` classes.

```dart
class ProductIdConverter extends EntityConverter<ProductId> {
    @override
    Document toDocument(ProductId entity, NitriteMapper nitriteMapper) {
        return emptyDocument()
            .put("uniqueId", entity.uniqueId)
            .put("productCode", entity.productCode);
    }

    @override
    ProductId fromDocument(Document document, NitriteMapper nitriteMapper) {
        var productId = ProductId();
        productId.uniqueId = document["uniqueId"];
        productId.productCode = document["productCode"];

        return productId;
    }
}

class ManufacturerConverter extends EntityConverter<Manufacturer> {
    @override
    Document toDocument(Manufacturer entity, NitriteMapper nitriteMapper) {
        return emptyDocument()
            .put("name", entity.name)
            .put("address", entity.address)
            .put("uniqueId", entity.uniqueId);
    }

    @override
    Manufacturer fromDocument(Document document, NitriteMapper nitriteMapper) {
        var manufacturer = Manufacturer();
        manufacturer.name = document["name"];
        manufacturer.address = document["address"];
        manufacturer.uniqueId = document["uniqueId"];

        return manufacturer;
    }
}
```

Once the converters are ready, we need to register them with `registerEntityConverter()` method on `NitriteBuilder` instance.

```dart
var db = await Nitrite.builder()
    .registerEntityConverter(ProductConverter())
    .registerEntityConverter(ProductIdConverter())
    .registerEntityConverter(ManufacturerConverter())
    .openOrCreate();
```

we can also register the converters with `SimpleNitriteMapper` instance and then pass the instance to `loadModule()` method.

```dart
var nitriteMapper = SimpleNitriteMapper();
nitriteMapper.registerEntityConverter(ProductConverter());
nitriteMapper.registerEntityConverter(ProductIdConverter());
nitriteMapper.registerEntityConverter(ManufacturerConverter());

var db = await Nitrite.builder()
    .loadModule(module(nitriteMapper))
    .openOrCreate();
```

!!!info
`NitriteMapper` is a `NitritePlugin`. So, you need to load it using `loadModule()` method on `NitriteBuilder`. More on Nitrite's module system can be found [here](../modules/module-system.md).
!!!

!!!warning Warning
If you have used the `registerEntityConverter()` method on `NitriteBuilder` instance, Nitrite will only use `SimpleNitriteMapper` to map the entities. It will ignore any other `NitriteMapper` implementation you have provided using `loadModule()` method.
!!!

### Mapping Collections

The `EntityConverter` interface provides some static methods to convert a collection of Dart objects to a collection of Nitrite documents and vice-versa. 

These methods are:

* `fromList()` - Converts a `List` of Dart objects to a `List` of Nitrite documents. If the type of the elements is a registered value type, it will return the same list without any conversion.
* `fromIterable()` - Converts an `Iterable` of Dart objects to a `List` of Nitrite documents. If the type of the elements is a registered value type, it will return the same list without any conversion.
* `fromSet()` - Converts a `Set` of Dart objects to a `Set` of Nitrite documents. If the type of the elements is a registered value type, it will return the same set without any conversion.
* `fromMap()` - Converts a `Map` of key-value pair to a `Map` of Nitrite documents. If the type of the `key` or the `value` is a registered value type, it will not convert those objects to Nitrite document and return as is.
* `toList()` - Converts a `List` of Nitrite documents to a `List` of Dart objects. 
* `toIterable()` - Converts an `Iterable` of Nitrite documents to an `List` of Dart objects.
* `toSet()` - Converts a `Set` of Nitrite documents to a `Set` of Dart objects.
* `toMap()` - Converts a `Map` of Nitrite documents to a `Map` of key value pair.

These methods are useful when you have a collection of objects as a property of a Dart class.

#### Example

Let's take an example of `Company` class which has a map of `Employee` as a property.

```dart
class Company {
  final int companyId;
  final String companyName;
  final DateTime? dateCreated;
  final List<String> departments;
  final Map<String, List<Employee>> employeeRecord;

  Company({
    this.companyId = 0,
    this.companyName = '',
    this.dateCreated,
    this.departments = const [],
    this.employeeRecord = const {},
  });
}

class Employee {
  int? empId;
  DateTime? joinDate;
  String address;
  String emailAddress;
  List<int> blob;
  Company? company;
  Note? employeeNote;

  Employee({
    this.empId,
    this.joinDate,
    this.address = '',
    this.emailAddress = '',
    this.blob = const [],
    this.company,
    this.employeeNote,
  });
}

class Note {
  final int noteId;
  final String text;

  Note({
    this.noteId = 0,
    this.text = '',
  });
}

```

To map this to Nitrite document, we need to provide an implementation of `EntityConverter` for `Company` class.

```dart
class CompanyConverter extends EntityConverter<Company> {
  @override
  Company fromDocument(
    Document document,
    NitriteMapper nitriteMapper,
  ) {
    var map = <String, List<Employee>>{};
    var employeeRecords = document['employeeRecord'] == null
        ? {}
        : document['employeeRecord'] as Map;

    for (var entry in employeeRecords.entries) {
      var key = nitriteMapper.tryConvert<String, dynamic>(entry.key);
      var value = EntityConverter.toList<Employee>(entry.value, nitriteMapper);
      map[key] = value;
    }

    var entity = Company(
      companyId: document['company_id'] ?? 0,
      companyName: document['companyName'] ?? "",
      dateCreated: document['dateCreated'],
      departments:
          EntityConverter.toList(document['departments'], nitriteMapper),
      employeeRecord: map,
    );

    return entity;
  }

  @override
  Document toDocument(
    Company entity,
    NitriteMapper nitriteMapper,
  ) {
    var document = emptyDocument();
    document.put('company_id', entity.companyId);
    document.put('companyName', entity.companyName);
    document.put('dateCreated', entity.dateCreated);
    document.put('departments',
        EntityConverter.fromList(entity.departments, nitriteMapper));
    document.put('employeeRecord',
        EntityConverter.fromMap(entity.employeeRecord, nitriteMapper));
    return document;
  }
}

class EmployeeConverter extends EntityConverter<Employee> {
  @override
  Employee fromDocument(
    Document document,
    NitriteMapper nitriteMapper,
  ) {
    var entity = Employee(
      empId: document['empId'],
      joinDate: document['joinDate'],
      address: document['address'] ?? "",
      emailAddress: document['emailAddress'] ?? "",
      blob: EntityConverter.toList(document['blob'], nitriteMapper),
      company: nitriteMapper.tryConvert<Company, Document>(document['company']),
      employeeNote:
          nitriteMapper.tryConvert<Note, Document>(document['employeeNote']),
    );
    return entity;
  }

  @override
  Document toDocument(
    Employee entity,
    NitriteMapper nitriteMapper,
  ) {
    var document = emptyDocument();
    document.put('empId', entity.empId);
    document.put('joinDate', entity.joinDate);
    document.put('address', entity.address);
    document.put('emailAddress', entity.emailAddress);
    document.put('blob', EntityConverter.fromList(entity.blob, nitriteMapper));
    document.put('employeeNote',
        nitriteMapper.tryConvert<Document, Note>(entity.employeeNote));
    return document;
  }
}

class NoteConverter extends EntityConverter<Note> {
  @override
  Note fromDocument(
    Document document,
    NitriteMapper nitriteMapper,
  ) {
    var entity = Note(
      noteId: document['noteId'] ?? 0,
      text: document['text'] ?? "",
    );
    return entity;
  }

  @override
  Document toDocument(
    Note entity,
    NitriteMapper nitriteMapper,
  ) {
    var document = emptyDocument();
    document.put('noteId', entity.noteId);
    document.put('text', entity.text);
    return document;
  }
}
```


## Code Generation

Nitrite provides a code generator to generate the `EntityConverter` implementation for you. You have to annotate the class with some specific annotations and then run the code generator. The generated code will be saved in a separate file. You still need to register the generated converter with Nitrite.

The code generator is a separate package and more on the entity generator can be found [here](codegen.md).

Nitrite uses below annotations to generate the `EntityConverter` implementation.

- `@Convertable`
- `@DocumentKey`
- `@IgnoredKey`

### @Convertable

`@Convertable` annotation is used to mark a class as convertable. This annotation is mandatory for the code generator to work. This annotation specifies the code generator to generate the `EntityConverter` implementation for the class. It takes an optional parameter `className` which is used to specify the name of the converter class. If this parameter is not specified, then the code generator will use the class name with `Converter` suffix as the converter class name.

+++ No Parameter
```dart
@Convertable()
class Book {
  BookId? bookId;
  String? publisher;
  double? price;
  List<String>? tags;
  String? description;

  Book({this.bookId, this.publisher, this.price, this.tags, this.description});
}

@Convertable()
class BookId {
  String? isbn;
  String? name;
  String? author;
}

```
+++ With Parameter
```dart
@Convertable(className: 'MyBookConverter')
class Book {
  BookId? bookId;
  String? publisher;
  double? price;
  List<String>? tags;
  String? description;

  Book({this.bookId, this.publisher, this.price, this.tags, this.description});
}

@Convertable()
class BookId {
  String? isbn;
  String? name;
  String? author;
}

```
+++

This will generate the following code:

+++ No Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// ConverterGenerator
// **************************************************************************

class BookConverter extends EntityConverter<Book> {
  @override
  Book fromDocument(Document document, NitriteMapper nitriteMapper) {
    var book = Book();
    book.bookId = nitriteMapper.tryConvert<BookId, Document>(document["bookId"]);
    book.publisher = document["publisher"];
    book.price = document["price"];
    book.tags = EntityConverter.toList<String>(document["tags"], nitriteMapper);
    book.description = document["description"];
    return book;
  }

  @override
  Document toDocument(Book entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("bookId", nitriteMapper.tryConvert<Document, BookId>(entity.bookId))
        .put("publisher", entity.publisher)
        .put("price", entity.price)
        .put("tags", EntityConverter.fromList<String>(entity.tags, nitriteMapper))
        .put("description", entity.description);
  }
}

class BookIdConverter extends EntityConverter<BookId> {
  @override
  BookId fromDocument(Document document, NitriteMapper nitriteMapper) {
    var bookId = BookId();
    bookId.isbn = document["isbn"];
    bookId.name = document["name"];
    bookId.author = document["author"];
    return bookId;
  }

  @override
  Document toDocument(BookId entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("isbn", entity.isbn)
        .put("name", entity.name)
        .put("author", entity.author);
  }
}
```
+++ With Parameter
```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// ConverterGenerator
// **************************************************************************

class MyBookConverter extends EntityConverter<Book> {
  @override
  Book fromDocument(Document document, NitriteMapper nitriteMapper) {
    var book = Book();
    book.bookId = nitriteMapper.tryConvert<BookId, Document>(document["bookId"]);
    book.publisher = document["publisher"];
    book.price = document["price"];
    book.tags = EntityConverter.toList<String>(document["tags"], nitriteMapper);
    book.description = document["description"];
    return book;
  }

  @override
  Document toDocument(Book entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("bookId", nitriteMapper.tryConvert<Document, BookId>(entity.bookId))
        .put("publisher", entity.publisher)
        .put("price", entity.price)
        .put("tags", EntityConverter.fromList<String>(entity.tags, nitriteMapper))
        .put("description", entity.description);
  }
}

class BookIdConverter extends EntityConverter<BookId> {
  @override
  BookId fromDocument(Document document, NitriteMapper nitriteMapper) {
    var bookId = BookId();
    bookId.isbn = document["isbn"];
    bookId.name = document["name"];
    bookId.author = document["author"];
    return bookId;
  }

  @override
  Document toDocument(BookId entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("isbn", entity.isbn)
        .put("name", entity.name)
        .put("author", entity.author);
  }
}
```
+++

You need to register the generated converters with Nitrite by calling `registerEntityConverter()` method on `NitriteBuilder` instance.

```dart
var db = await Nitrite.builder()
    .registerEntityConverter(BookConverter())
    .registerEntityConverter(BookIdConverter())
    .openOrCreate();
```

### @DocumentKey

`@DocumentKey` annotation is used to mark a field as a document key. This annotation provides an alias to the field. Normally the field name is used as the key in the Nitrite document. If you want to use a different name, then you can specify the alias using this annotation. This annotation can be used on a field or a getter/setter method.

```dart
@Convertable()
class Book {
  @DocumentKey(alias: 'book_id')
  BookId? bookId;
  String? publisher;
  double? price;
  List<String>? tags;
  String? description;

  Book({this.bookId, this.publisher, this.price, this.tags, this.description});
}

@Convertable()
class BookId {
  String? isbn;
  @DocumentKey(alias: "book_name")
  String? name;
  String? author;
}
```

This will generate the following code:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// ConverterGenerator
// **************************************************************************

class BookConverter extends EntityConverter<Book> {
  @override
  Book fromDocument(Document document, NitriteMapper nitriteMapper) {
    var book = Book();
    book.bookId = nitriteMapper.tryConvert<BookId, Document>(document["book_id"]);
    book.publisher = document["publisher"];
    book.price = document["price"];
    book.tags = EntityConverter.toList<String>(document["tags"], nitriteMapper);
    book.description = document["description"];
    return book;
  }

  @override
  Document toDocument(Book entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("book_id", nitriteMapper.tryConvert<Document, BookId>(entity.bookId))
        .put("publisher", entity.publisher)
        .put("price", entity.price)
        .put("tags", EntityConverter.fromList<String>(entity.tags, nitriteMapper))
        .put("description", entity.description);
  }
}

class BookIdConverter extends EntityConverter<BookId> {
  @override
  BookId fromDocument(Document document, NitriteMapper nitriteMapper) {
    var bookId = BookId();
    bookId.isbn = document["isbn"];
    bookId.name = document["book_name"];
    bookId.author = document["author"];
    return bookId;
  }

  @override
  Document toDocument(BookId entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("isbn", entity.isbn)
        .put("book_name", entity.name)
        .put("author", entity.author);
  }
}
```

### @IgnoredKey

`@IgnoredKey` annotation is used to mark a field as ignored. This annotation is useful when you want to ignore a field while mapping to Nitrite document. This annotation can be used on a field or a getter/setter method.

```dart
@Convertable()
class BookId {
  String? isbn;
  @DocumentKey(alias: "book_name")
  String? name;
  @IgnoredKey()
  String? author;
}
```

This will generate the following code:

```dart
// GENERATED CODE - DO NOT MODIFY BY HAND

part of 'book.dart';

// **************************************************************************
// ConverterGenerator
// **************************************************************************

class BookIdConverter extends EntityConverter<BookId> {
  @override
  BookId fromDocument(Document document, NitriteMapper nitriteMapper) {
    var bookId = BookId();
    bookId.isbn = document["isbn"];
    bookId.name = document["book_name"];
    return bookId;
  }

  @override
  Document toDocument(BookId entity, NitriteMapper nitriteMapper) {
    return emptyDocument()
        .put("isbn", entity.isbn)
        .put("book_name", entity.name);
  }
}
```

!!!warning
The code generator imposes some limitations on the Dart classes. These limitations are discussed [here](codegen.md#converter-generator).
!!!

## Custom NitriteMapper

Apart from `SimpleNitriteMapper`, you can also provide your own implementation of `NitriteMapper`. You need to implement `NitriteMapper` interface and provide your own implementation. Once the implementation is ready, you need to load it using `loadModule()` method on `NitriteBuilder` while building the database.

```java
Nitrite db = Nitrite.builder()
    .loadModule(module(new CustomNitriteMapper()))
    .openOrCreate();
```
