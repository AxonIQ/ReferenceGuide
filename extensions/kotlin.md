# Kotlin

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

[Kotlin](https://kotlinlang.org/) is a programming language which interoperates fully with Java and the JVM. As Axon is written in Java it can be used in conjunction with Kotlin too, offering a different feel when using the framework.

Some of Axon's API's work perfectly well in Java, but have a rather awkward feel when transitioning over to Kotlin. The goal of the [Kotlin Extension](https://github.com/AxonFramework/extension-kotlin) is to remove that awkwardness, by providing [inline and reified](https://kotlinlang.org/docs/reference/inline-functions.html) methods of Axon's API.

Several solutions are currently given, which can roughly be segregated into the distinct types of messages used by Axon. This thus provides a [commands](#commands), [events](#events) and [queries](#queries) section on this page.

> **Experimental Release**
>
> Currently, the [Kotlin Extension](https://github.com/AxonFramework/extension-kotlin) has been release experimentally (e.g. release 0.1.0).
> This means that all implementations are subject to change until a full release (e.g. a release 1.0.0) has been made.

## Commands

This section describes the additional functionality attached to Axon's [command dispatching and handling](../axon-framework/axon-framework-commands/README.md) logic.

### CommandGateway

An inlined method has been introduced on the `CommandGateway` which allows the introduction of a dedicated function to be invoked upon success or failure of handling the command. As such it provides a shorthand instead of using the [`CommandCallback`](../axon-framework/axon-framework-commands/infrastructure.md) directly yourself.

Here is a sample of how this can be utilized within your own project:

```kotlin
import org.axonframework.commandhandling.CommandMessage
import org.axonframework.commandhandling.gateway.CommandGateway
import org.axonframework.messaging.MetaData
import org.slf4j.LoggerFactory

class CommandDispatcher(private val commandGateway: CommandGateway) {
    
    private val logger = LoggerFactory.getLogger(CommandDispatcher::class.java)

    // Sample usage providing specific logging logic, next to for example the LoggingInterceptor
    fun issueCardCommand() {
        commandGateway.send(
                command = IssueCardCommand(),
                onSuccess = { message: CommandMessage<out IssueCardCommand>, result: Any, _: MetaData ->
                    logger.info("Successfully handled [{}], resulting in [{}]", message, result)
                },
                onError = { result: Any, exception: Throwable, _: MetaData ->
                    logger.warn(
                            "Failed handling the IssueCardCommand, with output [{} and exception [{}]",
                            result, exception
                    )
                }
        )
    }
}

class IssueCardCommand
```

## Events

This section describes the additional functionality attached to Axon's [event publication and handling](../axon-framework/events/README.md) logic.

### Event Upcasters

A simplified implementation of the [Single Event Upcaster](../axon-framework/events/event-versioning.md#event-upcasting) is given, which allows for a shorter implementation cycle.
Making an upcaster to upcast the `CardIssuedEvent` from revision `0` to `1` can be written as follows:

```kotlin
import com.fasterxml.jackson.databind.JsonNode
import org.axonframework.serialization.upcasting.event.SingleEventUpcaster

fun `CardIssuedEvent 0 to 1 Upcaster`(): SingleEventUpcaster =
        EventUpcaster.singleEventUpcaster(
                eventType = CardIssuedEvent::class,
                storageType = JsonNode::class,
                revisions = Revisions("0", "1")
        ) { event ->
            // Perform your upcasting process of the CardIssuedEvent here
            event
        }

class CardIssuedEvent
```
Alternatively, since `Revisions` is essentially a `Pair` of `String`, it is also possible to use Kotlin's [`to` function](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/to.html):

```kotlin               
EventUpcaster.singleEventUpcaster(
        eventType = CardIssuedEvent::class,
        storageType = JsonNode::class,
        revisions = "0" to "1"
) { event ->
    // Perform your upcasting process of the CardIssuedEvent here
    event
}
```
## Queries

This section describes the additional functionality attached to Axon's [query dispatching and handling](../axon-framework/queries/README.md) logic.

### QueryGateway

Several inlined methods have been introduced on the `QueryGateway` to use generics instead of explicit `Class` objects and `ResponseType` parameters. 

```kotlin
import org.axonframework.queryhandling.QueryGateway

class QueryDispatcher(private val queryGateway: QueryGateway) {
    fun getTotalNumberOfCards(): Int {
           val query = CountCardSummariesQuery()
           // Query will return a CompletableFuture so it has to be handled
           return queryGateway.query<Int, CountCardSummariesQuery>(query)
                   .join()
       }
}

data class CountCardSummariesQuery(val filter: String = "")
```

In some cases, Kotlin's type inference system can deduce types without explicit generic parameters. One example of this would be an explicit return parameter:

```kotlin
import org.axonframework.queryhandling.QueryGateway
import java.util.concurrent.CompletableFuture

class QueryDispatcher(private val queryGateway: QueryGateway) {
    fun getTotalNumberOfCards(): CompletableFuture<Int> =
            queryGateway.query(CountCardSummariesQuery())
}

data class CountCardSummariesQuery(val filter: String = "")
```

There are multiple variants of the `query` method provided, for each type of `ResponseType`:
- `query`
- `queryOptional`
- `queryMany`

### QueryUpdateEmitter

An inline `emit` method has been added to `QueryUpdateEmitter` to simplify emit method's call by using generics and moving the lambda predicate at the end of parameter list. This way the lambda function can be moved outside of the parentheses.
```kotlin
import org.axonframework.queryhandling.QueryUpdateEmitter
import org.axonframework.eventhandling.EventHandler

class CardSummaryProjection (private val queryUpdateEmitter : QueryUpdateEmitter) {
    @EventHandler
    fun on(event : CardIssuedEvent) {
        // Update projection here

        // Then emit the CountChangedUpdate to subscribers of CountCardSummariesQuery
        // with the given filter
        queryUpdateEmitter
                .emit<CountCardSummariesQuery, CountChangedUpdate>(CountChangedUpdate()) { query ->
                    // Sample filter based on ID field
                    event.id.startsWith(query.idFilter)
                }
    }
}

class CardIssuedEvent(val id : String)
class CountChangedUpdate
data class CountCardSummariesQuery(val idFilter: String = "")
```
