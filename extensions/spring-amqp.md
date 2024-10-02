# Spring AMQP

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

Spring AMQP is an alternative approach to distributing events, besides Axon Server which is the default.

Axon provides out-of-the-box support to transfer events to and from an AMQP message broker, such as [RabbitMQ](https://www.rabbitmq.com/).

To use the Spring AMQP components from Axon, make sure the `axon-amqp` module is available on the classpath.

## Forwarding events to an AMQP Exchange

The `SpringAMQPPublisher` forwards events to an AMQP exchange. It is initialized with a `SubscribableMessageSource`, which is generally the `EventBus` or `EventStore`. Theoretically, this could be any source of events that the publisher can subscribe to.

To forward events generated in the application to an AMQP Channel, a single line of `application.properties` configuration is sufficient:

```text
axon.amqp.exchange=ExchangeName
```

This will automatically send all published events to the AMQP Exchange with the given name.

By default, no AMQP transactions are used when sending. This can be overridden using the `axon.amqp.transaction-mode` property, and setting it to `transactional` or `publisher-ack`.

> **Note**
>
> Note that exchanges are not automatically created. You must still declare the Queues, Exchanges and Bindings you wish to use. Check the Spring documentation for more information.

## Reading events from an AMQP Queue

Spring has extensive support for reading messages from an AMQP Queue. However, this needs to be 'bridged' to Axon, so that these messages can be handled from Axon as if they are regular event messages.

The `SpringAMQPMessageSource` allows event processors to read messages from a queue instead of the event store or event bus. It acts as an adapter between Spring AMQP and the `SubscribableMessageSource` needed by these processors.

To receive events from a queue and process them inside an Axon application, you need to configure a `SpringAMQPMessageSource`:

```java
@Bean
public SpringAMQPMessageSource myQueueMessageSource(AMQPMessageConverter messageConverter) {
    return new SpringAMQPMessageSource(messageConverter) {

        @RabbitListener(queues = "myQueue")
        @Override
        public void onMessage(Message message, Channel channel) throws Exception {
            super.onMessage(message, channel);
        }
    };
}
```

and then configure a processor to use this bean as the source for its messages:

```text
axon.eventhandling.processors.name.source=myQueueMessageSource
```

Note that tracking processors are not compatible with the `SpringAMQPMessageSource`.

