# Kafka architecture

## Kafka storage architecture

![](https://user-images.githubusercontent.com/17776979/251782588-53566595-df75-4fdd-b0f5-8355655f1950.png)

**Topic**

Kafka topics are the categories used to organize messages. Kafka Topics stores millions message of data

**Partitions**

Kafka organizes data into topics, which are further divided into partitions. Each partition is an ordered and immutable sequence of records. Partitioning allows Kafka to parallelize the data ingestion and processing across multiple brokers and enables horizontal scalability.

Kafka creates a separate directory for each topic partition. If you determined n partition count, Kafka going to create n directories for that topic (Eg: topic-0, topic-1, ..., topic-n).

**Replication Factor**

Kafka provides built-in replication for fault tolerance. Each partition can have multiple replicas distributed across different brokers. Replication ensures that data is replicated across multiple nodes, allowing for high availability and fault tolerance in case of broker failures.

Like partition, you can define the number of it when creating a new topic.

```
Number of Replicas(15) = Partitions(5) x Replication(3)
```

**Segment**

Kafka brokers splits each partition into segments. Each segment is stored in a single data file on the disk attached to the broker. By default, each segment contains either 1 GB of data or a week of data, whichever limit is attained first. When the Kafka broker receives data for a partition, as the segment limit is reached, it will close the file and start a new one:

![](https://user-images.githubusercontent.com/17776979/251786040-9efc2fe8-e45e-4a38-98da-166cdd801429.png)

Configuration:

- log.segment.bytes: the max size of a single segment in bytes (default 1 GB)
- log.segment.ms: the time Kafka will wait before committing the segment if not full (default 1 week)

**Kafka Topic Segments and Indexes**

Kafka allows consumers to start fetching messages from any available offset. In order to help brokers quickly locate the message for a given offset, Kafka maintains two indexes for each segment:

- An offset to position index - It helps Kafka know what part of a segment to read to find a message
- A timestamp to offset index - It allows Kafka to find messages with a specific timestamp

**Inspecting the Kafka Directory Structure**

Kafka stores all of its data in a directory on the broker disk. This directory is specified using the property `log.dirs` in the broker's configuration file.

Explore the directory and notice that there is a folder for each topic partition. All the segments of the partition are located inside the partition directory. Here, the topic named `configured-topic` has three partitions, each having one directory - `configured-topic-0`, `configured-topic-1` and `configured-topic-2`. Descend into a directory for a topic partition. Notice the indexes - time and offset for the segment and the segment file itself where the messages are stored.

![](https://user-images.githubusercontent.com/17776979/251789946-5c06e6a4-9dbe-4022-b804-221c266cf0b1.png)

## Kafka cluster architecture

![](https://user-images.githubusercontent.com/17776979/251801443-5b6d3515-975f-4f17-b6f9-5e2ee9f53b6b.png)

**Cluster membership**

A Kafka cluster consists of a group of brokers that are identified by a unique numeric ID. When the brokers connect to their configured ZooKeeper instances, a group znode is created with each broker creating an `ephemeral znode` under this group znode. If a node fails, the ephemeral nature of this znode means it gets removed from the group znode.

**Controller broker**

A controller broker is a normal broker that simply has additional responsibility.

The most important part of that additional responsibility is keeping track of nodes in the cluster and appropriately handling nodes that leave, join or fail. This includes rebalancing partitions and assigning new partition leaders.

- Cluster Management: The Controller broker is responsible for maintaining and managing the metadata of the Kafka cluster. It keeps track of information such as the list of topics, their partitions, replicas, and their current leaders.
- Leader Election: When a leader for a partition is unavailable or needs to be reassigned, the Controller broker is responsible for coordinating the leader election process.
- Topic and Partition Management: The Controller broker handles operations related to topic creation, deletion, and configuration changes. It ensures that partitions are evenly distributed across the brokers and handles partition reassignment when necessary, such as when brokers are added or removed from the cluster.
- Broker Failure Detection: The Controller continuously monitors the health of all brokers in the cluster. It detects and handles broker failures by initiating leader elections and replica reassignments for affected partitions.
- Controller Failover: In case the current Controller broker fails, another broker automatically takes over the role of the Controller to ensure uninterrupted cluster management. The Controller uses Apache ZooKeeper for leader election and coordination during failover scenarios. `/controller`

## Kafka work distribution architecture
