# Pattern 7: Tree Construction & Serialization

## Identify This Pattern
- "Construct binary tree from preorder + inorder"
- "Construct binary tree from preorder + postorder"
- "Serialize and deserialize binary tree"
- "Reconstruct BST from preorder"
- Given traversal arrays, rebuild the original tree

## Core Insight
```
Preorder:  FIRST element  = root of current subtree
Postorder: LAST  element  = root of current subtree
Inorder:   root divides array → left of root = left subtree, right = right subtree
```

---

## Templates

### Build Tree: Preorder + Inorder
```cpp
// preorder[preStart] = root
// find root in inorder → elements left = left subtree, right = right subtree
// Use hashmap for O(1) inorder index lookup

unordered_map<int, int> inIdx;  // val → index in inorder
int preStart = 0;

TreeNode* build(int inL, int inR) {
    if (inL > inR) return nullptr;
    int rootVal = preorder[preStart++];     // consume preorder left to right
    TreeNode* root = new TreeNode(rootVal);
    int mid = inIdx[rootVal];               // split point in inorder
    root->left  = build(inL, mid - 1);     // left subtree
    root->right = build(mid + 1, inR);     // right subtree
    return root;
}
```

### Build Tree: Inorder + Postorder
```cpp
// postorder[postEnd] = root (consume from right to left)
// MUST build right subtree FIRST (postEnd decrements)

int postEnd = postorder.size() - 1;

TreeNode* build(int inL, int inR) {
    if (inL > inR) return nullptr;
    int rootVal = postorder[postEnd--];    // consume postorder right to left
    TreeNode* root = new TreeNode(rootVal);
    int mid = inIdx[rootVal];
    root->right = build(mid + 1, inR);   // ← RIGHT first! (postEnd goes backward)
    root->left  = build(inL, mid - 1);
    return root;
}
```

### Serialize: Preorder DFS with null markers
```cpp
// "#" marks null nodes — critical for unique reconstruction
string serialize(TreeNode* root) {
    if (!root) return "#,";
    return to_string(root->val) + ","
         + serialize(root->left)
         + serialize(root->right);
}
```

### Deserialize: Rebuild from token stream
```cpp
TreeNode* deserialize(queue<string>& q) {
    string val = q.front(); q.pop();
    if (val == "#") return nullptr;
    TreeNode* node = new TreeNode(stoi(val));
    node->left  = deserialize(q);
    node->right = deserialize(q);
    return node;
}
```

### Reconstruct BST from Preorder Only
```cpp
// Use BST property: valid range for each node shrinks as you go deeper
// Preorder: first element = root, then left subtree (values < root), then right (values > root)

TreeNode* build(vector<int>& pre, int& idx, int mn, int mx) {
    if (idx == pre.size() || pre[idx] < mn || pre[idx] > mx)
        return nullptr;
    int val = pre[idx++];
    TreeNode* root = new TreeNode(val);
    root->left  = build(pre, idx, mn, val);    // left: [mn, val)
    root->right = build(pre, idx, val, mx);    // right: (val, mx]
    return root;
}
```

---

## Problems

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [105](https://leetcode.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/) | Build from Preorder + Inorder | Hashmap for inorder index. Consume preorder left-to-right | Build left THEN right (preorder order) |
| [106](https://leetcode.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal/) | Build from Inorder + Postorder | Same as #105 but consume postorder right-to-left | Build RIGHT first (postorder reversed = root, right, left) |
| [1008](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/) | BST from Preorder | Use min/max bounds to decide subtree boundaries | No inorder needed — BST property gives bounds |
| [449](https://leetcode.com/problems/serialize-and-deserialize-bst/) | Serialize/Deserialize BST | Preorder + use BST bounds during deserialize | Smaller file size than #297 — no null markers needed |
| [431](https://leetcode.com/problems/encode-n-ary-tree-to-binary-tree/) | Encode N-ary to Binary | First child → left pointer. Next sibling → right pointer | Classic LCRS (left-child right-sibling) encoding |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [297](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/) | Serialize/Deserialize Binary Tree | Preorder DFS with "#" nulls. Deserialize using queue of tokens | Include null markers to uniquely reconstruct any binary tree |
| [536](https://leetcode.com/problems/construct-binary-tree-from-string/) | Build Tree from String | Parse `val(left_subtree)(right_subtree)` recursively | Track index pointer as you parse |

---

## Why Inorder Alone is NOT Enough

```
You cannot reconstruct a unique binary tree from a single traversal:
  Preorder [1,2,3] → could be:   1      OR   1
                                   \          / \
                                    2         2   3
                                     \
                                      3
Need TWO traversals (one must be inorder) to uniquely reconstruct.
Exception: BST preorder alone is enough (BST property gives implicit bounds).
```

## Why Postorder Needs Right Built First

```
Postorder sequence: [left..., right..., root]
Reading from the END: root, then right subtree root, then left subtree root
So consuming postEnd-- means you get: root → right child → left child
Must build RIGHT subtree first before decrementing further into left
```

---

## Optimizations & Common Mistakes

**Common mistakes:**
- Building left before right in postorder construction (order matters because of index decrement)
- Not using a hashmap for inorder index lookup → O(n) per search → O(n²) total
- In serialize: not including null markers → can't distinguish `[1,2]` from `[1,null,2]`

**Optimizations:**
- Hashmap for inorder lookup: reduces from O(n²) to O(n)
- BST serialization (#449): no null markers needed → more compact. Use bounds during deserialize.

**Follow-up interviewers love:**
- "Why do you need a hashmap?" → Without it, finding root in inorder is O(n) per call → O(n²) total
- "Can you reconstruct from preorder + postorder?" → Not uniquely (unless full binary tree)
- "Can you serialize more compactly?" → For BST, no null markers needed (#449)
- "What if values aren't unique?" → Can't use hashmap; need to pass index ranges more carefully
