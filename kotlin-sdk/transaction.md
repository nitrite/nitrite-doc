---
label: Transaction
icon: git-pull-request
order: 11
---

Nitrite supports transactional operations on its collections and repositories. A transaction can be committed or rolled back. Once a transaction is committed, all the changes are persisted to the disk. If a transaction is rolled back, all the changes are discarded.

More information about Nitrite transactions can be found [here](../java-sdk/transaction.md).

## Kotlin API for Transaction

Potassium Nitrite provides a set of builder functions for `Session` and `Transaction` to make transaction usage more like natural to Kotlin.

## Session

A session represents a transactional context for a Nitrite database. Session is used to create a new transaction. More details on session can be found [here](../java-sdk/transaction.md#session).

### Kotlin API for Session

To create a new session, you can use the `session()` extension function on `Nitrite`.

```kotlin
val session = db.session {
    // perform transactional operations
}
```

Session will be automatically closed once the block is executed.

## Transaction

A transaction is a single logical unit of work which accesses and possibly modifies the contents of a database. Transactions access data using read and write operations. More details on transaction can be found [here](../java-sdk/transaction.md#managing-transactions).

### Kotlin API for Transaction

To start a transaction, you can use the `tx()` extension function on `Session`.

```kotlin
db.session {
    tx {
        // perform transactional operations
        val collection = getCollection("test")
        collection.insert(doc1)

        // commit the transaction
        commit()
    }
}
```

Transaction will be automatically closed once the block is executed.