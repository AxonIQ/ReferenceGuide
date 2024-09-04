# Minor Releases

This page provides a dedicated overview of patch releases for Axon Server. For information about the
older releases check either [Axon Server Enterprise Edition](rn-asee-minor-releases.md) or [Axon Server Standard Edition](rn-asse-minor-releases.md).

## Release 2024.1

## Release 2024.1.2

Bug fixes and improvements:
- Redistribute clients across Axon Server nodes when a node is restarted
- Event processor operations fail when the processing group contains a forward slash
- Potential replication issue when trying to apply events for already closed contexts during shutdown of Axon Server
- Increased maximum length for the username to 255 characters
- Update the event store size when a new index file is created
- Visual improvements in search table: headers not visible by default & action not visible by default
- Improved logging in the event store
- Stop replication applying process when the replication group is stopped
- Global Index pre-load for configured contexts

## Release 2024.1.1

Bug fixes and improvements:
- Revert optimization in replication from version 2023.2.4, as it could lead to a node entering fatal state
- Stop Axon Server from redirecting a client to a node that is in fatal state
- Reduce communication between the leader and follower and logging when a node is starting up
- Search page improvements
- Set correct permissions for persistent stream API calls
- Add validation of newly created index files
- Allow non-pristine clusters to connect to Console
- Fix the event store size in the context page

## Release 2024.0

## Release 2024.0.4

Fixes and improvements:
- Fix for a problem starting up Axon Server with plugins configured
- Removed race condition causing a possible delay in receiving the first event on a newly registered event handler
- Improve the diagnostics package to contain full log information when "logging.config" property is set
- 
### Release 2024.0.3

Fixes and improvements:
- Add an option to reduce the number of global index segments Axon Server checks when the first event for a new
  aggregate is stored. This can be configured globally with the property
  "axoniq.axonserver.event.global-index-segments-check" or on a context level with the property
  "event.global-index-segments-check". The value is the number of global index segments to check, with a
  minimal value of 2.
- Fix for Control DB migration in case of plugin configuration properties with long values
- Updating a license through Axon Console now takes effect immediately
- Improved distribution of queries to different instances of the query handlers

### Release 2024.0.2

Fixes and improvements:
- Updating a license through Axon Console now takes effect immediately
- Reduced memory usage for internal communication
- Reduced the number of threads used with a large number of contexts
- UI improvements
  * The dialogs for adding replication groups, API tokens, and users were not always cleared when opened
  * show the number of events in each context
  * improved notification when the current version is not the latest one
  * add an option to set X-Frame-Options to SAMEORIGIN in the response messages

New configuration parameters:
- axoniq.axonserver.accesscontrol.same-origin=false (`true` sets the X-Frame-Options header to SAMEORIGIN)
- axoniq.axonserver.event-store-background-thread-count=8
- axoniq.axonserver.event-store-processors-thread-count=8


### Release 2024.0.1

- Fix the increasing number of threads on the running Axon Server nodes when one node in the cluster is down.
- Small fixes in the replication process:
  * remove delay in starting to synchronize with a node that is far behind
  * improve the performance for a follower catching up with the leader
  * prevent situations where a follower attempts to apply replication log entries that were already included in a snapshot
- Fix for authentication issue when multiple applications have the same token
- UI, copy token to clipboard fails when not running on a trusted URL
- UI, improved validations for applications, replication groups and contexts operations
- Improved handling for missing connection to Axon Console
- Support for Google Marketplace licenses
- Axon Server now performs a clean shutdown when it was started with an incorrect node name or internal hostname/port

## Release 2023.2

## Release 2023.2.9

Bug fixes and improvements:
- Redistribute clients across Axon Server nodes when a node is restarted
- Event processor operations fail when the processing group contains a forward slash
- Potential replication issue when trying to apply events for already closed contexts during shutdown of Axon Server

## Release 2023.2.8

Bug fixes and improvements:
- Revert optimization in replication from version 2023.2.4, as it could lead to a node entering fatal state
- Stop Axon Server from redirecting a client to a node that is in fatal state
- Reduce communication between the leader and follower and logging when a node is starting up

### Release 2023.2.7

Bug fixes:
- Improved distribution of queries to different instances of the query handlers
- Prevent stale threads when an Axon Server node closes the connection to another node
- Clean up metrics from disconnected clients
- prevent WARN log messages when a query completed message was received from an unexpected client

Dependency updates:
- GRPC version updated to 1.65.1

### Release 2023.2.6

Fixes and improvements:
- Add an option to reduce the number of global index segments Axon Server checks when the first event for a new
  aggregate is stored. This can be configured globally with the property
  "axoniq.axonserver.event.global-index-segments-check" or on a context level with the property
  "event.global-index-segments-check". The value is the number of global index segments to check, with a
  minimal value of 2.

### Release 2023.2.5

- Reduced memory usage for internal communication

### Release 2023.2.4

- Fix the increasing number of threads on the running Axon Server nodes when one node in the cluster is down.
- Small fixes in the replication process:
  * remove delay in starting to synchronize with a node that is far behind
  * improve the performance for a follower catching up with the leader
  * prevent situations where a follower attempts to apply replication log entries that were already included in a snapshot
- Fix for authentication issue when multiple applications have the same token

### Release 2023.2.3

Bug fix:

- Increasing number of threads on the running Axon Server nodes when one node in the cluster is down.

### Release 2023.2.2

Bug fixes:

- Fix for an error handling subscription query responses during the upgrade from a version before 2023.2.0 to 2023.2.0 or 2023.2.1.
- Improved readiness probe to return 200 (OK) once the communication services are ready and the replication groups are completely initialized.
  The endpoint for the new readiness probe is /actuator/health/readiness.

### Release 2023.2.1

#### Bug Fixes

This release contains fixes for the following issues:

- TLS communication between Axon Server nodes cannot validate trusted certificates when there is no trust manager file configured
- Deleting a context does not delete all its metrics


## Release 2023.1

### Release 2023.1.2

#### Bug Fixes

This release contains fixes for the following issues:
- Metrics no longer collected when an application reconnects to Axon Server

### Release 2023.1.1

#### New Features and Enhancements:

_Initialize standalone_ 

To simplify initialization of Axon Server, it now supports a new property "axoniq.axonserver.standalone=true". When this property is set on a clean Axon Server instance it initializes the server with a "default" context.

_Development mode_

Fixed the option to reset the event store from the UI (in development mode). This option now also works in an
Axon Server cluster.

_LDAP extension update_

The new version of the LDAP extension supports configuration of a trust manager file. The location of the file
can be specified through the property "axoniq.axonserver.enterprise.ldap.trust-manager-file".

#### Bug Fixes

This release contains fixes for the following issues:
- Validation of tiered storage properties when not using the UI
- Race condition while writing to the global index
- Limitation on the number of requests per context fails if there are timed out requests

