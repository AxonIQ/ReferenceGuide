# Monitoring and Metrics

The ability to monitor and measure what is going on is very important. Especially in a location transparent environment like an Axon application it is very important to be able to trace your message and check the ingestion rate of it.

## Monitoring

Monitoring a message centric application will require you to be able to see where your messages are at a given point in time. This translates to being able to track your commands, events and queries from one component to another in an Axon application.

### Correlation Data

One import aspect in regards to this is tracing a given message. To that end the framework provides the `CorrelationDataProvider`, as described briefly [here](../../configuring-infrastructure-components/messaging-concepts/message-intercepting.md). This interface and its implementations provide you the means to populate the meta-data of your messages with specific fields, like a 'trace-id', 'correlation-id' or any other field you might be interested in.

For configuring the `MessageOriginProvider` you can do the following:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class MonitoringConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    public void configureMessageOriginProvider(Configurer configurer) {
        configurer.configureCorrelationDataProviders(configuration -> {
            List<CorrelationDataProvider> correlationDataProviders = new ArrayList<>();
            correlationDataProviders.add(new MessageOriginProvider());
            return correlationDataProviders;
        });
    }

}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```java
public class MonitoringConfiguration {

    // When using Spring Boot, simply defining a CorrelationDataProvider bean is sufficient
    public CorrelationDataProvider messageOriginProvider() {
        return new MessageOriginProvider();
    }

}
```
{% endtab %}
{% endtabs %}

### Interceptor Logging

Another good approach to track the flow of messages throughout an Axon application is by setting up the right interceptors in your application.  
There are two flavors of interceptors, the Dispatch and Handler Interceptors \(as discussed [here](../../configuring-infrastructure-components/messaging-concepts/message-intercepting.md)\), which intercept a message prior to publishing \(Dispatch Interceptor\) or while it is being handled \(Handler Interceptor\). The interceptor mechanism lends itself quite nicely to introduce a way to consistently log when a message is being dispatched/handled. The `LoggingInterceptor` is an out of the box solution to log any type of message to SLF4J, but also provides a simple overridable template to set up your own desired logging format. We refer to the command, event and query sections for the specifics on how to configure message interceptors.

### Event Tracker Status

Since [Tracking Tokens](../../configuring-infrastructure-components/event-processing/event-processors.md#token-store) "track" the progress of a given Tracking Event Processor, they provide a sensible monitoring hook in any Axon application. Such a hook proves its usefulness when we want to rebuild our view model and we want to check when the processor has caught up with all the events.

To that end the `TrackingEventProcessor` exposes the `processingStatus()` method. It returns a map where the key is the segment identifier and the value is an "Event Tracker Status". The Event Tracker Status exposes a couple of metrics:

* The `Segment` it reflects the status of.
* A boolean through `isCaughtUp()` specifying whether it is caught up with the Event Stream.
* A boolean through `isReplaying()` specifying whether the given Segment is

  [replaying](../../configuring-infrastructure-components/event-processing/event-processors.md#replaying-events).

* A boolean through `isMerging()` specifying whether the given Segment is

  [merging](../../configuring-infrastructure-components/event-processing/event-processors.md#splitting-and-merging-tracking-tokens).

* The `TrackingToken` of the given Segment.
* A boolean through `isErrorState()` specifying whether the Segment is in an error state.
* An optional `Throwable` if the Event Tracker reached an error state.
* An optional `Long` through `getCurrentPosition` defining the current position of the `TrackingToken`.
* An optional `Long` through `getResetPosition` defining the position at reset of the `TrackingToken`. 

  This field will be `null` in case the `isReplaying()` returns `false`.

  It is possible to derive an estimated duration of replaying by comparing the current position with this field.

* An optional `Long` through `mergeCompletedPosition()` defining the position on the `TrackingToken` when merging will be completed. 

  This field will be `null` in case the `isMerging()` returns `false`.

  It is possible to derive an estimated duration of merging by comparing the current position with this field.

## Metrics

Interesting metrics in a message centric system come in several forms and flavors, like count, capacity and latency for example. Axon Framework allows you to retrieve such measurements through the use of the `axon-metrics` or `axon-micrometer` module. With these modules you can register a number of `MessageMonitor` implementations to your messaging components, like the [`CommandBus`](../../configuring-infrastructure-components/command-processing/command-dispatching.md#the-command-bus), [`EventBus`](../../configuring-infrastructure-components/event-processing/event-bus-and-event-store.md#event-bus), [`QueryBus`](../../configuring-infrastructure-components/query-processing/query-dispatching.md#query-bus) and [`EventProcessors`](../../configuring-infrastructure-components/event-processing/event-processors.md#event-processors).

`axon-metrics` module uses [Dropwizard Metrics](https://metrics.dropwizard.io/) for registering the measurements correctly. That means that `MessageMonitors` are registered against the Dropwizard `MetricRegistry`.

`axon-micrometer` module uses [Micrometer](https://micrometer.io/) which is a dimensional-first metrics collection facade whose aim is to allow you to time, count, and gauge your code with a vendor neutral API. That means that `MessageMonitors` are registered against the Micrometer `MeterRegistry`.

The following monitor implementations are currently provided:

1. `CapacityMonitor` - Measures message capacity by keeping track of the total time spent on message handling compared to total time it is active. 

   This returns a number between 0 and n number of threads. Thus, if there are 4 threads working, the maximum capacity is 4 if every thread is active 100% of the time. 

2. `EventProcessorLatencyMonitor` - Measures the difference in message timestamps between the last ingested and the last processed event message.
3. `MessageCountingMonitor` - Counts the number of ingested, successful, failed, ignored and processed messages.
4. `MessageTimerMonitor` - Keeps a timer for all successful, failed and ignored messages, as well as an overall timer for all three combined.
5. `PayloadTypeMessageMonitorWrapper` - A special `MessageMonitor` implementation which allows setting a monitor per message type instead of per message publishing/handling component. 

You are free to configure any combination of `MessageMonitors` through constructors on your messaging components, simply by using the Configuration API. The `GlobalMetricRegistry` contained in the `axon-metrics` and `axon-micrometer` modules provides a set of sensible defaults per type of messaging component. The following example shows you how to configure default metrics for your message handling components:

### Dropwizard

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    // The MetricRegistry is a class from the Dropwizard Metrics framework
    public void configureDefaultMetrics(Configurer configurer, MetricRegistry metricRegistry) {
        GlobalMetricRegistry globalMetricRegistry = new GlobalMetricRegistry(metricRegistry);
        // We register the default monitors to our messaging components by doing the following
        globalMetricRegistry.registerWithConfigurer(configurer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
# The default value is `true`. Thus you will have Metrics configured if `axon-metrics` and `io.dropwizard.metrics` are on your classpath.
axon.metrics.auto-configuration.enabled=true
```
{% endtab %}
{% endtabs %}

### Micrometer

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
public class MetricsConfiguration {

    public Configurer buildConfigurer() {
        return DefaultConfigurer.defaultConfiguration();
    }

    // The MeterRegistry is a class from the Micrometer library
    public void configureDefaultMetrics(Configurer configurer, MeterRegistry meterRegistry) {
        GlobalMetricRegistry globalMetricRegistry = new GlobalMetricRegistry(meterRegistry);
        // We register the default monitors to our messaging components by doing the following
        globalMetricRegistry.registerWithConfigurer(configurer);
    }
}
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
# The default value is `true`. Thus you will have Metrics configured if `axon-micrometer` and appropriate metric implementation (for example: `micrometer-registry-prometheus`) are on your classpath.
axon.metrics.auto-configuration.enabled=true
# Spring Boot metrics enabled
management.endpoint.metrics.enabled=true
# Spring Boot (Prometheus) endpoint (`/actuator/prometheus`) enabled and exposed
management.metrics.export.prometheus.enabled=true
management.endpoint.prometheus.enabled=true
```
{% endtab %}
{% endtabs %}

If you want to have more specific metrics on a message handling component like the `CommandBus`, `EventBus` \(or more specifically `TrackingEventProcessor`\), you can do the following:

```java
// Java (Spring Boot Configuration) - Micrometer example
@Configuration
public class MetricConfig {

    @Bean
    public ConfigurerModule metricConfigurer(MeterRegistry meterRegistry){
        return configurer -> {
            instrumentEventProcessors(meterRegistry, configurer);
            instrumentCommandBus(meterRegistry, configurer);
        };
    }

    private void instrumentEventProcessors(MeterRegistry meterRegistry, Configurer configurer) {
        MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {
            // We want to count the messages per type of event being published.
            PayloadTypeMessageMonitorWrapper<MessageCountingMonitor> messageCounterPerType =
                    new PayloadTypeMessageMonitorWrapper<>(monitorName -> MessageCountingMonitor.buildMonitor(monitorName, meterRegistry),
                                                           clazz ->  componentName + "_" + clazz.getSimpleName());
            // And we also want to set a message timer per payload type
            PayloadTypeMessageMonitorWrapper<MessageTimerMonitor> messageTimerPerType =
                    new PayloadTypeMessageMonitorWrapper<>(monitorName -> MessageTimerMonitor.buildMonitor(monitorName, meterRegistry),
                                                           clazz ->  componentName + "_" + clazz.getSimpleName());
            //Which we group in a MultiMessageMonitor
            return new MultiMessageMonitor<>(messageCounterPerType, messageTimerPerType);
        };
        configurer.configureMessageMonitor(TrackingEventProcessor.class, messageMonitorFactory);
    }

    private void instrumentCommandBus(MeterRegistry meterRegistry, Configurer configurer) {
            MessageMonitorFactory messageMonitorFactory = (configuration, componentType, componentName) -> {
                PayloadTypeMessageMonitorWrapper<MessageCountingMonitor> messageCounterPerType =
                        new PayloadTypeMessageMonitorWrapper<>(monitorName -> MessageCountingMonitor.buildMonitor(monitorName, meterRegistry),
                                                               clazz ->  componentName + "_" + clazz.getSimpleName());

                PayloadTypeMessageMonitorWrapper<MessageTimerMonitor> messageTimerPerType =
                        new PayloadTypeMessageMonitorWrapper<>(monitorName -> MessageTimerMonitor.buildMonitor(monitorName, meterRegistry),
                                                               clazz -> componentName + "_" + clazz.getSimpleName());

                PayloadTypeMessageMonitorWrapper<CapacityMonitor> capacityMonitor =
                        new PayloadTypeMessageMonitorWrapper<>(monitorName -> CapacityMonitor.buildMonitor(monitorName, meterRegistry),
                                                               clazz -> componentName + "_" + clazz.getSimpleName());

                return new MultiMessageMonitor<>(messageCounterPerType, messageTimerPerType, capacityMonitor);
            };
            configurer.configureMessageMonitor(CommandBus.class, messageMonitorFactory);
        }

}
```

