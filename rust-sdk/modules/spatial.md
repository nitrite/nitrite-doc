---
label: Spatial Module
icon: globe
order: 2
---

The `nitrite-spatial` crate adds spatial indexes and spatial filter builders for geometry-aware queries.

## Add the dependency

```toml
[dependencies]
nitrite = "0.3"
nitrite-spatial = "0.3"
```

## Load the module

```rust
use nitrite::nitrite::Nitrite;
use nitrite_spatial::SpatialModule;

let db = Nitrite::builder()
    .load_module(SpatialModule)
    .open_or_create(None, None)
    .expect("failed to open database");
```

## Create a spatial index

```rust
use nitrite_spatial::spatial_index;

let collection = db.collection("locations").expect("collection open failed");
collection.create_index(vec!["location"], &spatial_index())
    .expect("spatial index failed");
```

## Query with spatial filters

The fluent entry point is `spatial_field(name)`.

```rust
use nitrite_spatial::{spatial_field, GeoPoint, Geometry};

let bbox = Geometry::envelope(-74.0, 40.7, -73.9, 40.9);
let mut within_city = collection.find(
    spatial_field("location").within(bbox)
).expect("spatial query failed");

let origin = GeoPoint::new(45.0, -93.265).expect("invalid point");
let mut nearby = collection.find(
    spatial_field("location")
        .geo_near(origin, 5_000.0)
        .expect("invalid geo_near query")
).expect("spatial query failed");

let _ = within_city.next();
let _ = nearby.next();
```

Available spatial operations include:

- `intersects(...)`
- `within(...)`
- `near(...)`
- `geo_near(...)`
- `knearest(...)`

The module also exports geometry helpers such as `Geometry`, `Point`, `GeoPoint`, `Coordinate`, and parsing helpers for WKT and GeoJSON.