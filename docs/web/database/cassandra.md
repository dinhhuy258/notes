# Cassandra

## 1. Cassandra Architecture

Some of the features of Cassandra architecture are as follows:

- Cassandra is designed such that it has no master or slave nodes.
- It has a ring-type architecture, that is, its nodes are logically distributed like a ring.
- Data is automatically distributed across all the nodes.
- Similar to HDFS, data is replicated across the nodes for redundancy.
- Data is kept in memory and lazily written to the disk.
- Hash values of the keys are used to distribute the data among nodes in the cluster.

![](../../assets/images/database/cassandra_architecture_1.png)

Additional features of Cassandra architecture are:

- Cassandra architecture supports multiple data centers.
- Data can be replicated across data centers.

You can keep three copies of data in one data center and the fourth copy in a remote data center for remote backup. Data reads prefer a local data center to a remote data center.

![](../../assets/images/database/cassandra_architecture_2.png)

### Node

Node is the place where data is stored. It is the basic component of Cassandra.

### Rack

A rack is a group of machines housed in the same physical box.

![](../../assets/images/database/cassandra_rack.png)

- All machines in the rack are connected to the network switch of the rack
- The rack’s network switch is connected to the cluster.
- All machines on the rack have a common power supply. It is important to notice that a rack can fail due to two reasons: a network switch failure or a power supply failure.
- If a rack fails, none of the machines on the rack can be accessed. So it would seem as though all the nodes on the rack are down.

### Datacenter

A datacenter is a logical set of racks/ nodes. A common use case is AWS-EAST vs AWS-WEST...

### Cluster

The cluster is the collection of many data centers.

![](../../assets/images/database/cassandra_architecture_3.png)

## 2. Data distribution and replication

### 2.1 Data Partitions

A **partition key** is converted to a **token** by a **partitioner**. The tokens are signed integer values between -2^63 to +2^63-1, and this range is referred to as token range.

If we consider there are only 100 tokens used for a Cassandra cluster with three nodes. Each node is assigned approximately 33 tokens like

```
node1: 0-33
node2: 34-66
node3: 67-99
```

If there are nodes added or removed, the token range distribution should be shuffled to suit the new topology. This process takes a lot of calculation and configuration change for each cluster operation.

![](../../assets/images/database/cassandra_token_ring.png)

If one node is removed, data in removed node is placed on the next neighbor node in clockwise manner.

![](../../assets/images/database/cassandra_token_ring_2.png)

### 2.2 Virtual nodes/Vnodes

Virtual nodes in a Cassandra cluster are also called vnodes. Vnodes can be defined for each physical node in the cluster. Each node in the ring can hold multiple virtual nodes.

The default number of Vnodes owned by a node in Cassandra is `256`, which is set by `num_tokens` property. When a node is added into a cluster, the token allocation algorithm allocates tokens to the node. The algorithm selects random token values to ensure uniform distribution.

In your case you have 6 nodes, each set with 256 token ranges so you have 6\*256 token ranges and each psychical node contains 256 token ranges.

![](../../assets/images/database/cassandra_virtual_node.png)

### 2.3 Replication

The data in each keyspace is replicated with a **replication factor**. There is one primary replica of data that resides with the token owner node as explained in the data partitioning section. The remainder of replicas is placed by Cassandra on specific nodes using the replica placement strategy.

The total number of replicas for a keyspace across a Cassandra cluster is referred to as the keyspace's replication factor. A replication factor of one means that there is only one copy of each row in the Cassandra cluster. A replication factor of two means there are two copies of each row, where each copy is on a different node. All replicas are equally important; there is no primary or master replica.

There are two settings that mainly impact replica placement:

- First is snitch, which determines the data center, and the rack a Cassandra node belongs to, and it is set at the node level
- The second setting is the replication strategy. The replication strategy is set at the keyspace level. There are two strategies: SimpleStrategy and NetworkTopologyStrategy.

SimpleStrategy: does not consider racks and multiple data centers. It places data replicas on nodes sequentially.
NetworkTopologyStrategy: is rack aware and data center aware

![](../../assets/images/database/cassandra_replication.png)

### 2.4 Consistency level

The Cassandra consistency level is defined as the minimum number of Cassandra nodes that must acknowledge a read or write operation before the operation can be considered successful. Different consistency levels can be assigned to different Edge keyspaces.

You can find all cassandra's consistency level [here](https://docs.scylladb.com/getting-started/consistency/#consistency-levels-reference)

![](../../assets/images/database/cassandra_consistency.png)

**Write Consistency**

1. A client sends a write request to the coordinator.
2. The coordinator forwards the write request (INSERT, UPDATE or DELETE) to all replica nodes whatever write CL you have set.
3. The coordinator waits for n number of replica nodes to respond. n is set by the write CL.
4. The coordinator sends the response back to the client.

**Read Consistency**

1. A client sends a read request to the coordinator.
2. The coordinator forwards the read (SELECT) request to n number of replica nodes. n is set by the read CL.
3. The coordinator waits for n number of replica nodes to respond.
4. The coordinator then merges (finds out most recent copy of written data) the n number of responses to a single response and sends response to the client.
