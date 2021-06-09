---
title: Transactions
parent: Database
---

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

| Isolation Levels | Dirty Reads | Non-repeatable Reads | Phantom Reads |
| :--------------: | :---------: | :------------------: | :-----------: |
| Read Uncommitted |      ✔      |          ✔           |       ✔       |
|  Read Committed  |      ✘      |          ✔           |       ✔       |
| Repeatable Reads |      ✘      |          ✘           |       ✔       |
|   Serializable   |      ✘      |          ✘           |       ✘       |
