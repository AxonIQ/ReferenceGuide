# Quick Start

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

Axon provides a _**Quick Start Toolkit**_ to familiarize yourself with the setup required for Axon Framework and Axon Server SE \(Standard Edition\).

The only pre-requisite to run the Quick Start is to have a Java 8+ JRE in your system.

## Quick Start Toolkit Download

The Quick Start Toolkit package is available for download at the following location -&gt; [https://axoniq.io/download](https://axoniq.io/download).

This package contains,

* Axon Framework Binaries
* Axon Server Standard Edition
* Gift card sample application - Demo application designed to show various aspects of the platform
* Getting started guide

## Running the Quick Start \(Java 9+\)

1. Unzip `AxonQuickStart-VERSION.zip`
2. Run the [Axon Server](../axon-server-introduction.md): `$ java -jar AxonServer/axonserver-VERSION.jar`
3. Axon Server web dashboard should be available here [`http://localhost:8024/`](http://localhost:8024/)
4. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`
5. Demo application should be available here [`http://localhost:8080/`](http://localhost:8080/)
6. Explore the `README.md`

## Running the Quick Start on Java 8

The Quick Start Demo app can also run on Java 8 with a small change in the `pom.xml` file:

1. From the extracted folder, open `giftcard-demo\pom.xml` with your favourite editor
2. Replace `<java.version>11</java.version>` with `<java.version>1.8</java.version>`
3. Find the `maven-compiler-plugin` plugin block
4. Replace `<release>${java.version}</release>` with `<source>${java.version}</source> <target>${java.version}</target>`
5. Run the demo application: `$ cd giftcard-demo && ./mvnw spring-boot:run`

## Resources

### Axon Framework - Video Tutorials

The following 5-part video tutorial offers a quick start guided path to understand Axon Framework.

| Tutorial Name                                           | Purpose                                    |
|:--------------------------------------------------------|:-------------------------------------------|
| [Part - 1](https://www.youtube.com/watch?v=tqn9p8Duy54) | Structure of an Axon Framework Application |
| [Part - 2](https://www.youtube.com/watch?v=vnCxjWZrrk0) | Core API development                       |
| [Part - 3](https://www.youtube.com/watch?v=7oy4w5THFEU) | Command Model of the Application           |
| [Part - 4](https://www.youtube.com/watch?v=jS1vfc5EohM) | Query Model of the Application             |
| [Part - 5](https://www.youtube.com/watch?v=lxonQnu1txQ) | Connecting the UI                          |

### Axon Server

We recommend following the [Axon Server training](https://www.axoniq.io/training/axon-server-training) which is an easy way to get familiar with all aspects of it.
The training is free and is given periodically.

If you are looking for a fast way to get Axon Server up and running and get familiar with it through a hands-on approach, 
[this blog by Bert Laverman](https://axoniq.io/blog-overview/running-axon-server) is a great way to start.

### Intro to Event-Driven Microservices

AxonIQ provides an [Intro to Event-Driven Microservices using DDD, CQRS & Event Sourcing](https://www.axoniq.io/training/intro-to-event-driven-microservices). 
It's given periodically and is free to attend. During the introduction, you will be brought up to speed on topics such as DDD, CQRS, Event Sourcing and other concepts related to the Axon stack.

### Implementing Event-Driven Systems

AxonIQ provides a training which will get you fully familiar with everything around Axon Framework and Axon Server:
[Implementing Event-Driven Systems using DDD, CQRS and Event Sourcing](https://www.axoniq.io/training/axon-training).
It provides a more hands-on approach complete with instructor-led content and labs. 
This program is intended to deepen your knowledge concerning the concepts of Event-Driven systems, DDD, CQRS and how Axon supports this journey.

