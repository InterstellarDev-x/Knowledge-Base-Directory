# Arrays & Strings Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Contiguous subarray with condition (sum, max, min)?   → Sliding Window
Find pair / triplet satisfying condition?              → Two Pointers
Running sum / range sum query?                         → Prefix Sum
Index where element equals its index?                  → Binary Search
Rearrange in-place (partition, cycle)?                 → In-Place Array
Pattern match / anagram / substring?                   → Sliding Window + HashMap
Characters in string, frequency?                       → Array of size 26
```

**The one question to always ask first:**
> "Is the subarray/substring contiguous?" — if yes, sliding window or prefix sum.
> "Do I need two elements?" — if yes, two pointers or hash map.

---

## Pattern 1: Sliding Window

**When to think about it:**
- "Longest/shortest subarray/substring with property X"
- "Maximum sum subarray of size k"
- "Minimum window containing all characters"
- Window = contiguous range [left, right] that expands and shrinks

**Two types:**
```
Fixed window:    size k never changes → slide right, drop left
Variable window: expand right until invalid, shrink left until valid again
```

**Core insight:** Instead of re-computing for every window from scratch (O(n²)),
maintain a running state. Slide right adds one element, slide left removes one.

### Fixed Window Template
```cpp
// Maximum sum subarray of size k
int maxSumFixed(vector<int>& nums, int k) {
    int windowSum = 0, maxSum = 0;

    for (int i = 0; i < nums.size(); i++) {
        windowSum += nums[i];
        if (i >= k - 1) {
            maxSum = max(maxSum, windowSum);
            windowSum -= nums[i - k + 1];   // remove element leaving window
        }
    }
    return maxSum;
}
```

### Variable Window Template (expand + shrink)
```cpp
// Longest subarray with sum <= target
int longestSubarray(vector<int>& nums, int target) {
    int left = 0, windowSum = 0, maxLen = 0;

    for (int right = 0; right < nums.size(); right++) {
        windowSum += nums[right];               // expand

        while (windowSum > target)              // shrink until valid
            windowSum -= nums[left++];

        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### Minimum Window Substring
```cpp
string minWindow(string s, string t) {
    unordered_map<char, int> need, window;
    for (char c : t) need[c]++;

    int left = 0, valid = 0;
    int start = 0, minLen = INT_MAX;

    for (int right = 0; right < s.size(); right++) {
        char c = s[right];
        window[c]++;
        if (need.count(c) && window[c] == need[c]) valid++;  // char satisfied

        while (valid == need.size()) {           // window has all chars
            if (right - left + 1 < minLen) { minLen = right - left + 1; start = left; }

            char d = s[left++];
            if (need.count(d) && window[d] == need[d]) valid--;  // losing a needed char
            window[d]--;
        }
    }
    return minLen == INT_MAX ? "" : s.substr(start, minLen);
}
```

### Longest Substring Without Repeating Characters
```cpp
int lengthOfLongestSubstring(string s) {
    unordered_map<char, int> lastSeen;
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); right++) {
        if (lastSeen.count(s[right]) && lastSeen[s[right]] >= left)
            left = lastSeen[s[right]] + 1;   // jump left past the duplicate
        lastSeen[s[right]] = right;
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

### Longest Subarray with At Most K Distinct Characters
```cpp
int longestSubarrayKDistinct(string s, int k) {
    unordered_map<char, int> freq;
    int left = 0, maxLen = 0;

    for (int right = 0; right < s.size(); right++) {
        freq[s[right]]++;

        while (freq.size() > k) {
            freq[s[left]]--;
            if (freq[s[left]] == 0) freq.erase(s[left]);
            left++;
        }
        maxLen = max(maxLen, right - left + 1);
    }
    return maxLen;
}
```

**Sliding window decision tree:**
```
Want LONGEST window satisfying condition?
  → Expand right always, shrink left when violated
  → Answer = max window size seen

Want SHORTEST window satisfying condition?
  → Expand right until satisfied, shrink left as much as possible
  → Answer = min window size when condition holds
```

**Problems:** #3 Longest Substring No Repeat, #76 Minimum Window Substring, #567 Permutation in String, #438 Find All Anagrams, #239 Sliding Window Maximum, #424 Longest Repeating Character Replacement, #1004 Max Consecutive Ones III, #209 Minimum Size Subarray Sum

---

## Pattern 2: Two Pointers

**When to think about it:**
- Sorted array, need pairs/triplets
- "Remove duplicates", "move zeros", "reverse"
- Two ends of array moving toward each other
- One slow + one fast pointer through same array

**Two sub-types:**
```
Opposite ends:   left=0, right=n-1, move toward center
Same direction:  slow=0, fast=0, fast runs ahead
```

### Opposite Ends Template
```cpp
// Two Sum in sorted array
vector<int> twoSum(vector<int>& nums, int target) {
    int left = 0, right = nums.size() - 1;
    while (left < right) {
        int sum = nums[left] + nums[right];
        if (sum == target) return {left, right};
        if (sum < target) left++;
        else right--;
    }
    return {};
}

// Container with Most Water
int maxArea(vector<int>& height) {
    int left = 0, right = height.size() - 1, maxWater = 0;
    while (left < right) {
        maxWater = max(maxWater, min(height[left], height[right]) * (right - left));
        if (height[left] < height[right]) left++;
        else right--;
    }
    return maxWater;
}
```

### Same Direction (Fast/Slow) Template
```cpp
// Remove duplicates from sorted array in-place
int removeDuplicates(vector<int>& nums) {
    int slow = 0;
    for (int fast = 1; fast < nums.size(); fast++)
        if (nums[fast] != nums[slow]) nums[++slow] = nums[fast];
    return slow + 1;
}

// Move Zeros to end
void moveZeroes(vector<int>& nums) {
    int slow = 0;
    for (int fast = 0; fast < nums.size(); fast++)
        if (nums[fast] != 0) swap(nums[slow++], nums[fast]);
}
```

### Three Sum
```cpp
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> res;

    for (int i = 0; i < nums.size() - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;   // skip duplicate i

        int left = i+1, right = nums.size()-1;
        while (left < right) {
            int sum = nums[i] + nums[left] + nums[right];
            if (sum == 0) {
                res.push_back({nums[i], nums[left], nums[right]});
                while (left < right && nums[left] == nums[left+1]) left++;
                while (left < right && nums[right] == nums[right-1]) right--;
                left++; right--;
            } else if (sum < 0) left++;
            else right--;
        }
    }
    return res;
}
```

### Trapping Rain Water
```cpp
int trap(vector<int>& height) {
    int left = 0, right = height.size()-1;
    int leftMax = 0, rightMax = 0, water = 0;

    while (left < right) {
        if (height[left] < height[right]) {
            leftMax = max(leftMax, height[left]);
            water += leftMax - height[left++];
        } else {
            rightMax = max(rightMax, height[right]);
            water += rightMax - height[right--];
        }
    }
    return water;
}
```

**Problems:** #167 Two Sum II, #15 3Sum, #11 Container with Most Water, #42 Trapping Rain Water, #26 Remove Duplicates, #283 Move Zeros, #977 Squares of Sorted Array, #16 3Sum Closest

---

## Pattern 3: Prefix Sum

**When to think about it:**
- "Sum of subarray from index i to j"
- "Subarray with sum equal to k"
- "Count subarrays with sum divisible by k"
- Any range query on a static array

**Core insight:** prefix[i] = sum of first i elements.
Sum of subarray [i, j] = prefix[j+1] - prefix[i].
For "count subarrays with sum = k": use hashmap of prefix sums.

### Prefix Sum Array
```cpp
// Build prefix sum
vector<int> buildPrefix(vector<int>& nums) {
    vector<int> prefix(nums.size() + 1, 0);
    for (int i = 0; i < nums.size(); i++)
        prefix[i+1] = prefix[i] + nums[i];
    return prefix;
}

// Range sum query O(1)
int rangeSum(vector<int>& prefix, int l, int r) {
    return prefix[r+1] - prefix[l];
}
```

### Subarray Sum Equals K
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixCount;
    prefixCount[0] = 1;   // empty prefix
    int sum = 0, count = 0;

    for (int x : nums) {
        sum += x;
        // If (sum - k) exists as a prefix sum, we found a valid subarray
        count += prefixCount[sum - k];
        prefixCount[sum]++;
    }
    return count;
}
```

### Subarray Sum Divisible by K
```cpp
int subarraysDivByK(vector<int>& nums, int k) {
    unordered_map<int, int> remainderCount;
    remainderCount[0] = 1;
    int sum = 0, count = 0;

    for (int x : nums) {
        sum += x;
        int rem = ((sum % k) + k) % k;   // handle negative mod in C++
        count += remainderCount[rem];
        remainderCount[rem]++;
    }
    return count;
}
```

### 2D Prefix Sum
```cpp
// Matrix region sum query
vector<vector<int>> build2DPrefix(vector<vector<int>>& mat) {
    int m = mat.size(), n = mat[0].size();
    vector<vector<int>> p(m+1, vector<int>(n+1, 0));

    for (int i = 1; i <= m; i++)
        for (int j = 1; j <= n; j++)
            p[i][j] = mat[i-1][j-1] + p[i-1][j] + p[i][j-1] - p[i-1][j-1];
    return p;
}

// Sum of region (r1,c1) to (r2,c2) — 0-indexed
int regionSum(vector<vector<int>>& p, int r1, int c1, int r2, int c2) {
    return p[r2+1][c2+1] - p[r1][c2+1] - p[r2+1][c1] + p[r1][c1];
}
```

**Problems:** #560 Subarray Sum Equals K, #974 Subarray Sums Divisible by K, #238 Product of Array Except Self, #303 Range Sum Query, #304 Range Sum Query 2D, #525 Contiguous Array, #1480 Running Sum

---

## Pattern 4: In-Place Array Manipulation

**When to think about it:**
- Modify array without extra space
- "Next permutation", "rotate array", "reverse"
- Cyclic sort (covered in sorting.md)

### Next Permutation
```cpp
void nextPermutation(vector<int>& nums) {
    int n = nums.size(), i = n-2;

    // Step 1: find first decreasing element from right
    while (i >= 0 && nums[i] >= nums[i+1]) i--;

    if (i >= 0) {
        // Step 2: find smallest element larger than nums[i] to its right
        int j = n-1;
        while (nums[j] <= nums[i]) j--;
        swap(nums[i], nums[j]);
    }

    // Step 3: reverse suffix
    reverse(nums.begin() + i + 1, nums.end());
}
```

### Rotate Array
```cpp
// Rotate right by k — three reverses trick
void rotate(vector<int>& nums, int k) {
    k %= nums.size();
    reverse(nums.begin(), nums.end());
    reverse(nums.begin(), nums.begin() + k);
    reverse(nums.begin() + k, nums.end());
}
```

### Find Duplicate Number (Floyd's on array)
```cpp
int findDuplicate(vector<int>& nums) {
    int slow = nums[0], fast = nums[0];
    do {
        slow = nums[slow];
        fast = nums[nums[fast]];
    } while (slow != fast);

    slow = nums[0];
    while (slow != fast) { slow = nums[slow]; fast = nums[fast]; }
    return slow;
}
```

**Problems:** #31 Next Permutation, #189 Rotate Array, #287 Find the Duplicate, #41 First Missing Positive, #73 Set Matrix Zeroes

---

## Pattern 5: String Patterns

### Anagram / Permutation Check (sliding window + frequency)
```cpp
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    vector<int> freq(26, 0);
    for (char c : s) freq[c-'a']++;
    for (char c : t) { if (--freq[c-'a'] < 0) return false; }
    return true;
}

// Find all anagram start indices in string s
vector<int> findAnagrams(string s, string p) {
    vector<int> need(26,0), window(26,0), res;
    for (char c : p) need[c-'a']++;
    int k = p.size();

    for (int i = 0; i < s.size(); i++) {
        window[s[i]-'a']++;
        if (i >= k) window[s[i-k]-'a']--;
        if (window == need) res.push_back(i-k+1);
    }
    return res;
}
```

### Z-Algorithm (pattern matching in O(n))
```cpp
// Z[i] = length of longest substring starting at i that matches prefix of s
vector<int> zFunction(string s) {
    int n = s.size();
    vector<int> z(n, 0);
    z[0] = n;
    int l = 0, r = 0;

    for (int i = 1; i < n; i++) {
        if (i < r) z[i] = min(r-i, z[i-l]);
        while (i+z[i] < n && s[z[i]] == s[i+z[i]]) z[i]++;
        if (i+z[i] > r) { l = i; r = i+z[i]; }
    }
    return z;
}
```

### KMP (pattern in text, O(n+m))
```cpp
vector<int> buildLPS(string pat) {
    int n = pat.size();
    vector<int> lps(n, 0);
    int len = 0, i = 1;
    while (i < n) {
        if (pat[i] == pat[len]) lps[i++] = ++len;
        else if (len) len = lps[len-1];
        else lps[i++] = 0;
    }
    return lps;
}

vector<int> kmpSearch(string text, string pat) {
    vector<int> lps = buildLPS(pat);
    vector<int> matches;
    int i = 0, j = 0;
    while (i < text.size()) {
        if (text[i] == pat[j]) { i++; j++; }
        if (j == pat.size()) { matches.push_back(i-j); j = lps[j-1]; }
        else if (i < text.size() && text[i] != pat[j]) {
            if (j) j = lps[j-1]; else i++;
        }
    }
    return matches;
}
```

**Problems:** #242 Valid Anagram, #438 Find All Anagrams, #28 Find Index of First Occurrence (KMP), #459 Repeated Substring Pattern, #1071 GCD of Strings

---

## Pattern 6: Kadane's Algorithm (Maximum Subarray)

**When to think about it:**
- "Maximum sum contiguous subarray"
- "Maximum product subarray"
- Extends to 2D (maximum sum rectangle)

**Core insight:** At each position, either extend previous subarray or start fresh.
dp[i] = max(nums[i], dp[i-1] + nums[i]) — start fresh if dp[i-1] is negative.

```cpp
// Maximum Subarray Sum
int maxSubArray(vector<int>& nums) {
    int curSum = nums[0], maxSum = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        curSum = max(nums[i], curSum + nums[i]);
        maxSum = max(maxSum, curSum);
    }
    return maxSum;
}

// Also track start/end indices
int maxSubArrayWithIndices(vector<int>& nums) {
    int curSum = nums[0], maxSum = nums[0];
    int start = 0, end = 0, tempStart = 0;

    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] > curSum + nums[i]) {
            curSum = nums[i];
            tempStart = i;
        } else {
            curSum += nums[i];
        }
        if (curSum > maxSum) {
            maxSum = curSum;
            start = tempStart;
            end = i;
        }
    }
    return maxSum;
}

// Maximum Product Subarray (track both min and max — negatives flip sign)
int maxProduct(vector<int>& nums) {
    int maxP = nums[0], minP = nums[0], ans = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        if (nums[i] < 0) swap(maxP, minP);
        maxP = max(nums[i], maxP * nums[i]);
        minP = min(nums[i], minP * nums[i]);
        ans  = max(ans, maxP);
    }
    return ans;
}
```

**Problems:** #53 Maximum Subarray, #152 Maximum Product Subarray, #918 Maximum Circular Subarray, #1749 Maximum Absolute Sum

---

## The 5 Core Templates to Memorize

```cpp
// 1. VARIABLE SLIDING WINDOW (longest valid window)
int left = 0; // window state vars
for (int right = 0; right < n; right++) {
    // add nums[right] to window
    while (/* window invalid */) {
        // remove nums[left] from window
        left++;
    }
    ans = max(ans, right - left + 1);
}

// 2. FIXED SLIDING WINDOW (window of size k)
for (int i = 0; i < n; i++) {
    // add nums[i]
    if (i >= k) { /* remove nums[i-k] */ }
    if (i >= k-1) ans = max(ans, /* window value */);
}

// 3. TWO POINTERS OPPOSITE ENDS
int l = 0, r = n-1;
while (l < r) {
    if (/* condition */) l++;
    else r--;
}

// 4. PREFIX SUM + HASHMAP (count subarrays with sum k)
unordered_map<int,int> cnt; cnt[0] = 1;
int sum = 0, ans = 0;
for (int x : nums) {
    sum += x;
    ans += cnt[sum - k];
    cnt[sum]++;
}

// 5. KADANE (max subarray)
int cur = nums[0], ans = nums[0];
for (int i = 1; i < n; i++) {
    cur = max(nums[i], cur + nums[i]);
    ans = max(ans, cur);
}
```

---

## Universal Checklist Before Coding

- [ ] Contiguous subarray/substring? → Sliding window or prefix sum
- [ ] Finding pairs in sorted array? → Two pointers
- [ ] Range sum queries? → Prefix sum
- [ ] Sliding window: am I looking for LONGEST (expand) or SHORTEST (shrink)?
- [ ] Two pointers: sorted input needed? If not, sort first
- [ ] String: 26-size array faster than unordered_map for lowercase letters
- [ ] Prefix sum + hashmap: initialize cnt[0] = 1 (empty prefix)

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 53   | Maximum Subarray                       | Medium | Kadane's              |
| 2  | 3    | Longest Substring Without Repeating    | Medium | Sliding window        |
| 3  | 560  | Subarray Sum Equals K                  | Medium | Prefix sum + hashmap  |
| 4  | 15   | 3Sum                                   | Medium | Two pointers          |
| 5  | 76   | Minimum Window Substring               | Hard   | Sliding window        |
| 6  | 11   | Container With Most Water              | Medium | Two pointers          |
| 7  | 42   | Trapping Rain Water                    | Hard   | Two pointers          |
| 8  | 238  | Product of Array Except Self           | Medium | Prefix sum            |
| 9  | 209  | Minimum Size Subarray Sum              | Medium | Sliding window        |
| 10 | 438  | Find All Anagrams in String            | Medium | Sliding window        |
| 11 | 567  | Permutation in String                  | Medium | Sliding window        |
| 12 | 424  | Longest Repeating Character Replacement| Medium | Sliding window        |
| 13 | 152  | Maximum Product Subarray               | Medium | Kadane's variant      |
| 14 | 31   | Next Permutation                       | Medium | In-place              |
| 15 | 287  | Find the Duplicate Number              | Medium | Floyd's on array      |
| 16 | 974  | Subarray Sums Divisible by K           | Medium | Prefix sum            |
| 17 | 1004 | Max Consecutive Ones III               | Medium | Sliding window        |
| 18 | 239  | Sliding Window Maximum                 | Hard   | Monotonic deque       |
| 19 | 525  | Contiguous Array                       | Medium | Prefix sum            |
| 20 | 128  | Longest Consecutive Sequence           | Medium | HashSet               |

**Study order:** #53 → #3 → #560 → #15 → #11 → #42 → #238 → #76 → #424

---

## Company-Wise Most Asked

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #3, #53, #560, #15, #209, #238 |
| **Google** | #76, #239, #42, #567, #128 |
| **Meta** | #3, #15, #42, #76, #238, #560 |
| **Microsoft** | #53, #3, #238, #15, #283 |
| **Bloomberg** | #53, #560, #11, #42, #209 |
