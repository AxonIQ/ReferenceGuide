# Reactor

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

Overlooking Axon Frameworks architecture, you can notice that in general systems using the framework are "Message Driven", "Responsive", "Resilient" and "Elastic".
According to [Reactive Manifesto](https://www.reactivemanifesto.org/), the same holds for Reactive Systems in general. 

Although we can state that Axon Framework is a _type_ of reactive system, we can't say that it is fully reactive.

{% hint style="info" %}
Reactive programming is an approach to writing software that embraces asynchronous I/O. 
Asynchronous I/O is a small idea that portends big changes for software. 
The idea is simple: alleviate inefficient resource utilization by reclaiming resources that would otherwise be idle, as they waited for I/O activity. 
Asynchronous I/O inverts the normal design I/O processing: the clients are notified of new data instead of asking for it, which frees the client to do other work while waiting for these notifications. 
{% endhint %}

By their nature, a reactive API and Axon are a great fit, as most of framework's operations are async and non-blocking.
Providing a dedicated extension for this was thus a logical step to take.
To that end, we chose to use Pivotal’s [Project Reactor](https://projectreactor.io/) to build this extension.
Reactor builds on top of the [Reactive Streams](https://www.reactive-streams.org/) specification and is the de-facto standard for Java enterprise and Spring applications.
As such, we feel it to be a great fit to provide an extension in, making Axon more reactive.

{% hint style="info" %}
Not all Axon components offer a reactive API, yet. 
We will incrementally introduce more "reactiveness" to this extension, giving priority to components where users can benefit the most. 
{% endhint %}

To use the [Axon Reactor Extension](https://github.com/AxonFramework/extension-reactor), make sure that `axon-reactor` module is available on the classpath.
