# Shortest Path Algorithms

The shortest path problem is about finding a path between 2 vertices in a graph such that the total sum of the edges weights is minimum.

|                                   | BFS                             | Dijkstra        | Bellman Ford | Floyd Warshall |
| --------------------------------- | ------------------------------- | --------------- | ------------ | -------------- |
| Complexity                        | O(V+E)                          | O(V + E log(V)) | O(VE)        | O(V3)          |
| Recommended grapth size           | Large                           | Large/Medium    | Medium/Small | Small          |
| Good for APSP                     | Only works on unweighted graphs | Ok              | Bad          | Yes            |
| Can detect negative cycles        | No                              | No              | Yes          | Yes            |
| SP on graph with weighted edges   | Incorrect SP answer             | Best algorithm  | Works        | Bad in general |
| SP on graph with unweighted edges | Best algorithm                  | Ok              | Bad          | Bad in general |

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

## 2. Dijkstra's Algorithm

Dijkstra's algorithm is used to find the shortest paths from the **source** vertex to **all other vertices** in a weighted graph.

One contraint for Dijkstra's algorithm is that the graph must only contain **non-negative edge weight**. This contraint is imposed to ensure that once a node has been visited its optimal distance can not be improved.

**Algorithm Steps:**

- Maintain a `dist` array where the distance to every node is positive infinity, mark the distance to the start node `s` to be `0`
- Maintain a PQ of key-value pairs of (node_idx, distance) pairs and insert (s, 0) in to the PQ
- Pull out the pair (node_idx, distance) from PQ
- Update the distance of the connected vertices to node in `node_idx` in case of `current vertex distance + edge weight < next vertex distance`, then push them to PQ
- If the popped vertex is visited before, just continue without using it.
- Apply the same algorithm again until the PQ is empty.

```cpp
std::vector<int> dist(n, std::numeric_limits<int>::max());
std::vector<bool> visited(n, false);
std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, std::greater<std::pair<int, int>>> queue;

dist[0] = 0;
queue.push(std::make_pair(0, 0));

while(!queue.empty()) {
  auto w = queue.top().first;
  auto node_idx = queue.top().second;
  queue.pop();

  if (visited[node_idx]) {
    continue;
  }

  visited[node_idx] = true;

  if (dist[node_idx] < w) {
    continue;
  }

  for (int i = 0; i < adj[node_idx].size(); ++i) {
    auto to = adj[node_idx][i].first;
    auto cost = adj[node_idx][i].second;

    if (dist[node_idx] + cost < dist[to]) {
      dist[to] = dist[node_idx] + cost;
      queue.push(std::make_pair(dist[to], to));
    }
  }
}
```

The time complexity of the algorithm is O(V + E log(V)).

## 3. BFS 0-1

We know that we can use Dijkstra algorithm with time complexity O(V + E log(V)) to solve SSSP problems.

However if the weights are more constrained we can do better, in this case we can use BFS to solve the SSSP problem in O(E), if weight of each edge is either 0 or 1.

The algorithm is based on the Dijkstra algorithm

```cpp
std::vector<int> d(n, INF);
d[s] = 0;
std::deque<int> q;
q.push_front(s);

while (!q.empty()) {
  int v = q.front();
  q.pop_front();

  for (auto edge : adj[v]) {
    int u = edge.first;
    int w = edge.second;
    if (d[v] + w < d[u]) {
      d[u] = d[v] + w;
      if (w == 1) {
        q.push_back(u);
      }
      else {
        q.push_front(u);
      }
    }
  }
}
```

## 4. Floyd Warshall's Algorithm

Floyd Warshall's Algorithm is used to find the shortest paths between between **all pairs of vertices** in a graph, where each edge in the graph has a weight which is positive or negative.

**Algorithm Steps:**

- Initialize the shortest paths between any 2 vertices with Infinity.
- Find all pair shortest paths that use 0 intermediate vertices, then find the shortest paths that use 1 intermediate vertex and so on.. until using all V vertices as intermediate nodes.
- Minimize the shortest paths between any 2 pairs in the previous operation.
- For any 2 vertices (i,j), one should actually minimize the distances between this pair using the first K nodes, so the shortest path will be: min(dist[i][k]+dist[k][j], dist[i][j])
- Repeat to find nodes caught in a negative cycle

```cpp
for(int k = 1; k <= n; k++){
  for(int i = 1; i <= n; i++){
    for(int j = 1; j <= n; j++){
      dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
    }
  }
}

// Detect negative cycle
for(int k = 1; k <= n; k++){
  for(int i = 1; i <= n; i++){
    for(int j = 1; j <= n; j++){
      dist[i][j] = std::numeric_limits<int>::min();
    }
  }
}
```

The time complexity of the algorithm is O(V^3).
