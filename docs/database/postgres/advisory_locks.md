# Advisory Locks

Postgres offers a special type of lock that is completely driven by the client application.
They are not tied to any particular table or row — instead, you define a lock key (a number), and Postgres manages concurrency around that key.

- Locks can be exclusive (only one session) or shared (many sessions at once, but block exclusive).
- Keys can be provided either as a single bigint or as a pair of two ints.
- Locks are application-defined: Postgres doesn’t know what your keys mean; it only ensures consistency across sessions.

There are two types of advisory locks:

- **Session-level advisory locks** - Held until explicitly released or the session ends
- **Transaction-level advisory locks** - Released automatically at the end of the transaction

## Session-Level Advisory Lock

```sql
-- Acquire exclusive lock (blocks until available)
SELECT pg_advisory_lock(key);
SELECT pg_advisory_lock(key1 int, key2 int);

-- Try to acquire exclusive lock (returns immediately)
SELECT pg_try_advisory_lock(key);
SELECT pg_try_advisory_lock(key1 int, key2 int);

-- Acquire shared lock
SELECT pg_advisory_lock_shared(key);
SELECT pg_advisory_lock_shared(key1 int, key2 int);

-- Release specific lock
SELECT pg_advisory_unlock(key);
SELECT pg_advisory_unlock(key1 int, key2 int);
SELECT pg_advisory_unlock_shared(key bigint);
SELECT pg_advisory_unlock_shared(key1 int, key2 int);

-- Release all session locks
SELECT pg_advisory_unlock_all();
```

## Transaction-Level Advisory Lock

Transaction-level locks are automatically released at the end of the transaction (commit or rollback).

```sql
-- Acquire exclusive lock (blocks until available)
SELECT pg_advisory_xact_lock(key bigint);
SELECT pg_advisory_xact_lock(key1 int, key2 int);

-- Try to acquire exclusive lock (non-blocking, returns true/false)
SELECT pg_try_advisory_xact_lock(key bigint);
SELECT pg_try_advisory_xact_lock(key1 int, key2 int);

-- Acquire shared lock
SELECT pg_advisory_xact_lock_shared(key bigint);
SELECT pg_advisory_xact_lock_shared(key1 int, key2 int);

-- Try to acquire shared lock (non-blocking, returns true/false)
SELECT pg_try_advisory_xact_lock_shared(key bigint);
SELECT pg_try_advisory_xact_lock_shared(key1 int, key2 int);
```

## Exclusive vs Shared

- **Exclusive lock** (pg_advisory_lock): only one session can hold it at a time.
- **Shared lock** (pg_advisory_lock_shared): multiple sessions can hold simultaneously, but will block if someone tries to acquire the exclusive version.

