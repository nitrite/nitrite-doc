---
icon: rocket
label: Getting Started
description: 'This guide will help you get started with Nitrite database. It will show you how to create a database, create a collection, insert documents, and query documents.'
order: 20
---

# Getting Started in Java

To get started with Nitrite database, you need to add the Nitrite BOM to your project. The BOM will help you to manage the dependencies. Details of the BOM can be found [here](modules/module-system.md#nitrite-bill-of-materials).

To add the BOM to your project, follow the steps below:

## Add dependency

Add Nitrite dependency to your project:

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
</dependencies>
```

### Gradle

Add the nitrite dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation platform('org.dizitart:nitrite-bom:[latest version]')
    implementation 'org.dizitart:nitrite'
}
```

The latest released version of Nitrite can be found [here](https://mvnrepository.com/artifact/org.dizitart/nitrite).


## Snapshot builds

Snapshot builds are available from [Sonatype](https://oss.sonatype.org/content/repositories/snapshots/org/dizitart/nitrite-bom/).

To use snapshot builds, you need to add the following repository to your `pom.xml` file:

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

Or, if you are using Gradle, add the following repository to your `build.gradle` file:

```groovy
repositories {
    mavenCentral()
    maven {
        url 'https://oss.sonatype.org/content/repositories/snapshots/'
    }
}
```

## Upgrade from 3.x

If you are upgrading from 3.x, please note that there are lots of breaking changes in the API. The whole library is re-written from scratch. It is recommended to go through this guide before upgrading. 

!!!primary
You need to use the [MVStore](modules/store-modules/mvstore.md) as your storage module to upgrade from 3.x. The RocksDB module is not backward compatible.
!!!

Nitrite will try to migrate your existing database to the latest version on the [!badge variant="warning" text="best effort basis without any guarantee"] provided you are using the MVStore module. If you are using the RocksDB module, you need to migrate your database manually. However, it is recommended to take a backup of your database before upgrading.