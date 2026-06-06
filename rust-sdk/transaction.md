---
label: Transaction
icon: shield-check
order: 11
---

Nitrite transactions are session-based. A `Session` owns one or more `NitriteTransaction` instances, and each transaction provides isolated collection and repository views until you commit or roll back.

## Execute work with `with_session`

`Nitrite::with_session()` is the simplest way to run transactional work.

```rust
use nitrite::doc;
use nitrite::nitrite::Nitrite;

let db = Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

db.with_session(|session| {
    let tx = session.begin_transaction()?;

    let users = tx.collection("users")?;
    let audit_log = tx.collection("audit_log")?;

    users.insert(doc! {
        "name": "Alice",
        "active": true
    })?;

    audit_log.insert(doc! {
        "event": "user-created",
        "user": "Alice"
    })?;

    tx.commit()
}).expect("transaction failed");
```

This keeps transaction setup local to the database handle and ensures the session is cleaned up when the closure ends.

## Session model

A `Session` is the parent transactional context. It has:

- an ID
- an active/closed lifecycle
- a registry of active transactions

When a session closes, any uncommitted transactions are rolled back.

That matters in two situations:

- if you call `rollback()` explicitly
- if your `with_session()` closure returns an error before `commit()` runs

## Transactional collections and repositories

Inside a transaction, you access data through the transaction handle instead of the root database handle.

```rust
use nitrite::filter::field;
use nitrite::repository::ObjectRepository;
use nitrite_derive::{Convertible, NitriteEntity};

#[derive(Default, Convertible, NitriteEntity)]
#[entity(id(field = "id"))]
struct Account {
    id: i64,
    owner: String,
    balance: i64,
}

let db = nitrite::nitrite::Nitrite::builder()
    .open_or_create(None, None)
    .expect("failed to open database");

db.with_session(|session| {
    let tx = session.begin_transaction()?;
    let repo: ObjectRepository<Account> = tx.repository()?;

    let mut accounts = repo.find(field("owner").eq("Alice"))?;
    let current = accounts.first().transpose()?;

    if let Some(mut account) = current {
        account.balance += 100;
        repo.update_one(account, false)?;
    }

    tx.commit()
}).expect("transaction failed");
```

`NitriteTransaction` supports:

- `collection(name)`
- `repository::<T>()`
- `keyed_repository::<T>(key)`
- `commit()`
- `rollback()`

## Commit and rollback semantics

Nitrite's transaction implementation coordinates multi-collection and multi-repository work through a dedicated transaction store and a two-phase commit flow.

In practice, that means:

- reads inside the transaction see the transactional view
- writes are isolated until commit succeeds
- rollback discards uncommitted changes across all touched collections and repositories in that transaction

If you need an explicit abort path, call `rollback()` directly.

```rust
db.with_session(|session| {
    let tx = session.begin_transaction()?;
    let collection = tx.collection("jobs")?;

    collection.insert(nitrite::doc! { "state": "running" })?;

    let should_abort = true;
    if should_abort {
        tx.rollback()?;
        return Ok(());
    }

    tx.commit()
}).expect("transaction failed");
```

## Crash-atomic writes on Fjall

On a Fjall-backed database in the current 0.4 line, a logical write now lands in one atomic store transaction. That includes:

- a transaction `commit()` across all touched collections and repositories
- a single `insert`, `update`, or `remove` together with its index updates

After an unclean stop, Nitrite recovers to either the state before that logical write or the state after it. You do not get a persisted data row without its index entries, or the reverse.

## When to use transactions

Use a transaction when a sequence of writes must succeed or fail as a unit, especially when:

- more than one collection is involved
- you are updating multiple repositories together
- you want predictable rollback behavior on validation or business-rule failures