`Document` is Nitrite's schemaless record type. It stores string keys and `Value` entries, supports nested documents and arrays, and uses a lock-free persistent map internally so cloning is cheap.

## Create documents

You can create a document explicitly with `Document::new()` or use the `doc!` macro for inline literals.

```rust
use nitrite::collection::Document;
use nitrite::doc;

let mut manual = Document::new();
manual.put("name", "Alice").expect("put failed");
manual.put("age", 30_i64).expect("put failed");

let inline = doc! {
    "name": "Bob",
    "age": 28,
    "active": true
};
```

## Nested fields and arrays

By default, Nitrite uses `.` as the nested field separator. That means `address.city` targets the `city` field inside an embedded `address` document.

```rust
use nitrite::common::Value;
use nitrite::doc;

let document = doc! {
    "name": "Warehouse A",
    "address": {
        "city": "Berlin",
        "zip": 10115
    },
    "tags": ["inventory", "primary"]
};

assert_eq!(
    document.get("address.city").expect("lookup failed"),
    Value::String("Berlin".to_string())
);
assert_eq!(
    document.get("tags.0").expect("lookup failed"),
    Value::String("inventory".to_string())
);
```

If a field does not exist, `get()` returns `Value::Null` instead of failing.

```rust
use nitrite::common::Value;
use nitrite::doc;

let document = doc! { "name": "Alice" };
let missing = document.get("profile.timezone").expect("lookup failed");

assert!(matches!(missing, Value::Null));
```

If you configure a different separator through `NitriteBuilder::field_separator()`, the same rule applies with the new delimiter.

## Updating fields

`put()` inserts a new value or overwrites an existing one.

```rust
use nitrite::collection::Document;

let mut document = Document::new();
document.put("status", "draft").expect("put failed");
document.put("status", "published").expect("put failed");
document.put("author.name", "Nitrite Team").expect("put failed");
```

This is the same document shape that collection updates consume.

## Reserved fields

Nitrite manages a small set of reserved metadata fields on every stored document:

- `_id`
- `_revision`
- `_source`
- `_modified`

The `_id` field is special. It is a `NitriteId` and is generated automatically when needed.

```rust
use nitrite::doc;

let mut document = doc! { "name": "Alice" };
let id = document.id().expect("id generation failed");

println!("document id: {}", id);
```

You cannot assign `_id` to an arbitrary non-`NitriteId` value through `put()`.

## Iteration and conversion

Documents can be iterated key by key, passed through processors, and converted to `Value::Document` for repository mapping. In most collection code you will work with them directly, while typed repositories use the same document representation behind the scenes.

If you need compile-time mapping instead of raw document access, move to [Object Repository](repository/intro.md).
