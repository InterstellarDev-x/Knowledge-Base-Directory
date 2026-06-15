# Backtracking Patterns — LeetCode C++ Reference

## Backtracking vs Plain Recursion

| | Plain Recursion | Backtracking |
|---|---|---|
| **Goal** | Compute a value | Find all / any valid solutions |
| **Search space** | Defined by recurrence | Implicit tree of all candidates |
| **Pruning** | Usually none | Aggressive — cut invalid branches early |
| **State** | Parameters only | Shared state that must be restored |
| **Undo step** | Never needed | Always needed (choose -> explore -> unchoose) |

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

**The three-step heartbeat:** CHOOSE -> EXPLORE -> UNCHOOSE
Every backtracking solution follows this rhythm.

---

## Pattern 1: Subsets / Power Set

**When to think about it:**
- "Generate all subsets", "power set", "all possible selections"
- No size constraint, no target — every partial state is a valid answer
- 2^n total subsets

**Core insight:** At each index, you either include the element or skip it.
Record the current state at every **node** of the recursion tree (not just leaves).

```
nums = [1,2,3]

[]                        <- record
+-- [1]                   <- record
|   +-- [1,2]             <- record
|   |   +-- [1,2,3]      <- record
|   +-- [1,3]             <- record
+-- [2]                   <- record
|   +-- [2,3]             <- record
+-- [3]                   <- record
```

```cpp
void subsets(vector<int>& nums, int start,
             vector<int>& cur, vector<vector<int>>& res) {

    res.push_back(cur);         // record at every node, not just leaf

    for (int i = start; i < (int)nums.size(); i++) {
        cur.push_back(nums[i]);             // choose
        subsets(nums, i + 1, cur, res);     // explore
        cur.pop_back();                     // unchoose
    }
}
// Call: subsets(nums, 0, cur, res);

// Subsets II (with duplicates) — sort first, skip duplicate siblings
void subsetsII(vector<int>& nums, int start,
               vector<int>& cur, vector<vector<int>>& res) {
    res.push_back(cur);
    for (int i = start; i < (int)nums.size(); i++) {
        if (i > start && nums[i] == nums[i-1]) continue;  // skip duplicate
        cur.push_back(nums[i]);
        subsetsII(nums, i + 1, cur, res);
        cur.pop_back();
    }
}
```

**Problems:** #78 Subsets, #90 Subsets II

---

## Pattern 2: Combinations

**When to think about it:**
- "Find all combinations of size k" or "combinations that sum to target"
- Order does NOT matter (unlike permutations)
- `start` pointer prevents re-picking earlier elements

**Core insight:** Only move forward (pass `i` or `i+1` to next call), never backward.
This prevents [1,2] and [2,1] from both appearing.

```
candidates = [2,3,6,7], target = 7

[] -> try 2 -> [2]
           -> try 2 -> [2,2]
                    -> try 2 -> [2,2,2]
                             -> try 2 -> [2,2,2,2] (8 > 7, prune)
                             -> try 3 -> [2,2,3] (7 == 7) FOUND
```

```cpp
// Combination Sum — elements reusable, no duplicates in input
void combinationSum(vector<int>& cands, int target, int start,
                    vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }

    for (int i = start; i < (int)cands.size(); i++) {
        if (cands[i] > target) break;          // pruning (sort input first)

        cur.push_back(cands[i]);
        combinationSum(cands, target - cands[i], i, cur, res);  // i: reuse allowed
        cur.pop_back();
    }
}

// Combination Sum II — no reuse, input may have duplicates
void combinationSumII(vector<int>& cands, int target, int start,
                      vector<int>& cur, vector<vector<int>>& res) {
    if (target == 0) { res.push_back(cur); return; }

    for (int i = start; i < (int)cands.size(); i++) {
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
Reuse allowed   -> pass i   (same index again)
No reuse        -> pass i+1 (move to next element)
```

**Problems:** #39 Combination Sum, #40 Combination Sum II, #77 Combinations, #216 Combination Sum III

---

## Pattern 3: Permutations

**When to think about it:**
- "Find all permutations", "all orderings", "all arrangements"
- Order MATTERS — [1,2,3] and [3,2,1] are different answers
- Every element is used exactly once per permutation

**Core insight:** At each position, try every unused element.
Use either a `visited[]` array or the swap trick.

```
nums = [1,2,3]

fix 1 at pos 0 -> permute [2,3]:
    fix 2 at pos 1 -> [1,2,3] ✓
    fix 3 at pos 1 -> [1,3,2] ✓
fix 2 at pos 0 -> permute [1,3]:
    fix 1 at pos 1 -> [2,1,3] ✓
    fix 3 at pos 1 -> [2,3,1] ✓
fix 3 at pos 0 -> permute [1,2]:
    fix 1 at pos 1 -> [3,1,2] ✓
    fix 2 at pos 1 -> [3,2,1] ✓
```

```cpp
// Permutations — visited array approach
void permute(vector<int>& nums, vector<bool>& visited,
             vector<int>& cur, vector<vector<int>>& res) {

    if (cur.size() == nums.size()) { res.push_back(cur); return; }

    for (int i = 0; i < (int)nums.size(); i++) {
        if (visited[i]) continue;

        visited[i] = true;
        cur.push_back(nums[i]);             // choose
        permute(nums, visited, cur, res);   // explore
        cur.pop_back();                     // unchoose
        visited[i] = false;
    }
}

// Permutations — swap approach (modifies array in-place)
void permuteSwap(vector<int>& nums, int start, vector<vector<int>>& res) {
    if (start == (int)nums.size()) { res.push_back(nums); return; }

    for (int i = start; i < (int)nums.size(); i++) {
        swap(nums[start], nums[i]);             // choose
        permuteSwap(nums, start + 1, res);      // explore
        swap(nums[start], nums[i]);             // unchoose (swap back)
    }
}

// Permutations II — with duplicates, sort first
void permuteUnique(vector<int>& nums, vector<bool>& visited,
                   vector<int>& cur, vector<vector<int>>& res) {
    if (cur.size() == nums.size()) { res.push_back(cur); return; }

    for (int i = 0; i < (int)nums.size(); i++) {
        if (visited[i]) continue;
        // skip: same value as previous sibling that was already fully explored
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

## Pattern 4: N-Queens (Constraint Satisfaction)

**When to think about it:**
- "Place pieces on a board with no conflicts"
- Strong geometric/logical constraints that prune most branches
- Build row by row — each row must have exactly one queen

**Core insight:** Before placing, check column, left diagonal, and right diagonal.
Only recurse on safe placements — this prunes the search tree drastically.

```
Row 0: try col 0, 1, 2, ...
Row 1: for each row-0 queen, try remaining safe columns
...
When row == n: record board as a solution
```

```cpp
bool isSafe(vector<string>& board, int row, int col, int n) {
    for (int i = 0; i < row; i++)
        if (board[i][col] == 'Q') return false;            // same column

    for (int i = row-1, j = col-1; i >= 0 && j >= 0; i--, j--)
        if (board[i][j] == 'Q') return false;              // upper-left diagonal

    for (int i = row-1, j = col+1; i >= 0 && j < n; i--, j++)
        if (board[i][j] == 'Q') return false;              // upper-right diagonal

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
// Call: solveNQueens(0, n, board, res);
```

**Problems:** #51 N-Queens, #52 N-Queens II (count only)

---

## Pattern 5: Sudoku Solver

**When to think about it:**
- "Fill a grid satisfying constraints per row / column / box"
- Try digits 1–9 for each empty cell, backtrack when stuck

**Core insight:** Return `false` the moment no digit fits a cell.
This propagates the backtrack signal up to the previous decision.

```cpp
bool isValid(vector<vector<char>>& b, int r, int c, char d) {
    for (int i = 0; i < 9; i++) {
        if (b[r][i] == d) return false;                          // row
        if (b[i][c] == d) return false;                          // col
        if (b[3*(r/3)+i/3][3*(c/3)+i%3] == d) return false;    // 3x3 box
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

            return false;   // no digit worked -> backtrack to previous cell
        }
    }
    return true;    // all cells filled
}
```

**Problems:** #37 Sudoku Solver

---

## Pattern 6: Word Search / Grid Path

**When to think about it:**
- "Find a word in a 2D grid", "all paths from source to destination"
- Move in 4 (or 8) directions, cannot revisit a cell in the same path
- Mark cell visited before exploring neighbors, unmark after

**Core insight:** The `visited` flag IS the "state" being managed.
Mark it as the CHOOSE step, unmark it as the UNCHOOSE step.

```cpp
bool dfs(vector<vector<char>>& board, string& word, int idx,
         int r, int c, vector<vector<bool>>& visited) {

    if (idx == (int)word.size()) return true;   // found full word

    if (r < 0 || r >= (int)board.size() ||
        c < 0 || c >= (int)board[0].size() ||
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

## Pattern 7: Palindrome Partitioning

**When to think about it:**
- "Partition string such that every segment satisfies a property"
- Try every possible cut point, prune if the current prefix fails the check

**Core insight:** The pruning condition is the property check (palindrome here).
Only recurse into a suffix when the current segment passes the check.

```cpp
bool isPalin(const string& s, int l, int r) {
    while (l < r) if (s[l++] != s[r--]) return false;
    return true;
}

void partition(const string& s, int start,
               vector<string>& cur, vector<vector<string>>& res) {

    if (start == (int)s.size()) { res.push_back(cur); return; }

    for (int end = start; end < (int)s.size(); end++) {
        if (!isPalin(s, start, end)) continue;      // prune non-palindromes

        cur.push_back(s.substr(start, end - start + 1));    // choose
        partition(s, end + 1, cur, res);                     // explore
        cur.pop_back();                                      // unchoose
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

## Universal Checklist Before Coding

- [ ] What does one **choice** look like? (element, position, digit, direction)
- [ ] What is the **base case**? (target==0, index==size, all cells filled)
- [ ] What **state** needs to be restored? (vector, board cell, visited flag)
- [ ] What can I **prune**? (add constraint check BEFORE the recursive call)
- [ ] Combinations: pass `i+1` (no reuse) or `i` (reuse allowed)?
- [ ] Permutations: use `visited[]` or swap-based approach?
- [ ] Duplicates in input? → **sort first**, then skip duplicate siblings

---

## Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1 | 78  | Subsets                    | Medium | Subsets                    |
| 2 | 46  | Permutations               | Medium | Permutations               |
| 3 | 39  | Combination Sum            | Medium | Combinations (reuse)       |
| 4 | 90  | Subsets II                 | Medium | Subsets + dedup            |
| 5 | 40  | Combination Sum II         | Medium | Combinations (no reuse)    |
| 6 | 47  | Permutations II            | Medium | Permutations + dedup       |
| 7 | 77  | Combinations               | Medium | Combinations (k items)     |
| 8 | 216 | Combination Sum III        | Medium | Combinations (constrained) |
| 9 | 131 | Palindrome Partitioning    | Medium | Partition                  |
| 10| 79  | Word Search                | Medium | Grid path                  |
| 11| 784 | Letter Case Permutation    | Medium | Permutations (chars)       |
| 12| 51  | N-Queens                   | Hard   | Constraint satisfaction    |
| 13| 52  | N-Queens II                | Hard   | Constraint (count only)    |
| 14| 37  | Sudoku Solver              | Hard   | Constraint satisfaction    |
| 15| 212 | Word Search II             | Hard   | Grid + Trie                |

**Study order:** #78 -> #46 -> #39 -> #90 -> #40 -> #47 -> #131 -> #51
These eight cover every core concept.

---

## Complexity Reference

| Pattern                 | Time       | Space  | Key Insight                              |
|-------------------------|------------|--------|------------------------------------------|
| Subsets                 | O(2^n)     | O(n)   | Every element included or excluded       |
| Combinations            | O(2^n)     | O(n)   | Only move forward, no backtracking on i  |
| Permutations            | O(n!)      | O(n)   | Each position tries all unused elements  |
| N-Queens                | O(N!)      | O(N)   | Pruning makes it practical               |
| Sudoku                  | O(9^cells) | O(81)  | Heavy pruning drastically cuts space     |
| Word Search             | O(N * 4^L) | O(N)   | N = cells, L = word length               |
| Palindrome Partition    | O(2^n)     | O(n)   | Every cut point tried, pruned by check   |
