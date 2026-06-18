# Pattern 1: Sliding Window

## Identify This Pattern
- "Longest / shortest subarray/substring with condition X"
- "Maximum sum of subarray of size k"
- "Minimum window containing all characters"
- Window = a contiguous range [left, right] that slides through the array
- Key signal: **contiguous** + **optimize length or value**

## Two Types
```
Fixed window:    size k never changes. Slide right, drop left.
Variable window: expand right until invalid, shrink left until valid.
                 → LONGEST: expand freely, shrink when violated
                 → SHORTEST: expand until satisfied, shrink as much as possible
```

---

## Templates

### Fixed Window (size k)
```cpp
// Add nums[right], remove nums[right - k] when window is full
for (int i = 0; i < n; i++) {
    // add nums[i] to window state
    if (i >= k) {
        // remove nums[i - k] from window state
    }
    if (i >= k - 1) {
        ans = max(ans, /* window value */);
    }
}
```

### Variable Window — Longest valid
```cpp
int left = 0;
for (int right = 0; right < n; right++) {
    // add nums[right] to window
    while (/* window is invalid */) {
        // remove nums[left] from window
        left++;
    }
    ans = max(ans, right - left + 1);
}
```

### Variable Window — Shortest valid
```cpp
int left = 0;
for (int right = 0; right < n; right++) {
    // add nums[right] to window
    while (/* window is valid — try to shrink */) {
        ans = min(ans, right - left + 1);
        // remove nums[left] from window
        left++;
    }
}
```

### Window with Frequency Map (char counting)
```cpp
unordered_map<char,int> window;
int left = 0, formed = 0;
int required = need.size();  // distinct chars needed
for (int right = 0; right < s.size(); right++) {
    window[s[right]]++;
    if (need.count(s[right]) && window[s[right]] == need[s[right]])
        formed++;
    while (formed == required) {
        // update answer
        char c = s[left++];
        if (need.count(c) && window[c] == need[c]) formed--;
        window[c]--;
    }
}
```

### Monotonic Deque Window (max/min in sliding window)
```cpp
// Deque stores INDICES. Front = max element of current window.
deque<int> dq;
for (int i = 0; i < n; i++) {
    // Remove elements outside window
    while (!dq.empty() && dq.front() < i - k + 1) dq.pop_front();
    // Maintain decreasing order (for max)
    while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back();
    dq.push_back(i);
    if (i >= k - 1) result.push_back(nums[dq.front()]);
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [643](https://leetcode.com/problems/maximum-average-subarray-i/) | Max Average Subarray I | Fixed window of size k, track sum | Divide by k at each step or at end |
| [1480](https://leetcode.com/problems/running-sum-of-1d-array/) | Running Sum of 1D Array | Prefix sum | Not sliding window but related |
| [1004](https://leetcode.com/problems/max-consecutive-ones-iii/) | Max Consecutive Ones III | Variable window: track count of 0s flipped | Shrink when flipped > k |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [3](https://leetcode.com/problems/longest-substring-without-repeating-characters/) | Longest Substring Without Repeating | Variable window. Map char → last index. Jump left on duplicate | `left = max(left, lastSeen[c]+1)` avoids stale duplicates |
| [209](https://leetcode.com/problems/minimum-size-subarray-sum/) | Minimum Size Subarray Sum | Shortest valid window with sum ≥ target. Shrink while valid | Start with left=right=0, shrink inner while loop |
| [424](https://leetcode.com/problems/longest-repeating-character-replacement/) | Longest Repeating Character Replacement | `windowLen - maxFreq <= k` means at most k replacements needed | Don't shrink maxFreq when left moves — only need to track max seen |
| [567](https://leetcode.com/problems/permutation-in-string/) | Permutation in String | Fixed window of size `p.size()`. Compare freq arrays | Use int[26] instead of map for O(1) comparison |
| [438](https://leetcode.com/problems/find-all-anagrams-in-a-string/) | Find All Anagrams in a String | Same as #567, collect all positions | Fixed window + freq comparison |
| [1208](https://leetcode.com/problems/get-equal-substrings-within-budget/) | Get Equal Substrings Within Budget | Variable window: sum of abs(s[i]-t[i]) ≤ maxCost | Convert to array of costs, then standard variable window |
| [930](https://leetcode.com/problems/binary-subarrays-with-sum/) | Binary Subarrays With Sum | Count subarrays with sum == goal = subarrays(≤goal) - subarrays(≤goal-1) | "At most k" sliding window trick |
| [992](https://leetcode.com/problems/subarrays-with-k-different-integers/) | Subarrays with K Different Integers | Same trick: exact(k) = atMost(k) - atMost(k-1) | Can't do exact k directly with sliding window |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [76](https://leetcode.com/problems/minimum-window-substring/) | Minimum Window Substring | Variable window with freq map. Track `formed` count | When `formed == required`: record answer, then shrink |
| [239](https://leetcode.com/problems/sliding-window-maximum/) | Sliding Window Maximum | Monotonic decreasing deque of indices | Pop back when new element is larger, pop front when out of window |
| [480](https://leetcode.com/problems/sliding-window-median/) | Sliding Window Median | Two heaps (max-heap for left half, min-heap for right half) | Maintain balance between heaps as window slides |

---

## The "Exact K → At Most K" Trick

```
Problem: count subarrays with EXACTLY k distinct/sum
Sliding window naturally computes "at most k" (stop expanding when > k)

Solution: exact(k) = atMost(k) - atMost(k-1)

int atMost(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    int left = 0, res = 0;
    for (int right = 0; right < nums.size(); right++) {
        if (++freq[nums[right]] == 1) k--;   // new distinct element
        while (k < 0) {
            if (--freq[nums[left]] == 0) k++;
            left++;
        }
        res += right - left + 1;  // all subarrays ending at right
    }
    return res;
}
// Applies to: #930, #992, #1248
```

---

## Optimizations & Common Mistakes

**Common mistakes:**
- For #3: using `last = map[c]` without checking if it's ≥ left — stale entries pull left backwards
- For #424: don't decrement `maxFreq` when shrinking — it only needs to go up for the answer
- For #76: forgetting to update `formed` in BOTH expansion and contraction phases
- For #239: storing values instead of indices in deque — can't check if front is out of window

**Optimizations:**
- `int freq[26]` beats `unordered_map<char,int>` for lowercase char problems
- For #567 and #438: comparing `int[26]` arrays is O(26) = O(1) — faster than map
- For #239: deque stores indices, not values — enables O(1) window expiry check

**Follow-up interviewers love:**
- "What is the time complexity?" → O(n) — each element enters and leaves deque once
- "Why monotonic deque and not just max?" → Can't remove arbitrary elements from a running max in O(1)
- "Can you find minimum of sliding window?" → Same deque trick but maintain increasing order
