# Pattern 5: Dijkstra / Weighted Shortest Path

## Identify This Pattern
- "Minimum COST / WEIGHT to reach destination"
- "Cheapest flights", "network delay", "minimum effort"
- Graph has weighted edges (not just hop count)
- All edge weights are NON-NEGATIVE (if negative → Bellman-Ford)

## Recognition Signal
> BFS gives shortest PATH (hops). Dijkstra gives shortest COST (weighted).
> If problem says "cost", "weight", "time", "distance" between nodes → Dijkstra.

---

## Templates

### Standard Dijkstra
```cpp
vector<int> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    // adj[u] = {{v, weight}, ...}
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq; // min-heap
    dist[src] = 0;
    pq.push({0, src});   // {cost, node}

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;   // stale entry — skip

        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;  // dist[i] = min cost from src to i, INT_MAX if unreachable
}
```

### Dijkstra with State (node + extra constraint)
```cpp
// When state is (node, stops_remaining) or (node, fuel_left) etc.
// dist[node][state] = min cost
priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
// {cost, node, extra_state}
vector<vector<int>> dist(n, vector<int>(maxState+1, INT_MAX));
dist[src][initialState] = 0;
pq.push({0, src, initialState});

while (!pq.empty()) {
    auto [cost, u, state] = pq.top(); pq.pop();
    if (cost > dist[u][state]) continue;
    if (u == dst) return cost;
    // ... expand neighbors with updated state
}
```

### Dijkstra on 2D Grid
```cpp
// dist[r][c] = min cost to reach cell (r,c)
vector<vector<int>> dist(m, vector<int>(n, INT_MAX));
priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
dist[0][0] = 0;
pq.push({0, 0, 0});  // {cost, row, col}

while (!pq.empty()) {
    auto [cost, r, c] = pq.top(); pq.pop();
    if (cost > dist[r][c]) continue;
    if (r == m-1 && c == n-1) return cost;  // reached destination

    for (int d = 0; d < 4; d++) {
        int nr = r + dr[d], nc = c + dc[d];
        if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
        int newCost = /* cost function */;
        if (newCost < dist[nr][nc]) {
            dist[nr][nc] = newCost;
            pq.push({newCost, nr, nc});
        }
    }
}
```

---

## Problems

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [743](https://leetcode.com/problems/network-delay-time/) | Network Delay Time | Dijkstra from source k. Answer = max of all dist[], -1 if any unreachable | Foundation Dijkstra problem |
| [1631](https://leetcode.com/problems/path-with-minimum-effort/) | Path with Minimum Effort | Dijkstra on grid. Edge weight = abs difference between neighbor cells | Cost = max effort on path. Use `max(curCost, abs(diff))` not sum |
| [1514](https://leetcode.com/problems/path-with-maximum-probability/) | Path with Max Probability | Dijkstra variant with max instead of min. Use max-heap | dist = max probability. Relax: `dist[v] = max(dist[v], dist[u] * prob)` |
| [1976](https://leetcode.com/problems/number-of-ways-to-arrive-at-destination/) | Number of Ways to Arrive | Dijkstra + count paths with same shortest distance | Track `ways[v]`: reset if shorter found, add if equal |
| [778](https://leetcode.com/problems/swim-in-rising-water/) | Swim in Rising Water | Dijkstra on grid: edge weight = max(cur, neighbor). Reach (n-1,n-1) | Same as #1631 but different cost function. Also solvable with binary search |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [787](https://leetcode.com/problems/cheapest-flights-within-k-stops/) | Cheapest Flights K Stops | Dijkstra with state (node, stops_used). OR Bellman-Ford with k iterations | Dijkstra: `dist[node][stops]`. BF: relax edges exactly k+1 times |
| [1293](https://leetcode.com/problems/shortest-path-in-a-grid-with-obstacles-elimination/) | Shortest Path with Obstacle Elimination | BFS with state (row, col, k_remaining). State space = m*n*k | BFS ensures minimum steps. State must include remaining eliminations |
| [882](https://leetcode.com/problems/reachable-nodes-in-subdivided-graph/) | Reachable Nodes in Subdivided Graph | Dijkstra to find how many sub-nodes each edge can "use" | Complex but pure Dijkstra |

---

## Key Distinctions

### BFS vs Dijkstra
```
BFS:      all edges have equal weight (1 hop = 1 cost)
Dijkstra: edges have different weights (non-negative)

If all weights = 1 → BFS is faster (O(V+E) vs O((V+E)logV))
If weights differ → must use Dijkstra (BFS gives wrong answer)
```

### Dijkstra vs Bellman-Ford
```
Dijkstra:     non-negative weights only. O((V+E) log V)
Bellman-Ford: handles negative weights. O(V*E). Slower.

For #787 (K stops): both work. Bellman-Ford (k+1 iterations) is simpler to code.
```

### The "stale entry" check
```cpp
if (d > dist[u]) continue;  // This line is critical
```
Why: Dijkstra uses a lazy deletion min-heap. When we find a shorter path to u, we push a new entry but don't remove the old one. When the old entry is popped, its distance is already stale — we skip it.

### Cost function variations
```
Network delay (#743):  cost = sum of edge weights (standard)
Min effort (#1631):    cost = max of all edge differences (bottleneck)
Max probability (#1514): cost = product of probabilities (use max-heap)
Swim in water (#778):  cost = max of all cell values on path (bottleneck)
```

---

## Common Mistakes

- Using a max-heap instead of min-heap. In C++: `priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>>` for min-heap.
- Forgetting the stale entry check `if (d > dist[u]) continue` — leads to processing same node multiple times and TLE.
- For grid Dijkstra: initializing `dist` with INT_MAX and trying to add to it → overflow. Use large but safe value like 1e9.
- For #787 with k stops: off-by-one on stops. "k stops" means k+1 edges.

## Follow-up Interviewers Love

- "What if there are negative edges?" → Bellman-Ford
- "What's Dijkstra's time complexity?" → O((V+E) log V) with binary heap
- "Can you use Dijkstra if some edges are 0?" → Yes, 0 is non-negative. Works fine.
- "For #1631, why max() and not sum()?" → Problem asks for minimum of the maximum effort — it's a bottleneck problem, not total cost
- "What if we want all pairs shortest paths?" → Floyd-Warshall O(V³) or run Dijkstra from each node O(V * (V+E)logV)
