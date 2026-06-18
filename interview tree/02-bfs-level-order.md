# Pattern 2: BFS / Level Order Traversal

## Identify This Pattern
- "Level by level", "row by row", "breadth first"
- "Right/left side view"
- "Zigzag traversal", "spiral order"
- "Average / max / min of each level"
- "Connect nodes at same depth"
- Minimum depth (find first leaf — BFS is optimal)
- Any problem where node's DEPTH matters

## Recognition Signal
> When depth or level number of a node is relevant to the answer → BFS

---

## Template (memorize exactly)

```cpp
queue<TreeNode*> q;
q.push(root);

while (!q.empty()) {
    int sz = q.size();              // ← CRITICAL: snapshot before loop

    for (int i = 0; i < sz; i++) {
        auto node = q.front(); q.pop();
        // process node, use i and sz for position info
        if (node->left)  q.push(node->left);
        if (node->right) q.push(node->right);
    }
}
```

**The one trick that unlocks all BFS variants:** `int sz = q.size()` before the for loop.
- `i == 0` → first node of level (left side view)
- `i == sz-1` → last node of level (right side view)
- `int idx = leftToRight ? i : sz-1-i` → zigzag direction
- Track `depth` counter outside → level number available

---

## All Variants from One Template

```
Base BFS
 ├─ take last of each level             → Right Side View (#199)
 ├─ flip index direction each level     → Zigzag (#103)
 ├─ return at first leaf                → Min Depth (#111)
 ├─ sum / avg inside level loop         → Average of Levels (#637)
 ├─ node->next = q.front()              → Connect Next Pointers (#116)
 └─ reverse final result                → Level Order Bottom (#107)
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [111](https://leetcode.com/problems/minimum-depth-of-binary-tree/) | Minimum Depth | BFS: return depth when first leaf found | Don't use DFS — it visits whole tree before returning |
| [637](https://leetcode.com/problems/average-of-levels-in-binary-tree/) | Average of Levels | Sum inside level loop, divide by sz | Straightforward BFS variant |
| [107](https://leetcode.com/problems/binary-tree-level-order-traversal-ii/) | Level Order II (bottom-up) | Run normal BFS, reverse result | Or push_front instead of push_back |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [102](https://leetcode.com/problems/binary-tree-level-order-traversal/) | Level Order Traversal | Core BFS template | Foundation problem |
| [199](https://leetcode.com/problems/binary-tree-right-side-view/) | Right Side View | Take last node per level (`i == sz-1`) | DFS alt: pass depth, overwrite res[depth] going right-first |
| [103](https://leetcode.com/problems/binary-tree-zigzag-level-order-traversal/) | Zigzag Level Order | Pre-allocate `vector<int>(sz)`, write by flipped index | Cleaner than reversing alternate levels |
| [116](https://leetcode.com/problems/populating-next-right-pointers-in-each-node/) | Connect Next Pointers I | `node->next = q.front()` when `i < sz-1` | O(1) space: use already-set `.next` pointers as implicit queue |
| [117](https://leetcode.com/problems/populating-next-right-pointers-in-each-node-ii/) | Connect Next Pointers II | Same BFS approach works | O(1) space harder — use dummy head for each level |
| [515](https://leetcode.com/problems/find-largest-value-in-each-tree-row/) | Largest Value Per Row | Track max inside level loop | Trivial BFS variant |
| [662](https://leetcode.com/problems/maximum-width-of-binary-tree/) | Maximum Width of Binary Tree | Assign index to each node (left=2i, right=2i+1). Width = last_idx - first_idx + 1 | Store (node, index) pairs in queue. Normalize index per level to avoid overflow |
| [863](https://leetcode.com/problems/all-nodes-distance-k-in-binary-tree/) | All Nodes Distance K | Convert tree to undirected graph (track parent pointers), then BFS from target node | Key: you need to go UP the tree too, so parent map is needed |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [428](https://leetcode.com/problems/serialize-and-deserialize-n-ary-tree/) | Serialize N-ary Tree | BFS serialize: encode child count | Similar to #297 but handle variable children |

---

## Optimizations & Common Mistakes

**Common mistakes:**
- Not taking `int sz = q.size()` before the inner for loop — then newly added children bleed into current level
- Using DFS for minimum depth — DFS visits whole tree; BFS stops at first leaf
- For #662 (max width): not normalizing indexes per level → integer overflow on large trees

**Optimizations:**
- For #116 perfect binary tree: O(1) space by using already-connected `.next` pointers to traverse the level without a queue
- For right side view: DFS is actually simpler code — pass depth, store node at that depth (last write wins)
- For zigzag: pre-size the level vector and write by index — avoids reversing

**Follow-up interviewers love:**
- "Can you do it with O(1) extra space?" → Only possible for #116 (perfect tree), not general trees
- "Left side view instead?" → Same as right side view but take `i == 0`
- "What's the max width of a tree?" → #662, introduces index encoding trick
- "What's space complexity?" → O(w) where w = max width of tree. Worst case O(n) for complete binary tree at bottom level

## BFS vs DFS Decision
```
Need DEPTH/LEVEL info?                      → BFS (level is naturally tracked)
Want SHORTEST path to first leaf?           → BFS
Node's level matters for the answer?        → BFS
Just visiting all nodes (depth irrelevant)? → DFS (simpler, less memory for balanced trees)
```
