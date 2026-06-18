# Pattern 3: Prefix Sum

## Identify This Pattern
- "Sum of subarray [i..j]" — range query
- "Count subarrays with sum = k"
- "Count subarrays with sum divisible by k"
- "Subarray with equal 0s and 1s"
- "Running total" / "product except self"
- Key signal: subarray sum, range sum, equality of two sub-sums

## Core Formula
```
prefix[i] = sum(nums[0..i-1])   (1-indexed, prefix[0] = 0)
sum(i, j) = prefix[j+1] - prefix[i]

Count subarrays with sum = k:
  At each i: how many j < i have prefix[j] = prefix[i] - k?
  → hashmap of prefix sums seen so far
```

---

## Templates

### Build Prefix Sum (1D)
```cpp
vector<int> pre(n+1, 0);
for (int i = 0; i < n; i++) pre[i+1] = pre[i] + nums[i];
// rangeSum(l, r) = pre[r+1] - pre[l]
```

### Count Subarrays with Sum = k (hashmap)
```cpp
unordered_map<int,int> cnt;
cnt[0] = 1;     // ← ALWAYS initialize with 0→1 (empty prefix handles prefix sum == k)
int sum = 0, ans = 0;
for (int x : nums) {
    sum += x;
    ans += cnt[sum - k];   // count prefixes that would make sum == k
    cnt[sum]++;
}
```

### Count Subarrays with Sum Divisible by k
```cpp
unordered_map<int,int> cnt;
cnt[0] = 1;
int sum = 0, ans = 0;
for (int x : nums) {
    sum += x;
    int rem = ((sum % k) + k) % k;   // ((x%k)+k)%k handles negative mod in C++
    ans += cnt[rem];
    cnt[rem]++;
}
```

### Equal 0s and 1s (convert 0→-1, find sum=0 subarray)
```cpp
// Replace 0 with -1. Now "equal 0s and 1s" = "subarray with sum 0"
unordered_map<int,int> firstSeen;
firstSeen[0] = -1;   // sum 0 seen at index -1 (before array)
int sum = 0, ans = 0;
for (int i = 0; i < n; i++) {
    sum += (nums[i] == 0) ? -1 : 1;
    if (firstSeen.count(sum)) ans = max(ans, i - firstSeen[sum]);
    else firstSeen[sum] = i;
}
```

### Product of Array Except Self (prefix + suffix product)
```cpp
// No division. Forward pass = prefix product. Backward pass = suffix product.
vector<int> res(n, 1);
int prefix = 1;
for (int i = 0; i < n; i++) { res[i] = prefix; prefix *= nums[i]; }
int suffix = 1;
for (int i = n-1; i >= 0; i--) { res[i] *= suffix; suffix *= nums[i]; }
```

### 2D Prefix Sum
```cpp
// p[i][j] = sum of rectangle (0,0) to (i-1,j-1)
vector<vector<int>> p(m+1, vector<int>(n+1, 0));
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        p[i][j] = mat[i-1][j-1] + p[i-1][j] + p[i][j-1] - p[i-1][j-1];
// regionSum(r1,c1,r2,c2) = p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1480](https://leetcode.com/problems/running-sum-of-1d-array/) | Running Sum of 1D Array | `res[i] = res[i-1] + nums[i]` | Direct prefix sum |
| [724](https://leetcode.com/problems/find-pivot-index/) | Find Pivot Index | `leftSum == totalSum - leftSum - nums[i]` | Running left sum, check if equals right |
| [303](https://leetcode.com/problems/range-sum-query-immutable/) | Range Sum Query | Precompute prefix, O(1) per query | Classic prefix sum setup |
| [1588](https://leetcode.com/problems/sum-of-all-odd-length-subarrays/) | Sum of All Odd Length Subarrays | Each element contributes based on how many odd-length subarrays contain it | O(n) math formula or O(n) prefix |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [238](https://leetcode.com/problems/product-of-array-except-self/) | Product of Array Except Self | Prefix product + suffix product. No division | Two passes, O(1) extra space |
| [560](https://leetcode.com/problems/subarray-sum-equals-k/) | Subarray Sum Equals K | Prefix sum + hashmap: count occurrences of `prefix[i] - k` | Initialize `cnt[0]=1` before loop |
| [974](https://leetcode.com/problems/subarray-sums-divisible-by-k/) | Subarray Sums Divisible by K | Same as #560 but use modulo. Handle negative mod | `((sum%k)+k)%k` normalizes remainder |
| [525](https://leetcode.com/problems/contiguous-array/) | Contiguous Array (equal 0s and 1s) | Replace 0→-1. Find longest subarray with sum=0 | Use `firstSeen[sum]` with initial `{0: -1}` |
| [523](https://leetcode.com/problems/continuous-subarray-sum/) | Continuous Subarray Sum (divisible by k) | Store remainder → first index. Subarray of length ≥ 2 | `if (i - firstSeen[rem] >= 2) return true` |
| [304](https://leetcode.com/problems/range-sum-query-2d-immutable/) | Range Sum Query 2D | 2D prefix sum. Inclusion-exclusion formula | `p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1]` |
| [1314](https://leetcode.com/problems/matrix-block-sum/) | Matrix Block Sum | 2D prefix sum, query every (i,j) with clamped bounds | `max(0, i-k)` and `min(m-1, i+k)` for clamping |
| [1371](https://leetcode.com/problems/find-the-longest-substring-containing-vowels-in-even-counts/) | Longest Substring with Vowels in Even Counts | XOR bitmask as "prefix sum" for parity of each vowel. Find first occurrence of same mask | `firstSeen[mask] = index`. Longest = `i - firstSeen[mask]` |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [327](https://leetcode.com/problems/count-of-range-sum/) | Count of Range Sum | Count prefix sums in range [lower, upper]. Use merge sort or BIT | Hard — merge sort on prefix array |
| [363](https://leetcode.com/problems/max-sum-of-rectangle-no-larger-than-k/) | Max Sum Rectangle No Larger Than K | Fix top/bottom rows, compress to 1D, use sorted set for prefix sums | For each column pair: 1D problem with BST |

---

## The `cnt[0] = 1` Rule

```
Why initialize cnt[0] = 1 before the loop?

If prefix[i] == k exactly, then the subarray nums[0..i] has sum k.
But there's no previous prefix[j] with prefix[j] = 0 in the map yet.
Initializing cnt[0] = 1 represents the "empty prefix" before index 0.

Without it: miss subarrays starting from index 0.
```

## Common Mistakes

- Forgetting `cnt[0] = 1` → misses subarrays starting at index 0
- Negative mod: `sum % k` in C++ can be negative. Always `((sum % k) + k) % k`
- 2D prefix: off-by-one in indices — use 1-indexed prefix array to simplify boundary handling
- #525: storing `firstSeen[sum] = i` even when sum already exists — use `if (!firstSeen.count(sum))` to store FIRST occurrence only

## Follow-up Interviewers Love

- "#560 — what if all numbers are positive?" → Can also use sliding window in O(n)
- "#238 — what if zeros are present?" → Same solution still works (no division used)
- "#303 — what if array is mutable?" → Segment tree or BIT (Fenwick tree)
- "What is the time and space complexity of prefix sum?" → O(n) build, O(1) query, O(n) space
