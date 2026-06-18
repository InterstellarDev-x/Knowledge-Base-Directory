# Pattern 3: Tree DP (Combine Left + Right)

## Identify This Pattern
- Answer for a node depends on results from BOTH subtrees
- "Height / depth", "diameter", "balanced?", "max path sum"
- "House robber on tree" (include/exclude choices)
- You need to RETURN something useful to the parent AND update a global answer
- The recursive function does double duty

## Recognition Signal
> "At every node I need info from BOTH my left AND right children to compute the answer"
> If yes → Tree DP. The function returns something to parent AND updates global answer.

---

## The Mental Model

```
At each node:
1. Get left_result  = solve(node->left)
2. Get right_result = solve(node->right)
3. global_ans = max(global_ans, combine(left, right, node->val))  ← update global
4. return something_for_parent                                      ← return to parent

The return value and the global update are DIFFERENT.
```

---

## Templates

### Basic Tree DP
```cpp
int globalAns = 0; // or INT_MIN

int solve(TreeNode* node) {
    if (!node) return BASE_VALUE;          // 0 for height/sum, -inf for max

    int left  = solve(node->left);
    int right = solve(node->right);

    // Update global: path THROUGH this node (can use both sides)
    globalAns = max(globalAns, /* left + right + node->val */);

    // Return to parent: only ONE side (path must be linear going up)
    return /* node->val + max(left, right) */;
}
```

### Include/Exclude DP (House Robber pattern)
```cpp
pair<int,int> solve(TreeNode* node) {
    // returns {include_node, exclude_node}
    if (!node) return {0, 0};

    auto [lInc, lExc] = solve(node->left);
    auto [rInc, rExc] = solve(node->right);

    int include = node->val + lExc + rExc;          // take this, skip children
    int exclude = max(lInc, lExc) + max(rInc, rExc); // skip this, best of children

    return {include, exclude};
}
// Answer: max(include, exclude) at root
```

### Height-returning DP (balanced tree / diameter)
```cpp
int height(TreeNode* node) {
    if (!node) return 0;
    int left  = height(node->left);
    int right = height(node->right);

    // global update using both
    globalAns = max(globalAns, left + right);   // diameter through this node

    return 1 + max(left, right);                // height returned to parent
}
```

### Return -1 as sentinel (invalid state propagation)
```cpp
// Used for: balanced tree, valid BST range checks
int height(TreeNode* node) {
    if (!node) return 0;
    int left  = height(node->left);
    if (left == -1) return -1;    // early exit: already invalid
    int right = height(node->right);
    if (right == -1) return -1;
    if (abs(left - right) > 1) return -1;   // this node is unbalanced
    return 1 + max(left, right);
}
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Maximum Depth | `1 + max(left, right)` — no global needed | Simplest Tree DP |
| [543](https://leetcode.com/problems/diameter-of-binary-tree/) | Diameter | At each node: `left_height + right_height`. Global tracks max | Return height to parent, update global with both |
| [110](https://leetcode.com/problems/balanced-binary-tree/) | Balanced Binary Tree | Return -1 if subtree unbalanced, else height | Sentinel -1 early exits the recursion |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [337](https://leetcode.com/problems/house-robber-iii/) | House Robber III | Return pair {rob_this, skip_this} | Include/exclude template exactly |
| [1372](https://leetcode.com/problems/longest-zigzag-path-in-a-binary-tree/) | Longest ZigZag Path | At each node: `dfs(node, direction, length)`. From root go both ways | Pass direction (left/right) and current length down |
| [1026](https://leetcode.com/problems/maximum-difference-between-node-and-ancestor/) | Max Difference Ancestor | Pass (min, max) seen on path top-down | At each node: max(abs(node-min), abs(node-max)) |
| [2246](https://leetcode.com/problems/longest-path-with-different-adjacent-values/) | Longest Path Different Values | Take top-2 longest chains from children with different val | Sort children results, take 2 largest valid ones |
| [979](https://leetcode.com/problems/distribute-coins-in-binary-tree/) | Distribute Coins | Excess = (coins in subtree) - (nodes in subtree). Moves = sum of abs(excess) | Return excess from each subtree; moves += abs(excess) |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Binary Tree Maximum Path Sum | At each node: `max(0, left) + node->val + max(0, right)` updates global. Return `node->val + max(0, max(left, right))` | Ignore negative paths (max with 0). Return only ONE side upward |
| [834](https://leetcode.com/problems/sum-of-distances-in-tree/) | Sum of Distances in Tree | Two-pass DFS on graph (not binary tree). First: count subtree sizes + distances to subtree. Second: re-root and propagate | Rerooting technique — when root changes, distances adjust by ±1 |

---

## Key Distinctions

### Global update vs return value
```
Path THROUGH this node (can bend here):
  → use both left and right  → update global_ans

Path that CONTINUES upward from this node (must be linear):
  → use only MAX(left, right) → return to parent

This is why #124 and #543 have a global variable.
```

### When to max with 0
```
If problem allows "not using" a subtree (skip negative paths):
    left  = max(0, solve(node->left))   ← clamp negative gains
    right = max(0, solve(node->right))

If problem requires using all nodes:
    left  = solve(node->left)           ← no clamping
```

---

## Optimizations & Common Mistakes

**Common mistakes:**
- Updating global inside the `if (!node) return` block (off-by-one, will miss nodes)
- Returning `left + right + node->val` to parent (wrong — path can't branch going up)
- Forgetting `max(0, left)` when problem says path can skip negative subtrees

**Optimizations:**
- Use `function<int(TreeNode*)>` lambda with capture `[&]` to avoid passing global as param
- Early exit: return -1 sentinel propagates invalidity without redundant checks

**Follow-up interviewers love:**
- "#124 — what if path must include at least one node?" → `INT_MIN` not 0 as base
- "What if it's an N-ary tree?" → Same template, loop over all children
- "Can the path go through the root?" → Yes, global captures all cases
