# Btree vs B+tree

## Btree

- Balanced Data structure for fast traversal
- In B-Tree of `m` degree can have up to `m` child nodes
- Node has up to `m - 1` elements (each element has a key and a value)
- The value is usually data pointer to the row (data pointer can point to primary key or tuple)

![](https://user-images.githubusercontent.com/17776979/277109351-5c2a1824-af18-4066-9db9-a79fc8a39be6.png)

### Limitation

- Elements in all nodes store both the key and the **VALUE**
- Internal nodes take more space thus require more IO and can slow down traversal (Hard to fit internal nodes in memory)
- Range queries are slow because of random access (give me all values 1-5)

## B+Tree

- Exactly like B-Tree but only stores keys in internal nodes
- Values are only stored in leaf nodes
- Internal nodes are smaller since they only store keys and they can fit more elements
- Leaf nodes are “linked” so once you find a key you can find all values before and after that key.
- Great for range queries

![](https://user-images.githubusercontent.com/17776979/277109355-2de23e1e-e888-4ea2-8a82-19d6c4bba264.png)

## B+Tree Storage cost

![](https://user-images.githubusercontent.com/17776979/277110144-a7262006-a973-4969-a76b-56f6464c4e02.png)
