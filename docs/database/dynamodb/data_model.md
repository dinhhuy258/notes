# DynamoDB Data Model

## Tables

Tables are the basic store of data. We store data as items in our tables. However, tables in DynamoDB are schema-less. There is no fixed column structure that we have to specify and follow. We can literally put anything inside our table, as long as the primary key is kept unique.

## Items

Items in DynamoDB are similar to the rows in relational databases. An item belongs to a table and can contain multiple attributes. An item in DynamoDB can also be represented as a JSON object (a collection of key-value pairs). Size of the item should not exceed 400KB.

## Attributes

Each individual key-value pair of an item is known as an attribute. We can think of attributes as the properties of the item when we think of the item as a JSON object.

## Primary Key

Each table in DynamoDB contains a primary key. A primary key is a special set of attributes. Its value is unique for every item and is used to identify the item in the database. Under the hood, it is used to partition and store the data in order.

There are two types of primary keys:

- Partition key: Here, we have a **unique** key of scalar type (string, number, boolean), which determines the storage partition the item will go into.
- Partition key and Sort key: Here, we have two keys. The partition key determines the partition where the item goes into the storage and the sort key determines the rank of the item in the partition. Their combination should be **unique**.

## Sort key

Sort key is also known as range key. Ideally, we do not need a range key. It is optional. However, it comes in handy when:

- Our partition key is not unique by itself. In this case, we can use a combination of partition key and range key as the primary key.
- Our access patterns require us to get values in some **range**. For example, if we have an order table partitioned on the `customer_id`, we may have an access pattern to get all the orders with less than $400 amount. Now to get this access pattern, if we only rely on the customer_id you will be in trouble. But if we make the amount as the sort key, we can easily get the results.

## Data types

**Scalar**

- S, String
- N, Number
- BOOL, Boolean
- B, Byte Buffer
- Null, Null

**Document**

- M, Map
- L, List

**Set**

- SS, String Set
- NS, Number Set
- BS, Binary Set

## Time to Live (TTL)

Time To Live (TTL) for DynamoDB is a cost-effective method for deleting items that are no longer relevant. TTL allows you to define a per-item expiration timestamp that indicates when an item is no longer needed. DynamoDB automatically deletes expired items within a few days of their expiration time, without consuming write throughput.

To use TTL, first enable it on a table and then define a specific attribute to store the TTL expiration timestamp. The timestamp must be stored in Unix epoch time format at the seconds granularity. Each time an item is created or updated, you can compute the expiration time and save it in the TTL attribute.

## Capacity Units

Amazon DynamoDB has two read/write capacity modes for processing reads and writes on your tables:

- On-demand
- Provisioned (default, free-tier eligible)

The read/write capacity mode controls how you are charged for read and write throughput and how you manage capacity. You can set the read/write capacity mode when creating a table or you can change it later.

**Write capacity**

- One write request unit represents **one** write for an item up to 1 KB in size.
- Transactional write requests require **two** write request units to perform one write for items up to 1 KB

Eg:

- Size of one item (KB): 21KB
- Write capacity: 150
- Strong consistency write per second: 150 / (21 * 2) = 3
- Eventually consistency write per second: 150 / 21 = 7

**Read capacity**

DynamoDB read requests can be either strongly consistent, eventually consistent, or transactional.

- A strongly consistent read request of an item up to 4 KB requires **one** read request unit.
- An eventually consistent read request of an item up to 4 KB requires **one-half** (1/2) read request unit.
- A transactional read request of an item up to 4 KB requires **two** read request units.

If you need to read an item that is larger than 4 KB, DynamoDB needs additional read request units. The total number of read request units required depends on the item size, and whether you want an eventually consistent or strongly consistent read.

Eg: 

- Size of one item (KB): 21KB
- Read capacity: 150
- Strong consistency write per second: 150 / (21 / 4) = 28
- Eventually consistency read per second: 150 / (21 / 4) * (1 / 2) = 57

Sample command to create DynamoDB table

```bash
aws dynamodb create-table \
    --table-name Blog \
    --attribute-definitions \
    AttributeName=Author,AttributeType=S \
    AttributeName=Topic_Title,AttributeType=S \
    --key-schema \
    AttributeName=Author,KeyType=HASH \
    AttributeName=Topic_Title,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1
```

In this table, I have used the table-name as Blog. And I have used the following primary keys:

- Partition key: Author
- Range key: Topic_Title (The range key is the concatenation of the topic of the post and its title. Both the keys are of string datatype)
