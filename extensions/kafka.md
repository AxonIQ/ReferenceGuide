# Kafka

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

Apache Kafka is a very popular system for publishing and consuming events. Its architecture is fundamentally different from most messaging systems and combines speed with reliability.

Axon provides an extension dedicated to _publishing_ and _receiving_ event messages from Kafka. The Kafka Extension should be regarded as an alternative approach to distributing events, besides \(the default\) Axon Server. It's also possible to use the extension to stream events from Kafka to Axon server, or the other way around.

The implementation of the extension can be found [here](https://github.com/AxonFramework/extension-kafka). The shared repository also contains a [sample project](https://github.com/AxonFramework/extension-kafka/tree/master/kafka-axon-example) using the extension.

To use the Kafka Extension components from Axon, make sure the `axon-kafka` module is available on the classpath. Using the extension requires setting up and configuring Kafka following your project's requirements. How this is achieved is outside of the scope of this reference guide and should be found in Kafka's [documentation](https://kafka.apache.org/).

{% hint style="info" %}
Note that Kafka is a perfectly fine event distribution mechanism, but it is not an event store. Along those lines this extension **only** provides the means to distributed Axon's events through Kafka. Due to this the extension cannot be used to event source aggregates, as this requires an event store implementation. We recommend using a built-for-purpose event store like [Axon Server](../axon-server-introduction.md), or alternatively an RDBMS based \(the JPA or JDBC implementations for example\).
{% endhint %}

## Publishing Events to Kafka

When Event Messages are published to an Event Bus \(or Event Store\), they can be forwarded to a Kafka topic using the `KafkaPublisher`. To achieve this it will utilize a Kafka `Producer`, retrieved through Axon's `ProducerFactory`. The `KafkaPublisher` in turn receives the events to publish from a `KafkaEventPublisher`.

Since the `KafkaEventPublisher` is an event message handler in Axon terms, we can provide it to any [Event Processor](../axon-framework/events/event-processors/README.md) to receive the published events. The choice of event processor which brings differing characteristics for event publication to Kafka:

* **Subscribing Event Processor** - publication of messages to Kafka will occur in the same thread \(and Unit of Work\) which published the events to the event bus.
  This approach ensures failure to publish to Kafka enforces failure of the initial event publication on the event bus

* **Tracking Event Processor** - publication of messages to Kafka is run in a different thread \(and Unit of Work\) than the one which published the events to the event bus.
  This approach ensures the event has been published on the event bus regardless of whether publication to Kafka works

When setting up event publication it is also important to take into account which `ConfirmationMode` is used. The `ConfirmationMode` influences the process of actually producing an event message on a Kafka topic, but also what kind of `Producer` the `ProducerFactory` will instantiate:

* **TRANSACTIONAL** - This will require the `Producer` to start, commit and \(in case of failure\) rollback the transaction of publishing an event message.
  Alongside this, it will create a pool of `Producer` instances in the `ProducerFactory` to avoid continuous creation of new ones, requiring the user to provide a "transactional id prefix" to uniquely identify every `Producer` in the pool.

* **WAIT\_FOR\_ACK** - Setting "WAIT\_FOR\_ACK" as the `ConfirmationMode` will require the `Producer` instance to wait for a default of 1 second \(configurable on the `KafkaPublisher`\) until the event message publication has been acknowledged.
  Alongside this, it will create a single, shareable `Producer` instance from within the `ProducerFactory`.

* **NONE** - This is the _default_ mode, which only ensures a single, shareable `Producer` instance from within the `ProducerFactory`.

### Configuring Event Publication to Kafka

It is a several step process to configure Event publication to Kafka, which starts with the `ProducerFactory`. Axon provides the `DefaultProducerFactory` implementation of the `ProducerFactory`, which should be instantiated through the provided `DefaultProducerFactory.Builder`.

The builder has one hard requirement, which is the `Producer` configuration `Map`. The `Map` contains the settings to use for the Kafka `Producer` client, such as the Kafka instance locations. Please check the Kafka [documentation](https://kafka.apache.org/) for the possible settings and their values.

```java
public class KafkaEventPublicationConfiguration {
    // ...
    public ProducerFactory<String, byte[]> producerFactory(Duration closeTimeout,
                                                           int producerCacheSize,
                                                           Map<String, Object> producerConfiguration,
                                                           ConfirmationMode confirmationMode,
                                                           String transactionIdPrefix) {
        return DefaultProducerFactory.<String, byte[]>builder()
                .closeTimeout(closeTimeout)                 // Defaults to "30" seconds
                .producerCacheSize(producerCacheSize)       // Defaults to "10"; only used for "TRANSACTIONAL" mode
                .configuration(producerConfiguration)       // Hard requirement
                .confirmationMode(confirmationMode)         // Defaults to a Confirmation Mode of "NONE"
                .transactionalIdPrefix(transactionIdPrefix) // Hard requirement when in "TRANSACTIONAL" mode
                .build();
    }
    // ...
}
```

The second infrastructure component to introduce is the `KafkaPublisher`, which has a hard requirement on the `ProducerFactory`. Additionally, this would be the place to define the Kafka topics upon which Axon event messages will be published. You can set a function from event to `Optional<String>`. You can use this to only publish certain events, or put different events to different topics. Its not uncommon for Kafka topics to only contain one type of message. Note that the `KafkaPublisher` needs to be `shutDown` properly, to ensure all `Producer` instances are properly closed.

```java
public class KafkaEventPublicationConfiguration {
    // ...

    public KafkaPublisher<String, byte[]> kafkaPublisher(String topic,
                                                         ProducerFactory<String, byte[]> producerFactory,
                                                         KafkaMessageConverter<String, byte[]> kafkaMessageConverter,
                                                         int publisherAckTimeout) {
        return KafkaPublisher.<String, byte[]>builder()
                .topicResolver(m -> Optional.of(topic))     // Defaults to "Axon.Events" for all events
                .producerFactory(producerFactory)           // Hard requirement
                .messageConverter(kafkaMessageConverter)    // Defaults to a "DefaultKafkaMessageConverter"
                .publisherAckTimeout(publisherAckTimeout)   // Defaults to "1000" milliseconds; only used for "WAIT_FOR_ACK" mode
                .build();
    }
    // ...
}
```

Lastly, we need to provide Axon's event messages to the `KafkaPublisher`. To that end a `KafkaEventPublisher` should be instantiated through the builder pattern. Remember to add the `KafkaEventPublisher` to an event processor implementation of your choice. It is recommended to use the `KafkaEventPublisher#DEFAULT_PROCESSING_GROUP` as the processing group name of the event processor to distinguish it from other event processors.

```java
public class KafkaEventPublicationConfiguration {
    // ...
    public KafkaEventPublisher<String, byte[]> kafkaEventPublisher(KafkaPublisher<String, byte[]> kafkaPublisher) {
        return KafkaEventPublisher.<String, byte[]>builder()
                .kafkaPublisher(kafkaPublisher)             // Hard requirement
                .build();
    }

    public void registerPublisherToEventProcessor(EventProcessingConfigurer eventProcessingConfigurer,
                                                  KafkaEventPublisher<String, byte[]> kafkaEventPublisher) {
        String processingGroup = KafkaEventPublisher.DEFAULT_PROCESSING_GROUP;
        eventProcessingConfigurer.registerEventHandler(configuration -> kafkaEventPublisher)
                                 .assignHandlerTypesMatching(
                                         processingGroup,
                                         clazz -> clazz.isAssignableFrom(KafkaEventPublisher.class)
                                 )
                                 .registerSubscribingEventProcessor(processingGroup);
        // Replace `registerSubscribingEventProcessor` for `registerTrackingEventProcessor` to use a tracking processor
    }
    // ...
}
```

### Topic partition publication considerations

Kafka ensures message ordering on a topic-partition level, not on an entire topic. To control events of a certain group to be placed in a dedicated partition, based on aggregate identifier for example, the [message converter's](kafka.md#customizing-event-message-format) `SequencingPolicy` can be utilized.

The topic-partition pair events have been published in also has impact on event consumption. This extension mitigates any ordering concerns with the [streamable](kafka.md#consuming-events-with-a-streamable-message-source) solution, by ensuring a `Consumer` always receives **all** events of a topic to be able to perform a complete ordering. This guarantee is however not given when using the [subscribable](kafka.md#consuming-events-with-a-subscribable-message-source) event consumption approach. The subscribable stream leaves all the ordering specifics in the hands of Kafka, which means the events should be published on a consistent partition to ensure ordering.

## Consuming Events from Kafka

Event messages in an Axon application can be consumed through either a Subscribing or a Tracking [Event Processor](../axon-framework/events/event-processors/README.md). Both options are maintained when it comes to consuming events from a Kafka topic, which from a set-up perspective translates to a [SubscribableMessageSource](kafka.md#consuming-events-with-a-subscribable-message-source) or a [StreamableKafkaMessageSource](kafka.md#consuming-events-with-a-streamable-message-source) respectively, Both will be described in more detail later on, as we first shed light on the general requirements for event consumption in Axon through Kafka.

Both approaches use a similar mechanism to poll events with a Kafka `Consumer`, which breaks down to a combination of a `ConsumerFactory` and a `Fetcher`. The extension provides a `DefaultConsumerFactory`, whose sole requirement is a `Map` of configuration properties. The `Map` contains the settings to use for the Kafka `Consumer` client, such as the Kafka instance locations. Please check the Kafka [documentation](https://kafka.apache.org/) for the possible settings and their values.

```java
public class KafkaEventConsumptionConfiguration {
    // ...
    public ConsumerFactory<String, byte[]> consumerFactory(Map<String, Object> consumerConfiguration) {
        return new DefaultConsumerFactory<>(consumerConfiguration);
    }
    // ...
}
```

It is the `Fetcher` instance's job to retrieve the actual messages from Kafka by directing a `Consumer` instance it receives from the message source. You can draft up your own implementation or use the provided `AsyncFetcher` to this end. The `AsyncFetcher` doesn't need to be explicitly started, as it will react on the message source starting it. It does need to be shut down, to ensure any thread pool or active connections are properly closed.

```java
public class KafkaEventConsumptionConfiguration {
    // ...
    public Fetcher<?, ?, ?> fetcher(long timeoutMillis,
                                    ExecutorService executorService) {
        return AsyncFetcher.builder()
                           .pollTimeout(timeoutMillis)          // Defaults to "5000" milliseconds
                           .executorService(executorService)    // Defaults to a cached thread pool executor
                           .build();
    }
    // ...
}
```

### Consuming Events with a Subscribable Message Source

Using the `SubscribableKafkaMessageSource` means you are inclined to use a `SubscribingEventProcessor` to consume the events in your event handlers.

When using this source, Kafka's idea of pairing `Consumer` instances into "Consumer Groups" is used. This is strengthened by making the `groupId` upon source construction a _hard requirement_. To use a common `groupId` essentially means that the event-stream-workload can be shared on Kafka's terms, whereas a `SubscribingEventProcessor` typically works on its own accord regardless of the number of instances. The workload sharing can be achieved by having several application instances with the same `groupId` or by adjusting the consumer count through the `SubscribableKafkaMessageSource`'s builder. The same benefit holds for [resetting](../axon-framework/events/event-processors/streaming.md#replaying-events) an event stream, which in Axon is reserved to the `TrackingEventProcessor`, but is now opened up through Kafka's own API's.

Although the `SubscribableKafkaMessageSource` thus provides the niceties the tracking event processor normally provides, it does come with two catches:

1. Axon's approach of the `SequencingPolicy` to deduce which thread receives which events is entirely lost.

   It is thus dependent on which topic-partition pairs are given to a `Consumer` for the events your handlers receives.

   From a usage perspective this means event message ordering is no longer guaranteed by Axon.

   It is thus the user's job to ensure events are published in the right topic-partition pair.

2. The API Axon provides for resets is entirely lost,

   since this API can only be correctly triggered through the `TrackingEventProcessor#resetTokens` operation

Due to the above it is recommended the user is knowledgeable about Kafka's specifics on message consumption.

When it comes to configuring a `SubscribableKafkaMessageSource` as a message source for a `SubscribingEventProcessor`, there is one additional requirement beside source creation and registration. The source should only start with polling for events as soon as all interested subscribing event processors have been subscribed to it. To ensure the `SubscribableKafkaMessageSource#start()` operation is called at the right point in the configuration lifecycle, the `KafkaMessageSourceConfigurer` should be utilized:

```java
public class KafkaEventConsumptionConfiguration {
    // ...
    public KafkaMessageSourceConfigurer kafkaMessageSourceConfigurer(Configurer configurer) {
        KafkaMessageSourceConfigurer kafkaMessageSourceConfigurer = new KafkaMessageSourceConfigurer();
        configurer.registerModule(kafkaMessageSourceConfigurer);
        return kafkaMessageSourceConfigurer;
    }

    public SubscribableKafkaMessageSource<String, byte[]> subscribableKafkaMessageSource(List<String> topics,
                                                                                         String groupId,
                                                                                         ConsumerFactory<String, byte[]> consumerFactory,
                                                                                         Fetcher<String, byte[], EventMessage<?>> fetcher,
                                                                                         KafkaMessageConverter<String, byte[]> messageConverter,
                                                                                         int consumerCount,
                                                                                         KafkaMessageSourceConfigurer kafkaMessageSourceConfigurer) {
        SubscribableKafkaMessageSource<String, byte[]> subscribableKafkaMessageSource = SubscribableKafkaMessageSource.<String, byte[]>builder()
                .topics(topics)                     // Defaults to a collection of "Axon.Events"
                .groupId(groupId)                   // Hard requirement
                .consumerFactory(consumerFactory)   // Hard requirement
                .fetcher(fetcher)                   // Hard requirement
                .messageConverter(messageConverter) // Defaults to a "DefaultKafkaMessageConverter"
                .consumerCount(consumerCount)       // Defaults to a single Consumer
                .build();
        // Registering the source is required to tie into the Configurers lifecycle to start the source at the right stage
        kafkaMessageSourceConfigurer.registerSubscribableSource(configuration -> subscribableKafkaMessageSource);
        return subscribableKafkaMessageSource;
    }

    public void configureSubscribableKafkaSource(EventProcessingConfigurer eventProcessingConfigurer,
                                                 String processorName,
                                                 SubscribableKafkaMessageSource<String, byte[]> subscribableKafkaMessageSource) {
        eventProcessingConfigurer.registerSubscribingEventProcessor(
                processorName,
                configuration -> subscribableKafkaMessageSource
        );
    }
    // ...
}
```

The `KafkaMessageSourceConfigurer` is an Axon `ModuleConfiguration` which ties in to the start and end lifecycle of the application. It should receive the `SubscribableKafkaMessageSource` as a source which should start and stop. The `KafkaMessageSourceConfigurer` instance itself should be registered as a module to the main `Configurer`.

If only a single subscribing event processor will be subscribed to the kafka message source, `SubscribableKafkaMessageSource.Builder#autoStart()` can be toggled on. This will start the `SubscribableKafkaMessageSource` upon the first subscription.

### Consuming Events with a Streamable Message Source

Using the `StreamableKafkaMessageSource` means you are inclined to use a `TrackingEventProcessor` to consume the events in your event handlers.

Whereas the [subscribable kafka message source](kafka.md#consuming-events-with-a-subscribable-message-source) uses Kafka's idea of sharing the workload through multiple `Consumer` instances in the same "Consumer Group", the streamable approach doesn't use a consumer group, and assigns all available partitions.

```java
public class KafkaEventConsumptionConfiguration {
    // ...
    public StreamableKafkaMessageSource<String, byte[]> streamableKafkaMessageSource(List<String> topics,
                                                                                     ConsumerFactory<String, byte[]> consumerFactory,
                                                                                     Fetcher<String, byte[], KafkaEventMessage> fetcher,
                                                                                     KafkaMessageConverter<String, byte[]> messageConverter,
                                                                                     int bufferCapacity) {
        return StreamableKafkaMessageSource.<String, byte[]>builder()
                .topics(topics)                                                 // Defaults to a collection of "Axon.Events"
                .consumerFactory(consumerFactory)                               // Hard requirement
                .fetcher(fetcher)                                               // Hard requirement
                .messageConverter(messageConverter)                             // Defaults to a "DefaultKafkaMessageConverter"
                .bufferFactory(
                        () -> new SortedKafkaMessageBuffer<>(bufferCapacity))   // Defaults to a "SortedKafkaMessageBuffer" with a buffer capacity of "1000"
                .build();
    }

    public void configureStreamableKafkaSource(EventProcessingConfigurer eventProcessingConfigurer,
                                               String processorName,
                                               StreamableKafkaMessageSource<String, byte[]> streamableKafkaMessageSource) {
        eventProcessingConfigurer.registerTrackingEventProcessor(
                processorName,
                configuration -> streamableKafkaMessageSource
        );
    }
    // ...
}
```

Note that as with any tracking event processor, the progress on the event stream is stored in a `TrackingToken`. Using the `StreamableKafkaMessageSource` means a `KafkaTrackingToken` containing topic-partition to offset pairs is stored in the `TokenStore`. If no other `TokenStore` is provided, and auto-configuration is used, a `KafkaTokenStore` will be set instead of an `InMemoryTokenStore`. The `KafkaTokenStore` by default uses the `__axon_token_store_updates` topic. This should be a compacted topic, which should be created and configured automatically.

## Customizing event message format

In the previous sections the `KafkaMessageConverter<K, V>` has been shown as a requirement for event production and consumption. The `K` is the format of the message's key, where the `V` stand for the message's value. The extension provides a `DefaultKafkaMessageConverter` which converts an axon `EventMessage` to a Kafka `ProducerRecord`, and an `ConsumerRecord` back into an `EventMessage`. This `DefaultKafkaMessageConverter` uses `String` as the key and `byte[]` as the value of the message to de-/serialize.

Albeit the default, this implementation allows for some customization, such as how the `EventMessage`'s `MetaData` is mapped to Kafka headers. This is achieved by adjusting the "header value mapper" in the `DefaultKafkaMessageConverter`'s builder.

The `SequencingPolicy` can be adjusted to change the behaviour of the record key being used. The default sequencing policy is the `SequentialPerAggregatePolicy`, which leads to the aggregate identifier of an event being the key of a `ProducerRecord` and `ConsumerRecord`.

The format of an event message defines an API between the producer and the consumer of the message. 
This API may change over time, leading to incompatibility between the event class' structure on the receiving side and the event structure of a message containing the old format. 
Axon addresses the topic of [Event Versioning](../axon-framework/events/event-versioning.md) by introducing Event Upcasters. 
The `DefaultKafkaMessageConverter` supports this by provisioning an `EventUpcasterChain` and run the upcasting process on the `MetaData` and `Payload` of individual messages converted from `ConsumerRecord` before those are passed to the `Serializer` and converted into `Event` instances.

Note that the `KafkaMessageConverter` feeds the upcasters with messages one-by-one, limiting it to one-to-one or one-to-many upcasting <b>only</b>. 
Upcasters performing a many-to-one or many-to-many operation thus won't be able to operate inside the extension (yet).

Lastly, the `Serializer` used by the converter can be adjusted. See the [Serializer](../axon-framework/serialization.md) section for more details on this.

```java
public class KafkaMessageConversationConfiguration {
    // ...
    public KafkaMessageConverter<String, byte[]> kafkaMessageConverter(Serializer serializer,
                                                                       SequencingPolicy<? super EventMessage<?>> sequencingPolicy,
                                                                       BiFunction<String, Object, RecordHeader> headerValueMapper,
                                                                       EventUpcasterChain upcasterChain) {
        return DefaultKafkaMessageConverter.builder()
                                           .serializer(serializer)                  // Hard requirement
                                           .sequencingPolicy(sequencingPolicy)      // Defaults to a "SequentialPerAggregatePolicy"
                                           .upcasterChain(upcasterChain)            // Defaults to empty upcaster chain
                                           .headerValueMapper(headerValueMapper)    // Defaults to "HeaderUtils#byteMapper()"
                                           .build();
    }
    // ...
}
```

Make sure to use an identical `KafkaMessageConverter` on both the producing and consuming end, as otherwise exception upon deserialization should be expected. A `CloudEventKafkaMessageConverter` is also available using the [Cloud Events](https://cloudevents.io/) spec.

## Configuration in Spring Boot

This extension can be added as a Spring Boot starter dependency to your project using group id `org.axonframework.extensions.kafka` and artifact id `axon-kafka-spring-boot-starter`. When using the auto configuration, the following components will be created for you automatically:

**Generic Components:**

* A `DefaultKafkaMessageConverter` using the configured `eventSerializer` \(which defaults to `XStreamSerializer`\), which is used by default to convert between Axon Event messages and Kafka records.

  Uses a `String` for the keys and a `byte[]` for the record's values.

  When the property `axon.kafka.message-converter-mode` is set to `cloud_event` a `CloudEventKafkaMessageConverter` will be used instead. This will use `String` for the keys and `CloudEvent`.

  For each the matching Kafka (de)serializers will also be set as default.

**Producer Components:**

* A `DefaultProducerFactory` using a `String` for the keys and a `byte[]` for the record's values.

  This creates a `ProducerFactory` in confirmation mode "NONE", as is specified [here](kafka.md#publishing-events-to-kafka).

  The `axon.kafka.publisher.confirmation-mode` should be adjusted to change this mode,

  where the "TRANSACTIONAL" mode requires `axon.kafka.producer.transaction-id-prefix` property to be provided.

  If the `axon.kafka.producer.transaction-id-prefix` is non-null and non-empty,

  it is assumed a "TRANSACTIONAL" confirmation mode is desired

* A `KafkaPublisher`.

  Uses a `Producer` instance from the `ProducerFactory` to publish events to the configured Kafka topic.

* A `KafkaEventPublisher`. Used to provide events to the `KafkaPublisher` and to assign a processor name

  and processing group called `__axon-kafka-event-publishing-group` to it. Defaults to a `SubscribingEventProcessor`.

  If a `TrackingEventProcessor` is desired, the `axon.kafka.producer.event-processor-mode` should be set to `tracking`

**Consumer Components:**

* A `DefaultConsumerFactory` using a `String` for the keys and a `byte[]` for the record's values
* An `AsyncFetcher`. To adjust the `Fetcher`'s poll timeout, the `axon.kafka.fetcher.poll-timeout` can be set.
* A `StreamableKafkaMessageSource` which can be used for `TrackingEventProcessor` instances

When using the Spring Boot auto configuration be mindful to provide an `application.properties` file. The Kafka extension configuration specifics should be placed under prefix `axon.kafka`. On this level, the `bootstrapServers` \(defaults to `localhost:9092`\) and `default-topic` used by the producing and consuming side can be defined.

The `DefaultProducerFactory` and `DefaultConsumerFactory` expects a `Map` of configuration properties, which correspond to Kafka `Producer` and `Consumer` specific properties respectively. As such, Axon itself passes along these properties without using them directly itself. The `application.properties` file provides a number of named properties under the `axon.kafka.producer.` and `axon.kafka.consumer.` prefixes. If the property you are looking for is not predefined in Axon `KafkaProperties` file, you are always able to introduce properties in a map style.

```yaml
# This is a sample properties file to configure the Kafka Extension
axon:
  kafka:
    bootstrap-servers: localhost:9092
    client-id: kafka-axon-example
    default-topic: local.event
    properties:
      security.protocol: PLAINTEXT

    publisher:
      confirmation-mode: transactional

    producer:
      transaction-id-prefix: kafka-sample
      retries: 0
      event-processor-mode: subscribing
      # For additional unnamed properties, add them to the `properties` map like so
      properties:
        some-key: [some-value]

    fetcher:
      poll-timeout: 3000

    consumer:
      enable-auto-commit: true
      auto-commit-interval: 3000
      event-processor-mode: tracking
      # For additional unnamed properties, add them to the `properties` map like so
      properties:
        some-key: [some-value]
```

> **Auto configuring a `SubscribableKafkaMessageSource`**
>
> The auto configured `StreamableKafkaMessageSource` can be toggled off by setting the `axon.kafka.consumer.event-processing-mode` to `subscribing`.
>
> Note that this **does not** create a `SubscribableKafkaMessageSource` for you out of the box. To set up a subscribable message, we recommend to read [this](kafka.md#consuming-events-with-a-subscribable-message-source) section.
