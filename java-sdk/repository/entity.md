---
label: Entity
icon: diamond
order: 16
---

An entity is a Java object that can be stored in a Nitrite database. An entity can be a simple POJO or a complex object with nested objects. Every entity in an `ObjectRepository` is actually converted to a `Document` before storing in the underlying `NitriteCollection`. When an entity is converted to a `Document`, the fields of the entity are mapped to the fields of the `Document`. While retrieving an entity from the `ObjectRepository`, the `Document` is converted back to the entity.

Nitrite uses a `NitriteMapper` implementation to convert an entity to a `Document` and vice versa. By default, Nitrite uses an `EntityConverter` based implementation of `NitriteMapper` to convert an entity to a `Document`. You can also provide your own implementation of `NitriteMapper` to convert an entity to a `Document`.

More on `NitriteMapper` can be found [here](mapper.md).

## Annotations

Nitrite uses annotations to define an entity. There are several annotations available to define an entity. These annotations are:

- `@Entity`
- `@Id`
- `@Index`
- `@Indices`
- `@InheritIndices`

In case you cannot modify the entity class to add annotations, you can use `EntityDecorator` to add metadata to an entity. More on `EntityDecorator` can be found [here](entity.md#entitydecorator).

### @Entity

`@Entity` annotation is used to mark a class as an entity. It is a class level annotation and takes optional `value` and `indices` parameters. 

The `value` parameter is used to specify the name of the `NitriteCollection` in which the entity will be stored as a `Document`. If the `value` parameter is not specified, then the name of the collection will be the name of the class. The `indices` parameter is used to specify the indices on the repository. 

+++ No Parameter
```java
@Entity
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```
+++ With Parameters
```java
@Entity(value = "employees", indices = {
    @Index(fields = "firstName", type = IndexType.NON_UNIQUE),
    @Index(fields = "lastName", type = IndexType.NON_UNIQUE),
    @Index(fields = { "address.city", "firstName" }),
})
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```
+++

### @Id

`@Id` annotation is used to mark a field as the unique identifier of the entity. It is a field level annotation and takes optional `fieldName` parameter. The `fieldName` parameter is used to specify the name of the field in the document. If the `fieldName` parameter is not specified, then the name of the field in the class will be used as the name of the field in the document. 

+++ No Parameter
```java
@Entity
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```
+++ With Parameter
```java
@Entity
public class Employee {
    @Id(fieldName = "empId")
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```
+++

#### Embedded Id

Nitrite also supports embedded id. The embedded id is used when the entity has a composite primary key. The `@Id` annotation can also be used to mark a field as an embedded id. In case of embedded id, the field should be marked with `@Id` annotation and the `fieldName` and `embeddedFields` parameters should be specified. The `fieldName` parameter is used to specify the name of the field in the document. The `embeddedFields` parameter is used to specify the name of the fields in the embedded object.

```java
@Entity
public class Employee {
    @Id(fieldName = "emp_Id", embeddedFields = { "uniqueId", "companyId" })
    private EmployeeId empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}

public class EmployeeId {
    private Long uniqueId;
    private String companyId;
    // getters and setters
}
```

The above `@Id` annotation will create a unique compound index on the fields - `emp_Id.uniqueId` and `emp_Id.companyId`.

#### Data Type

The data type of the field marked with `@Id` annotation can be either `Comparable` or `NitriteId`. If the data type is `Comparable`, then the value of the field should be unique. If the data type is `NitriteId`, then the value of the field will be generated by Nitrite. But in the case of embedded id, the data type of the embedded fields can only be `Comparable`, it can't be `NitriteId`.


### @Index

`@Index` annotation is used to declare an index on an entity field. It is a class level annotation and takes mandatory `fields` parameter and optional `type` parameter. The `fields` parameter is used to specify the names of the fields in the document to be indexed. The `type` parameter is used to specify the type of the index. If the `type` parameter is not specified, then the type of the index will be `IndexType.UNIQUE`.

```java
@Entity(value = "employees", indices = {
    @Index(fields = "firstName", type = IndexType.NON_UNIQUE),
    @Index(fields = "lastName", type = IndexType.NON_UNIQUE),
    @Index(fields = { "address.city", "firstName" }),
})
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```

### @Indices

`@Indices` annotation is used declare multiple indices on an entity. It is a class level annotation and takes mandatory `value` parameter. The `value` parameter is used to specify the list of `@Index` annotations.

```java
@Indices({
    @Index(fields = "firstName", type = IndexType.NON_UNIQUE),
    @Index(fields = "address.streetName", type = IndexType.FULL_TEXT)
})
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}
```

### @InheritIndices

`@InheritIndices` annotation is used to inherit the indices from the parent class. It is a class level annotation and takes no parameter.

```java
@Entity
@Indices({
    @Index(fields = "firstName", type = IndexType.NON_UNIQUE),
    @Index(fields = "address.streetName", type = IndexType.FULL_TEXT)
})
public class Employee {
    @Id
    private Long empId;
    private String firstName;
    private String lastName;
    private Address address;
    private List<String> phoneNumbers;
    private Map<String, String> properties;
    // getters and setters
}

@Entity
@InheritIndices
@Index(fields = "department", type = IndexType.NON_UNIQUE),
public class Manager extends Employee {
    private String department;
    // getters and setters
}
```

If the `@InheritIndices` annotation is not specified, then the indices will not be inherited from the parent class.


## EntityDecorator

`EntityDecorator` is used to add metadata to an entity. If you cannot modify the entity class to add annotations, then you can use `EntityDecorator` to add metadata to an entity. You can use `EntityDecorator` to add id, indices and collection name for an entity. 

While creating or opening a repository, you can pass an instance of `EntityDecorator` to the `getRepository()` method on `Nitrite` class. Nitrite will extract the metadata from the `EntityDecorator` instance and use it to create the repository.

```java
Nitrite db = Nitrite.builder()
    .openOrCreate();

ObjectRepository<Product> repository = db.getRepository(new ProductDecorator());
```

To write an `EntityDecorator` for an entity, you need to implement the `EntityDecorator` interface. Let's take an example of a `Product` entity.

```java
public class Product {
    private ProductId productId;
    private Manufacturer manufacturer;
    private String productName;
    private Double price;
    // getters and setters
}

public class ProductId {
    private String uniqueId;
    private String productCode;
    // getters and setters
}

public class Manufacturer {
    private String name;
    private String uniqueId;
    // getters and setters
}
```

The `Product` entity has an embedded id `ProductId` and two indices on a nested object `Manufacturer`. To write an `EntityDecorator` for the `Product` entity, you need to implement the `EntityDecorator` interface as follows.

```java
public class ProductDecorator implements EntityDecorator<Product> {
    @Override
    public Class<Product> getEntityType() {
        return Product.class;
    }

    @Override
    public EntityId getIdField() {
        return new EntityId("productId", "uniqueId", "productCode");
    }

    @Override
    public List<EntityIndex> getIndexFields() {
        return Arrays.asList(
            new EntityIndex(IndexType.NON_UNIQUE, "manufacturer.name"),
            new EntityIndex(IndexType.UNIQUE, "productName", "manufacturer.uniqueId")
        );
    }

    @Override
    public String getEntityName() {
        return "product";
    }
}
```

which is equivalent to the following entity class.

```java
@Entity(value = "product", indices = {
    @Index(fields = "manufacturer.name", type = IndexType.NON_UNIQUE),
    @Index(fields = { "productName", "manufacturer.uniqueId" }, type = IndexType.UNIQUE)
})
public class Product {
    @Id(fieldName = "productId", embeddedFields = { "uniqueId", "productCode" })
    private ProductId productId;
    private Manufacturer manufacturer;
    private String productName;
    private Double price;
}
```

### EntityId

`EntityId` is used to specify the id field of an entity. It takes mandatory `fieldName` parameter and optional `embeddedFields` parameter in case of embedded id. The `fieldName` parameter is used to specify the name of the field in the document. The `embeddedFields` parameter is used to specify the name of the fields in the embedded object.

```java
@Override
public EntityId getIdField() {
    return new EntityId("productId", "uniqueId", "productCode");
}
```

Here `productId` is the name of the field in the document and `uniqueId` and `productCode` are the names of the fields in the embedded object. This will create a unique compound index on the fields - `productId.uniqueId` and `productId.productCode`.

### EntityIndex

`EntityIndex` is used to specify the indices on an entity. It takes mandatory `type` parameter and `fields` parameter. The `type` parameter is used to specify the type of the index. The `fields` parameter is used to specify the name of the fields in the document to be indexed.

```java
@Override
public List<EntityIndex> getIndexFields() {
    return Arrays.asList(
        new EntityIndex(IndexType.NON_UNIQUE, "manufacturer.name"),
        new EntityIndex(IndexType.UNIQUE, "productName", "manufacturer.uniqueId")
    );
}
```

Here `manufacturer.name` is the name of the field in the document and `productName` and `manufacturer.uniqueId` are the names of the fields in the document. This will create a non-unique index on the field `manufacturer.name` and a unique compound index on the fields - `productName` and `manufacturer.uniqueId`.

### Entity Name

Entity name is used to specify the name of the collection in which the entity will be stored. It takes no parameter.

```java
@Override
public String getEntityName() {
    return "product";
}
```

Here `product` is the name of the `NitriteCollection` in which the product document will be stored.


