# Offset is bad

We often use `offset` in our database queries to fetch paginated data. This helps in implementing server-side pagination where it is not convenient to fetch all the data at once for performance reasons.

## Why offset is bad?

Performing offset on your database affects performance because of the way database fetches the rows. Offset happens after the database has fetched all the matching rows.

![](https://user-images.githubusercontent.com/17776979/277726526-a856512b-aba6-4d20-8ed2-e02a7f45c57d.png)

As shown above, the number of rows that database fetches is `offset + limit` instead of just limit.

## How to avoid offset?

As shown in this [article](https://use-the-index-luke.com/no-offset)

```sql
SELECT someCol, anotherCol
  FROM someTable
 WHERE 'some_condition' > 1
   AND id < ?last_seen_id
 ORDER BY id DESC
 FETCH FIRST 10 ROWS ONLY
```

This is called `keyset_pagination` and it offers the advantages of being faster than offset and also prevents the strange duplication of resuls, or other frustrating anomalies.

There are some downsides however. You can’t go to an arbitrary page for example, so it’s a method better suited to things like infinite scrolling in a way similar to Instagram, than it is to clicking on page numbers. It’s a judgment call as to what will fit your use-case the best.

## References

- [https://mysql.rjweb.org/doc.php/pagination](https://mysql.rjweb.org/doc.php/pagination)  
- [https://use-the-index-luke.com/no-offset](https://use-the-index-luke.com/no-offset) 
