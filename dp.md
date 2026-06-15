# Dynamic Programming Patterns — LeetCode C++ Reference

## What is DP and When to Use It

DP = recursion + memoization, or equivalently, filling a table bottom-up.

**Use DP when ALL three are true:**
1. **Optimal substructure** — optimal solution built from optimal sub-solutions
2. **Overlapping sub-problems** — same sub-problem solved more than once
3. **No "undo"** — choices made earlier cannot be reversed (unlike backtracking)

**Clues in the problem statement:**
```
"maximum / minimum"          → optimization DP
"number of ways / count"     → counting DP
"is it possible / can you"   → existence DP (return bool)
"longest / shortest"         → sequence DP
"all subsets / partition"     → knapsack / partition DP
"string matching / edit"      → string DP
"buy and sell / cooldown"     → state machine DP
"pick elements from ends"     → interval DP
```

---

## Two Ways to Write DP

```cpp
// TOP-DOWN (memoization) — start from answer, recurse down
// Easier to think about, same complexity as bottom-up
unordered_map<int, int> memo;
int solve(int n) {
    if (n <= 1) return n;                      // base case
    if (memo.count(n)) return memo[n];         // cache hit
    return memo[n] = solve(n-1) + solve(n-2); // store result
}

// BOTTOM-UP (tabulation) — start from base cases, build up
// No recursion overhead, better cache performance
vector<int> dp(n+1);
dp[0] = 0; dp[1] = 1;                         // base cases
for (int i = 2; i <= n; i++)
    dp[i] = dp[i-1] + dp[i-2];               // fill table
```

**Which to use:**
```
Top-down:    easier to write, only computes needed states
Bottom-up:   faster in practice, easier to optimize space
Interviews:  top-down is fine; bottom-up shows DP mastery
```

---

## How to Think Through Any DP Problem

```
1. Define the STATE — what parameters fully describe a sub-problem?
   dp[i], dp[i][j], dp[i][j][k] — keep it minimal

2. Define what dp[i] MEANS — "dp[i] = minimum cost to reach index i"
   Write this in English before writing any code

3. Write the RECURRENCE — how does dp[i] depend on smaller states?
   "To reach i, I came from i-1 or i-2, so dp[i] = min(dp[i-1], dp[i-2]) + cost[i]"

4. Identify BASE CASES — smallest inputs that don't recurse

5. Identify ANSWER — is it dp[n]? max of all dp[i]? dp[m][n]?

6. OPTIMIZE space if needed — rolling array, two variables
```

---

## Pattern 1: Linear DP (1D)

**When to think about it:**
- Process array from left to right
- Each state depends on a few previous states
- "Climbing stairs", "house robber", "jump game"

**Core insight:** dp[i] = answer for the first i elements.
Recurrence looks backward: dp[i] = f(dp[i-1], dp[i-2], ...)

### Climbing Stairs / Fibonacci
```cpp
int climbStairs(int n) {
    if (n <= 2) return n;
    int a = 1, b = 2;
    for (int i = 3; i <= n; i++) {
        int c = a + b;
        a = b; b = c;
    }
    return b;
}
```

### House Robber (can't rob adjacent)
```cpp
int rob(vector<int>& nums) {
    int prev2 = 0, prev1 = 0;
    for (int x : nums) {
        int cur = max(prev1, prev2 + x);  // skip or rob current
        prev2 = prev1;
        prev1 = cur;
    }
    return prev1;
}

// House Robber II (circular — solve twice, exclude first or last)
int robII(vector<int>& nums) {
    int n = nums.size();
    if (n == 1) return nums[0];
    auto robRange = [&](int l, int r) {
        int prev2 = 0, prev1 = 0;
        for (int i = l; i <= r; i++) {
            int cur = max(prev1, prev2 + nums[i]);
            prev2 = prev1; prev1 = cur;
        }
        return prev1;
    };
    return max(robRange(0, n-2), robRange(1, n-1));
}
```

### Jump Game (can you reach the end?)
```cpp
// Greedy — but DP version helps understand the pattern
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i < nums.size(); i++) {
        if (i > maxReach) return false;       // can't even get here
        maxReach = max(maxReach, i + nums[i]);
    }
    return true;
}

// Jump Game II — minimum jumps (greedy DP)
int jump(vector<int>& nums) {
    int jumps = 0, curEnd = 0, farthest = 0;
    for (int i = 0; i < nums.size() - 1; i++) {
        farthest = max(farthest, i + nums[i]);
        if (i == curEnd) { jumps++; curEnd = farthest; }
    }
    return jumps;
}
```

### Min Cost Climbing Stairs
```cpp
int minCostClimbingStairs(vector<int>& cost) {
    int n = cost.size();
    for (int i = 2; i < n; i++)
        cost[i] += min(cost[i-1], cost[i-2]);
    return min(cost[n-1], cost[n-2]);
}
```

**Problems:** #70 Climbing Stairs, #198 House Robber, #213 House Robber II, #55 Jump Game, #45 Jump Game II, #746 Min Cost Climbing Stairs, #152 Maximum Product Subarray

---

## Pattern 2: Knapsack DP

**The most important DP pattern. Three variations:**

```
0/1 Knapsack:      each item used AT MOST once
Unbounded:         each item used ANY number of times
Bounded/Grouping:  each item used within a limit
```

**Core insight:** Two decisions at each item — include it or skip it.
State: dp[i][w] = best value using first i items with weight limit w.

### 0/1 Knapsack Template
```cpp
// dp[i][w] = max value using first i items, weight capacity w
int knapsack01(vector<int>& weights, vector<int>& values, int W) {
    int n = weights.size();
    vector<vector<int>> dp(n+1, vector<int>(W+1, 0));

    for (int i = 1; i <= n; i++) {
        for (int w = 0; w <= W; w++) {
            dp[i][w] = dp[i-1][w];                       // skip item i
            if (w >= weights[i-1])
                dp[i][w] = max(dp[i][w],
                               dp[i-1][w - weights[i-1]] + values[i-1]); // take item i
        }
    }
    return dp[n][W];
}

// Space-optimized: 1D array (traverse RIGHT to LEFT to avoid reuse)
int knapsack01_1D(vector<int>& w, vector<int>& v, int W) {
    vector<int> dp(W+1, 0);
    for (int i = 0; i < w.size(); i++)
        for (int cap = W; cap >= w[i]; cap--)           // RIGHT TO LEFT — no reuse
            dp[cap] = max(dp[cap], dp[cap - w[i]] + v[i]);
    return dp[W];
}
```

### Unbounded Knapsack Template (items reusable)
```cpp
// Space-optimized: traverse LEFT to RIGHT to allow reuse
int knapsackUnbounded(vector<int>& w, vector<int>& v, int W) {
    vector<int> dp(W+1, 0);
    for (int i = 0; i < w.size(); i++)
        for (int cap = w[i]; cap <= W; cap++)           // LEFT TO RIGHT — allows reuse
            dp[cap] = max(dp[cap], dp[cap - w[i]] + v[i]);
    return dp[W];
}
```

**The KEY difference:**
```
0/1 Knapsack:   inner loop RIGHT TO LEFT  → each item used at most once
Unbounded:      inner loop LEFT TO RIGHT  → each item used any times
```

### Partition Equal Subset Sum (0/1 Knapsack application)
```cpp
// Can we partition array into two equal-sum subsets?
bool canPartition(vector<int>& nums) {
    int total = accumulate(nums.begin(), nums.end(), 0);
    if (total % 2 != 0) return false;
    int target = total / 2;

    vector<bool> dp(target+1, false);
    dp[0] = true;

    for (int num : nums)
        for (int j = target; j >= num; j--)   // right to left = 0/1
            dp[j] = dp[j] || dp[j - num];

    return dp[target];
}
```

### Coin Change (Unbounded Knapsack application)
```cpp
// Minimum coins to make amount (unbounded — coins reusable)
int coinChange(vector<int>& coins, int amount) {
    vector<int> dp(amount+1, INT_MAX);
    dp[0] = 0;

    for (int i = 1; i <= amount; i++)
        for (int coin : coins)
            if (coin <= i && dp[i - coin] != INT_MAX)
                dp[i] = min(dp[i], dp[i - coin] + 1);

    return dp[amount] == INT_MAX ? -1 : dp[amount];
}

// Number of ways to make amount (Coin Change II)
int change(int amount, vector<int>& coins) {
    vector<int> dp(amount+1, 0);
    dp[0] = 1;

    for (int coin : coins)                     // outer = items
        for (int j = coin; j <= amount; j++)   // inner = left to right (unbounded)
            dp[j] += dp[j - coin];

    return dp[amount];
}
```

**Why outer=items for Coin Change II but not always:**
- Outer=items → each item processed once → no duplicate combinations
- Outer=amounts → counts permutations (different orderings counted separately)

**Problems:** #416 Partition Equal Subset Sum, #322 Coin Change, #518 Coin Change II, #474 Ones and Zeroes, #494 Target Sum, #1049 Last Stone Weight II

---

## Pattern 3: LCS — Longest Common Subsequence Family

**When to think about it:**
- Two strings/arrays, need to find common structure
- "Edit distance", "longest common subsequence/substring", "shortest common supersequence"
- State is always dp[i][j] = answer for first i chars of s1 and first j chars of s2

**Core insight:**
```
If s1[i] == s2[j]: they match → dp[i][j] = dp[i-1][j-1] + 1 (or similar)
If s1[i] != s2[j]: take best of skipping either → dp[i][j] = max/min(dp[i-1][j], dp[i][j-1])
```

### Longest Common Subsequence
```cpp
int longestCommonSubsequence(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;         // match
            else
                dp[i][j] = max(dp[i-1][j], dp[i][j-1]); // skip one

    return dp[m][n];
}
```

### Edit Distance (Levenshtein)
```cpp
// Minimum operations (insert, delete, replace) to convert s1 to s2
int minDistance(string s1, string s2) {
    int m = s1.size(), n = s2.size();
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));

    for (int i = 0; i <= m; i++) dp[i][0] = i;  // delete all of s1
    for (int j = 0; j <= n; j++) dp[0][j] = j;  // insert all of s2

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1];                        // no op needed
            else
                dp[i][j] = 1 + min({dp[i-1][j],               // delete from s1
                                     dp[i][j-1],               // insert into s1
                                     dp[i-1][j-1]});           // replace

    return dp[m][n];
}
```

### Longest Common Substring (contiguous)
```cpp
int longestCommonSubstring(string s1, string s2) {
    int m = s1.size(), n = s2.size(), res = 0;
    vector<vector<int>> dp(m+1, vector<int>(n+1, 0));

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++) {
            if (s1[i-1] == s2[j-1])
                dp[i][j] = dp[i-1][j-1] + 1;   // must be contiguous
            else
                dp[i][j] = 0;                    // reset on mismatch
            res = max(res, dp[i][j]);
        }
    return res;
}
```

### Shortest Common Supersequence
```cpp
// Length of shortest string containing both s1 and s2 as subsequences
int shortestCommonSupersequence(string s1, string s2) {
    int lcs = longestCommonSubsequence(s1, s2);
    return s1.size() + s2.size() - lcs;
}
```

**LCS family cheat sheet:**
```
LCS length:                  dp[i][j] = dp[i-1][j-1]+1 if match, else max(skip either)
Edit distance:               dp[i][j] = dp[i-1][j-1]   if match, else 1+min(3 ops)
Longest Common Substring:    dp[i][j] = dp[i-1][j-1]+1 if match, else 0 (reset!)
Shortest Common Supersequence: s1.len + s2.len - LCS
Longest Palindromic Subseq:  LCS(s, reverse(s))
Min deletions to make palindrome: n - LPS(s)
```

**Problems:** #1143 LCS, #72 Edit Distance, #583 Delete Operation for Two Strings, #1092 Shortest Common Supersequence, #516 Longest Palindromic Subsequence, #44 Wildcard Matching, #10 Regular Expression Matching

---

## Pattern 4: LIS — Longest Increasing Subsequence Family

**When to think about it:**
- "Longest increasing subsequence"
- "Number of increasing subsequences"
- "Box stacking", "envelope nesting", "Russian dolls"
- Single array, find longest chain satisfying a condition

**Core insight:** dp[i] = length of LIS ending at index i.
For each i, look back at all j < i where arr[j] < arr[i].

### LIS — O(n²) DP
```cpp
int lengthOfLIS(vector<int>& nums) {
    int n = nums.size();
    vector<int> dp(n, 1);   // LIS ending at each index, min = 1

    for (int i = 1; i < n; i++)
        for (int j = 0; j < i; j++)
            if (nums[j] < nums[i])
                dp[i] = max(dp[i], dp[j] + 1);

    return *max_element(dp.begin(), dp.end());
}
```

### LIS — O(n log n) with Binary Search (patience sorting)
```cpp
int lengthOfLIS_nlogn(vector<int>& nums) {
    vector<int> tails;   // tails[i] = smallest tail of IS with length i+1

    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);  // extend LIS
        else *it = x;                                // replace to keep tails small
    }
    return tails.size();
}
// tails is NOT the actual LIS — it's a virtual array for length computation
```

### Russian Doll Envelopes (2D LIS)
```cpp
// Sort by width asc, then height DESC (key trick for 2D LIS)
// The desc sort on height prevents counting two envelopes with same width
int maxEnvelopes(vector<vector<int>>& env) {
    sort(env.begin(), env.end(), [](auto& a, auto& b) {
        return a[0] != b[0] ? a[0] < b[0] : a[1] > b[1];  // height desc for same width
    });
    vector<int> heights;
    for (auto& e : env) {
        auto it = lower_bound(heights.begin(), heights.end(), e[1]);
        if (it == heights.end()) heights.push_back(e[1]);
        else *it = e[1];
    }
    return heights.size();
}
```

**Problems:** #300 LIS, #354 Russian Doll Envelopes, #673 Number of LIS, #368 Largest Divisible Subset, #1964 Longest Obstacle Course

---

## Pattern 5: Grid / 2D DP

**When to think about it:**
- Movement on a 2D grid (usually only right and down)
- "Unique paths", "min path sum", "triangle"
- "Maximal square of 1s"

**Core insight:** dp[i][j] = answer at cell (i,j), computed from dp[i-1][j] and dp[i][j-1].

### Unique Paths
```cpp
int uniquePaths(int m, int n) {
    vector<vector<int>> dp(m, vector<int>(n, 1));  // base: edges = 1 path

    for (int i = 1; i < m; i++)
        for (int j = 1; j < n; j++)
            dp[i][j] = dp[i-1][j] + dp[i][j-1];

    return dp[m-1][n-1];
}
```

### Minimum Path Sum
```cpp
int minPathSum(vector<vector<int>>& grid) {
    int m = grid.size(), n = grid[0].size();

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (i == 0 && j == 0) continue;
            else if (i == 0) grid[i][j] += grid[i][j-1];
            else if (j == 0) grid[i][j] += grid[i-1][j];
            else grid[i][j] += min(grid[i-1][j], grid[i][j-1]);
        }
    return grid[m-1][n-1];
}
```

### Maximal Square of 1s
```cpp
int maximalSquare(vector<vector<char>>& matrix) {
    int m = matrix.size(), n = matrix[0].size(), maxSide = 0;
    vector<vector<int>> dp(m, vector<int>(n, 0));

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++) {
            if (matrix[i][j] == '0') continue;
            if (i == 0 || j == 0) dp[i][j] = 1;
            else
                // Minimum of three neighbors + 1
                dp[i][j] = min({dp[i-1][j], dp[i][j-1], dp[i-1][j-1]}) + 1;
            maxSide = max(maxSide, dp[i][j]);
        }
    return maxSide * maxSide;
}
```

### Triangle (min path from top to bottom)
```cpp
int minimumTotal(vector<vector<int>>& triangle) {
    int n = triangle.size();
    vector<int> dp = triangle[n-1];     // start from bottom row

    for (int i = n-2; i >= 0; i--)
        for (int j = 0; j <= i; j++)
            dp[j] = triangle[i][j] + min(dp[j], dp[j+1]);  // pick better child

    return dp[0];
}
```

**Problems:** #62 Unique Paths, #63 Unique Paths II, #64 Minimum Path Sum, #221 Maximal Square, #120 Triangle, #931 Minimum Falling Path Sum

---

## Pattern 6: Interval DP

**When to think about it:**
- "Burst balloons", "matrix chain multiplication", "stone merging"
- "Minimum/maximum cost to process a range"
- You're picking which element to process LAST within a range

**Core insight:** dp[i][j] = answer for the subarray/subproblem on range [i, j].
Try every possible "last operation" split point k in [i, j].
**Must iterate by length**, not by i or j directly.

```
for length 2 to n:
    for every starting index i:
        j = i + length - 1
        for every split k in [i, j]:
            dp[i][j] = optimal(dp[i][k] + dp[k][j] + cost(i,k,j))
```

### Burst Balloons (classic interval DP)
```cpp
// dp[i][j] = max coins from bursting all balloons between i and j (exclusive)
// Trick: think of k as the LAST balloon to burst, not the first
int maxCoins(vector<int>& nums) {
    int n = nums.size();
    nums.insert(nums.begin(), 1);
    nums.push_back(1);       // add virtual boundary balloons of value 1

    int N = nums.size();
    vector<vector<int>> dp(N, vector<int>(N, 0));

    for (int len = 2; len < N; len++) {           // interval length
        for (int i = 0; i < N - len; i++) {
            int j = i + len;
            for (int k = i+1; k < j; k++) {       // k = last balloon to burst
                dp[i][j] = max(dp[i][j],
                    dp[i][k] + nums[i]*nums[k]*nums[j] + dp[k][j]);
            }
        }
    }
    return dp[0][N-1];
}
```

### Matrix Chain Multiplication
```cpp
// Minimum multiplications to compute chain of matrices
int matrixChain(vector<int>& dims) {
    int n = dims.size() - 1;   // n matrices
    vector<vector<int>> dp(n, vector<int>(n, 0));

    for (int len = 2; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = INT_MAX;
            for (int k = i; k < j; k++)
                dp[i][j] = min(dp[i][j],
                    dp[i][k] + dp[k+1][j] + dims[i]*dims[k+1]*dims[j+1]);
        }
    }
    return dp[0][n-1];
}
```

### Strange Printer / Remove Boxes
```cpp
// Minimum turns to print a string (strange printer)
int strangePrinter(string s) {
    int n = s.size();
    vector<vector<int>> dp(n, vector<int>(n, 0));

    for (int len = 1; len <= n; len++) {
        for (int i = 0; i <= n - len; i++) {
            int j = i + len - 1;
            dp[i][j] = dp[i][j-1] + 1;    // worst: print s[j] separately
            for (int k = i; k < j; k++)
                if (s[k] == s[j])
                    dp[i][j] = min(dp[i][j], dp[i][k] + (k+1 <= j-1 ? dp[k+1][j-1] : 0));
        }
    }
    return dp[0][n-1];
}
```

**Problems:** #312 Burst Balloons, #1039 Min Score Triangulation, #664 Strange Printer, #516 Longest Palindromic Subsequence, #375 Guess Number, #546 Remove Boxes

---

## Pattern 7: State Machine DP (Stock Problems)

**When to think about it:**
- Problem has discrete states you transition between
- "Buy and sell stocks with cooldown / fee / k transactions"
- "At each step, you are in one of N states"

**Core insight:** Define states explicitly and write transitions between them.
For stocks: states are (holding / not holding), transitions are buy/sell/hold.

```
States:
  held  = max profit when currently HOLDING a stock
  free  = max profit when NOT holding a stock

Transitions:
  held  = max(held,        free - price)   // keep holding OR buy today
  free  = max(free,        held + price)   // keep free OR sell today
```

### Best Time to Buy and Sell Stock (one transaction)
```cpp
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, maxProfit = 0;
    for (int p : prices) {
        minPrice = min(minPrice, p);
        maxProfit = max(maxProfit, p - minPrice);
    }
    return maxProfit;
}
```

### Unlimited Transactions
```cpp
int maxProfitII(vector<int>& prices) {
    int profit = 0;
    for (int i = 1; i < prices.size(); i++)
        profit += max(0, prices[i] - prices[i-1]);  // take every upward move
    return profit;
}
```

### With Cooldown (1 day after sell)
```cpp
int maxProfitCooldown(vector<int>& prices) {
    int held = INT_MIN, free = 0, cooldown = 0;

    for (int p : prices) {
        int prevFree = free;
        free     = max(free, held + p);       // sell today or stay free
        held     = max(held, cooldown - p);   // buy today (from cooldown) or keep holding
        cooldown = prevFree;                  // yesterday's free becomes today's cooldown
    }
    return free;
}
```

### With Transaction Fee
```cpp
int maxProfitFee(vector<int>& prices, int fee) {
    int held = INT_MIN, free = 0;
    for (int p : prices) {
        held = max(held, free - p);          // buy
        free = max(free, held + p - fee);    // sell (pay fee)
    }
    return free;
}
```

### At Most k Transactions
```cpp
int maxProfitK(int k, vector<int>& prices) {
    int n = prices.size();
    if (k >= n/2) {   // unlimited transactions
        int profit = 0;
        for (int i = 1; i < n; i++) profit += max(0, prices[i]-prices[i-1]);
        return profit;
    }

    // buy[j] = max profit after j+1 buys, sell[j] = max profit after j+1 sells
    vector<int> buy(k, INT_MIN), sell(k, 0);
    for (int p : prices) {
        for (int j = k-1; j >= 0; j--) {
            sell[j] = max(sell[j], buy[j] + p);
            buy[j]  = max(buy[j],  (j > 0 ? sell[j-1] : 0) - p);
        }
    }
    return sell[k-1];
}
```

**State machine general template:**
```cpp
// Define all states, update in one pass
for each day:
    for each state:
        state = max(stay in state, transition from other state)
```

**Problems:** #121 Best Time I, #122 Best Time II, #123 Best Time III (k=2), #188 Best Time IV (k=k), #309 With Cooldown, #714 With Fee

---

## Pattern 8: String DP (Palindrome)

**When to think about it:**
- "Longest palindromic substring/subsequence"
- "Minimum cuts for palindrome partitioning"
- "Count palindromic substrings"

**Core insight:** dp[i][j] = is substring s[i..j] a palindrome?
Base: dp[i][i] = true (length 1), dp[i][i+1] = (s[i]==s[i+1]).
Transition: dp[i][j] = (s[i]==s[j]) && dp[i+1][j-1].

### Count Palindromic Substrings
```cpp
int countSubstrings(string s) {
    int n = s.size(), count = 0;

    // Expand around center approach (O(n²) time, O(1) space)
    auto expand = [&](int l, int r) {
        while (l >= 0 && r < n && s[l] == s[r]) { l--; r++; count++; }
    };
    for (int i = 0; i < n; i++) { expand(i, i); expand(i, i+1); }
    return count;
}
```

### Longest Palindromic Substring
```cpp
string longestPalindrome(string s) {
    int n = s.size(), start = 0, maxLen = 1;

    auto expand = [&](int l, int r) {
        while (l >= 0 && r < n && s[l] == s[r]) { l--; r++; }
        if (r - l - 1 > maxLen) { maxLen = r-l-1; start = l+1; }
    };
    for (int i = 0; i < n; i++) { expand(i, i); expand(i, i+1); }
    return s.substr(start, maxLen);
}
```

### Palindrome Partitioning II (minimum cuts)
```cpp
int minCut(string s) {
    int n = s.size();
    vector<vector<bool>> isPalin(n, vector<bool>(n, false));
    vector<int> dp(n);  // dp[i] = min cuts for s[0..i]

    for (int i = 0; i < n; i++) {
        dp[i] = i;   // worst case: cut every character
        for (int j = 0; j <= i; j++) {
            if (s[j] == s[i] && (j+1 > i-1 || isPalin[j+1][i-1])) {
                isPalin[j][i] = true;
                dp[i] = (j == 0) ? 0 : min(dp[i], dp[j-1] + 1);
            }
        }
    }
    return dp[n-1];
}
```

**Problems:** #647 Palindromic Substrings, #5 Longest Palindromic Substring, #516 Longest Palindromic Subsequence, #132 Palindrome Partitioning II, #1312 Min Insertions for Palindrome

---

## Pattern 9: Digit DP

**When to think about it:**
- "Count numbers from 1 to N satisfying some property"
- "Count integers with no two adjacent same digits"
- "Count numbers with digit sum divisible by k"
- The constraint involves the digits of the number itself

**Core insight:** Process the number digit by digit.
State: (position, tight constraint, any other constraint like sum/count).
`tight` = are we still bounded by N's digits? If tight=false, remaining digits can be 0-9 freely.

```cpp
// Count numbers from 1 to n with no repeated adjacent digits
int countSpecialNumbers(int n) {
    string s = to_string(n);
    int m = s.size();

    // dp(pos, mask, tight, started)
    // mask = bitmask of digits used so far
    // tight = still bounded by n's prefix
    // started = have we placed a non-zero digit yet
    map<tuple<int,int,bool,bool>, int> memo;

    function<int(int,int,bool,bool)> dp = [&](int pos, int mask, bool tight, bool started) -> int {
        if (pos == m) return started ? 1 : 0;

        auto key = make_tuple(pos, mask, tight, started);
        if (memo.count(key)) return memo[key];

        int limit = tight ? (s[pos]-'0') : 9;
        int res = 0;

        for (int d = 0; d <= limit; d++) {
            if (started && (mask >> d & 1)) continue;   // digit already used
            bool newTight = tight && (d == limit);
            bool newStarted = started || (d != 0);
            int newMask = newStarted ? (mask | (1 << d)) : mask;
            res += dp(pos+1, newMask, newTight, newStarted);
        }
        return memo[key] = res;
    };

    return dp(0, 0, true, false);
}
```

**General Digit DP Template:**
```
State:     (position, tight, [problem-specific state])
Base case: reached end of digits → check if valid
Transition: try each digit 0 to (tight ? current_digit : 9)
            update tight: newTight = tight && (d == current_digit)
```

**Problems:** #233 Number of Digit One, #357 Count Numbers with Unique Digits, #902 Numbers At Most N Given Digit Set, #2376 Count Special Integers

---

## Pattern 10: Bitmask DP

**When to think about it:**
- Small set of items (n ≤ 20) and you need to track which are used
- "Assign tasks to workers", "visit all cities exactly once"
- State includes a subset — represent it as a bitmask

**Core insight:** Use an integer as a bitmask where bit i = 1 means item i is included.
dp[mask] = answer when exactly the items in `mask` have been used.

### Minimum Cost to Visit Every City (TSP-like)
```cpp
// dp[mask][i] = min cost to have visited cities in mask, currently at city i
int tsp(vector<vector<int>>& dist, int n) {
    vector<vector<int>> dp(1 << n, vector<int>(n, INT_MAX));
    dp[1][0] = 0;   // start at city 0, only city 0 visited

    for (int mask = 1; mask < (1 << n); mask++) {
        for (int u = 0; u < n; u++) {
            if (dp[mask][u] == INT_MAX) continue;
            if (!(mask >> u & 1)) continue;        // u not in mask

            for (int v = 0; v < n; v++) {
                if (mask >> v & 1) continue;       // v already visited
                int newMask = mask | (1 << v);
                dp[newMask][v] = min(dp[newMask][v], dp[mask][u] + dist[u][v]);
            }
        }
    }

    int ans = INT_MAX;
    int fullMask = (1 << n) - 1;
    for (int u = 0; u < n; u++)
        if (dp[fullMask][u] != INT_MAX)
            ans = min(ans, dp[fullMask][u] + dist[u][0]);
    return ans;
}
```

### Assign K Workers to N Jobs
```cpp
// dp[mask] = min cost to complete jobs in mask
// Assign jobs one by one — worker = popcount(mask) - 1
int assignmentProblem(vector<vector<int>>& cost, int n) {
    vector<int> dp(1 << n, INT_MAX);
    dp[0] = 0;

    for (int mask = 0; mask < (1 << n); mask++) {
        if (dp[mask] == INT_MAX) continue;
        int worker = __builtin_popcount(mask);   // which worker is next
        if (worker == n) continue;

        for (int job = 0; job < n; job++) {
            if (mask >> job & 1) continue;       // job already assigned
            dp[mask | (1 << job)] = min(dp[mask | (1 << job)],
                                        dp[mask] + cost[worker][job]);
        }
    }
    return dp[(1 << n) - 1];
}
```

**Bitmask tricks:**
```cpp
mask | (1 << i)      // set bit i
mask & ~(1 << i)     // clear bit i
mask >> i & 1        // check bit i
__builtin_popcount(mask)  // count set bits
mask & (mask - 1)    // remove lowest set bit
mask & (-mask)       // isolate lowest set bit
```

**Problems:** #847 Shortest Path Visiting All Nodes, #1125 Smallest Sufficient Team, #465 Optimal Account Balancing, #1986 Minimum Number of Work Sessions, #526 Beautiful Arrangement

---

## Pattern 11: DP on Trees

**When to think about it:**
- "Rob houses on a tree" (can't take parent + child)
- "Max independent set on tree"
- "Assign weights to nodes with constraints"
- Already covered in trees.md — key pattern is return pair<int,int> (take, skip)

```cpp
// Generic Tree DP returning (take_root, skip_root)
pair<int,int> treeDP(TreeNode* root) {
    if (!root) return {0, 0};

    auto [lt, ls] = treeDP(root->left);
    auto [rt, rs] = treeDP(root->right);

    int take = root->val + ls + rs;          // take root, must skip children
    int skip = max(lt, ls) + max(rt, rs);    // skip root, take best of children

    return {take, skip};
}
```

**Problems:** #337 House Robber III, #968 Binary Tree Cameras, #1372 Longest Zigzag Path, #2246 Longest Path with Different Adjacent Characters

---

## Pattern 12: DP + Binary Search (Advanced)

**When to think about it:**
- LIS in O(n log n)
- "Allocate minimum pages" — binary search on answer, DP to verify
- Optimization DP where transitions can be sped up with binary search

```cpp
// LIS in O(n log n) — already in sorting.md, repeated here for completeness
int lis_nlogn(vector<int>& nums) {
    vector<int> tails;
    for (int x : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), x);
        if (it == tails.end()) tails.push_back(x);
        else *it = x;
    }
    return tails.size();
}

// Allocate Books — binary search on answer + greedy check
bool canAllocate(vector<int>& pages, int students, long long maxPages) {
    int count = 1; long long cur = 0;
    for (int p : pages) {
        if (p > maxPages) return false;
        if (cur + p > maxPages) { count++; cur = p; }
        else cur += p;
    }
    return count <= students;
}
int allocateBooks(vector<int>& pages, int students) {
    long long lo = *max_element(pages.begin(), pages.end());
    long long hi = accumulate(pages.begin(), pages.end(), 0LL);
    while (lo < hi) {
        long long mid = lo + (hi - lo) / 2;
        if (canAllocate(pages, students, mid)) hi = mid;
        else lo = mid + 1;
    }
    return lo;
}
```

**Problems:** #300 LIS (n log n), #354 Russian Dolls, #1235 Max Profit Job Scheduling, #410 Split Array Largest Sum

---

## Space Optimization Techniques

### Rolling Array (2D → 1D)
```cpp
// When dp[i][j] only uses dp[i-1][j] and dp[i][j-1]
// 2D table → 1D array, updated in place
vector<int> dp(n+1, 0);
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[j] = dp[j] + dp[j-1];   // dp[j] was dp[i-1][j], dp[j-1] is dp[i][j-1]
```

### Two Variables (1D → O(1))
```cpp
// When dp[i] only uses dp[i-1] and dp[i-2]
int prev2 = 0, prev1 = 1;
for (int i = 2; i <= n; i++) {
    int cur = prev1 + prev2;
    prev2 = prev1;
    prev1 = cur;
}
```

### When to optimize:
```
2D dp[m][n] where m,n ≤ 1000 → 4MB, usually fine
2D dp[n][n] where n = 10000  → 400MB, must optimize to 1D
Bitmask dp[1<<20][n]          → too large, check constraints
```

---

## The 8 Most Important Templates to Memorize

```cpp
// 1. 0/1 KNAPSACK
vector<int> dp(W+1, 0);
for (auto& item : items)
    for (int w = W; w >= item.weight; w--)   // RIGHT TO LEFT
        dp[w] = max(dp[w], dp[w - item.weight] + item.val);

// 2. UNBOUNDED KNAPSACK (coin change)
vector<int> dp(amount+1, INT_MAX); dp[0] = 0;
for (int coin : coins)
    for (int j = coin; j <= amount; j++)      // LEFT TO RIGHT
        if (dp[j-coin] != INT_MAX)
            dp[j] = min(dp[j], dp[j-coin] + 1);

// 3. LCS
vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
for (int i = 1; i <= m; i++)
    for (int j = 1; j <= n; j++)
        dp[i][j] = s1[i-1]==s2[j-1] ? dp[i-1][j-1]+1 : max(dp[i-1][j], dp[i][j-1]);

// 4. LIS O(n log n)
vector<int> tails;
for (int x : nums) {
    auto it = lower_bound(tails.begin(), tails.end(), x);
    if (it == tails.end()) tails.push_back(x); else *it = x;
}

// 5. INTERVAL DP
vector<vector<int>> dp(n, vector<int>(n, 0));
for (int len = 2; len <= n; len++)
    for (int i = 0; i <= n-len; i++) {
        int j = i+len-1;
        for (int k = i; k < j; k++)
            dp[i][j] = max(dp[i][j], dp[i][k] + dp[k+1][j] + cost(i,k,j));
    }

// 6. STATE MACHINE (stock buy/sell)
int held = INT_MIN, free = 0;
for (int p : prices) {
    held = max(held, free - p);    // buy
    free = max(free, held + p);    // sell
}

// 7. PALINDROME EXPAND
int count = 0;
auto expand = [&](int l, int r) {
    while (l>=0 && r<n && s[l]==s[r]) { l--; r++; count++; }
};
for (int i = 0; i < n; i++) { expand(i,i); expand(i,i+1); }

// 8. BITMASK DP
vector<int> dp(1<<n, INT_MAX); dp[0] = 0;
for (int mask = 0; mask < (1<<n); mask++) {
    if (dp[mask] == INT_MAX) continue;
    int next = getNextItem(mask);                  // problem-specific
    for (int choice : getChoices(next))
        dp[mask|(1<<choice)] = min(dp[mask|(1<<choice)], dp[mask] + cost(choice));
}
```

---

## Universal Checklist Before Coding

- [ ] What is the **state**? Write dp[i] means "..." in English first
- [ ] What is the **recurrence**? How does dp[i] depend on smaller states?
- [ ] What are the **base cases**?
- [ ] What is the **final answer**? dp[n]? max of all dp[i]? dp[0][n-1]?
- [ ] 0/1 knapsack → right to left; unbounded → left to right
- [ ] Interval DP → iterate by **length**, not i or j
- [ ] State machine → define all states and transitions explicitly
- [ ] Can I optimize space? (rolling array, two variables)
- [ ] Trace through small example before coding

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 70   | Climbing Stairs                         | Easy   | Linear DP                |
| 2  | 198  | House Robber                            | Medium | Linear DP                |
| 3  | 322  | Coin Change                             | Medium | Unbounded Knapsack       |
| 4  | 300  | Longest Increasing Subsequence          | Medium | LIS                      |
| 5  | 1143 | Longest Common Subsequence              | Medium | LCS                      |
| 6  | 416  | Partition Equal Subset Sum              | Medium | 0/1 Knapsack             |
| 7  | 72   | Edit Distance                           | Medium | LCS family               |
| 8  | 121  | Best Time to Buy & Sell Stock           | Easy   | State machine            |
| 9  | 5    | Longest Palindromic Substring           | Medium | Palindrome               |
| 10 | 62   | Unique Paths                            | Medium | Grid DP                  |
| 11 | 221  | Maximal Square                          | Medium | Grid DP                  |
| 12 | 312  | Burst Balloons                          | Hard   | Interval DP              |
| 13 | 309  | Best Time with Cooldown                 | Medium | State machine            |
| 14 | 518  | Coin Change II                          | Medium | Unbounded Knapsack       |
| 15 | 516  | Longest Palindromic Subsequence         | Medium | LCS / Palindrome         |
| 16 | 213  | House Robber II                         | Medium | Linear DP circular       |
| 17 | 354  | Russian Doll Envelopes                  | Hard   | LIS 2D                   |
| 18 | 132  | Palindrome Partitioning II              | Hard   | Palindrome DP            |
| 19 | 188  | Best Time to Buy & Sell Stock IV        | Hard   | State machine k-tx       |
| 20 | 494  | Target Sum                              | Medium | 0/1 Knapsack             |
| 21 | 152  | Maximum Product Subarray                | Medium | Linear DP (track min/max)|
| 22 | 139  | Word Break                              | Medium | Linear DP                |
| 23 | 115  | Distinct Subsequences                   | Hard   | LCS family               |
| 24 | 32   | Longest Valid Parentheses               | Hard   | Linear DP + stack        |
| 25 | 1312 | Minimum Insertions to Make Palindrome   | Hard   | Palindrome DP            |

**Study order:** #70 → #198 → #322 → #416 → #300 → #1143 → #72 → #121 → #309 → #312
These ten cover every core pattern.

---

## Company-Wise Most Asked Problems

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #322 Coin Change, #300 LIS, #198 House Robber, #139 Word Break, #62 Unique Paths |
| **Google** | #72 Edit Distance, #312 Burst Balloons, #354 Russian Dolls, #188 Stock IV, #132 Palindrome II |
| **Meta** | #70 Climbing Stairs, #198 House Robber, #5 Longest Palindrome, #121 Stock I, #1143 LCS |
| **Microsoft** | #70 Climbing Stairs, #322 Coin Change, #62 Unique Paths, #300 LIS, #416 Partition |
| **Bloomberg** | #198 House Robber, #322 Coin Change, #121 Stock, #5 Palindrome, #139 Word Break |
| **Apple** | #70 Climbing Stairs, #198 House Robber, #121 Stock, #322 Coin Change |
| **Uber** | #312 Burst Balloons, #354 Russian Dolls, #188 Stock IV |

---

## Recent Interview Trends (2023–2025)

### 1. Knapsack is the single most tested DP pattern
#416, #494, #518, #322 — all knapsack. Know 0/1 vs unbounded cold.
The direction of the inner loop (left/right) is the most common interview mistake.

### 2. Stock problems are asked at every company
All 6 stock variations (#121, #122, #123, #188, #309, #714) share the same state machine skeleton.
Interviewers often ask you to add a constraint (fee, cooldown, limit) after you solve the base.

### 3. Edit Distance is a Google / Meta classic
#72 is consistently in the top-5 DP problems at Google. Know it from memory.
Understand the three operations (insert, delete, replace) and which direction each corresponds to.

### 4. Interval DP appears at senior-level interviews
#312 Burst Balloons is the go-to hard DP at Google/Uber. The "think of k as the last one processed"
insight is non-obvious — practice explaining it clearly.

### 5. LIS in O(n log n) is expected at mid-level+
The O(n²) solution will be accepted but interviewers follow up: "can you do O(n log n)?"
Know the patience sorting / binary search approach.

### 6. DP + string is rising at Meta
Wildcard matching (#44), regex (#10), distinct subsequences (#115) test both DP and string parsing.
These combine LCS patterns with careful base case handling.

### 7. State machine framing impresses interviewers
Instead of writing ad-hoc DP for stock problems, explicitly naming your states
(held, free, cooldown) and writing clean transitions shows structured thinking.
