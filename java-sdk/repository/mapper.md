---
label: Nitrite Mapper
icon: workflow
order: 15
---

Nitrite Mapper is a simple and lightweight object mapper which can be used to map Java objects to Nitrite documents and vice-versa. Nitrite uses a `NitriteMapper` implementation to map Java entities to Nitrite documents and vice-versa while storing and retrieving objects from an `ObjectRepository`.

## SimpleNitriteMapper

Nitrite provides a default `NitriteMapper` implementation called `SimpleNitriteMapper`. This is a simple and lightweight mapper which uses `EntityConverter` to map a Java entity to Nitrite documents and vice-versa. This mapper is suitable for most of the use cases.

### EntityConverter

`EntityConverter` is a simple interface which provides methods to convert a Java entity to Nitrite document and vice-versa. For each entity, you need to provide an implementation of `EntityConverter` and register it with `SimpleNitriteMapper`. `SimpleNitriteMapper` will use this converter to map the entity to Nitrite document and vice-versa.

Let's take an example of `Product` class.

```java
public class Product {
    private ProductId productId;
    private Manufacturer manufacturer;
    private String productName;
    private Double price;
    // getter and setter
}

public class ProductId {
    private String uniqueId;
    private String productCode;
    // getter and setter
}

public class Manufacturer {
    private String name;
    private String address;
    private Integer uniqueId;
    // getter and setter
}
```

To map this entity to Nitrite document, we need to provide an implementation of `EntityConverter` for each entity. Let's take a look at the converter for `Product` class.

```java
public class ProductConverter implements EntityConverter<Product> {
    @Override
    public Class<Product> getEntityType() {
        return Product.class;
    }

    @Override
    public Document toDocument(Product entity, NitriteMapper nitriteMapper) {
        Document productId = (Document) nitriteMapper.tryConvert(entity.getProductId(), Document.class);
        Document manufacturer = (Document) nitriteMapper.tryConvert(entity.getManufacturer(), Document.class);

        return Document.createDocument()
            .put("productId", productId)
            .put("manufacturer", manufacturer)
            .put("productName", entity.getProductName())
            .put("price", entity.getPrice());
    }

    @Override
    public Product fromDocument(Document document, NitriteMapper nitriteMapper) {
        Product entity = new Product();
        ProductId productId = (ProductId) nitriteMapper.tryConvert(document.get("productId", Document.class), ProductId.class);
        Manufacturer manufacturer = (Manufacturer) nitriteMapper.tryConvert(document.get("manufacturer", Document.class),
            Manufacturer.class);
        entity.setProductId(productId);
        entity.setManufacturer(manufacturer);
        entity.setProductName(document.get("productName", String.class));
        entity.setPrice(document.get("price", Double.class));
        return entity;
    }
}
```

Similarly, we need to provide converter for `ProductId` and `Manufacturer` class.

```java
public class ProductIdConverter implements EntityConverter<ProductId> {
    @Override
    public Class<ProductId> getEntityType() {
        return ProductId.class;
    }

    @Override
    public Document toDocument(ProductId entity, NitriteMapper nitriteMapper) {
        return Document.createDocument("uniqueId", entity.getUniqueId())
            .put("productCode", entity.getProductCode());
    }

    @Override
    public ProductId fromDocument(Document document, NitriteMapper nitriteMapper) {
        ProductId entity = new ProductId();
        entity.setUniqueId(document.get("uniqueId", String.class));
        entity.setProductCode(document.get("productCode", String.class));
        return entity;
    }
}

public class ManufacturerConverter implements EntityConverter<Manufacturer> {
    @Override
    public Class<Manufacturer> getEntityType() {
        return Manufacturer.class;
    }

    @Override
    public Document toDocument(Manufacturer entity, NitriteMapper nitriteMapper) {
        return Document.createDocument("name", entity.getName())
            .put("address", entity.getAddress())
            .put("uniqueId", entity.getUniqueId());
    }

    @Override
    public Manufacturer fromDocument(Document document, NitriteMapper nitriteMapper) {
        Manufacturer manufacturer = new Manufacturer();
        manufacturer.setName(document.get("name", String.class));
        manufacturer.setAddress(document.get("address", String.class));
        manufacturer.setUniqueId(document.get("uniqueId", Integer.class));
        return manufacturer;
    }
}
```

Once the converters are ready, we need to register them with `SimpleNitriteMapper`.

```java
SimpleNitriteMapper nitriteMapper = new SimpleNitriteMapper();
nitriteMapper.registerEntityConverter(new ProductConverter());
nitriteMapper.registerEntityConverter(new ProductIdConverter());
nitriteMapper.registerEntityConverter(new ManufacturerConverter());
```

Now, we need to load this mapper while building the database.

```java
Nitrite db = Nitrite.builder()
    .loadModule(module(nitriteMapper))
    .openOrCreate();
```

!!! info
`NitriteMapper` is a `NitritePlugin`. So, you need to load it using `loadModule()` method on `NitriteBuilder`. More on Nitrite's module system can be found [here](../modules/module-system.md).
!!!

## JacksonMapper

Nitrite also provides a Jackson based mapper called `JacksonMapper`. This mapper uses Jackson's `ObjectMapper` to map Java entities to Nitrite documents and vice-versa. Here you don't need to provide any `EntityConverter`. Jackson will use its own `ObjectMapper` to map the entities.

More on this mapper can be found [here](../modules/jackson.md).

## Custom NitriteMapper

Apart from `SimpleNitriteMapper` and `JacksonMapper`, you can also provide your own implementation of `NitriteMapper`. You need to implement `NitriteMapper` interface and provide your own implementation. Once the implementation is ready, you need to load it using `loadModule()` method on `NitriteBuilder` while building the database.

```java
Nitrite db = Nitrite.builder()
    .loadModule(module(new CustomNitriteMapper()))
    .openOrCreate();
```




