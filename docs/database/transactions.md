# Transactions

## 1. ACID properties

ACID is a set of properties of database transaction intended to guarantee data validity despite errors.

### Atomicity

Transactions are often composed of multiple stataments. Atomicity guarantees that each transaction is treated as a single unit, which either succeeds completely or fails completely. If any of the statements constituting a transaction fails to complete, the entire transaction fails and the database is left unchanged.

### Consistency

The data is in consistent state before and after transaction.

Eg: We have a transaction to transfer money from account A to account B. The total amount before and after the transaction must be the same.

### Isolation

If the multiple transactions are running concurrently, they should not be affected by each other.

### Durability

Changes that have been committed to the database should remain even in the case of software and system failure.

## 2. Read phenomena

### Dirty read

A dirty read is the situation when a transaction reads a uncommitted data from other transactions.

### Non-repeatable read

Non repeatable read occurs when a transaction reads same row twice, and get a different value each time.

For example, suppose transaction T1 reads data. Due to concurrency, another transaction T2 updates the same data and commit, Now if transaction T1 rereads the same data, it will retrieve a different value.

### Phantom read

Phantom read occurs when two same range queries are executed, but the number of rows retrieved by the two, are different.

For example, suppose transaction T1 retrieves a set of rows that satisfy some search criteria. Now, Transaction T2 generates some new rows that match the search criteria for transaction T1. If transaction T1 re-executes the statement that reads the rows, it gets a different set of rows this time.

## 3. Isolation levels

**Serializalble:** This is the highest isolation level. Concurrent transactions are guaranteed to be executed in sequence.

**Repeatable Read:** Data read during the transaction stays the same as the transaction starts.

**Read Committed:** Data modification can only be read after the transaction is committed.

**Read Uncommitted:** The data modification can be read by other transactions before a transaction is committed.

The isolation is guaranteed by MVCC (Multi-Version Consistency Control) and locks.

![](https://user-images.githubusercontent.com/17776979/194693921-c6368a0c-f62a-4854-98c6-88de25217d82.png) 

The diagram above takes Repeatable Read as an example to demonstrate how MVCC works:

- There are two hidden columns for each row: transaction_id and roll_pointer
- When transaction A starts, a new Read View with transaction_id=201 is created.
- Shortly afterward, transaction B starts, and a new Read View with transaction_id=202 is created. 
- Now transaction A modifies the balance to 200, a new row of the log is created, and the roll_pointer points to the old row
- Before transaction A commits, transaction B reads the balance data. Transaction B finds that transaction_id 201 is not committed, it reads the next committed record(transaction_id=200).
- Even when transaction A commits, transaction B still reads data based on the Read View created when transaction B starts. So transaction B always reads the data with balance=100. 

**Note:** In `Repeatable Reads` isolation or higher if we are updating row data at the same time another concurrent transaction is also updating the same row then we are getting an error `ERROR: could not serialize access due to concurrent update` (In MySQL we need Serializable isolation level).
