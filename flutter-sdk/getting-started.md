---
icon: rocket
label: Getting Started
description: 'This guide will help you get started with Nitrite database. It will show you how to create a database, create a collection, insert documents, and query documents in Flutter.'
order: 20
---

# Getting Started in Flutter

Nitrite is a pure Dart database. It does not depend on any native library. So it can be used in any platform where Dart is supported. It is a server-less embedded database ideal for desktop, mobile, or web applications. It is written in pure Dart and runs in Flutter, Dart VM, and the browser.

To get started with Nitrite database, you need to add the dependency to your pubspec.yaml file.

## Add dependency

Add Nitrite dependency to your project:

```yaml
dependencies:
  nitrite: ^[latest version]
```

The latest released version of Nitrite can be found [here](https://pub.dev/packages/nitrite).

To add the nitrite entity generator to your project, add the following to your `pubspec.yaml` file:

```yaml
dev_dependencies:
  build_runner: ^2.4.6
  nitrite_entity_generator: ^[latest version]
```

More details about the entity generator can be found [here](repository/entity.md#code-generator).

To add the hive adapter to your project, add the following to your `pubspec.yaml` file:

```yaml
dependencies:
  nitrite_hive_adapter: ^[latest version]
```

More details about the hive adapter can be found [here](modules/store-modules/hive.md).

!!!info
Nitrite is null safe. So you can use it in your null safe project.
!!!