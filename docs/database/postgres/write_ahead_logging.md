# Write Ahead Logging — WAL

Write-Ahead Log (WAL) is a very important term in transaction processing. In PostgreSQL, it is also known as a transaction log. A log is a record of all the events or changes and WAL data is just a description of changes made to the actual data

The term `Write-Ahead Log` implies that any change that you make to the database must first be appended to the log file, and then the log file should be flushed to disk

The basic purpose of Write-Ahead Logging is to ensure that when there is a crash in the operating system or PostgreSQL or the hardware, the database can be recovered.

## Insertion Operations without WAL

1. Issuing the first INSERT statement, PostgreSQL loads the `TABLE_A`'s page from a database cluster into the in-memory shared buffer pool, and inserts a tuple into the page. This page is not written into the database cluster immediately. (a modified pages are generally called a dirty page.)
2. Issuing the second INSERT statement, PostgreSQL inserts a new tuple into the page on the buffer pool. This page has not been written into the storage yet.
3. If the operating system or PostgreSQL server should fail for any reasons such as a power failure, all of the inserted data would be lost.

Note: Before WAL was introduced (version 7.0 or earlier), PostgreSQL did synchronous writes to the disk by issuing a sync system call whenever changing a page in memory in order to ensure durability. Therefore, the modification commands such as INSERT and UPDATE were very poor-performance.
