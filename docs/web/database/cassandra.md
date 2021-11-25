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
