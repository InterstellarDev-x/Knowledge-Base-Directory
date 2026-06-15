# HashMap / HashSet Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Count frequency of elements?                       → unordered_map<T, int>
Check existence / visited?                         → unordered_set<T>
Two Sum / complement lookup?                       → HashMap (val → index)
Anagram / permutation check?                       → Frequency array or map
Group elements by property?                        → HashMap (key → list)
Cache / memoize?                                   → unordered_map
Design O(1) insert, delete, random access?         → HashMap + vector
```

**The one question to always ask first:**
> "Do I need to look something up in O(1)?" — if yes, it's a hash map or set.

---

## C++ Quick Reference

```cpp
#include <unordered_map>
#include <unordered_set>

// unordered_map — O(1) average insert/lookup/delete
unordered_map<int, int> mp;
mp[key] = val;
mp.count(key);          // 1 if exists, 0 if not
mp.find(key);           // iterator, mp.end() if not found
mp.erase(key);
mp.size();
for (auto& [k, v] : mp) { }  // iterate

// unordered_set — O(1) average
unordered_set<int> st;
st.insert(x);
st.count(x);            // 1 if exists
st.erase(x);

// ordered versions (O(log n) but sorted)
map<int,int> ordered;
set<int> orderedSet;

// Frequency count shortcut
for (int x : nums) freq[x]++;
// or
unordered_map<int,int> freq(nums.begin(), nums.end()); // doesn't count — just shows pattern
```

> **C++ gotcha:** `mp[key]` creates entry with default value 0 if key doesn't exist.
> Use `mp.count(key)` or `mp.find(key)` to check existence without creating entries.

---

## Pattern 1: Two Sum Family (Complement Lookup)

**When to think about it:**
- "Find two elements that sum to target"
- "Find pair with given difference/product"
- Brute force is O(n²) — hashmap reduces to O(n)

**Core insight:** For each element x, check if (target - x) already exists.
Store what you've seen so far in the map.

```cpp
// Two Sum — return indices
vector<int> twoSum(vector<int>& nums, int target) {
    unordered_map<int,int> seen;  // value -> index
    for (int i = 0; i < nums.size(); i++) {
        int complement = target - nums[i];
        if (seen.count(complement)) return {seen[complement], i};
        seen[nums[i]] = i;
    }
    return {};
}

// Two Sum — return values (sorted array, multiple pairs)
vector<pair<int,int>> twoSumAllPairs(vector<int>& nums, int target) {
    unordered_set<int> seen;
    unordered_set<int> used;
    vector<pair<int,int>> res;

    for (int x : nums) {
        int comp = target - x;
        if (seen.count(comp) && !used.count(x)) {
            res.push_back({min(x,comp), max(x,comp)});
            used.insert(x); used.insert(comp);
        }
        seen.insert(x);
    }
    return res;
}

// Four Sum Count (count tuples from 4 arrays summing to 0)
int fourSumCount(vector<int>& A, vector<int>& B, vector<int>& C, vector<int>& D) {
    unordered_map<int,int> sumAB;
    for (int a : A) for (int b : B) sumAB[a+b]++;

    int count = 0;
    for (int c : C) for (int d : D)
        count += sumAB[-(c+d)];
    return count;
}
```

**Problems:** #1 Two Sum, #167 Two Sum II, #454 4Sum II, #653 Two Sum IV BST, #1010 Pairs of Songs

---

## Pattern 2: Frequency Counting

**When to think about it:**
- "Most frequent element"
- "Check if two strings are anagrams"
- "Find all elements appearing more than n/3 times"
- Count occurrences, then use counts

```cpp
// Valid Anagram
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int freq[26] = {};
    for (char c : s) freq[c-'a']++;
    for (char c : t) if (--freq[c-'a'] < 0) return false;
    return true;
}

// Most Common Word (excluding banned)
string mostCommonWord(string paragraph, vector<string>& banned) {
    unordered_set<string> ban(banned.begin(), banned.end());
    unordered_map<string,int> freq;

    // Normalize: lowercase, split by non-alpha
    for (auto& c : paragraph) c = isalpha(c) ? tolower(c) : ' ';
    istringstream iss(paragraph);
    string word;
    while (iss >> word)
        if (!ban.count(word)) freq[word]++;

    return max_element(freq.begin(), freq.end(),
        [](auto& a, auto& b) { return a.second < b.second; })->first;
}

// Majority Element (Boyer-Moore — O(1) space)
int majorityElement(vector<int>& nums) {
    int candidate = nums[0], count = 1;
    for (int i = 1; i < nums.size(); i++) {
        if (count == 0) { candidate = nums[i]; count = 1; }
        else count += (nums[i] == candidate) ? 1 : -1;
    }
    return candidate;
}

// Element appearing more than n/3 times — at most 2 such elements
vector<int> majorityElementII(vector<int>& nums) {
    int c1 = 0, c2 = 1, cnt1 = 0, cnt2 = 0;
    for (int x : nums) {
        if (x == c1) cnt1++;
        else if (x == c2) cnt2++;
        else if (cnt1 == 0) { c1 = x; cnt1 = 1; }
        else if (cnt2 == 0) { c2 = x; cnt2 = 1; }
        else { cnt1--; cnt2--; }
    }
    // Verify candidates
    cnt1 = cnt2 = 0;
    for (int x : nums) { if (x==c1) cnt1++; if (x==c2) cnt2++; }
    vector<int> res;
    if (cnt1 > nums.size()/3) res.push_back(c1);
    if (cnt2 > nums.size()/3) res.push_back(c2);
    return res;
}
```

**Problems:** #242 Valid Anagram, #169 Majority Element, #229 Majority Element II, #347 Top K Frequent, #451 Sort Characters by Frequency

---

## Pattern 3: Grouping / Bucketing

**When to think about it:**
- "Group anagrams together"
- "Group elements by some canonical form"
- HashMap where key = canonical form, value = list of elements

**Core insight:** Define a "signature" or "key" that is the same for all elements in a group.
For anagrams: sorted string. For equivalent fractions: reduced form. For patterns: shape string.

```cpp
// Group Anagrams
vector<vector<string>> groupAnagrams(vector<string>& strs) {
    unordered_map<string, vector<string>> groups;
    for (auto& s : strs) {
        string key = s;
        sort(key.begin(), key.end());   // canonical form
        groups[key].push_back(s);
    }

    vector<vector<string>> res;
    for (auto& [k, v] : groups) res.push_back(v);
    return res;
}

// Efficient key without sorting (frequency string "a2b1c3...")
string getKey(string& s) {
    int freq[26] = {};
    for (char c : s) freq[c-'a']++;
    string key;
    for (int i = 0; i < 26; i++)
        if (freq[i]) key += string(1,'a'+i) + to_string(freq[i]);
    return key;
}

// Isomorphic Strings (bijective character mapping)
bool isIsomorphic(string s, string t) {
    unordered_map<char,char> s2t, t2s;
    for (int i = 0; i < s.size(); i++) {
        if (s2t.count(s[i]) && s2t[s[i]] != t[i]) return false;
        if (t2s.count(t[i]) && t2s[t[i]] != s[i]) return false;
        s2t[s[i]] = t[i];
        t2s[t[i]] = s[i];
    }
    return true;
}

// Word Pattern ("aabb" matches "dog dog cat cat")
bool wordPattern(string pattern, string s) {
    unordered_map<char,string> c2w;
    unordered_map<string,char> w2c;
    istringstream iss(s);
    vector<string> words;
    string word;
    while (iss >> word) words.push_back(word);
    if (pattern.size() != words.size()) return false;

    for (int i = 0; i < pattern.size(); i++) {
        char p = pattern[i]; string& w = words[i];
        if (c2w.count(p) && c2w[p] != w) return false;
        if (w2c.count(w) && w2c[w] != p) return false;
        c2w[p] = w; w2c[w] = p;
    }
    return true;
}
```

**Problems:** #49 Group Anagrams, #205 Isomorphic Strings, #290 Word Pattern, #249 Group Shifted Strings

---

## Pattern 4: Prefix Sum + HashMap

**When to think about it:**
- "Number of subarrays with sum = k"
- "Longest subarray with equal 0s and 1s"
- "Subarray with sum divisible by k"
- Already in arrays-strings.md — restated here as a HashMap pattern

```cpp
// Subarray Sum Equals K
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int,int> prefixCount;
    prefixCount[0] = 1;   // empty subarray
    int sum = 0, count = 0;
    for (int x : nums) {
        sum += x;
        count += prefixCount[sum - k];
        prefixCount[sum]++;
    }
    return count;
}

// Longest subarray with equal number of 0s and 1s
// Trick: treat 0 as -1, find longest subarray with sum 0
int findMaxLength(vector<int>& nums) {
    unordered_map<int,int> firstSeen;
    firstSeen[0] = -1;
    int sum = 0, maxLen = 0;
    for (int i = 0; i < nums.size(); i++) {
        sum += nums[i] == 1 ? 1 : -1;
        if (firstSeen.count(sum)) maxLen = max(maxLen, i - firstSeen[sum]);
        else firstSeen[sum] = i;
    }
    return maxLen;
}
```

**Problems:** #560 Subarray Sum Equals K, #525 Contiguous Array, #974 Subarray Sums Divisible by K, #1371 Longest Even Vowel Substring

---

## Pattern 5: HashSet for O(1) Lookup

**When to think about it:**
- "Longest consecutive sequence"
- "Contains duplicate"
- Avoid visited array for sparse graphs
- Check membership efficiently

```cpp
// Longest Consecutive Sequence — O(n)
int longestConsecutive(vector<int>& nums) {
    unordered_set<int> numSet(nums.begin(), nums.end());
    int maxLen = 0;

    for (int x : numSet) {
        if (!numSet.count(x-1)) {   // only start sequence from the smallest
            int len = 1;
            while (numSet.count(x + len)) len++;
            maxLen = max(maxLen, len);
        }
    }
    return maxLen;
}

// Contains Duplicate Within k Distance
bool containsNearbyDuplicate(vector<int>& nums, int k) {
    unordered_map<int,int> lastIdx;
    for (int i = 0; i < nums.size(); i++) {
        if (lastIdx.count(nums[i]) && i - lastIdx[nums[i]] <= k) return true;
        lastIdx[nums[i]] = i;
    }
    return false;
}
```

**Problems:** #128 Longest Consecutive Sequence, #217 Contains Duplicate, #219 Contains Duplicate II, #36 Valid Sudoku

---

## Pattern 6: Design with HashMap

**When to think about it:**
- O(1) insert, delete, get operations needed
- LRU/LFU Cache
- Random access by value

```cpp
// Insert Delete GetRandom O(1)
class RandomizedSet {
    unordered_map<int,int> valToIdx;  // value -> index in vals
    vector<int> vals;

public:
    bool insert(int val) {
        if (valToIdx.count(val)) return false;
        vals.push_back(val);
        valToIdx[val] = vals.size()-1;
        return true;
    }

    bool remove(int val) {
        if (!valToIdx.count(val)) return false;
        int idx = valToIdx[val];
        int last = vals.back();
        vals[idx] = last;             // move last to deleted position
        valToIdx[last] = idx;
        vals.pop_back();
        valToIdx.erase(val);
        return true;
    }

    int getRandom() {
        return vals[rand() % vals.size()];
    }
};
```

**Problems:** #380 Insert Delete GetRandom O(1), #146 LRU Cache (see linkedlist.md), #460 LFU Cache, #432 All O(1) Data Structure

---

## The 4 Core Templates to Memorize

```cpp
// 1. TWO SUM (complement lookup)
unordered_map<int,int> seen;
for (int i = 0; i < n; i++) {
    if (seen.count(target - nums[i])) return {seen[target-nums[i]], i};
    seen[nums[i]] = i;
}

// 2. FREQUENCY COUNT
unordered_map<int,int> freq;
for (int x : nums) freq[x]++;
// use freq for top-k, anagram, majority etc.

// 3. PREFIX SUM + HASHMAP (count subarrays)
unordered_map<int,int> cnt; cnt[0] = 1;
int sum = 0, ans = 0;
for (int x : nums) { sum += x; ans += cnt[sum-k]; cnt[sum]++; }

// 4. GROUPING (canonical key)
unordered_map<string, vector<string>> groups;
for (auto& s : strs) {
    string key = getCanonicalForm(s);   // sorted, frequency string, etc.
    groups[key].push_back(s);
}
```

---

## Universal Checklist Before Coding

- [ ] Use `mp.count(key)` to check existence (not `mp[key]` which creates entry)
- [ ] For strings: 26-size int array faster than unordered_map for lowercase alpha
- [ ] Prefix sum + map: always initialize `cnt[0] = 1` (empty prefix case)
- [ ] Two sum: store value → index in map
- [ ] Group anagrams: sorted string or frequency string as key
- [ ] unordered_map worst case is O(n) due to hash collisions — use map if needed

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 1    | Two Sum                                | Easy   | Complement lookup     |
| 2  | 242  | Valid Anagram                          | Easy   | Frequency count       |
| 3  | 49   | Group Anagrams                         | Medium | Grouping              |
| 4  | 128  | Longest Consecutive Sequence           | Medium | HashSet               |
| 5  | 560  | Subarray Sum Equals K                  | Medium | Prefix + HashMap      |
| 6  | 347  | Top K Frequent Elements                | Medium | Frequency + heap      |
| 7  | 169  | Majority Element                       | Easy   | Boyer-Moore / map     |
| 8  | 380  | Insert Delete GetRandom O(1)           | Medium | Design + map          |
| 9  | 525  | Contiguous Array                       | Medium | Prefix + HashMap      |
| 10 | 205  | Isomorphic Strings                     | Easy   | Bijective map         |
| 11 | 454  | 4Sum II                                | Medium | Two-pass complement   |
| 12 | 290  | Word Pattern                           | Easy   | Bijective map         |
| 13 | 36   | Valid Sudoku                           | Medium | HashSet rows/cols/box |
| 14 | 146  | LRU Cache                              | Medium | HashMap + linked list |
| 15 | 460  | LFU Cache                              | Hard   | HashMap design        |

**Study order:** #1 → #242 → #49 → #128 → #560 → #380 → #146

---

## Company-Wise Most Asked

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #1, #49, #128, #146, #347, #560 |
| **Google** | #128, #49, #380, #460, #525, #454 |
| **Meta** | #1, #49, #169, #560, #146 |
| **Microsoft** | #1, #242, #49, #128, #217 |
| **Bloomberg** | #1, #49, #146, #242, #380 |
