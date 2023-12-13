---
label: Spatial Module
icon: location
order: 2
---

Nitrite Spatial module provides support for spatial queries. The module uses [JTS](https://locationtech.github.io/jts/) library for spatial operations. 

## Adding Spatial Module

Add Spatial module to your project:

### Maven

Add the Spatial dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dizitart</groupId>
    <artifactId>nitrite-spatial</artifactId>
</dependency>
```

### Gradle

Add the Spatial dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation 'org.dizitart:nitrite-spatial'
}
```

## Using Spatial Module

To use Spatial module, you need to load the `SpatialModule` while opening the database.

```java
Nitrite db = Nitrite.builder()
            .loadModule(new SpatialModule())
            .openOrCreate();
```

### Using Spatial Modules with Jackson

If you are using Jackson module for serialization, you need to register the `SpatialModule` with Jackson as well.

```java
Nitrite db = Nitrite.builder()
            .loadModule(new SpatialModule())
            .loadModule(new JacksonMapperModule(new GeometryModule()))
            .openOrCreate();
```

The `GeometryModule` is required to serialize the `Geometry` object of JTS via Jackson.

## Spatial Index

Spatial module uses [R-Tree](https://en.wikipedia.org/wiki/R-tree) index to store the spatial data. 

To create a spatial index, you need to pass the index type as `SpatialIndexer.SPATIAL_INDEX` while creating the index.

```java
collection.createIndex(IndexOptions.indexOptions(SpatialIndexer.SPATIAL_INDEX), "location");
```

## Spatial Filter

Spatial module supports several filters to query spatial data. To know more about filters, please refer to [Geospatial Filters](../filter.md#geospatial-filters).