# Graph Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Explore all nodes / find connected components?     → DFS or BFS
Shortest path in UNWEIGHTED graph?                 → BFS
Shortest path in WEIGHTED graph (no neg edges)?    → Dijkstra
Shortest path with NEGATIVE edges?                 → Bellman-Ford
Shortest path between ALL pairs?                   → Floyd-Warshall
Ordering tasks with dependencies?                  → Topological Sort
Detect cycle in directed graph?                    → DFS (color states)
Detect cycle in undirected graph?                  → Union-Find or DFS
Connect components / check if same group?          → Union-Find (DSU)
Minimum cost to connect all nodes?                 → MST (Kruskal / Prim)
Grid problem (islands, paths, regions)?            → BFS/DFS on grid
Check if graph is bipartite?                       → BFS/DFS 2-coloring
```

**The two questions to always ask first:**
> 1. "Is the graph directed or undirected?"
> 2. "Are edges weighted or unweighted?"
> These two answers immediately cut the pattern choices in half.

---

## C++ Graph Quick Reference

### Graph Representations
```cpp
int n = 5;  // number of nodes

// 1. ADJACENCY LIST (most common — use this by default)
vector<vector<int>> adj(n);
adj[u].push_back(v);              // directed
adj[u].push_back(v); adj[v].push_back(u);  // undirected

// 2. WEIGHTED ADJACENCY LIST
vector<vector<pair<int,int>>> adj(n);   // {neighbor, weight}
adj[u].push_back({v, w});

// 3. ADJACENCY MATRIX (dense graphs / quick edge lookup)
vector<vector<int>> mat(n, vector<int>(n, 0));
mat[u][v] = 1;  // or weight

// 4. EDGE LIST (Kruskal's MST)
vector<tuple<int,int,int>> edges;   // {weight, u, v}
edges.push_back({w, u, v});

// Visited array — always needed
vector<bool> visited(n, false);
// For directed graphs — 3 colors
vector<int> color(n, 0);  // 0=unvisited, 1=in-stack, 2=done
```

---

## Pattern 1: DFS on Graph

**When to think about it:**
- Explore all reachable nodes from a source
- Find connected components
- Detect cycles in directed/undirected graph
- Topological sort (DFS-based)
- Any "flood fill" style problem

**Core insight:** Go as deep as possible before backtracking.
Mark node visited BEFORE exploring neighbors (prevents infinite loops in cycles).

```
Graph: 0-1-2
       |   |
       3---4

DFS from 0: visit 0 -> 1 -> 2 -> 4 -> 3 (backtrack up)
```

### Template: DFS Iterative + Recursive
```cpp
// RECURSIVE DFS
void dfs(int node, vector<vector<int>>& adj, vector<bool>& visited) {
    visited[node] = true;
    // process node here

    for (int neighbor : adj[node]) {
        if (!visited[neighbor])
            dfs(neighbor, adj, visited);
    }
}

// ITERATIVE DFS (stack)
void dfsIterative(int start, vector<vector<int>>& adj, vector<bool>& visited) {
    stack<int> st;
    st.push(start);

    while (!st.empty()) {
        int node = st.top(); st.pop();
        if (visited[node]) continue;    // skip already visited
        visited[node] = true;
        // process node here

        for (int neighbor : adj[node]) {
            if (!visited[neighbor])
                st.push(neighbor);
        }
    }
}

// COUNT CONNECTED COMPONENTS
int countComponents(int n, vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    int count = 0;

    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(i, adj, visited);
            count++;
        }
    }
    return count;
}
```

### Cycle Detection — Undirected Graph (DFS)
```cpp
bool hasCycleUndirected(int node, int parent,
                         vector<vector<int>>& adj, vector<bool>& visited) {
    visited[node] = true;

    for (int neighbor : adj[node]) {
        if (!visited[neighbor]) {
            if (hasCycleUndirected(neighbor, node, adj, visited))
                return true;
        } else if (neighbor != parent) {   // visited but not parent = cycle
            return true;
        }
    }
    return false;
}
```

### Cycle Detection — Directed Graph (3-color DFS)
```cpp
// 0 = unvisited, 1 = in current path (gray), 2 = fully processed (black)
bool hasCycleDirected(int node, vector<vector<int>>& adj, vector<int>& color) {
    color[node] = 1;   // mark as in-progress

    for (int neighbor : adj[node]) {
        if (color[neighbor] == 1) return true;   // back edge = cycle
        if (color[neighbor] == 0)
            if (hasCycleDirected(neighbor, adj, color)) return true;
    }

    color[node] = 2;   // fully processed
    return false;
}
```

**Problems:** #200 Number of Islands, #547 Number of Provinces, #695 Max Area of Island, #133 Clone Graph, #417 Pacific Atlantic Water Flow

---

## Pattern 2: BFS on Graph

**When to think about it:**
- Shortest path in an **unweighted** graph (BFS guarantees minimum hops)
- Level-by-level exploration
- "Minimum steps", "minimum distance", "minimum moves"
- Multi-source BFS (start from multiple nodes simultaneously)

**Core insight:** BFS explores nodes in order of their distance from source.
First time you reach a node = shortest path to it. Never revisit.

```
Graph: 0 - 1 - 3
       |       |
       2 - - - 4

BFS from 0:
Level 0: [0]
Level 1: [1, 2]       distance 1
Level 2: [3, 4]       distance 2
```

### Template: BFS Shortest Path
```cpp
int bfsShortestPath(int start, int end, vector<vector<int>>& adj, int n) {
    vector<bool> visited(n, false);
    queue<int> q;
    q.push(start);
    visited[start] = true;
    int dist = 0;

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            int node = q.front(); q.pop();
            if (node == end) return dist;

            for (int neighbor : adj[node]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    q.push(neighbor);
                }
            }
        }
        dist++;
    }
    return -1;  // not reachable
}
```

### Multi-Source BFS (start from all sources at once)
```cpp
// Example: 0-1 matrix — shortest distance from any 0 for each cell
vector<vector<int>> updateMatrix(vector<vector<int>>& mat) {
    int m = mat.size(), n = mat[0].size();
    vector<vector<int>> dist(m, vector<int>(n, INT_MAX));
    queue<pair<int,int>> q;

    // Push ALL sources (0-cells) into queue at the start
    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (mat[i][j] == 0) { dist[i][j] = 0; q.push({i, j}); }

    int dr[] = {-1, 1, 0, 0};
    int dc[] = {0, 0, -1, 1};

    while (!q.empty()) {
        auto [r, c] = q.front(); q.pop();
        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr >= 0 && nr < m && nc >= 0 && nc < n &&
                dist[nr][nc] > dist[r][c] + 1) {
                dist[nr][nc] = dist[r][c] + 1;
                q.push({nr, nc});
            }
        }
    }
    return dist;
}
```

### Word Ladder (BFS on implicit graph)
```cpp
int ladderLength(string begin, string end, vector<string>& wordList) {
    unordered_set<string> wordSet(wordList.begin(), wordList.end());
    if (!wordSet.count(end)) return 0;

    queue<string> q;
    q.push(begin);
    int steps = 1;

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            string word = q.front(); q.pop();
            if (word == end) return steps;

            for (int j = 0; j < word.size(); j++) {
                char orig = word[j];
                for (char c = 'a'; c <= 'z'; c++) {
                    word[j] = c;
                    if (wordSet.count(word)) {
                        q.push(word);
                        wordSet.erase(word);   // remove to avoid revisiting
                    }
                }
                word[j] = orig;
            }
        }
        steps++;
    }
    return 0;
}
```

**Problems:** #994 Rotting Oranges, #127 Word Ladder, #542 01 Matrix, #1091 Shortest Path in Binary Matrix, #433 Minimum Genetic Mutation

---

## Pattern 3: Topological Sort

**When to think about it:**
- "Order tasks given dependencies"
- "Is there a valid ordering / detect cycle in directed graph?"
- "Course schedule" — can you finish all courses?
- Only works on **Directed Acyclic Graphs (DAGs)**

**Core insight:** A node can be "processed" only after ALL its dependencies are done.
Two approaches: Kahn's (BFS, intuitive) or DFS (postorder, elegant).

```
Dependencies: A->C, B->C, C->D
Valid order: A, B, C, D  (or B, A, C, D)
```

### Kahn's Algorithm (BFS — recommended, easier to understand)
```cpp
vector<int> topoSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : adj[u]) indegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);   // start with nodes that have no deps

    vector<int> order;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        order.push_back(node);

        for (int neighbor : adj[node]) {
            if (--indegree[neighbor] == 0)  // one less dependency
                q.push(neighbor);
        }
    }

    // If order.size() != n, there's a cycle (not a DAG)
    return order.size() == n ? order : vector<int>{};
}

// Course Schedule (can you finish all courses?)
bool canFinish(int n, vector<vector<int>>& prerequisites) {
    vector<vector<int>> adj(n);
    vector<int> indegree(n, 0);

    for (auto& p : prerequisites) {
        adj[p[1]].push_back(p[0]);
        indegree[p[0]]++;
    }

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);

    int processed = 0;
    while (!q.empty()) {
        int node = q.front(); q.pop();
        processed++;
        for (int next : adj[node])
            if (--indegree[next] == 0) q.push(next);
    }
    return processed == n;   // all courses processable = no cycle
}
```

### DFS-Based Topological Sort
```cpp
void dfsTopo(int node, vector<vector<int>>& adj,
             vector<bool>& visited, stack<int>& st) {
    visited[node] = true;
    for (int neighbor : adj[node])
        if (!visited[neighbor])
            dfsTopo(neighbor, adj, visited, st);
    st.push(node);   // push AFTER all neighbors processed (postorder)
}

vector<int> topoSortDFS(int n, vector<vector<int>>& adj) {
    vector<bool> visited(n, false);
    stack<int> st;

    for (int i = 0; i < n; i++)
        if (!visited[i]) dfsTopo(i, adj, visited, st);

    vector<int> order;
    while (!st.empty()) { order.push_back(st.top()); st.pop(); }
    return order;
}
```

**Kahn's vs DFS:**
```
Kahn's (BFS): easier to detect cycle (check processed count)
DFS:          elegant, naturally gives reverse postorder
Use Kahn's in interviews — cleaner logic
```

**Problems:** #207 Course Schedule, #210 Course Schedule II, #269 Alien Dictionary, #310 Minimum Height Trees, #2115 Find All Possible Recipes

---

## Pattern 4: Union-Find (Disjoint Set Union — DSU)

**When to think about it:**
- "Are these two nodes connected / in the same group?"
- "How many connected components?"
- "Detect cycle in undirected graph"
- "Minimum spanning tree" (Kruskal's uses DSU)
- Dynamic connectivity: edges added one by one, query connectivity

**Core insight:** Each component has a "representative" (root). Two nodes are in the
same component iff they have the same root. Union merges two components,
Find returns the root.

```
Initially: {0} {1} {2} {3} {4}

union(0,1): {0,1} {2} {3} {4}
union(1,2): {0,1,2} {3} {4}
union(3,4): {0,1,2} {3,4}

find(0) == find(2)?  YES — same component
find(0) == find(3)?  NO  — different component
```

### DSU Template (with path compression + union by rank)
```cpp
struct DSU {
    vector<int> parent, rank;

    DSU(int n) : parent(n), rank(n, 0) {
        iota(parent.begin(), parent.end(), 0);  // parent[i] = i
    }

    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);   // path compression
        return parent[x];
    }

    bool unite(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;         // already same component

        // Union by rank — attach smaller tree under larger
        if (rank[px] < rank[py]) swap(px, py);
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
        return true;
    }

    bool connected(int x, int y) {
        return find(x) == find(y);
    }
};

// Number of Connected Components
int countComponents(int n, vector<vector<int>>& edges) {
    DSU dsu(n);
    int components = n;
    for (auto& e : edges)
        if (dsu.unite(e[0], e[1])) components--;
    return components;
}

// Detect Cycle (undirected) — if same component before uniting, cycle exists
bool hasCycleDSU(int n, vector<vector<int>>& edges) {
    DSU dsu(n);
    for (auto& e : edges)
        if (!dsu.unite(e[0], e[1])) return true;  // unite failed = same component = cycle
    return false;
}
```

**Problems:** #547 Number of Provinces, #684 Redundant Connection, #1319 Make Network Connected, #990 Satisfiability of Equality Equations, #721 Accounts Merge, #778 Swim in Rising Water

---

## Pattern 5: Dijkstra's Algorithm (Weighted Shortest Path)

**When to think about it:**
- Shortest path in a weighted graph with **non-negative** edge weights
- "Minimum cost to reach destination"
- "Cheapest flights", "network delay time"
- Greedy: always process the cheapest unvisited node next

**Core insight:** Use a min-heap (priority queue). Always expand the node with
the smallest known distance. Once a node is popped, its distance is finalized.

```
Graph: 0 --1-- 1 --4-- 3
       |               |
       2               |
       |               |
       2 --1-- 3 --1-- (same 3)

Dijkstra from 0:
dist[0]=0, dist[1]=1, dist[2]=2, dist[3]=3 (via 0->2->3, not 0->1->3)
```

### Dijkstra Template
```cpp
vector<int> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;  // min-heap

    dist[src] = 0;
    pq.push({0, src});   // {distance, node}

    while (!pq.empty()) {
        auto [d, node] = pq.top(); pq.pop();

        if (d > dist[node]) continue;   // stale entry, skip

        for (auto [neighbor, weight] : adj[node]) {
            if (dist[node] + weight < dist[neighbor]) {
                dist[neighbor] = dist[node] + weight;
                pq.push({dist[neighbor], neighbor});
            }
        }
    }
    return dist;  // dist[i] = shortest distance from src to i
}

// Network Delay Time (#743)
int networkDelayTime(vector<vector<int>>& times, int n, int k) {
    vector<vector<pair<int,int>>> adj(n + 1);
    for (auto& t : times) adj[t[0]].push_back({t[1], t[2]});

    vector<int> dist = dijkstra(k, n + 1, adj);

    int ans = *max_element(dist.begin() + 1, dist.end());
    return ans == INT_MAX ? -1 : ans;
}
```

### Dijkstra with State (path + constraint)
```cpp
// Cheapest Flights Within K Stops (#787)
int findCheapestPrice(int n, vector<vector<int>>& flights, int src, int dst, int k) {
    vector<vector<pair<int,int>>> adj(n);
    for (auto& f : flights) adj[f[0]].push_back({f[1], f[2]});

    // {cost, node, stops_remaining}
    priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
    pq.push({0, src, k});

    vector<vector<int>> dist(n, vector<int>(k + 2, INT_MAX));
    dist[src][k] = 0;

    while (!pq.empty()) {
        auto [cost, node, stops] = pq.top(); pq.pop();
        if (node == dst) return cost;
        if (stops == 0) continue;

        for (auto [next, price] : adj[node]) {
            int newCost = cost + price;
            if (newCost < dist[next][stops - 1]) {
                dist[next][stops - 1] = newCost;
                pq.push({newCost, next, stops - 1});
            }
        }
    }
    return -1;
}
```

**Problems:** #743 Network Delay Time, #787 Cheapest Flights K Stops, #1631 Path with Min Effort, #778 Swim in Rising Water, #1514 Path with Max Probability

---

## Pattern 6: Bellman-Ford (Negative Edges / Negative Cycle Detection)

**When to think about it:**
- Graph has **negative edge weights**
- Need to detect **negative cycles**
- Simpler than Dijkstra when graph is small or edge list form

**Core insight:** Relax ALL edges n-1 times. After n-1 iterations, all shortest
paths are found. If you can still relax on the nth iteration → negative cycle exists.

```cpp
vector<int> bellmanFord(int src, int n, vector<tuple<int,int,int>>& edges) {
    vector<int> dist(n, INT_MAX);
    dist[src] = 0;

    // Relax all edges n-1 times
    for (int i = 0; i < n - 1; i++) {
        for (auto [u, v, w] : edges) {
            if (dist[u] != INT_MAX && dist[u] + w < dist[v])
                dist[v] = dist[u] + w;
        }
    }

    // nth iteration: if still relaxing -> negative cycle
    for (auto [u, v, w] : edges) {
        if (dist[u] != INT_MAX && dist[u] + w < dist[v])
            return {};   // negative cycle detected
    }

    return dist;
}
```

**Problems:** #787 Cheapest Flights (BF version), #743 Network Delay Time (BF version)

---

## Pattern 7: Floyd-Warshall (All-Pairs Shortest Path)

**When to think about it:**
- Need shortest path between **ALL pairs** of nodes
- Small graph (n ≤ 200 typically)
- "Find if path exists between any pair", "transitive closure"

**Core insight:** For each intermediate node k, check if path i→k→j is shorter
than the current best i→j. Three nested loops.

```cpp
void floydWarshall(vector<vector<int>>& dist, int n) {
    // dist[i][j] = edge weight, INF if no edge, 0 if i==j

    for (int k = 0; k < n; k++)           // intermediate node
        for (int i = 0; i < n; i++)
            for (int j = 0; j < n; j++)
                if (dist[i][k] != INT_MAX && dist[k][j] != INT_MAX)
                    dist[i][j] = min(dist[i][j], dist[i][k] + dist[k][j]);
}
```

**Problems:** #1334 Find City with Fewest Neighbors, #743 (multi-source variant), #399 Evaluate Division

---

## Pattern 8: Minimum Spanning Tree (MST)

**When to think about it:**
- "Connect all nodes with minimum total edge cost"
- "Minimum cost to build roads / cables / network"
- Result: n-1 edges connecting all n nodes with minimum weight

### Kruskal's Algorithm (sort edges + DSU)
```cpp
int kruskalMST(int n, vector<tuple<int,int,int>>& edges) {
    sort(edges.begin(), edges.end());  // sort by weight
    DSU dsu(n);
    int totalCost = 0, edgesUsed = 0;

    for (auto [w, u, v] : edges) {
        if (dsu.unite(u, v)) {       // add edge if it connects two components
            totalCost += w;
            if (++edgesUsed == n - 1) break;  // MST complete
        }
    }
    return edgesUsed == n - 1 ? totalCost : -1;  // -1 if not connected
}
```

### Prim's Algorithm (greedy, min-heap)
```cpp
int primMST(int n, vector<vector<pair<int,int>>>& adj) {
    vector<bool> inMST(n, false);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    pq.push({0, 0});   // {weight, node}
    int totalCost = 0;

    while (!pq.empty()) {
        auto [w, node] = pq.top(); pq.pop();
        if (inMST[node]) continue;
        inMST[node] = true;
        totalCost += w;

        for (auto [next, weight] : adj[node])
            if (!inMST[next]) pq.push({weight, next});
    }
    return totalCost;
}
```

**Kruskal's vs Prim's:**
```
Kruskal's: sort edges + DSU  — better for sparse graphs
Prim's:    min-heap           — better for dense graphs
Both give the same MST weight. Kruskal's is easier to code in interviews.
```

**Problems:** #1584 Min Cost to Connect All Points, #1135 Connecting Cities with Min Cost, #1168 Optimize Water Distribution

---

## Pattern 9: Bipartite Check (2-Coloring)

**When to think about it:**
- "Can you divide nodes into two groups with no edges within a group?"
- "Is this graph bipartite?"
- Equivalent: does the graph contain an ODD-length cycle? If yes → not bipartite

**Core insight:** Try to 2-color the graph. Assign alternating colors via BFS/DFS.
If you find two adjacent nodes with the same color → not bipartite.

```cpp
bool isBipartite(vector<vector<int>>& graph) {
    int n = graph.size();
    vector<int> color(n, -1);

    for (int start = 0; start < n; start++) {
        if (color[start] != -1) continue;

        queue<int> q;
        q.push(start);
        color[start] = 0;

        while (!q.empty()) {
            int node = q.front(); q.pop();
            for (int neighbor : graph[node]) {
                if (color[neighbor] == -1) {
                    color[neighbor] = 1 - color[node];   // flip color
                    q.push(neighbor);
                } else if (color[neighbor] == color[node]) {
                    return false;   // same color = not bipartite
                }
            }
        }
    }
    return true;
}
```

**Problems:** #785 Is Graph Bipartite, #886 Possible Bipartition

---

## Pattern 10: Grid as Graph (Islands Pattern)

**When to think about it:**
- 2D grid with cells that are 0/1, 'W'/'L', or similar
- "Count islands", "max area", "flood fill", "surrounded regions"
- Each cell is a node, edges connect adjacent cells (4-directional or 8-directional)

**Core insight:** Treat the grid as an implicit graph. DFS/BFS from unvisited cells.
Mark visited by modifying the grid itself (common trick) or use a visited array.

### Universal Grid DFS Template
```cpp
int dr[] = {-1, 1, 0, 0};   // 4-directional
int dc[] = {0, 0, -1, 1};
// For 8-directional add: {-1,-1},{-1,1},{1,-1},{1,1}

void dfs(vector<vector<char>>& grid, int r, int c) {
    int m = grid.size(), n = grid[0].size();
    if (r < 0 || r >= m || c < 0 || c >= n) return;  // out of bounds
    if (grid[r][c] != '1') return;                    // not land or visited

    grid[r][c] = '0';   // mark visited (in-place)

    for (int d = 0; d < 4; d++)
        dfs(grid, r + dr[d], c + dc[d]);
}

int numIslands(vector<vector<char>>& grid) {
    int count = 0;
    for (int i = 0; i < grid.size(); i++)
        for (int j = 0; j < grid[0].size(); j++)
            if (grid[i][j] == '1') { dfs(grid, i, j); count++; }
    return count;
}
```

### Surrounded Regions (reverse thinking — save border-connected cells)
```cpp
void solve(vector<vector<char>>& board) {
    int m = board.size(), n = board[0].size();

    // DFS from all border 'O' cells and mark them safe
    function<void(int,int)> dfs = [&](int r, int c) {
        if (r < 0 || r >= m || c < 0 || c >= n || board[r][c] != 'O') return;
        board[r][c] = 'S';   // safe
        dfs(r+1,c); dfs(r-1,c); dfs(r,c+1); dfs(r,c-1);
    };

    for (int i = 0; i < m; i++) { dfs(i, 0); dfs(i, n-1); }
    for (int j = 0; j < n; j++) { dfs(0, j); dfs(m-1, j); }

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            board[i][j] = (board[i][j] == 'S') ? 'O' : 'X';
}
```

**Problems:** #200 Number of Islands, #695 Max Area of Island, #130 Surrounded Regions, #994 Rotting Oranges, #417 Pacific Atlantic, #1020 Number of Enclaves, #827 Making a Large Island

---

## The 6 Most Important Templates to Memorize

```cpp
// 1. DFS (connected components, cycle, flood fill)
void dfs(int node, vector<vector<int>>& adj, vector<bool>& vis) {
    vis[node] = true;
    for (int nb : adj[node])
        if (!vis[nb]) dfs(nb, adj, vis);
}

// 2. BFS (unweighted shortest path)
int bfs(int src, int dst, vector<vector<int>>& adj, int n) {
    vector<bool> vis(n, false);
    queue<int> q;
    q.push(src); vis[src] = true;
    int dist = 0;
    while (!q.empty()) {
        for (int sz = q.size(); sz--; ) {
            int node = q.front(); q.pop();
            if (node == dst) return dist;
            for (int nb : adj[node]) if (!vis[nb]) { vis[nb]=true; q.push(nb); }
        }
        dist++;
    }
    return -1;
}

// 3. TOPOLOGICAL SORT — KAHN'S (dependency ordering)
vector<int> topo(int n, vector<vector<int>>& adj) {
    vector<int> indeg(n, 0);
    for (int u = 0; u < n; u++) for (int v : adj[u]) indeg[v]++;
    queue<int> q;
    for (int i = 0; i < n; i++) if (!indeg[i]) q.push(i);
    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop(); order.push_back(u);
        for (int v : adj[u]) if (!--indeg[v]) q.push(v);
    }
    return order.size() == n ? order : vector<int>{};
}

// 4. UNION-FIND (DSU)
struct DSU {
    vector<int> p, r;
    DSU(int n) : p(n), r(n, 0) { iota(p.begin(), p.end(), 0); }
    int find(int x) { return p[x] == x ? x : p[x] = find(p[x]); }
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (r[x] < r[y]) swap(x, y);
        p[y] = x; if (r[x] == r[y]) r[x]++;
        return true;
    }
};

// 5. DIJKSTRA (weighted shortest path, non-negative)
vector<int> dijkstra(int src, int n, vector<vector<pair<int,int>>>& adj) {
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    dist[src] = 0; pq.push({0, src});
    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;
        for (auto [v, w] : adj[u])
            if (dist[u] + w < dist[v]) { dist[v] = dist[u]+w; pq.push({dist[v],v}); }
    }
    return dist;
}

// 6. GRID DFS (islands / flood fill)
int dr[] = {-1,1,0,0}, dc[] = {0,0,-1,1};
void gridDFS(vector<vector<char>>& g, int r, int c) {
    if (r<0||r>=(int)g.size()||c<0||c>=(int)g[0].size()||g[r][c]!='1') return;
    g[r][c] = '0';
    for (int d=0;d<4;d++) gridDFS(g, r+dr[d], c+dc[d]);
}
```

---

## Universal Checklist Before Coding

- [ ] Directed or undirected? Weighted or unweighted?
- [ ] Build adjacency list first, not matrix (unless dense graph)
- [ ] Need `visited[]` array — always. Mark BEFORE pushing to queue/stack
- [ ] Disconnected graph? → loop over ALL nodes as potential start
- [ ] Cycle detection: undirected → track parent; directed → 3-color DFS
- [ ] Shortest path: unweighted → BFS; weighted → Dijkstra; negative → Bellman-Ford
- [ ] Dependency order → Topological sort (Kahn's)
- [ ] Dynamic connectivity → DSU
- [ ] Trace through a small example (5 nodes, 5 edges) before coding

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 200  | Number of Islands                   | Medium | Grid DFS/BFS             |
| 2  | 133  | Clone Graph                         | Medium | DFS/BFS traversal        |
| 3  | 207  | Course Schedule                     | Medium | Topological sort         |
| 4  | 210  | Course Schedule II                  | Medium | Topological sort (order) |
| 5  | 547  | Number of Provinces                 | Medium | DFS / DSU                |
| 6  | 994  | Rotting Oranges                     | Medium | Multi-source BFS         |
| 7  | 417  | Pacific Atlantic Water Flow         | Medium | Multi-source DFS/BFS     |
| 8  | 785  | Is Graph Bipartite                  | Medium | BFS 2-coloring           |
| 9  | 743  | Network Delay Time                  | Medium | Dijkstra                 |
| 10 | 684  | Redundant Connection                | Medium | DSU cycle detection      |
| 11 | 695  | Max Area of Island                  | Medium | Grid DFS                 |
| 12 | 130  | Surrounded Regions                  | Medium | Boundary DFS             |
| 13 | 127  | Word Ladder                         | Hard   | BFS implicit graph       |
| 14 | 787  | Cheapest Flights Within K Stops     | Medium | Dijkstra with state      |
| 15 | 1584 | Min Cost to Connect All Points      | Medium | MST (Kruskal/Prim)       |
| 16 | 269  | Alien Dictionary                    | Hard   | Topological sort         |
| 17 | 721  | Accounts Merge                      | Medium | DSU                      |
| 18 | 827  | Making a Large Island               | Hard   | Grid DFS + DSU           |
| 19 | 1631 | Path with Minimum Effort            | Medium | Dijkstra on grid         |
| 20 | 310  | Minimum Height Trees                | Medium | Topological sort (trim)  |
| 21 | 1319 | Number of Operations to Make Connected | Medium | DSU                   |
| 22 | 542  | 01 Matrix                           | Medium | Multi-source BFS         |
| 23 | 332  | Reconstruct Itinerary               | Hard   | DFS + Eulerian path      |
| 24 | 1091 | Shortest Path in Binary Matrix      | Medium | BFS                      |
| 25 | 399  | Evaluate Division                   | Medium | BFS on weighted graph    |

**Study order:** #200 -> #207 -> #994 -> #547 -> #785 -> #210 -> #743 -> #684 -> #417 -> #1584
These ten cover every core pattern.

---

## How Patterns Combine (Hard Problems)

| Problem | Patterns Combined |
|---------|------------------|
| #417 Pacific Atlantic | Multi-source DFS from both oceans |
| #827 Making a Large Island | Grid DFS (label) + DSU (merge) |
| #269 Alien Dictionary | BFS topological sort + string parsing |
| #787 Cheapest Flights K Stops | Dijkstra + state (stops remaining) |
| #1631 Path with Min Effort | Dijkstra on grid |
| #332 Reconstruct Itinerary | DFS + Eulerian path (Hierholzer's) |
| #721 Accounts Merge | DSU + HashMap |

---

## Company-Wise Most Asked Problems

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #200 Islands, #207 Course Schedule, #994 Rotting Oranges, #547 Provinces, #127 Word Ladder |
| **Google** | #207/210 Course Schedule, #743 Network Delay, #269 Alien Dict, #332 Itinerary, #1584 MST |
| **Meta** | #200 Islands, #133 Clone Graph, #785 Bipartite, #417 Pacific Atlantic, #1091 Shortest Path |
| **Microsoft** | #200 Islands, #207 Course Schedule, #130 Surrounded Regions, #994 Rotting Oranges |
| **Bloomberg** | #200 Islands, #207 Course Schedule, #542 01 Matrix, #684 Redundant Connection |
| **Uber** | #743 Network Delay, #787 Cheapest Flights, #332 Itinerary |
| **Airbnb** | #207 Course Schedule, #269 Alien Dictionary |

---

## Recent Interview Trends (2023–2025)

### 1. Grid/Island problems are the most asked at all levels
#200, #695, #994, #130, #417 — companies use grid problems to test BFS/DFS without
requiring knowledge of graph theory. If you know grid DFS cold, you handle ~30% of graph questions.

### 2. Topological sort appears at every company
Course Schedule (#207/#210) is consistently in the top-5 most asked graph problems
at Amazon, Google, Meta. Know Kahn's algorithm — it's cleaner and cycle detection is built-in.

### 3. Dijkstra is now expected at mid-level interviews
Previously a "senior" topic, Dijkstra is now asked at SDE-2 level. The pattern:
build adj list → min-heap → relax edges. Grid + Dijkstra (#1631) is a common combo.

### 4. DSU is underrated but frequently asked
Redundant Connection (#684), Accounts Merge (#721), and MST problems (#1584) all use DSU.
Interviewers like it because it tests both data structure design and graph thinking.

### 5. "Implicit graph" BFS is a rising trend
Word Ladder (#127), Gene Mutation (#433), Knight moves — the graph isn't given to you,
you generate neighbors on-the-fly. Recognize this when the problem says "one transformation at a time."

### 6. Multi-source BFS is heavily tested at Amazon/Meta
Rotting Oranges (#994), 01 Matrix (#542), Pacific Atlantic (#417) all use the technique of
pushing MULTIPLE starting nodes into the queue before BFS begins.

### 7. Graph + DP combos appear at Google/Meta for senior roles
```
#329 Longest Increasing Path in Matrix — DFS + memoization on a directed graph
#787 Cheapest Flights — Dijkstra / Bellman-Ford + state
#1334 Find City — Floyd-Warshall
```

---

## Complexity Reference

| Pattern            | Time               | Space    | When to Use                          |
|--------------------|--------------------|----------|--------------------------------------|
| DFS                | O(V + E)           | O(V)     | Traversal, cycle, components         |
| BFS                | O(V + E)           | O(V)     | Unweighted shortest path             |
| Topological Sort   | O(V + E)           | O(V)     | Dependency ordering (DAG only)       |
| DSU (Path+Rank)    | O(α(n)) ≈ O(1)    | O(V)     | Dynamic connectivity, cycle detect   |
| Dijkstra           | O((V+E) log V)     | O(V)     | Weighted shortest path (no neg)      |
| Bellman-Ford       | O(V * E)           | O(V)     | Negative edges / cycle detect        |
| Floyd-Warshall     | O(V³)              | O(V²)    | All-pairs shortest path              |
| Kruskal's MST      | O(E log E)         | O(V)     | Minimum spanning tree                |
| Prim's MST         | O((V+E) log V)     | O(V)     | Minimum spanning tree (dense graphs) |
| Grid DFS/BFS       | O(m * n)           | O(m * n) | Island/region problems               |
| Bipartite Check    | O(V + E)           | O(V)     | 2-coloring check                     |
