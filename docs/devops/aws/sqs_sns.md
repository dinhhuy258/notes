# SQS/SNS

## SQS

### Standard Queue

- Unlimited throughput, unlimited number of messages in queue
- Default retention of messages: 4 days, maximum of 14 days
- Low latency 
- Limitation of 256KB per message sent
- Can have duplicate message (at least once delivery)
- Can have out of orders messages

### Visibility timeout

When the producer sends a message to SQS, it is stored in the queue until consumed by a consumer. When the consumer is ready, it polls SQS for new messages and ultimately receives the message.

Once a message is received by a consumer, SQS doesn’t automatically delete the message. Because there’s no way for SQS to guarantee that the message has been received by the consumer. The message might get lost in transit or the consumer can fail while processing the message.

So the consumer must delete the message from the queue after receiving and processing it.

While a consumer is processing a message in the queue, SQS temporary hides the message from other consumers. This is done by setting a visibility timeout on the message, a period of time during which SQS prevents other consumers from receiving and processing the message.

The default visibility timeout for a message is `30` seconds.

![](https://user-images.githubusercontent.com/17776979/203337084-164448d2-e8ac-470b-b8f8-02f9564582fc.png) 

If the message is not processed within the visibility timeout, it will be processed twice. If visibility timeout is high, and consumer crashes, re-processing will take time; if visibility timeout is low, we may get duplicates

### Dead Letter Queue

Dead letter queue is a very useful feature in message queuing systems. By using this feature, we can automatically transfer messages, that have exceeded the maximum number of receiving message, to the dead letter queue.

We have a configuration `Maximum Receives` means the maximum number of retries effectively if the number of retries exceed `Maximum Receives` value then the message will be sent to `Dead Letter Queue`

### Delay queue

- Can be configured via `Delivery delay` configuration
- Delay a message (consumer don't see it immediately) up to 15 minutes
- Default is `0`
- Can override the default on send using the `DelaySeconds` in parameter

### Long pooling

- When a consumer requests messages from SQS, it can optionally `wait` for messages to arrive if there are none in the queue (this is long pooling).
- The wait time can be between 1 to 20 seconds
- Can configure in queue level via `Receive message wait time`
- Can enable in API level using `WaitTimeSeconds`

### FIFO queue

FIFO queues have essentially the same features as standard queues, but provide the added benefits of supporting **ordering** and **exactly-once** processing and ensure that the order in which messages are sent and received is strictly preserved.

- Limited throughput: 300 msg/s without batching, 3000 msg/s with batching
- Exactly once
- Ordering

**Deduplication**

- De-duplication interval is 5 minutes
- Two de-duplication method:

+ Content-based de-duplication (SHA-256 hash of the message body)
+ Explicitly provide a message 

**Message Group ID**

The tag indicates whether or not a message belongs to a particular message group. Messages from the same message group are always handled one by one, in the sequence in which they were received (however, messages that belong to different message groups might be processed out of order).

![](https://user-images.githubusercontent.com/17776979/203588989-9a881daf-584c-4023-9f2f-d81b17e639ea.png) 

For SQS FIFO, if you don't use a Group ID, message are consumed in the order they are sent, **with only one consumer**

## SNS

- Pub/sub pattern
- The producer only sends message to one SNS topic
- Many subscribers can listen to that topic
- Up to 12,500,000 subscribers per topic
- 100,000 topics limit

### Fanout

![](https://user-images.githubusercontent.com/17776979/203590506-f8cca5b8-8389-4d1d-bc74-bd678d6eec36.png) 

### FIFO topic

- Similar features as FIFO queue
- Can only have SQS FIFO queues as subscribers
- Limited throughput (same as FIFO queue)

### Message filtering

JSON policy used to filter messages sent to SNS topic's subscriptions

![](https://user-images.githubusercontent.com/17776979/203591498-6cb5393f-f13f-4592-bf65-f7847f8eb626.png) 
