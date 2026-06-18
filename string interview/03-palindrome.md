# Pattern 3: Palindrome Problems

## Identify This Pattern
- "Is this string a palindrome?"
- "Longest palindromic substring"
- "Count palindromic substrings"
- "Minimum cuts to partition into palindromes"
- "Can we form a palindrome with these characters?"

## Two Approaches
```
Expand Around Center: find ALL palindromes in O(n²). Simple. No extra space.
DP:                   build 2D table. O(n²) time + space. Good for counting/partitioning.
Manacher's:           find ALL palindromes in O(n). Used for max length optimally.
```

---

## Templates

### Expand Around Center (O(n²) time, O(1) space)
```cpp
// Returns [start, maxLen] of the longest palindrome centered at (l, r)
pair<int,int> expand(string& s, int l, int r) {
    while (l >= 0 && r < s.size() && s[l] == s[r]) { l--; r++; }
    return {l+1, r-l-1};  // start index, length
}

// Check both odd (center at i) and even (center between i and i+1)
string longestPalindrome(string s) {
    int start = 0, maxLen = 1;
    for (int i = 0; i < s.size(); i++) {
        auto [l1, len1] = expand(s, i, i);     // odd length
        auto [l2, len2] = expand(s, i, i+1);   // even length
        if (len1 > maxLen) { start = l1; maxLen = len1; }
        if (len2 > maxLen) { start = l2; maxLen = len2; }
    }
    return s.substr(start, maxLen);
}
```

### Count Palindromic Substrings (expand around each center)
```cpp
int countSubstrings(string s) {
    int count = 0;
    for (int i = 0; i < s.size(); i++) {
        // Odd
        for (int l=i, r=i; l>=0 && r<s.size() && s[l]==s[r]; l--,r++) count++;
        // Even
        for (int l=i, r=i+1; l>=0 && r<s.size() && s[l]==s[r]; l--,r++) count++;
    }
    return count;
}
```

### DP — Is substring [i..j] a palindrome?
```cpp
// dp[i][j] = true if s[i..j] is palindrome
vector<vector<bool>> dp(n, vector<bool>(n, false));
for (int i = 0; i < n; i++) dp[i][i] = true;   // single chars
for (int i = 0; i < n-1; i++) dp[i][i+1] = (s[i] == s[i+1]);

for (int len = 3; len <= n; len++) {
    for (int i = 0; i <= n-len; i++) {
        int j = i + len - 1;
        dp[i][j] = (s[i] == s[j]) && dp[i+1][j-1];
    }
}
```

### Manacher's Algorithm (O(n) — all palindromic radii)
```cpp
// Transform: "abc" → "#a#b#c#" (handles odd/even uniformly)
// p[i] = palindrome radius at position i in transformed string
string t = "#";
for (char c : s) { t += c; t += '#'; }
int n = t.size();
vector<int> p(n, 0);
int center = 0, right = 0;
for (int i = 0; i < n; i++) {
    if (i < right) p[i] = min(right - i, p[2*center - i]);
    while (i-p[i]-1 >= 0 && i+p[i]+1 < n && t[i-p[i]-1] == t[i+p[i]+1])
        p[i]++;
    if (i + p[i] > right) { center = i; right = i + p[i]; }
}
// p[i] in transformed = radius. Actual length in original = p[i].
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [125](https://leetcode.com/problems/valid-palindrome/) | Valid Palindrome | Two pointers, skip non-alnum | `isalnum()`, `tolower()` |
| [680](https://leetcode.com/problems/valid-palindrome-ii/) | Valid Palindrome II | Try skipping s[l] or s[r] on first mismatch | Helper `isPalin(s,l,r)`. Only one skip. |
| [9](https://leetcode.com/problems/palindrome-number/) | Palindrome Number | Reverse half the number, compare | Or convert to string |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [5](https://leetcode.com/problems/longest-palindromic-substring/) | Longest Palindromic Substring | Expand around center (odd + even). Track max | Don't use DP — expand is simpler and same complexity |
| [647](https://leetcode.com/problems/palindromic-substrings/) | Palindromic Substrings | Expand around each center, count each expansion | Both odd and even centers |
| [266](https://leetcode.com/problems/palindrome-permutation/) | Palindrome Permutation (Premium) | At most one char can have odd frequency | Use XOR or frequency count |
| [409](https://leetcode.com/problems/longest-palindrome/) | Longest Palindrome (build from chars) | All even-frequency chars + one odd-frequency char | Count chars; all-even can be used fully; add 1 if any odd |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [132](https://leetcode.com/problems/palindrome-partitioning-ii/) | Palindrome Partitioning II (min cuts) | DP: `cut[i]` = min cuts for s[0..i]. Precompute `isPalin[i][j]` | `cut[i] = min(cut[j-1]+1)` for all j where s[j..i] is palindrome |
| [131](https://leetcode.com/problems/palindrome-partitioning/) | Palindrome Partitioning | Backtracking + precomputed `isPalin[i][j]` | Build result array incrementally |
| [214](https://leetcode.com/problems/shortest-palindrome/) | Shortest Palindrome | Find longest palindromic prefix. KMP on `s + "#" + reverse(s)` | The answer = reverse(s[lps_len:]) + s |
| [336](https://leetcode.com/problems/palindrome-pairs/) | Palindrome Pairs | Trie or hashmap. For each word: check if complement makes palindrome | Check prefix and suffix palindromes |

---

## Manacher's vs Expand: When to Use Which

```
Interview (usually):    Expand around center — O(n²), easy to code, easy to explain
Needs O(n):             Manacher's — mention it, but expand is acceptable in most interviews
Counting + DP needed:   Use DP table — partition/cut problems need it
```

## Common Mistakes

- Expand: not handling even-length palindromes (only checking `s[i], i, i` misses `"abba"` type)
- Valid Palindrome: using `isalpha` instead of `isalnum` — ignores digits
- Longest palindrome from chars (#409): can always add 1 (center char) if any odd-frequency char exists

## Follow-up Interviewers Love

- "#5 — can you do O(n)?" → Yes, Manacher's algorithm
- "#647 — difference from #5?" → #5 finds the longest; #647 counts all
- "What's the palindrome permutation condition?" → At most 1 character has odd frequency
