---
label: Filters
icon: filter
order: 15
---

The Rust SDK exposes a fluent filter DSL through `field("...")`. Filters can be combined, negated, and reused across collections and repositories.

## Equality and comparison filters

Start with `field(name)` and finish with a comparison operator.

```rust
use nitrite::filter::field;

let by_name = field("name").eq("Alice");
let not_deleted = field("deleted").ne(true);
let adults = field("age").gte(18);
let seniors = field("age").gt(64);
let before_cutoff = field("score").lt(50);
let at_most = field("priority").lte(3);
```

## Range filters

Nitrite provides three range helpers depending on how explicit you want the bound rules to be.

```rust
use nitrite::filter::field;

let inclusive = field("age").between_optional_inclusive(18, 65);
let same_rule = field("age").between_inclusive(18, 65, true);
let custom = field("price").between(10, 20, true, false);
```

On indexed fields in the current 0.3 line, bounded ranges such as `between(...)` and paired `gte(...)`/`lte(...)` filters use bounded index scans instead of a broad one-sided scan plus a post-filter.

## Text, regex, and array filters

Core filters cover substring matching, regex matching, membership, and array element matching.

```rust
use nitrite::filter::field;

let contains = field("address").text("street");
let contains_ignore_case = field("address").text_case_insensitive("main street");
let regex = field("email").text_regex(".*@example\\.com$");
let roles = field("role").in_array(vec!["admin", "editor"]);
let excluded = field("status").not_in_array(vec!["archived", "deleted"]);
let matching_tags = field("tags").elem_match(field("name").eq("database"));
```

## Logical composition

You can compose filters either as methods on `Filter` or with the top-level helpers.

```rust
use nitrite::filter::{all, and, field, not, or};

let active_adults = field("active")
    .eq(true)
    .and(field("age").gte(18));

let privileged = or(vec![
    field("role").eq("admin"),
    field("role").eq("moderator"),
]);

let visible = and(vec![active_adults, not(field("deleted").eq(true)), privileged]);
let everything = all();
```

## Match by document ID

If you already have a `NitriteId`, you can query directly by internal document identifier.

```rust
use nitrite::collection::NitriteId;
use nitrite::filter::by_id;

let id = NitriteId::new();
let filter = by_id(id);
```

This is mostly useful in collection-level workflows. Repository-level code usually prefers `get_by_id()` or `update_by_nitrite_id()`.

## Using filters with collections and repositories

The same `Filter` values work with both collection and repository query APIs.

```rust
use nitrite::filter::field;

let filter = field("name").eq("Alice");

let documents = collection.find(filter.clone()).expect("collection query failed");
let users = repository.find(filter).expect("repository query failed");
```

## Specialized filters from modules

The core filter module covers general document queries. Optional modules add dedicated fluent APIs for richer index types:

- `nitrite_spatial::spatial_field("location")` for spatial intersections, containment, and nearest-neighbor queries
- `nitrite_tantivy_fts::fts_field("content")` for Tantivy-backed full-text search

Those module-specific filters are documented in [Spatial Module](modules/spatial.md) and [Full-Text Search](modules/full-text-search.md).