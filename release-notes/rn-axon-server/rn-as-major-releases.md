# Major Releases

## Release 2024.1

### Persistent streams

Persistent streams provide the option to open an event stream from a client and let Axon Server track the progress. This
was already available as a preview version in 2024.0, but is now available by default. Persistent streams are supported
in Axon Framework 4.10 as an alternative to tracking or pooled streaming event processors.

For more information see [subcribing event processor](https://library.axoniq.io/axon_framework_old_ref/events/event-processors/subscribing.html) in the Axon Framework section.

### Bug fixes and improvements

- Prevent stale threads when an Axon Server node closes the connection to another node
- Clean up metrics from disconnected clients
- prevent WARN log messages when a query completed message was received from an unexpected client
- Allow context with ephemeral events without a license
- Fix for listing event processors when there are more than 512 event processors

### Docker image changes

The default Java version for the Docker images has changed from Java 11 to Java 17. This means that the docker images with tag "latest", "latest-nonroot", "2024.1.0", and "2024.1.0-nonroot" use Java 17. Java 11 based images are still available with the "-jdk-11" extensions in the tag name.

### Dependency updates

- gRPC version updated to 1.65.1


## Release 2024.0

### Database Update

Updated H2 database to store the Control DB, addressing some issues from previous H2 version (see the upgrade instructions in https://library.axoniq.io/axon_server_upgrade/upgrading_as_2024.html).

### New Features and Improvements

Redesigned User Interface: The UI has been completely revamped with a modern look and feel for a better user experience. The changes include:

- Simplified Overview Page: Access node information easily with filtering and scaling options.
- Dedicated License Page: Track license expiry dates and view available features for non-enterprise users.
- Monitoring Page: View important health information, display logs, and download diagnostic packages.
- System Tasks: List and cancel running system tasks.
- Search Event Store Page: Improved usability with removable columns, formatted code styles, and auto-composable queries.
- Command and Queries Pages: Revamped for a better overview of messages in the system.
- Long-Running Commands Component: View and cancel commands running longer than 1 second.
- Scheduled Events Page: View and cancel scheduled events.
- Streams Page (Experimental): Accessible for persistent streams if dev mode is enabled.
- API Tokens (formerly Applications): Renamed for clarity, with improved token management.
- Support for Wide Screens and Dark/Light Themes: Enhanced viewing experience.
- Connection, Health, and Early Event Processor Issue Detection: Improved issue detection and resolution.
- Embedded Documentation Snippets: Access documentation directly within the UI.

Preview of new persistent streams feature, event streams where Axon Server manages the publication of events to clients and keeps track of the progress. This feature is enabled when development mode is enabled or when axoniq.axonserver.preview.persistent-streams property is set to true.


## Release 2023.2

### TLS Certificate and Key Replacement at Runtime

Axon Server now supports the hot (runtime) replacement of certificates and keys used for TLS, eliminating the need for server restarts.

### Enhanced Metrics Exposure

We have revamped the metrics exposed by Axon Server for better clarity and comprehensibility. Adhering to the 4 golden signals terminology, metrics are now systematically organized.
Users can access both old and new style metrics in this version. However, there’s an option to disable the old style metrics.

### Upgraded Diagnostics Package

To aid in issue resolution, Axon Server now provides a more comprehensive diagnostics package.
The package now contains more detailed information about raft status.
It offers a snapshot of metrics and health information.
There’s a listing of files in the replication group.
Information about multi-tier storage is included.
Logs are included as well.

### Other Improvements

- We’ve addressed various security concerns through dependency updates. Additionally, several bugs have been identified and rectified.
  The role ‘MONITOR’ is now granted permission to access the ‘internal/raft/status’ endpoint.
- We’ve transitioned to new versions for some of the external libraries used in Axon Server.

## Release 2023.1

### New Features and Enhancements

#### Event Transformation

The new Event Transformation feature allows users to perform specific event transformations like updates and deletes in the event store, utilizing the Event Transformation API. This functional change is intended to facilitate more flexible event management in rare instances where modifications are unavoidable.

#### Forced Client Reconnection

In the application view, users are now provided with an option to force the client to reconnect. This addition aims to offer a practical tool for addressing client connectivity issues.

#### Node Removal from Cluster

It is now possible to remove a node from the cluster through the user interface (UI). This functionality, previously accessible only via the command-line interface (CLI) and REST API, has been expanded to the UI for broader accessibility.

#### Temporary Adjustment to Development Mode

In this release, we have temporarily disabled the 'Development Mode/Event Purge' feature. Users should now utilize the 'Delete/Create Context' operation as an alternative. This change will remain in place until a more efficient solution is implemented.

#### Enhanced Memory Management
  In an effort to optimize performance, we have updated Axon Server's approach to memory management for file resources. Prior to this release, Axon Server primarily depended on the Java garbage collector to reclaim memory used by memory-mapped files. With this update, memory management is now undertaken directly by Axon Server, enhancing efficiency in file resource usage.

### Bug Fixes

This release also contains fixes for the following issues:

- Replication group creation did not work in conjunction with the HTTPS (-s) option
- Race condition in unregister node leaves node partially in the cluster

### Product Updates

#### Unified Axon Server artifact

The Axon Server artifact has been updated to simplify the deployment process. Instead of separate artifacts for the Axon Server Standard Edition and Enterprise Edition, we are releasing a single artifact from now on. The Axon Server features will adjust automatically based on the presence of a provided license. Note that the Axon Server Standard Edition remains open-source, but separate releases will no longer be made.

The **axoniq/axonserver-enterprise** docker image is no longer updated. To use the latest version, use the **axoniq/axonserver image** with tags ending with -dev. For instance, if you use **axoniq/axonserver-enterprise:2023.0.1-dev**, you can switch to **axoniq/axonserver:2023.1.0-dev**.

If you are used to Axon Server Standard Edition, note that there is a difference in the server's initialization. As the server can now run standalone or as a node in a cluster, you have to tell it how it should initialize itself. The dashboard provides an initial page to do this initialization. To automatically initialize the server as a standalone server, add the following properties:

```
axoniq.axonserver.autocluster.first=<the hostname of the server>
axoniq.axonserver.autocluster.contexts=default
```
## Release 2023.0

### New release schedule
Our new release strategy involves releasing three versions each year, with new releases being named based on the year, such as 2023.0, 2023.1, and 2023.2. The third release is dedicated to providing long-term support. Customers that wish to upgrade less frequently can choose to stay on these long-term support releases.

We chose this approach to separate versioning from the Axon Framework and maintained flexibility and independence in versioning. This strategy provides predictable release dates, better alignment with customer needs, faster feature updates, and increased responsiveness to user feedback and market demands. Overall, this approach will enable us to deliver better products and services to our customers while ensuring maximum flexibility and independence in versioning.

### New features
The 2023.0 release brings us these new features:
- Tiered Storage
- Ephemeral Contexts

#### Tiered Storage

Tiered Storage is a highly anticipated feature of the Axon Server Enterprise's 2023.0.0 release, allowing each node to store its data across different storage locations. With its ability to optimize performance and reduce storage costs, Tiered Storage is a powerful tool for businesses seeking to improve their storage management strategies. By distributing data across different levels of storage media based on access speed and cost, Tiered Storage enables organizations to minimize the cost of storing infrequently accessed data while ensuring fast access to frequently accessed data.

The Tiered Storage feature is especially powerful in combination with the use of secondary nodes, a feature that has been around since Axon Server Enterprise version 4.4:

Tiered Storage enables the configuration and maintenance of a variable number of storage tiers for each node, depending on its role, such as primary, secondary, or backup nodes.

Secondary nodes enable you to reduce the number of copies of data that are stored by keeping only the most recent events stored on your primary nodes and keeping the full event store on the secondary nodes. The configuration of Tiered Storage on these secondary nodes can focus on lower storage costs and large volumes of data, while Primary nodes focus on speed of access.

With its advanced capabilities, Tiered Storage is poised to become a critical asset for enterprises seeking to streamline their data storage and management processes.


#### Ephemeral Contexts

Ephemeral contexts are a new type of context that store events for a limited time before automatically removing them permanently. These contexts are particularly useful in scenarios such as time-limited audit systems or integration contexts, where events are broadcasted to multiple observers in real-time, after which they are no more of use.

As events might become outdated and irrelevant over time, ephemeral contexts can help to manage storage space and maintain system efficiency by automatically deleting these events after a specified time period. By using ephemeral contexts, organizations can efficiently store only necessary data and reduce clutter in their systems.


## Release 4.6.0

Standard Edition and Enterprise Edition

New features:
- Streaming queries (requires Axon Framework version 4.6.0). When returning a collection of results from a query, the results can be streamed instead of collecting
  them in the query handler first.

Enhancements:
- Number of events per transaction is no longer limited to 32K.
- Support for using the CLI when the caller is behind a proxy.
- livenessstate and readinessstate probes are now included in the /actuator/health endpoint output by default
- Properties now support more readable values using units
- Changing event processor states through Axon Server now waits for a result from the client
- Plugins can now use AxonServerInformationProvider to get information on the Axon Server version
- UI updated
- show complex metadata values in query results

Dependency updates:
- updated gRPC and Netty versions
- updated Spring Boot version
- moved to OpenAPI for Swagger support

Bug fixes:
- moved reading indexes from the gRPC thread to prevent blocking these threads

Notes:
- For the Swagger endpoint use  /swagger-ui.html or /swagger-ui/index.html.
- The generic endpoint for actuator is /actuator (/actuator/ no longer works)

Enterprise Edition only

New features:
- Clients can now perform administrative operations in Axon Server through the axonserver-connector-java (requires version 4.6.0 of the connector).
- Support for updating properties for an existing context

Enhancements:
- Large message support. Before increasing the max-message-size on Axon Server had the effect that the replication log segments were also increased in size. This
  dependency is no longer there.
- Clients can now also connect to nodes in a replication group that are not primary or messaging-only nodes.
  To force clients to connect to primary or message-only nodes, set the property force-connection-to-primary-or-messaging-node to true

Bug fixes:
- not able to use an empty internal domain value if the domain was specified for the client connections (requires property
  axoniq.axonserver.experimental.allow-empty-domain set to true and axoniq.axonserver.internal-domain defined with an empty string)
- fix a timing issue in LeadershipStatusNotifier that can cause the leader to forget about its own leader status

Notes:
- For the swagger endpoint use  /swagger-ui.html or /swagger-ui/index.html.
- The generic endpoint for actuator is /actuator (/actuator/ no longer works)
- If you are using the LDAP or OATH extension you need to use the 4.6.0 version
  of these extensions as because of the Spring Boot version update


## Release 4.5

Standard Edition and Enterprise Edition

New features:
- Support for customer-defined plugins to add custom actions to adding/reading events and snapshots and executing commands and (subscription queries)
  For more information see [plugins](../../axon-server/administration/plugins.md).
- Search snapshots in Axon Dashboard

Enhancements:
- Flow control for reading aggregates
- Updates in user permissions now have immediate effect (even if the user is logged in)
- Logging of illegal access to Axon Server gRPC services
- Improved monitoring of available disk space (see [actuator-endpoints](../../axon-server/administration/monitoring/actuator-endpoints.md))
- List of used 3rd party libraries available from Dashboard
- Axon Dashboard checks for Axon Server version updates

Dependency updates:
- updated gRPC and Netty versions
- updated Spring Boot version
- updated Swagger version

Notes:
- Due to the update of the Spring Boot version there are some minor changes to the output of the /actuator/health endpoint.
  This endpoint now uses the element "components" instead of "details" to output the health per subcategory.

- The swagger endpoint has changed from /swagger-ui.html to /swagger-ui/.

Standard Edition only

Bug fixes:
- Read aggregate snapshots from closed segments fixed

Enterprise Edition only

New features:
- Authentication via Google OAuth (via extension)
- Authentication and authorization with LDAP
- New role VIEW_CONFIGURATION allowing user read only access to the configuration of Axon Server in the Dashboard

Enhancements:
- Option to use a separate SSL key file for communication between Axon Server nodes (see property internal-private-key-file in [configuration](../../axon-server/administration/admin-configuration/configuration.md))
- License status added to health page (see [actuator-endpoints](../../axon-server/administration/monitoring/actuator-endpoints.md))
- Migration tool now supports migrating from a MongoDB event store to Axon Server

Bug fixes:
- memory leak in the replication group leader could cause elections
- incorrect version number shown in login page and error page in Dashboard
- invalid number of active subscription queries displayed in Dashboard
- exception initializing a context for the replication group leader on restart after unclean shutdown

## Release 4.4

Standard Edition and Enterprise Edition

* Axon Server can now act as an event scheduler
* Tag-based routing of commands and queries
* Support fom token store identifiers to identify which tracking event processors share a token store

Enterprise Edition only

* Tracking event processors can now read from any primary node
* Reading aggregates will read events from followers and only request the leader for latest events
* Introduction of replication groups containing one or more contexts to reduce overhead in replication process
* New index type to improve speed in reading aggregates
* Axon Server Enterprise can now start without a license and has the option to upload licenses to the cluster
* Cluster templates to initialize a cluster including replication groups, contexts, users and applications based on a
    template
* Multi-tier storage to keep only recent data at primary nodes
* Some storage properties can now be set at the context level

## Release 4.3

* Introduced new roles for nodes in context \(ACTIVE\_BACKUP, PASSIVE\_BACKUP and MESSAGING\_ONLY\)
* Improved support for running in containers
* New option to configure cluster information in configuration files, to automatically build cluster at startup
* Support load factor for commands
* Support for moving Axon Server cluster to other nodes
* Option to remove a node from a context without deleting the event store on that node
* Separate audit log for configuration changes
* Changed metrics to use common names and tags

## Release 4.2

* Delete leader from group is now possible.
* Removing a context from a node, deletes all data \(including event data\) on that node.
* Deleting a context removes all data \(including event data\).
* Blacklisting event types for applications that cannot handle these events \(requires AxonFramework v4.2.\)
* Expose tracking event processor position and status  \(requires AxonFramework v4.2.\)
* Clients can provide preferences on which node to connect to  \(requires AxonFramework v4.2.\)
* New access control roles
* Reduced leader changes by new pre-vote phase
* Improved health page

## Release 4.1

* Support for Split/Merge of tracking event processors through user interface
* Introduction of \_admin context for all cluster management operations
* Updated process for getting started \(see above\)
* Replication of data and configuration between nodes in the cluster is now based on transaction log replication.

  You will see new files created on AxonServer nodes in the log directory \(axoniq.axonserver.replication.log-storage-folder\).

  Do not delete those files manually!

* Default setting for health endpoint \(/actuator/heath\) has changed to show more details.
* Change in TLS configuration for communication between AxonServer nodes \(new property axoniq.axonserver.ssl.internal-trust-manager-file\)

