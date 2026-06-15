# Tree Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Visit every node in a specific order?              → DFS Traversal (pre/in/post)
Process level by level?                            → BFS / Level Order
Find a path, sum, or max through nodes?            → DFS Path Pattern
Lowest common ancestor of two nodes?               → LCA Pattern
Problem on a BST (sorted property)?                → BST Pattern
Build tree from traversal arrays?                  → Tree Construction
Combine results from left + right subtree?         → Tree DP
Serialize / deserialize / clone the tree?          → Serialization Pattern
```

**The one question to always ask first:**
> "Do I need info from BOTH subtrees to answer for a node?" — if yes, it's Tree DP.
> "Do I process parent BEFORE or AFTER children?" — that decides pre/in/post order.

---

## C++ Tree Quick Reference

```cpp
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// Base case for almost every tree function:
if (!root) return ...;        // null check first
if (!root->left && !root->right) return ...;  // leaf node check
```

> **Key rules:**
> - Every tree function must handle `root == nullptr`
> - Recursion depth = tree height → O(n) worst case (skewed), O(log n) balanced
> - When combining left + right results, think about what the CURRENT node contributes

---

## Pattern 1: DFS Traversals (Pre / In / Post Order)

**When to think about it:**
- Need to visit all nodes in a specific order
- "Inorder of BST gives sorted output" — BST problems
- Preorder = serialize / copy a tree (parent before children)
- Postorder = delete tree / evaluate expression tree (children before parent)
- Any problem where you need to process a node relative to its subtrees

**Core insight:**
- **Preorder** (root → left → right): process node BEFORE subtrees
- **Inorder** (left → root → right): process node BETWEEN subtrees (BST sorted order)
- **Postorder** (left → right → root): process node AFTER subtrees (need children info first)

```
Tree:        1
            / \
           2   3
          / \
         4   5

Preorder:  1 2 4 5 3   (root first)
Inorder:   4 2 5 1 3   (left, root, right)
Postorder: 4 5 2 3 1   (root last)
```

### Recursive Templates (memorize these)
```cpp
// PREORDER
void preorder(TreeNode* root, vector<int>& res) {
    if (!root) return;
    res.push_back(root->val);       // visit ROOT first
    preorder(root->left, res);
    preorder(root->right, res);
}

// INORDER
void inorder(TreeNode* root, vector<int>& res) {
    if (!root) return;
    inorder(root->left, res);
    res.push_back(root->val);       // visit ROOT between
    inorder(root->right, res);
}

// POSTORDER
void postorder(TreeNode* root, vector<int>& res) {
    if (!root) return;
    postorder(root->left, res);
    postorder(root->right, res);
    res.push_back(root->val);       // visit ROOT last
}
```

### Iterative Templates (asked in interviews)
```cpp
// ITERATIVE INORDER
vector<int> inorderIterative(TreeNode* root) {
    stack<TreeNode*> st;
    vector<int> res;
    TreeNode* cur = root;

    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }  // go left
        cur = st.top(); st.pop();
        res.push_back(cur->val);                         // visit
        cur = cur->right;                                // go right
    }
    return res;
}

// ITERATIVE PREORDER
vector<int> preorderIterative(TreeNode* root) {
    if (!root) return {};
    stack<TreeNode*> st;
    vector<int> res;
    st.push(root);

    while (!st.empty()) {
        auto node = st.top(); st.pop();
        res.push_back(node->val);                        // visit
        if (node->right) st.push(node->right);           // right first (LIFO)
        if (node->left)  st.push(node->left);
    }
    return res;
}

// ITERATIVE POSTORDER (reverse of modified preorder)
vector<int> postorderIterative(TreeNode* root) {
    if (!root) return {};
    stack<TreeNode*> st;
    vector<int> res;
    st.push(root);

    while (!st.empty()) {
        auto node = st.top(); st.pop();
        res.push_back(node->val);
        if (node->left)  st.push(node->left);            // left first this time
        if (node->right) st.push(node->right);
    }
    reverse(res.begin(), res.end());                     // reverse at end
    return res;
}
```

**Problems:** #94 Inorder, #144 Preorder, #145 Postorder, #589 N-ary Preorder, #590 N-ary Postorder

---

## Pattern 2: BFS / Level Order Traversal

**When to think about it:**
- "Level by level", "row by row", "zigzag", "right side view"
- Find the shortest path in an unweighted tree
- Connect nodes at the same level
- Any problem where depth/level matters

**Core insight:** Use a queue. Process all nodes at current level before moving to next.
Track level size (`int sz = q.size()`) to know when one level ends and next begins.

```
Tree:    1
        / \
       2   3
      / \   \
     4   5   6

Level 0: [1]
Level 1: [2, 3]
Level 2: [4, 5, 6]
```

### Universal BFS Template
```cpp
vector<vector<int>> levelOrder(TreeNode* root) {
    if (!root) return {};
    queue<TreeNode*> q;
    vector<vector<int>> res;
    q.push(root);

    while (!q.empty()) {
        int sz = q.size();              // number of nodes at this level
        vector<int> level;

        for (int i = 0; i < sz; i++) {
            auto node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}
```

### Variants from the same template
```cpp
// RIGHT SIDE VIEW — take last element of each level
vector<int> rightSideView(TreeNode* root) {
    if (!root) return {};
    queue<TreeNode*> q;
    vector<int> res;
    q.push(root);

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            auto node = q.front(); q.pop();
            if (i == sz - 1) res.push_back(node->val);  // last of level
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
    return res;
}

// ZIGZAG LEVEL ORDER — alternate direction each level
vector<vector<int>> zigzagLevelOrder(TreeNode* root) {
    if (!root) return {};
    queue<TreeNode*> q;
    vector<vector<int>> res;
    bool leftToRight = true;
    q.push(root);

    while (!q.empty()) {
        int sz = q.size();
        vector<int> level(sz);
        for (int i = 0; i < sz; i++) {
            auto node = q.front(); q.pop();
            int idx = leftToRight ? i : sz - 1 - i;    // index direction
            level[idx] = node->val;
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
        leftToRight = !leftToRight;
    }
    return res;
}

// MINIMUM DEPTH — first time we hit a leaf
int minDepth(TreeNode* root) {
    if (!root) return 0;
    queue<TreeNode*> q;
    q.push(root);
    int depth = 1;

    while (!q.empty()) {
        int sz = q.size();
        for (int i = 0; i < sz; i++) {
            auto node = q.front(); q.pop();
            if (!node->left && !node->right) return depth;  // first leaf
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        depth++;
    }
    return depth;
}
```

**Problems:** #102 Level Order, #103 Zigzag, #199 Right Side View, #111 Min Depth, #116 Connect Next Pointers, #107 Level Order II, #637 Average of Levels

---

## Pattern 3: DFS Path Problems

**When to think about it:**
- "Does a root-to-leaf path with sum X exist?"
- "Find all root-to-leaf paths"
- "Maximum path sum through any node"
- "Diameter of tree"
- Path starts at root OR can start/end at any node (two different sub-patterns)

**Core insight:**
- **Root-to-leaf path**: carry a running sum/string down, check at leaf
- **Any-node path**: at each node, ask "what's the best path through ME?" using left + right results — this is Tree DP

### Root-to-Leaf Path Sum
```cpp
bool hasPathSum(TreeNode* root, int target) {
    if (!root) return false;
    if (!root->left && !root->right)        // leaf
        return root->val == target;

    return hasPathSum(root->left,  target - root->val) ||
           hasPathSum(root->right, target - root->val);
}

// All root-to-leaf paths
void dfs(TreeNode* root, string path, vector<string>& res) {
    if (!root) return;
    path += to_string(root->val);
    if (!root->left && !root->right) { res.push_back(path); return; }
    dfs(root->left,  path + "->", res);
    dfs(root->right, path + "->", res);
}
```

### Maximum Path Sum (any node to any node) — Tree DP
```cpp
int maxPathSum(TreeNode* root) {
    int globalMax = INT_MIN;

    function<int(TreeNode*)> dfs = [&](TreeNode* node) -> int {
        if (!node) return 0;

        int left  = max(0, dfs(node->left));   // ignore negative paths
        int right = max(0, dfs(node->right));

        // Path through current node (can go left + node + right)
        globalMax = max(globalMax, left + right + node->val);

        // Return only ONE side (path must be linear upward)
        return node->val + max(left, right);
    };

    dfs(root);
    return globalMax;
}
```

### Diameter of Binary Tree — Tree DP
```cpp
int diameterOfBinaryTree(TreeNode* root) {
    int diameter = 0;

    function<int(TreeNode*)> depth = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int left  = depth(node->left);
        int right = depth(node->right);
        diameter = max(diameter, left + right);   // path through this node
        return 1 + max(left, right);              // return height upward
    };

    depth(root);
    return diameter;
}
```

**The key insight for "any-node" paths:**
```
At every node, compute:
  left  = best gain from left subtree  (max with 0 — don't use negative)
  right = best gain from right subtree (max with 0)

Update global answer: left + node->val + right  (full path through node)
Return to parent:     node->val + max(left, right)  (can only go ONE way up)
```

**Problems:** #112 Path Sum, #113 Path Sum II, #257 Binary Tree Paths, #124 Max Path Sum, #543 Diameter, #687 Longest Univalue Path

---

## Pattern 4: Tree DP (Combining Left + Right)

**When to think about it:**
- Answer for a node depends on answers from BOTH subtrees
- "Height", "diameter", "balanced", "max path", "rob the house on tree"
- You return something useful to the parent from each subtree call

**Core insight:** The recursive function does double duty:
1. Updates a global answer using both left and right results
2. Returns a value the parent needs (usually height or max gain)

### Generic Tree DP Template
```cpp
int globalAns = 0;  // or INT_MIN depending on problem

int solve(TreeNode* node) {
    if (!node) return 0;  // base value for null

    int left  = solve(node->left);
    int right = solve(node->right);

    // Combine left + right + current node to update global answer
    globalAns = max(globalAns, /* expression using left, right, node->val */);

    // Return value that parent needs (usually height or max one-sided path)
    return /* value for parent */;
}
```

### Check if Tree is Balanced
```cpp
bool isBalanced(TreeNode* root) {
    function<int(TreeNode*)> height = [&](TreeNode* node) -> int {
        if (!node) return 0;
        int left  = height(node->left);
        if (left == -1) return -1;           // already unbalanced
        int right = height(node->right);
        if (right == -1) return -1;
        if (abs(left - right) > 1) return -1; // unbalanced here
        return 1 + max(left, right);
    };
    return height(root) != -1;
}
```

### House Robber III (can't rob parent + child together)
```cpp
int rob(TreeNode* root) {
    function<pair<int,int>(TreeNode*)> dfs = [&](TreeNode* node) -> pair<int,int> {
        if (!node) return {0, 0};

        auto [lRob, lSkip] = dfs(node->left);
        auto [rRob, rSkip] = dfs(node->right);

        int rob  = node->val + lSkip + rSkip;  // take this node, skip children
        int skip = max(lRob, lSkip) + max(rRob, rSkip);  // skip this, best of children

        return {rob, skip};
    };

    auto [rob, skip] = dfs(root);
    return max(rob, skip);
}
```

**Problems:** #104 Max Depth, #110 Balanced, #543 Diameter, #124 Max Path Sum, #337 House Robber III, #1372 Longest Zigzag Path

---

## Pattern 5: Lowest Common Ancestor (LCA)

**When to think about it:**
- "Find LCA of two nodes p and q"
- "Distance between two nodes"
- "Path between two nodes"
- The LCA is the deepest node that has both p and q in its subtree

**Core insight:** If current node is p or q, return it. Otherwise, recurse both sides.
- Both sides return non-null → current node IS the LCA
- Only one side returns non-null → LCA is in that subtree

```
Tree:      3
          / \
         5   1
        / \ / \
       6  2 0  8
         / \
        7   4

LCA(5, 1) = 3   (5 is left, 1 is right, meet at root)
LCA(5, 4) = 5   (4 is in subtree of 5, so 5 is LCA)
```

### LCA in Binary Tree
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;  // found one of them

    TreeNode* left  = lowestCommonAncestor(root->left,  p, q);
    TreeNode* right = lowestCommonAncestor(root->right, p, q);

    if (left && right) return root;   // p in left, q in right -> root is LCA
    return left ? left : right;       // both on same side
}
```

### LCA in BST (use sorted property — faster)
```cpp
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (p->val < root->val && q->val < root->val)
        return lowestCommonAncestor(root->left, p, q);   // both in left

    if (p->val > root->val && q->val > root->val)
        return lowestCommonAncestor(root->right, p, q);  // both in right

    return root;  // split point = LCA
}

// Iterative BST LCA
TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
    while (root) {
        if (p->val < root->val && q->val < root->val) root = root->left;
        else if (p->val > root->val && q->val > root->val) root = root->right;
        else return root;
    }
    return nullptr;
}
```

**Problems:** #236 LCA of Binary Tree, #235 LCA of BST, #1644 LCA II, #1650 LCA with Parent Pointer, #1676 LCA of Multiple Nodes

---

## Pattern 6: BST Patterns

**When to think about it:**
- Tree has the BST property: left < node < right
- "Search", "insert", "delete", "validate", "kth smallest"
- Inorder traversal of BST gives sorted sequence — use this constantly

**Core insight:** BST inorder = sorted array. Every BST problem can be reduced to
thinking about the sorted order. Use the BST property to prune half the tree at each step.

### Validate BST
```cpp
// Wrong: just check node->left->val < node->val (misses deeper violations)
// Right: pass min/max bounds down

bool isValidBST(TreeNode* root, long min = LONG_MIN, long max = LONG_MAX) {
    if (!root) return true;
    if (root->val <= min || root->val >= max) return false;
    return isValidBST(root->left,  min, root->val) &&
           isValidBST(root->right, root->val, max);
}
```

### Kth Smallest in BST (inorder = sorted)
```cpp
int kthSmallest(TreeNode* root, int k) {
    int count = 0, result = 0;

    function<void(TreeNode*)> inorder = [&](TreeNode* node) {
        if (!node) return;
        inorder(node->left);
        if (++count == k) { result = node->val; return; }
        inorder(node->right);
    };

    inorder(root);
    return result;
}
```

### Convert Sorted Array to BST
```cpp
TreeNode* sortedArrayToBST(vector<int>& nums, int l, int r) {
    if (l > r) return nullptr;
    int mid = l + (r - l) / 2;
    TreeNode* root = new TreeNode(nums[mid]);
    root->left  = sortedArrayToBST(nums, l, mid - 1);
    root->right = sortedArrayToBST(nums, mid + 1, r);
    return root;
}
```

### Delete Node in BST
```cpp
TreeNode* deleteNode(TreeNode* root, int key) {
    if (!root) return nullptr;

    if (key < root->val) {
        root->left = deleteNode(root->left, key);
    } else if (key > root->val) {
        root->right = deleteNode(root->right, key);
    } else {
        // Found node to delete
        if (!root->left)  return root->right;  // no left child
        if (!root->right) return root->left;   // no right child

        // Has both children: replace with inorder successor (min of right subtree)
        TreeNode* successor = root->right;
        while (successor->left) successor = successor->left;
        root->val   = successor->val;
        root->right = deleteNode(root->right, successor->val);
    }
    return root;
}
```

### Recover BST (two nodes swapped)
```cpp
void recoverTree(TreeNode* root) {
    TreeNode* first = nullptr, *second = nullptr, *prev = nullptr;

    function<void(TreeNode*)> inorder = [&](TreeNode* node) {
        if (!node) return;
        inorder(node->left);
        if (prev && prev->val > node->val) {
            if (!first) first = prev;   // first violation: bigger node
            second = node;              // second violation: smaller node
        }
        prev = node;
        inorder(node->right);
    };

    inorder(root);
    swap(first->val, second->val);
}
```

**Problems:** #98 Validate BST, #230 Kth Smallest, #108 Sorted Array to BST, #450 Delete Node, #99 Recover BST, #173 BST Iterator, #501 Find Mode in BST

---

## Pattern 7: Tree Construction

**When to think about it:**
- "Construct binary tree from preorder + inorder"
- "Construct binary tree from preorder + postorder"
- "Serialize and deserialize binary tree"
- Given two traversal arrays, rebuild the original tree

**Core insight:**
- First element of preorder = root
- Find root in inorder → elements left of it = left subtree, right = right subtree
- Use a hashmap for O(1) inorder index lookup

### Build from Preorder + Inorder
```cpp
TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
    unordered_map<int, int> inIdx;
    for (int i = 0; i < inorder.size(); i++) inIdx[inorder[i]] = i;

    int preStart = 0;
    function<TreeNode*(int, int)> build = [&](int inL, int inR) -> TreeNode* {
        if (inL > inR) return nullptr;

        int rootVal = preorder[preStart++];         // next in preorder = root
        TreeNode* root = new TreeNode(rootVal);
        int mid = inIdx[rootVal];                   // root's position in inorder

        root->left  = build(inL, mid - 1);          // left subtree
        root->right = build(mid + 1, inR);          // right subtree
        return root;
    };

    return build(0, inorder.size() - 1);
}
```

### Build from Inorder + Postorder
```cpp
TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
    unordered_map<int, int> inIdx;
    for (int i = 0; i < inorder.size(); i++) inIdx[inorder[i]] = i;

    int postEnd = postorder.size() - 1;
    function<TreeNode*(int, int)> build = [&](int inL, int inR) -> TreeNode* {
        if (inL > inR) return nullptr;

        int rootVal = postorder[postEnd--];         // last in postorder = root
        TreeNode* root = new TreeNode(rootVal);
        int mid = inIdx[rootVal];

        root->right = build(mid + 1, inR);          // RIGHT first (postEnd goes backward)
        root->left  = build(inL, mid - 1);
        return root;
    };

    return build(0, inorder.size() - 1);
}
```

### Serialize / Deserialize Binary Tree
```cpp
// Serialize: preorder with "#" for nulls
string serialize(TreeNode* root) {
    if (!root) return "#,";
    return to_string(root->val) + "," + serialize(root->left) + serialize(root->right);
}

TreeNode* deserialize(string data) {
    queue<string> q;
    stringstream ss(data);
    string token;
    while (getline(ss, token, ',')) q.push(token);

    function<TreeNode*()> build = [&]() -> TreeNode* {
        string val = q.front(); q.pop();
        if (val == "#") return nullptr;
        TreeNode* node = new TreeNode(stoi(val));
        node->left  = build();
        node->right = build();
        return node;
    };
    return build();
}
```

**Problems:** #105 Build from Pre+In, #106 Build from In+Post, #297 Serialize/Deserialize, #449 Serialize BST, #1008 Construct BST from Preorder

---

## Pattern 8: Morris Traversal (O(1) Space)

**When to think about it:**
- Inorder/preorder traversal with O(1) space (no stack, no recursion)
- Rarely asked but can appear at Google/Meta for senior roles
- Key trick: temporarily thread right pointers to avoid stack

**Core insight:** Before going left, link the rightmost node of left subtree back to current node.
This creates a "thread" — a way to come back without a stack.

```cpp
vector<int> morrisInorder(TreeNode* root) {
    vector<int> res;
    TreeNode* cur = root;

    while (cur) {
        if (!cur->left) {
            res.push_back(cur->val);    // no left, visit and go right
            cur = cur->right;
        } else {
            // Find inorder predecessor (rightmost of left subtree)
            TreeNode* pred = cur->left;
            while (pred->right && pred->right != cur) pred = pred->right;

            if (!pred->right) {
                pred->right = cur;      // thread: link back to cur
                cur = cur->left;        // go left
            } else {
                pred->right = nullptr;  // remove thread
                res.push_back(cur->val);
                cur = cur->right;
            }
        }
    }
    return res;
}
```

**Problems:** #94 Inorder (O(1) space), #99 Recover BST (O(1) space), #501 Find Mode

---

## The 6 Most Important Templates to Memorize

These six cover ~85% of all tree interview questions:

```cpp
// 1. DFS (applies to any traversal / path problem)
void dfs(TreeNode* root) {
    if (!root) return;
    // preorder: process root here
    dfs(root->left);
    // inorder: process root here
    dfs(root->right);
    // postorder: process root here
}

// 2. BFS LEVEL ORDER
vector<vector<int>> bfs(TreeNode* root) {
    if (!root) return {};
    queue<TreeNode*> q;
    vector<vector<int>> res;
    q.push(root);
    while (!q.empty()) {
        int sz = q.size();
        vector<int> level;
        for (int i = 0; i < sz; i++) {
            auto node = q.front(); q.pop();
            level.push_back(node->val);
            if (node->left)  q.push(node->left);
            if (node->right) q.push(node->right);
        }
        res.push_back(level);
    }
    return res;
}

// 3. TREE DP (global answer updated at every node)
int ans = 0;
int treeDP(TreeNode* node) {
    if (!node) return 0;
    int left  = max(0, treeDP(node->left));
    int right = max(0, treeDP(node->right));
    ans = max(ans, left + right + node->val);   // update global
    return node->val + max(left, right);        // return to parent
}

// 4. LCA
TreeNode* lca(TreeNode* root, TreeNode* p, TreeNode* q) {
    if (!root || root == p || root == q) return root;
    TreeNode* left  = lca(root->left,  p, q);
    TreeNode* right = lca(root->right, p, q);
    if (left && right) return root;
    return left ? left : right;
}

// 5. VALIDATE BST (pass bounds)
bool validBST(TreeNode* root, long mn = LONG_MIN, long mx = LONG_MAX) {
    if (!root) return true;
    if (root->val <= mn || root->val >= mx) return false;
    return validBST(root->left, mn, root->val) &&
           validBST(root->right, root->val, mx);
}

// 6. BUILD TREE FROM PREORDER + INORDER
TreeNode* build(vector<int>& pre, vector<int>& in) {
    unordered_map<int,int> idx;
    for (int i = 0; i < in.size(); i++) idx[in[i]] = i;
    int p = 0;
    function<TreeNode*(int,int)> f = [&](int l, int r) -> TreeNode* {
        if (l > r) return nullptr;
        int v = pre[p++];
        TreeNode* node = new TreeNode(v);
        node->left  = f(l, idx[v] - 1);
        node->right = f(idx[v] + 1, r);
        return node;
    };
    return f(0, in.size() - 1);
}
```

---

## Universal Checklist Before Coding

- [ ] Handle `root == nullptr` as the very first line
- [ ] Which traversal order? Pre (top-down) / In (sorted BST) / Post (bottom-up)
- [ ] Is answer computed top-down (pass info down) or bottom-up (return info up)?
- [ ] Need a global variable? (Tree DP pattern — diameter, max path sum)
- [ ] For BST: use the sorted property to prune, don't treat as generic tree
- [ ] For level problems: BFS with `int sz = q.size()` to track level boundaries
- [ ] Trace through a small tree (3-5 nodes) by hand before coding

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 104 | Maximum Depth of Binary Tree          | Easy   | DFS / Tree DP            |
| 2  | 226 | Invert Binary Tree                    | Easy   | DFS preorder             |
| 3  | 572 | Subtree of Another Tree               | Easy   | DFS                      |
| 4  | 102 | Binary Tree Level Order Traversal     | Medium | BFS                      |
| 5  | 199 | Binary Tree Right Side View           | Medium | BFS (last of level)      |
| 6  | 543 | Diameter of Binary Tree               | Easy   | Tree DP                  |
| 7  | 112 | Path Sum                              | Easy   | DFS path                 |
| 8  | 236 | LCA of Binary Tree                    | Medium | LCA pattern              |
| 9  | 98  | Validate Binary Search Tree           | Medium | BST + bounds             |
| 10 | 230 | Kth Smallest in BST                   | Medium | BST inorder              |
| 11 | 105 | Build Tree from Pre + Inorder         | Medium | Tree construction        |
| 12 | 124 | Binary Tree Maximum Path Sum          | Hard   | Tree DP                  |
| 13 | 235 | LCA of BST                            | Easy   | BST LCA                  |
| 14 | 297 | Serialize and Deserialize Binary Tree | Hard   | Serialization            |
| 15 | 103 | Zigzag Level Order                    | Medium | BFS variant              |
| 16 | 110 | Balanced Binary Tree                  | Easy   | Tree DP (height)         |
| 17 | 450 | Delete Node in BST                    | Medium | BST operations           |
| 18 | 337 | House Robber III                      | Medium | Tree DP                  |
| 19 | 99  | Recover BST                           | Hard   | BST inorder              |
| 20 | 108 | Sorted Array to BST                   | Easy   | Tree construction        |
| 21 | 116 | Populating Next Right Pointers        | Medium | BFS / DFS                |
| 22 | 114 | Flatten Binary Tree to Linked List    | Medium | DFS postorder            |
| 23 | 101 | Symmetric Tree                        | Easy   | DFS                      |
| 24 | 1448| Count Good Nodes in Binary Tree       | Medium | DFS (pass max down)      |
| 25 | 173 | BST Iterator                          | Medium | BST inorder + stack      |

**Study order:** #104 -> #226 -> #102 -> #543 -> #112 -> #236 -> #98 -> #230 -> #105 -> #124 -> #297
These eleven cover every core pattern.

---

## How Patterns Combine (Hard / Senior-Level Problems)

| Problem | Patterns Combined |
|---------|------------------|
| #124 Max Path Sum | Tree DP + global answer |
| #543 Diameter | Tree DP + height |
| #297 Serialize / Deserialize | Preorder DFS + queue reconstruction |
| #105 Build from Pre+In | Tree construction + hashmap |
| #99 Recover BST | BST inorder + find two swapped nodes |
| #114 Flatten to Linked List | Postorder + pointer reconnect |
| #337 House Robber III | Tree DP + include/exclude choice |

---

## Company-Wise Most Asked Problems

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #102 Level Order, #104 Max Depth, #236 LCA, #98 Validate BST, #112 Path Sum |
| **Google** | #297 Serialize/Deserialize, #124 Max Path Sum, #236 LCA, #99 Recover BST, #543 Diameter |
| **Meta** | #199 Right Side View, #543 Diameter, #124 Max Path Sum, #236 LCA, #102 Level Order |
| **Microsoft** | #226 Invert, #104 Max Depth, #102 Level Order, #110 Balanced, #98 Validate BST |
| **Bloomberg** | #102 Level Order, #103 Zigzag, #230 Kth Smallest, #98 Validate BST |
| **Apple** | #104 Max Depth, #226 Invert, #543 Diameter, #112 Path Sum |
| **Uber** | #297 Serialize/Deserialize, #124 Max Path Sum, #105 Build Tree |

---

## Recent Interview Trends (2023–2025)

### 1. Tree DP is the most tested hard pattern
Max path sum, diameter, house robber on tree, longest zigzag — all use the same
Tree DP skeleton. Master the "update global, return to parent" pattern.

### 2. BFS variants are very common at Meta/Amazon
Right side view, zigzag, level averages, connect next pointers — all come from the
same BFS template. Just change what you do inside the level loop.

### 3. Serialize/Deserialize is a Google favorite
#297 appears repeatedly in Google interviews. It tests preorder DFS + string parsing.
Also tests your ability to handle edge cases (null nodes, negative values).

### 4. LCA is asked everywhere, in many forms
Plain LCA (#236), BST LCA (#235), LCA with parent pointer (#1650), LCA of multiple nodes (#1676).
Know the core LCA logic and you can adapt to all variants.

### 5. BST + Iterator = system design lite
#173 BST Iterator (return next in-order element) is asked at Google/Meta as a
design question. Uses a stack to simulate inorder traversal lazily.

### 6. "Flatten" problems bridge trees and linked lists
#114 Flatten Binary Tree to Linked List appears at Amazon/Microsoft.
Tests postorder thinking + pointer manipulation together.

### 7. Trie (Prefix Tree) is now a tree sub-topic in interviews
#208 Implement Trie, #212 Word Search II, #211 Design Add/Search Words —
these appear frequently at Google/Meta and use a tree of characters.

```cpp
// Trie Node
struct TrieNode {
    unordered_map<char, TrieNode*> children;
    bool isEnd = false;
};

class Trie {
    TrieNode* root;
public:
    Trie() { root = new TrieNode(); }

    void insert(string word) {
        TrieNode* cur = root;
        for (char c : word) {
            if (!cur->children.count(c))
                cur->children[c] = new TrieNode();
            cur = cur->children[c];
        }
        cur->isEnd = true;
    }

    bool search(string word) {
        TrieNode* cur = root;
        for (char c : word) {
            if (!cur->children.count(c)) return false;
            cur = cur->children[c];
        }
        return cur->isEnd;
    }

    bool startsWith(string prefix) {
        TrieNode* cur = root;
        for (char c : prefix) {
            if (!cur->children.count(c)) return false;
            cur = cur->children[c];
        }
        return true;
    }
};
```

**Trie problems:** #208 Implement Trie, #211 Design Add/Search Words, #212 Word Search II, #648 Replace Words

---

## Complexity Reference

| Pattern              | Time       | Space      | Key Insight                                    |
|----------------------|------------|------------|------------------------------------------------|
| DFS Traversal        | O(n)       | O(h)       | h = height, O(n) worst (skewed), O(log n) balanced |
| BFS Level Order      | O(n)       | O(w)       | w = max width, O(n) worst case                 |
| Tree DP              | O(n)       | O(h)       | Each node visited once                         |
| LCA                  | O(n)       | O(h)       | O(log n) for balanced BST                      |
| BST Search/Insert    | O(h)       | O(h)       | O(log n) balanced, O(n) skewed                 |
| Build from Traversals| O(n)       | O(n)       | HashMap for O(1) index lookup                  |
| Serialize/Deserialize| O(n)       | O(n)       | One pass each way                              |
| Morris Traversal     | O(n)       | O(1)       | Threads pointers, no stack                     |
| Trie Insert/Search   | O(L)       | O(L)       | L = word length                                |
