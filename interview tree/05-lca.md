# Pattern 5: Lowest Common Ancestor (LCA)

## Identify This Pattern
- "Find LCA of nodes p and q"
- "Distance between two nodes in tree"
- "Path between two nodes"
- "Deepest node that has both X and Y as descendants"

## Core Logic
```
At current node:
  - If null → return null
  - If node == p OR node == q → return node (found one)
  - Recurse left and right

  After recursion:
  - Both sides non-null  → this node IS the LCA (p in left, q in right)
  - Only one side        → LCA is in that subtree, return it
  - Both null            → neither p nor q in this subtree
```

---

## Templates

### Binary Tree LCA (standard)
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;

    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);

    if (left && right) return root;    // split: one on each side
    return left ? left : right;        // both on same side
}
```

### BST LCA (use sorted property — O(log n) balanced)
```cpp
// Iterative (preferred for BST)
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val)
            root = root->left;
        else if (p->val > root->val && q->val > root->val)
            root = root->right;
        else return root;   // split point = LCA
    }
    return nullptr;
}
```

### LCA with Parent Pointers
```cpp
// Like finding intersection of two linked lists
// Approach 1: Set of ancestors of p, walk q's ancestors until hit
// Approach 2: Find depths, equalize, walk up together
TreeNode* lowestCommonAncestor(Node* p, Node* q) {
    unordered_set<Node*> ancestors;
    while (p) { ancestors.insert(p); p = p->parent; }
    while (q) {
        if (ancestors.count(q)) return q;
        q = q->parent;
    }
    return nullptr;
}
```

### Distance Between Two Nodes (using LCA)
```cpp
// distance(p, q) = depth(p) + depth(q) - 2 * depth(LCA(p, q))
int findDepth(TreeNode* root, TreeNode* target, int depth) {
    if (!root) return -1;
    if (root == target) return depth;
    int left = findDepth(root->left, target, depth + 1);
    return left != -1 ? left : findDepth(root->right, target, depth + 1);
}
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [235](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | LCA of BST | Use BST property — if both < root go left, both > root go right, else root IS LCA | No need for generic tree logic; O(h) iteratively |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [236](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree/) | LCA of Binary Tree | Return node when you find p or q. If both sides return non-null, current node is LCA | The "both sides non-null" case is the split |
| [1650](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-iii/) | LCA with Parent Pointers | Walk both up simultaneously OR use a visited set | Same logic as finding linked list intersection |
| [1644](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-ii/) | LCA II (nodes may not exist) | Can't return early at first found node — must verify both exist | Count found nodes; return only if both were seen |
| [1676](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-tree-iv/) | LCA of Multiple Nodes | Generalize: return node if it's in the target set | Use a set for O(1) lookup; logic otherwise same |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1123](https://leetcode.com/problems/lowest-common-ancestor-of-deepest-leaves/) | LCA of Deepest Leaves | Find max depth. LCA is the deepest node whose subtree contains all deepest leaves | Return (node, depth) pair. LCA when left_depth == right_depth == max |

---

## Key Variations to Know

### Variation 1: p or q might not exist in tree (#1644)
```
Problem: return null if either p or q not found
Fix: DON'T return early when root == p or q
     Continue searching to verify the other one exists
     Use a found counter or boolean flags
```

### Variation 2: Nodes have parent pointers (#1650)
```
Problem: tree given with parent pointers, find LCA
Approach A: collect all ancestors of p in a set, walk q upward until hit
Approach B: like linked list intersection (equalize depths, walk up together)
```

### Variation 3: LCA of multiple nodes (#1676)
```
Same template: just use a set instead of comparing to p and q
Return node if node is IN the set, not just == p or == q
```

### Variation 4: Distance between two nodes
```
distance(u, v) = depth(u) + depth(v) - 2 * depth(LCA(u, v))
```

---

## Optimizations & Common Mistakes

**Common mistakes:**
- For #1644: returning early when root==p (you still need to confirm q exists)
- For BST LCA: using generic tree LCA when O(log n) BST approach is available
- Returning `left` instead of `root` when both sides are non-null

**Optimizations:**
- BST LCA: iterative is O(1) space vs O(h) recursive
- Parent pointer LCA: depth-equalization approach avoids hashing

**Follow-up interviewers love:**
- "What if node might not exist in tree?" → #1644, mention need to not early-return
- "What about N-ary tree LCA?" → Same logic, loop over all children
- "Distance between two nodes?" → Use LCA depth formula
- "Time complexity of LCA?" → O(n) binary tree, O(h) BST. O(h) = O(log n) balanced
