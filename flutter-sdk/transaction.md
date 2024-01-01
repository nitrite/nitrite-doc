---
label: Transaction
icon: git-pull-request
order: 11
---

A transaction is a single logical unit of work which accesses and possibly modifies the contents of a database. Transactions access data using read and write operations.

Nitrite supports transactional operations on its collections and repositories. A transaction can be committed or rolled back. Once a transaction is committed, all the changes are persisted to the disk. If a transaction is rolled back, all the changes are discarded.

## Transaction on NitriteCollection

A transaction can be started from a session using the `Session.beginTransaction()` method. To start a transactional operation on a `NitriteCollection`, the `Transaction.getCollection()` method can be used. All the operations performed on the collection will be part of the transaction.

```dart
// start a transaction
var transaction = await session.beginTransaction();

// get a collection
var collection = await transaction.getCollection("test");

// perform operations on the collection
await collection.insert(doc1);

// commit the transaction
await transaction.commit();
```

Nitrite also provides a `Session.executeTransaction()` method to execute a transactional operation. The `Session.executeTransaction()` method takes a callback function as an argument. The callback function will be executed inside a transaction.

```dart
await session.executeTransaction((transaction) async {
  var collection = await transaction.getCollection("test");
  await collection.insert(doc1);
});
```

If any exception occurs inside the transaction, the transaction will be rolled back. If you want to rollback a transaction only for certain exceptions, you can use the `Session.executeTransaction()` method with `rollbackFor` parameter. The `rollbackFor` parameter takes a list of exception types for which the transaction will be rolled back. For any other exceptions, the transaction will be committed.

```dart
await session.executeTransaction((transaction) async {
  var collection = await transaction.getCollection("test");
  await collection.insert(doc1);
}, rollbackFor: [NitriteException]);
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

```dart
// start a transaction
var transaction = await session.beginTransaction();

// get a repository
var repository = await transaction.getRepository<Employee>();

// perform operations on the repository
await repository.insert(employee);

// commit the transaction
await transaction.commit();
```

Nitrite also provides a `Session.executeTransaction()` method to execute a transactional operation. The `Session.executeTransaction()` method takes a callback function as an argument. The callback function will be executed inside a transaction.

```dart
await session.executeTransaction((transaction) async {
  var repository = await transaction.getRepository<Employee>();
  await repository.insert(employee);
});
```

If any exception occurs inside the transaction, the transaction will be rolled back. If you want to rollback a transaction only for certain exceptions, you can use the `Session.executeTransaction()` method with `rollbackFor` parameter. The `rollbackFor` parameter takes a list of exception types for which the transaction will be rolled back. For any other exceptions, the transaction will be committed.

```dart
await session.executeTransaction((transaction) async {
  var repository = await transaction.getRepository<Employee>();
  await repository.insert(employee);
}, rollbackFor: [NitriteException]);
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

```dart
// create a session
var session = db.createSession();
```

### Close a Session

A session can be closed using the `Session.close()` method. If a session is closed and the transaction is not committed, all opened transactions will be rolled back.

```java
// close a session
await session.close();
```

### Checking If Session is Active

A session can be checked if it is active or not using the `Session.isActive` getter.

```dart
// check if session is active
if (session.isActive) {
  // do something
}
```

## Managing Transactions

A transaction can be started using the `Session.beginTransaction()` method.

```dart
// start a transaction
var transaction = await session.beginTransaction();
```

A transaction can be committed using the `Transaction.commit()` method. If a transaction is committed, all the changes are persisted to the disk.

```dart
// commit a transaction
await transaction.commit();
```

A transaction can be rolled back using the `Transaction.rollback()` method. If a transaction is rolled back, all the changes are discarded.

```dart
// rollback a transaction
await transaction.rollback();
```

A transaction can be closed using the `Transaction.close()` method. If a transaction is closed and the transaction is not committed, all opened transactions will be rolled back.

```dart
// close a transaction
await transaction.close();
```

### Querying Transaction State

The current state of a transaction can be retrieved using the `Transaction.getState()` method. It returns an enum of type `TransactionState`.

```dart
// get transaction state
var state = transaction.getState();
```

Possible values for `TransactionState` are:

- `TransactionState.active` - The transaction is active.
- `TransactionState.committed` - The transaction is committed.
- `TransactionState.partiallyCommitted` - The transaction is partially committed.
- `TransactionState.closed` - The transaction is closed.
- `TransactionState.failed` - The transaction is failed.
- `TransactionState.aborted` - The transaction is aborted.
