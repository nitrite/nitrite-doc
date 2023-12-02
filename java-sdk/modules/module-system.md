---
label: Module System
icon: gear
order: 4
---

Nitrite is a modular database engine. The core functionality of Nitrite is provided by the `nitrite` module. The `nitrite` module is the only mandatory module for Nitrite. All other modules are optional and can be added to the project as per the requirement.

Nitrite's module system is built on top of `NitritePlugin` and `NitriteModule` interfaces. A module is a collection of plugins. While opening a database, all the modules must be loaded. Each module then load and initialize all the plugins it contains.

The `nitrite` module provides default implementation of all the interfaces. For example, the `nitrite` module provides in-memory storage implementation. If you want to use on-disk storage, you need to add a storage module to your project.

The main advantage of Nitrite's module system is that it allows you to extend the functionality of Nitrite without adding all the dependencies at once in your project. You only need to add dependencies that you want to use. This helps to keep the application size small. 

You can write your own module and plugin and add it to the project. The module system also allows you to replace the default implementation of any plugin. For example, if you want to use a different storage engine, you can write your own storage module and add it to the project.



