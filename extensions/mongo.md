# Mongo

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

The `MongoEventStorageEngine` has an `@PostConstruct` annotated method, called `ensureIndexes` which will generate the indexes required for correct operation. That means, when running in a container that automatically calls `@PostConstruct` handlers, the required unique index on "Aggregate Identifier" and "Event Sequence Number" is created when the event store is created.

Note that there is always a balance between query optimization and update speed. Load testing is ultimately the best way to discover which indices provide the best performance.

* Normal operational use

  An index is automatically created on `"aggregateIdentifier"`, `"type"` and `"sequenceNumber"` in the domain events \(default name: `"domainevents"`\) collection. Additionally, a non-unique index on `"timestamp"` and `"sequenceNumber"` is configured on the domain events \(default name: `"domainevents"`\) collection, for tracking event processors.

* Snapshotting

  A \(unique\) index on `"aggregateIdentifier"` and `"sequenceNumber"` is automatically created in the snapshot events \(default name: `"snapshotevents"`\) collection.

* Sagas

  Put a \(unique\) index on the `"sagaIdentifier"` in the saga \(default name: `"sagas"`\) collection. Put an index on the `"sagaType"`, `"associations.key"` and `"associations.value"` properties in the saga \(default name: `"sagas"`\) collection.

* Dead letter queue

  Put a \(unique\) index on the combination of `"processingGroup"`, `"sequenceIdentifier"` and `"index"` in the dead letter \(default name: `"deadletters"`\) collection. Put an index on the `"processingGroup"`, and `"sequenceIdentifier"` properties in the dead letter \(default name: `"deadletters"`\) collection. Put an index on the `"processingGroup"` property in the dead letter \(default name: `"deadletters"`\) collection.

{% hint style="info" %}
In pre Axon Framework 3 release we found MongoDb to be a very good fit as an Event Store. However, with the introduction of Tracking Event Processors and how they track their events, we have encountered some inefficiencies regarding the Mongo Event Store implementation. We recommend using a built-for-purpose event store like [Axon Server](../axon-server-introduction.md), or alternatively an RDBMS based \(the JPA or JDBC implementations for example\), and would only suggest to use Mongo for this use case if you have found its performance to be beneficial for your application.
{% endhint %}

## Configuration of the Event Store with Spring

```java
@Configuration
public class AxonConfig {
    // omitting other configuration methods...
  
    // The EmbeddedEventStore delegates actual storage and retrieval of events to an EventStorageEngine.
    @Bean
    public EventStore eventStore(EventStorageEngine storageEngine,
                                 GlobalMetricRegistry metricRegistry) {
        return EmbeddedEventStore.builder()
                                 .storageEngine(storageEngine)
                                 .messageMonitor(metricRegistry.registerEventBus("eventStore"))
                                 .spanFactory(spanFactory)
                                 // ...
                                 .build();
    }
  
    // The MongoEventStorageEngine stores each event in a separate MongoDB document.
    @Bean
    public EventStorageEngine storageEngine(MongoDatabaseFactory factory,
                                            TransactionManager transactionManager) {
        return MongoEventStorageEngine.builder()
                                      .mongoTemplate(SpringMongoTemplate.builder()
                                                                        .factory(factory)
                                                                        .build())
                                      .transactionManager(transactionManager)
                                      // ...
                                      .build();
    }
}
```

## Configuration in Spring Boot

This extension can be added as a Spring Boot starter dependency to your project using group id `org.axonframework.extensions.mongo` and artifact id `axon-mongo-spring-boot-starter`. When using the autoconfiguration, by  default the following components will be created for you automatically:
* A `MongoTransactionManager` to enable transactions with Mongo.
* A `SpringMongoTransactionManager`, this is the wrapped Spring mongo transaction manager, and will also be injected where applicable in other components created by the auto-config.
* A `SpringMongoTemplate`, this will use a `MongoDatabaseFactory` that should be available. To use transaction with Mongo the collections need to be accessed in a certain way, and this component makes sure of that.
* A `MongoTokenStore`, this will be used by the event processors to can keep track which events have been processed, and which segments are claimed.
* A `MongoSagaStore`, this will be used to store and retrieve saga's.

It's also possible to autoconfigure the `StorageStrategy` and `EventStorageEngine` by setting the `mongo.event-store.enabled` to true. The creation of the token store and the saga store can be turned off by setting `mongo.token-store.enabled` or `mongo.saga-store.enabled` to false. It's also possible to use a different database for the axon collections than the default the `MongoDatabaseFactory` uses by setting the `axon.mongo.database-name` property.

The relevant configuration could look like this:

```yaml
spring:
  data:
    mongodb:
      uri: mongodb://localhost:27017/test
  mongo:
    database-name: axon
    token-store:
      enabled: true
    saga-store:
      enabled: false
    event-store:
      enabled: false
```

While `test` is the default database name, for the axon collections the `axon` database will be used instead. The saga store will not be initialised.

## Configuration of the Mongo Dead-Letter Queue with Spring

See [Dead-Letter Queue](../axon-framework/events/event-processors/README.md#dead-letter-queue) for the general information about the Dead-Letter Queue.

```java
@Configuration
public class AxonConfig {
  // omitting other configuration methods...
  @Bean
  public ConfigurerModule deadLetterQueueConfigurerModule(
          MongoTemplate mongoTemplate    
  ) {
    // Replace "my-processing-group" for the processing group you want to configure the DLQ on.
    return configurer -> configurer.eventProcessing().registerDeadLetterQueue(
            "my-processing-group",
            config -> MongoSequencedDeadLetterQueue.builder()
                                                 .processingGroup("my-processing-group")
                                                 .maxSequences(256)
                                                 .maxSequenceSize(256)
                                                 .mongoTemplate(mongoTemplate)
                                                 .transactionManager(config.getComponent(TransactionManager.class))
                                                 .serializer(config.serializer())
                                                 .build()
    );
  }
}
```

