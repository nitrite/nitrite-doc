---
label: Jackson Mapper Module
icon: zap
order: 1
---

Nitrite provides a Jackson mapper module to convert Java objects to `Document` and vice-versa. The module is useful when you don't want to use the `EntityConverter` interface to map Java objects to `Document` and vice-versa.

## Adding Jackson Mapper Module

Add the Jackson mapper module to your project:

### Maven

Add the Jackson mapper module dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dizitart</groupId>
    <artifactId>nitrite-jackson-mapper</artifactId>
</dependency>
```

### Gradle

Add the Jackson mapper module dependency to your `build.gradle` file:

```groovy
dependencies {
    implementation 'org.dizitart:nitrite-jackson-mapper'
}
```

## Using Jackson Mapper Module

To use Jackson mapper module, you need to load the `JacksonMapperModule` while opening the database.

```java
Nitrite db = Nitrite.builder()
            .loadModule(new JacksonMapperModule())
            .openOrCreate();
```

### Using Custom Jackson Modules

If you want to register any custom Jackson module, you can do so by passing the module to `JacksonMapperModule` constructor.

```java
Nitrite db = Nitrite.builder()
            .loadModule(new JacksonMapperModule(new MyCustomModule()))
            .openOrCreate();
```

If you want to register multiple modules, you can do so by passing an array of modules to `JacksonMapperModule` constructor.

```java
Nitrite db = Nitrite.builder()
            .loadModule(new JacksonMapperModule(new MyCustomModule(), new MyAnotherCustomModule()))
            .openOrCreate();
```

And here is an example of a custom Jackson module.

```java
public class MyCustomModule extends SimpleModule {

    @Override
    public void setupModule(SetupContext context) {
        addSerializer(MyCustomType.class, new MyCustomTypeSerializer());
        addDeserializer(MyCustomType.class, new MyCustomTypeDeserializer());
        super.setupModule(context);
    }
}
```

## Customizing ObjectMapper

If you want to customize the `ObjectMapper` used by the Jackson mapper module, you can do so by extending the `JacksonMapper` class and overriding the `getObjectMapper()` method.

```java
public class MyCustomJacksonMapper extends JacksonMapper {
    @Override
    public ObjectMapper getObjectMapper() {
        ObjectMapper objectMapper = super.getObjectMapper();
        objectMapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        return objectMapper;
    }
}
```

Now, you can use this custom mapper while building the database.

```java
Nitrite db = Nitrite.builder()
            .loadModule(NitriteModule.module(new MyCustomJacksonMapper()))
            .openOrCreate();
```
