# Pattern 2: Two Pointers on Strings

## Identify This Pattern
- "Is this a palindrome?"
- "Reverse a string / reverse words"
- "Remove characters in-place"
- "Check if s is a subsequence of t"
- Two ends of string converging, or slow/fast pointer

---

## Templates

### Palindrome Check
```cpp
int l = 0, r = s.size() - 1;
while (l < r) {
    if (s[l] != s[r]) return false;
    l++; r--;
}
return true;
```

### Valid Palindrome (skip non-alphanumeric)
```cpp
int l = 0, r = s.size() - 1;
while (l < r) {
    while (l < r && !isalnum(s[l])) l++;
    while (l < r && !isalnum(s[r])) r--;
    if (tolower(s[l]) != tolower(s[r])) return false;
    l++; r--;
}
return true;
```

### Is Subsequence
```cpp
int i = 0, j = 0;
while (i < s.size() && j < t.size()) {
    if (s[i] == t[j]) i++;  // match: advance s pointer
    j++;                     // always advance t pointer
}
return i == s.size();  // all of s matched
```

### Reverse Words
```cpp
// Reverse entire string, then reverse each word
reverse(s.begin(), s.end());
int start = 0;
for (int i = 0; i <= s.size(); i++) {
    if (i == s.size() || s[i] == ' ') {
        reverse(s.begin() + start, s.begin() + i);
        start = i + 1;
    }
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [125](https://leetcode.com/problems/valid-palindrome/) | Valid Palindrome | Two pointers, skip non-alphanumeric. `isalnum()`, `tolower()` | Standard template above |
| [344](https://leetcode.com/problems/reverse-string/) | Reverse String | Swap from both ends | Most basic |
| [392](https://leetcode.com/problems/is-subsequence/) | Is Subsequence | i on s, j on t. Advance i only on match | If k follow-up queries: preprocess t with binary search |
| [345](https://leetcode.com/problems/reverse-vowels-of-a-string/) | Reverse Vowels | Two pointers, swap only when both are vowels | `string vowels = "aeiouAEIOU"` |
| [541](https://leetcode.com/problems/reverse-string-ii/) | Reverse String II | Reverse every first k chars in each 2k block | `for (int i = 0; i < n; i += 2*k) reverse(i, min(i+k, n))` |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [680](https://leetcode.com/problems/valid-palindrome-ii/) | Valid Palindrome II | Two pointers. On mismatch: try skipping s[l] OR s[r]. Check if either remainder is palindrome | Helper `isPalin(s, l, r)`. Only one skip allowed. |
| [151](https://leetcode.com/problems/reverse-words-in-a-string/) | Reverse Words in a String | Trim extra spaces, reverse whole string, reverse each word | Watch for leading/trailing/multiple spaces |
| [186](https://leetcode.com/problems/reverse-words-in-a-string-ii/) | Reverse Words II (in-place) (Premium) | Same 3-step: reverse all, reverse each word | In-place — no extra string |
| [143](https://leetcode.com/problems/reorder-list/) | Reorder List | Find mid, reverse second half, merge two halves | Two-pointer to find mid + reverse + interleave |

---

## Optimizations & Common Mistakes

**Common mistakes:**
- #125: using `isalpha` instead of `isalnum` — misses digits
- #680: trying to skip in a loop — only one skip is allowed, so just try both and OR the results
- #392 follow-up (k queries): naive O(n) per query → O(n log n) preprocessing with binary search on next-occurrence arrays

**Follow-up interviewers love:**
- "#392 — what if there are billions of queries?" → Preprocess t: for each position and each char, store next occurrence. O(n × 26) space, O(log n) per query.
- "#680 — what if you can skip up to k characters?" → DP approach needed
