# Submask Enumeration

## Enumerating all submasks of a given mask

Given a bitmask `m`, you want to efficiently iterate through all of its submasks, that is, masks `s` in which only bits that were included in mask `m` are set.

1101 -> [1101, 1100, 1001, 1000, 0101, 0100, 0001]

```cpp
for (int s = m; s > 0; s = (s - 1) & m) {
 ... you can use s ...
}


int s = m;
while (s > 0) {
 ... you can use s ...
 s = (s - 1) & m;
}
```

**Why does it work?**

Suppose we have a current bitmask `s`, and we want to move on to the next bitmask. By subtracting from the mask `s` one unit `(s - 1)`, we will remove the **rightmost** set bit and **all bits to the right of it** will become `1`. Then we remove all the `extra` one bits that are not included in the mask `m` and therefore can't be a part of a submask. We do this removal by using the bitwise operation `(s-1) & m`. As a result, we **cut** mask `s - 1` to determine the highest value that it can take, that is, the next submask after `s` in descending order.

## Iterating through all masks with their submasks

In many problems, especially those that use bitmask dynamic programming, you want to iterate through all bitmasks and for each mask, iterate through all of its submasks:

```cpp
for (int m = 0; m < (1 << n); ++m)
  for (int s = m; s > 0; s = (s - 1) & m)
    ... s and m ...
```

**Time complexity**

O(3<sup>n</sup>)
