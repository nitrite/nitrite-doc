---
label: Hive Modules
icon: heading
order: 10
---

Nitrite provides a persistent storage module based on [Hive](https://github.com/isar/hive). Hive is a lightweight and blazing fast, file based key-value store written in pure Dart.

## Adding Hive Module

To use Hive module, you need add below dependency to your `pubspec.yaml` file:

```yaml
dependencies:
  nitrite: ^[latest-version]
  nitrite_hive_adapter: ^[latest-version]
```

## Using Hive Module

To use Hive as your persistent storage, you need to load the `HiveModule` while opening the database. You can also configure the module as per your requirement.

Once the module is loaded, you can use the database as usual. All the data will be persisted in the file you specified.

```dart
import 'package:nitrite/nitrite.dart';
import 'package:nitrite_hive_adapter/nitrite_hive_adapter.dart';

// configure the module
var storeModule = HiveModule.withConfig()
            .path(filePath)
            .build();

var db = await Nitrite.builder()
            .loadModule(storeModule)
            .openOrCreate();
```

## Configuring Hive Module

You can configure the Hive module as per your requirement. The following configuration options are available:

- `path` - the file path where the database will be stored. It is a directory path.
- `backendPreference` - the preference for the backend to use. It takes an enum `HiveStorageBackendPreference`.
- `encryptionCipher` - the encryption cipher used to encrypt the database. If not specified, the database will not be encrypted. It takes a `HiveCipher` instance.
- `compactionStrategy` - the compaction strategy to use. It takes a function of type `CompactionStrategy`.
- `crashRecovery` - if set to `true`, the database will be recovered from crash. The default value is `false`.
- `addTypeAdapter()` - you can add your own type adapter to the Hive database. It takes a `TypeAdapter` instance.
- `addStoreEventListener()` - you can add your own store event listener to the Hive database. It takes a `StoreEventListener` function as argument.

Nitrite provides a builder pattern to configure the module. You can use the `HiveModule.withConfig()` method to get the builder instance. The builder has a `build()` method which returns the module instance.

```dart
var storeModule = HiveModule.withConfig()
            .path(filePath)
            .backendPreference(HiveStorageBackendPreference.native)
            .encryptionCipher(StandardAESEncryption(password))
            .compactionStrategy((entries, deletedEntries) => true)
            .crashRecovery(true)
            .addTypeAdapter(MyTypeAdapter())
            .addStoreEventListener((event) => print(event))
            .build();
```

For more details about Hive configuration, please visit [Hive Documentation](https://docs.hivedb.dev/).