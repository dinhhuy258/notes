# Kafka Connect

Kafka Connect is an opensource component of Apache Kafka and provides scalable and reliable way to transfer data from Kafka to other data systems like databases, filesystems, key-value stores and search indexes. It uses Connectors that moves large data sets into and out of Kafka.

![](https://user-images.githubusercontent.com/17776979/193088811-bd11466b-7990-45e6-b8ab-ebc7d4619b21.png)

## Kafka Connect Concepts

**Connectors**: the high level abstraction that coordinates data streaming by managing tasks

**Tasks** – the implementation of how data is copied to or from Kafka

**Workers** – the running processes that execute connectors and tasks

**Converters** – the code used to translate data between Connect and the system sending or receiving data

**Transforms** – simple logic to alter each message produced by or sent to a connector

**Dead Letter Queue** – how Connect handles connector errors

### Connectors

Connectors in Kafka Connect define where data should be copied to and from. A connector instance is a logical job that is responsible for managing the copying of data between Kafka and another system.

Connectors can be one of the following type:

- Source connectors that push data into Kafka
- Sink connectors that extract data out of Kafka

Plugins provide the implementation for Kafka Connect to run connector instances. Connector instances create the tasks required to transfer data in and out of Kafka. The Kafka Connect runtime orchestrates the tasks to split the work required between the worker pods.

We can find list of available connectors [here](https://www.confluent.io/product/connectors/). If we want to buld our own connector we can find more information in [developer guide](https://docs.confluent.io/platform/current/connect/devguide.html#core-concepts-and-apis)

### Tasks

Data transfer orchestrated by the Kafka Connect runtime is split into tasks that run in parallel.

- A source connector task polls the external data system and returns a list of records that a worker sends to the Kafka brokers.
- A sink connector task receives Kafka records from a worker for writing to the external data system.

For sink connectors, the number of tasks created relates to the number of partitions being consumed.

For source connectors, how the source data is partitioned is defined by the connector. You can control the maximum number of tasks that can run in parallel by setting tasksMax in the connector configuration. The connector might create fewer tasks than the maximum setting

### Workers

Connectors and tasks are logical units of work and must be scheduled to execute in a process. Kafka Connect calls these processes workers and has two types of workers: standalone and distributed.

### Converters

When a worker receives data, it converts the data into an appropriate format using a converter. You specify converters for workers in the worker config in the KafkaConnect resource.

Kafka Connect can convert data to and from formats supported by Kafka, such as JSON or Avro...

### Transforms

Kafka Connect translates and transforms external data. Single-message transforms change messages into a format suitable for the target destination. For example, a transform might insert or rename a field. Transforms can also filter and route data.

- Source connectors apply transforms before converting data into a format supported by Kafka.
- Sink connectors apply transforms after converting data into a format suitable for an external data system.

This is list of [Kafka Connect Transformations ](https://docs.confluent.io/platform/current/connect/transforms/overview.html#single-message-transforms-for-cp)

### Dead Letter Queue

An invalid record may occur for a number of reasons. One example is when a record arrives at the sink connector serialized in JSON format, but the sink connector configuration is expecting Avro format. When an invalid record cannot be processed by a sink connector, the error is handled based on the connector configuration property `errors.tolerance`.

Dead letter queues are only applicable for sink connectors.

There are two valid values for this configuration property: `none` (default) or `all`.

When `errors.tolerance` is set to `none` an error or invalid record causes the connector task to immediately fail and the connector goes into a failed state. To resolve this issue, you would need to review the Kafka Connect Worker log to find out what caused the failure, correct it, and restart the connector.

When `errors.tolerance` is set to `all`, all errors or invalid records are ignored and processing continues. No errors are written to the Connect Worker log. To determine if records are failing you must use internal metrics or count the number of records at the source and compare that with the number of records processed.

An error-handling feature is available that will route all invalid records to a special topic and report the error. This topic contains a dead letter queue of records that could not be processed by the sink connector.

### How Kafka Connect work

![](https://user-images.githubusercontent.com/17776979/193172263-8855684a-1a26-4b48-9047-c22abeec48f7.png)

1. A plugin provides the implementation artifacts for the source connector
2. A single worker initiates the source connector instance
3. The source connector creates the tasks to stream data
4. Tasks run in parallel to poll the external data system and return records
5. Transforms adjust the records, such as filtering or relabelling them
6. Converters put the records into a format suitable for Kafka
7. The source connector is managed using KafkaConnectors or the Kafka Connect API

![](https://user-images.githubusercontent.com/17776979/193172302-07ba81d0-9877-41cf-87be-99413de40314.png)

1. A plugin provides the implementation artifacts for the sink connector
2. A single worker initiates the sink connector instance
3. The sink connector creates the tasks to stream data
4. Tasks run in parallel to poll Kafka and return records
5. Converters put the records into a format suitable for the external data system
6. Transforms adjust the records, such as filtering or relabelling them
7. The sink connector is managed using KafkaConnectors or the Kafka Connect API
2. A single worker initiates the sink connector instance
3. The sink connector creates the tasks to stream data
4. Tasks run in parallel to poll Kafka and return records
5. Converters put the records into a format suitable for the external data system
6. Transforms adjust the records, such as filtering or relabelling them
7. The sink connector is managed using KafkaConnectors or the Kafka Connect API

