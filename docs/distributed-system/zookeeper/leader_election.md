# Leader Election

Let us analyze how a leader node can be elected in a ZooKeeper ensemble. Consider there are N number of nodes in a cluster. The process of leader election is as follows

- All the nodes create a sequential, ephemeral znode with the same path, `/app/leader_election/guid_`.
- ZooKeeper ensemble will append the 10-digit sequence number to the path and the znode created will be `/app/leader_election/guid_0000000001`, `/app/leader_election/guid_0000000002`, etc.
- For a given instance, the node which creates the **smallest number** in the znode becomes the leader and all the other nodes are followers.
- Each follower node watches the znode having the next smallest number. For example, the node which creates znode `/app/leader_election/guid_0000000008` will watch the znode `/app/leader_election/guid_0000000007` and the node which creates the znode `/app/leader_election/guid_0000000007` will watch the znode `/app/leader_election/guid_0000000006`.
- If the leader goes down, then its corresponding znode `/app/leader_electionN` gets deleted.
- The next in line follower node will get the notification through watcher about the leader removal.
- The next in line follower node will check if there are other znodes with the smallest number. If none, then it will assume the role of the leader. Otherwise, it finds the node which created the znode with the smallest number as leader.
- Similarly, all other follower nodes elect the node which created the znode with the smallest number as leader.
