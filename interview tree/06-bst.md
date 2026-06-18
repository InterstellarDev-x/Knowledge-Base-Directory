# Pattern 6: BST Operations

## Identify This Pattern
- Tree has sorted property: left subtree < node < right subtree
- "Search / insert / delete in BST"
- "Validate BST"
- "Kth smallest/largest"
- "Range queries in BST"
- "Two nodes swapped by mistake"
- "Convert sorted array/list to BST"

## The Golden Rule
> **Inorder traversal of BST = sorted array**
> Almost every BST problem reduces to: "what does this look like in sorted order?"

---

## Templates

### Search / Insert (use BST property to navigate)
```cpp
// Navigate: if val < node go left, if val > node go right
TreeNode* search(TreeNode* root, int val) {
    if (!root || root->val == val) return root;
    return val < root->val ? search(root->left, val)
                           : search(root->right, val);
}
```

### Validate BST (pass min/max bounds — NOT just checking neighbors)
```cpp
bool isValidBST(TreeNode* root, long mn = LONG_MIN, long mx = LONG_MAX) {
    if (!root) return true;
    if (root->val <= mn || root->val >= mx) return false;
    return isValidBST(root->left,  mn, root->val) &&
           isValidBST(root->right, root->val, mx);
}
// WRONG approach: only checking root->left->val < root->val
// CORRECT: pass tightening bounds (a deep right child of a left node must still be < root)
```

### Delete Node in BST
```cpp
// 3 cases:
// 1. No left child  → return right
// 2. No right child → return left
// 3. Both children  → replace with inorder successor (min of right subtree), delete successor
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;
    if (key < root->val)      root->left  = deleteNode(root->left,  key);
    else if (key > root->val) root->right = deleteNode(root->right, key);
    else {
        if (!root->left)  return root->right;
        if (!root->right) return root->left;
        TreeNode* successor = root->right;
        while (successor->left) successor = successor->left;
        root->val   = successor->val;
        root->right = deleteNode(root->right, successor->val);
    }
    return root;
}
```

### Inorder for Kth Smallest (early exit)
```cpp
// Inorder iterates sorted order; stop at k-th visit
int count = 0, result = 0;
void inorder(TreeNode* node, int k) {
    if (!node) return;
    inorder(node->left, k);
    if (++count == k) { result = node->val; return; }
    inorder(node->right, k);
}
```

### Sorted Array → Balanced BST (binary divide)
```cpp
TreeNode* sortedArrayToBST(vector<int>& nums, int l, int r) {
    if (l > r) return nullptr;
    int mid = l + (r - l) / 2;
    TreeNode* root = new TreeNode(nums[mid]);
    root->left  = sortedArrayToBST(nums, l, mid - 1);
    root->right = sortedArrayToBST(nums, mid + 1, r);
    return root;
}
// Median becomes root → balanced height
```

### Recover BST (two nodes swapped — inorder finds the violation)
```cpp
// Inorder of BST should be ascending
// Two swapped nodes create 1 or 2 violations in the inorder sequence
// First violation:  prev->val > cur->val → first = prev
// Second violation: prev->val > cur->val → second = cur
// Swap first->val and second->val
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [700](https://leetcode.com/problems/search-in-a-binary-search-tree/) | Search in BST | Navigate left/right using BST property | Base case returns |
| [108](https://leetcode.com/problems/convert-sorted-array-to-binary-search-tree/) | Sorted Array to BST | Mid element = root, recurse halves | Pick mid consistently (low + (high-low)/2) |
| [235](https://leetcode.com/problems/lowest-common-ancestor-of-a-binary-search-tree/) | LCA of BST | Use BST property to find split point | Both < root → left, both > root → right, else root |
| [501](https://leetcode.com/problems/find-mode-in-bst/) | Find Mode in BST | Inorder traversal, track consecutive duplicates | O(1) space with Morris traversal |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [98](https://leetcode.com/problems/validate-binary-search-tree/) | Validate BST | Pass (min, max) bounds down | Don't just check neighbors — deep violations need bounds |
| [230](https://leetcode.com/problems/kth-smallest-element-in-a-bst/) | Kth Smallest in BST | Inorder = sorted; stop at kth visit | Iterative inorder with early exit is most efficient |
| [450](https://leetcode.com/problems/delete-node-in-a-bst/) | Delete Node in BST | 3 cases: no left, no right, both. For both: replace with inorder successor | Find leftmost of right subtree = successor |
| [701](https://leetcode.com/problems/insert-into-a-binary-search-tree/) | Insert into BST | Navigate like search, attach where null | Return new node to reconnect |
| [173](https://leetcode.com/problems/binary-search-tree-iterator/) | BST Iterator | Lazy inorder: initialize with leftmost path on stack | `next()`: pop, push right subtree's left path |
| [1038](https://leetcode.com/problems/binary-search-tree-to-greater-sum-tree/) | BST to Greater Sum Tree | Reverse inorder (right→root→left) + running suffix sum | Right-root-left visits nodes in descending order |
| [530](https://leetcode.com/problems/minimum-absolute-difference-in-bst/) | Min Absolute Difference | Inorder + track previous node | min diff in BST = min(cur - prev) in inorder |
| [653](https://leetcode.com/problems/two-sum-iv-input-is-a-bst/) | Two Sum IV - BST | Inorder to sorted array + two pointers OR hashset DFS | Sorted array approach is cleanest |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [99](https://leetcode.com/problems/recover-bst/) | Recover BST | Inorder traversal — find 1 or 2 violations. Swap the two nodes | 1st violation: first=prev, 2nd violation: second=cur. Always update second |
| [315](https://leetcode.com/problems/count-of-smaller-numbers-after-self/) | Count Smaller After Self | BST insertion from right + count left subtree size | Or merge sort / BIT — BST approach intuitive |

---

## BST Iterator Design — Important for Interviews

```
Design an iterator that returns BST elements in sorted order.
Constraint: O(h) memory (not O(n))

Idea: Don't flatten the tree. Use a stack.
- Constructor: push root and all left children
- next(): pop top, push all left children of popped->right
- hasNext(): !stack.empty()

Why O(h) memory: at most one path from root to leaf in stack at any time.
```

---

## Common Mistakes

- **Validate BST:** Checking only `left->val < node->val` is wrong. A left subtree node deep in the right could violate the global constraint. Always use bounds.
- **Delete in BST:** Forgetting the 3-case split. "Both children" case: don't physically remove the successor node from its original position before updating root's value.
- **Kth Smallest:** Off-by-one on count. Start count at 0 and check `++count == k` (pre-increment).

## Follow-up Interviewers Love

- "What if BST is very large and kth smallest is called frequently?" → Augment each node with subtree size (order-statistics tree)
- "BST Iterator — what are the time complexities?" → `next()` is amortized O(1), not O(h) per call
- "Can you validate BST iteratively?" → Yes, use iterative inorder and check prev < cur
- "What if node values can equal boundary?" → Use strict comparison: `val <= mn` or `val >= mx` is invalid
