# Tuning Event Processing

Typically, application components contain one or more Event Processors which are responsible for processing incoming events. Tracking Event Processors have configuration aspects that can be changed at runtime to accommodate for changes in the system topology.

## Increasing and decreasing segment counts

Tracking Event Processors that handle events in multiple threads use segments to separate the events in the stream across these threads in a reliable way. However, especially when these threads are spread across multiple instances of a component, and the number of instances changes, it may be useful to scale the number of segments accordingly.

To this end, Axon Framework provides a [split and merge API](../../configuring-infrastructure-components/event-processing/event-processors.md#splitting-and-merging-tracking-tokens). This API can be utilized directly or through Axon Server, where the latter takes required coordination into account.

### Segment tuning through Axon Server

Axon Server provides an API to trigger the increase and decrease of the number of Segments, at runtime, across different instances of a processor. Follow these steps to increase or decrease the number of segments for a processor:

* In the UI, navigate to the component overview, and select one of the application instances that contains the component to change the number of segments for.
* In the Component Details screen, scroll down to the list of processors.
* In the list of processors, click the "Scale" icon to open the Scaling dialogue
* In the Scaling dialogue, either click "Increase" or "Decrease" to adjust the number of segments for the processor.

> **Using the REST API**
>
> The exposed buttons in the UI can also be targeted directly through REST calls through curl for example. It is recommended to check the Swagger UI for the split and merge endpoint information if such an approach is necessary.

### Segment tuning through Axon Framework

The Tracking Event Processors in Axon Framework provide methods to increase or decrease the number of segments for that particular instance. When using this API, one must provide the ID of the segment to increase/decrease. Additionally, the instance on which the method is invoked, must be actively processing that segment.

First, the instance of the Tracking Event Processor must be obtained. This can be done using Axon's Configuration API like so:

```java
// The `Configuration` was returned through the `Configurer` or is available as a bean in the Spring Application Context
public TrackingEventProcessor retrieveTrackingProcessor(org.axonframework.config.Configuration axonConfig,
                                                        String processorName) {
    return axonConfig.eventProcessingConfiguration()
                     .eventProcessor(processorName) // This call returns an Optional
                     .filter(eventProcessor -> eventProcessor instanceof TrackingEventProcessor)
                     .map(eventProcessor -> (TrackingEventProcessor) eventProcessor)
                     .orElseThrow(() -> new IllegalStateException(
                             "No Tracking Event Processor found with name " + processorName
                     ));
}
```

Using the above snippet, a split or merge can be called as follows:

```java
int segmentId;
TrackingEventProcessor trackingProcessor = retrieveTrackingProcessor(axonConfig, processorName);

// Split...
CompletableFuture<Boolean> futureResult = trackingProcessor.splitSegment(segmentId);

// Merge...
CompletableFuture<Boolean> futureResult = trackingProcessor.mergeSegment(segmentId);
```

> **Multi instance set up**
>
> If you have several instances of a given Axon application, that regularly means you have duplicated your Tracking Event Processors. Such a set up is a regular scenario to require segment tuning.
>
> Note that, especially, in such a setup you would need to delegate said split or merge to the correct instance. The "correct instance", in this case, is the instance owning the segment you want to split and merge.

## Blacklisting Events

In a heterogeneously distributed application landscape your event handling components might receive events they do not have actual event handling members for. That this occurs is completely fine; the chances of a single application handling the entirety of all existing events is very small. This fact however does open up the possibility for optimization by _blacklisting_ events.

To this end Axon has to option to automatically blacklist events it cannot handle. The Tracking Event Processor takes the lead in actual blacklisting, which it does by signaling the utilized event stream when none of its handlers can handle the event in question. The event stream provided by the Axon Server connection in turn implements the functionality to notify an Axon Server node that certain events cannot be handled by it.

By default, blacklisting is turned for an Axon client connected to Axon Server. To disable blacklisting, the `disableEventBlacklisting` property can be adjusted as follows:

{% tabs %}
{% tab title="Axon Configuration API" %}
```java
AxonServerConfiguration axonServerConfig = new AxonServerConfiguration();
axonServerConfig.setDisableEventBlacklisting(true);

Configurer configurer = 
    DefaultConfigurer.defaultConfiguration()
        .registerComponent(AxonServerConfiguration.class, c -> axonServerConfig);
```
{% endtab %}

{% tab title="Spring Boot AutoConfiguration" %}
```text
axon.axonserver.disableEventBlacklisting=true
```
{% endtab %}
{% endtabs %}

> **Retrying Blacklisted Events**
>
> The topology of Event Handlers might change in the lifecycle of a given application. This thus means that once blacklisted events might at a later stage do have Event Handler members present. To cover this scenario, Axon Server will periodically send over blacklisted events to refresh the blacklisted set.

