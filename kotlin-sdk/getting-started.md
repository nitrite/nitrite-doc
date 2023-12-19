---
icon: rocket
label: Getting Started
description: 'This guide will help you get started with Nitrite database. It will show you how to create a database, create a collection, insert documents, and query documents in Kotlin.'
order: 20
---

# Getting Started in Kotlin

Nitrite's Kotlin SDK is a wrapper around the Java SDK. It provides some Kotlin friendly extensions for some of the Java API to work with Nitrite database. The Kotlin SDK is available as a separate module named `potassium-nitrite` (KNO<sub>2</sub>). In this guide, we will discuss the Kotlin specific API only. For rest of the API, please refer to the [Java SDK](../java-sdk/getting-started.md). So it is highly recommended to go through the Java SDK guide first.

To get started with Nitrite database, you need to add the Nitrite BOM to your project. The BOM will help you to manage the dependencies. Details of the BOM can be found [here](../java-sdk/modules/module-system.md#nitrite-bill-of-materials).

To add the BOM to your project, follow the steps below:

## Add dependency

To enable Kotlin support, you need to add the `potassium-nitrite` library to your project. It will automatically add the `nitrite` library as a dependency along with `nitrite-spatial` and `nitrite-jackson-mapper` libraries.

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
        <artifactId>potassium-nitrite</artifactId>
    </dependency>
</dependencies>
```

### Gradle

Add the nitrite dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation platform('org.dizitart:nitrite-bom:[latest version]')
    implementation 'org.dizitart:potassium-nitrite'
}
```

The latest released version of Nitrite can be found [here](https://mvnrepository.com/artifact/org.dizitart/potassium-nitrite).


## Snapshot builds

Snapshot builds are available from [Sonatype](https://oss.sonatype.org/content/repositories/snapshots/org/dizitart/nitrite-bom/).

To use snapshot builds, you need to add the snapshot repository to your `pom.xml` or `build.gradle` file:

### Maven

```xml
<repositories>
    <repository>
        <id>sonatype-snapshot</id>
        <url>https://oss.sonatype.org/content/repositories/snapshots/</url>
        <releases>
            <enabled>false</enabled>
        </releases>
        <snapshots>
            <enabled>true</enabled>
        </snapshots>
    </repository>
</repositories>
```

### Gradle

```groovy
repositories {
    maven {
        url "https://oss.sonatype.org/content/repositories/snapshots/"
    }
}
```

Nitrite's Kotlin SDK is not as lean as the Java SDK. It has a lot of dependencies. If you are using Kotlin in your project, it is recommended to use the Kotlin SDK. Otherwise, you can use the Java SDK with Kotlin as well.

## Upgrade from 3.x

If you are upgrading from 3.x, please note that there are lots of breaking changes in the API. The whole library is re-written from scratch. It is recommended to go through this guide before upgrading. 

!!!primary
You need to use the [MVStore](../java-sdk/modules/store-modules/mvstore.md) as your storage module to upgrade from 3.x. The RocksDB module is not backward compatible.
!!!

Nitrite will try to migrate your existing database to the latest version on the [!badge variant="warning" text="best effort basis without any guarantee"] provided you are using the MVStore module. If you are using the RocksDB module, you need to migrate your database manually. However, it is recommended to take a backup of your database before upgrading.