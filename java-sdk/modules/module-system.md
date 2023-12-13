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

## NitriteModule

The `NitriteModule` interface is the base interface for all the modules. It encapsulates a set of plugins. A module must be loaded before opening a database. 

```java
MyModule module = new MyModule();
Nitrite db = Nitrite.builder()
    .loadModule(module)
    .openOrCreate("user", "password");
```

To create a module, you need to implement the `NitriteModule` interface. The `NitriteModule` interface has a single method `plugins` which returns a set of plugins. 

```java
public class MyModule implements NitriteModule {
    @Override
    public Set<NitritePlugin> plugins() {
        // return a set of plugins
    }
}
```

### Dynamic Module

A dynamic module can be created using the static `module` method of the `NitriteModule` interface. A dynamic module is a module which is not loaded from a jar file. It is created at runtime. 

```java
NitriteModule module = NitriteModule.module(new MyPlugin1(), new MyPlugin2());
Nitrite db = Nitrite.builder()
    .loadModule(module)
    .openOrCreate("user", "password");
```

A dynamic module is useful when you want to create a module from a set of plugins at runtime.

## NitritePlugin

The `NitritePlugin` interface is the base interface for all the plugins. A plugin is a single unit of functionality. A plugin must be loaded via a `NitriteModule` before opening a database. 

```java
MyPlugin plugin = new MyPlugin();
MyModule module = new MyModule(plugin);
Nitrite db = Nitrite.builder()
    .loadModule(module)
    .openOrCreate("user", "password");
```

### Initializing Plugin

A plugin can be initialized by implementing the `initialize` method of the `NitritePlugin` interface. The `initialize` method is called by the module when the plugin is loaded. 

```java
public class MyPlugin implements NitritePlugin {
    @Override
    public void initialize() {
        // initialize the plugin
    }
}
```

### Closing Plugin

A `NitritePlugin` is a closable resource. The `close` method is called by the Nitrite when the plugin is unloaded to release any resources held by the plugin. 

```java
public class MyPlugin implements NitritePlugin {
    @Override
    public void close() {
        // close the plugin
    }
}
```

## Modules

The following modules are available from Nitrite.

- [Storage Module](store-modules/store-modules.md)
- [Jackson Module](jackson.md)
- [Geo-spatial Module](spatial.md)

## Nitrite Bill of Materials

The `nitrite-bom` is a bill of materials (BOM) for Nitrite. It is a pom file that contains the version of all the modules. It is useful when you want to use multiple modules in your project and want to keep the version of all the modules in sync.

### Maven

Add the nitrite dependency to your `pom.xml` file:

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.dizitart</groupId>
            <artifactId>nitrite-bom</artifactId>
            <version>[latest version]</version>
            <scope>import</scope>
            <type>pom</type>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.dizitart</groupId>
        <artifactId>nitrite</artifactId>
    </dependency>

    <!-- add other modules here -->
    <dependency>
        <groupId>org.dizitart</groupId>
        <artifactId>nitrite-spatial</artifactId>
    </dependency>
    <dependency>
        <groupId>org.dizitart</groupId>
        <artifactId>nitrite-jackson-mapper</artifactId>
    </dependency>
</dependencies>
```

### Gradle

Add the nitrite dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation platform('org.dizitart:nitrite-bom:[latest version]')
    implementation 'org.dizitart:nitrite'

    // add other modules here
    implementation 'org.dizitart:nitrite-spatial'
    implementation 'org.dizitart:nitrite-jackson-mapper'
}
```



