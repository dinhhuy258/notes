# Range Types

When working on applications such as a reservation app or calendar app, you need to store the `start time` and `end time` of an event.
You may also need to query events occurring in a specific time frame or ensure that certain events do not overlap.

## The Problem with Traditional Date Columns

Traditionally, when dealing with events or periods, developers often use two separate columns to represent the start and end of a range. For example:

```sql
create table reservations (
  id serial primary key,
  title text,
  start_at timestamptz,
  end_at timestamptz
);
```

While this approach works, it has a few drawbacks:

- Querying Complexity: Writing queries to find overlapping events or events within a specific period becomes complex and error-prone.
- Data Integrity: Ensuring that reservations do not overlap is difficult.

## Enter range types

PostgreSQL has a better solution for these problems — range types. It comes with these additional built-in data types:

- int4range: Range of integer
- int8range: Range of bigint
- numrange: Range of numeric
- tsrange: Range of timestamp without time zone
- tstzrange: Range of timestamp with time zone
- daterange: Range of date

You can use them as a column type in a table:

```sql
create table reservations (
  id serial primary key,
  title text,
  table_id int4,
  duration tstzrange
);
```

### Querying range columns

```sql
select * from reservations where duration && '[2024-07-04 16:00, 2024-07-04 19:00)';
```

Postgres provides more range-specific operators. The official Postgres documentation provides a complete list of [range operators](https://www.postgresql.org/docs/9.3/functions-range.html).

### Querying range columns

When working on a reservations app, you might want to ensure there are no overlapping reservations. Range columns make it easy to add such constraints. The following SQL statement adds an exclude constraint that prevents new inserts/ updates from overlapping on any of the existing reservations.

```sql
-- Enable the btree_gist index required for the constraint.
create extension btree_gist

-- Add a constraint to prevent overlaps with the same table_id
alter table reservations
  add constraint exclude_duration
  exclude using gist (table_id WITH =, duration WITH &&);
```

```sql
-- Add a first reservation
insert into reservations (title, table_id, duration)
values ('Tyler Dinner', 1, '[2024-07-04 18:00, 2024-07-04 21:00)');

-- Insert fails, because table 1 is taken from 18:00 - 21:00
insert into reservations (title, table_id, duration)
values ('Thor Dinner', 1, '[2024-07-04 20:00, 2024-07-04 22:00)');

-- Insert succeeds because table 2 is not taken by anyone
insert into reservations (title, table_id, duration)
values ('Thor Dinner', 2, '[2024-07-04 20:00, 2024-07-04 22:00)');
```

## References

- [Simplifying Time-Based Queries with Range Columns](https://supabase.com/blog/range-columns)
