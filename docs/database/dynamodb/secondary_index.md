# Secondary Index

AWS DynamoDB being a No SQL database doesn’t support queries such as SELECT with a condition such as the following query.

```sql
SELECT * FROM Users WHERE email='username@email.com';
```

It is possible to obtain the same query result using DynamoDB scan operation. However, scan operations access every item in a table which is slower than query operations that access items at specific indices. Imagine, you have to look for a book in a library by going through possibly all the books in the library versus you know which shelf the book is at.

Thus, there is a need for another table or data structure that stores data with different primary key and maps a subset of attributes from this base table. This other table is called a **secondary index** and is managed by AWS DynamoDB. When items are added, modified, or deleted in the base table, associated secondary indexes will be updated to reflect the changes.

## Global Secondary Index(GSI)

Global secondary index is an index that have a partition key and an optional sort key that are different from base table's primary key. It is deemed **global** because queries on the index can access the data across different partitions of the base table. 

It can viewed as a different table that contains attributes based on the base table.

```bash
aws dynamodb create-table \
    --table-name MusicCollection \
    --attribute-definitions AttributeName=Artist,AttributeType=S AttributeName=Song,AttributeType=S AttributeName=Genre,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH \
                 AttributeName=Song,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --global-secondary-indexes \
    "[
          {
              \"IndexName\" : \"GenreIndex\",
              \"KeySchema\" : [
                  {\"AttributeName\" : \"Genre\", \"KeyType\" : \"HASH\"},
                  {\"AttributeName\" : \"Artist\", \"KeyType\" : \"RANGE\"}
              ],
              \"Projection\": {
                  \"ProjectionType\" : \"INCLUDE\",
                  \"NonKeyAttributes\" :[\"Song\"]
              },
              \"ProvisionedThroughput\": {
                  \"ReadCapacityUnits\" : 10,
                  \"WriteCapacityUnits\" : 6
              }
          }
    ]"
```

## Local Secondary Index(LSI)

Local secondary index is an index that must have the **same partition key** but a **different sort key** from the base table. It is considered **local** because every partition of a local secondary index is bounded by the same partition key value of the base table. 

It enables data query with different sorting order of the specified sort key attribute.

```bash
aws dynamodb create-table \
    --table-name Music \
    --attribute-definitions AttributeName=Artist,AttributeType=S AttributeName=SongTitle,AttributeType=S \
        AttributeName=AlbumTitle,AttributeType=S  \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput \
        ReadCapacityUnits=10,WriteCapacityUnits=5 \
    --local-secondary-indexes \
        "[{\"IndexName\": \"AlbumTitleIndex\",
        \"KeySchema\":[{\"AttributeName\":\"Artist\",\"KeyType\":\"HASH\"},
                      {\"AttributeName\":\"AlbumTitle\",\"KeyType\":\"RANGE\"}],
        \"Projection\":{\"ProjectionType\":\"INCLUDE\",  \"NonKeyAttributes\":[\"Genre\", \"Year\"]}}]"
```

**Data projection**

When setting up an index, DynamoDB will allow three data projection options. A data projection is a definition of which item attributes are projected – or copied – into the index:

1. Key-only projection: only the `primary-key` and `sort-key` item attributes will be available in index
2. All projection: an exact copy of all item attributes will be available in the index, which doubles the storage space the item takes in total
3. Custom projection: the developer chooses which items will be available in the index apart from the `primary-key` and `sort-key`

|                                               | Global secondary index                                                                                                                                                                                                                                                                                                | Local secondary index                                                                                                                                                                                                                |
| --------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Definition**                                | An index with a partition key and a sort key that can be different from those on the base table.                                                                                                                                                                                                                      | An index that has the same partition key as the base table, but a different sort key.                                                                                                                                                |
| **Span of query**                             | Queries on the index can span all of the data in the base table, across all partitions.                                                                                                                                                                                                                               | Every partition of a local secondary index is scoped to a base table partition that has the same partition key value.                                                                                                                |
| **Primary Key Schema**                        | The partition key and, optionally, the sort key.                                                                                                                                                                                                                                                                      | Must be both partition key and sort key.                                                                                                                                                                                             |
| **Partition Key Attributes**                  | Any base table attribute of type string, number, or binary.                                                                                                                                                                                                                                                           | Must have the same attribute as the partition key of the base table.                                                                                                                                                                 |
| **Sort Key Attributes**                       | Any base table attribute of type string, number, or binary.                                                                                                                                                                                                                                                           | Any base table attribute of type string, number, or binary.                                                                                                                                                                          |
| **Key Values**                                | Do not need to be unique.                                                                                                                                                                                                                                                                                             | The sort key value does not need to be unique for a given partition key value.                                                                                                                                                       |
| **Size Restrictions Per Partition Key Value** | No restriction.                                                                                                                                                                                                                                                                                                       | For each partition key value, the total size of all indexed items must be 10 GB or less.                                                                                                                                             |
| **Index Operations**                          | Can be created during creation of a table.<br> Can be created on an existing table.<br> Can be deleted from an existing table.                                                                                                                                                                                        | Can be created during creation of a table.                                                                                                                                                                                           |
| **Read Consistency for Queries**              | Supports eventual consistency only from asynchronous updates and deletes.                                                                                                                                                                                                                                             | You can choose either eventual consistency or strong consistency.                                                                                                                                                                    |
| **Provisioned Throughput Consumption**        | Every global secondary index has its own provisioned throughput settings for read and write activity that you need to specify. Queries or scans on a global secondary index consume capacity units from the index, not from the base table. This is also case for global secondary index updates due to table writes. | Queries or scans on a local secondary index consume read capacity units from the base table. When you write to a table, and its local secondary indexes are updated, these updates consume write capacity units from the base table. |
| **Projected Attributes**                      | With global secondary queries or scans, you can only request attributes that are projected into this index.                                                                                                                                                                                                           | If you query or scan a local secondary index, you can request attributes that are not projected into the index. DynamoDB will automatically fetch those attributes from the table.                                                   |
| **Index limit (default)**                     | 20 indexes.                                                                                                                                                                                                                                                                                                           | 5 indexes per table.                                                                                                                                                                                                                 |
