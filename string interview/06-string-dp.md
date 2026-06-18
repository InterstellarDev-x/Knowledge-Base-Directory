# Pattern 6: String DP

## Identify This Pattern
- "Edit distance between two strings"
- "Does pattern P match string S with wildcards/regex?"
- "Count distinct subsequences"
- "Longest common subsequence / substring"
- "Can string be split into dictionary words?"
- Any "comparison of two strings character by character" where choices compound → DP

## Core Framework
```
dp[i][j] = answer for s1[0..i-1] and s2[0..j-1]

Transition usually depends on:
  s1[i-1] == s2[j-1] → dp[i][j] = dp[i-1][j-1] + something
  s1[i-1] != s2[j-1] → dp[i][j] = min/max of dp[i-1][j], dp[i][j-1], dp[i-1][j-1] + something
```

---

## Templates

### Edit Distance (Insert / Delete / Replace)
```cpp
// dp[i][j] = min edits to convert s1[0..i-1] to s2[0..j-1]
vector<vector<int>> dp(m+1, vector<int>(n+1, 0));
for (int i = 0; i <= m; i++) dp[i][0] = i;  // delete all of s1
for (int j = 0; j <= n; j++) dp[0][j] = j;  // insert all of s2

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (s1[i-1] == s2[j-1]) dp[i][j] = dp[i-1][j-1];  // match, free
        else dp[i][j] = 1 + min({dp[i-1][j],    // delete from s1
                                  dp[i][j-1],    // insert into s1
                                  dp[i-1][j-1]}); // replace
    }
}
```

### Longest Common Subsequence (LCS)
```cpp
// dp[i][j] = LCS length of s1[0..i-1] and s2[0..j-1]
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (s1[i-1] == s2[j-1]) dp[i][j] = dp[i-1][j-1] + 1;
        else dp[i][j] = max(dp[i-1][j], dp[i][j-1]);
    }
}
```

### Wildcard Matching (? matches one, * matches any)
```cpp
// dp[i][j] = does p[0..i-1] match s[0..j-1]?
dp[0][0] = true;
for (int i = 1; i <= m; i++)
    dp[i][0] = dp[i-1][0] && p[i-1] == '*';  // * can match empty

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (p[i-1] == '*')
            dp[i][j] = dp[i-1][j] || dp[i][j-1];  // * matches empty(dp[i-1][j]) or one more(dp[i][j-1])
        else if (p[i-1] == '?' || p[i-1] == s[j-1])
            dp[i][j] = dp[i-1][j-1];
    }
}
```

### Regex Matching (. matches one, * matches zero or more of preceding)
```cpp
// dp[i][j] = does p[0..i-1] match s[0..j-1]?
dp[0][0] = true;
for (int i = 2; i <= m; i++)
    dp[i][0] = dp[i-2][0] && p[i-1] == '*';  // a* → match empty

for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        if (p[i-1] == '*') {
            dp[i][j] = dp[i-2][j] ||                              // * = zero of preceding
                       ((p[i-2] == '.' || p[i-2] == s[j-1]) && dp[i][j-1]);  // * = one more
        } else if (p[i-1] == '.' || p[i-1] == s[j-1]) {
            dp[i][j] = dp[i-1][j-1];
        }
    }
}
```

### Word Break (can split string into dictionary words?)
```cpp
// dp[i] = can s[0..i-1] be segmented?
vector<bool> dp(n+1, false);
dp[0] = true;
unordered_set<string> dict(wordDict.begin(), wordDict.end());

for (int i = 1; i <= n; i++) {
    for (int j = 0; j < i; j++) {
        if (dp[j] && dict.count(s.substr(j, i-j))) {
            dp[i] = true; break;
        }
    }
}
```

### Distinct Subsequences (count ways s contains t as subsequence)
```cpp
// dp[i][j] = ways to form t[0..j-1] from s[0..i-1]
dp[i][0] = 1  // empty t: always 1 way
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        dp[i][j] = dp[i-1][j];  // don't use s[i-1]
        if (s[i-1] == t[j-1])
            dp[i][j] += dp[i-1][j-1];  // also use s[i-1] to match t[j-1]
    }
}
```

---

## Problems

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1143](https://leetcode.com/problems/longest-common-subsequence/) | Longest Common Subsequence | `dp[i][j] = dp[i-1][j-1]+1` if match, else `max(dp[i-1][j], dp[i][j-1])` | Classic 2-string DP |
| [139](https://leetcode.com/problems/word-break/) | Word Break | `dp[i] = true` if any `dp[j] && dict.has(s[j..i])` | Try all splits ending at i |
| [516](https://leetcode.com/problems/longest-palindromic-subsequence/) | Longest Palindromic Subsequence | LCS of s and reverse(s) | Or expand DP: `dp[i][j]` = LPS of `s[i..j]` |
| [647](https://leetcode.com/problems/palindromic-substrings/) | Palindromic Substrings | DP table `isPalin[i][j]` + count | Or expand-around-center (simpler) |
| [583](https://leetcode.com/problems/delete-operation-for-two-strings/) | Delete Operation for Two Strings | `len(s1) + len(s2) - 2 * LCS(s1, s2)` | Minimum deletions = total chars minus chars in LCS |
| [97](https://leetcode.com/problems/interleaving-string/) | Interleaving String | `dp[i][j]` = can form `s3[0..i+j-1]` using `s1[0..i-1]` and `s2[0..j-1]` | `dp[i][j] = (s1[i-1]==s3[i+j-1] && dp[i-1][j]) || (s2[j-1]==s3[i+j-1] && dp[i][j-1])` |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | Classic 3-operation DP | Foundation — know the recurrence cold |
| [44](https://leetcode.com/problems/wildcard-matching/) | Wildcard Matching | `*` can match empty (dp[i-1][j]) or more (dp[i][j-1]) | Different from regex: `*` here matches any sequence alone |
| [10](https://leetcode.com/problems/regular-expression-matching/) | Regular Expression Matching | `a*` = zero a's (dp[i-2][j]) or more a's (dp[i][j-1] if match) | `*` here always refers to preceding char — different from wildcard |
| [115](https://leetcode.com/problems/distinct-subsequences/) | Distinct Subsequences | Count ways to form t from s using subsequences | `dp[i][j] = dp[i-1][j] + (s[i-1]==t[j-1] ? dp[i-1][j-1] : 0)` |
| [140](https://leetcode.com/problems/word-break-ii/) | Word Break II | Backtracking + memoization. Or DFS with memo for valid splits | Use unordered_map to cache results at each start position |

---

## Wildcard vs Regex — Key Difference

```
Wildcard (#44): * matches any sequence (including empty) independently
  * by itself → match empty/any

Regex (#10): * means zero or more of the PRECEDING char
  a* → "", "a", "aa", ...
  .* → "", any char(s)

When seeing * in regex: always look at dp[i-2][...] (skip the x* pair for zero match)
```

## Space Optimization

```
Most 2D string DPs only need previous row:
dp[i][j] depends on dp[i-1][j-1], dp[i-1][j], dp[i][j-1]
→ Can compress to 1D rolling array

Edit distance: O(min(m,n)) space possible
LCS: O(min(m,n)) space possible
```

## Common Mistakes

- Edit distance: base cases `dp[i][0] = i` and `dp[0][j] = j` — frequently forgotten
- Word Break: using `s.find` instead of hashset — O(n) per lookup vs O(k)
- Regex: forgetting `dp[0][j] = dp[0][j-2] && p[j-1]=='*'` — patterns like `a*b*c*` can match empty string
- Wildcard: confusing `dp[i-1][j]` (star matches empty) vs `dp[i][j-1]` (star matches one more)

## Follow-up Interviewers Love

- "Edit distance — what if only delete is allowed?" → `len(s1) + len(s2) - 2*LCS`
- "Can you reduce edit distance to O(n) space?" → Rolling array on the shorter string
- "Word Break II — what's the complexity?" → O(n³) worst case or O(n² * dictionary_lookup)
