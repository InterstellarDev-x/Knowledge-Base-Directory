# Pattern 4: Union-Find (Disjoint Set Union — DSU)

## Identify This Pattern
- "Are these two nodes connected / in the same group?"
- "How many connected components?"
- "Detect cycle in undirected graph"
- "Merge groups / accounts / islands"
- "Minimum spanning tree" (Kruskal's uses DSU)
- Edges added one by one, query connectivity dynamically

## Recognition Signal
> "Group things together, answer queries about whether X and Y are in the same group"
> OR "add edges one by one, detect when a cycle forms"
> → DSU is almost always the cleanest solution

---

## Template (with Path Compression + Union by Rank)

```cpp
struct DSU {
    vector<int> parent, rank;

    DSU(int n) : parent(n), rank(n, 0) {
        iota(parent.begin(), parent.end(), 0);  // parent[i] = i
    }

    int find(int x) {
        if (parent[x] != x)
            parent[x] = find(parent[x]);  // path compression
        return parent[x];
    }

    bool unite(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;        // already same component

        if (rank[px] < rank[py]) swap(px, py);
        parent[py] = px;
        if (rank[px] == rank[py]) rank[px]++;
        return true;
    }

    bool connected(int x, int y) { return find(x) == find(y); }
};
```

### DSU with Size (for Kruskal's, island area tracking)
```cpp
struct DSU {
    vector<int> parent, size;
    DSU(int n) : parent(n), size(n, 1) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) { return parent[x]==x ? x : parent[x]=find(parent[x]); }
    bool unite(int x, int y) {
        x = find(x); y = find(y);
        if (x == y) return false;
        if (size[x] < size[y]) swap(x, y);
        parent[y] = x;
        size[x] += size[y];
        return true;
    }
    int getSize(int x) { return size[find(x)]; }
};
```

### Count Components Pattern
```cpp
DSU dsu(n);
int components = n;
for (auto& [u, v] : edges)
    if (dsu.unite(u, v)) components--;
// components = number of connected components
```

### Cycle Detection Pattern
```cpp
// If unite() returns false → both nodes already in same component → cycle
for (auto& [u, v] : edges)
    if (!dsu.unite(u, v)) { /* cycle found */ }
```

---

## Problems

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [547](https://leetcode.com/problems/number-of-provinces/) | Number of Provinces | Unite i,j for each isConnected[i][j]==1. Answer = distinct components | Classic DSU counting |
| [684](https://leetcode.com/problems/redundant-connection/) | Redundant Connection | Process edges one by one. The first edge that doesn't reduce component count is redundant | `unite()` returns false → edge is redundant |
| [261](https://leetcode.com/problems/graph-valid-tree/) | Graph Valid Tree | n nodes, n-1 edges, no cycle, all connected | Check: exactly n-1 edges unite without cycle, final component count = 1 |
| [323](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Number of Connected Components | Unite all edges, count remaining components | Direct DSU application |
| [990](https://leetcode.com/problems/satisfiability-of-equality-equations/) | Satisfiability Equality Equations | Process all `==` first (unite), then check all `!=` (verify different roots) | Two-pass: unite equals, verify not-equals |
| [1319](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) | Make Network Connected | Need at least n-1 edges to connect n nodes. Count extra edges and components. | `extra_edges >= components - 1` → possible |
| [1584](https://leetcode.com/problems/min-cost-to-connect-all-points/) | Min Cost to Connect All Points | Kruskal's MST — sort all pairs by Manhattan distance, unite greedily | Generate all O(n²) edges, sort, Kruskal's |
| [721](https://leetcode.com/problems/accounts-merge/) | Accounts Merge | Union emails by account. Group emails by root. | Map email → id for DSU. Map root → list of emails. |
| [128](https://leetcode.com/problems/longest-consecutive-sequence/) | Longest Consecutive Sequence | DSU approach: unite n with n+1 if both exist. Or hashset approach (O(n)) | Hashset approach is simpler and also O(n) |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [685](https://leetcode.com/problems/redundant-connection-ii/) | Redundant Connection II | Directed graph — 3 cases: one node has 2 parents, cycle, or both | Track parent of each node. If double-parent exists, one of those 2 edges is the answer |
| [305](https://leetcode.com/problems/number-of-islands-ii/) | Number of Islands II | Dynamically add land cells. For each add: unite with adjacent land | DSU with size. Answer for each add = current component count |
| [827](https://leetcode.com/problems/making-a-large-island/) | Making a Large Island | Label islands with DSU. For each 0-cell: sum sizes of distinct neighbor islands + 1 | DSU with size per component; check 4 neighbors deduplicating by root |

---

## Path Compression vs Union by Rank

```
Without both optimizations:  O(n) per operation in worst case (chain)
With path compression only:  O(log n) amortized
With union by rank only:     O(log n)
With BOTH:                   O(α(n)) ≈ O(1) (inverse Ackermann, practically constant)

→ Always use both in interviews
```

## DSU vs DFS for Connectivity

```
Use DSU when:
  - Edges are added dynamically (one by one)
  - Many connectivity queries (is u connected to v?)
  - Counting/merging components is the main operation
  - Kruskal's MST

Use DFS/BFS when:
  - Graph is fully given upfront
  - You need to explore paths (not just connectivity)
  - You need to process nodes in a specific traversal order
```

---

## Common Mistakes

- **Path compression:** Not doing `parent[x] = find(parent[x])` (flat compression). Just `return find(parent[x])` without assignment = no compression.
- **Union by rank:** Forgetting to increment rank when both trees have same rank after union.
- **Accounts Merge:** Not creating a consistent mapping from email string to integer ID before using DSU.
- **#990 Equality equations:** Processing `!=` before `==`. Must unite all `==` pairs first, then verify `!=`.

## Follow-up Interviewers Love

- "What's the time complexity of DSU?" → O(α(n)) per operation ≈ O(1) with both optimizations
- "What if you need to undo a union?" → Standard DSU can't undo. Need "link-cut trees" for rollback.
- "Kruskal's vs Prim's for MST?" → Kruskal: sort edges + DSU, O(E log E). Prim: min-heap + adjacency, O((V+E) log V). Kruskal better for sparse, Prim for dense.
- "Can DSU detect cycles in directed graphs?" → No, DSU doesn't track edge direction. Use DFS 3-color for directed.
