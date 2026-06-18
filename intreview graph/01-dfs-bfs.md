# Pattern 1: DFS / BFS on Graph

## Identify This Pattern
- "Explore all nodes reachable from a source"
- "Count connected components"
- "Clone / copy a graph"
- "Detect cycle"
- "Check if path exists between two nodes"
- "Minimum steps / moves" → BFS (unweighted)
- "Implicit graph" — neighbors generated on the fly (word ladder, gene mutation)

## BFS vs DFS Decision
```
Shortest path (unweighted)?     → BFS (level-by-level = minimum hops)
Just need to visit all nodes?   → DFS (simpler, less memory for sparse)
Detect cycle in directed graph? → DFS (3-color states)
Detect cycle in undirected?     → DFS (track parent) or DSU
```

---

## Templates

### Graph Representations
```cpp
// Adjacency list — use this by default
vector<vector<int>> adj(n);
adj[u].push_back(v);               // directed
adj[u].push_back(v); adj[v].push_back(u);  // undirected

// Weighted
vector<vector<pair<int,int>>> adj(n);  // {neighbor, weight}

// Visited — always needed
vector<bool> vis(n, false);
// For directed graphs
vector<int> color(n, 0);  // 0=unvisited, 1=in-stack, 2=done
```

### Recursive DFS
```cpp
void dfs(int node, vector<vector<int>>& adj, vector<bool>& vis) {
    vis[node] = true;
    // process node
    for (int nb : adj[node])
        if (!vis[nb]) dfs(nb, adj, vis);
}
// Drive: for (int i = 0; i < n; i++) if (!vis[i]) dfs(i, adj, vis);
```

### Iterative DFS (stack)
```cpp
stack<int> st;
st.push(start); vis[start] = true;
while (!st.empty()) {
    int node = st.top(); st.pop();
    // process node
    for (int nb : adj[node])
        if (!vis[nb]) { vis[nb] = true; st.push(nb); }
}
```

### BFS (unweighted shortest path)
```cpp
queue<int> q;
vector<int> dist(n, -1);
q.push(src); dist[src] = 0;
while (!q.empty()) {
    int node = q.front(); q.pop();
    for (int nb : adj[node]) {
        if (dist[nb] == -1) {
            dist[nb] = dist[node] + 1;
            q.push(nb);
        }
    }
}
// dist[dst] = shortest hops, -1 if unreachable
```

### Cycle Detection — Undirected (track parent)
```cpp
bool hasCycle(int node, int parent, vector<vector<int>>& adj, vector<bool>& vis) {
    vis[node] = true;
    for (int nb : adj[node]) {
        if (!vis[nb]) {
            if (hasCycle(nb, node, adj, vis)) return true;
        } else if (nb != parent) return true;  // back edge
    }
    return false;
}
```

### Cycle Detection — Directed (3-color DFS)
```cpp
// 0=white(unvisited), 1=gray(in current path), 2=black(done)
bool hasCycle(int node, vector<vector<int>>& adj, vector<int>& color) {
    color[node] = 1;  // gray
    for (int nb : adj[node]) {
        if (color[nb] == 1) return true;   // back edge = cycle
        if (color[nb] == 0 && hasCycle(nb, adj, color)) return true;
    }
    color[node] = 2;  // black
    return false;
}
```

### BFS on Implicit Graph (neighbors generated on-the-fly)
```cpp
// Pattern: state → neighbors via transformation
queue<State> q;
unordered_set<State> visited;
q.push(startState); visited.insert(startState);
int steps = 0;
while (!q.empty()) {
    int sz = q.size();
    for (int i = 0; i < sz; i++) {
        State cur = q.front(); q.pop();
        if (cur == targetState) return steps;
        for (State next : generateNeighbors(cur)) {
            if (!visited.count(next)) {
                visited.insert(next);
                q.push(next);
            }
        }
    }
    steps++;
}
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [547](https://leetcode.com/problems/number-of-provinces/) | Number of Provinces | DFS/BFS from each unvisited node, count starts | Adjacency matrix given — still applies same DFS |
| [1971](https://leetcode.com/problems/find-if-path-exists-in-graph/) | Find if Path Exists | Simple BFS/DFS from source to dest | Foundation problem |
| [997](https://leetcode.com/problems/find-the-town-judge/) | Find the Town Judge | in-degree == n-1 AND out-degree == 0 | Not a traversal — degree counting |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS/DFS + map{original → clone} | Create clone before recursing neighbors to handle cycles |
| [841](https://leetcode.com/problems/keys-and-rooms/) | Keys and Rooms | DFS: start at room 0, collect keys, visit rooms | Set tracks visited rooms |
| [802](https://leetcode.com/problems/find-eventual-safe-states/) | Find Eventual Safe States | Reverse edges + Kahn's topo sort. OR DFS 3-color: node safe if all neighbors safe | Safe = no cycle reachable from node |
| [1557](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/) | Min Vertices to Reach All | Nodes with in-degree == 0 must be in answer | If a node has no incoming edge, nothing can reach it |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [127](https://leetcode.com/problems/word-ladder/) | Word Ladder | BFS on implicit graph. Each word = node, neighbors = 1-char-diff words | Erase from wordSet immediately when visited to avoid re-adding |
| [433](https://leetcode.com/problems/minimum-genetic-mutation/) | Minimum Genetic Mutation | Same as word ladder, smaller alphabet | BFS level-by-level |
| [126](https://leetcode.com/problems/word-ladder-ii/) | Word Ladder II | BFS to find shortest length + DFS/backtrack to collect all paths | Two-phase: BFS builds level map, DFS traces back |

---

## Optimizations & Common Mistakes

**Common mistakes:**
- Not marking visited BEFORE pushing to queue → same node added multiple times
- Forgetting to handle disconnected graphs — always loop over all nodes as start
- For directed cycle: using `parent` trick (only valid for undirected). Must use 3-color for directed.

**Optimizations:**
- Word Ladder: erase from wordSet when visited (faster than a separate visited set + avoids re-adding)
- For large sparse graphs: adjacency list always beats adjacency matrix
- Bidirectional BFS (#127): start BFS from both ends, stop when frontiers meet → cuts search space from O(b^d) to O(b^(d/2))

**Follow-up interviewers love:**
- "What if graph is disconnected?" → Loop over all nodes
- "Can you do BFS iteratively and DFS recursively for cycle detection?" → Both are valid, explain the difference
- "What's the difference between a tree and a graph?" → Graph can have cycles, multiple paths between nodes
- "Word Ladder — why BFS and not DFS?" → BFS guarantees shortest path; DFS might find a longer path first
