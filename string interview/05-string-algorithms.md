# Pattern 5: String Matching Algorithms

## Covered
1. KMP (Knuth-Morris-Pratt)
2. Z-Algorithm
3. Rabin-Karp (Rolling Hash)
4. Boyer-Moore (conceptual)

## When to Use
```
Find pattern P in text T:
  Naive:        O(n*m) — mention it, then improve
  KMP:          O(n+m) — use when pattern has repeated prefixes
  Z-Algorithm:  O(n+m) — alternative to KMP, often cleaner
  Rabin-Karp:   O(n+m) avg — good when matching multiple patterns
  Boyer-Moore:  O(n/m) best — faster in practice, harder to code
```

---

## 1. KMP Algorithm

### Core Idea
Build a "failure function" (LPS array) for the pattern.
LPS[i] = length of longest proper prefix of `pattern[0..i]` that is also a suffix.
When mismatch occurs, use LPS to skip ahead instead of starting over.

### Build LPS (Failure Function)
```cpp
vector<int> buildLPS(string pat) {
    int n = pat.size();
    vector<int> lps(n, 0);
    int len = 0, i = 1;
    while (i < n) {
        if (pat[i] == pat[len]) {
            lps[i++] = ++len;
        } else if (len > 0) {
            len = lps[len-1];   // fall back, don't increment i
        } else {
            lps[i++] = 0;
        }
    }
    return lps;
}
```

### KMP Search
```cpp
// Returns all start indices of pat in text
vector<int> kmpSearch(string text, string pat) {
    vector<int> lps = buildLPS(pat);
    vector<int> matches;
    int i = 0, j = 0;   // i = text pointer, j = pattern pointer
    while (i < text.size()) {
        if (text[i] == pat[j]) { i++; j++; }
        if (j == pat.size()) {
            matches.push_back(i - j);  // full match
            j = lps[j-1];              // continue searching
        } else if (i < text.size() && text[i] != pat[j]) {
            if (j != 0) j = lps[j-1];
            else i++;
        }
    }
    return matches;
}
```

### Key Interview Use
```cpp
// Check if s is a rotation of t:
// s is rotation of t iff s is a substring of t+t
// Use KMP to find s in t+t in O(n)
bool isRotation(string s, string t) {
    if (s.size() != t.size()) return false;
    return kmpSearch(t + t, s).size() > 0;
}

// Repeated substring pattern (#459):
// If s = k copies of some substring, then s[1:] + s[:-1] contains s
// Use: lps[n-1] > 0 && n % (n - lps[n-1]) == 0
```

---

## 2. Z-Algorithm

### Core Idea
Z[i] = length of the longest substring starting at i that is also a prefix of the entire string.

### Template
```cpp
vector<int> zFunction(string s) {
    int n = s.size();
    vector<int> z(n, 0);
    z[0] = n;
    int l = 0, r = 0;
    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = min(r - i, z[i - l]);
        while (i + z[i] < n && s[z[i]] == s[i + z[i]]) z[i]++;
        if (i + z[i] > r) { l = i; r = i + z[i]; }
    }
    return z;
}

// Pattern matching: concatenate pat + "#" + text
// Z values of positions in text part that equal pat.size() → match found
vector<int> zSearch(string text, string pat) {
    string concat = pat + "#" + text;
    vector<int> z = zFunction(concat);
    vector<int> matches;
    for (int i = pat.size() + 1; i < concat.size(); i++)
        if (z[i] == pat.size()) matches.push_back(i - pat.size() - 1);
    return matches;
}
```

---

## 3. Rabin-Karp (Rolling Hash)

### Core Idea
Compute hash of pattern. Slide a window of same size over text.
Compare hashes first — only compare strings when hashes match (avoids O(m) per position).

### Template
```cpp
int rabinKarp(string text, string pat) {
    int n = text.size(), m = pat.size();
    if (n < m) return -1;

    long long MOD = 1e9 + 7, BASE = 31;
    long long patHash = 0, winHash = 0, power = 1;

    for (int i = 0; i < m; i++) {
        patHash = (patHash * BASE + pat[i]) % MOD;
        winHash = (winHash * BASE + text[i]) % MOD;
        if (i > 0) power = power * BASE % MOD;
    }

    for (int i = 0; i <= n - m; i++) {
        if (winHash == patHash && text.substr(i, m) == pat)
            return i;  // verify on hash match (handle collisions)
        if (i < n - m) {
            winHash = (winHash - text[i] * power % MOD + MOD) % MOD;
            winHash = (winHash * BASE + text[i + m]) % MOD;
        }
    }
    return -1;
}
```

### Best Use: Multiple Pattern Matching
```
Rabin-Karp is best when searching for multiple patterns simultaneously.
Hash all patterns into a set. Slide one window; check hash against set.
O(n + sum(m_i)) instead of O(n * sum(m_i)).
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Find Index of First Occurrence | KMP: O(n+m). Or just use `s.find()` in practice | Interviewers expect KMP or Z here |
| [459](https://leetcode.com/problems/repeated-substring-pattern/) | Repeated Substring Pattern | KMP: `lps[n-1] > 0 && n % (n - lps[n-1]) == 0` | Or check if s is in `(s+s)[1:-1]` |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [1392](https://leetcode.com/problems/longest-happy-prefix/) | Longest Happy Prefix | Build LPS array for the string itself. Answer = lps[n-1] | KMP failure function applied to single string |
| [796](https://leetcode.com/problems/rotate-string/) | Rotate String | Check if s is substring of goal+goal (rotation check) | KMP or just `(goal+goal).find(s) != npos` |
| [686](https://leetcode.com/problems/repeated-string-match/) | Repeated String Match | Repeat a until length ≥ b. Check with KMP | Minimum repeats = ceil(b.size() / a.size()), try that and +1 |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [214](https://leetcode.com/problems/shortest-palindrome/) | Shortest Palindrome | Find longest palindromic prefix using KMP on `s + "#" + reverse(s)` | `lps` of combined string tells us the longest prefix that's also a suffix of reversed |
| [1397](https://leetcode.com/problems/find-all-good-strings/) | Find All Good Strings | KMP automaton + DP | Very hard — KMP failure function as DP state |

---

## KMP vs Z-Algorithm

```
Both O(n+m). Same power.
KMP:  more commonly taught, LPS array is well-known
Z:    often cleaner to implement for "does pattern appear in text" problems
      Z-search pattern: concat pat + "#" + text, check Z[i] == pat.size()

In interviews: know one well. KMP is more often expected.
```

## Common Mistakes

**KMP LPS build:**
- Forgetting to fall back to `lps[len-1]` when mismatch and `len > 0` (not just `len--`)
- Starting `i = 0` instead of `i = 1` (lps[0] is always 0)

**Z-function:**
- Initializing `z[0] = n` manually (can't be computed by the loop)
- Off-by-one in bounds check: `i + z[i] < n`

**Rabin-Karp:**
- Forgetting to verify actual strings on hash match (hash collisions)
- Negative hash after subtraction in rolling — always add MOD before taking mod

## Follow-up Interviewers Love

- "Why KMP over naive?" → O(n+m) vs O(n*m) — no backtracking in text pointer
- "What does LPS[i] mean?" → Longest proper prefix of `pat[0..i]` that is also a suffix
- "When would you use Rabin-Karp over KMP?" → Multiple pattern search — hash all patterns, one pass
