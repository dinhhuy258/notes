# Kafka architecture

## Kafka storage architecture

![](https://user-images.githubusercontent.com/17776979/251782588-53566595-df75-4fdd-b0f5-8355655f1950.png)

### Topic

Kafka topics are the categories used to organize messages. Kafka Topics stores millions message of data

### Partitions

Kafka organizes data into topics, which are further divided into partitions. Each partition is an ordered and immutable sequence of records. Partitioning allows Kafka to parallelize the data ingestion and processing across multiple brokers and enables horizontal scalability.

Kafka creates a separate directory for each topic partition. If you determined n partition count, Kafka going to create n directories for that topic (Eg: topic-0, topic-1, ..., topic-n).

### Replication Factor

Kafka provides built-in replication for fault tolerance. Each partition can have multiple replicas distributed across different brokers. Replication ensures that data is replicated across multiple nodes, allowing for high availability and fault tolerance in case of broker failures.

Like partition, you can define the number of it when creating a new topic.

```
Number of Replicas(15) = Partitions(5) x Replication(3)
```

### Segment

Kafka brokers splits each partition into segments. Each segment is stored in a single data file on the disk attached to the broker. By default, each segment contains either 1 GB of data or a week of data, whichever limit is attained first. When the Kafka broker receives data for a partition, as the segment limit is reached, it will close the file and start a new one:

![](https://user-images.githubusercontent.com/17776979/251786040-9efc2fe8-e45e-4a38-98da-166cdd801429.png)

Configuration:

- log.segment.bytes: the max size of a single segment in bytes (default 1 GB)
- log.segment.ms: the time Kafka will wait before committing the segment if not full (default 1 week)

### Kafka Topic Segments and Indexes

Kafka allows consumers to start fetching messages from any available offset. In order to help brokers quickly locate the message for a given offset, Kafka maintains two indexes for each segment:

- An offset to position index - It helps Kafka know what part of a segment to read to find a message
- A timestamp to offset index - It allows Kafka to find messages with a specific timestamp

### Inspecting the Kafka Directory Structure

Kafka stores all of its data in a directory on the broker disk. This directory is specified using the property `log.dirs` in the broker's configuration file.

Explore the directory and notice that there is a folder for each topic partition. All the segments of the partition are located inside the partition directory. Here, the topic named `configured-topic` has three partitions, each having one directory - `configured-topic-0`, `configured-topic-1` and `configured-topic-2`. Descend into a directory for a topic partition. Notice the indexes - time and offset for the segment and the segment file itself where the messages are stored.

![](https://user-images.githubusercontent.com/17776979/251789946-5c06e6a4-9dbe-4022-b804-221c266cf0b1.png)

## Kafka cluster architecture

![](https://user-images.githubusercontent.com/17776979/251801443-5b6d3515-975f-4f17-b6f9-5e2ee9f53b6b.png)

### Cluster membership

A Kafka cluster consists of a group of brokers that are identified by a unique numeric ID. When the brokers connect to their configured ZooKeeper instances, a group znode is created with each broker creating an `ephemeral znode` under this group znode. If a node fails, the ephemeral nature of this znode means it gets removed from the group znode.

### Controller broker

A controller broker is a normal broker that simply has additional responsibility.

The most important part of that additional responsibility is keeping track of nodes in the cluster and appropriately handling nodes that leave, join or fail. This includes rebalancing partitions and assigning new partition leaders.

- Cluster Management: The Controller broker is responsible for maintaining and managing the metadata of the Kafka cluster. It keeps track of information such as the list of topics, their partitions, replicas, and their current leaders.
- Leader Election: When a leader for a partition is unavailable or needs to be reassigned, the Controller broker is responsible for coordinating the leader election process.
- Topic and Partition Management: The Controller broker handles operations related to topic creation, deletion, and configuration changes. It ensures that partitions are evenly distributed across the brokers and handles partition reassignment when necessary, such as when brokers are added or removed from the cluster.
- Broker Failure Detection: The Controller continuously monitors the health of all brokers in the cluster. It detects and handles broker failures by initiating leader elections and replica reassignments for affected partitions.
- Controller Failover: In case the current Controller broker fails, another broker automatically takes over the role of the Controller to ensure uninterrupted cluster management. The Controller uses Apache ZooKeeper for leader election and coordination during failover scenarios. `/controller`

## Kafka work distribution architecture

### Partition assignment

![](https://user-images.githubusercontent.com/17776979/251937918-1d33f192-8c16-4bce-a42a-713b81d4064f.png)

To distribute partitions, Kafka applies the following steps:

1. Ordered List of Brokers

Starting from a randomly chosen broker, the next broker must be one in a different rack and so on.

2. Leader and Follower Assignment

![](https://user-images.githubusercontent.com/17776979/251938385-38161798-fba3-41ef-ab72-78ff2cc27232.png)

Using the round robin method, Kafka assigns the leader partitions first.

![](https://user-images.githubusercontent.com/17776979/251938396-5c652391-2839-4863-bbac-677dba26cf4f.png)

Then it assigns the first follower, starting from the second broker in the list and following the round robin method.

![](https://user-images.githubusercontent.com/17776979/251938403-bb05bb36-7d51-4f0b-b8d5-c4a36056accf.png)

The second follower of the partition is assigned starting from the third broker in the list and similarly following the round robin method to distribute partitions.
![](https://user-images.githubusercontent.com/17776979/251938412-88ecfac6-62cc-4e99-bf1c-02d70fe5dd07.png)

### Leader Partition

- The leader partition is responsible for handling all read and write requests for that partition.
- Producers send messages to the leader partition, and consumers fetch messages from the leader partition.
- The leader partition maintains the in-sync replicas (ISRs) list, which includes the followers that are up-to-date with the leader's data.

### Follower Partition

- Follower partitions are replicas of a topic partition that replicate the data stored in the leader partition.
- Follower partitions are not directly involved in handling read and write requests from clients.
- The primary role of follower partitions is to replicate the leader partition's data and stay in sync with it.

**How producer and consumer can know which broker is leader partition**

Producers and consumer can retrieve metadata information about the Kafka cluster. The metadata includes the list of brokers, topics, partitions, and the current leader for each partition.

### In Sync Replicas

In sync Replicas (or ISR’s) are replicas which are in sync with the leader.

Follower can not be in synced with the leader because of:

- Network congestion
- Follower broker cash/ restart

The leader partition maintains the in-sync replicas (ISRs) list and persisted in the ZooKeeper. All of the follower in ISRs list are known to be in synced with the leader, and they are an excellent candidate to be elected as the new leader when something wrong with the current leader.

How leader know if follower is in synced or lagging behind?

Followers stay in sync by sending fetch requests to the leader, which returns messages to the follower in order. The follower is considered to be in sync if it has caught up with the most recently committed message on the leader. The leader checks this by looking at the last offset requested by the follower.

![](https://user-images.githubusercontent.com/17776979/251939390-6fd62169-e89f-41d3-a63c-a7e39c420785.png)

If the replica has requested the most recent message in the last 10 seconds they deserve to be in the ISR list. If not, the leader removes the replica from the IRS list.

### How Kafka deals with messages in-consistency

![](https://user-images.githubusercontent.com/17776979/251939893-1f3306e4-b331-4a80-8c81-43f16883303c.png)

Assuming that all the followers in the ISR list are 11 seconds behind the leader, the ISR list would be empty. If the leader crashed and we had to elect a new leader among the followers, then we would lose messages.

The solution is simple which is implemented using two concepts:

**Committed and Uncommitted messages**

![](https://user-images.githubusercontent.com/17776979/251939919-99299222-0b05-48ae-9cc9-29628a1d0b68.png)

The first idea is to introduce the notion of committed and uncommited messages.

We can configure the leader to not consider a message commited until the message is copied at all the followers in the ISR list => the leader may have some committed as well as some uncommitted messages.

If the leader crashed the uncommitted messages can be resent by the producer. (Producers can choose to receive acknowledgments of sent messages only after the message is fully committed)

**Minimum In-Sync replicas**

![](https://user-images.githubusercontent.com/17776979/251939967-6088e759-d95a-4283-9c30-a0654f49296e.png)

Let's Assume that we start with three replicas and all of them are healthy enough to be in the ISR. However, after some time two of them failed and as a result of that, the leader will remove them from the ISR, we are left with a single in-sync replica that is the leader itself.

Now the data is considered committed when it is written to all in-sync replicas, even when all means just one replica (the leader itself).

It is a risky scenario for data consistency because data could be lost if we lose the leader.

The second idea is to set a minimum in-sync replicas configuration.

![](https://user-images.githubusercontent.com/17776979/251939990-12f9cdde-855e-40d9-bf92-7275039e9519.png)

If you would like to be sure that committed data is written to at least two replicas, you need to set the minimum number of in-sync replicas as two.
