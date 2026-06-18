# Pattern 4: Path Problems

## Identify This Pattern
- "Root-to-leaf path with sum X"
- "All root-to-leaf paths"
- "Path sum equals target"
- "Count paths that sum to X" (any node start)

## Two Sub-Patterns

```
Root-to-leaf path only:
  → Carry running sum/string DOWN the tree
  → Check at leaf (when !left && !right)
  → Preorder DFS

Path from any node to any node:
  → This is Tree DP (see pattern 3)
  → At each node: combine left + right + current
  → Use a global variable
```

---

## Templates

### Root-to-Leaf: Check at Leaf
```cpp
bool hasPathSum(TreeNode* root, int target) {
    if (!root) return false;
    target -= root->val;
    if (!root->left && !root->right) return target == 0;  // leaf check
    return hasPathSum(root->left, target) || hasPathSum(root->right, target);
}
```

### Root-to-Leaf: Collect All Paths (backtracking)
```cpp
void dfs(TreeNode* node, vector<int>& path, vector<vector<int>>& res) {
    if (!node) return;
    path.push_back(node->val);                      // choose
    if (!node->left && !node->right) {
        res.push_back(path);                        // found leaf path
    }
    dfs(node->left,  path, res);
    dfs(node->right, path, res);
    path.pop_back();                                // unchoose (backtrack)
}
```

### Count Paths with Target Sum (any node, any direction)
```cpp
// Two approaches:
// 1. Prefix sum + hashmap (O(n))
// 2. Brute force from each node (O(n^2))

// Prefix sum approach
unordered_map<long long, int> prefixCount;
prefixCount[0] = 1;
int count = 0;

void dfs(TreeNode* node, long long curSum, int target) {
    if (!node) return;
    curSum += node->val;
    count += prefixCount[curSum - target];  // how many paths ending here sum to target
    prefixCount[curSum]++;
    dfs(node->left,  curSum, target);
    dfs(node->right, curSum, target);
    prefixCount[curSum]--;  // backtrack: remove this path's contribution
}
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [112](https://leetcode.com/problems/path-sum/) | Path Sum | Subtract val as you go down. At leaf, check if remainder == 0 | Remember: only count root-to-leaf, not root-to-any-node |
| [257](https://leetcode.com/problems/binary-tree-paths/) | Binary Tree Paths | DFS + string building. Add "->" before recursing | Backtrack with the string, or pass by value (simpler) |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [113](https://leetcode.com/problems/path-sum-ii/) | Path Sum II | DFS + backtracking with a path vector | Classic backtrack: push → recurse → pop |
| [437](https://leetcode.com/problems/path-sum-iii/) | Path Sum III | Prefix sum + hashmap. Track running sum, check `curSum - target` exists | Must decrement map on backtrack, or counts leak into sibling branches |
| [129](https://leetcode.com/problems/sum-root-to-leaf-numbers/) | Sum Root to Leaf Numbers | Carry `num = num * 10 + node->val` down | At leaf: add num to total. Standard root-to-leaf carry pattern |
| [1457](https://leetcode.com/problems/pseudo-palindromic-paths-in-a-binary-tree/) | Pseudo-Palindromic Paths | Carry a bitmask of odd-count digits. At leaf: at most 1 bit set = palindrome possible | XOR bit for each digit. `bitmask & (bitmask-1) == 0` checks at most 1 bit set |
| [988](https://leetcode.com/problems/smallest-string-starting-from-leaf/) | Smallest String From Leaf | Build string from leaf to root. Compare lexicographically | Reverse the path string or build it bottom-up |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [124](https://leetcode.com/problems/binary-tree-maximum-path-sum/) | Max Path Sum | See Tree DP pattern — path can go through any node | `max(0, left) + node->val + max(0, right)` at each node |
| [687](https://leetcode.com/problems/longest-univalue-path/) | Longest Univalue Path | At each node: extend only if child has same val | Return longest arm to parent. Global = left_arm + right_arm |

---

## Common Mistakes

**Root-to-leaf vs any-node confusion:**
- #112 Path Sum → root-to-LEAF only. Not root-to-any-node. Check `!left && !right`.
- #437 Path Sum III → ANY node to ANY node (downward only). Needs prefix sum.

**Backtracking leak (#437):**
- Must do `prefixCount[curSum]--` after both recursive calls
- If you forget, sibling branches inherit this path's prefix counts

**String paths (#257):**
- Passing `string path` by value (copying) is cleaner than push/pop for strings
- For large trees, pass by reference with explicit backtrack

---

## Optimizations

| Problem | Brute Force | Optimal |
|---------|------------|---------|
| #437 Path Sum III | O(n²) from each node | O(n) prefix sum + hashmap |
| #112 Path Sum | O(n) — already optimal | — |
| #113 Path Sum II | O(n * h) — unavoidable (must copy path at leaf) | — |

**For #437:** The prefix sum trick is the key optimization to mention in an interview.
```
prefixCount[0] = 1    ← the "empty prefix" — catches paths starting at root
count += prefixCount[curSum - target]
prefixCount[curSum]++
... recurse ...
prefixCount[curSum]-- ← MUST backtrack
```

**Follow-up interviewers love:**
- "#112 — what if path doesn't need to end at a leaf?" → Remove the leaf check, adjust base case
- "#437 — why do you decrement the map?" → To prevent sibling branches from using this node's prefix
- "#113 — complexity?" → O(n * h), because copying path at each leaf costs O(h)
