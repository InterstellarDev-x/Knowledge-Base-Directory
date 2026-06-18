# Pattern 5: Dutch National Flag & Partition Algorithms

## What Falls Under This Pattern
- **Dutch National Flag (DNF):** 3-way partition of an array into 3 groups in O(n) and O(1) space
- **Lomuto / Hoare Partition:** used internally in QuickSort
- **Boyer-Moore Voting:** find majority element in O(n) time, O(1) space
- **Flood Fill / 3-color partition:** any "sort into 3 groups without extra space" problem

---

## 1. Dutch National Flag Algorithm

### The Problem
Sort an array of 0s, 1s, and 2s in a single pass without counting.

### Core Idea
Three regions maintained by three pointers:
```
[0..low-1]  = all 0s
[low..mid-1] = all 1s
[mid..high] = unknown (processing zone)
[high+1..n-1] = all 2s
```

### Template
```cpp
void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = nums.size() - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low++], nums[mid++]);  // 0 → swap to front, advance both
        } else if (nums[mid] == 1) {
            mid++;                            // 1 → already in place
        } else {
            swap(nums[mid], nums[high--]);    // 2 → swap to back, DON'T advance mid
        }
    }
}
```

**Why not advance `mid` after swapping with `high`?**
The element swapped from `high` is unknown — it needs to be checked. So `mid` stays until the new value is processed.

### Generalization: k-color partition
```
Same idea with k pointers for k groups.
Each pass handles one "color", like counting sort but in-place.
```

---

## 2. Boyer-Moore Voting Algorithm

### The Problem
Find the majority element (appears > n/2 times) in O(n) time, O(1) space.

### Core Idea
```
Votes cancel out: one majority vote vs one minority vote → net 0.
Since majority > n/2, after all cancellations, the candidate remains.
```

### Template
```cpp
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], count = 1;
    for (int i = 1; i < nums.size(); i++) {
        if (count == 0) { candidate = nums[i]; count = 1; }
        else if (nums[i] == candidate) count++;
        else count--;
    }
    return candidate;
    // If not guaranteed majority exists, verify with a second pass
}
```

### Extended: Find All Majority Elements (> n/3)
```cpp
// At most 2 elements can appear > n/3 times
// Use 2 candidates + 2 counts
int candidate1 = INT_MIN, count1 = 0;
int candidate2 = INT_MIN, count2 = 0;

for (int x : nums) {
    if (x == candidate1) count1++;
    else if (x == candidate2) count2++;
    else if (count1 == 0) { candidate1 = x; count1 = 1; }
    else if (count2 == 0) { candidate2 = x; count2 = 1; }
    else { count1--; count2--; }
}
// Verify both candidates with a second pass
```

---

## 3. Partition (QuickSort building block)

### Lomuto Partition (simpler, pivot at end)
```cpp
int partition(vector<int>& nums, int lo, int hi) {
    int pivot = nums[hi];
    int i = lo;  // i = next position for elements <= pivot
    for (int j = lo; j < hi; j++) {
        if (nums[j] <= pivot) swap(nums[i++], nums[j]);
    }
    swap(nums[i], nums[hi]);  // place pivot
    return i;  // pivot's final position
}
```

### Hoare Partition (faster in practice, pivot at start)
```cpp
int partition(vector<int>& nums, int lo, int hi) {
    int pivot = nums[lo];
    int i = lo - 1, j = hi + 1;
    while (true) {
        do { i++; } while (nums[i] < pivot);
        do { j--; } while (nums[j] > pivot);
        if (i >= j) return j;
        swap(nums[i], nums[j]);
    }
}
```

### Quick Select (kth smallest in O(n) average)
```cpp
int quickSelect(vector<int>& nums, int lo, int hi, int k) {
    if (lo == hi) return nums[lo];
    int pivot = partition(nums, lo, hi);
    if (pivot == k) return nums[k];
    else if (pivot < k) return quickSelect(nums, pivot+1, hi, k);
    else return quickSelect(nums, lo, pivot-1, k);
}
// kth smallest: quickSelect(nums, 0, n-1, k-1)
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [75](https://leetcode.com/problems/sort-colors/) | Sort Colors | Dutch National Flag: 3 pointers | Classic DNF — the whole point of the algorithm |
| [169](https://leetcode.com/problems/majority-element/) | Majority Element | Boyer-Moore Voting | Count cancellation, single pass |
| [283](https://leetcode.com/problems/move-zeroes/) | Move Zeroes | DNF variant: 2-way partition (0s and non-0s) | Write pointer / slow-fast pattern |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [229](https://leetcode.com/problems/majority-element-ii/) | Majority Element II (> n/3) | Boyer-Moore with 2 candidates | At most 2 elements can exceed n/3. Verify both. |
| [215](https://leetcode.com/problems/kth-largest-element-in-an-array/) | Kth Largest Element | QuickSelect: O(n) avg, O(n²) worst. Or min-heap: O(n log k) | QuickSelect in interviews — mention both approaches |
| [905](https://leetcode.com/problems/sort-array-by-parity/) | Sort Array by Parity | 2-way partition: evens left, odds right | DNF with 2 groups |
| [922](https://leetcode.com/problems/sort-array-by-parity-ii/) | Sort Array by Parity II | Two pointers: one for even idx, one for odd idx | Fill even positions with even numbers |
| [324](https://leetcode.com/problems/wiggle-sort-ii/) | Wiggle Sort II | Sort + interleave from two halves | Or virtual index mapping with DNF |

---

## DNF Variants Summary

```
2-group partition:     slow/fast pointers (pattern 2 — two pointers)
3-group partition:     Dutch National Flag (low, mid, high)
Majority > n/2:        Boyer-Moore 1 candidate
Majority > n/3:        Boyer-Moore 2 candidates
Majority > n/k:        Boyer-Moore k-1 candidates
kth element:           QuickSelect (partition-based)
```

## Common Mistakes

**DNF:**
- Advancing `mid` after swapping with `high` — the swapped element is unknown, must recheck
- Using `<=` or `>=` instead of strict equality for the 3 groups

**Boyer-Moore:**
- Not doing a second verification pass (problem guarantees majority exists → skip; otherwise → required)
- Forgetting to reset `candidate` when `count == 0` (must also reset count to 1)

**QuickSelect:**
- Not handling the case where `lo == hi` (base case)
- Using 0-indexed k vs 1-indexed k — clarify before coding

## Follow-up Interviewers Love

- "Sort Colors — can you do it in one pass?" → DNF is exactly one pass
- "Kth Largest — what's the worst case for QuickSelect?" → O(n²) for sorted input with bad pivot. Use randomized pivot to make O(n) expected.
- "Majority Element — what if no majority is guaranteed?" → Add a verification pass: count occurrences of candidate
- "What if k groups instead of 3?" → Generalize with k pointers or multiple passes
