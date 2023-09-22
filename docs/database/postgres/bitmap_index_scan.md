# Bitmap index scan

While `Index Scan` is effective at high correlation, it falls short when the correlation drops to the point where the scanning pattern is more **random** than **sequential** and the number of page fetches increases. One solution here is to collect all the tuple IDs beforehand, sort them by page number, and then use them to scan the table. This is how Bitmap Scan, the second basic index scan method, works.

Bitmap scans are a multi-step process that consist of a Bitmap Heap Scan, one or more Bitmap Index Scans and optionally BitmapOr and BitmapAnd operations.

## How bitmap scan work?

In order to understand what how bitmap index scan work we first need to understand a little bit more about how Postgres stores its data. The columns of a table in Postgres are separated into groups called heap pages. We can picture our table being composed of several pages as shown in the following diagram:

![](https://user-images.githubusercontent.com/17776979/270002996-e6600aab-0063-4293-97d9-d63e9bb9bcad.png)

When we create an index on a column, Postgres can, in general, very efficiently figure out the pages where the rows resulting from a query are located. It can also tell their position within the pages.

Using an `Index Scan` with a b-tree, for each row in the result Postgres will ask the b-tree for the heap page and row location on the page, load the heap page and then retrieve the row.

Heap pages are stored in disk and thus loading a page in memory is **costly**. Thus, when using `Index Scan`, if the query results in a lot of rows, the query might become very slow because an Index Scan will make a page load for each row in the result.

The following diagram shows a scenario where the query results in four rows (show in color) located in three different pages. Each color represents a consecutive group of rows that belong to the result of the query. Using an `Index Scan` Postgres will load pages 2 and 7 once and page 5 twice.

![](https://user-images.githubusercontent.com/17776979/270003033-2bb155af-ad3f-42db-a991-ab778838ef55.png)

**Note**: Random access is also the main problem causing slow performance in `Index Scan` when it returns multiple rows.

Consider the following plan:

```sh
EXPLAIN
SELECT * FROM bookings WHERE total_amount = 48500.00;

   QUERY PLAN
−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−−
 Bitmap Heap Scan on bookings  (cost=54.63..7040.42 rows=2865 wid...
   Recheck Cond: (total_amount = 48500.00)
   −> Bitmap Index Scan on bookings_total_amount_idx
       (cost=0.00..53.92 rows=2865 width=0)
       Index Cond: (total_amount = 48500.00)
(5 rows)
```

The query plan that we obtained above shows an alternative strategy where first, Postgres uses the index to figure out the heap pages that contain results. Then it will read each of those pages entirely in order to fetch the results.

The advantage here is that each page is loaded only once into memory and pages can be loaded sequentially. The disadvantage is that each of those pages will be fully read, meaning that Postgres will inspect some rows that do not belong to the result.

We can thus summarize the scans as follows:

- Bitmap Index Scan: Identify all heap pages that contain results that match the query
- Bitmap Heap Scan: Scan each heap page fully and recheck conditions

The whole bitmap is a single [bit array](https://en.wikipedia.org/wiki/Bit_array), with as many bits as there are heap pages in the relation being scanned.

Bitmap starting off with all entries 0 (false). Whenever an index entry that matches the search condition is found, the heap address pointed to by that index entry is looked up as an offset into the bitmap, and that bit is set to 1 (true). So rather than looking up the heap page directly, the bitmap index scan looks up the corresponding bit position in the bitmap.

Then the bitmap contains information for heap pages we need to bother to load and examine.

Since each heap page might contain multiple rows, we then have to examine each row to see if it matches all the conditions - that's what the `Recheck Cond` part is about.

## Bitmap operations

A query may include multiple fields in its filter conditions. These fields may each have a separate index. Bitmap Scan allows us to take advantage of multiple indexes at once. Each index gets a row version bitmap built for it, and the bitmaps are then ANDed and ORed together. Example:

```sh
Bitmap Heap Scan on customers  (cost=25.76..61.62 rows=10 width=13) (actual time=0.077..0.077 rows=2 loops=1)
  Recheck Cond: (((username)::text < 'user100'::text) AND (customerid < 1000))
  ->  BitmapAnd  (cost=25.76..25.76 rows=10 width=0) (actual time=0.073..0.073 rows=0 loops=1)
        ->  Bitmap Index Scan on ix_cust_username  (cost=0.00..5.75 rows=200 width=0) (actual time=0.006..0.006 rows=2 loops=1)
              Index Cond: ((username)::text < 'user100'::text)
        ->  Bitmap Index Scan on customers_pkey  (cost=0.00..19.75 rows=1000 width=0) (actual time=0.065..0.065 rows=999 loops=1)
              Index Cond: (customerid < 1000)
```

Graphical example:

```cpp
Heap, one square = one page:
+---------------------------------------------+
|c____u_____X___u___X_________u___cXcc______u_|
+---------------------------------------------+
Rows marked c match customers pkey condition.
Rows marked u match username condition.
Rows marked X match both conditions.


Bitmap scan from customers_pkey:
+---------------------------------------------+
|100000000001000000010000000000000111100000000| bitmap 1
+---------------------------------------------+
One bit per heap page, in the same order as the heap
Bits 1 when condition matches, 0 if not

Bitmap scan from ix_cust_username:
+---------------------------------------------+
|000001000001000100010000000001000010000000010| bitmap 2
+---------------------------------------------+
```

Once the bitmaps are created a bitwise AND is performed on them:

```cpp
+---------------------------------------------+
|100000000001000000010000000000000111100000000| bitmap 1
|000001000001000100010000000001000010000000010| bitmap 2
 &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
|000000000001000000010000000000000010000000000| Combined bitmap
+-----------+-------+--------------+----------+
            |       |              |
            v       v              v
Used to scan the heap only for matching pages:
+---------------------------------------------+
|___________X_______X______________X__________|
+---------------------------------------------+
```

The bitmap heap scan then seeks to the start of each page and reads the page:

```cpp
+---------------------------------------------+
|___________X_______X______________X__________|
+---------------------------------------------+
seek------->^seek-->^seek--------->^
            |       |              |
            ------------------------
            only these pages read
```

Each read page is then re-checked against the condition since there can be >1 row per page and not all necessarily match the condition.
