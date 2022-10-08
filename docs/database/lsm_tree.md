# LSM Trees

A log-structured merge-tree (LSM tree) is a data structure typically used when dealing with write-heavy workloads.

The write path is optimized by only performing sequential writes. LSM trees are the core data structure behind many databases, including BigTable, Cassandra, Scylla, and RocksDB.

## Hash Indexes

Hash indexes are basically key-value hash maps.

Let’s say our data storage is just appending to a file, then the simplest possible indexing strategy is this: keep an in-memory hash map where every key is mapped to a byte offset in the data file

![](https://user-images.githubusercontent.com/17776979/194316392-8b364eb8-2dc6-4107-87ba-626161e726e1.png)

Whenever you append a new key-value pair to the file, you also update the hash map to reflect the offset of the data you just wrote (this works both for inserting new keys and for updating existing keys)

When you want to look up a value, use the hash map to **find the offset in the data file**, **seek** to that location and **read** the value.

This may sound simplistic, but it is a viable approach. In fact, this is essentially what Bitcask (the default storage engine in Riak) does. Bitcask offers high-performance reads and writes, subject to the requirement that all the keys fit in the available RAM, since the hash map is kept completely in memory.

A storage engine like Bitcask is well suited to situations where the value for each key is updated frequently. For example, the key might be the URL of a cat video, and the value might be the number of times it has been played (incremented every time someone hits the play button). In this kind of workload, there are a lot of writes, but there are not too many distinct keys—you have a large number of writes per key, but it’s feasible to keep all keys in memory.

There are many optimizations to reclaim needlessly occupied space by log files, for example, we can use multiple log files, so when one log file goes out of capacity, it gets frozen and new writes are done in a new log file. Frozen log files can later be compacted (duplicate keys are removed and only the most recent value of a key is kept).

Each segment now has its own in-memory hash table, mapping keys to file offsets. In order to find the value for a key, we first check the most recent segment’s hash map; if the key is not present we check the second-most-recent segment, and so on. The merging process keeps the number of segments small, so lookups don’t need to check many hash maps.

However, the hash table index also has limitations:

- The hash table must fit in memory, so if you have a very large number of keys, you’re out of luck
- Range queries are not efficient. For example, you cannot easily scan over all keys between kitty00000 and kitty99999 —you’d have to look up each key individually in the hash maps.

## SSTables

As we saw in hash tables, new logs are added to the end of the table, so logs at the end of the table take precedence over logs that came earlier in the table and this is how hash tables detect updates. Other than that, the order of keys in the table is irrelevant. As we explained in the last part of the previous section, hash tables have two major limitations, namely, **having to entirely fit in memory** and **not being able to serve range queries efficiently**.

Now let's consider this, what if we can store **all segments on disk** and only **store a few keys** in memory and build a mechanism that would allow us to use the few keys stored in memory to know which segment(s) to load from disk in search of some key? That would eliminate the first limitation of having to fit all segments in memory.

This is exactly what Sorted Strings Table (SSTables) do, they store all segments on disk and only keep a few keys in memory, but with a minor twist, those keys kept in memory will have to be **sorted**.

![](https://user-images.githubusercontent.com/17776979/194320481-aeb4a9e1-7c5a-4cb8-b79d-f9baeede7a31.png)

In the previous diagram, let's say we are looking for the key `handiwork`, because the keys in the SSTable are sorted, we know it must fall between `handbag` and `handsome`, so we can start from the offset of `handbag` and read all segments until we reach `handsome` to locate the needed key or determine if it doesn't exist at all.

This new structure also helps us to solve the second limitation of not being able to perform range queries efficiently, since we sort keys now.

## Constructing and maintaining SSTables

The main question is how can we construct the SSTable so that all keys are in sorted order. Luckily, we have a data structure can maintain the sorted key: AVL tree or read-black tree

We can now make our storage engine work as follows:

- When a write comes in, add it to an in-memory balanced tree data structure (for example, a red-black tree). This in-memory tree is sometimes called a **memtable**.
- When the memtable gets **bigger than some threshold**, typically a few megabytes, write it out to disk as an **SSTable** file. This can be done efficiently because the tree already maintains the key-value pairs sorted by key.
- In order to serve a read request, first try to find the key in the memtable, then in the most recent on-disk segment, then in the next-older segment, etc.
- From time to time, run a merging and compaction process in the background to combine segment files and to discard overwritten or deleted values.

This scheme looks good so far, the only limitation that's clear now, is what if the database crashes? The recent writes that are still in the memtable but not yet persisted on disk will be lost.

To solve this, most databases maintain what's called a write-ahead-log (WAL) which is a separate file (on disk) to which new writes are written (in the order they come, just appends).

Now if the database crashes, the WAL file can be read and the recent writes that were in the memtable but not on disk can be recovered. In order to keep this WAL file at a constant size, when each memtable instance is flushed the disk, the corresponding WAL file is deleted because it's no longer needed.

## Writing Data

When a write comes in, add it to an in-memory balanced tree data structure (memtable). When the memtable gets **bigger than some threshold**, typically a few megabytes, write it out to disk as an **SSTable** file.

We can also apply write-ahead-log (WAL) technique to avoid the case memtable not persisted on disk when the server crash.

![](https://user-images.githubusercontent.com/17776979/194327144-1a89b275-e274-47ba-9e2c-223adc5332ba.png)

## Reading Data

In order to serve a read request, first try to find the key in the memtable, then in the on-disk segments. But how can we identify if the segment contains our record. This is something that a **bloom filter** can help us out with. A bloom filter is a space-efficient data structure that can tell us if a value is missing from our data.

## Deleting Data

How do you delete data from the SSTable when the segment files are considered immutable? Deletes actually follow the exact same path as writing data.

Whenever a delete request is received, a unique marker called a tombstone is written for that key.

![](https://user-images.githubusercontent.com/17776979/194328490-777a47ae-e481-4146-b43b-30cadd7d7f0c.png)

## Compaction

Over time, this system will accumulate more segment files as it continues to run. These segment files need to be cleaned up and maintained in order to prevent the number of segment files from getting out of hand. This is the responsibility of a process called **compaction**.

Compaction is a background process that is continuously combining old segments together into newer segments. Since we sort keys now, we can make segments also sorted by key during compaction. Notice that during compaction, we can remove duplicate keys (perform updates) while writing the new compacted segment. This can be easily done while segments are resident on disk using an algorithm that's very similar to a normal **merge sort**

![](https://user-images.githubusercontent.com/17776979/194321372-84bc8a7b-4209-4dcc-ba81-2aa589143415.png)

## Disadvantages of LSM Tree

- Compaction process sometime interfere with the performance of ongoing reads and writes.
- Although use of bloom filter some how increase the performance in case of key not present, but if key exist then each key may exist at multiple places and thus checking if a key doesn’t exist needs all segments to be scanned.
