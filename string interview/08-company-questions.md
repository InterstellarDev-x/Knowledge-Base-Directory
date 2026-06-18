# Company-Specific String Questions

> Data source: hxu296/leetcode-company-wise-problems-2022 (real occurrence counts from interviews)
> Format: Problem — frequency (times appeared in interviews)

---

## Amazon

| # | Problem | Freq | Pattern |
|---|---------|------|---------|
| [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | 51 | Anagram/Frequency |
| [20](https://leetcode.com/problems/valid-parentheses/) | Valid Parentheses | 43 | Stack/Parsing |
| [5](https://leetcode.com/problems/longest-palindromic-substring/) | Longest Palindromic Substring | 35 | Palindrome (Expand) |
| [139](https://leetcode.com/problems/word-break/) | Word Break | 34 | String DP |
| [13](https://leetcode.com/problems/roman-to-integer/) | Roman to Integer | 27 | Parsing/Simulation |
| [394](https://leetcode.com/problems/decode-string/) | Decode String | 12 | Parsing (Stack) |
| [140](https://leetcode.com/problems/word-break-ii/) | Word Break II | 11 | String DP + Backtrack |
| [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | 11 | Sliding Window |
| [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | 9 | Parsing/Simulation |
| [10](https://leetcode.com/problems/regular-expression-matching/) | Regular Expression Matching | 8 | String DP |
| [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | 8 | String DP |
| [28](https://leetcode.com/problems/find-the-index-of-the-first-occurrence-in-a-string/) | Implement strStr() | 7 | KMP |
| [516](https://leetcode.com/problems/longest-palindromic-subsequence/) | Longest Palindromic Subsequence | 6 | String DP |
| [1143](https://leetcode.com/problems/longest-common-subsequence/) | Longest Common Subsequence | 6 | String DP |

**Amazon focus**: String DP is very prominent (`Word Break`, `Edit Distance`, `Regex`). Also `Group Anagrams` (a warm-up but asked a lot).

---

## Meta (Facebook)

| # | Problem | Freq | Pattern |
|---|---------|------|---------|
| [1249](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/) | Minimum Remove to Make Valid Parentheses | 282 | Stack/Parsing |
| [680](https://leetcode.com/problems/valid-palindrome-ii/) | Valid Palindrome II | 252 | Two Pointers/Palindrome |
| [125](https://leetcode.com/problems/valid-palindrome/) | Valid Palindrome | 94 | Two Pointers |
| [139](https://leetcode.com/problems/word-break/) | Word Break | 33 | String DP |
| [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | 32 | Parsing/Simulation |
| [20](https://leetcode.com/problems/valid-parentheses/) | Valid Parentheses | 24 | Stack/Parsing |
| [140](https://leetcode.com/problems/word-break-ii/) | Word Break II | 22 | String DP + Backtrack |
| [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | 20 | Sliding Window |
| [10](https://leetcode.com/problems/regular-expression-matching/) | Regular Expression Matching | 19 | String DP |
| [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | 17 | Anagram/Frequency |
| [267](https://leetcode.com/problems/palindrome-permutation/) | Palindrome Permutation | 12 | Anagram/Frequency |
| [394](https://leetcode.com/problems/decode-string/) | Decode String | 11 | Parsing (Stack) |

**Meta focus**: Palindrome problems dominate (#680 freq=252 is the highest string frequency at any company!). `Minimum Remove to Make Valid Parentheses` (282) is Meta's signature question.

---

## Google

| # | Problem | Freq | Pattern |
|---|---------|------|---------|
| [394](https://leetcode.com/problems/decode-string/) | Decode String | 19 | Parsing (Stack) |
| [5](https://leetcode.com/problems/longest-palindromic-substring/) | Longest Palindromic Substring | 12 | Palindrome (Expand) |
| [9](https://leetcode.com/problems/palindrome-number/) | Palindrome Number | 11 | Palindrome |
| [13](https://leetcode.com/problems/roman-to-integer/) | Roman to Integer | 10 | Parsing/Simulation |
| [10](https://leetcode.com/problems/regular-expression-matching/) | Regular Expression Matching | 9 | String DP |
| [20](https://leetcode.com/problems/valid-parentheses/) | Valid Parentheses | 6 | Stack/Parsing |
| [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | 6 | Parsing/Simulation |
| [44](https://leetcode.com/problems/wildcard-matching/) | Wildcard Matching | 4 | String DP |
| [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | 4 | String DP |
| [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | 5 | Anagram/Frequency |
| [459](https://leetcode.com/problems/repeated-substring-pattern/) | Repeated Substring Pattern | 2 | KMP |
| [151](https://leetcode.com/problems/reverse-words-in-a-string/) | Reverse Words in a String | 2 | Two Pointers |

**Google focus**: `Decode String` is Google's #1 string question. Parsing + simulation + DP. More complex problems expected.

---

## Microsoft

| # | Problem | Freq | Pattern |
|---|---------|------|---------|
| [3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Longest Substring Without Repeating Characters | 29 | Sliding Window |
| [5](https://leetcode.com/problems/longest-palindromic-substring/) | Longest Palindromic Substring | ~15 | Palindrome |
| [20](https://leetcode.com/problems/valid-parentheses/) | Valid Parentheses | ~12 | Stack/Parsing |
| [139](https://leetcode.com/problems/word-break/) | Word Break | ~10 | String DP |
| [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | ~8 | Anagram |
| [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | ~8 | Sliding Window |
| [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | ~7 | Parsing |
| [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | ~6 | String DP |
| [242](https://leetcode.com/problems/valid-anagram/) | Valid Anagram | ~5 | Anagram |

---

## Universal Must-Do (Top 15 String Problems)

| Priority | # | Problem | Why |
|----------|---|---------|-----|
| ★★★ | [125](https://leetcode.com/problems/valid-palindrome/) | Valid Palindrome | Foundation for all palindrome questions |
| ★★★ | [680](https://leetcode.com/problems/valid-palindrome-ii/) | Valid Palindrome II | Meta's #1 string question (freq=252!) |
| ★★★ | [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | Amazon #1 string, asked everywhere |
| ★★★ | [5](https://leetcode.com/problems/longest-palindromic-substring/) | Longest Palindromic Substring | Amazon #3, Google #2 |
| ★★★ | [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | Hard sliding window — differentiator |
| ★★★ | [1249](https://leetcode.com/problems/minimum-remove-to-make-valid-parentheses/) | Minimum Remove to Make Valid Parentheses | Meta's #1 overall string (freq=282) |
| ★★★ | [139](https://leetcode.com/problems/word-break/) | Word Break | String DP — Amazon, Meta, Google, Microsoft |
| ★★★ | [394](https://leetcode.com/problems/decode-string/) | Decode String | Google's #1 string question |
| ★★★ | [20](https://leetcode.com/problems/valid-parentheses/) | Valid Parentheses | Stack-based parsing — appears everywhere |
| ★★☆ | [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | Meta (32), Amazon (9), Google (6) |
| ★★☆ | [10](https://leetcode.com/problems/regular-expression-matching/) | Regular Expression Matching | Amazon (8), Meta (19), Google (9) |
| ★★☆ | [72](https://leetcode.com/problems/edit-distance/) | Edit Distance | Amazon (8), Google (4), Microsoft common |
| ★★☆ | [3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Longest Substring Without Repeating Characters | Microsoft #1 string (29) |
| ★★☆ | [13](https://leetcode.com/problems/roman-to-integer/) | Roman to Integer | Amazon (27), Google (10) |
| ★★☆ | [140](https://leetcode.com/problems/word-break-ii/) | Word Break II | Amazon (11), Meta (22) |

---

## 2024–2025 Trends

- **Meta palindromes**: `Valid Palindrome II` (252) and `Minimum Remove to Make Valid Parentheses` (282) are Meta's two most frequently asked questions across ALL categories — not just strings.
- **Parsing questions growing**: `Decode String` at Google (19 freq), `atoi` at multiple companies. Stack-based parsing is expected.
- **String DP expected at senior level**: `Word Break`, `Edit Distance`, `Regex Matching` appear at Amazon, Meta, Google — especially for SDE-2/senior roles.
- **KMP less commonly asked directly**: `strStr` (#28) appears at Amazon (7) but generally companies test pattern matching via application questions like `Decode String` or `Shortest Palindrome` rather than asking you to implement KMP directly.
- **Anagram/frequency counting is warm-up**: `Group Anagrams` (51 at Amazon) but usually tested as a first round or phone screen, not the hard round.

## Company-Specific Preparation Tips

**Amazon**: Deep-dive String DP — `Word Break` + `Word Break II` + `Edit Distance` + `Regex`. Also prepare `Group Anagrams` as a warm-up.

**Meta**: The two landmark questions: `Minimum Remove to Make Valid Parentheses` (282) and `Valid Palindrome II` (252). Know all palindrome variants. Also `atoi` and `Word Break`.

**Google**: Focus on `Decode String` (stack-based nested parsing). Expect regex/wildcard DP and harder parsing problems like `Text Justification`.

**Microsoft**: Sliding window strings are the entry point. Then String DP. `Valid Parentheses` and basic stack problems expected.
