# Spring Cloud

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

Spring Cloud is an alternative approach to distributing the command bus \(commands\), besides Axon Server as the default.

The Spring Cloud Extension uses the service registration and discovery mechanism described by [Spring Cloud](https://spring.io/projects/spring-cloud) for distributing the command bus. 
You thus have the choice of which Spring Cloud implementation to use when discovering the routes to distribute your commands. 
An example of that would be Netflix' [Eureka Discovery/Eureka Server](https://cloud.spring.io/spring-cloud-netflix/multi/multi__service_discovery_eureka_clients.html) combination or HashiCorp's [Consul](https://www.consul.io/use-cases/service-discovery-and-health-checking).

To use the Spring Cloud components from Axon, make sure the `axon-springcloud` module is available on the classpath.
The easiest way is to include the Spring Cloud starter (`axon-springcloud-spring-boot-starter`) from this extension to your project.

Giving a description of every Spring Cloud implementation would push this reference guide too far. 
For information on other Spring Cloud implementation options out there, please refer to their respective documentations.

The Spring Cloud connector setup is a combination of the `SpringCloudCommandRouter` and a `SpringHttpCommandBusConnector`.
The former is the `CommandRouter` and latter the `CommandBusConnector`, both used by the `DistributedCommandBus` to enable command distribution.

## Discovering Command Routes

The `SpringCloudCommandRouter` uses Spring Cloud's discovery mechanism to find the other nodes in the cluster.
To that end it uses the `DiscoveryClient` and `Registration` from Spring Cloud.
These are respectively used to gather remote command routing information and maintain local information.
The most straightforward way to retrieve both is by annotating your application with `@EnableDiscoveryClient`.

Gathering and storing the command routing information revolves around Spring Cloud's `ServiceInstance`s.
A `Registration` is just the local `ServiceInstance`, whereas the `DiscoveryClient` provides the API to find remote `ServiceInstance`s.
Furthermore, it is the `ServiceInstance` which provides us with the required information (e.g. the URI) to retrieve a node's capabilities.

> **Spring Cloud's Heartbeat Requirement**
>
> When using the `SpringCloudCommandRouter`, make sure your Spring application has heartbeat events enabled.
> The heartbeat events published by a Spring Cloud application are the trigger to check if the set of `ServiceInstance`s from the `DiscoveryClient` has been changed.
> Additionally, it is used to validate whether the command routing capabilities for known nodes has been altered.
>
> Thus, if heartbeat events are disabled, your instance will no longer be updated with the current command routing capabilities.
> If so, this will cause issues during command routing.

The logic to store the local capabilities and discovering the remote capabilities of a `ServiceInstance` is maintained in the `CapabilityDiscoveryMode`. 
It is thus the `CapabilityDiscoveryMode` which provides us the means to actually retrieve a `ServiceInstance`'s set of commands it can handle (if any).
The sole full implementation provided of the `CapabilityDiscoveryMode`, is the `RestCapabilityDiscoveryMode`, using a `RestTemplate` and the `ServiceInstance` URI to invoke a configurable endpoint.
This endpoint leads to the `MemberCapabilitiesController` which in turn exposes the `MemberCapabilities` on the `RestCapabilityDiscoveryMode` of that instance.

There are decorators present for the `CapabilityDiscoveryMode`, providing two additional features:

1. `IgnoreListingDiscoveryMode` - a `CapabilityDiscoveryMode` decorator which on failure of retrieving the `MemberCapabilities` will place the given `ServiceInstance` on a list to be ignored for future validation. It thus effectively removes discoverable `ServiceInstance`s from the set.
2. `AcceptAllCommandsDiscoveryMode` - a `CapabilityDiscoveryMode` decorator which regardless of what _this_ instance can handle as commands, state it can handle anything. This decorator comes in handy if the nodes in the system are homogeneous (aka, everybody can handle the same set of commands). 

The `Registration`, `DiscoveryClient` and `CapabilityDiscoveryMode` are arguably the heart of the `SpringCloudCommandRouter`.
There are, however, a couple of additional things you can configure for this router, which are the following:

* `RoutingStrategy` - The component in charge of deciding which of the nodes receives the commands consistently. By default, a `AnnotationRoutingStrategy` is used (see [Distributing the Command Bus](../axon-framework/axon-framework-commands/infrastructure.md#distributedcommandbus) for more).
* A `ServiceInstance` filter - This `Predicate` is used to filter out `ServiceInstance`s retrieved through the `DiscoveryClient`. For example, it allows the removal of instances which are known to not handle any command messages. This might be useful if you have several services within the Spring Cloud Discovery Service set up, which you do not ever want to take into account for command handling.
* `ConsistentHashChangeListener` - Adding a consistent hash change listener provides you with the opportunity to perform a specific task if new nodes have been added to the known command handlers set.

> **Differing Command Capabilities per Node**
>
> It is not required for all nodes to have the same set of command handlers. 
> You may use different segments for different command types altogether. 
> The Distributed Command Bus will always choose a node to dispatch a command to the one that has support for that specific type of command.

## Sending Commands between nodes

The `CommandBusConnector` is in charge of sending commands, based on a given route, from one node to another.
This extension to that end provides the `SpringHttpCommandBusConnector`, which uses plain REST for sending commands.

There are three hard requirements when creating this service and one optional configuration:

1. Local `CommandBus` - This "local segment" is the command bus which dispatches commands into the local JVM. 
   It is thus invoked when the `SpringHttpCommandBusConnector` receives a command from the outside, or if it receives a command which is meant for itself.
2. `RestOperations` - The service used to POST a command message to another instance. 
   In most situations the `RestTemplate` is used for this.
3. `Serializer` - The serializer is used to serialize the command messages before they are sent over and deserialize when they are received.
4. `Executor` (optional) - The `Executor` is used to handle incoming commands and to dispatch commands. 
   Defaults to a `DirectExecutor` instance for backwards compatibility.

> **Non-blocking command dispatching**
> 
> Note that the configurable `Executor` impacts how command dispatching acts when invoking `CommandGateway#send` methods returning a `CompletableFuture`.
> Although the `CompletableFuture` return type suggests a non-blocking result, if the bus under the hood reuses the dispatching thread we are still faced with a blocking operation.
> Hence, to make the `SpringHttpCommandBusConnector` fully non-blocking, it is recommended to adjust the `Executor` to use its own thread pool.

## Configuring this Extension

Chances are high that you will be using Spring Boot if you are also using Spring Cloud.
As configuring goes, this would opt for usage of the `axon-springcloud-spring-boot-starter` dependency to automatically retrieve all required beans.
In either case, your application should be marked to enable it as a discoverable service through Spring Cloud.
This can, for example, be done by annotating the main class with `@EnableDiscoveryClient`. 

There are still quite a few customizable components.
For some suggestions, take a look at the following examples:

{% tabs %}
{% tab title="Custom Bean Configuration" %}
```java
// Custom Spring Boot app, enabling a 'DiscoveryClient' and 'Registration' through `@EnableDiscoveryClient`
@EnableDiscoveryClient
@SpringBootApplication
public class MyApplication {

    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }

    @Bean
    public CapabilityDiscoveryMode capabilityDiscoveryMode(RestTemplate restTemplate, Serializer serializer) {
        return RestCapabilityDiscoveryMode.builder()
                                          .restTemplate(restTemplate)
                                          .serializer(serializer)
                                          // Allows changing the endpoint used to find member capabilities
                                          .messageCapabilitiesEndpoint(/* custom message information endpoint */)
                                          .build();
    }

    @Bean
    public CommandRouter springCloudCommandRouter(DiscoveryClient discoveryClient,
                                                  Registration localServiceInstance,
                                                  CapabilityDiscoveryMode capabilityDiscoveryMode) {
        return SpringCloudCommandRouter.builder()
                                       .discoveryClient(discoveryClient)
                                       .routingStrategy(new AnnotationRoutingStrategy())
                                       .localServiceInstance(localServiceInstance)
                                       .capabilityDiscoveryMode(capabilityDiscoveryMode)
                                       .serviceInstanceFilter(/* custom ServiceInstance filter */)
                                       .consistentHashChangeListener(/* ConsistentHash change listener */)
                                       .build();
    }

    // Only required if Axon Spring Boot Starter is not used
    @Bean
    @Qualifier("localSegment")
    public CommandBus localSegment() {
        return SimpleCommandBus.builder().build();
    }

    @Bean
    public CommandBusConnector springHttpCommandBusConnector(@Qualifier("localSegment") CommandBus localSegment,
                                                             RestOperations restOperations,
                                                             Serializer serializer) {
        return SpringHttpCommandBusConnector.builder()
                                            .localCommandBus(localSegment)
                                            .restOperations(restOperations)
                                            .serializer(serializer)
                                            .executor(/* custom Executor */)
                                            .build();
    }

    @Bean
    @Primary
    public DistributedCommandBus distributedCommandBus(CommandRouter commandRouter,
                                                       CommandBusConnector commandBusConnector) {
        return DistributedCommandBus.builder()
                                    .commandRouter(commandRouter)
                                    .connector(commandBusConnector)
                                    .build();
    }
}
```
{% endtab %}
{% tab title="Spring Boot AutoConfiguration" %}
```properties
# Required to enabled the DistributedCommandBus
axon.distributed.enabled=true
# Defines the load factor used for this segment. Defaults to 100
axon.distributed.load-factor=100
# Defines the CapabilityDiscoveryMode used. Defaults to REST
axon.distributed.spring-cloud.mode=rest
# Defines the endpoint used to retrieve member capabilities from. Defaults to "/member-capabilities"
axon.distributed.spring-cloud.rest-mode-url="/my-custom-endpoint"
# Defines whether the CapabilityDiscoveryMode should be decorated to ignore faulty ServiceInstances 
axon.distributed.spring-cloud.enable-ignore-listing=true
# Defines whether the CapabilityDiscoveryMode should be decorated to accept all types of commands
axon.distributed.spring-cloud.enable-accept-all-commands=true
```
{% endtab %}
{% endtabs %}
