---
label: FAQ
icon: question
order: 4
---

# Frequently Asked Questions

## What is Nitrite?

Nitrite is a server-less embedded database ideal for desktop, mobile or small web applications. It's an open source, self-contained database. It supports both in-memory and file based persistent store.

## Is Nitrite an RDBMS?

No, Nitrite is not an RDBMS. It's a NoSQL database. It's a document store with support for indexing, full-text search, event listeners, embedded and in-memory modes, encryption, and many other features.

## What is the difference between Nitrite and SQLite?

Nitrite is a NoSQL document store, whereas SQLite is an RDBMS. Nitrite is a document store, whereas SQLite is a relational store. Nitrite has pure Java, Dart implementations, whereas SQLite has C/C++ implementation and native bindings are needed for other languages.

## Which platforms are supported by Nitrite?

Nitrite Java is supported on all platforms where Java is supported. Nitrite Flutter is supported on all platforms where Dart is supported. 

## When should I use Nitrite?

Nitrite is a good choice for desktop, mobile or small web applications. It's a good choice for applications where you don't want to install a separate database server. It's a good choice for applications where you want to store data in-memory.

## What is the performance of Nitrite?

Nitrite is highly performant, though it heavily depends on the underlying storage backend implementation. A JMH benchmark of Nitrite for Java is available [here](https://github.com/nitrite/nitrite-jmh) where you can find its performance amongst various storage backends against SQLite.

## Does Nitrite support replication over network?

No, Nitrite doesn't support replication over network. It's a server-less embedded database. If you are looking for replication, you can subscribe to collection events using `CollectionEventListener` and replicate the data over network.

## What happened to Nitrite DataGate Server?

Nitrite DataGate Server is now deprecated. It's no longer maintained due to lack of interest from the community. If you are looking for it to be revived, please vote [here](https://github.com/orgs/nitrite/discussions/1). If enough people are interested, it will be revived.

## What if I find any issues in the documentation?

If you find any issues in the documentation, please raise an issue [here](https://github.com/nitrite/nitrite-doc/issues).

## What if I had any suggestions/questions?

If you have any suggestions or questions about Nitrite, please start a discussion [here](https://github.com/orgs/nitrite/discussions).

## What is the license of Nitrite?

Nitrite is licensed under Apache License 2.0. It's free to use in both commercial and non-commercial applications. If you are using Nitrite in your commercial application, please consider donating to the project.

## How can I donate to Nitrite?

If you like Nitrite and want to support the project, please consider donating to the project. You can donate via [GitHub Sponsors](https://github.com/sponsors/anidotnet).