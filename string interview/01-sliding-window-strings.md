# Pattern 1: Sliding Window on Strings

## Identify This Pattern
- "Longest substring with at most k distinct characters"
- "Minimum window containing all of t's characters"
- "Find all anagram positions in s"
- "Permutation of p exists as substring of s?"
- Key: substring is **contiguous**, and you're optimizing its **length** or **count**

## Same as array sliding window — just the "state" is character frequencies

---

## Templates

### Fixed Window (anagram / permutation check)
```cpp
// Check if any window of size p.size() matches p's frequency
vector<int> need(26,0), have(26,0);
for (char c : p) need[c-'a']++;
int k = p.size();
for (int i = 0; i < s.size(); i++) {
    have[s[i]-'a']++;
    if (i >= k) have[s[i-k]-'a']--;
    if (have == need) /* found anagram at i-k+1 */;
}
```

### Variable Window — Minimum (shrink while valid)
```cpp
// Minimum window substring
unordered_map<char,int> need, have;
for (char c : t) need[c]++;
int formed = 0, required = need.size();
int left = 0, minLen = INT_MAX, start = 0;
for (int right = 0; right < s.size(); right++) {
    have[s[right]]++;
    if (need.count(s[right]) && have[s[right]] == need[s[right]]) formed++;
    while (formed == required) {
        if (right - left + 1 < minLen) { minLen = right - left + 1; start = left; }
        char c = s[left++];
        if (need.count(c) && have[c] == need[c]) formed--;
        have[c]--;
    }
}
```

### Variable Window — Maximum (expand until invalid)
```cpp
// Longest substring with at most k distinct chars
unordered_map<char,int> freq;
int left = 0, maxLen = 0;
for (int right = 0; right < s.size(); right++) {
    freq[s[right]]++;
    while ((int)freq.size() > k) {
        freq[s[left]]--;
        if (freq[s[left]] == 0) freq.erase(s[left]);
        left++;
    }
    maxLen = max(maxLen, right - left + 1);
}
```

---

## Problems

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Longest Substring Without Repeating | Map char → last seen index. Jump left on dup | `left = max(left, lastSeen[c]+1)` |
| [567](https://leetcode.com/problems/permutation-in-string/) | Permutation in String | Fixed window of p.size(), compare freq arrays | `int[26]` comparison is O(26) = O(1) |
| [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Find All Anagrams in String | Same as #567, collect all valid left positions | Push `i - k + 1` when arrays match |
| [424](https://leetcode.com/problems/longest-repeating-character-replacement/) | Longest Repeating Char Replacement | `windowLen - maxFreq <= k` → valid | Track max freq of any char; don't decrease maxFreq |
| [340](https://leetcode.com/problems/longest-substring-with-at-most-k-distinct-characters/) | Longest Substring K Distinct (Premium) | Variable window: shrink when distinct > k | Use freq map, erase when count hits 0 |
| [1100](https://leetcode.com/problems/find-k-length-substrings-with-no-repeated-characters/) | K-Length Substrings No Repeat (Premium) | Fixed window of size k, track char set | — |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | Variable window. Two maps + `formed` counter | Only update `formed` when a char is exactly satisfied (not over-counted) |
| [30](https://leetcode.com/problems/substring-with-concatenation-of-all-words/) | Substring With Concatenation of All Words | Fixed window of `len(words) * wordLen`. Slide by word units | Outer loop over starting positions 0..wordLen-1 |

---

## Key Insight: `formed` Counter

```
Instead of comparing two hashmaps on every step (expensive):

formed = count of chars where have[c] >= need[c]
required = need.size() (distinct chars needed)

Add to formed when have[c] BECOMES EQUAL to need[c] (not every increment)
Remove from formed when have[c] DROPS BELOW need[c] (not every decrement)

When formed == required → window is valid
```

## Optimizations & Common Mistakes

**Common mistakes:**
- Comparing entire `unordered_map` objects is O(k), not O(1). Use `int[26]` for lowercase letters.
- For #3: not bounding `left` by `max(left, ...)` allows it to move backward when duplicate was before current window
- For #76: updating `formed` on every character change instead of only at the threshold

**Optimizations:**
- `int freq[26]` → array comparison with `==` works in C++ for `vector<int>` — use `vector<int>(26)`
- For #76 with large alphabets: store only chars that appear in t in the map

**Follow-ups interviewers love:**
- "#76 — what if s and t contain Unicode?" → Use `unordered_map<char,int>` instead of `int[26]`
- "#438 — why fixed window?" → All anagrams have the same length as p
- "What's the difference between #567 and #438?" → #567 returns bool, #438 returns all positions
