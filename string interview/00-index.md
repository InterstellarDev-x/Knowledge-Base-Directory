# String Interview Master Index

## Pattern Files

| File | Pattern | Difficulty | Most Asked At |
|------|---------|-----------|--------------|
| [01-sliding-window-strings.md](01-sliding-window-strings.md) | Sliding Window on Strings | Medium–Hard | All companies |
| [02-two-pointers-strings.md](02-two-pointers-strings.md) | Two Pointers on Strings | Easy–Medium | Meta, Amazon |
| [03-palindrome.md](03-palindrome.md) | Palindrome Patterns | Easy–Hard | Amazon, Meta, Google |
| [04-anagram-frequency.md](04-anagram-frequency.md) | Anagram / Frequency Counting | Easy–Medium | All companies |
| [05-string-algorithms.md](05-string-algorithms.md) | KMP, Z-Algorithm, Rabin-Karp | Medium–Hard | Google, Amazon |
| [06-string-dp.md](06-string-dp.md) | String DP (Edit Distance, Wildcard, Regex) | Hard | Google, Meta, Amazon |
| [07-parsing-simulation.md](07-parsing-simulation.md) | Parsing & Simulation | Medium | Bloomberg, Amazon |
| [08-company-questions.md](08-company-questions.md) | Company-Specific Lists (2024–2025) | All | All |

---

## Pattern Decision Tree

```
Longest/shortest substring with constraint?     → Sliding Window
Reverse / validate / compare chars?             → Two Pointers
Is it a palindrome / find palindromic sub?      → Expand Around Center or DP
Anagram check / find anagram positions?         → Frequency array + Sliding Window
Find pattern in text efficiently?               → KMP / Z-Algorithm
Edit distance / match with wildcards?           → String DP
Parse expression / decode string?               → Stack or Recursion
```

## Must-Do 15

```
#3    Longest Substring Without Repeating  → Sliding Window
#76   Minimum Window Substring             → Sliding Window
#438  Find All Anagrams in String          → Sliding Window + Freq
#567  Permutation in String                → Sliding Window + Freq
#5    Longest Palindromic Substring        → Expand Around Center
#125  Valid Palindrome                     → Two Pointers
#49   Group Anagrams                       → Freq key / sort key
#242  Valid Anagram                        → Frequency count
#424  Longest Repeating Char Replacement   → Sliding Window
#20   Valid Parentheses                    → Stack
#394  Decode String                        → Stack / Recursion
#72   Edit Distance                        → String DP
#44   Wildcard Matching                    → String DP / Greedy
#10   Regular Expression Matching          → String DP
#28   Find Index of First Occurrence       → KMP
```
