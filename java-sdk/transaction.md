---
label: Transaction
icon: git-pull-request
order: 11
---

A transaction is a single logical unit of work which accesses and possibly modifies the contents of a database. Transactions access data using read and write operations.

Nitrite supports transactional operations on its collections and repositories. A transaction can be committed or rolled back. Once a transaction is committed, all the changes are persisted to the disk. If a transaction is rolled back, all the changes are discarded.


## Transaction on NitriteCollection

A transaction can be started from a session using the `Session.beginTransaction()` method. To start a transactional operation on a `NitriteCollection`, the `Transaction.getCollection()` method can be used. All the operations performed on the collection will be part of the transaction.

```java
// start a transaction
Transaction transaction = session.beginTransaction();

// get a collection
NitriteCollection collection = transaction.getCollection("test");

// perform operations on the collection
collection.insert(doc1);

// commit the transaction
transaction.commit();
```

`Transaction` is a closeable resource. It is recommended to use it with try-with-resource block.

```java
try (Session session = db.createSession()) {
    try (Transaction transaction = session.beginTransaction()) {
        NitriteCollection collection = transaction.getCollection("test");
        collection.insert(doc1);
        transaction.commit();
    }
}
```

!!!info
Any find operations performed inside a transaction will return all the documents including the uncommitted ones.

Any find operations performed outside a transaction will return only the committed documents.
!!!


### Auto-commit Operations

Certain operations are auto-committed in Nitrite. Those operations are not part of a transaction and cannot be rolled back. The following operations are auto-committed:

- `NitriteCollection.createIndex()`
- `NitriteCollection.rebuildIndex()`
- `NitriteCollection.dropIndex()`
- `NitriteCollection.dropAllIndices()`
- `NitriteCollection.drop()`
- `NitriteCollection.clear()`
- `NitriteCollection.close()`

## Transaction on ObjectRepository

A transaction can be started from a session using the `Session.beginTransaction()` method. To start a transactional operation on a `ObjectRepository`, the `Transaction.getRepository()` method can be used. All the operations performed on the repository will be part of the transaction.

```java
// start a transaction
Transaction transaction = session.beginTransaction();

// get a repository
ObjectRepository<Employee> repository = transaction.getRepository(Employee.class);

// perform operations on the repository
repository.insert(employee);

// commit the transaction
transaction.commit();
```

`Transaction` is a closeable resource. It is recommended to use it with try-with-resource block.

```java
try (Session session = db.createSession()) {
    try (Transaction transaction = session.beginTransaction()) {
        ObjectRepository<Employee> repository = transaction.getRepository(Employee.class);
        repository.insert(employee);
        transaction.commit();
    }
}
```

!!!info
Any find operations performed inside a transaction will return all the entities including the uncommitted ones.

Any find operations performed outside a transaction will return only the committed entities.
!!!

### Auto-commit Operations

Certain operations are auto-committed in Nitrite. Those operations are not part of a transaction and cannot be rolled back. The following operations are auto-committed:

- `ObjectRepository.createIndex()`
- `ObjectRepository.rebuildIndex()`
- `ObjectRepository.dropIndex()`
- `ObjectRepository.dropAllIndices()`
- `ObjectRepository.drop()`
- `ObjectRepository.clear()`
- `ObjectRepository.close()`


## Session

A session represents a transactional context for a Nitrite database. Session is used to create a new transaction. A session should be closed after use to release any resources held by it. If a session is closed and the transaction is not committed, all opened transactions will be rolled back.

### Create a Session

A session can be created using the `Nitrite.createSession()` method. Multiple sessions can be created for a Nitrite database.

```java
// create a session
Session session = db.createSession();
```

### Close a Session

A session can be closed using the `Session.close()` method. If a session is closed and the transaction is not committed, all opened transactions will be rolled back.

```java
// close a session
session.close();
```

### Checking Session State

The current state of a session can be checked using the `Session.checkState()` method. If the session is not active, it will throw an `TransactionException`.

```java
// check session state
session.checkState();
```

## Managing Transactions

A transaction can be started using the `Session.beginTransaction()` method.

```java
// start a transaction
Transaction transaction = session.beginTransaction();
```

A transaction can be committed using the `Transaction.commit()` method. If a transaction is committed, all the changes are persisted to the disk.

```java
// commit a transaction
transaction.commit();
```

A transaction can be rolled back using the `Transaction.rollback()` method. If a transaction is rolled back, all the changes are discarded.

```java
// rollback a transaction
transaction.rollback();
```

A transaction can be closed using the `Transaction.close()` method. If a transaction is closed and the transaction is not committed, all opened transactions will be rolled back.

```java
// close a transaction
transaction.close();
```

### Querying Transaction State

The current state of a transaction can be retrieved using the `Transaction.getState()` method. It returns an enum of type `TransactionState`.

```java
// get transaction state
TransactionState state = transaction.getState();
```

The following states are available:

- `TransactionState.Active` - The transaction is active.
- `TransactionState.Committed` - The transaction is committed.
- `TransactionState.PartiallyCommitted` - The transaction is partially committed.
- `TransactionState.Closed` - The transaction is closed.
- `TransactionState.Failed` - The transaction is failed.
- `TransactionState.Aborted` - The transaction is aborted.


