# Pattern 4: Anagram & Frequency Counting

## Identify This Pattern
- "Are these two strings anagrams?"
- "Group anagrams together"
- "Find all anagram positions in s"
- "Valid permutation / rearrangement"
- "At most one char has odd frequency?"
- Character frequency as the core data structure

## Core Tools
```
int freq[26] = {}            → lowercase letters, O(1) compare
unordered_map<char,int>      → general characters
sort(s) == sort(t)           → simple anagram check but O(n log n)
```

---

## Templates

### Anagram Check (O(n))
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int freq[26] = {};
    for (char c : s) freq[c-'a']++;
    for (char c : t) { if (--freq[c-'a'] < 0) return false; }
    return true;
}
```

### Group Anagrams — Key Techniques
```cpp
// Key 1: Sort each word → same key for anagrams
unordered_map<string, vector<string>> groups;
for (string& w : words) {
    string key = w; sort(key.begin(), key.end());
    groups[key].push_back(w);
}

// Key 2: Frequency array as key (avoids sort, O(n) per word)
unordered_map<string, vector<string>> groups;
for (string& w : words) {
    int cnt[26] = {};
    for (char c : w) cnt[c-'a']++;
    string key(cnt, cnt+26);  // 26-char string as key
    groups[key].push_back(w);
}
```

### Ransom Note / Magazine Check
```cpp
bool canConstruct(string note, string magazine) {
    int freq[26] = {};
    for (char c : magazine) freq[c-'a']++;
    for (char c : note) { if (--freq[c-'a'] < 0) return false; }
    return true;
}
```

### Palindrome Permutation Check
```cpp
// A string can form a palindrome iff at most one char has odd frequency
bool canFormPalindrome(string s) {
    int freq[26] = {};
    for (char c : s) freq[c-'a']++;
    int oddCount = 0;
    for (int f : freq) oddCount += (f % 2 != 0);
    return oddCount <= 1;
}
// Using XOR bitmask (cleaner):
int mask = 0;
for (char c : s) mask ^= (1 << (c-'a'));
return __builtin_popcount(mask) <= 1;
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [242](https://leetcode.com/problems/valid-anagram/) | Valid Anagram | Frequency count both strings, compare | `int[26]` — size check first |
| [383](https://leetcode.com/problems/ransom-note/) | Ransom Note | Count magazine chars, decrement for each note char | Stop early if count goes negative |
| [409](https://leetcode.com/problems/longest-palindrome/) | Longest Palindrome | Count chars. All even pairs + 1 center if any odd | `sum += (f/2)*2; if f odd → +1 center` |
| [1002](https://leetcode.com/problems/find-common-characters/) | Find Common Characters | For each char, take min frequency across all words | Use `min` of frequency arrays |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [49](https://leetcode.com/problems/group-anagrams/) | Group Anagrams | Sort each word as key OR use freq array as key | Freq array key avoids O(k log k) sort per word |
| [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Find All Anagrams | Fixed sliding window + freq array comparison | Update window counts as you slide |
| [567](https://leetcode.com/problems/permutation-in-string/) | Permutation in String | Same as #438 but return bool | Fixed window of p.size() |
| [266](https://leetcode.com/problems/palindrome-permutation/) | Palindrome Permutation (Premium) | At most 1 odd-frequency char | XOR bitmask check |
| [954](https://leetcode.com/problems/array-of-doubled-pairs/) | Array of Doubled Pairs | Sort by abs value, greedily match x with 2x using freq map | Process from smallest abs value up |
| [1347](https://leetcode.com/problems/minimum-number-of-steps-to-make-two-anagrams/) | Min Steps to Make Two Strings Anagrams | `sum of max(0, freq[c] - have[c])` for all chars | Count excess chars in s1 vs s2 |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [884](https://leetcode.com/problems/uncommon-words-from-two-sentences/) | Uncommon Words | Count word frequency across both. Return words with count == 1 | Concat both sentences, split, count |
| [149](https://leetcode.com/problems/max-points-on-a-line/) | Max Points on a Line | For each point: count slopes to all others. Group by slope using map | `slope = dy/dx` as reduced fraction (gcd). Handle vertical separately |

---

## The Frequency Array as HashMap Key

```cpp
// Why use freq array as key instead of sort?
// sort: O(k log k) per word
// freq array key: O(k) per word

// How to create string key from int[26]:
int cnt[26] = {};
for (char c : word) cnt[c-'a']++;
string key(cnt, cnt+26);   // converts int array → string (each char = one byte)

// This works because:
// - Two anagrams always produce the same cnt[]
// - string comparison/hashing is O(26) = O(1)
```

## Common Mistakes

- `sort(s) == sort(t)` doesn't sort in-place for string args — need to assign: `string a=s; sort(a.begin(),a.end());`
- Group Anagrams: not handling empty strings (empty string should group with other empty strings)
- Ransom Note: counting note chars and decrementing from magazine (reversed) — check magazine chars are sufficient for note

## Follow-up Interviewers Love

- "Group Anagrams with millions of words?" → Freq array key + unordered_map is O(n·k) total where k = word length
- "Can you check if two strings are anagrams in O(1) space?" → Yes, `int[26]` is O(1) since alphabet is fixed size
- "What if characters aren't lowercase letters?" → Use `unordered_map<char,int>` instead of `int[26]`
