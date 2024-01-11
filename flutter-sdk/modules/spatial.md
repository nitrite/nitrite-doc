---
label: Spatial Module
icon: location
order: 2
---

Nitrite Spatial module provides support for spatial queries. The module uses [JTS](https://locationtech.github.io/jts/) port of the dart package [dart_jts](https://pub.dev/packages/dart_jts) for spatial operations.

## Adding Spatial Module

To use Spatial module to your project, you need to add below dependency to your project:

```bash
dart pub add nitrite
dart pub add nitrite_spatial
```

## Using Spatial Module

To use Spatial module, you need to load the `SpatialModule` while opening the database.

```dart
import 'package:nitrite/nitrite.dart';
import 'package:nitrite_spatial/nitrite_spatial.dart';

var db = await Nitrite.builder()
            .loadModule(SpatialModule())
            .openOrCreate();
```

If you are using `nitrite_hive_adapter` as your storage module, you need to register the `GeometryAdapter` with the Hive database.

```dart
import 'package:nitrite_hive_adapter/nitrite_hive_adapter.dart';

var storeModule = HiveModule.withConfig()
            .path(filePath)
            .addTypeAdapter(GeometryAdapter())
            .build();

var db = await Nitrite.builder()
            .loadModule(storeModule)
            .loadModule(SpatialModule())
            .openOrCreate();
```

If Nitrite is using the default `SimpleNitriteMapper`, then the `SpatialModule` will automatically register the `GeometryConverter` with the mapper. If you are using your own custom mapper, then you need to register the `GeometryConverter` with the mapper.

### GeometryConverter

`GeometryConverter` is an [`EntityConverter`](../repository/mapper.md#entityconverter) which is used to convert `Geometry` object of Dart JTS to `Document` and vice-versa. It is used to store the `Geometry` object in the database.

## Spatial Index

Spatial module uses [R-Tree](https://en.wikipedia.org/wiki/R-tree) index to store the spatial data. 

To create a spatial index, you need to pass the index type as `spatialIndex` while creating the index. `spatialIndex` is a global constant defined in `nitrite_spatial` package.

```dart
await collection.createIndex(["location"], indexOptions(spatialIndex));
```

!!!info
Spatial index is not supported on multiple fields.
!!!

## Spatial Filter

Spatial module supports several filters to query spatial data. To know more about filters, please refer to [Spatial Filters](../filter.md#spatial-filters).
