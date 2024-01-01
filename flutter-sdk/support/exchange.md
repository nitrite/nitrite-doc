---
label: Import/Export
icon: arrow-switch
order: 2
---

Nitrite provides import/export functionality via `nitrite-support` library. You can import/export your database to/from a file in JSON format. You can choose to import/export the entire database or a specific collection.

## Exporting Database

To export the entire database in JSON format, you can use the `Exporter` class of `nitrite-support` library. You can use the `Exporter.withOptions()` method to construct an instance of `Exporter`.

```dart
// create an exporter with export options
var exporter = Exporter.withOptions(
    dbFactory: () async => createDb('nitrite_source.db'),
    collections: ['first'],
    repositories: ['Employee'],
    keyedRepositories: {
        'key': {'Employee'},
    },
);

// export the database to a file
await exporter.exportTo(dataFile);
```

!!!primary
The database must be in closed state before exporting.
!!!

### Export Options

The `Exporter.withOptions()` method provides various options to configure the export process. You can set the following options:

- `dbFactory` - the `NitriteFactory` function to use for export. It is used to create the `Nitrite` instance for export. It's a mandatory field.
- `exportIndices` - whether to export indices or not. It's an optional field, if not provided, it will be set to `true`.
- `exportData` - whether to export data or not. It's an optional field, if not provided, it will be set to `true`.
- `collections` - the list of collections to export. The rules for specifying the collection names are as follows:
    - If null is specified, all collections will be exported.
    - If an empty list is specified, no collection will be exported.
    - If a non-empty list is specified, only the collections in the list will be exported.
- `repositories` - the list of repositories to export. The rules for specifying the repository names are as follows:
    - If null is specified, all repositories will be exported.
    - If an empty list is specified, no repository will be exported.
    - If a non-empty list is specified, only the repositories in the list will be exported.
- `keyedRepositories` - the map of keyed repositories to export. The rules for specifying the keyed repository names are as follows:
    - If null is specified, all keyed repositories will be exported.
    - If an empty map is specified, no keyed repository will be exported.
    - If a non-empty map is specified, only the keyed repositories in the map will be exported.

### Example

First create a method to create a `Nitrite` instance, which will be used as the `nitriteFactory`.

```dart
Future<Nitrite> createDb(String filePath) async {
    // create a Hive based file store
    var storeModule = Hive.withConfig()
        .path(filePath)
        .build();

    // open the database
    return Nitrite.builder()
        .loadModule(storeModule)
        .build();
}
```

Then create the `Exporter` instance and export the database.

```dart
// create an exporter with export options
var exporter = Exporter.withOptions(
    dbFactory: () async => createDb('nitrite_source.db'),
    collections: ['first'],
    repositories: ['Employee'],
    keyedRepositories: {
        'key': {'Employee'},
    },
);

// export the database to a file
await exporter.exportTo('source.json');
```

The `source.json` file will contain the exported data in JSON format.

## Importing Database

To import the entire database from an exported JSON data, you can use the `Importer` class of `nitrite-support` library. You can use the `Importer.withOptions()` method to construct an instance of `Importer`.

```dart
// create an importer with import options
var importer = Importer.withOptions(
    dbFactory: () async => createDb(getTempPath('nitrite_dest.db')),
);

// import the database from a file
await importer.importFrom('source.json');
```

!!!primary
The database must be in closed state before importing.
!!!

### Import Options

The `Importer.withOptions()` method provides various options to configure the import process. You can set the following options:

- `dbFactory` - the `NitriteFactory` function to use for import. It is used to create the `Nitrite` instance for import. It's a mandatory field.


### Example

First create the `Importer` instance and configure it.

```dart
// create an importer with import options
var importer = Importer.withOptions(
    dbFactory: () async => createDb('nitrite_dest.db'),
);

// import the database from a file
await importer.importFrom('source.json');
```

The `nitrite_dest.db` file will contain the imported data. You can open the database and use it.
