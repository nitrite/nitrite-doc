Collections also expose metadata, lifecycle, event, and processing hooks through the shared `PersistentCollection` interface.

## Collection events

You can subscribe to collection events such as insert, update, remove, and index lifecycle notifications.

```rust
use nitrite::collection::{CollectionEventInfo, CollectionEventListener, CollectionEvents};

let subscriber = collection.subscribe(CollectionEventListener::new(
    |event: CollectionEventInfo| {
        match event.event_type() {
            CollectionEvents::Insert => println!("insert event from {}", event.originator()),
            CollectionEvents::Update => println!("update event"),
            CollectionEvents::Remove => println!("remove event"),
            CollectionEvents::IndexStart | CollectionEvents::IndexEnd => {}
        }
        Ok(())
    },
)).expect("subscribe failed");

if let Some(handle) = subscriber {
    collection.unsubscribe(handle).expect("unsubscribe failed");
}
```

## Processors

Processors transform documents before writes and after reads. They are useful for enrichment, normalization, or field redaction.

```rust
use nitrite::collection::Document;
use nitrite::common::{Processor, ProcessorProvider};
use nitrite::errors::NitriteResult;

struct AuditProcessor;

impl ProcessorProvider for AuditProcessor {
    fn name(&self) -> String {
        "audit-processor".to_string()
    }

    fn process_before_write(&self, mut doc: Document) -> NitriteResult<Document> {
        doc.put("updated_at", 1_725_000_000_i64)?;
        Ok(doc)
    }

    fn process_after_read(&self, doc: Document) -> NitriteResult<Document> {
        Ok(doc)
    }
}

collection.add_processor(Processor::new(AuditProcessor))
    .expect("processor registration failed");
```

## Attributes

Collections can store custom metadata through `Attributes`.

```rust
use nitrite::common::{Attributes, Value};

let mut attributes = Attributes::new_for_collection("users");
attributes.put("retention_policy", Value::String("90-days".to_string()));

collection.set_attributes(attributes).expect("set attributes failed");

let stored = collection.attributes().expect("attributes failed");
assert!(stored.is_some());
```

## Lifecycle and maintenance

`NitriteCollection` also exposes administrative methods that are useful for maintenance flows.

- `size()`
- `clear()`
- `is_open()`
- `is_dropped()`
- `close()`
- `dispose()`
- `store()`

```rust
let size = collection.size().expect("size failed");
let open = collection.is_open().expect("is_open failed");

println!("collection size: {}, open: {}", size, open);
```

Use `dispose()` with care: it is intended for permanently discarding the collection state, not just closing a handle.
