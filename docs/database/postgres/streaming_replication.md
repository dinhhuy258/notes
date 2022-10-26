# Streaming Replication

Streaming replication, a standard feature of PostgreSQL, allows the updated information on the primary server to be transferred to the standby server in real time, so that the databases of the primary server and standby server can be kept in sync.

## Streaming replication mechanism

### What is shipped

PostgreSQL saves the updated information of the primary server as a transaction log known as write-ahead log, or WAL, in preparation for crash recovery or rollback. Streaming replication works by transferring, or shipping, the WAL to the standby server in real time, and applying it on the standby server.

![](https://user-images.githubusercontent.com/17776979/198082725-f4303de3-37f8-422c-8f24-6922e620955b.png)

### How the WAL is shipped and applied

WAL shipping between the primary server and the standby server is performed by the **WAL sender** process on the primary server to the **WAL receiver** process on the standby server. These processes are started by setting `postgresql.conf` and `pg_hba.conf` parameters

![](https://user-images.githubusercontent.com/17776979/198082906-61f0dcfe-951e-410a-90a8-f08cb8ec1271.png)

A walsender and a walreceiver communicate using a single TCP connection.

The `pg_stat_replication` view shows the state of all running walsenders. An example is shown below:

```bash
testdb=# SELECT application_name,state FROM pg_stat_replication;
 application_name |   state
------------------+-----------
 standby1         | streaming
 standby2         | streaming
 pg_basebackup    | backup
(3 rows)
```

## Setup

### Multi-standby setup and cascade setup

Streaming replication can be built in a 1:N configuration, where only one primary server is configurable, but multiple standby servers can be set up.

The configuration that connects (ships WAL from) a primary server to all standby servers is called a multi-standby setup.

![](https://user-images.githubusercontent.com/17776979/198083204-82e810dd-07de-4324-bb35-40c0e71bfe58.png)

You can also build a cascade setup where a standby server connects (ships WAL) to another standby server.

![](https://user-images.githubusercontent.com/17776979/198083224-f337633c-9ba5-4025-9bfc-3a992be9ab30.png)

### Synchronous replication and asynchronous replication

For streaming replication, you can select either synchronous or asynchronous replication for each standby server.

**Synchronous replication**

- The primary server waits for a response from the standby server before completing a process.
  Therefore, the overall response time includes the log shipping time.
- Since there is no delay in WAL shipping to the standby server, the data freshness (reliability) of the standby server is improved.
- Suitable for failover and read-only load balancing operations.

**Asynchronous replication**

- The primary server completes a process without waiting for a response from the standby server. Therefore, the overall response time is about the same as when streaming replication is not used.
- Since WAL shipping and its application (data update) on the standby server are done asynchronously, the updated result on the primary server may not be immediately available on the standby server. Depending on the timing of failover, data may be lost.
- Suitable for replication to remote areas for disaster recovery.

### Synchronous/asynchronous setup

Synchronous setup is configured by synchronous_standby_names in `postgresql.conf` on the primary server. If there are multiple standby servers, you can specify the servers to be synchronized and the order of priority for COMMIT. Standby servers not specified in this parameter will be asynchronous.

You can also specify the method by which to select synchronous standby servers from a list of servers, such as `FIRST n (list)` or `ANY n (list)`

The `FIRST` keyword specifies that the `n` first standby servers in list will be synchronous and that transaction commits will **wait** until their WAL records are replicated to the first n standby servers in list.

The `ANY` keyword specifies a quorum-based synchronous replication and that transaction commits should **wait** until their WAL records are replicated to at least `n` standby servers in list.

For example, in an environment using standby servers `s1`, `s2`, `s3`, and `s4`, setting `synchronous_standby_names` to `FIRST 2 (s1, s2, s3')` will configure `s1` and `s2` for **synchronous** replication, and `s3` and `s4` for **asynchronous** replication. The primary server will wait `s1` and `s2` to complete processing before it commits. If `s1` or `s2` fails, then `s3` changes to synchronous replication.

Make sure that the name set in `synchronous_standby_names` matches the name set in `application_name` of `primary_conninfo` in `postgresql.conf` on the standby server.

### Setting the synchronization level

To set the synchronization level of the standby server, set `synchronous_commit` in `postgresql.conf` on the primary server.

| Sync Level       | Set Value    | Overview                                                                                                                                                                                                                                                                                      | Guaranteed range |
| ---------------- | ------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- |
| Full synchronous | remote_apply | Commits wait until after the WAL is replicated to the standby servers and the updated data on the standby server is available to read-only queries. It is fully assured that the data is in sync, so it is well suited for load balancing of read-only workloads that require data freshness. | 1 to 9           |
| Synchronous      | on (default) | Commits wait until after the WAL is replicated to the standby servers. This setting achieves the best balance between performance and reliability.                                                                                                                                            | 1 to 6           |
| Semi-synchronous | remote_write | Commits wait until after WAL has been shipped to the standby servers.                                                                                                                                                                                                                         | 1 to 5           |
| Asynchronous     | local        | Commits wait until after the WAL write on the primary server is completed.                                                                                                                                                                                                                    | 1 to 2           |
| Asynchronous     | off          | Commits do not wait for the WAL write on the primary server to complete. This setting is not recommended.                                                                                                                                                                                     | 1 to 1           |

![](https://user-images.githubusercontent.com/17776979/198086492-4a24474a-cef4-4e39-8c4c-e4b63afcf025.png) 
