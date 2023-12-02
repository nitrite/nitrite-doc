---
icon: rocket
label: Getting Started
description: 'This guide will help you get started with Nitrite database. It will show you how to create a database, create a collection, insert documents, and query documents.'
order: 20
---

# Getting Started in Java

To get started with Nitrite database, you can follow the steps below:

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