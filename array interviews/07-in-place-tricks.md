# Pattern 7: In-Place Tricks, Cyclic Sort & Floyd's Cycle

## Covered
1. Cyclic Sort — find missing/duplicate numbers in O(n) O(1)
2. Floyd's Cycle Detection — find duplicate in array as linked list
3. Next Permutation — generate next lexicographic arrangement
4. Rotate Array — three reverses trick
5. Set Matrix Zeroes — use first row/col as marker

---

## 1. Cyclic Sort

### Core Idea
If the array contains values in range [1, n], each value `nums[i]` should be at index `nums[i] - 1`.
Swap each element to its correct position. Missing values will be gaps after sorting.

### Template
```cpp
void cyclicSort(vector<int>& nums) {
    int i = 0;
    while (i < nums.size()) {
        int correct = nums[i] - 1;   // where nums[i] should go
        if (nums[i] != nums[correct]) swap(nums[i], nums[correct]);
        else i++;
    }
}
// After sort: if nums[i] != i+1 → i+1 is missing, nums[i] is duplicate
```

### Problems
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [268](https://leetcode.com/problems/missing-number/) | Missing Number | Cyclic sort [0..n]. After sort, find where `nums[i] != i` | Or use math: `n*(n+1)/2 - sum` |
| [448](https://leetcode.com/problems/find-all-numbers-disappeared-in-an-array/) | Disappeared Numbers | Cyclic sort, then collect indices where `nums[i] != i+1` | O(n) time, O(1) extra space |
| [442](https://leetcode.com/problems/find-all-duplicates-in-an-array/) | Find All Duplicates | Cyclic sort, collect positions where `nums[i] != i+1` | Values 1..n each appear 1 or 2 times |
| [41](https://leetcode.com/problems/first-missing-positive/) | First Missing Positive | Cyclic sort values in [1,n]. Scan for first `nums[i] != i+1` | Ignore values outside [1,n] during sort |

---

## 2. Floyd's Cycle Detection on Array

### Core Idea
Array values in [1, n] with one duplicate → treat array as a linked list where index → value → next index.
Duplicate value = two edges pointing to same node → cycle.

### Template (same as linked list cycle detection)
```cpp
int findDuplicate(vector<int>& nums) {
    // Phase 1: Find meeting point inside cycle
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);

    // Phase 2: Find cycle entry (= duplicate value)
    slow = nums[0];
    while (slow != fast) {
        slow = nums[slow];
        fast = nums[fast];
    }
    return slow;
}
```

### Problems
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [287](https://leetcode.com/problems/find-the-duplicate-number/) | Find the Duplicate Number | Floyd's on array. No modification, O(1) space | Cycle exists because one value maps to two positions |

---

## 3. Next Permutation

### Core Idea: 3 Steps
```
1. Find rightmost index i where nums[i] < nums[i+1]  (descending from right ends here)
2. Find rightmost j > i where nums[j] > nums[i]       (smallest larger element on the right)
3. Swap nums[i] and nums[j]. Reverse suffix from i+1.
```

### Template
```cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), i = n - 2;
    while (i >= 0 && nums[i] >= nums[i+1]) i--;  // step 1

    if (i >= 0) {
        int j = n - 1;
        while (nums[j] <= nums[i]) j--;            // step 2
        swap(nums[i], nums[j]);
    }
    reverse(nums.begin() + i + 1, nums.end());     // step 3
}
// If all descending (no step 1 found): just reverse entire array
```

### Problems
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [31](https://leetcode.com/problems/next-permutation/) | Next Permutation | 3-step: find descent, find swap target, reverse suffix | If no descent found (fully descending), reverse entire array |
| [60](https://leetcode.com/problems/permutation-sequence/) | Permutation Sequence | Use factorial number system. For each position, pick the `k/fact(n-1)`th remaining element | O(n²) with list; O(n log n) with BIT |

---

## 4. Rotate Array

### Three Reverses Trick
```cpp
// Rotate right by k
void rotate(vector<int>& nums, int k) {
    int n = nums.size();
    k %= n;
    reverse(nums.begin(), nums.end());          // reverse all
    reverse(nums.begin(), nums.begin() + k);    // reverse first k
    reverse(nums.begin() + k, nums.end());      // reverse rest
}
// Same trick works for: rotate string, rotate matrix rows
```

### Rotate Matrix (90° clockwise)
```cpp
// Step 1: Transpose (swap [i][j] with [j][i])
// Step 2: Reverse each row
void rotate(vector<vector<int>>& matrix) {
    int n = matrix.size();
    for (int i = 0; i < n; i++)
        for (int j = i+1; j < n; j++)
            swap(matrix[i][j], matrix[j][i]);
    for (auto& row : matrix) reverse(row.begin(), row.end());
}
```

### Problems
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [189](https://leetcode.com/problems/rotate-array/) | Rotate Array | Three reverses: all → first k → rest | `k %= n` first to handle k > n |
| [48](https://leetcode.com/problems/rotate-image/) | Rotate Image | Transpose then reverse each row (90° CW) | Or: reverse rows then transpose (90° CCW) |

---

## 5. Set Matrix Zeroes & In-Place Markers

### Use Array Itself as Extra Space
```cpp
// Set Matrix Zeroes: O(1) extra space
// Use first row and first column as markers
void setZeroes(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    bool firstRow = false, firstCol = false;

    // Check if first row/col should be zeroed
    for (int j = 0; j < n; j++) if (matrix[0][j] == 0) firstRow = true;
    for (int i = 0; i < m; i++) if (matrix[i][0] == 0) firstCol = true;

    // Use first row/col to mark zeros
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            if (matrix[i][j] == 0) { matrix[i][0] = 0; matrix[0][j] = 0; }

    // Zero out based on markers
    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            if (matrix[i][0] == 0 || matrix[0][j] == 0) matrix[i][j] = 0;

    // Handle first row and col
    if (firstRow) for (int j = 0; j < n; j++) matrix[0][j] = 0;
    if (firstCol) for (int i = 0; i < m; i++) matrix[i][0] = 0;
}
```

### Negative Marking (mark visited cells without extra space)
```cpp
// For arrays with values in known positive range:
// Negate nums[abs(val)-1] to mark that val has been seen
for (int x : nums) {
    int idx = abs(x) - 1;
    nums[idx] = -abs(nums[idx]);   // mark as visited
}
for (int i = 0; i < n; i++)
    if (nums[i] > 0) result.push_back(i+1);  // not marked = missing
```

### Problems
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [73](https://leetcode.com/problems/set-matrix-zeroes/) | Set Matrix Zeroes | Use first row/col as O(1) marker. Two bool flags for first row/col themselves | Process in correct order to avoid overwriting markers |
| [289](https://leetcode.com/problems/game-of-life/) | Game of Life | Encode next state in the same cell using extra bits | `2 = was 0, becomes 1`. `3 = was 1, becomes 0`. Extract at end with `& 1` and `>> 1` |

---

## Common Mistakes

**Cyclic Sort:**
- Not handling values outside [1,n] — skip them (`if (nums[i] < 1 || nums[i] > n) i++`)
- Infinite loop if swapping a duplicate: `if (nums[i] != nums[correct]) swap...` — stop if values at both positions are equal

**Floyd's on array:**
- Starting both slow and fast from `nums[0]`, not `0` — the "linked list" starts at the first value, not index 0

**Next Permutation:**
- Using `nums[i] > nums[i+1]` instead of `>=` — equal elements must be considered descending

**Rotate Matrix:**
- Transposing with wrong indices — transpose swaps `[i][j]` with `[j][i]`, only for `j > i`

## Follow-up Interviewers Love

- "First Missing Positive — can you do better than cyclic sort?" → Not without modifying the array; the constraint O(1) space forces index tricks
- "Find the Duplicate — why Floyd's and not just sorting?" → Sorting modifies array and is O(n log n); constraint says O(1) space and can't modify input
- "Game of Life — how to do it in-place?" → Negative marking / bit manipulation to store old and new state simultaneously
