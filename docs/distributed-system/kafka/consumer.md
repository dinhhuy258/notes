# Kafka consumer

Kafka producer is a component of the Kafka ecosystem which is used to pull messages from a Kafka topic.

![](https://user-images.githubusercontent.com/17776979/193178397-667469d8-d76c-493e-b088-bc81ae03d8b1.png)

A consumer always reads data from a lower offset to a higher offset and cannot read data backwards.

If the consumer consumes data from more than one partition, the message order is not guaranteed across multiple partitions because they are consumed simultaneously, but the message read order is still guaranteed within each individual partition.

## Kafka Consumer Groups

Kafka consumers are typically part of a consumer group.

When multiple consumers are subscribed to a topic and belong to the same consumer group, each consumer in the group will receive messages from a different subset of the partitions in the topic.

For example: Let’s take topic `T1` with four partitions. Now suppose we created a new consumer, `C1`, which is the only consumer in group `G1`, and use it to subscribe to topic `T1`. Consumer `C1` will get all messages from all four `T1` partitions.

![](https://user-images.githubusercontent.com/17776979/193178220-d4421709-549a-47a1-8477-677742107336.png)

If we add another consumer, `C2`, to group `G1`, each consumer will only get messages from two partitions. Perhaps messages from partition `0` and `2` go to `C1`, and messages from partitions `1` and `3` go to consumer `C2`

![](https://user-images.githubusercontent.com/17776979/193178264-45b5a949-6668-4b90-a238-28630e25cb71.png)

**Note**: If we add more consumers to a single group with a single topic than we have partitions, some of the consumers will be idle and get no messages at all

## Kafka Consumer Offsets

Kafka brokers use an internal topic named `__consumer_offsets` that keeps track of what messages a given consumer group last successfully processed.

As we know, each message in a Kafka topic has a partition ID and an offset ID attached to it.

Therefore, in order to checkpoint how far a consumer has been reading into a topic partition, the consumer will regularly **commit** the latest processed message, also known as **consumer offset**.

### Automatic Commit

The easiest way to commit offsets is to allow the consumer to do it for you. If you configure `enable.auto.commit=true`, then every `auto.commit.interval.ms` the consumer will commit the latest offset that your client received from `poll()`

This method may lead to duplicate message processing.

### Commit Current Offset

By setting `enable.auto.commit=false`, offsets will only be committed when the application explicitly chooses to do so.

There are 2 ways to commit the current offset:

- Synchronous: `commitSync()`
- Asynchronous: `commitAsync`

It is important to remember that `commitSync()` and `commitAsync` will commit the latest offset returned by `poll()`, so if you call `commitSync()` or `commitAsync()` before you are done processing all the records in the collection, you risk missing the messages that were committed but not processed, in case the application crashes.

**Note**: You can commit a specific offset in Kafka
