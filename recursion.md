# Recursion Patterns — LeetCode C++ Reference

## How to Decide Which Recursion Pattern

```
Single step dependency (f(n) needs f(n-1))?          → Linear Recursion
Multiple dependencies (f(n) needs f(n-1) + f(n-2))?  → Tree Recursion + Memoization
Split into independent halves, merge result?           → Divide and Conquer
Include this item OR skip it (knapsack/coin)?          → Multi-Dimensional Recursion
Last op is the recursive call, building accumulator?   → Tail Recursion
```

**The one question to always ask first:**
> "Does this problem break into smaller versions of itself?" — if yes, it's recursion.
> Then ask "do sub-problems overlap?" — if yes, add memoization.

---

## Recursion Pattern 1: Linear Recursion

**When to think about it:**
- f(n) depends only on f(n-1) — single chain, no branching
- "Reverse a list", "traverse nodes one by one", "process elements sequentially"
- Can always be replaced with a loop, but recursion is often cleaner

**Core insight:** Each call does one unit of work and delegates the rest.
The call stack naturally unwinds in reverse order — useful for reversals.

```
factorial(4)
  = 4 * factorial(3)
       = 3 * factorial(2)
            = 2 * factorial(1)
                 = 1  <- base case
```

```cpp
// Linear Recursion Template
int factorial(int n) {
    if (n <= 1) return 1;           // base case
    return n * factorial(n - 1);   // one recursive call
}

// Reverse a linked list
ListNode* reverse(ListNode* head) {
    if (!head || !head->next) return head;
    ListNode* rest = reverse(head->next);
    head->next->next = head;
    head->next = nullptr;
    return rest;
}
```

**Problems:** #206 Reverse Linked List, #21 Merge Two Sorted Lists, #118 Pascal's Triangle

---

## Recursion Pattern 2: Tree Recursion + Memoization

**When to think about it:**
- f(n) depends on f(n-1) AND f(n-2) — multiple prior states, branching calls
- Fibonacci, climbing stairs, branching decisions
- Naive version is exponential — same sub-problem computed many times
- You draw the call tree and see **repeated sub-trees** — that's the memoization signal

**Core insight:** Without memoization, the call tree has 2^n nodes. With memoization,
each unique sub-problem is solved exactly once → O(n).

```
fib(5) call tree (naive — notice fib(3) computed TWICE):

         fib(5)
        /      \
    fib(4)    fib(3)
    /    \    /    \
 fib(3) fib(2) fib(2) fib(1)
 /   \
fib(2) fib(1)
```

```cpp
// Naive — O(2^n), DO NOT use for large n
int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
}

// With Memoization — O(n)
unordered_map<int, int> memo;

int fib(int n) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];         // cache hit
    memo[n] = fib(n - 1) + fib(n - 2);        // compute once
    return memo[n];
}

// Cleaner: pass memo as parameter
int fib(int n, unordered_map<int,int>& memo) {
    if (n <= 1) return n;
    if (memo.count(n)) return memo[n];
    return memo[n] = fib(n-1, memo) + fib(n-2, memo);
}
```

**When to add memoization:** Draw the call tree. If you see the same (n, params) appear
more than once — add a cache. If every call is unique — memoization won't help.

**Problems:** #509 Fibonacci Number, #70 Climbing Stairs, #198 House Robber, #322 Coin Change

---

## Recursion Pattern 3: Divide and Conquer

**When to think about it:**
- Problem splits into two **independent** halves (not overlapping like DP)
- "Sort this", "find max subarray", "merge k sorted lists"
- Key phrase: **divide** the input, **conquer** each half, **combine** the results
- Sub-problems do NOT share state — that's what separates it from memoized DP

**Core insight:** Split the input in half each time → O(log n) depth.
Each level does O(n) work → total O(n log n).

```
mergeSort([5,3,1,4,2])
    mergeSort([5,3,1])            mergeSort([4,2])
        mergeSort([5,3]) mergeSort([1])  mergeSort([4]) mergeSort([2])
          merge([3,5],[1])=[1,3,5]         merge([4],[2])=[2,4]
    merge([1,3,5], [2,4]) = [1,2,3,4,5]
```

```cpp
// Divide and Conquer Template
ReturnType solve(vector<int>& arr, int left, int right) {
    if (left == right) return arr[left];       // base: single element

    int mid = left + (right - left) / 2;

    auto L = solve(arr, left, mid);            // conquer left
    auto R = solve(arr, mid + 1, right);       // conquer right

    return combine(L, R);                      // combine
}

// Merge Sort
void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    merge(arr, l, mid, r);          // combine step
}

// Maximum Subarray (Divide & Conquer)
int maxSub(vector<int>& nums, int l, int r) {
    if (l == r) return nums[l];
    int mid = l + (r - l) / 2;
    int leftMax  = maxSub(nums, l, mid);
    int rightMax = maxSub(nums, mid + 1, r);

    // crossing sum — expand outward from mid
    int leftCross = INT_MIN, sum = 0;
    for (int i = mid; i >= l; i--)    leftCross  = max(leftCross,  sum += nums[i]);
    int rightCross = INT_MIN; sum = 0;
    for (int i = mid+1; i <= r; i++)  rightCross = max(rightCross, sum += nums[i]);

    return max({leftMax, rightMax, leftCross + rightCross});
}
```

**Problems:** #23 Merge k Sorted Lists, #148 Sort List, #53 Maximum Subarray, #241 Different Ways to Add Parentheses

---

## Recursion Pattern 4: Multi-Dimensional / Include-Exclude

**When to think about it:**
- "How many ways to make amount X using these coins?"
- "Can we partition this array into equal subsets?"
- At each element you make a **binary choice**: include it OR skip it
- Two parameters change together → 2D state → must memoize both

**Core insight:** At each step, branch into two recursive calls:
one that uses the current item, one that skips it.
Add memoization on both parameters — the state is (target, index).

```
coins = [1,2,5], amount = 5

solve(5, idx=0):
  use coin[0]=1 → solve(4, idx=0)
  skip coin[0]  → solve(5, idx=1)
    use coin[1]=2 → solve(3, idx=1)
    skip coin[1]  → solve(5, idx=2)
      use coin[2]=5 → solve(0, idx=2) <- FOUND
```

```cpp
// Include/Exclude Template with 2D memoization
map<pair<int,int>, int> memo;

int solve(vector<int>& items, int target, int idx) {
    if (target == 0) return 1;                       // found valid combo
    if (idx == (int)items.size() || target < 0) return 0;

    auto key = make_pair(target, idx);
    if (memo.count(key)) return memo[key];

    int skip    = solve(items, target, idx + 1);           // skip item
    int include = solve(items, target - items[idx], idx);  // reuse allowed
    // for no-reuse: solve(items, target - items[idx], idx + 1)

    return memo[key] = skip + include;
}
```

**Problems:** #322 Coin Change, #518 Coin Change II, #416 Partition Equal Subset Sum, #377 Combination Sum IV

---

## Recursion Pattern 5: Tail Recursion

**When to think about it:**
- The recursive call is the **very last** thing the function does
- You're building an answer in an **accumulator** parameter passed forward
- No "pending work" after the recursive call returns

**Core insight:** Nothing is deferred — the accumulator carries the result forward.
Compilers can optimize this to a loop (O(1) stack space vs O(n) for normal recursion).

```
// Normal recursion — result computed on the WAY BACK UP (deferred work)
factorial(4) = 4 * factorial(3) = 4 * (3 * factorial(2)) = ...

// Tail recursion — result built on the WAY DOWN (no deferred work)
factTail(4, acc=1) -> factTail(3, acc=4) -> factTail(2, acc=12)
                   -> factTail(1, acc=24) -> return 24
```

```cpp
// Tail Recursion Template
int factTail(int n, int acc = 1) {
    if (n <= 1) return acc;
    return factTail(n - 1, acc * n);   // acc carries the result forward
}

// GCD — classic tail recursion
int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);              // last operation is the recursive call
}
```

**Problems:** Mainly a pattern to recognize and convert — GCD, accumulator-style problems

---

## Universal Checklist Before Coding

- [ ] What is the **base case**? (smallest valid input that returns directly)
- [ ] What is the **recurrence**? (how does f(n) use f(smaller)?)
- [ ] Do sub-problems **overlap**? → add memoization
- [ ] Are sub-problems **independent**? → divide and conquer, no memo needed
- [ ] Is the stack depth O(n)? → risk of overflow on large inputs, consider iterative
- [ ] Is the return type a count/value or a collection? → affects how you combine results

---

## Problem List

| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 206 | Reverse Linked List              | Easy   | Linear recursion          |
| 21  | Merge Two Sorted Lists           | Easy   | Linear recursion          |
| 104 | Maximum Depth of Binary Tree     | Easy   | Tree recursion            |
| 509 | Fibonacci Number                 | Easy   | Tree recursion + memo     |
| 70  | Climbing Stairs                  | Easy   | Tree recursion + memo     |
| 118 | Pascal's Triangle                | Easy   | Linear recursion          |
| 198 | House Robber                     | Medium | Tree recursion + memo     |
| 322 | Coin Change                      | Medium | Multi-dimensional + memo  |
| 518 | Coin Change II                   | Medium | Multi-dimensional + memo  |
| 416 | Partition Equal Subset Sum       | Medium | Multi-dimensional + memo  |
| 53  | Maximum Subarray                 | Medium | Divide and conquer        |
| 23  | Merge k Sorted Lists             | Hard   | Divide and conquer        |
| 148 | Sort List                        | Medium | Divide and conquer        |
| 241 | Different Ways to Add Parentheses| Medium | Divide and conquer        |

---

## Complexity Reference

| Pattern              | Time       | Space  | Key Insight                              |
|----------------------|------------|--------|------------------------------------------|
| Linear Recursion     | O(n)       | O(n)   | One call per element, stack depth = n    |
| Tree Recursion       | O(2^n)     | O(n)   | Exponential without memoization          |
| Tree + Memoization   | O(n)       | O(n)   | Each unique sub-problem solved once      |
| Divide and Conquer   | O(n log n) | O(log n) | Half input each level, log n depth     |
| Multi-Dimensional    | O(n * m)   | O(n * m) | State is (index, target) pair          |
| Tail Recursion       | O(n)       | O(1)*  | Compiler optimizes to loop (*if TCO)     |
