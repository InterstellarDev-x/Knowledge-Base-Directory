# Pattern 3: Topological Sort

## Identify This Pattern
- "Order tasks given dependencies" / "Schedule with prerequisites"
- "Is there a valid ordering?" / "Detect cycle in directed graph"
- "Can you finish all courses?" (Course Schedule)
- "Build order" / "Compile order"
- Works ONLY on Directed Acyclic Graphs (DAGs). If cycle exists → no valid order.

## Recognition Signal
> "Task A must come before task B" → directed edge A → B → topological sort
> "Can all tasks be completed?" → same problem, just check for cycles

---

## Templates

### Kahn's Algorithm (BFS — recommended for interviews)
```cpp
vector<int> topoSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0);
    for (int u = 0; u < n; u++)
        for (int v : adj[u]) indegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);  // nodes with no dependencies

    vector<int> order;
    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--indegree[v] == 0) q.push(v);
    }

    // order.size() != n → cycle exists (not all nodes processed)
    return order.size() == n ? order : vector<int>{};
}
```

### DFS-Based (postorder, push to stack)
```cpp
void dfs(int u, vector<vector<int>>& adj, vector<bool>& vis, stack<int>& st) {
    vis[u] = true;
    for (int v : adj[u])
        if (!vis[v]) dfs(v, adj, vis, st);
    st.push(u);   // push AFTER all descendants (postorder)
}
// Reverse the stack = topological order
```

### Cycle Detection in Directed Graph (3-color Kahn's check)
```cpp
// With Kahn's: if order.size() < n → cycle (some nodes never reached indegree 0)
// With DFS: use color[node]: 0=unvisited, 1=in-stack, 2=done
bool hasCycle(int u, vector<vector<int>>& adj, vector<int>& color) {
    color[u] = 1;  // in current DFS path
    for (int v : adj[u]) {
        if (color[v] == 1) return true;  // back edge = cycle
        if (color[v] == 0 && hasCycle(v, adj, color)) return true;
    }
    color[u] = 2;  // done
    return false;
}
```

### Build Adjacency List from Prerequisites
```cpp
// prerequisites[i] = [a, b] means "b must come before a" (b → a)
vector<vector<int>> adj(n);
vector<int> indegree(n, 0);
for (auto& p : prerequisites) {
    adj[p[1]].push_back(p[0]);  // edge from p[1] to p[0]
    indegree[p[0]]++;
}
```

---

## Problems

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Build graph from prereqs, run Kahn's. If all courses processed → no cycle | Cycle = impossible to finish |
| [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Same as #207 but return the order | If cycle, return empty array |
| [310](https://leetcode.com/problems/minimum-height-trees/) | Minimum Height Trees | Iteratively trim leaf nodes (degree 1) until 1 or 2 nodes remain | Like topological sort from outside in — roots of MHT are always the center(s) |
| [802](https://leetcode.com/problems/find-eventual-safe-states/) | Find Eventual Safe States | Reverse all edges, run Kahn's on reversed graph. Safe nodes = nodes that complete topo sort | A node is safe if it can't reach a cycle |
| [1557](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/) | Min Vertices to Reach All | Return nodes with in-degree == 0 | Nodes with no incoming edge can't be reached from anywhere |
| [2115](https://leetcode.com/problems/find-all-possible-recipes-from-given-supplies/) | Find All Possible Recipes | Topo sort where ingredients → recipe. Supplies = sources (indegree 0) | Treat supplies + recipes as nodes in a DAG |
| [1203](https://leetcode.com/problems/sort-items-by-groups-respecting-dependencies/) | Sort Items by Groups | Two-level topo sort: sort within groups, then sort groups | Groups form a DAG, items within groups form a DAG |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Compare adjacent words char by char → find ordering constraints → topo sort | Edge case: "abc" before "ab" in input = invalid (prefix longer = should come after) |
| [444](https://leetcode.com/problems/sequence-reconstruction/) | Sequence Reconstruction | Build graph from seqs, check if topo sort is unique AND matches target | Unique topo order: at every step, exactly 1 node with indegree 0 |

---

## Kahn's vs DFS — When to Use Which

```
Kahn's (BFS):
  ✓ Cycle detection is automatic (processed count < n → cycle)
  ✓ More intuitive — matches real "dependency resolving" logic
  ✓ Easier to modify (e.g., lexicographic order → use min-heap instead of queue)
  Use Kahn's in interviews

DFS (postorder):
  ✓ More concise code
  ✓ Natural for recursive thinking
  ✗ Cycle detection needs extra color array
  Use DFS when asked for O(1) extra space variant or recursion is cleaner
```

---

## Alien Dictionary — Step by Step

This is the trickiest topo sort problem. Break it into 3 clear steps:

```
Step 1: Build character set (nodes)
Step 2: Compare adjacent words pairwise
        - Find first differing character → directed edge word1[i] → word2[i]
        - If word1 is longer than word2 AND word2 is a prefix of word1 → invalid ("abc" > "ab")
Step 3: Run Kahn's topo sort on character graph
        - If all chars processed → valid, return order
        - If cycle → return ""
```

---

## Common Mistakes

- **Building the graph:** `prerequisites[i] = [a, b]` means b → a (b before a), NOT a → b
- **Alien dictionary:** Forgetting the edge case where a longer word is followed by its prefix (invalid ordering)
- **Detecting cycle with Kahn's:** Just check `order.size() == n`. Don't add a separate DFS cycle check.
- **Minimum height trees:** Wrong approach is running BFS/DFS from each node (O(n²)). Correct is leaf-trimming (O(n)).

## Follow-up Interviewers Love

- "What if there are multiple valid topological orders? How do you find the lexicographically smallest?" → Use min-heap instead of queue in Kahn's
- "What if the graph has self-loops?" → Self-loop = cycle, handle by checking `u == v` when building the graph
- "#207 — DFS vs Kahn's which is better?" → Kahn's: cycle detection is free (count check). DFS: needs explicit 3-color state.
- "What does it mean if Kahn's doesn't process all nodes?" → There's a cycle — those nodes circularly depend on each other
