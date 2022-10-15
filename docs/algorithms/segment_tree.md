# Segment Tree

A segment tree is a data structure used to store information about array segments and answer segment queries efficiently:

- range(i, j): gives the sum of the array elements starting at index i and ending at index j.
- update(i, val): updates the value at index i to the val in the original array and updates the segment tree accordingly.

## Structure of the Segment Tree

Consider an array `arr` of size `n` and a corresponding Segment Tree `T`:

- The root of `T` will represent the whole array `arr[0..n-1]`
- Each leaf in the segment tree `T` represent a single element arr[i] such that `0 <= i < n`
- The internal nodes in the segment tree `T` represent the union of elementary intervals arr[i..j] where `0 <= i < j < n`

We can take a divide-and-conquer approach when it comes to array segments. We compute and store the sum of the elements of the whole array, i.e. the sum of the segment `arr[0..n-1]`. We then split the array into two halves `arr[0..n / 2 - 1]` and `arr[n / 2..n - 1]` and compute the sum of each halve and store them. Each of these two halves in turn are split in half, and so on until all segments reach size `1`.

![](https://user-images.githubusercontent.com/17776979/194563256-3e8adfbf-2632-465a-96c5-80d0d4a9fec8.png)

## Construction

Before constructing the segment tree, we need to decide:

- The value that gets stored at each node of the segment tree. For example, in a sum segment tree, a node would store the sum of the elements in its range `[l..r]`.- The merge operation that merges two siblings in a segment tree. For example, in a sum segment tree, the two nodes corresponding to the ranges `arr[l1..r1]` and `arr[l2..r2]` would be merged into a node corresponding to the range `arr[l1..r2]` by adding the values of the two nodes.

Note that a vertex is a **leaf vertex**, if its corresponding segment covers only one value in the original array. It is present at the lowermost level of a segment tree. Its value would be equal to the (corresponding) element `arr[i]`

We can use an array to represent Segment Trees. For each node at index i

- The left child is at index (2 \* i + 1)
- The right child at (2 \* i + 2)
- The parent is at floor((i – 1) / 2).

Segment trees have some nice properties:

- If the underlying array has `n` elements, the segment tree has exactly `(2n - 1)` nodes — `n` leaves and `(n - 1)` internal nodes
- The height of the tree is `O(logn)`
- Each segment can be split into `O(log n)` non-intersecting segments that correspond to the nodes of the segment tree

When `n` is not a perfect power of two, not all levels are filled entirely — the last layer may be incomplete.

```cpp
void build(std::vector<int>& arr, int node, int start, int end) {
  if (start == end) {
    // leaf node will have a single element
    segmentTree[node] = arr[start];
  } else {
    int mid = (start + end) / 2;
    // recurse on the start child
    build(arr, node * 2, start, mid);
    // recurse on the end child
    build(arr, node * 2 + 1, mid + 1, end);
    // internal node will have the sum of both of its children
    segmentTree[node] = segmentTree[node * 2] + segmentTree[node * 2 + 1];
  }
}
```

- arr: the input array
- node: the index of the current vertex
- left and right: the boundaries of the current segment

In the main program this function will be called with the parameters `node = 1`, `left = 0`, `right = n - 1`

Time complexity: `O(n)`.

## Update

Now we want to modify a specific element in the array, let's say we want to do the assignment `arr[i] = x` And we have to rebuild the Segment Tree, such that it correspond to the new, modified array.

It is easy to see, that the update request can be implemented using a recursive function. The function gets passed the current tree vertex, and it recursively calls itself with one of the two child vertices (the one that contains `arr[i]` in its segment), and after that recomputes its sum value, similar how it is done in the build method (that is as the sum of its two children).

```cpp
void update(int node, int start, int end, int pos, int newVal) {
  if (start == end) {
    // leaf node
    segmentTree[node] = newVal;
    return;
  }

  int mid = (start + end) / 2;
  if (pos <= mid) {
    // if idx is in the left child, recurse on the left child
    update(node * 2, start, mid, pos, newVal);
  } else {
    // if idx is in the right child, recurse on the end child
    update(node * 2 + 1, mid + 1, end, pos, newVal);
  }

  // internal node will have the sum of both of its children
  segmentTree[node] = segmentTree[node * 2] + segmentTree[node * 2 + 1];
}
```

Time complexity: `O(logn)`.

## Query

For now we are going to answer sum queries. As an input we receive two integers `l` and `r`, and we have to compute the sum of the segment `arr[l..r]` in `O(logn)` time.

There are three possible cases.

- The easiest case is when the segment `arr[l..r]` is equal to the corresponding segment of the current vertex. Eg: `arr[l..r] = segmentTree[tl..tr]`, then we are finished and can return the precomputed sum that is stored in the vertex.
- The segment of the query can fall completely into the domain of either the left or the right child. In this case we can simply go to the child vertex, which corresponding segment covers the query segment.
- And then there is the last case, the query segment intersects with both children. In this case we have no other option as to make two recursive calls, one for each child. First we go to the left child, compute a partial answer for this vertex, then go to the right child, compute the partial answer using that vertex, and then combine the answers by adding them.

```cpp
int query(int node, int start, int end, int left, int right) {
  if (right < start || end < left || start > end || left > right) {
    // range represented by a node is completely outside the given range
    return 0;
  }

  if (left <= start && end <= right) {
    // range represented by a node is completely inside the given range
    return segmentTree[node];
  }

  // range represented by a node is partially inside and partially outside the given range
  int mid = (start + end) / 2;

  return query(node * 2, start, mid, left, right)
         + query(node * 2 + 1, mid + 1, end, left, right);
}
```

Time complexity: `O(logn)`.

## Updating an interval (Lazy Propagation)

Sometimes problems will ask you to update an interval from `l` to `r`, instead of a single element. One solution is to update all the elements one by one. Complexity of this approach will be `O(N)` per operation since where are `N` elements in the array and updating a single element will take `O(logN)` time.

To avoid multiple call to update function, we can modify the update function to work on an interval.

Let's be Lazy i.e., do work only when needed. How ? When we need to update an interval, we will update a node and mark its child that it needs to be updated and update it when needed. For this we need an array `lazy[]` of the same size as that of segment tree.

Initially all the elements of the `lazy[]` array will be `0` representing that there is no pending update. If there is non-zero element `lazy[k]` then this element needs to update node `k` in the segment tree before making any query operation.

To update an interval we will keep 3 things in mind.

- If current segment tree node has any pending update, then first add that pending update to current node.
- If the interval represented by current node lies completely in the interval to update, then update the current node and update the lazy[] array for children nodes.
- If the interval represented by current node overlaps with the interval to update, then update the nodes as the earlier update function

```cpp
void updateRange(int node, int start, int end, int left, int right, int val)
{
  if(lazy[node] != 0) {
    // this node needs to be updated
    // update it
    segmentTree[node] += (end - start + 1) * lazy[node];
    if(start != end) {
        // mark child as lazy
        lazy[node * 2] += lazy[node];
        lazy[node * 2 + 1] += lazy[node];
    }

    // reset it
    lazy[node] = 0;
  }

  if(start > end || start > right || end < left) {
    // current segment is not within range [left, right]
    return;
  }

  if(left <= start && end <= right) {
    // segment is fully within range
    segmentTree[node] += (end - start + 1) * val;
    if(start != end) {
        // not leaf node
        lazy[node * 2] += val;
        lazy[node * 2 + 1] += val;
    }
    return;
  }

  int mid = (start + end) / 2;

  // updating left child
  updateRange(node * 2, start, mid, left, right, val);
  // updating right child
  updateRange(node * 2 + 1, mid + 1, end, left, right, val);

  // updating root with max value
  segmentTree[node] = segmentTree[node * 2] + segmentTree[node * 2 + 1];
}
```

Query

```cpp
int queryRange(int node, int start, int end, int left, int right) {
  if(start > end || start > right || end < left) {
    // out of range
    return 0;
  }

  if(lazy[node] != 0) {
    // this node needs to be updated
    // update it
    segmentTree[node] += (end - start + 1) * lazy[node];
    if(start != end) {
      // mark child as lazy
      lazy[node * 2] += lazy[node];
      // mark child as lazy
      lazy[node * 2 + 1] += lazy[node];
    }

    // reset it
    lazy[node] = 0;
  }

  if(left <= start && end <= right) {
    // current segment is totally within range [l, r]
    return tree[node];
  }

  int mid = (start + end) / 2;

  // query the left child
  int p1 = queryRange(node*2, start, mid, left, right);
  // query the right child
  int p2 = queryRange(node*2 + 1, mid + 1, end, left, right);

  return (p1 + p2);
}
```

## Advanced versions of Segment Trees

### Finding the maximum

Let us slightly change the condition of the problem described above: instead of querying the sum, we will now make maximum queries.

The tree will have exactly the same structure as the tree described above. We only need to change the way `segmentTree[node]` is computed in the `build` and `update` functions. `segmentTree[node]` will now store the maximum of the corresponding segment. And we also need to change the calculation of the returned value of the `query` function (replacing the summation by the maximum).

### Finding the maximum and the number of times it appears

This task is very similar to the previous one. In addition of finding the maximum, we also have to find the number of occurrences of the maximum.

To solve this problem, we store a pair of numbers at each vertex in the tree: In addition to the maximum we also store the number of occurrences of it in the corresponding segment.

### Compute the greatest common divisor / least common multiple

In this problem we want to compute the GCD / LCM of all numbers of given ranges of the array.

This interesting variation of the Segment Tree can be solved in exactly the same way as the Segment Trees we derived for sum / minimum / maximum queries: it is enough to store the GCD / LCM of the corresponding vertex in each vertex of the tree. Combining two vertices can be done by computing the GCD / LCM of both vertices.

### Counting the number of zeros, searching for the k-th zero

In this problem we want to find the number of zeros in a given range, and additionally find the index of the `k-th` zero using a second function.

Again we have to change the store values of the tree a bit: This time we will store the number of zeros in each segment in `segmentTree[]`. It is pretty clear, how to implement the `build`, `update` and `countZero` functions, we can simply use the ideas from the sum query problem. Thus we solved the first part of the problem.

Now we learn how to solve the problem of finding the `k-th` zero in the array `arr`. To do this task, we will descend the Segment Tree, starting at the root vertex, and moving each time to either the left or the right child, depending on which segment contains the `k-th` zero. In order to decide to which child we need to go, it is enough to look at the number of zeros appearing in the segment corresponding to the left vertex. If this precomputed count is greater or equal to `k`, it is necessary to descend to the left child, and otherwise descent to the right child. Notice, if we chose the right child, we have to subtract the number of zeros of the left child from `k`.

```cpp
int findKth(int node, int start, int end, int k) {
  if (k > segmentTree[node]) {
    return -1;
  }

  if (start == end) {
    return start;
  }

  int mid = (start + end) / 2;
  if (segmentTree[node * 2] >= k) {
    return findKth(node * 2, start, mid, k);
  } else {
    return findKth(node * 2 + 1, mid + 1, end, k - segmentTree[node * 2]);
  }
}
```

### Searching for an array prefix with a given amount

The task is as follows: for a given value `x` we have to quickly find smallest index `i` such that the sum of the first `i` elements of the array `arr` is greater or equal to `x` (assuming that the array `arr` only contains non-negative values).

This task can be solved using binary search, computing the sum of the prefixes with the Segment Tree. However this will lead to a `O(logn*logn)` solution

Instead we can use the same idea as in the previous section, and find the position by descending the tree: by moving each time to the left or the right, depending on the sum of the left child. Thus finding the answer in `O(logn)` time

### Searching for the first element greater than a given amount

The task is as follows: for a given value `x` and a range `[l..r]` find the smallest `i` in the range `arr[l..r]`, such that `arr[i]` is greater than `x`.

This task can be solved using binary search over max prefix queries with the Segment Tree. However, this will lead to a `O(logn*logn)` solution.

Instead, we can use the same idea as in the previous sections, and find the position by descending the tree: by moving each time to the left or the right, depending on the maximum value of the left child. Thus finding the answer in `O(logn)` time

### Finding subsegments with the maximal sum

Here again we receive a range `arr[l..r]` for each query, this time we have to find a subsegment `arr[l'..r']` such that `l <= l'` and `r' <= r` and the sum of the elements of this segment is maximal

This time we will store four values for each vertex:

- the sum of the segment
- the maximum prefix sum
- the maximum suffix sum
- the sum of the maximal subsegment in it

You can find more [here](https://cp-algorithms.com/data_structures/segment_tree.html)
