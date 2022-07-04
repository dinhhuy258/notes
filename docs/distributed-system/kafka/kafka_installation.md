# Kafka installation

## Installing ZooKeeper

Apache Kafka uses Apache ZooKeeper to store metadata about the Kafka cluster, as well as consumer client details.

![](../../assets/images/distributed-system/kafka/kafka_and_zookeeper.png)

We can download and install ZooKeeper from this [site](https://zookeeper.apache.org/releases.html). We can also using the docker image from [here](https://github.com/confluentinc/cp-docker-images/wiki/Getting-Started)

### Standalone ZooKeeper configuration

```
tickTime=2000
dataDir=/var/zookeeper
clientPort=2181
```

**tickTime:** the basic time unit in milliseconds used by ZooKeeper
**dataDir:** the location to store the in-memory database snapshots
**clientPort:** the port to listen for client connections

### ZooKeeper ensemble configuration

ZooKeeper is designed to work as a cluster, called an ensemble, to ensure high availability.

To configure ZooKeeper servers in an ensemble, they must have a common configuration that lists all servers, and each server needs a myid file in the data directory that specifies the ID number of the server

If the hostnames of the servers in the ensemble are `zoo1.example.com`, `zoo2.example.com`, and `zoo3.example.com`, the configuration file might look like this:

```
tickTime=2000
dataDir=/var/lib/zookeeper
clientPort=2181
initLimit=20
syncLimit=5
server.1=zoo1.example.com:2888:3888
server.2=zoo2.example.com:2888:3888
server.3=zoo3.example.com:2888:3888
```

**initLimit:** is the amount of time to allow followers to connect with a leader
**syncLimit:** limits how long out-of-sync followers can be with the leader.

Both values are a number of `tickTime` units, which makes the `initLimit` 20 × 2,000 ms, or 40 seconds.

The configuration also lists each server in the ensemble. The servers are specified in the format `server.X=hostname:peerPort:leaderPort`, with the following parameters:

**X:** The ID number of the server. This must be an integer.
**hostname:** The hostname or IP address of the server
**peerPort:** The TCP port over which servers in the ensemble communicate with one another.
**leaderPort:** The TCP port over which leader election is performed.

Clients only need to be able to connect to the ensemble over the `clientPort`, but the members of the ensemble must be able to communicate with one another over all three ports

## Installing a Kafka Broker

We can download and install Kafka from this [site](https://kafka.apache.org/downloads.html). We can also using the docker image from [here](https://github.com/confluentinc/cp-docker-images/wiki/Getting-Started)

### Configuring the Broker

**broker.id**

Every Kafka broker must have an integer identifier, which is set using the `broker.id` configuration.

**zookeeper.connect**

The location of the ZooKeeper used for storing the broker metadata is set using the `zookeeper.connect` configuration parameter

**log.dirs**

Kafka persists all messages to disk, and these log segments are stored in the directory specified in the `log.dir` configuration
For multiple directories, the config `log.dirs` is preferable, `log.dirs` is a comma-separated list of paths on the local system.
If more than one path is specified, the broker will store partitions on them in a **least-used** fashion, with one partition’s log segments stored within the same path.
Note that the broker will place a new partition in the path that has the least number of partitions currently stored in it, not the least amount of disk space used, so an even distribution of data across multiple directories is not guaranteed.

**num.recovery.threads.per.data.dir**

The number of threads per data directory to be used for log recovery at startup and flushing at shutdown

By default, only **one** thread per log directory is used. As these threads are only used during startup and shutdown, it is reasonable to set a larger number of threads in order to parallelize operations.

Default: 1

Please refer this [site](https://docs.confluent.io/platform/current/installation/configuration/broker-configs.html) for more broker config
