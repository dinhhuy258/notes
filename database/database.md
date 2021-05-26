# Database

## ACID properties

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
