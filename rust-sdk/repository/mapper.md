Rust Nitrite does not use the Java or Flutter `NitriteMapper` model. Instead, object mapping is part of the type contract: repositories require entities to implement `Convertible`, and repository metadata comes from `NitriteEntity`.

## The core mapping trait

`Convertible` has two responsibilities:

- `to_value()` converts a Rust value into Nitrite's `Value`
- `from_value()` reconstructs the Rust value from `Value`

For repository entities, `to_value()` normally returns `Value::Document(...)`.

## Prefer derive macros

For straightforward structs, derive `Convertible` and `NitriteEntity` together.

```rust
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct Product {
    id: i64,
    sku: String,
    price_cents: i64,
}
```

This is the recommended default because it keeps mapping boilerplate out of the application layer.

## Manual `Convertible` implementations

If you need custom field transformations, manual validation, or an intentionally different persisted shape, implement `Convertible` yourself.

```rust
use nitrite::collection::Document;
use nitrite::common::{Convertible, Value};
use nitrite::errors::{ErrorKind, NitriteError, NitriteResult};

#[derive(Default)]
struct Product {
    sku: String,
    price_cents: i64,
}

impl Convertible for Product {
    type Output = Product;

    fn to_value(&self) -> NitriteResult<Value> {
        let mut doc = Document::new();
        doc.put("sku", self.sku.clone())?;
        doc.put("price_cents", self.price_cents)?;
        Ok(Value::Document(doc))
    }

    fn from_value(value: &Value) -> NitriteResult<Self::Output> {
        let doc = match value {
            Value::Document(document) => document,
            _ => {
                return Err(NitriteError::new(
                    "expected document value",
                    ErrorKind::ObjectMappingError,
                ));
            }
        };

        let sku = match doc.get("sku")? {
            Value::String(text) => text,
            _ => {
                return Err(NitriteError::new(
                    "expected string sku",
                    ErrorKind::ObjectMappingError,
                ));
            }
        };
        let price_cents = match doc.get("price_cents")? {
            Value::I64(value) => value,
            _ => {
                return Err(NitriteError::new(
                    "expected i64 price_cents",
                    ErrorKind::ObjectMappingError,
                ));
            }
        };

        Ok(Product { sku, price_cents })
    }
}
```

## Mapping nested values

`Convertible` is implemented for many standard Rust and Nitrite value types, so nested fields, vectors, sets, maps, options, boxed values, and tuples can participate in mapping without extra glue code.

That makes the derive macros the best option for most repository models. Reach for a manual implementation only when the persisted document shape should differ from the in-memory struct.
