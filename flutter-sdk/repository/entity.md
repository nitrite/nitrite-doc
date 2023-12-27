---
label: Entity
icon: diamond
order: 15
---

An entity is a Dart object that can be stored in a collection. An entity can be a simple class with a few fields or a complex class with nested objects. Every entity in an `ObjectRepository` is actually converted to a `Document` before storing in the underlying `NitriteCollection`. When an entity is converted to a `Document`, the fields of the entity are mapped to the fields of the `Document`. While retrieving an entity from the `ObjectRepository`, the `Document` is converted back to the entity.

Nitrite uses a `NitriteMapper` implementation to convert an entity to a `Document` and vice versa. By default, Nitrite uses an `EntityConverter` based implementation of `NitriteMapper` to convert an entity to a `Document`. You can also provide your own implementation of `NitriteMapper` to convert an entity to a `Document`.

More on `NitriteMapper` can be found [here](mapper.md).

