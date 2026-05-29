Here we discuss a Rust project tracker example that exercises most of Nitrite Rust's current feature set. It uses a Fjall-backed database for persistence, a typed repository for structured task data, a document collection for flexible activity records, Tantivy full-text search for text discovery, the spatial module for location-based lookups, transactions for batched writes, projection for lightweight reads, and sorted queries for reporting. The full runnable source code is available [here](https://github.com/nitrite/nitrite-rust/blob/main/nitrite-int-test/examples/rust_feature_showcase.rs).

## Setup

Add the following dependencies to your `Cargo.toml` file:

```toml
[dependencies]
nitrite = "0.2"
nitrite_derive = "0.2"
nitrite_fjall_adapter = "0.2"
nitrite_spatial = "0.2"
nitrite_tantivy_fts = "0.2"
uuid = { version = "1", features = ["v4"] }
```

## Domain Model

The example keeps structured project data in a typed repository and uses entity-derived indexes for the common access paths:

```rust
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Debug, Clone, Convertible, NitriteEntity, Default)]
#[entity(
    name = "tasks",
    id(field = "id"),
    index(type = "unique", fields = "slug"),
    index(type = "non-unique", fields = "status"),
    index(type = "non-unique", fields = "owner")
)]
struct Task {
    pub id: i64,
    pub slug: String,
    pub title: String,
    pub owner: String,
    pub status: String,
    pub priority: i64,
    pub estimate_hours: i64,
    pub tags: Vec<String>,
}

#[derive(Debug, Clone, Convertible, NitriteEntity, Default)]
struct TaskSummary {
    pub title: Option<String>,
    pub status: Option<String>,
    pub priority: Option<i64>,
    pub owner: Option<String>,
}
```

## Database Initialization

The database is opened with the Fjall storage adapter and both optional search modules loaded:

```rust
use nitrite::nitrite::Nitrite;
use nitrite_fjall_adapter::FjallModule;
use nitrite_spatial::SpatialModule;
use nitrite_tantivy_fts::TantivyFtsModule;

let fjall_module = FjallModule::with_config()
    .db_path(db_path)
    .build();

let db = Nitrite::builder()
    .load_module(fjall_module)
    .load_module(TantivyFtsModule::default())
    .load_module(SpatialModule)
    .open_or_create(None, None)?;
```

The example also creates a flexible document collection with dedicated FTS and spatial indexes:

```rust
use nitrite_tantivy_fts::fts_index;
use nitrite_spatial::spatial_index;

let activity = db.collection("task_activity")?;
activity.create_index(vec!["summary"], &fts_index())?;
activity.create_index(vec!["location"], &spatial_index())?;
```

## Transactional Writes

Initial task data is written inside a session transaction so the structured repository changes commit together before the indexed activity documents are added:

```rust
use nitrite::doc;
use nitrite::repository::ObjectRepository;

db.with_session(|session| {
    let tx = session.begin_transaction()?;
    let tx_tasks: ObjectRepository<Task> = tx.repository()?;

    tx_tasks.insert(Task::new(
        1,
        "release-dashboard",
        "Prepare the release dashboard",
        "alice",
        "open",
        3,
        8,
        &["release", "dashboard"],
    ))?;

    tx.commit()
})?;
```

The indexed activity collection is then populated through the normal collection API:

```rust
use nitrite::doc;

let activity = db.collection("task_activity")?;
activity.insert(doc! {
    "task_slug": "release-dashboard",
    "summary": "Release dashboard shipped with offline sync metrics",
    "location": {
        "x": (-122.335167f64),
        "y": 47.608013f64
    }
})?;
```

## Queries, Projection, and Reporting

After the seed write, the example demonstrates repository filtering, projection, sorted reads, updates, full-text search, and spatial queries:

```rust
use nitrite::collection::order_by;
use nitrite::common::SortOrder;
use nitrite::filter::{all, and, field};
use nitrite_tantivy_fts::fts_field;
use nitrite_spatial::{spatial_field, Geometry};

let tasks: ObjectRepository<Task> = db.repository()?;
let high_priority_open = tasks.find(and(vec![
    field("status").eq("open"),
    field("priority").gte(2i64),
]))?;

let mut owner_cursor = tasks.find(field("owner").eq("alice"))?;
let summaries = owner_cursor.project::<TaskSummary>()?;

let ordered = tasks.find_with_options(all(), &order_by("priority", SortOrder::Descending))?;

let activity = db.collection("task_activity")?;
let text_matches = activity.find(fts_field("summary").matches("offline sync"))?;

let seattle = Geometry::envelope(-122.5, 47.5, -122.2, 47.7);
let nearby = activity.find(spatial_field("location").within(seattle))?;
```

## Run the Example

The source lives in the integration-test crate because it already depends on the optional Nitrite Rust modules used by the showcase example.

```bash
cd nitrite-rust
cargo run -p nitrite_int_test --example rust_feature_showcase
```

When the command finishes, it prints counts and summaries for the repository, full-text, and spatial queries, then closes and removes the temporary Fjall database directory.
