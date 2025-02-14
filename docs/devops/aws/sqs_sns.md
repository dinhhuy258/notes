# SQS/SNS

## SQS

Amazon Simple Queue Service (SQS) is a message queuing service that can help us decouple our application components. It uses queues to store the messages sent by producers and sends them to consumers using a polling model where consumers check the queues for messages sent by the producer whenever they are ready to process them.

### Standard Queue

A Standard Queue in Amazon SQS provides a flexible messaging service with the following key characteristics:

- **Unlimited Throughput**: There is no limit on the number of messages or the rate at which messages can be sent or received.
- **Message Retention**: Default retention period: 4 days.
- **Maximum retention period**: 14 days.
- **Low Latency**: Standard queues offer low latency for message delivery, typically in the millisecond range.
- **Message Size Limitation**: Each message can be up to 256 KB in size.
- **At-Least-Once Delivery**: Messages are guaranteed to be delivered at least once, but duplicates may occur.
- **Out-of-Order Delivery**: Messages may not always be delivered in the order they were sent.

This queue type is suitable for scenarios where high throughput and availability are critical, and occasional duplicate or out-of-order messages are acceptable.

### FIFO queue

FIFO (First-In-First-Out) queues provide the same core features as standard queues, with added benefits:

- **Strict Message Ordering**: Ensures that messages are sent and received in the exact order they are queued.
- **Exactly-Once Processing**: Guarantees that each message is processed only once, without duplicates.

**Key Characteristics**:

- **Limited Throughput**:
  - 300 messages/second without batching.
  - 3,000 messages/second with batching.
- **Ordering**: Messages are processed in the same sequence in which they are sent.
- **Message Uniqueness**: FIFO queues ensure that duplicate messages are not delivered within a 5-minute deduplication window.

**Deduplication**

FIFO queues prevent duplicate message delivery within a **5-minute deduplication** interval. There are two deduplication methods:

- Content-Based Deduplication: Uses a SHA-256 hash of the message body to automatically detect duplicates.
- Explicit Deduplication: You can explicitly provide a `MessageDeduplicationId` for each message.

> [!NOTE]  
> When using JSON-encoded messages, deduplication may be affected if the ordering of fields in the JSON changes, even though the content remains the same. AWS treats such messages as different because the SHA-256 hash is computed on the raw message body.

**Message Group ID**

The Message Group ID tag indicates if a message belongs to a specific group:

- Messages within the same group are processed one-by-one, preserving strict order.
- Messages in different groups can be processed independently and in parallel, potentially out of order.

Using Message Group IDs allows parallel processing of multiple message groups while still ensuring strict ordering within each group. This improves throughput and scalability by enabling more efficient use of resources, as different consumers can handle different message groups concurrently.

If no Message Group ID is provided, all messages are handled in the order they are sent, with only one consumer processing them.

![](https://user-images.githubusercontent.com/17776979/203588989-9a881daf-584c-4023-9f2f-d81b17e639ea.png)

## Features of SQS

### Polling

Polling is the process through which messages are received from a SQS queue. In this process, the consumers query the queue, whenever they are available, in order to receive messages from the queue. Amazon SQS supports short and long polling. These models provide different trade-offs in terms of latency, throughput, and cost, allowing us to choose the model that best fits our application’s requirements.

**Short polling**

In this method, only a **subset of the servers** (based on a weighted random distribution) are queried when the SQS queue receives a message retrieval request. SQS sends an empty response immediately, in case it doesn’t find any messages in the queue. Due to this, there is a chance we might not receive messages using short polling. However, if our queue contains less than 1000 messages, we’ll be able to receive these messages with short polling. Also, if we keep sending message retrieval requests to our queue, all available servers are queried to send back a response.

![imgur.png](https://i.imgur.com/Me9hNEV.png)

In the above diagram, only the `S1`, `S3`, and `S5` servers are queried during short polling. Due to this, the message `d` is not delivered to the consumer.

Short polling is useful in applications that have low latency requirements and send frequent polling requests.

**Long pooling**

In long polling, **all available servers** are queried for messages, and a response is sent only after SQS finds at least one message to deliver to the consumer who made the fetch request, and an empty response is sent only in case our request times out.

By using long polling, we can reduce the number of empty responses sent by SQS. We can also reduce the false empty responses by querying all servers rather than a subset of servers
Since we are charged on every polling request made to the SQS queue, long polling is preferred to reduce cost by reducing the number of empty responses received. However, it is not beneficial to use long polling in case our application expects an immediate response from the SQS queue. In this case, short polling will be prefered.

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

![imgur.png](https://i.imgur.com/XO1A87c.png)

### Delay queue

Delay queues allow you to postpone the delivery of new messages to consumers for a specified number of seconds. This can be useful when your consumer application requires extra time to process messages or manage workflows more effectively.

**How Delay Queues Work**

When a delay queue is configured, any messages sent to the queue remain invisible to consumers for the duration of the specified delay period.

- Minimum delay: 0 seconds (no delay).
- Maximum delay: 15 minutes.

**Configuring Delay Queues**

The delay period for a queue can be set using the `Delivery delay` parameter, which applies to all messages in the queue.

**Individual Message Timers**

If you want to set a delay for individual messages (instead of applying a queue-wide delay), you can use message timers. With message timers, Amazon SQS uses the message’s `DelaySeconds` value instead of the queue's default delay.

## SNS

Amazon SNS is a messaging service that delivers messages from a publisher to one or more subscribers using a publish-subscribe (pub/sub) model.

- The producer sends messages to an SNS topic
- Multiple subscribers can listen to messages from the same topic.
- Maximum subscribers per topic: 12,500,000
- Topic limit per account: 100,000

### Standard topics

- Messages are delivered at least once, but duplicates can occur.
- There is no guarantee of message order.
- Duplicate messages may happen if the same message is published multiple times.

### FIFO topics

- Exactly-once delivery: Ensures each message is delivered only once to subscribers.
- Strict ordering: Messages are delivered in the same order they are received.
- Uses a deduplication ID to eliminate duplicates within a 5-minute window.

### Subscriber endpoints in Amazon SNS

Amazon SNS supports different types of endpoints for delivering messages, enabling both:

- Application-to-Application (A2A) messaging (e.g., AWS Lambda, SQS, HTTP/S endpoints).
- Application-to-Person (A2P) messaging (e.g., SMS, email, mobile push notifications).

You can apply filter policies to control which messages are sent to each subscriber based on message content.

![imgur.png](https://i.imgur.com/j4xo23R.png)

### Message filtering

By default, Amazon SNS forwards all messages published to a topic, to the topic’s subscribers. However, we can provide filter policies to ensure only a subset of the messages being published are sent to a subscriber.

![](https://user-images.githubusercontent.com/17776979/203591498-6cb5393f-f13f-4592-bf65-f7847f8eb626.png)

### Fanout Pattern

The fanout pattern sends a single message to multiple subscribers at the same time, enabling asynchronous processing.

![](https://user-images.githubusercontent.com/17776979/203590506-f8cca5b8-8389-4d1d-bc74-bd678d6eec36.png)
