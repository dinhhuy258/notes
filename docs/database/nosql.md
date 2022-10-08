# NoSQL

NoSQL DB is a database used to manage huge sets of unstructured data, where the data is not stored in tabular relations like relational databases.

## 1. BASE properties

The relational databases strongly follow the ACID properties while the NoSQL databases follow BASE principles. In comparison with the **CAP Theorem**, **BASE** chooses availability over consistency.

- **Basic Availability:** The system guarantees availability. There will be a response to any request (can be failure too).
- **Soft state:** The data stored in the system may change because of the eventual consistency model (even without an input, the system state may change).
- **Eventual consistency:** The system will eventually become consistent once it stops receiving input.
