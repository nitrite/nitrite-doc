---
label: Installation
icon: code
order: 1
---

Nitrite provides some additional functionalities via `nitrite-support` library. You can use the library to enable encryption, import/export database, and database recovery.

## Adding nitrite-support

Add the `nitrite-support` dependency to your project:

### Maven

Add the `nitrite-support` dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dizitart</groupId>
    <artifactId>nitrite-support</artifactId>
</dependency>
```

### Gradle

Add the `nitrite-support` dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation 'org.dizitart:nitrite-support'
}
```