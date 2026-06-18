# Tree Interview — Cross-Pattern Tips & Edge Cases

## Before You Code: 3 Questions to Ask

```
1. "Is this a BST?" → If yes, use sorted property. O(h) instead of O(n).
2. "Do I need info from BOTH subtrees at once?" → If yes, it's Tree DP (global var).
3. "Does depth/level matter?" → If yes, BFS. If not, DFS is simpler.
```

---

## Universal Checklist

- [ ] `if (!root) return ...` — first line always
- [ ] Handle single-node tree explicitly if edge case matters
- [ ] Leaf check: `!root->left && !root->right`
- [ ] Which traversal order? Pre (top-down) / In (sorted BST) / Post (bottom-up)
- [ ] Is answer computed top-down (pass info down) or bottom-up (return info up)?
- [ ] Need a global variable? (diameter, max path sum, count of nodes)
- [ ] Trace through a 3-node tree by hand before coding

---

## Complexity Reference

| Pattern | Time | Space | Notes |
|---------|------|-------|-------|
| DFS Traversal | O(n) | O(h) | h = O(n) skewed, O(log n) balanced |
| BFS Level Order | O(n) | O(w) | w = max width, O(n) complete tree |
| Tree DP | O(n) | O(h) | One pass, each node visited once |
| LCA (binary tree) | O(n) | O(h) | O(log n) for balanced BST |
| BST Search/Insert | O(h) | O(h) | O(log n) balanced, O(n) skewed |
| BST Inorder | O(n) | O(h) | — |
| Build from Traversals | O(n) | O(n) | Hashmap for O(1) index lookup |
| Serialize/Deserialize | O(n) | O(n) | One pass each direction |
| Trie Insert/Search | O(L) | O(L) | L = word length |

---

## Most Common Interview Follow-Ups (by Problem)

| Problem | Follow-up |
|---------|-----------|
| Any recursive solution | "Can you do it iteratively?" |
| Any tree problem | "What if it's an N-ary tree?" |
| Max depth, diameter | "Space complexity?" → O(h) |
| LCA | "What if nodes may not exist in tree?" → #1644 |
| BST inorder | "What if k is very large and called frequently?" → augment with subtree sizes |
| Serialize/Deserialize | "Can you make it more compact for BST?" → #449 |
| Validate BST | "Can you do it iteratively?" → iterative inorder, check prev < cur |
| Word Search II | "How do you handle duplicates?" → remove word from trie after finding |

---

## Patterns That Get Confused — Clarification

### Tree DP vs DFS Path
```
DFS Path:  path flows ROOT → LEAF. Carry running sum/string DOWN.
Tree DP:   path can BEND at a node. Both left+right contribute. Use global var.

#112 Path Sum     → DFS Path (root to leaf only)
#124 Max Path Sum → Tree DP (path can go left+node+right)
```

### Preorder vs Postorder
```
Preorder:  process node BEFORE subtrees  → need parent info while going down
           Examples: invert tree, count good nodes, serialize
Postorder: process node AFTER subtrees   → need children result before processing
           Examples: height, diameter, flatten, delete tree
```

### BFS Min Depth vs DFS Min Depth
```
BFS: returns at FIRST LEAF seen → guaranteed minimum → correct
DFS: must explore ENTIRE tree before returning minimum → correct but slow
→ Always use BFS for min depth (early exit is the whole point)
```

---

## Tricky Edge Cases to Always Consider

| Scenario | Common Bug |
|----------|-----------|
| Single node tree | Check leaf logic `!left && !right` before accessing left/right |
| Skewed tree (like linked list) | Stack overflow for deep recursion. Mention iterative for production |
| Negative values | `max(0, left)` in tree DP — when do you clamp? Only when problem says "optional" paths |
| Duplicate values in BST | Standard BST assumes unique values. If duplicates allowed, `<=` vs `<` matters |
| Node values at INT_MIN / INT_MAX | Use `long` for BST validation bounds |
| Empty tree root == null | Handle in first line of every function |

---

## Company-Wise Most Asked (2023–2025)

| Company | Top Problems |
|---------|-------------|
| **Amazon** | #102 Level Order, #104 Max Depth, #236 LCA, #98 Validate BST, #112 Path Sum |
| **Google** | #297 Serialize/Deserialize, #124 Max Path Sum, #236 LCA, #99 Recover BST, #543 Diameter |
| **Meta** | #199 Right Side View, #543 Diameter, #124 Max Path Sum, #572 Subtree, #102 Level Order |
| **Microsoft** | #226 Invert, #104 Max Depth, #102 Level Order, #110 Balanced, #98 Validate BST |
| **Bloomberg** | #102 Level Order, #103 Zigzag, #230 Kth Smallest, #98 Validate BST |

---

## Hard Problems Worth Knowing (Senior / FAANG+)

| # | Problem | Pattern Combo | Why It's Asked |
|---|---------|--------------|----------------|
| [124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Max Path Sum | Tree DP + global | Tests "return vs global" insight |
| [297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Serialize/Deserialize | Preorder + string parsing | Google favorite — tests multiple skills |
| [99](https://leetcode.com/problems/recover-bst/) | Recover BST | BST inorder + two-pointer | Subtle inorder violation detection |
| [968](https://leetcode.com/problems/binary-tree-cameras/) | Binary Tree Cameras | Tree DP + 3-state greedy | Greedy decisions in postorder |
| [834](https://leetcode.com/problems/sum-of-distances-in-tree/) | Sum of Distances | Rerooting DP | Rare but tests advanced Tree DP |
| [987](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/) | Vertical Order Traversal | DFS + sorting by (col, row, val) | Tests coordinate tracking |
| [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + DFS backtracking | Combines two hard patterns |

---

## Things to Mention Without Being Asked

1. **Skewed tree space:** "Recursive depth = O(h), which is O(n) for a skewed tree. For production I'd use iterative to avoid stack overflow."
2. **BST property when relevant:** Even if not asked to use it, mention it — shows you recognized it's a BST.
3. **Inorder = sorted:** Whenever you see BST + any sorted-order operation.
4. **Global variable justification:** "I'm using a global because the optimal path can bend at any node — the return value to parent is different from the global answer."
