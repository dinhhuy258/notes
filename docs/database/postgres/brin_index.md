# BRIN Index

A BRIN is a Block Range Index. A block (page) is Postgres’ base unit of storage and is by default 8kB of data.
BRIN samples a range of blocks (default 128), storing the location of the first block in the range as well as the minimum and maximum values for all values in those blocks.

It’s very useful for ordered data sets, offering significant **space savings** for similar and sometimes better performance.

BRIN is incredibly helpful in efficiently searching over large time-series data and has the benefit of taking up significantly less space on disk than a standard B-TREE index.

## Where It can be used?

Many applications today record data from sensors, devices, tracking information, real-time bank transactions, and other things that share a common attribute: a timestamp that is always increasing. This timestamp is very valuable, as it serves as the basis for types of lookups, analytical queries, and more.

when used appropriately, a BRIN index will not only outperform a B-tree but will also save over 99% of space on disk.

BRIN index can be considered to be use if your table have:

- The **physical** row ordering 
- The actual structure on disk and
- The logical ordering of column values
- Table update infrequently

## How does it works?

The BRIN index is a small table that associates a range of values with a range of pages in the table order.

Building the index just requires a single scan of the table, so compared to building a structure like a BTree, it is very fast (on write).

![](https://user-images.githubusercontent.com/17776979/270085942-d57f69bb-2b3a-4d0f-9afe-91344d1f65fd.png)

Because the BRIN has one entry for each range of pages, it's also very small. The number of pages in a range is configurable, but the default is `128` (16 pages, each page is 8KB).

## Compare to Btree

**Size**

B+Tree indexes tend to be larger in size compared to BRIN indexes because they store pointers to data rows and maintain a balanced tree structure.

**Query Performance**:

B+Tree indexes excel at point queries, where you need to locate a specific row quickly. They are also suitable for range queries but may become less efficient as the data size grows substantially.

BRIN indexes are optimized for range queries. They are highly efficient when you need to scan large ranges of data. However, they are not well-suited for point queries, as they do not provide direct access to individual rows.

Note: In some scenarios, BRIN may not offer a good performance compare to B+Tree index.

**Data Update Performance:**

B+Tree indexes can be relatively expensive to update

BRIN indexes are designed with data blocks in mind, making them suitable for scenarios where the data is append-only or where updates are infrequent. However update data in BRIN index can make a good performance compare to BTree. (It still affect query performance if the table is updated)

## References

[BRIN Index for PostgreSQL: Don’t Forget the Benefits](https://www.percona.com/blog/brin-index-for-postgresql-dont-forget-the-benefits/)
