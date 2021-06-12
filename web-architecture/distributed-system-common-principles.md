---
title: Distributed System - Common Principles
parent: Web architecture
nav_order: 1
---

# Distributed System - Common Principles

## 1) Tier

### 1.1) Single tier application

A single-tier application is an application where the user interface, backend business logic and database all reside in the same machine.

An example of an application with single-tier are applications such as MS Office, GIMP..

![](../assets/images/web-architecture/single-tier-application.png)

### 1.2) Two tier application

A two-tier application involves a client and a server. The client would contain the user interface and business logic in one machine and the server would be the database running on a different machine.

![](../assets/images/web-architecture/two-tier-application.png)

### 1.3) Three tier application

In three-tier application the user interface, application logic and the database all lie on different machine and thus have different tiers. They are physically separated.

![](../assets/images/web-architecture/three-tier-application.png)

### 1.4) N tier application

An N-tier application is an application which has more than three components involved.

These components can be:

- Cache
- Message queues
- Load balancers
- Search server
- ...

## 2) CAP Theorem

In a distributed computer system, you can only support two of the following guarantees:

- **Consistency:** means that all clients see the same data at the same time, no matter which node they connect to. For this to happen, whenever data is written to one node, it must be instantly forwarded or replicated to all the other nodes in the system before the write is deemed `successful`
- **Availability:** means that every request gets a (non-error) response regardless of the individual state of a node. This does not guarantee that the response contains the most recent write
- **Partition Tolerance:** means that the cluster must continue to work despite any number of communication breakdowns between nodes in the system

![](../assets/images/web-architecture/cap-theorem.png)

Networks aren't reliable, so you'll need to support partition tolerance (the P of CAP). That leaves a decision between the other two, C and A. When a network failure happens, one can choose to guarantee **consistency** or **availability**

- **AP**: When availability is chosen over consistency, the system is will always process the client request and try to return the most recent available version of the information even if it cannot guarantee it is up to date due to network partitioning. AP is a good choice if the business needs allow for eventual consistency or when the system needs to continue working despite external errors. AP is a good choice if the business needs allow for eventual consistency or when the system needs to continue working despite external errors.
- **CP**: When consistency is chosen over availability, the system will return an error or time-out if particular information cannot be updated to other nodes due to network partition or failures. CP is a good choice if your business needs require atomic reads and writes.

Database system designed with **ACID** guarantees (RDBMS) usually chooses consistency over availability whereas system Designed with **BASE** guarantees, chooses availability over consistency.
