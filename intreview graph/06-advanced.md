# Pattern 6: Advanced Graph Algorithms

## Patterns Covered
1. Bipartite Check (2-coloring)
2. Bellman-Ford (negative edges)
3. Floyd-Warshall (all-pairs shortest path)
4. MST — Kruskal's and Prim's
5. Eulerian Path / Hierholzer's
6. Strongly Connected Components (SCC)

---

## 1. Bipartite Check (2-Coloring)

### Identify
- "Can you divide nodes into 2 groups with no edges within a group?"
- "Is this graph bipartite?"
- Problems about conflict-free 2-partition (teams, sides, colors)
- Equivalent: does graph have an ODD-length cycle? (odd cycle → not bipartite)

### Template
```cpp
// BFS 2-coloring
bool isBipartite(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> color(n, -1);

    for (int start = 0; start < n; start++) {
        if (color[start] != -1) continue;  // already colored
        queue<int> q;
        q.push(start); color[start] = 0;

        while (!q.empty()) {
            int u = q.front(); q.pop();
            for (int v : graph[u]) {
                if (color[v] == -1) {
                    color[v] = 1 - color[u];  // flip color
                    q.push(v);
                } else if (color[v] == color[u]) {
                    return false;              // same color = not bipartite
                }
            }
        }
    }
    return true;
}
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [785](https://leetcode.com/problems/is-graph-bipartite/) | Is Graph Bipartite | BFS 2-color from each unvisited node | Must handle disconnected graph — loop over all starts |
| [886](https://leetcode.com/problems/possible-bipartition/) | Possible Bipartition | Build conflict graph (disliked pairs = edges), check bipartite | Same 2-coloring — "two groups with no conflict within group" = bipartite |

---

## 2. Bellman-Ford (Negative Edges)

### Identify
- Graph has NEGATIVE edge weights
- Need to detect NEGATIVE CYCLES
- Fewer edges (edge list form), small graphs

### Template
```cpp
vector<int> bellmanFord(int src, int n, vector<tuple<int,int,int>>& edges) {
    // edges = {u, v, weight}
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    // Relax ALL edges n-1 times
    for (int i = 0; i < n - 1; i++) {
        for (auto [u, v, w] : edges) {
            if (dist[u] != INT_MAX && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
        }
    }

    // nth pass: if relaxation still happens → negative cycle
    for (auto [u, v, w] : edges) {
        if (dist[u] != INT_MAX && dist[u] + w < dist[v])
            return {};  // negative cycle detected
    }

    return dist;
}
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [787](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Cheapest Flights K Stops | BF variant: run exactly k+1 relaxation rounds. Copy dist each round to not chain updates | Use `prev_dist` snapshot to avoid using new values in same round |

**Why copy dist each round for #787:**
```cpp
// Wrong: dist updated mid-round chains into multi-hop path
// Correct: use prev = dist before each round
vector<int> prev = dist;
for (auto [u, v, w] : flights)
    if (prev[u] != INT_MAX && prev[u] + w < dist[v])
        dist[v] = prev[u] + w;
```

---

## 3. Floyd-Warshall (All-Pairs Shortest Path)

### Identify
- Need shortest path between ALL pairs
- Small graph (n ≤ 200)
- "Can node i reach node j?" (transitive closure)
- Handles negative edges (but not negative cycles)

### Template
```cpp
// Initialize: dist[i][j] = edge weight, INF if no edge, 0 if i==j
void floydWarshall(vector<vector<int>>& dist, int n) {
    for (int k = 0; k < n; k++)           // intermediate node
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX)
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
}
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1334](https://leetcode.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/) | Find City with Fewest Neighbors | Floyd-Warshall all-pairs, count neighbors within threshold per city | O(n³) acceptable since n ≤ 100 |
| [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | Build weighted graph (a/b = weight). BFS/DFS to find product of path weights | Floyd-Warshall also works for all-pairs queries |

---

## 4. Minimum Spanning Tree (MST)

### Identify
- "Connect ALL nodes with MINIMUM total edge cost"
- "Minimum cost to build roads / cables / network"
- Result: n-1 edges, all nodes connected, minimum total weight

### Kruskal's (sort edges + DSU) — Preferred for interviews
```cpp
int kruskal(int n, vector<tuple<int,int,int>>& edges) {
    // edges = {weight, u, v}
    sort(edges.begin(), edges.end());
    DSU dsu(n);
    int cost = 0, used = 0;
    for (auto [w, u, v] : edges) {
        if (dsu.unite(u, v)) {
            cost += w;
            if (++used == n - 1) break;
        }
    }
    return used == n - 1 ? cost : -1;  // -1 = graph not connected
}
```

### Prim's (min-heap) — Better for dense graphs
```cpp
int prim(int n, vector<vector<pair<int,int>>>& adj) {
    vector<bool> inMST(n, false);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, 0});
    int cost = 0;
    while (!pq.empty()) {
        auto [w, u] = pq.top(); pq.pop();
        if (inMST[u]) continue;
        inMST[u] = true; cost += w;
        for (auto [v, wv] : adj[u])
            if (!inMST[v]) pq.push({wv, v});
    }
    return cost;
}
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1584](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Min Cost Connect All Points | Generate all O(n²) pairs as edges with Manhattan distance. Run Kruskal's | Can also use Prim's — pick nearest unconnected point greedily |
| [1135](https://leetcode.com/problems/connecting-cities-with-minimum-cost/) | Connecting Cities (Premium) | Direct Kruskal's application | — |

---

## 5. Eulerian Path / Hierholzer's Algorithm

### Identify
- "Use every EDGE exactly once" (Eulerian path)
- "Reconstruct itinerary" (visit every flight exactly once)
- Eulerian circuit: every node has even degree
- Eulerian path: exactly 2 nodes with odd degree

### Template (Hierholzer's — DFS, push to result after exhausting neighbors)
```cpp
// Reconstruct itinerary: nodes = airports, edges = flights
// Must use every edge exactly once, lexicographically smallest path
unordered_map<string, priority_queue<string, vector<string>, greater<>>> graph;
vector<string> result;

void dfs(string airport) {
    while (!graph[airport].empty()) {
        string next = graph[airport].top();
        graph[airport].pop();
        dfs(next);
    }
    result.push_back(airport);  // push AFTER exhausting all edges
}
// Reverse result at end
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [332](https://leetcode.com/problems/reconstruct-itinerary/) | Reconstruct Itinerary | Hierholzer's DFS. Use min-heap per node for lexicographic order. Push to result after exhausting | Result is reversed — reverse at end |
| [753](https://leetcode.com/problems/cracking-the-safe/) | Cracking the Safe | Eulerian circuit on de Bruijn graph | Hard — each node = (n-1)-length password prefix |

---

## 6. Strongly Connected Components (SCC)

### Identify
- "Find groups where every node can reach every other node (in directed graph)"
- Rarely asked directly, but concept underlies some problems

### Kosaraju's (Two-Pass DFS)
```cpp
// Pass 1: DFS on original graph, push to stack by finish time
// Pass 2: DFS on reversed graph in reverse finish order
// Each DFS tree in pass 2 = one SCC
```

### Problems

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1192](https://leetcode.com/problems/critical-connections-in-a-network/) | Critical Connections (Bridges) | Tarjan's bridge-finding algorithm | DFS with discovery time and low link values |

---

## When to Use Each

```
Bipartite:     2-color check. "Split into 2 groups with no intra-group edges"
Bellman-Ford:  negative edges OR k-step constraint (K Stops problem)
Floyd-Warshall: all-pairs shortest path. Small graph (n≤200). O(n³)
Kruskal's MST: sparse graph, edges given explicitly, sort + DSU
Prim's MST:    dense graph, adjacency list given, min-heap
Eulerian Path: "use every edge once" → Hierholzer's DFS
```

## Common Mistakes

- **Bipartite:** Forgetting to loop over all nodes as start — disconnected graph means some nodes never get colored
- **Bellman-Ford for #787:** Not copying `dist` before each round — chains updates incorrectly
- **MST:** Applying MST to directed graph. Kruskal's/Prim's are for UNDIRECTED graphs only
- **Hierholzer's:** Pushing the node to result BEFORE recursing instead of AFTER. Must exhaust all edges first.
