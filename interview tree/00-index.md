# Tree Interview Master Index

## How to Use This Folder
Each file = one pattern. Open in order for your study session.
Every file has: decision guide → template → top problems with full solutions → interview tips.

---

## Patterns

| File | Pattern | Difficulty | Most Asked At |
|------|---------|-----------|--------------|
| [01-dfs-traversal.md](01-dfs-traversal.md) | DFS Pre/In/Post Order | Easy–Medium | Microsoft, Amazon |
| [02-bfs-level-order.md](02-bfs-level-order.md) | BFS / Level Order | Easy–Medium | Meta, Amazon |
| [03-tree-dp.md](03-tree-dp.md) | Tree DP (combine left+right) | Medium–Hard | Google, Meta |
| [04-path-problems.md](04-path-problems.md) | Path Sum / Root-to-Leaf | Easy–Medium | Amazon, Apple |
| [05-lca.md](05-lca.md) | Lowest Common Ancestor | Medium | All top companies |
| [06-bst.md](06-bst.md) | BST Operations | Easy–Hard | Bloomberg, Microsoft |
| [07-tree-construction.md](07-tree-construction.md) | Build / Serialize Tree | Medium–Hard | Google, Uber |
| [08-trie.md](08-trie.md) | Trie (Prefix Tree) | Medium–Hard | Google, Meta |
| [09-interview-tips.md](09-interview-tips.md) | Cross-pattern tips & edge cases | — | All |

---

## Must-Do 15 (Cover All Patterns)

```
#104 Max Depth           → Tree DP / DFS
#226 Invert Tree         → DFS preorder
#102 Level Order         → BFS
#199 Right Side View     → BFS
#543 Diameter            → Tree DP
#112 Path Sum            → DFS path
#113 Path Sum II         → DFS path backtrack
#236 LCA                 → LCA
#98  Validate BST        → BST bounds
#230 Kth Smallest BST    → BST inorder
#105 Build from Pre+In   → Tree Construction
#124 Max Path Sum        → Tree DP (hardest)
#297 Serialize/Deserialize → Construction
#208 Implement Trie      → Trie
#1448 Count Good Nodes   → DFS (pass state down)
```

---

## Pattern Recognition Cheat Sheet

```
Visit all nodes in order?              → DFS Traversal
Level by level / depth matters?        → BFS
Both subtrees contribute to answer?    → Tree DP
Path from root to leaf?                → DFS Path (pass down)
Path through any node?                 → Tree DP (global var)
Deepest common node of p,q?            → LCA
Sorted property (left<node<right)?     → BST pattern
Rebuild from two traversal arrays?     → Tree Construction
Prefix matching / dictionary?          → Trie
```
