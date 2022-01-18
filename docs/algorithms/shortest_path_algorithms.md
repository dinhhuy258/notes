# Shortest Path Algorithms

The shortest path problem is about finding a path between 2 vertices in a graph such that the total sum of the edges weights is minimum.

## 1. Bellman Ford's Algorithm

Bellman Ford's algorithm is used to find the shortest paths from the **source** vertex to **all other vertices** in a weighted graph.

**Algorithm Steps:**

- Initialize an array D to keep track of the shortest path from `s` to all of the nodes
- Set every entry in D to +∞
- The outer loop traverses from: 0 -> n - 1 .
- Loop over all edges, check if the D[next] > D[current] + weight, in this case update D[next] = D[current] + weight.
- Repeat the step 4 to find nodes caught in a negative cycle

```cpp
std::vector<int> dist(n, std::numeric_limits<int>::max());
dist[0] = 0;

for (int i = 0; i < n - 1; ++i) {
  for (int j = 0; j < m; ++j){
    auto from = edges[j][0];
    auto to = edges[j][1];
    auto w = edges[j][2];

    if (dist[from] != std::numeric_limits<int>::max() && dist[from] + w < dist[to]) {
      dist[to] = dist[from] + w;
    }
  }
}

// Repeat to find nodes caught in a negative cycle
for (int i = 0; i < n - 1; ++i) {
  for (int j = 0; j < m; ++j){
    auto from = edges[j][0];
    auto to = edges[j][1];
    auto w = edges[j][2];

    if (dist[from] != std::numeric_limits<int>::max() && dist[from] + w < dist[to]) {
      dist[to] = std::numeric_limits<int>::min();
    }
  }
}
```

The time complexity of the algorithm is O(VE).
