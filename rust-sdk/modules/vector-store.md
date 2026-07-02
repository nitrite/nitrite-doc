---
label: Vector Store (ANN / RAG)
icon: graph
order: 3
---

The `nitrite-vector` crate turns Nitrite into an **embedded vector database**. It
adds approximate-nearest-neighbour (ANN) indexes and a fluent kNN query builder,
plus a higher-level `RagStore` for retrieval-augmented-generation (RAG)
workloads — store text + embedding + metadata, search by similarity, and combine
the result with ordinary Nitrite metadata filters.

Embeddings are **provided by the caller** (bring-your-own vectors, like
sqlite-vec / usearch / lance); this crate stores and searches vectors, it does
not generate them.

## Two backends

You choose a backend per database. Both expose the exact same query API.

| Backend | Storage | Best for | Durability |
|---------|---------|----------|-----------|
| **HNSW** (default) | in-memory graph, persisted to Nitrite's KV store | indexes that fit in RAM | **per-write**, atomic with the DB |
| **DiskANN** | memory-mapped flat file + PQ codes in RAM | **indexes larger than RAM** (e.g. mobile) | flush-on-close (rebuildable) |

- **HNSW** — an in-memory Malkov–Yashunin graph, written through to the store on
  every change (crash-safe). Fastest when everything fits in memory.
- **DiskANN** — a disk-resident Vamana graph with full vectors in a
  memory-mapped file and product-quantized (PQ) codes resident in RAM for fast
  traversal, followed by an **exact re-rank** from the on-disk vectors. Resident
  memory is bounded by the OS page cache (cold pages are reclaimed under
  pressure — the index cannot OOM from its vector data), so it serves indexes
  much larger than RAM.

## Add the dependency

```toml
[dependencies]
nitrite = "0.4.2"
nitrite-vector = "0.4.2"
# a storage backend (also where the DiskANN files live):
nitrite-fjall-adapter = "0.4.2"
```

## Load the module

The module is configured through `VectorModule::builder(dim, metric)`.

+++ HNSW (in-memory)

The first argument is the embedding dimension. This walkthrough uses `3` so the
vectors are readable; real embeddings are typically 384–1536-dimensional.

```rust
use nitrite::nitrite::Nitrite;
use nitrite_vector::{VectorModule, Metric};

let db = Nitrite::builder()
    .load_module(VectorModule::builder(3, Metric::Cosine).build())
    .open_or_create(None, None)
    .expect("failed to open database");
```

+++ DiskANN (disk-resident)

```rust
use nitrite::nitrite::Nitrite;
use nitrite_vector::{VectorModule, IndexBackend, Precision, Metric};
use nitrite_fjall_adapter::FjallModule;

let module = VectorModule::builder(384, Metric::Cosine)
    .backend(IndexBackend::DiskAnn)
    .precision(Precision::F16)          // stored-vector precision
    .degree(64)                         // Vamana out-degree R
    .search_beam(100)                   // default query search width
    .pq_subvectors(16)                  // PQ bytes per code
    .pq_train_threshold(10_000)         // train PQ once this many vectors exist
    .cache_bytes(128 * 1024 * 1024)     // advisory (OS page cache bounds RAM)
    .build();

let db = Nitrite::builder()
    .load_module(FjallModule::with_config().db_path("./data").build())
    .load_module(module)
    .open_or_create(None, None)
    .expect("failed to open database");
```

+++

## Create a vector index

Create the index on the document field that holds the embedding array.

```rust
use nitrite_vector::vector_index_options;

let collection = db.collection("docs").expect("collection open failed");
collection.create_index(vec!["embedding"], &vector_index_options())
    .expect("vector index failed");
```

## Insert vectors

The embedding is a normal numeric array field on the document.

```rust
use nitrite::doc;

collection.insert(doc! { "title": "fox",  "embedding": [1.0, 0.0, 0.0] })
    .expect("insert failed");
collection.insert(doc! { "title": "wolf", "embedding": [0.9, 0.1, 0.0] })
    .expect("insert failed");
```

## Query with `vector_field`

The fluent entry point is `vector_field(name).nearest(query, k)`.

```rust
use nitrite_vector::vector_field;

let filter = vector_field("embedding")
    .nearest(vec![1.0, 0.0, 0.0], 5)   // query vector, k
    .ef(64)                            // optional: search width (recall vs latency)
    .min_score(0.5)                    // optional: similarity cutoff
    .build();

let mut nearest = collection.find(filter).expect("vector query failed");
let _ = nearest.next();
```

Results come back through the normal `DocumentCursor`, ordered nearest-first, so
they compose with projections, joins, and the rest of the collection API.

## RAG store

`RagStore` is a thin, convenient layer over a collection. It stores a `text`
field, an `embedding` field, and any metadata you pass; it does kNN and combines
the result with ordinary Nitrite filters, returning documents with similarity
scores.

```rust
use nitrite_vector::{RagStore, Metric};
use nitrite::doc;
use nitrite::filter::field;

// The db must be built with a VectorModule using the same metric.
let store = RagStore::create(&db, "kb", Metric::Cosine).expect("rag store failed");

let id = store
    .add("the quick brown fox", embedding, doc! { "source": "wiki" })
    .expect("add failed");

let hits = store
    .search(query_vector, 5)                 // top-5
    .filter(field("source").eq("wiki"))      // combine with metadata
    .min_score(0.75)                         // drop dissimilar hits
    .ef(128)                                 // search width
    .run()                                   // Vec<SearchHit { id, text, score, document }>
    .expect("search failed");

for hit in &hits {
    println!("{} (score {:.3})", hit.text, hit.score);
}

store.delete(&id).expect("delete failed");
```

`RagStore` also exposes `add_many`, `get`, `len`, `is_empty`, and `collection()`.

## Distance metrics

Choose the metric when you build the module: `Metric::Cosine`,
`Metric::Euclidean` (L2), or `Metric::Dot`. Cosine vectors are L2-normalized on
insert. Similarity scores are metric-aware (higher = more similar); `min_score`
uses that scale.

## Configuration reference

Every knob is on `VectorModule::builder(dim, metric)`. Resolved parameters are
persisted in the index header, so a reopened index keeps its settings.

| Knob | Backend | Meaning | Default |
|------|---------|---------|---------|
| `backend` | both | `Hnsw` or `DiskAnn` | `Hnsw` |
| `precision` | both | `F32` / `F16` / `I8` stored-vector encoding | `F32` |
| `m` | HNSW | graph connectivity `M` | 16 |
| `ef_construction` | HNSW | build search width | 200 |
| `ef_search` | HNSW | default query search width | 64 |
| `degree` | DiskANN | Vamana out-degree `R` | 64 |
| `build_beam` | DiskANN | construction search width `L` | 100 |
| `search_beam` | DiskANN | default query search width `L` | 100 |
| `alpha` | DiskANN | RobustPrune diversity slack (≥ 1.0) | 1.2 |
| `pq_subvectors` | DiskANN | PQ bytes per code; `0` = exact traversal | 16 |
| `pq_train_threshold` | DiskANN | train PQ once N vectors are indexed | 10 000 |
| `consolidate_threshold` | DiskANN | background delete-consolidation trigger; `0` = manual | 1000 |
| `cache_bytes` | DiskANN | advisory RAM budget (see Durability) | 64 MiB |

Per-query, `.ef(n)` on the fluent filter overrides `ef_search` (HNSW) or
`search_beam` (DiskANN) — turn it up for higher recall, down for lower latency.

## Precision

`Precision` selects the stored-vector codec, trading size for exactness:

| Precision | Bytes/dim | Notes |
|-----------|-----------|-------|
| `F32` | 4 | exact (default) |
| `F16` | 2 | IEEE half; ~exact for normalized embeddings |
| `I8`  | 1 | per-vector scalar quantization; ~4× smaller, approximate |

For DiskANN, PQ codes only *guide* traversal; the final ranking is always an
exact re-rank against the stored vectors at the chosen precision.

## Deletes & consolidation (DiskANN)

- A delete is **correct immediately**: the freed slot is held aside (never reused
  until cleaned), and stale in-edges resolve to a dead sentinel that queries skip.
- Once `consolidate_threshold` deletes accumulate, a **background thread**
  (single-flight, chunked so queries interleave) runs a FreshDiskANN-style pass:
  it drops references to deleted nodes, reconnects through their surviving
  neighbors, re-prunes to `degree`, and reclaims the slots.
- Closing the database runs a final synchronous consolidation so the persisted
  state is clean. You can also trigger it manually via `DiskAnnIndex::consolidate()`.

## Persistence & durability

| Backend | Storage | Durability |
|---------|---------|-----------|
| HNSW | Nitrite `NitriteMap` (e.g. fjall) | **per-write**, atomic with the DB |
| DiskANN | memory-mapped flat file + sidecar next to the DB | **flush-on-close** |

The DiskANN index is *derived data* (rebuildable from the documents), so a crash
before flush costs at most a reindex — the documents themselves are always safe
in the main store. Use the HNSW backend if you need per-write index durability.

`cache_bytes` is **advisory** for DiskANN: because the store is memory-mapped,
the OS page cache bounds resident memory and reclaims cold pages under pressure,
so the index cannot OOM from its vector data.

## Performance

The crate is tuned to approach hand-optimized ANN libraries:

- **Portable SIMD** distance kernels (`wide` f32x8) — pure Rust, **no C
  dependency**, so it cross-compiles cleanly to ARM NEON for mobile.
- **Fast integer hashing** (`rustc-hash`) on the query hot path.
- **Allocation-free** vector decode into reused buffers.
- **Lock-free per-query read view** (DiskANN) — one lock per query, not per node,
  so concurrent queries scale instead of contending.

Indicative release-build numbers on a 10-core laptop, 384-dim embeddings, 2k
index:

| Operation | Result |
|-----------|--------|
| HNSW query (k=10) | ~0.10 ms |
| DiskANN query (k=10, PQ + exact re-rank) | ~0.19 ms |
| Distance kernel (128-dim, SIMD) | ~12 ns |
| Build throughput | ~0.4 ms / vector inserted |
| DiskANN, 128 queries across 8 threads | **~4.6× faster** than single-threaded |
| Recall vs brute force | HNSW ≥ 0.95, DiskANN + PQ ≥ 0.90 |

Numbers vary with dataset, dimension, recall target, and hardware. Run the
benchmarks yourself:

```bash
cargo bench -p nitrite_vector
```

!!!info Build in release
ANN is float-heavy; always benchmark and deploy `--release`. A debug build is
roughly an order of magnitude slower.
!!!

## How it compares

`nitrite-vector` is **embedded / in-process**, so the fair comparison is other
in-process libraries, not networked databases:

| Category | Examples | Query latency includes |
|----------|----------|------------------------|
| In-process (fair) | usearch, hnswlib, FAISS-HNSW, sqlite-vec, LanceDB | just the index lookup |
| Client–server | Qdrant, Milvus, Weaviate, Pinecone, pgvector | index **+ network + serialization** |

- Against **networked** vector DBs, any in-process index avoids the network tax
  (often several milliseconds of round-trip and serialization per query).
- Against **in-process** libraries, the distance kernels and query concurrency
  are in the same class; on AVX-512 servers a hand-tuned kernel can still be
  faster (the portable SIMD targets AVX2 / NEON).
- The **DiskANN** backend adds something most embedded libraries don't: it serves
  indexes **larger than RAM** at roughly sub-millisecond query latency, at the
  cost of SSD reads and flush-on-close durability.

Treat these as *same-class-as-embedded-ANN-libraries*, not a head-to-head
benchmark win.

## When to use which backend

- **HNSW** — the index fits comfortably in RAM and you want the fastest queries
  and per-write durability. This is the default.
- **DiskANN** — the index is (or will grow) larger than RAM, e.g. large
  on-device corpora on mobile, and you can accept flush-on-close durability for a
  derived index.

For small text-matching needs, the [Full-Text Search](full-text-search.md)
module or the core `field("...").text(...)` filters may be all you need — reach
for the vector store when you have embeddings and want semantic / kNN search.
