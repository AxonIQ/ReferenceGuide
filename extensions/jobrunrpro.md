# JobRunrPro

> Legacy Documentation Warning!
>
> Since the first of October 2024, we moved Axon's documentation from the "Reference Guide" to "AxonIQ Docs."
> You can find the latter at https://docs.axoniq.io/
>
> Hence, if you are looking for the documentation for newer versions of Axon Framework, Axon Server, or any of the Framework Extensions, we strongly recommend you browse to [AxonIQ Docs](https://docs.axoniq.io/) instead.

The purpose of this extension is to leverage some features only available in the Pro version of [JobRunr](https://www.jobrunr.io/en/documentation/pro/).
Only the Pro version allows to search existing jobs by status and label. This is required for the deadline manager to implement the `cancelAll` methods.
The [deadline managers](../axon-framework/deadlines/deadline-managers.md) section has more in depth information on deadline managers.
Although jobs created with the non-pro `DeadlineManager` will be eligible to be canceled, this is only true when they were created with the `4.8` or later versions.
Jobs created with the `4.7.x` version are missing the correct labels and will not be found when trying to cancel them.

## Spring usage

For Spring usage, be sure to include the starters, both of JobRunr Pro and the extension. The deadline manager should be available by using parameter-based injection.
It can be used in aggregates and sagas using something like:
{% tabs %}
{% tab title="Aggregate" %}
```java
@CommandHandler
public void handle(SomeCommand command, @Autowired DeadlineManager deadlineManager) { ... }
```
{% endtab %}
{% tab title="Saga" %}
```java
@StartSaga
public void handle(SomeEvent event, @Autowired DeadlineManager deadlineManager) { ... }
```

Alternatively, you could also auto wire the deadline manager to the class like:
```java
@Autowired
void setDeadlineManager(DeadlineManager deadlineManager) {
    this.deadlineManager = deadlineManager;
}
```
{% endtab %}
{% endtabs %}

## Non Spring Usage

An `JobRunrProDeadlineManager` instance can be created using the builder like this:
```java
JobRunrProDeadlineManager.proBuilder()
        .jobScheduler(jobScheduler)
        .storageProvider(storageProvider)
        .scopeAwareProvider(scopeAwareProvider)
        .serializer(serializer)
        .transactionManager(transactionManager)
        .spanFactory(spanFactory)
        .build();
```

You probably want to use some form of dependency injection instead of creating a new deadline manager each time you need one.



