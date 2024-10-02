# Diagnostics

When reaching out to AxonIQ for Axon Server support related questions, the support team needs information about the environment and its current state. Axon Server provides an endpoint that collects the relevant information into a zip file.

The URL is `v1/diagnose/download` (in Axon Server versions before 2024.0 the URL is `internal/diagnose/download`).

The zip file contains the following information:

- cluster layout (nodes in the cluster)
- application logs
- replication status per replication group
- list of files per context
- system information (available processors, memory and java version used)
- thread dump of the Axon Server process
- health status (information from /actuator/health)
- metrics 
