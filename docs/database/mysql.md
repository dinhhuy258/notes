# MySQL

## 1. Clustered index and non-clustered index

A clustered index is a B-Tree index whose leaf nodes are the actual data blocks on disk. In cluseted index the data are stored physically on the disk in the same order as the clustered index. Therefore, there can be only one clustered index.

A non-clustered index is a B-Tree index whose leaf nodes point to the clustered index key. We can have multiple non-clustered indexes per table.

Clustered indexes are faster than non-clustered indexes since they don’t involve any extra lookup step. In clustered index we just only traverse the tree once however in non-clustered index we need to do it twice. The first one for getting clustered index key in non-clustered B-tree and the second one for getting actual data from clustered B-tree.

When you define a primary key for an InnoDB table, MySQL uses the primary key as the clustered index.
If you do not have a primary key for a table, MySQL will search for the first UNIQUE index where all the key columns are NOT NULL and use this UNIQUE index as the clustered index.
In case the InnoDB table has no primary key or suitable UNIQUE index, MySQL internally generates a hidden clustered index named GEN_CLUST_INDEX on a synthetic column that contains the row ID values.

## 2. MySQL InnoDB vs MyISAM

| InnnoDB                                           | MyISAM                                    |
| :------------------------------------------------ | :---------------------------------------- |
| Row level locking                                 | Table level locking                       |
| Supports foreign key                              | Does not support relationship constraints |
| Transactional (Rollback, commit)                  | Non-transactional                         |
| ACID compliant                                    | Not ACID compliant                        |
| Row data stored in pages as per primary key order | No particular order for data stored       |
