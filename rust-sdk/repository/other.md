---
label: Other Operations
icon: tools
order: 11
---

Repositories inherit the same persistent-collection services as collections and also expose repository-specific helpers such as `document_collection()`.

## Work with the backing collection

`document_collection()` gives you the raw collection behind the repository.

```rust
let collection = repo.document_collection();
let size = collection.size().expect("size failed");
println!("raw collection size: {}", size);
```

This is useful when a typed workflow needs to drop down to raw document inspection or low-level maintenance.

## Events, processors, and attributes

Because repositories implement `PersistentCollection`, you can attach the same hooks you would use on a collection.

```rust
use nitrite::collection::CollectionEventListener;
use nitrite::common::{Attributes, Processor, ProcessorProvider, Value};
use nitrite::errors::NitriteResult;

repo.subscribe(CollectionEventListener::new(|event| {
    println!("repository event: {:?}", event.event_type());
    Ok(())
})).expect("subscribe failed");

struct TouchProcessor;

impl ProcessorProvider for TouchProcessor {
    fn name(&self) -> String {
        "touch-processor".to_string()
    }

    fn process_before_write(
        &self,
        mut doc: nitrite::collection::Document,
    ) -> NitriteResult<nitrite::collection::Document> {
        doc.put("touched", true)?;
        Ok(doc)
    }

    fn process_after_read(
        &self,
        doc: nitrite::collection::Document,
    ) -> NitriteResult<nitrite::collection::Document> {
        Ok(doc)
    }
}

repo.add_processor(Processor::new(TouchProcessor))
    .expect("processor registration failed");

let mut attributes = Attributes::new_for_collection("User");
attributes.put("stream", Value::String("primary".to_string()));
repo.set_attributes(attributes).expect("set attributes failed");
```

## Lifecycle and maintenance

Repositories also expose collection-like maintenance operations:

- `size()`
- `clear()`
- `is_open()`
- `close()`
- `dispose()`
- `store()`

Use `clear()` when you want to keep the repository structure but remove its data. Use disposal and destruction APIs only when the repository itself should be discarded.

## Destroy repositories from the database handle

Repository destruction happens at the database level.

```rust
db.destroy_repository::<User>().expect("destroy_repository failed");
db.destroy_keyed_repository::<User>("archived")
    .expect("destroy_keyed_repository failed");
```

That keeps repository lifecycle consistent whether the handle was typed or keyed.