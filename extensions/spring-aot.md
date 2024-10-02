# Spring Ahead of Time

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

[Spring AOT processing](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html#native-image.introducing-graalvm-native-images.understanding-aot-processing)
is part of the process to create a native binary from a Spring (Boot) application. This extension will help in adding a
lot of hints which are needed for Axon Framework. Please note this extension can only be used with Spring Boot 3, as
such it requires at least Java 17.

Besides the extension, it might be necessary to make more changes to successfully compile and run an application as a native
image. For example, when a message isn't used in a handler. This is quite common when the application is split, and the
application sending certain messages is not the same as the application handling the messages. In those cases these
messages need to be added to
the [ImportRuntimeHints](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/context/annotation/ImportRuntimeHints.html)
annotation. Otherwise, these messages can't be deserialized, leading to errors at runtime.

If something is not working or only works with additional hints, and it's Axon-specific, please let us know either
at [GitHub](https://github.com/AxonFramework/extension-spring-aot/issues)
or [Discuss](https://discuss.axoniq.io/c/axonframework/).

## Compiling to native

Before you set up this extension, it's important to read through
the [documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/native-image.html) from Spring
itself. There are some known limitations that might require additional changes to the application.
In addition, this extension needs to be added by adding the following dependency:

```xml

<dependency>
    <groupId>org.axonframework.extensions.spring-aot</groupId>
    <artifactId>axon-spring-aot</artifactId>
    <version>4.8.0</version>
</dependency>
```

This should be enough to have additional hints with ahead of time compilation to successfully build and run your Axon
application.

## Performance tips

It can be beneficial to move from JPA implementations to JDBC implementations. This likely decreases both the time it
takes to compile the image and the time to start the image.


