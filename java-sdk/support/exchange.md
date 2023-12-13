---
label: Import/Export
icon: arrow-switch
order: 3
---

Nitrite provides import/export functionality via `nitrite-support` library. You can import/export your database to/from a file in JSON format. You can choose to import/export the entire database or a specific collection.

## Exporting Database

To export the entire database in JSON format, you can use the `Exporter` class of `nitrite-support` library. You can construct an instance of `Exporter` from an `ExportOptions` instance using `Exporter.withOptions` method. The `ExportOptions` class provides various options to configure the export process.

```java
// create an exporter with export options
Exporter exporter = new Exporter(exportOptions);

// export the database to a file
exporter.exportTo(dataFile);
```

The `exportTo` method can take a `File` or a `String` or an `OutputStream` or a `Writer` as an argument. If you pass a `File` or a `String` argument, the exporter will create a `FileOutputStream` and write the exported data to the file. If you pass an `OutputStream` or a `Writer` argument, the exporter will write the exported data to the stream or writer.

!!!info Important
The database must be in closed state before exporting.
!!!

### Export Options

The `ExportOptions` class provides various options to configure the export process. You can set the following options:

- `nitriteFactory` - the `NitriteFactory` instance to use for export. It is used to create the `Nitrite` instance for export. It's a mandatory field.
- `jsonFactory` - the `JsonFactory` instance to use for export. It is used to create the `JsonGenerator` instance for export. It's an optional field, if not provided, a default one will be used.
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
- `keyedRepositories` - the list of keyed repositories to export. The rules for specifying the keyed repository names are as follows:
    - If null is specified, all keyed repositories will be exported.
    - If an empty map is specified, no keyed repository will be exported.
    - If a non-empty map is specified, only the keyed repositories in the map will be exported.

### Example

First create a method to create a `Nitrite` instance, which will be used as the `nitriteFactory`.

```java
public Nitrite createDb(String filePath) {
    // create a MVStore based file store
    MVStoreModule storeModule = MVStoreModule.withConfig()
        .filePath(filePath)
        .build();

    // register all entity converters used in ObjectRepository
    SimpleNitriteMapper documentMapper = new SimpleNitriteMapper();
    documentMapper.registerEntityConverter(new Employee.EmployeeConverter());
    documentMapper.registerEntityConverter(new Company.CompanyConverter());
    documentMapper.registerEntityConverter(new Note.NoteConverter());

    // create an instance of nitrite database
    return Nitrite.builder()
        .loadModule(storeModule)
        .loadModule(module(documentMapper))
        .openOrCreate();
}
```

Then create the `ExportOptions` instance and configure it.

```java
// create export options
ExportOptions exportOptions = new ExportOptions();
// set the nitrite factory
exportOptions.setNitriteFactory(() -> createDb("test.db"));
// set the collections to export
exportOptions.setCollections(List.of("first"));
// set the repositories to export
exportOptions.setRepositories(List.of("org.dizitart.no2.support.data.Employee", "org.dizitart.no2.support.data.Company"));
// set the keyed repositories to export
exportOptions.setKeyedRepositories(Map.of("key", Set.of("org.dizitart.no2.support.data.Employee")));
```

Then create the `Exporter` instance and export the database.

```java
// create an exporter with export options
Exporter exporter = Exporter.withOptions(exportOptions);
exporter.exportTo("test.json");
```

The `test.json` file will contain the exported data.

## Importing Database

To import the entire database from an exported JSON data, you can use the `Importer` class of `nitrite-support` library. You can construct an instance of `Importer` from an `ImportOptions` instance using `Importer.withOptions` method. The `ImportOptions` class provides various options to configure the import process.

```java
// create an importer with import options
Importer importer = new Importer(importOptions);

// import the database from a file
importer.importFrom(dataFile);
```

The `importFrom` method can take a `File` or a `String` or an `InputStream` or a `Reader` as an argument. If you pass a `File` or a `String` argument, the importer will create a `FileInputStream` and read the exported data from the file. If you pass an `InputStream` or a `Reader` argument, the importer will read the exported data from the stream or reader.

!!!info Important
The database must be in closed state before importing.
!!!

### Import Options

The `ImportOptions` class provides various options to configure the import process. You can set the following options:

- `nitriteFactory` - the `NitriteFactory` instance to use for import. It is used to create the `Nitrite` instance for import. It's a mandatory field.
- `jsonFactory` - the `JsonFactory` instance to use for import. It is used to create the `JsonParser` instance for import. It's an optional field, if not provided, a default one will be used.

### Example

First create a `ImportOptions` instance and configure it using the `createDb` method from the previous example.

```java
// create import options
ImportOptions importOptions = new ImportOptions();
// set the nitrite factory
importOptions.setNitriteFactory(() -> createDb("new-test.db"));
```

Then create the `Importer` instance and import the database.

```java
// create an importer with import options
Importer importer = Importer.withOptions(importOptions);
importer.importFrom("test.json");
```

The `new-test.db` file will contain the imported data. You can open the database and use it.
