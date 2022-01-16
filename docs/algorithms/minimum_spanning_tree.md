# Minimum Spanning Tree

## 1. What is a Spanning Tree?

In an undirected and connected graph G=(V,E), a spanning tree is a subgraph that is a tree which includes all of the vertices of G, with minimum possible number of edges. A graph may have several spanning trees. The cost of the spanning tree is the sum of the weights of all the edges in the tree

## 2. What is a Minimum Spanning Tree?

A minimum spanning tree (M- ST) is the spanning tree where the cost is minimum among all the spanning trees.

![](../assets/images/algorithms/minimum_spanning_tree.png)

## 3. Prim’s Algorithm

- Prim’s algorithm is a greedy algorithm that works well on **dense graphs**.
- It finds a minimum spanning tree for a weighted **UNDIRECTED** graph.

**Algorithm Steps:**

1. Choose any arbitrary node s as root node
2. Enqueues all edges incident to s into a Priority Queue (PQ)
3. Repeatedly do the following greedy steps until PQ is empty: If the vertex v with edge e (w -> v) in the PQ has not been visited then add e to MST and enqueue all edges connected to v into the PQ.

[Visualising](https://visualgo.net/en/mst)

```cpp
#include <iostream>
#include <vector>
#include <queue>

std::vector<std::vector<std::pair<int, int>>> adj;
std::vector<bool> visited;
std::priority_queue<std::pair<int, int>, std::vector<std::pair<int, int>>, std::greater<std::pair<int, int>>> queue;

void addEdges(int s) {
	visited[s] = true;
	for (int i = 0; i < adj[s].size(); ++i) {
		if (visited[adj[s][i].second]) {
			continue;
		}

		queue.push(adj[s][i]);
	}
}

int main() {
	int n, m, a, b, w;
	std::cin >> n >> m;
	adj = std::vector<std::vector<std::pair<int, int>>>(n + 1, std::vector<std::pair<int, int>>{});
	visited = std::vector<bool>(n + 1, false);

	for (int i = 0; i < m; ++i) {
		std::cin >> a >> b >> w;
		adj[a].push_back(std::make_pair(w, b));
		adj[b].push_back(std::make_pair(w, a));
	}

	int edgeCount = 0;
	int mstCost = 0;
	addEdges(1);

	while(!queue.empty() && edgeCount != n - 1) {
		auto cost = queue.top().first;
		auto des = queue.top().second;
		queue.pop();

		if (visited[des]) {
			continue;
		}

		mstCost += cost;
		++edgeCount;

		addEdges(des);
	}

	if (edgeCount != n - 1) {
		// No MST found
		std::cout << "0";
	} else {
		std::cout << mstCost;
	}
}
```

The time complexity of Prim's algorithm is O(E log V).

## 4. Kruskal’s Algorithm

- Kruskal’s algorithm is a greedy algorithm that works well on **dense graphs**.

**Algorithm Steps:**

1. Sort the set of edges E in increasing order
2. Start adding edges to the MST from the edge with the smallest weight until the edge of the largest weight.
3. Only add edges which doesn't form a cycle , edges which connect only disconnected components.

So now the question is how to check if  vertices are connected or not ?

This could be done using DFS but DFS will make time complexity large. So the best solution is `Disjoint Set`

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

class DisjointSet {
private:
	std::vector<int> parents;
public:
	DisjointSet(int n) {
		parents = std::vector<int>(n + 1, -1);
		
		for (int i = 1; i <= n; ++i) {
			parents[i] = i;
		}
	}

	int find(int x) {
		while (parents[x] != x) {
			parents[x] = parents[parents[x]];
			x = parents[x];
		}

		return parents[x];
	}

	void unionSet(int x, int y) {
		auto parentX = find(x);
		auto parentY = find(y);

		parents[parentY] = parentX;
	}
};

int main() {
	int n, m, a, b, w;
	std::cin >> n >> m;
	std::vector<std::pair<int, std::pair<int, int>>> edges;

	for (int i = 0; i < m; ++i) {
		std::cin >> a >> b >> w;
		edges.push_back(std::make_pair(w, std::make_pair(a, b)));
	}

	DisjointSet disjointSet(n);
	std::sort(edges.begin(), edges.end());
	int mst = 0;
	for (int i = 0; i < edges.size(); ++i) {
		auto cost = edges[i].first;
		auto s = edges[i].second.first;
		auto d = edges[i].second.second;

		if (disjointSet.find(s) != disjointSet.find(d)) {
			disjointSet.unionSet(s, d);

			mst += cost;
		}
	}

	std::cout << mst << std::endl;
}
```
