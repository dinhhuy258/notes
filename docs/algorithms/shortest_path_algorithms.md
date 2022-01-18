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

## 2. Floyd Warshall's Algorithm

Floyd Warshall's Algorithm is used to find the shortest paths between between **all pairs of vertices** in a graph, where each edge in the graph has a weight which is positive or negative.

**Algorithm Steps:**

- Initialize the shortest paths between any 2 vertices with Infinity.
- Find all pair shortest paths that use 0 intermediate vertices, then find the shortest paths that use 1 intermediate vertex and so on.. until using all V vertices as intermediate nodes.
- Minimize the shortest paths between any 2 pairs in the previous operation.
- For any 2 vertices (i,j), one should actually minimize the distances between this pair using the first K nodes, so the shortest path will be: min(dist[i][k]+dist[k][j], dist[i][j])

```cpp
for(int k = 1; k <= n; k++){
  for(int i = 1; i <= n; i++){
    for(int j = 1; j <= n; j++){
      dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
    }
  }
}
```

The time complexity of the algorithm is O(V^3).
