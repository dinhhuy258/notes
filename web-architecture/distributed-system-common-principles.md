---
title: Distributed System
parent: Web architecture
nav_order: 1
---

# Distributed System

## 1) CAP Theorem

In a distributed computer system, you can only support two of the following guarantees:

- **Consistency:** means that all clients see the same data at the same time, no matter which node they connect to. For this to happen, whenever data is written to one node, it must be instantly forwarded or replicated to all the other nodes in the system before the write is deemed `successful`
- **Availability:** means that every request gets a (non-error) response regardless of the individual state of a node. This does not guarantee that the response contains the most recent write
- **Partition Tolerance:** means that the cluster must continue to work despite any number of communication breakdowns between nodes in the system

![](../assets/images/web-architecture/cap-theorem.png)

Networks aren't reliable, so you'll need to support partition tolerance (the P of CAP). That leaves a decision between the other two, C and A. When a network failure happens, one can choose to guarantee **consistency** or **availability**

- **AP**: When availability is chosen over consistency, the system is will always process the client request and try to return the most recent available version of the information even if it cannot guarantee it is up to date due to network partitioning. AP is a good choice if the business needs allow for eventual consistency or when the system needs to continue working despite external errors. AP is a good choice if the business needs allow for eventual consistency or when the system needs to continue working despite external errors.
- **CP**: When consistency is chosen over availability, the system will return an error or time-out if particular information cannot be updated to other nodes due to network partition or failures. CP is a good choice if your business needs require atomic reads and writes.

Database system designed with **ACID** guarantees (RDBMS) usually chooses consistency over availability whereas system Designed with **BASE** guarantees, chooses availability over consistency.

## 2) Trade-offs

### 2.1) Performance vs scalability

A service is **scalable** if it results in increased performance in a manner proportional to the resources added.
Generally, increasing performance means serving more units of work, but it can also be to handle larger units of work, such as when datasets grow.

Another way to look at performance vs scalability:

- If you have a **performance** problem, your system is slow for a single user.
- If you have a **scalability** problem, your system is fast for a single user but slow under heavy load.

A system has to sacrifice performance for scalability.

For example you have a stock exchange engine running on an API.
At first you run it in one location and the performance is fast.
But then you start getting customers at the other end of the world and their latency numbers are bad.
If you choose to spin up and api instance that is geographically closer to those customers (scale), the latency is shortened for them but now your apis must synchronize between instances.
This synchronization will cost you time and extra computation steps, lowering your performance.

[http://highscalability.com/blog/2011/2/10/database-isolation-levels-and-their-effects-on-performance-a.html](http://highscalability.com/blog/2011/2/10/database-isolation-levels-and-their-effects-on-performance-a.html)

### 2.2) Latency vs throughput

- **Latency:** is the time to perform some action or produce some result

  - Network latency
  - Processing latency

- **Throughput:** is the number of such actions executed or results produced per unit of time (number of requests that servers can handle in the certain time)

Generally, you should aim for **maximal throughput** with **acceptable latency**.

For example:

An assembly line is manufacturing cars. It takes 8 hours to manufacture a car and that the factory produces 120 cars per day.

- The latency is: 8 hours.
- The throughput is: 120 cars / day or 5 cars / hour.

### 2.3) Availability vs consistency

Networks aren’t reliable, so you’ll need to support partition tolerance. According to **CAP Theorem**, you’ll need to make a software tradeoff between consistency and availability.

- **Consistency:** Every read receives the most recent write or an error
- **Availability:** Every request receives a response, without guarantee that it contains the most recent version of the information
