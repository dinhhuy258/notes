# ZooKeeper

ZooKeeper is a centralized service for providing **configuration information**, **naming**, **synchronization** and **group services** over large clusters in distributed systems.

## Use cases

**Naming service**: maps the names of network resources to their respective network addresses

**Configuration management**: Managing application configuration that can be shared across servers in a cluster. The idea is to maintain any configuration in a centralized place so that all servers will see any change in configuration files/data.

**Leader election**: Electing a leader in a multi-node cluster. You might need a leader to maintain a single point for an update request or distributing tasks from leader to worker nodes.

**Locks in distributed systems**: distributed locks enables different systems to operate on a shared resource in a mutually exclusive way.

**Manage cluster membership**: Maintain and detect if any server leaves or joins a cluster and store other complex information of a cluster.

## ZooKeeper architecture

![](https://user-images.githubusercontent.com/17776979/192682683-9e2101f6-2326-475d-83a1-63bc4f65ed29.png)

**Client**: Clients, one of the nodes in our distributed application cluster access information from the server. For a particular time interval, every client sends a message to the server to let the sever know that the client is alive.

**Server**: Server, one of the nodes in our ZooKeeper ensemble, provides all the services to clients. Gives acknowledgement to client to inform that the server is alive.

**Ensemble**: Group of ZooKeeper servers. The minimum number of nodes that is required to form an ensemble is 3.

**Leader**: Server node which performs automatic recovery if any of the connected node failed. Leaders are elected on service startup.

**Follower**: Server node which follows leader instruction.

## Data model

Zookeeper keeps internal data structure which is called **znode**. Znodes are in shape of directory. **znode** can contain either data and subnodes (each znode can store upto 1MB of data)

![](https://user-images.githubusercontent.com/17776979/192684369-7079b0db-e3ba-48fc-920e-e5116fd9891c.png)

Every znode in the ZooKeeper data model maintains a stat structure. A stat simply provides the metadata of a znode

**Version number** Every znode has a version number, which means every time the data associated with the znode changes, its corresponding version number would also increased. The use of version number is important when multiple zookeeper clients are trying to perform operations over the same znode.

**Action Control List (ACL)** ACL is basically an authentication mechanism for accessing the znode. It governs all the znode read and write operations.

**Timestamp** Timestamp represents time elapsed from znode creation and modification. It is usually represented in milliseconds. ZooKeeper identifies every change to the znodes from “Transaction ID” (zxid). Zxid is unique and maintains time for each transaction so that you can easily identify the time elapsed from one request to another request.

**Data length** Total amount of the data stored in a znode is the data length. You can store a maximum of 1MB of data.

## Types of Znodes

### Persistence znodes

Persistence znode is alive even after the client, which created that particular znode, is disconnected. By default, all znodes are persistent unless otherwise specified.

**Example**: Solr Cloud, uses these znodes to store server configuration and schema of database/collections.

### Ephemeral znode

Ephemeral znodes are active until the client is alive. When a client gets disconnected from the ZooKeeper ensemble, then the ephemeral znodes get deleted automatically. For this reason, only ephemeral znodes are not allowed to have a children further.

It is created using -e flag

**Example**: Let’s say you want to maintain a list of active servers in a cluster. So, you create a parent Znode `/live_servers`. Under it, you keep creating child Znode for every new server in the cluster. At any point, if a server crashes/dies, child Znode belonging to the respective server will be deleted. Other servers will get a notification of this deletion if they are watching the znode `\live_servers`.

### Ephemeral Sequential Znode

It is the same as ephemeral Znode, the only difference is Zookeeper attaches a sequential number as a suffix, and if any new sibling Znode of the same type is created, it will be assigned a number higher than the previous one.

This type of znode is created using -e -s flag.

For example, if a znode with path /myapp is created as a sequential znode, ZooKeeper will change the path to /myapp0000000001 and set the next sequence number as 0000000002.

This type of znode could be used in the leader election algorithm.

**Example**:

Say I have a parent node `/election`, and for any new node that joins the cluster, I add an ephemeral sequential Znode to this `/election` node. We can consider a server as the leader if any server that created the znode has the least sequential number attached to it.

So, even if a leader goes down, the zookeeper will delete the corresponding Znode created by the leader server and notify the client applications, then that client fetches the new lowermost sequence node and considers that as a new leader

### Persistent Sequential Znode

This is a persistent node with a sequence number attached to its name as a suffix.

## Sessions

Sessions are very important for the operation of ZooKeeper. Requests in a session are executed in FIFO order. Once a client connects to a server, the session will be established and a session id is assigned to the client.

The client sends heartbeats at a particular time interval to keep the session valid. If the ZooKeeper ensemble does not receive heartbeats from a client for more than the period (session timeout) specified at the starting of the service, it decides that the client died.

Session timeouts are usually represented in milliseconds. When a session ends for any reason, the ephemeral znodes created during that session also get deleted.

## Watches

Watches are a simple mechanism for the client to get notifications about the changes in the ZooKeeper ensemble. Clients can set watches while reading a particular znode. Watches send a notification to the registered client for any of the znode (on which client registers) changes.

Znode changes are modification of data associated with the znode or changes in the znode’s children.
