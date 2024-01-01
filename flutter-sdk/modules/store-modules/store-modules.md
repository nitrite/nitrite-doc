---
label: Storage Module
icon: database
order: 4
---

Nitrite provides a default storage module which is in-memory. It means all the data is stored in memory and is volatile. Once the application is closed, all the data is lost. If you want to persist the data, you want to use on-disk storage, you need to add a storage module to your project. There are choices of storage modules available for Nitrite. You can choose from below as per your requirement.

- [Hive Module](hive.md)

You can also write your own storage module and add it to the project. More details about writing your own storage module is available [here](custom.md).