# Pattern 1: DFS Traversals (Pre / In / Post Order)

## Identify This Pattern
- "Visit all nodes", "collect values in order"
- "Inorder / preorder / postorder traversal"
- "Mirror / invert / symmetric tree"
- "Same tree / subtree check"
- Need to process node BEFORE children → Preorder
- Need children result before processing node → Postorder
- BST + sorted order → Inorder

## Decision: Which order?
```
Process node first, then recurse?           → Preorder  (top-down)
Need children info before processing node?  → Postorder (bottom-up)
BST and need sorted output?                 → Inorder
```

---

## Templates

### Recursive Skeleton
```cpp
void dfs(TreeNode* root) {
    if (!root) return;
    // PREORDER: process root here
    dfs(root->left);
    // INORDER: process root here
    dfs(root->right);
    // POSTORDER: process root here
}
```

### Iterative Inorder (most asked in interviews)
```cpp
TreeNode* cur = root;
while (cur || !st.empty()) {
    while (cur) { st.push(cur); cur = cur->left; }
    cur = st.top(); st.pop();
    // process cur
    cur = cur->right;
}
```

### Iterative Preorder
```cpp
st.push(root);
while (!st.empty()) {
    auto node = st.top(); st.pop();
    // process node
    if (node->right) st.push(node->right);  // right before left (LIFO)
    if (node->left)  st.push(node->left);
}
```

### Iterative Postorder (trick: reverse of preorder variant)
```cpp
// preorder but push left before right → get root,right,left → reverse = postorder
st.push(root);
while (!st.empty()) {
    auto node = st.top(); st.pop();
    res.push_back(node->val);
    if (node->left)  st.push(node->left);
    if (node->right) st.push(node->right);
}
reverse(res.begin(), res.end());
```

### Pass State Down (preorder with parameter)
```cpp
void dfs(TreeNode* node, int stateFromParent) {
    if (!node) return;
    int newState = /* combine stateFromParent + node->val */;
    dfs(node->left,  newState);
    dfs(node->right, newState);
}
```

### Two-Node Recursion (mirror / same tree pattern)
```cpp
bool check(TreeNode* a, TreeNode* b) {
    if (!a && !b) return true;
    if (!a || !b) return false;
    return a->val == b->val
        && check(a->left,  b->right)   // outer / inner
        && check(a->right, b->left);
}
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [104](https://leetcode.com/problems/maximum-depth-of-binary-tree/) | Maximum Depth of Binary Tree | Postorder: 1 + max(left, right) | Base case: null → 0 |
| [226](https://leetcode.com/problems/invert-binary-tree/) | Invert Binary Tree | Preorder: swap children then recurse | Can also do BFS |
| [101](https://leetcode.com/problems/symmetric-tree/) | Symmetric Tree | Pass two nodes: check outer and inner pairs | `mirror(left, right)` with flipped args |
| [572](https://leetcode.com/problems/subtree-of-another-tree/) | Subtree of Another Tree | At each node: check `isSameTree`, else recurse | O(n*m) — mention it |
| [100](https://leetcode.com/problems/same-tree/) | Same Tree | Check val + recurse both sides | Classic two-node recursion |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [94](https://leetcode.com/problems/binary-tree-inorder-traversal/) | Inorder Traversal | Recursive trivial. Iterative: go left, pop, go right | Iterative is the real question |
| [144](https://leetcode.com/problems/binary-tree-preorder-traversal/) | Preorder Traversal | Push right before left (stack = LIFO) | Right first so left pops first |
| [145](https://leetcode.com/problems/binary-tree-postorder-traversal/) | Postorder Traversal | Trick: reverse of "preorder + push left then right" | Or use two stacks |
| [114](https://leetcode.com/problems/flatten-binary-tree-to-linked-list/) | Flatten to Linked List | Postorder: flatten left, flatten right, reconnect. O(1): Morris-style | Find rightmost of left, attach right subtree |
| [1448](https://leetcode.com/problems/count-good-nodes-in-binary-tree/) | Count Good Nodes | Preorder: pass max seen so far down | Node is good if val >= maxSoFar |
| [1026](https://leetcode.com/problems/maximum-difference-between-node-and-ancestor/) | Max Difference Node & Ancestor | Pass min and max seen down (preorder) | At leaf: max(abs(node-min), abs(node-max)) |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [987](https://leetcode.com/problems/vertical-order-traversal-of-a-binary-tree/) | Vertical Order Traversal | DFS + track (col, row). Sort by col, then row, then val | Use map<col, map<row, multiset<val>>> |
| [968](https://leetcode.com/problems/binary-tree-cameras/) | Binary Tree Cameras | Postorder + 3 states: covered, not covered, has camera | Greedy: place camera when a child is NOT covered |

---

## Optimizations & Common Mistakes

**Common mistakes:**
- Forgetting `if (!root) return` — always first line
- For symmetric tree: checking `root->left == root->right` (wrong, need recursive mirror check)
- For subtree: forgetting to check if the whole current tree matches, not just the root val

**Optimizations:**
- Iterative > recursive if asked for "O(1) extra space" (no recursion call stack)
- For flatten (#114): O(1) space iterative approach exists (Morris-style)
- For subtree (#572): serialize both trees and use string matching — O(n+m)

**Follow-up interviewers love:**
- "Can you do it iteratively?" → Know iterative inorder cold
- "What's the space complexity?" → O(h), O(n) worst for skewed tree
- "What if it's an N-ary tree?" → Same pattern, loop over children array
