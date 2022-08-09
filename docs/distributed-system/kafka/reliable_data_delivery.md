# Reliable Data Delivery

Reliability is a property of a system not of a single component so when we are talking about the reliability guarantees of Apache Kafka, we will need to keep the entire system and its use cases in mind

## Reliability Guarantees

Apache Kafka guarantee:

- Kafka provides order guarantee of messages in a partition. If message B was written after message A, using the same producer in the same partition, then Kafka guarantee that the offset of message B will be higher than message A, and that consumers will read message B after message A.
- Produced messages are considered `committed` when they were written to the partition on all its in-sync replicas. Producers can choose to receive acknowledgements of sent messages when the message was fully committed, when it was written to the leader, or when it was sent over the network. Messages that are committed will not be lost as long as at least one replica remain alive.
- Consumers can only read messages that are committed.

## Replication

Each Kafka topic is broken down into partitions, which are the basic data building blocks.

A partition is stored on a single disk. Kafka guarantees the order of events within a partition, and a partition can be either online (available) or offline (unavailable).

Each partition can have multiple replicas, one of which is a designated leader. All events are produced to the leader replica and are usually consumed from the leader replica as well. Other replicas just need to stay in sync with the leader and rep‐ licate all the recent events on time. If the leader becomes unavailable, one of the insync replicas becomes the new leader

A replica is considered in sync if it is the leader for a partition, or if it is a follower that:

- Has an active session with ZooKeeper, meaning that it sent a heartbeat to ZooKeeper in the last 6 seconds (configurable).
- Fetched messages from the leader in the last 10 seconds (configurable).
- Fetched the most recent messages from the leader in the last 10 seconds. That is, it isn’t enough that the follower is still getting messages from the leader; it must have had no lag at least once in the last 10 seconds (configurable).

## Broker

There are three configuration parameters in the broker that change Kafka’s behavior regarding reliable message storage.

### Replication Factor

The topic-level configuration is replication.factor. At the broker level, we control the `default.replication.factor` for automatically created topics.

A replication factor of N allows us to lose N-1 brokers while still being able to read and write data to the topic. So a higher replication factor leads to higher availability, higher reliability, and fewer disasters.

On the flip side, for a replication factor of N, we will need at least N brokers and we will store N copies of the data, meaning we will need N times as much disk space. We are basically trading availability for hardware.

### Unclean Leader Election

This configuration is only available at the broker (and in practice, cluster-wide) level. The parameter name is `unclean.leader.election.enable`, and by default it is set to `false`.

When the leader for a partition is no longer available, one of the in-sync replicas will be chosen as the new leader. This leader election is `clean` in the sense that it guarantees no loss of committed data—by definition, committed data exists on all in-sync replicas.

But what do we do when no in-sync replica exists except for the leader that just became unavailable?

For example: The partition had three replicas, and the two followers became unavailable. In this situation, as producers continue writing to the leader, all the messages are acknowledged and committed (since the leader is the one and only in-sync replica). Now let’s say that the leader becomes unavailable. In this scenario, if one of the out-of-sync followers starts first, we have an out-of-sync replica as the only available replica for the partition.

In this scenario, we need to make a difficult decision:

- If we don’t allow the out-of-sync replica to become the new leader, the partition will remain offline until we bring the old leader back to online
- If we do allow the out-of-sync replica to become the new leader, we are going to lose all messages that were written to the old leader while that replica was out of sync and also cause some inconsistencies in consumers

By default, `unclean.leader.election.enable` is set to false, which will not allow out-of-sync replicas to become leaders.

### Minimum In-Sync Replicas

Both the topic and the broker-level configuration are called `min.insync.replicas`.

This configuration specifies the minimum number of replicas that must acknowledge a write for the write to be considered successful.
If a topic has three replicas and we set `min.insync.replicas` to 2, then producers can only write to a partition in the topic if at least two out of the three replicas are in sync

When all three replicas are in sync, everything proceeds normally. This is also true if one of the replicas becomes unavailable. However, if two out of three replicas are not available, the brokers will no longer accept produce requests. Instead, producers that attempt to send data will receive `NotEnoughReplicasException`

## Producer

Even if we configure the brokers in the most reliable configuration possible, the system as a whole can still potentially lose data if we don’t configure the producers to be reliable as well.

Here are two example scenarios to demonstrate this:

- We configured the brokers with three replicas, and unclean leader election is disabled. However, we configured the producer to send messages with acks=1.
- We configured the brokers with three replicas, and unclean leader election is disabled. However, we configured the producer to send messages with acks=all. Suppose that we are attempting to write a message to Kafka, but the leader for the partition we are writing to just crashed. Kafka will respond with `Leader not Available`. At this point, if the producer doesn’t handle the error correctly and doesn’t retry until the write is successful, the message may be lost.

### Send Acknowledgments

Producers can choose between three different acknowledgment modes:

- ack=0
- ack=1
- ack=all

### Configuring Producer Retries

In general, when our goal is to never lose a message, our best approach is to configure the producer to keep trying to send the messages when it encounters a retriable error.

Retrying to send a failed message includes a risk that both messages were successfully written to the broker, leading to duplicates. Retries and careful error handling can guarantee that each message will be stored at least once, but not exactly once

## Consumer

There are four consumer configuration properties that are important to understand in order to configure our consumer for a desired reliability behavior

- `group.id`
- `auto.offset.reset`: This parameter controls what the consumer will do when no offsets were committed (e.g., when the consumer first starts) or when the consumer asks for offsets that don’t exist in the broker. There are only two options here. If we choose `earliest`, the consumer will start from the beginning of the partition whenever it doesn’t have a valid offset. If we choose `latest`, the consumer will start at the end of the partition.
- `enable.auto.commit`
- `auto.commit.interval.ms`
