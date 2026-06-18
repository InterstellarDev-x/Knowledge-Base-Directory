# Graph Interview Master Index

## How to Use This Folder
Each file = one pattern. Every file has: recognition signals → template → problems with hints → mistakes → follow-ups.

---

## Pattern Files

| File | Pattern | Difficulty | Most Asked At |
|------|---------|-----------|--------------|
| [01-dfs-bfs.md](01-dfs-bfs.md) | DFS / BFS on Graph | Easy–Medium | All companies |
| [02-grid-graph.md](02-grid-graph.md) | Grid as Graph (Islands) | Easy–Medium | Amazon, Meta, Google |
| [03-topological-sort.md](03-topological-sort.md) | Topological Sort | Medium | Amazon, Google, Meta |
| [04-union-find.md](04-union-find.md) | Union-Find (DSU) | Medium | Google, Meta, Bloomberg |
| [05-dijkstra.md](05-dijkstra.md) | Dijkstra / Weighted Shortest Path | Medium–Hard | Google, Uber, DoorDash |
| [06-advanced.md](06-advanced.md) | Bipartite, Bellman-Ford, Floyd-Warshall, MST | Hard | Google, Uber |
| [07-company-questions.md](07-company-questions.md) | Company-Specific Problem Lists (2024–2025) | All | All |
| [08-interview-tips.md](08-interview-tips.md) | Cross-pattern tips, edge cases, follow-ups | — | All |

---

## The Two Questions to Ask First

```
1. Directed or undirected?  → affects cycle detection, topological sort
2. Weighted or unweighted?  → unweighted = BFS, weighted = Dijkstra/Bellman-Ford
```

## Pattern Decision Tree

```
Explore all nodes / count components?      → DFS or BFS
Shortest path — unweighted?               → BFS
Shortest path — weighted, no neg edges?   → Dijkstra
Shortest path — negative edges?           → Bellman-Ford
Shortest path — all pairs?                → Floyd-Warshall
Order tasks with dependencies?            → Topological Sort (Kahn's)
Detect cycle — undirected?                → DSU or DFS (track parent)
Detect cycle — directed?                  → DFS 3-color (white/gray/black)
Connect components / merge groups?        → Union-Find (DSU)
Minimum cost to connect all nodes?        → MST (Kruskal's)
2D grid — islands, regions, flood fill?   → Grid DFS/BFS
Can divide into 2 groups (no intra-edge)? → Bipartite check (BFS 2-color)
```

---

## Must-Do 12 (Cover All Patterns)

```
#200  Number of Islands             → Grid DFS
#994  Rotting Oranges               → Multi-source BFS
#207  Course Schedule               → Topological Sort (cycle detect)
#210  Course Schedule II            → Topological Sort (order)
#417  Pacific Atlantic Water Flow   → Multi-source DFS
#684  Redundant Connection          → DSU
#743  Network Delay Time            → Dijkstra
#547  Number of Provinces           → DFS / DSU
#133  Clone Graph                   → BFS/DFS with map
#785  Is Graph Bipartite            → BFS 2-coloring
#1584 Min Cost to Connect Points    → MST (Kruskal's)
#127  Word Ladder                   → BFS on implicit graph
```
