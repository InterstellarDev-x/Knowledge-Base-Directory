# Stack Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
See "next greater/smaller"?         → Monotonic Stack
See brackets / nesting?             → Bracket Validation
Need getMin() in O(1)?              → Auxiliary Stack
String with math operators?         → Expression Evaluation
Implement queue with stacks?        → Two-Stack Queue
Heights array + max area?           → Histogram (Monotonic on indices)
Tree traversal without recursion?   → Iterative DFS
```

**The one question to always ask first:**
> "Do I need to look back at recent elements, and does the most recent one matter most?" — if yes, it's almost certainly a stack.

---

## C++ Stack Quick Reference

```cpp
#include <stack>

stack<int> st;

st.push(x);   // push element
st.pop();     // remove top (returns void!)
st.top();     // peek top (does NOT remove)
st.empty();   // true if empty
st.size();    // number of elements
```

> **Gotcha:** `st.pop()` returns `void` — always read with `st.top()` first, then call `st.pop()`.

---

## Pattern 1: Monotonic Stack — Next Greater/Smaller Element

**When to think about it:**
- Problem asks for "next greater", "next smaller", "previous larger", "previous smaller"
- Involves temperatures, stock prices, buildings, heights over time
- You find yourself doing O(n²) nested loops comparing each element to every other
- The phrase "for each element, find the nearest element bigger/smaller than it"

**Core insight:** When you're about to push a new element and it's larger than the top,
the top element has finally "found its answer" — pop it and record the answer.

```
nums = [2, 1, 3, 4]

Push 2  → stack: [2]
Push 1  → 1 < 2, just push        → stack: [2, 1]
Push 3  → 3 > 1, pop 1 (answer=3)
        → 3 > 2, pop 2 (answer=3) → stack: [3]
Push 4  → 4 > 3, pop 3 (answer=4) → stack: [4]

result = [3, 3, 4, -1]
```

```cpp
// Next GREATER (stack stays decreasing)
vector<int> nextGreater(vector<int>& nums) {
    stack<int> st;          // stores indices
    vector<int> result(nums.size(), -1);

    for (int i = 0; i < nums.size(); i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            result[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}

// Next SMALLER → change < to >
```

**Store index vs value:**
- Store **index** when you need distance or width → `nums[st.top()]`
- Store **value** when you only need the answer value → `st.top()`

**Problems:** #739 Daily Temperatures, #496 Next Greater Element I, #503 Next Greater Element II

---

## Pattern 2: Bracket / Parenthesis Validation

**When to think about it:**
- Problem has `( ) { } [ ]` or any open/close pairs
- Problem says "valid", "balanced", "matched", "well-formed"
- Nested structures — HTML tags, function calls, file paths
- Rule is: whatever opened last must close first (LIFO!)

**Core insight:** LIFO is exactly how nesting works. A stack naturally enforces that
the innermost bracket closes before the outer one.

```
s = "({[]})"

Push (   → stack: [(]
Push {   → stack: [(, {]
Push [   → stack: [(, {, []
See ]    → top is [, matches → pop   → stack: [(, {]
See }    → top is {, matches → pop   → stack: [(]
See )    → top is (, matches → pop   → stack: []
Stack empty = valid ✓
```

```cpp
bool isValid(string s) {
    stack<char> st;
    unordered_map<char, char> map = {{')', '('}, {'}', '{'}, {']', '['}};

    for (char ch : s) {
        if (map.count(ch)) {                        // closing bracket
            if (st.empty() || st.top() != map[ch])
                return false;
            st.pop();
        } else {
            st.push(ch);                            // opening bracket
        }
    }
    return st.empty();
}
```

**The three failure cases:**
```
"]"    → stack empty when closing  → false
"([)]" → top doesn't match        → false
"((("  → stack not empty at end   → false
```

**Problems:** #20 Valid Parentheses, #32 Longest Valid Parentheses, #921 Min Add to Make Valid

---

## Pattern 3: Min/Max Stack (Auxiliary Stack)

**When to think about it:**
- Asked to design a data structure with push, pop, top, getMin/getMax all in O(1)
- First instinct is to scan whole stack for min — that's O(n), too slow
- When you pop an element, the minimum might change

**Core insight:** Maintain a second stack where the top always holds the current minimum.
Only push to min stack when new value is <= current min.

```
push(5) → st: [5],      minSt: [5]
push(3) → st: [5,3],    minSt: [5,3]    (3 <= 5, push)
push(7) → st: [5,3,7],  minSt: [5,3]    (7 > 3, skip)
push(2) → st: [5,3,7,2] minSt: [5,3,2]  (2 <= 3, push)

getMin() = 2  ✓

pop() removes 2, 2 == minSt.top() → pop minSt too
      st: [5,3,7]  minSt: [5,3]
getMin() = 3  ✓
```

```cpp
class MinStack {
    stack<int> st;
    stack<int> minSt;

public:
    void push(int val) {
        st.push(val);
        if (minSt.empty() || val <= minSt.top())   // <= handles duplicates
            minSt.push(val);
    }

    void pop() {
        if (st.top() == minSt.top())
            minSt.pop();
        st.pop();
    }

    int top()    { return st.top(); }
    int getMin() { return minSt.top(); }
};
```

**Why `<=` not `<`:** If you push duplicate minimums (e.g., 3 twice), use `<=` so both
copies are tracked. Otherwise popping one copy would erase the only minSt entry.

**Problems:** #155 Min Stack, #716 Max Stack

---

## Pattern 4: Expression Evaluation

**When to think about it:**
- Input is a string with digits, `+`, `-`, `*`, `/`, spaces, maybe `()`
- Need to respect operator precedence (`*` `/` before `+` `-`)
- Cannot just evaluate left to right

**Core insight:**
- `+` and `-` can be deferred (push to stack, sum at the end)
- `*` and `/` must be resolved immediately (pop, compute, push back)
- Sum the stack at the end — precedence already handled

```
s = "3 + 2 * 2"

sign='+', num=3 → push 3              → stack: [3]
sign='+', update to '+'
sign='+', num=2 → push 2              → stack: [3, 2]
sign='+', update to '*'
sign='*', num=2 → pop 2, 2*2=4, push 4 → stack: [3, 4]

sum = 3 + 4 = 7  ✓
```

```cpp
int calculate(string s) {
    stack<int> st;
    int num = 0;
    char sign = '+';

    for (int i = 0; i < s.size(); i++) {
        char ch = s[i];

        if (isdigit(ch))
            num = num * 10 + (ch - '0');

        if ((!isdigit(ch) && ch != ' ') || i == s.size() - 1) {
            if      (sign == '+') st.push(num);
            else if (sign == '-') st.push(-num);
            else if (sign == '*') { int t = st.top(); st.pop(); st.push(t * num); }
            else if (sign == '/') { int t = st.top(); st.pop(); st.push(t / num); }
            num = 0;
            sign = ch;
        }
    }

    int result = 0;
    while (!st.empty()) { result += st.top(); st.pop(); }
    return result;
}
```

**Key trick:** Process the number when you see the NEXT operator, applying the PREVIOUS sign.

**Problems:** #227 Basic Calculator II, #150 Evaluate Reverse Polish Notation, #224 Basic Calculator, #394 Decode String

---

## Pattern 5: Two-Stack Queue

**When to think about it:**
- Asked to implement a queue using only stacks
- Need O(1) amortized push and pop
- Constraint says "only use stack operations"

**Core insight:** A stack reverses order. Two stacks reverse twice = original order (FIFO).

```
push(1), push(2), push(3)
inStack: [1, 2, 3]   outStack: []

pop() → outStack empty, transfer:
        inStack: []   outStack: [3, 2, 1]
        pop outStack → returns 1  (FIFO) ✓

pop() → outStack not empty, pop directly → returns 2  ✓
```

```cpp
class MyQueue {
    stack<int> inSt, outSt;

    void transfer() {
        if (outSt.empty())
            while (!inSt.empty()) { outSt.push(inSt.top()); inSt.pop(); }
    }

public:
    void push(int x)  { inSt.push(x); }
    int  pop()        { transfer(); int v = outSt.top(); outSt.pop(); return v; }
    int  peek()       { transfer(); return outSt.top(); }
    bool empty()      { return inSt.empty() && outSt.empty(); }
};
```

**Key rule:** Only transfer when `outSt` is empty — not on every pop. Each element moves
at most twice total → O(1) amortized.

**Problems:** #232 Implement Queue using Stacks, #225 Implement Stack using Queues

---

## Pattern 6: Histogram / Area (Monotonic Stack on Indices)

**When to think about it:**
- Heights/bars array, need maximum rectangle area
- Problem mentions "rectangle", "container", "water trapped"
- Brute force is checking all pairs of bars — O(n²)
- Key question: "how wide can a bar of this height extend?"

**Core insight:** A bar can extend left and right only while neighboring bars are >= its height.
The moment you see a shorter bar, the current bar's reach stops — that's the pop signal.

```
heights = [2, 1, 5, 6, 2, 3]

i=1: h=1 < h[0]=2 → pop 0
       width = i = 1 (stack empty)
       area = 2*1 = 2

i=4: h=2 < h[3]=6 → pop 3, width = 4-2-1=1, area = 6*1 = 6
     h=2 < h[2]=5 → pop 2, width = 4-1-1=2, area = 5*2 = 10  <- max
```

```cpp
int largestRectangleArea(vector<int>& heights) {
    stack<int> st;          // stores indices
    int maxArea = 0;
    int n = heights.size();

    for (int i = 0; i <= n; i++) {
        int h = (i == n) ? 0 : heights[i];   // sentinel 0 at end flushes stack

        while (!st.empty() && heights[st.top()] > h) {
            int height = heights[st.top()]; st.pop();
            int width  = st.empty() ? i : i - st.top() - 1;
            maxArea = max(maxArea, height * width);
        }
        st.push(i);
    }
    return maxArea;
}
```

**Width formula:**
```cpp
int width = st.empty() ? i : i - st.top() - 1;
//          no left boundary    left boundary is new top after pop
```

**Problems:** #84 Largest Rectangle in Histogram, #85 Maximal Rectangle, #42 Trapping Rain Water

---

## Pattern 7: Iterative DFS (Stack Replaces Recursion)

**When to think about it:**
- Tree or graph traversal, but need iterative solution
- Recursive solution risks stack overflow on large inputs
- Interview explicitly asks for iterative traversal

**Core insight:** Recursion uses the call stack implicitly — make it explicit.
Each "pending work" item goes on your own stack.

```
Recursive inorder:          Iterative:
inorder(node):              push node, go left until null
  inorder(node.left)   →    pop, visit, go right
  visit(node)
  inorder(node.right)
```

```cpp
// INORDER: left -> visit -> right
vector<int> inorderTraversal(TreeNode* root) {
    stack<TreeNode*> st;
    vector<int> result;
    TreeNode* cur = root;

    while (cur || !st.empty()) {
        while (cur) { st.push(cur); cur = cur->left; }
        cur = st.top(); st.pop();
        result.push_back(cur->val);
        cur = cur->right;
    }
    return result;
}

// PREORDER: visit -> left -> right
vector<int> preorderTraversal(TreeNode* root) {
    if (!root) return {};
    stack<TreeNode*> st;
    vector<int> result;
    st.push(root);

    while (!st.empty()) {
        auto node = st.top(); st.pop();
        result.push_back(node->val);            // visit first
        if (node->right) st.push(node->right);  // push right first
        if (node->left)  st.push(node->left);   // so left is processed first
    }
    return result;
}

// POSTORDER: left -> right -> visit
// Do modified preorder (visit, right, left), then reverse result
vector<int> postorderTraversal(TreeNode* root) {
    if (!root) return {};
    stack<TreeNode*> st;
    vector<int> result;
    st.push(root);

    while (!st.empty()) {
        auto node = st.top(); st.pop();
        result.push_back(node->val);
        if (node->left)  st.push(node->left);
        if (node->right) st.push(node->right);
    }
    reverse(result.begin(), result.end());
    return result;
}
```

**Problems:** #94 Inorder Traversal, #144 Preorder Traversal, #145 Postorder Traversal, #200 Number of Islands

---

## Universal Checklist Before Coding

- [ ] Store **indices or values**? (histogram/distance -> indices, brackets -> chars)
- [ ] Pop while condition is **true** (monotonic) or check once (brackets)?
- [ ] Is the array **circular**? -> use `i % n` trick (e.g., #503)
- [ ] Handle **remaining elements in stack** after the main loop?
- [ ] Check `if (!st.empty())` before every `.top()` or `.pop()`
- [ ] Does the problem need `<=` or `<` in the monotonic condition?

---

---

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
> "Does this problem break into smaller versions of itself?" — if yes, it's recursion. Then ask "do sub-problems overlap?" — if yes, add memoization.

---

## Recursion Pattern 1: Linear Recursion

**When to think about it:**
- f(n) depends only on f(n-1) — single chain
- "Reverse a list", "traverse nodes one by one", "process elements sequentially"
- Can always be replaced with a loop, but recursion is often cleaner

**Core insight:** Each call does one unit of work and delegates the rest.
The call stack naturally unwinds in reverse order — useful for reversals.

```
factorial(4)
  = 4 * factorial(3)
       = 3 * factorial(2)
            = 2 * factorial(1)
                 = 1  ← base case
```

```cpp
// Linear Recursion Template
int factorial(int n) {
    if (n <= 1) return 1;            // base case
    return n * factorial(n - 1);    // one recursive call
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
- f(n) depends on f(n-1) AND f(n-2) (multiple prior states)
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
    if (memo.count(n)) return memo[n];          // cache hit
    memo[n] = fib(n - 1) + fib(n - 2);         // compute once
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
- Sub-problems do NOT share state — that's what separates it from memoization DP

**Core insight:** Split the input in half each time → O(log n) depth.
Each level does O(n) work → total O(n log n).

```
mergeSort([5,3,1,4,2])
    mergeSort([5,3,1])        mergeSort([4,2])
        mergeSort([5,3])  mergeSort([1])  mergeSort([4])  mergeSort([2])
            merge([3,5], [1]) = [1,3,5]      merge([4],[2]) = [2,4]
    merge([1,3,5], [2,4]) = [1,2,3,4,5]
```

```cpp
// Divide and Conquer Template
ReturnType solve(vector<int>& arr, int left, int right) {
    if (left == right) return arr[left];        // base: single element

    int mid = left + (right - left) / 2;

    auto L = solve(arr, left, mid);             // conquer left
    auto R = solve(arr, mid + 1, right);        // conquer right

    return combine(L, R);                       // combine
}

// Merge Sort
void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    merge(arr, l, mid, r);           // combine step
}

// Maximum Subarray (Divide & Conquer approach)
int maxSubarray(vector<int>& nums, int l, int r) {
    if (l == r) return nums[l];
    int mid = l + (r - l) / 2;
    int leftMax  = maxSubarray(nums, l, mid);
    int rightMax = maxSubarray(nums, mid + 1, r);
    // also consider crossing the midpoint
    int leftCross = INT_MIN, sum = 0;
    for (int i = mid; i >= l; i--)   leftCross = max(leftCross, sum += nums[i]);
    int rightCross = INT_MIN; sum = 0;
    for (int i = mid+1; i <= r; i++) rightCross = max(rightCross, sum += nums[i]);
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
- Two parameters change together → 2D state

**Core insight:** At each step, branch into two recursive calls:
one that uses the current item, one that skips it.
Add memoization on both parameters.

```
coins = [1,2,5], amount = 5

coinChange(5, index=0):
  use coin[0]=1 → coinChange(4, index=0)
  skip coin[0]  → coinChange(5, index=1)
    use coin[1]=2 → coinChange(3, index=1)
    skip coin[1]  → coinChange(5, index=2)
      use coin[2]=5 → coinChange(0, index=2) ← FOUND
```

```cpp
// Include/Exclude Template
map<pair<int,int>, int> memo;

int solve(vector<int>& items, int target, int idx) {
    if (target == 0) return 1;                    // found valid combo
    if (idx == items.size() || target < 0) return 0;

    auto key = make_pair(target, idx);
    if (memo.count(key)) return memo[key];

    int skip    = solve(items, target, idx + 1);          // skip item
    int include = solve(items, target - items[idx], idx); // reuse allowed
    // for no-reuse: solve(items, target - items[idx], idx + 1)

    return memo[key] = skip + include;
}
```

**Problems:** #322 Coin Change, #518 Coin Change II, #416 Partition Equal Subset Sum, #377 Combination Sum IV

---

## Recursion Pattern 5: Tail Recursion

**When to think about it:**
- The recursive call is the **very last** thing the function does
- You're building an answer in an accumulator parameter
- No "pending work" after the recursive call returns

**Core insight:** Nothing is deferred — the accumulator carries the result forward.
Compilers can optimize this to a loop (O(1) stack space vs O(n)).

```
// Normal recursion — result computed on the WAY BACK UP
factorial(4) = 4 * factorial(3) = 4 * (3 * factorial(2)) = ...

// Tail recursion — result built on the WAY DOWN
factTail(4, acc=1) → factTail(3, acc=4) → factTail(2, acc=12)
                  → factTail(1, acc=24) → return 24
```

```cpp
// Tail Recursion Template
int factTail(int n, int acc = 1) {
    if (n <= 1) return acc;
    return factTail(n - 1, acc * n);    // acc carries the result
}

// GCD — classic tail recursion
int gcd(int a, int b) {
    if (b == 0) return a;
    return gcd(b, a % b);               // last operation is the call
}
```

**Problems:** Mainly a pattern to recognize and convert — GCD, accumulator-style DP

---

## Recursion: Universal Checklist

- [ ] What is the **base case**? (smallest valid input that returns directly)
- [ ] What is the **recurrence**? (how does f(n) use f(n-1) or smaller?)
- [ ] Do sub-problems **overlap**? → add memoization
- [ ] Are sub-problems **independent**? → divide and conquer, no memo needed
- [ ] Is the stack depth O(n)? → risk of overflow on large inputs, consider iterative

---

---

# Backtracking Patterns — LeetCode C++ Reference

## Backtracking vs Plain Recursion

| | Plain Recursion | Backtracking |
|---|---|---|
| **Goal** | Compute a value | Find all / any valid solutions |
| **Search space** | Defined by recurrence | Implicit tree of all candidates |
| **Pruning** | Usually none | Aggressive — cut invalid branches early |
| **State** | Parameters only | Shared state that must be restored |
| **Undo step** | Never needed | Always needed (choose → explore → unchoose) |

**Recognition clues for backtracking:**
- "Find ALL solutions / ALL combinations / ALL permutations"
- "Find ANY valid arrangement" with hard constraints
- Solution built incrementally — each step narrows choices
- No obvious DP or greedy approach
- Examples: Sudoku, N-Queens, word search, all subsets/permutations

---

## The Universal Backtracking Template

```cpp
void backtrack(State& state, vector<Solution>& results) {

    // 1. BASE CASE: valid complete solution found
    if (isGoal(state)) {
        results.push_back(state);
        return;
    }

    for (auto& choice : getChoices(state)) {

        // 2. PRUNING: skip invalid choices early
        if (!isValid(state, choice)) continue;

        // 3. CHOOSE: make the choice, modify state
        applyChoice(state, choice);

        // 4. EXPLORE: recurse
        backtrack(state, results);

        // 5. UNCHOOSE: undo the choice, restore state
        undoChoice(state, choice);
    }
}
```

**The three-step heartbeat:** CHOOSE → EXPLORE → UNCHOOSE
Every backtracking solution follows this rhythm.

---

## Backtracking Pattern 1: Subsets / Power Set

**When to think about it:**
- "Generate all subsets", "power set", "all possible selections"
- No size constraint, no target — every partial state is a valid answer
- 2^n total subsets

**Core insight:** At each index, you either include the element or skip it.
Record the current state at every node of the recursion tree (not just leaves).

```
nums = [1,2,3]

[]                           ← record
├── [1]                      ← record
│   ├── [1,2]                ← record
│   │   └── [1,2,3]         ← record
│   └── [1,3]                ← record
├── [2]                      ← record
│   └── [2,3]                ← record
└── [3]                      ← record
```

```cpp
void subsets(vector<int>& nums, int start,
             vector<int>& cur, vector<vector<int>>& res) {

    res.push_back(cur);          // record at every node, not just leaf

    for (int i = start; i < nums.size(); i++) {
        cur.push_back(nums[i]);              // choose
        subsets(nums, i + 1, cur, res);      // explore
        cur.pop_back();                      // unchoose
    }
}
// Call: subsets(nums, 0, cur, res);

// Subsets II (duplicates) — sort first, skip duplicate siblings
void subsetsII(vector<int>& nums, int start,
               vector<int>& cur, vector<vector<int>>& res) {
    res.push_back(cur);
    for (int i = start; i < nums.size(); i++) {
        if (i > start && nums[i] == nums[i-1]) continue;  // skip duplicate
        cur.push_back(nums[i]);
        subsetsII(nums, i + 1, cur, res);
        cur.pop_back();
    }
}
```

**Problems:** #78 Subsets, #90 Subsets II

---

## Backtracking Pattern 2: Combinations

**When to think about it:**
- "Find all combinations of size k" or "combinations that sum to target"
- Order does NOT matter (unlike permutations)
- `start` pointer prevents re-picking earlier elements

**Core insight:** Only move forward (pass `i` or `i+1` to next call), never backward.
This avoids duplicates like [1,2] and [2,1] both appearing.

```
candidates = [2,3,6,7], target = 7

[] → try 2 → [2]
          → try 2 → [2,2]
                  → try 2 → [2,2,2]
                           → try 2 → [2,2,2,2] (2+2+2+2=8 > 7, prune)
                           → try 3 → [2,2,3] (2+2+3=7) FOUND ✓
```

```cpp
// Combination Sum (elements reusable, no duplicates in input)
void combinationSum(vector<int>& cands, int target, int start,
                    vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }

    for (int i = start; i < cands.size(); i++) {
        if (cands[i] > target) break;           // pruning (sort first)

        cur.push_back(cands[i]);
        combinationSum(cands, target - cands[i], i, cur, res);  // i: reuse allowed
        cur.pop_back();
    }
}

// Combination Sum II (no reuse, may have duplicates in input)
void combinationSumII(vector<int>& cands, int target, int start,
                      vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }

    for (int i = start; i < cands.size(); i++) {
        if (i > start && cands[i] == cands[i-1]) continue;  // skip duplicate siblings
        if (cands[i] > target) break;

        cur.push_back(cands[i]);
        combinationSumII(cands, target - cands[i], i + 1, cur, res);  // i+1: no reuse
        cur.pop_back();
    }
}
```

**Reuse vs no-reuse:**
```
Reuse allowed   → pass i   (same index again)
No reuse        → pass i+1 (move to next element)
```

**Problems:** #39 Combination Sum, #40 Combination Sum II, #77 Combinations, #216 Combination Sum III

---

## Backtracking Pattern 3: Permutations

**When to think about it:**
- "Find all permutations", "all orderings", "all arrangements"
- Order MATTERS — [1,2,3] and [3,2,1] are different answers
- Every element is used exactly once per permutation

**Core insight:** At each position, try every unused element.
Use either a `visited[]` array or the swap trick.

```
nums = [1,2,3]

fix 1 at pos 0 → permute [2,3]:
    fix 2 at pos 1 → [1,2,3] ✓
    fix 3 at pos 1 → [1,3,2] ✓
fix 2 at pos 0 → permute [1,3]:
    fix 1 at pos 1 → [2,1,3] ✓
    fix 3 at pos 1 → [2,3,1] ✓
...
```

```cpp
// Permutations using visited array
void permute(vector<int>& nums, vector<bool>& visited,
             vector<int>& cur, vector<vector<int>>& res) {

    if (cur.size() == nums.size()) { res.push_back(cur); return; }

    for (int i = 0; i < nums.size(); i++) {
        if (visited[i]) continue;

        visited[i] = true;
        cur.push_back(nums[i]);              // choose
        permute(nums, visited, cur, res);    // explore
        cur.pop_back();                      // unchoose
        visited[i] = false;
    }
}

// Permutations using swap (modifies array in-place)
void permuteSwap(vector<int>& nums, int start, vector<vector<int>>& res) {
    if (start == nums.size()) { res.push_back(nums); return; }

    for (int i = start; i < nums.size(); i++) {
        swap(nums[start], nums[i]);              // choose
        permuteSwap(nums, start + 1, res);       // explore
        swap(nums[start], nums[i]);              // unchoose (swap back)
    }
}

// Permutations II (with duplicates) — sort first, skip same value at same position
void permuteUnique(vector<int>& nums, vector<bool>& visited,
                   vector<int>& cur, vector<vector<int>>& res) {
    if (cur.size() == nums.size()) { res.push_back(cur); return; }

    for (int i = 0; i < nums.size(); i++) {
        if (visited[i]) continue;
        // skip duplicate: same value, previous sibling not visited (already explored)
        if (i > 0 && nums[i] == nums[i-1] && !visited[i-1]) continue;

        visited[i] = true;
        cur.push_back(nums[i]);
        permuteUnique(nums, visited, cur, res);
        cur.pop_back();
        visited[i] = false;
    }
}
```

**Problems:** #46 Permutations, #47 Permutations II, #784 Letter Case Permutation

---

## Backtracking Pattern 4: N-Queens (Constraint Satisfaction)

**When to think about it:**
- "Place pieces on a board with no conflicts"
- Strong geometric/logical constraints → prune most branches
- Build row by row — each row has exactly one queen

**Core insight:** Check column, left diagonal, and right diagonal before placing.
Only recurse if placement is safe — prunes the tree drastically.

```
Row 0: try col 0,1,2,...
Row 1: for each row-0 placement, try remaining safe cols
...
```

```cpp
bool isSafe(vector<string>& board, int row, int col, int n) {
    for (int i = 0; i < row; i++)
        if (board[i][col] == 'Q') return false;                 // same column

    for (int i = row-1, j = col-1; i >= 0 && j >= 0; i--, j--)
        if (board[i][j] == 'Q') return false;                   // upper-left diagonal

    for (int i = row-1, j = col+1; i >= 0 && j < n; i--, j++)
        if (board[i][j] == 'Q') return false;                   // upper-right diagonal

    return true;
}

void solveNQueens(int row, int n, vector<string>& board,
                 vector<vector<string>>& res) {

    if (row == n) { res.push_back(board); return; }

    for (int col = 0; col < n; col++) {
        if (!isSafe(board, row, col, n)) continue;  // prune

        board[row][col] = 'Q';                      // choose
        solveNQueens(row + 1, n, board, res);        // explore
        board[row][col] = '.';                       // unchoose
    }
}
// Init: vector<string> board(n, string(n, '.'));
```

**Problems:** #51 N-Queens, #52 N-Queens II (count only)

---

## Backtracking Pattern 5: Sudoku Solver

**When to think about it:**
- "Fill a grid satisfying constraints per row / column / box"
- Try digits 1–9 for each empty cell, backtrack when stuck

**Core insight:** Return `false` immediately when no digit fits a cell —
this propagates the backtrack signal up to the previous cell.

```cpp
bool isValid(vector<vector<char>>& b, int r, int c, char d) {
    for (int i = 0; i < 9; i++) {
        if (b[r][i] == d) return false;                             // row
        if (b[i][c] == d) return false;                             // col
        if (b[3*(r/3)+i/3][3*(c/3)+i%3] == d) return false;       // 3x3 box
    }
    return true;
}

bool solve(vector<vector<char>>& board) {
    for (int r = 0; r < 9; r++) {
        for (int c = 0; c < 9; c++) {
            if (board[r][c] != '.') continue;

            for (char d = '1'; d <= '9'; d++) {
                if (!isValid(board, r, c, d)) continue;

                board[r][c] = d;                    // choose
                if (solve(board)) return true;      // explore
                board[r][c] = '.';                  // unchoose
            }

            return false;   // no digit worked → trigger backtrack
        }
    }
    return true;    // all cells filled
}
```

**Problems:** #37 Sudoku Solver

---

## Backtracking Pattern 6: Word Search / Grid Path

**When to think about it:**
- "Find a word in a 2D grid", "all paths from start to end"
- Move in 4 (or 8) directions, can't revisit a cell in the same path
- Mark cell visited before exploring neighbors, unmark after

**Core insight:** The `visited` flag is the "state" — mark it as the CHOOSE step,
unmark it as the UNCHOOSE step.

```cpp
bool dfs(vector<vector<char>>& board, string& word, int idx,
         int r, int c, vector<vector<bool>>& visited) {

    if (idx == word.size()) return true;        // found full word

    if (r < 0 || r >= board.size() ||
        c < 0 || c >= board[0].size() ||
        visited[r][c] || board[r][c] != word[idx])
        return false;

    visited[r][c] = true;                       // choose

    int dr[] = {-1, 1, 0, 0};
    int dc[] = {0, 0, -1, 1};
    for (int d = 0; d < 4; d++)
        if (dfs(board, word, idx+1, r+dr[d], c+dc[d], visited))
            return true;

    visited[r][c] = false;                      // unchoose
    return false;
}

bool exist(vector<vector<char>>& board, string word) {
    int m = board.size(), n = board[0].size();
    vector<vector<bool>> visited(m, vector<bool>(n, false));

    for (int i = 0; i < m; i++)
        for (int j = 0; j < n; j++)
            if (dfs(board, word, 0, i, j, visited)) return true;

    return false;
}
```

**Problems:** #79 Word Search, #212 Word Search II

---

## Backtracking Pattern 7: Palindrome Partitioning

**When to think about it:**
- "Partition string such that every part satisfies a property"
- Try every possible cut point, prune if the prefix fails the check

**Core insight:** The pruning condition is the palindrome check — only recurse
if the current segment is a palindrome.

```cpp
bool isPalin(const string& s, int l, int r) {
    while (l < r) if (s[l++] != s[r--]) return false;
    return true;
}

void partition(const string& s, int start,
               vector<string>& cur, vector<vector<string>>& res) {

    if (start == s.size()) { res.push_back(cur); return; }

    for (int end = start; end < s.size(); end++) {
        if (!isPalin(s, start, end)) continue;      // prune non-palindromes

        cur.push_back(s.substr(start, end - start + 1));   // choose
        partition(s, end + 1, cur, res);                    // explore
        cur.pop_back();                                     // unchoose
    }
}
```

**Problems:** #131 Palindrome Partitioning, #132 Palindrome Partitioning II

---

## Pruning Strategies (Critical for Performance)

| Strategy | When to Use | Code Pattern |
|---|---|---|
| Bound check | Sum exceeds target | `if (target < 0) return` |
| Sort + early exit | Skip all larger invalid items | `if (cands[i] > target) break` |
| Visited array | Prevent revisiting in same path | `visited[i] = true / false` |
| Constraint check | Board/grid problems | `isSafe()` before placing |
| Skip duplicate siblings | Sorted input with duplicates | `if (i > start && nums[i] == nums[i-1]) continue` |

---

## Backtracking: Universal Checklist

- [ ] What does one "choice" look like? (element, position, digit, direction)
- [ ] What is the base case? (target==0, index==size, all cells filled)
- [ ] What state needs to be restored? (vector, board cell, visited flag)
- [ ] What can I prune? (add the constraint check BEFORE the recursive call)
- [ ] For combinations: pass `i+1` (no reuse) or `i` (reuse allowed)?
- [ ] For permutations: use `visited[]` or swap-based approach?
- [ ] Are there duplicates in input? → sort first, skip duplicate siblings

---

## Backtracking Problem List

| # | Problem | Difficulty | Pattern |
|---|---------|-----------|---------|
| 78  | Subsets                    | Medium | Subsets              |
| 90  | Subsets II                 | Medium | Subsets + dedup      |
| 39  | Combination Sum            | Medium | Combinations (reuse) |
| 40  | Combination Sum II         | Medium | Combinations (no reuse) |
| 77  | Combinations               | Medium | Combinations (k items) |
| 216 | Combination Sum III        | Medium | Combinations (constrained) |
| 46  | Permutations               | Medium | Permutations         |
| 47  | Permutations II            | Medium | Permutations + dedup |
| 784 | Letter Case Permutation    | Medium | Permutations (chars) |
| 51  | N-Queens                   | Hard   | Constraint satisfaction |
| 52  | N-Queens II                | Hard   | Constraint (count only) |
| 37  | Sudoku Solver              | Hard   | Constraint satisfaction |
| 79  | Word Search                | Medium | Grid path            |
| 212 | Word Search II             | Hard   | Grid + Trie          |
| 131 | Palindrome Partitioning    | Medium | Partition            |

---

## Must-Do Problem List

| Priority | #   | Problem                           | Difficulty | Pattern              |
|----------|-----|-----------------------------------|------------|----------------------|
| 1        | 20  | Valid Parentheses                 | Easy       | Bracket validation   |
| 2        | 739 | Daily Temperatures                | Medium     | Monotonic stack      |
| 3        | 155 | Min Stack                         | Easy       | Auxiliary stack      |
| 4        | 496 | Next Greater Element I            | Easy       | Monotonic stack      |
| 5        | 232 | Implement Queue using Stacks      | Easy       | Two-stack            |
| 6        | 394 | Decode String                     | Medium     | Nested structure     |
| 7        | 150 | Evaluate Reverse Polish Notation  | Medium     | Expression eval      |
| 8        | 227 | Basic Calculator II               | Medium     | Expression eval      |
| 9        | 503 | Next Greater Element II           | Medium     | Monotonic + circular |
| 10       | 735 | Asteroid Collision                | Medium     | Stack simulation     |
| 11       | 901 | Online Stock Span                 | Medium     | Monotonic + state    |
| 12       | 84  | Largest Rectangle in Histogram    | Hard       | Histogram            |
| 13       | 42  | Trapping Rain Water               | Hard       | Monotonic variant    |
| 14       | 32  | Longest Valid Parentheses         | Hard       | DP + stack           |
| 15       | 85  | Maximal Rectangle                 | Hard       | Histogram reduction  |

---

## Complexity Reference

| Pattern            | Time  | Space | Key Insight                           |
|--------------------|-------|-------|---------------------------------------|
| Monotonic Stack    | O(n)  | O(n)  | Each element pushed/popped once       |
| Bracket Validation | O(n)  | O(n)  | LIFO matches nesting order            |
| Min/Max Stack      | O(1)  | O(n)  | Parallel stack tracks current min     |
| Expression Eval    | O(n)  | O(n)  | Defer +/-, resolve */ immediately     |
| Two-Stack Queue    | O(1)* | O(n)  | Amortized — each element moves twice  |
| Histogram          | O(n)  | O(n)  | Width = distance between boundaries   |
| Iterative DFS      | O(n)  | O(h)  | Explicit stack replaces call stack    |
