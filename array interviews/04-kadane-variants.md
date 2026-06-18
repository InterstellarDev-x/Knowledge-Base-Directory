# Pattern 4: Kadane's Algorithm & Variants

## Identify This Pattern
- "Maximum sum contiguous subarray"
- "Maximum product subarray"
- "Maximum sum circular subarray"
- Any "max/min over all subarrays" where the optimal choice at each position only depends on the previous state

## Core Insight of Kadane's
```
At each index i, either:
  1. Extend the previous best subarray: curSum + nums[i]
  2. Start a fresh subarray here:       nums[i]

dp[i] = max(nums[i], dp[i-1] + nums[i])
      = max(nums[i], prevBest + nums[i])
      = nums[i] + max(0, prevBest)   ← simpler form

Global answer = max of all dp[i]
```

---

## Templates

### Classic Kadane's — Maximum Sum Subarray
```cpp
// O(n) time, O(1) space
int maxSubArray(vector<int>& nums) {
    int curSum = nums[0], maxSum = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        curSum = max(nums[i], curSum + nums[i]);  // extend or restart
        maxSum = max(maxSum, curSum);
    }
    return maxSum;
}
// Equivalent: curSum = nums[i] + max(0, curSum)
```

### Kadane's with Subarray Indices
```cpp
int start = 0, end = 0, tempStart = 0;
int curSum = nums[0], maxSum = nums[0];
for (int i = 1; i < nums.size(); i++) {
    if (nums[i] > curSum + nums[i]) { curSum = nums[i]; tempStart = i; }
    else curSum += nums[i];
    if (curSum > maxSum) { maxSum = curSum; start = tempStart; end = i; }
}
// Subarray is nums[start..end]
```

### Maximum Product Subarray (track both min and max)
```cpp
// Negative × negative = positive → must track minimum too
int maxProduct(vector<int>& nums) {
    int curMax = nums[0], curMin = nums[0], ans = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] < 0) swap(curMax, curMin);  // negative flips max↔min
        curMax = max(nums[i], curMax * nums[i]);
        curMin = min(nums[i], curMin * nums[i]);
        ans = max(ans, curMax);
    }
    return ans;
}
```

### Maximum Sum Circular Subarray
```cpp
// Case 1: max subarray is non-wrapping → standard Kadane's
// Case 2: max subarray wraps around → totalSum - minSubarraySum
// Answer = max(case1, case2)
// Edge case: all negative → answer is max single element (case2 would be 0)
int maxSubarrayCircular(vector<int>& nums) {
    int totalSum = 0;
    int curMax = 0, maxSum = nums[0];
    int curMin = 0, minSum = nums[0];

    for (int x : nums) {
        curMax = max(x, curMax + x);
        maxSum = max(maxSum, curMax);
        curMin = min(x, curMin + x);
        minSum = min(minSum, curMin);
        totalSum += x;
    }
    // If all negative, totalSum - minSum = 0 which is wrong → return maxSum
    return maxSum > 0 ? max(maxSum, totalSum - minSum) : maxSum;
}
```

### Maximum Absolute Sum of Any Subarray
```cpp
// Max absolute sum = max of (maxSubarraySum, |minSubarraySum|)
int maxAbsoluteSum(vector<int>& nums) {
    int curMax = 0, curMin = 0;
    int maxSum = 0, minSum = 0;
    for (int x : nums) {
        curMax = max(x, curMax + x); maxSum = max(maxSum, curMax);
        curMin = min(x, curMin + x); minSum = min(minSum, curMin);
    }
    return max(maxSum, abs(minSum));
}
```

### 2D Maximum Sum Submatrix (Kadane on columns)
```cpp
// Fix top and bottom row, compress to 1D column sums, run Kadane
int maxSumSubmatrix(vector<vector<int>>& matrix) {
    int m = matrix.size(), n = matrix[0].size();
    int ans = INT_MIN;
    for (int top = 0; top < m; top++) {
        vector<int> colSum(n, 0);
        for (int bot = top; bot < m; bot++) {
            for (int c = 0; c < n; c++) colSum[c] += matrix[bot][c];
            // Run Kadane on colSum
            int cur = colSum[0], best = colSum[0];
            for (int c = 1; c < n; c++) {
                cur = max(colSum[c], cur + colSum[c]);
                best = max(best, cur);
            }
            ans = max(ans, best);
        }
    }
    return ans;
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [121](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) | Best Time Buy/Sell Stock | Track min price seen, max profit at each step | Kadane variant: `profit = max(profit, price - minPrice)` |
| [53](https://leetcode.com/problems/maximum-subarray/) | Maximum Subarray | Classic Kadane's | Foundation problem — know cold |
| [1749](https://leetcode.com/problems/maximum-absolute-sum-of-any-subarray/) | Maximum Absolute Sum | Run Kadane for both max and min subarray sum | `max(maxSum, abs(minSum))` |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [152](https://leetcode.com/problems/maximum-product-subarray/) | Maximum Product Subarray | Track both curMax and curMin. Swap on negative | Zero resets both to 1 (or just to nums[i]) |
| [918](https://leetcode.com/problems/maximum-sum-circular-subarray/) | Maximum Sum Circular Subarray | Non-wrap: Kadane. Wrap: total - minSubarray. Edge: all negative | If `maxSum < 0`, all elements negative — return maxSum |
| [978](https://leetcode.com/problems/longest-turbulent-subarray/) | Longest Turbulent Subarray | Kadane variant: extend if alternating signs, else reset | Track whether last diff was pos or neg |
| [1031](https://leetcode.com/problems/maximum-sum-of-two-non-overlapping-subarrays/) | Max Sum of Two Non-Overlapping Subarrays | Precompute max subarray of length L ending at each i and starting at each i | Try all split points |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [363](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/) | Max Sum Rectangle ≤ k | 2D Kadane + sorted set for prefix sums ≤ k | Fix rows, compress to 1D, binary search for valid prefix sum |
| [828](https://leetcode.com/problems/count-unique-characters-of-all-substrings-of-a-given-string/) | Count Unique Chars All Substrings | Kadane-style contribution: each char contributes based on gaps to prev/next same char | `contrib = (i - prev[c]) * (next[c] - i)` |

---

## Kadane's as DP

```
Kadane's is just 1D DP where we can discard previous state:

dp[i] = "max sum subarray ENDING at index i"
dp[i] = max(nums[i], dp[i-1] + nums[i])

Since dp[i] only depends on dp[i-1], we reduce to O(1) space.
Global max = max over all dp[i].
```

## Why Swap Works in Maximum Product

```
At index i with nums[i] < 0:
  newMax = max(nums[i], curMax * nums[i], curMin * nums[i])
         = max(nums[i], curMin * nums[i])   ← curMax * neg = very negative
  newMin = min(nums[i], curMax * nums[i], curMin * nums[i])
         = min(nums[i], curMax * nums[i])   ← curMin * neg = very positive

Shortcut: swap curMax and curMin BEFORE multiplying
  (so curMax * nums[i] uses the old minimum = now the largest product)
```

## Common Mistakes

- Classic Kadane: initializing `curSum = 0` → wrong if all elements are negative (answer should be the least negative, not 0)
- Product variant: not swapping curMax/curMin on negative — or handling zeros (reset both to nums[i])
- Circular variant: returning `max(maxSum, 0)` when all negative — should return `maxSum`

## Follow-up Interviewers Love

- "What if you can pick at most k non-contiguous elements?" → Greedy or DP, different problem
- "Track the actual subarray, not just the sum?" → Track `tempStart` and update `start`/`end` when new max found
- "Maximum product — what if the array contains zeros?" → Zero resets product. Handled naturally since `max(nums[i], 0 * nums[i]) = nums[i]`
- "What's the 2D version of max subarray?" → Fix top/bottom row, compress to 1D, run Kadane → O(m²n)
