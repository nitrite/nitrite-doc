---
icon: rocket
label: Getting Started
description: 'This guide will help you get started with Nitrite database. It will show you how to create a database, create a collection, insert documents, and query documents in Flutter.'
order: 20
---

# Getting Started in Flutter

Nitrite is a pure Dart database. It does not depend on any native library. So it can be used in any platform where Dart is supported. It is a server-less embedded database ideal for desktop, mobile, or web applications. It is written in pure Dart and runs in Flutter, Dart VM, and the browser.

## Add dependency

To use Nitrite in you project, add the following package to your package:

```bash
dart pub add nitrite
```

To add the nitrite code generator to your project:

```bash
dart pub add --dev build_runner
dart pub add --dev nitrite_generator
```

More details about the code generator can be found [here](repository/codegen.md).

To use the hive storage adapter to your project add the below package:

```bash
dart pub add nitrite_hive_adapter
```

More details about the hive adapter can be found [here](modules/store-modules/hive.md).

!!!info
Nitrite is null safe. So you can use it in your null safe project.
!!!