# Testing

One of the biggest benefits of CQRS, and especially that of event sourcing is that it is possible to express tests purely in terms of events and commands. Both being functional components, events and commands have clear meaning to the domain expert or business owner. Not only does this mean that tests expressed in terms of events and commands have a clear functional meaning, it also means that they hardly depend on any implementation choices.

The features described in this chapter require the `axon-test` module, which can be obtained by configuring a maven dependency \(use `<artifactId>axon-test</artifactId>` and `<scope>test</scope>`\) or from the full package download.

The fixtures described in this chapter work with any testing framework, such as JUnit and TestNG.

### Command component testing

The command handling component is typically the component in any CQRS based architecture that contains the most complexity. Being more complex than the others, this also means that there are extra test related requirements for this component.

Although being more complex, the API of a command handling component is fairly easy. It has a command coming in, and events going out. In some cases, there might be a query as part of command execution. Other than that, commands and events are the only part of the API. This means that it is possible to completely define a test scenario in terms of events and commands. Typically, in the shape of:

* Given certain events in the past
* When executing this command
* Expect these events to be published and/or stored

Axon Framework provides a test fixture that allows you to do exactly that. The `AggregateTestFixture` allows you to configure a certain infrastructure, composed of the necessary command handler and repository, and express your scenario in terms of "given-when-then" events and commands.

> **Note**
>
> Since the unit of testing here is the aggregate, `AggregateTestFixture` is meant to test one aggregate only. So, all commands in the `when` \(or `given`\) clause are meant to target the aggregate under test fixture. Also, all `given` and `expected` events are meant to be triggered from the aggregate under test fixture.

The following example shows the usage of the "given-when-then" test fixture with JUnit 4:

```java
public class MyCommandComponentTest {
    private FixtureConfiguration<MyAggregate> fixture;

    @Before
    public void setUp() {
        fixture = new AggregateTestFixture<>(MyAggregate.class);
    }

    @Test
    public void testFirstFixture() {
        fixture.given(new MyEvent(1))
               .when(new TestCommand())
               .expectSuccessfulHandlerExecution()
               .expectEvents(new MyEvent(2));
        /*
        These four lines define the actual scenario and its expected
        result. The first line defines the events that happened in the
        past. These events define the state of the aggregate under test.
        In practical terms, these are the events that the event store
        returns when an aggregate is loaded. The second line defines the
        command that we wish to execute against our system. Finally, we
        have two more methods that define expected behavior. In the
        example, we use the recommended void return type. The last method
        defines that we expect a single event as result of the command
        execution.
        */
    }
}
```

The "given-when-then" test fixture defines three stages: configuration, execution and validation. Each of these stages is represented by a different interface: `FixtureConfiguration`, `TestExecutor` and `ResultValidator`, respectively. The static `newGivenWhenThenFixture()` method on the `Fixtures` class provides a reference to the first of these, which in turn may provide the validator, and so forth.

> **Note**
>
> To make optimal use of the migration between these stages, it is best to use the fluent interface provided by these methods, as shown in the example above.

During the configuration phase \(i.e. before the first "given" is provided\), you provide the building blocks required to execute the test. Specialized versions of the event bus, command bus and event store are provided as part of the fixture. There are accessor methods in place to obtain references to them. Any command handlers not registered directly on the aggregate need to be explicitly configured using the `registerAnnotatedCommandHandler` method. Besides the `AnnotatedCommandHandler` you can configure a wide variety of components and settings that define how the infrastructure around the test should be set up.

Once the fixture is configured, you can define the "given" events. The test fixture will wrap these events as `DomainEventMessage`. If the "given" event implements `Message`, the payload and meta data of that message will be included in the `DomainEventMessage`, otherwise the given event is used as payload. The sequence numbers of the `DomainEventMessage` are sequential, starting at `0`.

Alternatively, you may also provide commands as "given" scenario. In that case, the events generated by those commands will be used to event source the aggregate when executing the actual command under test. Use the "`givenCommands(...)`" method to provide command objects.

The execution phase allows you to provide a Command to be executed against the command handling component. The behavior of the invoked handler \(either on the aggregate or as an external handler\) is monitored and compared to the expectations registered in the validation phase.

> **Note**
>
> During the execution of the test, Axon attempts to detect any illegal state changes in the aggregate under test. It does so by comparing the state of the aggregate after the command execution to the state of the aggregate if it sourced from all "given" and stored events. If that state is not identical, this means that a state change has occurred outside of an aggregate its event handler method. Static and transient fields are ignored in the comparison, as they typically contain references to resources.
>
> You can switch detection in the configuration of the fixture with the `setReportIllegalStateChange()` method.

The last phase is the validation phase, and allows you to check on the activities of the command handling component. This is done purely in terms of return values and events.

The test fixture allows you to validate return values of your command handlers. You can explicitly define the expected return value, or simply require that the method successfully returned. You may also express any exceptions you expect the CommandHandler to throw.

The other component is validation of published events. There are two ways of matching expected events.

The first is to pass in event instances that need to be literally compared with the actual events. All properties of the expected events are compared \(using `equals()`\) with their counterparts in the actual Events. If one of the properties is not equal, the test fails and an extensive error report is generated.

The other way of expressing expectancies is using "Matchers" \(provided by the Hamcrest library\). `Matcher` is an interface prescribing two methods: `matches(Object)` and `describeTo(Description)`. The first returns a boolean to indicate whether the matcher matches or not. The second allows you to express your expectation. For example, a "GreaterThanTwoMatcher" could append "any event with value greater than two" to the description. Descriptions allow expressive error messages to be created about why a test case fails.

Creating matchers for a list of events can be tedious and error-prone work. To simplify things, Axon provides a set of matchers that allow you to provide a set of event specific matchers and tell Axon how they should match against the list.

Below is an overview of the available event list matchers and their purpose:

* **List with all of**: `Matchers.listWithAllOf(event matchers...)`

  This matcher will succeed if all of the provided event matchers match against at least one event in the list of actual events. It does not matter whether multiple matchers match against the same event, nor if an event in the list does not match against any of the matchers.

* **List with any of**: `Matchers.listWithAnyOf(event matchers...)`

  This matcher will succeed if one or more of the provided event matchers matches against one or more of the events in the actual list of events. Some matchers may not even match at all, while another matches against multiple others.

* **Sequence of Events**: `Matchers.sequenceOf(event matchers...)`

  Use this matcher to verify that the actual events are match in the same order as the provided event matchers. It will succeed if each matcher matches against an event that comes after the event that the previous matcher matched against. This means that "gaps" with unmatched events may appear.

  If, after evaluating the events, more matchers are available, they are all matched against "`null`". It is up to the event matchers to decide whether they accept that or not.

* **Exact sequence of Events**: `Matchers.exactSequenceOf(event matchers...)`

  Variation of the "Sequence of Events" matcher where gaps of unmatched events are not allowed. This means each matcher must match against the event directly following the event the previous matcher matched against.

For convenience, a few commonly required event matchers are provided. They match against a single event instance:

* **Equal event**: `Matchers.equalTo(instance...)`

  Verifies that the given object is semantically equal to the given event. This matcher will compare all values in the fields of both actual and expected objects using a null-safe equals method. This means that events can be compared, even if they do not implement the equals method. The objects stored in fields of the given parameter _are_ compared using equals, requiring them to implement one correctly.

* **No more events**: `Matchers.andNoMore()` or `Matchers.nothing()`

  Only matches against a `null` value. This matcher can be added as last matcher to the _exact_ sequence of events matchers to ensure that no unmatched events remain.

Since the matchers are passed a list of event messages, you sometimes only want to verify the payload of the message. There are matchers to help you out:

* **Payload matching**: `Matchers.messageWithPayload(payload matcher)`

  Verifies that the payload of a message matches the given payload matcher.

* **Payloads matching**: `Matchers.payloadsMatching(list matcher)`

  Verifies that the payloads of the messages matches the given matcher. The given matcher must match against a list containing each of the messages payload. The payloads matching matcher is typically used as the outer matcher to prevent repetition of payload matchers.

Below is a small code sample displaying the usage of these matchers. In this example, we expect two events to be published. The first event must be a "ThirdEvent", and the second "aFourthEventWithSomeSpecialThings". There may be no third event, as that will fail against the "andNoMore" matcher.

```java
fixture.given(new FirstEvent(), new SecondEvent())
       .when(new DoSomethingCommand("aggregateId"))
       .expectEventsMatching(exactSequenceOf(
           // we can match against the payload only:
           messageWithPayload(equalTo(new ThirdEvent())),
           // this will match against a Message
           aFourthEventWithSomeSpecialThings(),
           // this will ensure that there are no more events
           andNoMore()
       ));

// or if we prefer to match on payloads only:
       .expectEventsMatching(payloadsMatching(
               exactSequenceOf(
                   // we only have payloads, so we can equalTo directly
                   equalTo(new ThirdEvent()),
                   // now, this matcher matches against the payload too
                   aFourthEventWithSomeSpecialThings(),
                   // this still requires that there is no more events
                   andNoMore()
               )
       ));
```
