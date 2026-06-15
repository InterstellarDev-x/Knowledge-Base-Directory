# Heap / Priority Queue Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Kth largest / smallest element?                    → Min-Heap of size k
Top K frequent elements?                           → Min-Heap of size k
Merge K sorted lists/arrays?                       → Min-Heap
Continuously find median from stream?              → Two Heaps (max + min)
Shortest path / minimum cost?                      → Min-Heap (Dijkstra)
Schedule tasks / CPU scheduling?                   → Max-Heap
Sliding window maximum/minimum?                    → Monotonic Deque
```

**The one question to always ask first:**
> "Do I repeatedly need the minimum or maximum from a changing set?"
> — If yes, it's a heap.

---

## C++ Priority Queue Quick Reference

```cpp
#include <queue>

// MAX-HEAP (default) — largest element on top
priority_queue<int> maxPQ;
maxPQ.push(x);
maxPQ.top();    // largest
maxPQ.pop();

// MIN-HEAP — smallest element on top
priority_queue<int, vector<int>, greater<int>> minPQ;

// CUSTOM COMPARATOR (min-heap by first element of pair)
auto cmp = [](pair<int,int>& a, pair<int,int>& b) {
    return a.first > b.first;   // greater = min-heap
};
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);

// Build from vector — O(n) heapify
vector<int> v = {3,1,4,1,5};
priority_queue<int, vector<int>, greater<int>> pq(v.begin(), v.end());
```

> **Key rule:** In C++ comparator for priority_queue, `return a > b` gives MIN-heap
> (opposite of sort comparator). Think: "return true if a should go LOWER in priority."

---

## Pattern 1: Kth Largest / Smallest

**When to think about it:**
- "Find kth largest element"
- "Find k closest points"
- "Top k frequent elements"
- Don't need all elements sorted — only need the kth one

**Core insight:** Use a min-heap of size k.
Maintain only the k largest elements. The top of the heap = kth largest.
When heap exceeds size k, pop the minimum — that element is not in top-k.

```
Finding 3rd largest in [4,1,7,3,8,5]:

Push 4 → heap: [4]
Push 1 → heap: [1,4]
Push 7 → heap: [1,4,7]   size == k, top = 1 (3rd largest so far = 1)
Push 3 → heap: [1,3,4,7] size > k, pop 1 → heap: [3,4,7]
Push 8 → heap: [3,4,7,8] size > k, pop 3 → heap: [4,7,8]
Push 5 → heap: [4,5,7,8] size > k, pop 4 → heap: [5,7,8]

Answer = heap.top() = 5  ✓
```

```cpp
// Kth Largest Element
int findKthLargest(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> minHeap;  // min-heap

    for (int x : nums) {
        minHeap.push(x);
        if (minHeap.size() > k) minHeap.pop();   // remove smallest, keep k largest
    }
    return minHeap.top();   // kth largest
}

// K Closest Points to Origin
vector<vector<int>> kClosest(vector<vector<int>>& points, int k) {
    // Max-heap by distance — when size > k, pop farthest
    auto cmp = [](vector<int>& a, vector<int>& b) {
        return a[0]*a[0]+a[1]*a[1] < b[0]*b[0]+b[1]*b[1];  // max-heap
    };
    priority_queue<vector<int>, vector<vector<int>>, decltype(cmp)> pq(cmp);

    for (auto& p : points) {
        pq.push(p);
        if (pq.size() > k) pq.pop();   // remove farthest
    }

    vector<vector<int>> res;
    while (!pq.empty()) { res.push_back(pq.top()); pq.pop(); }
    return res;
}

// Top K Frequent Elements
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int x : nums) freq[x]++;

    // Min-heap by frequency
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    for (auto& [val, cnt] : freq) {
        pq.push({cnt, val});
        if (pq.size() > k) pq.pop();
    }

    vector<int> res;
    while (!pq.empty()) { res.push_back(pq.top().second); pq.pop(); }
    return res;
}
```

**Min-heap of size k rule:**
```
Top K LARGEST  → Min-heap of size k (pop when size > k, top = kth largest)
Top K SMALLEST → Max-heap of size k (pop when size > k, top = kth smallest)
```

**Problems:** #215 Kth Largest, #973 K Closest Points, #347 Top K Frequent, #692 Top K Frequent Words, #264 Ugly Number II, #1985 Find the Kth Largest Integer

---

## Pattern 2: Merge K Sorted Lists / Arrays

**When to think about it:**
- "Merge k sorted linked lists"
- "Find smallest range covering elements from k lists"
- "Kth smallest in k sorted arrays"
- Multiple sorted sources, need global sorted order

**Core insight:** Use a min-heap to always pick the globally smallest next element.
Push the first element of each list. Pop min, push next from same list.

```cpp
// Merge K Sorted Lists
ListNode* mergeKLists(vector<ListNode*>& lists) {
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

    for (auto* head : lists)
        if (head) pq.push(head);

    ListNode* dummy = new ListNode(0), *cur = dummy;

    while (!pq.empty()) {
        auto* node = pq.top(); pq.pop();
        cur->next = node;
        cur = cur->next;
        if (node->next) pq.push(node->next);
    }
    return dummy->next;
}

// Kth Smallest in K Sorted Arrays
int kthSmallest(vector<vector<int>>& arrays, int k) {
    // {value, array_index, element_index}
    priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;

    for (int i = 0; i < arrays.size(); i++)
        if (!arrays[i].empty()) pq.push({arrays[i][0], i, 0});

    int count = 0;
    while (!pq.empty()) {
        auto [val, i, j] = pq.top(); pq.pop();
        if (++count == k) return val;
        if (j+1 < arrays[i].size()) pq.push({arrays[i][j+1], i, j+1});
    }
    return -1;
}
```

**Problems:** #23 Merge K Sorted Lists, #378 Kth Smallest in Sorted Matrix, #373 Find K Pairs with Smallest Sums, #632 Smallest Range Covering Elements from K Lists

---

## Pattern 3: Two Heaps — Running Median

**When to think about it:**
- "Find median from a data stream"
- "Sliding window median"
- Need to efficiently access both the lower half max and upper half min

**Core insight:** Split data into two halves:
- `maxHeap` (lower half) — top = median candidate or lower median
- `minHeap` (upper half) — top = upper median

Invariant: `maxHeap.size() == minHeap.size()` or `maxHeap.size() == minHeap.size() + 1`

```
Stream: [5, 15, 1, 3]

Add 5:
  maxHeap: [5]    minHeap: []     median = 5

Add 15:
  Push to maxHeap: [15,5], rebalance → push max to minHeap
  maxHeap: [5]    minHeap: [15]   median = (5+15)/2 = 10

Add 1:
  Push to maxHeap: [5,1], size equal → median = maxHeap.top() = 5
  maxHeap: [5,1]  minHeap: [15]   median = 5

Add 3:
  Push to maxHeap: [5,3,1], rebalance → push max to minHeap
  maxHeap: [3,1]  minHeap: [5,15] median = (3+5)/2 = 4
```

```cpp
class MedianFinder {
    priority_queue<int> maxHeap;                          // lower half
    priority_queue<int, vector<int>, greater<int>> minHeap; // upper half

public:
    void addNum(int num) {
        maxHeap.push(num);

        // Ensure all maxHeap elements <= all minHeap elements
        if (!minHeap.empty() && maxHeap.top() > minHeap.top()) {
            minHeap.push(maxHeap.top()); maxHeap.pop();
        }

        // Balance sizes: maxHeap can have at most 1 more than minHeap
        if (maxHeap.size() > minHeap.size() + 1) {
            minHeap.push(maxHeap.top()); maxHeap.pop();
        } else if (minHeap.size() > maxHeap.size()) {
            maxHeap.push(minHeap.top()); minHeap.pop();
        }
    }

    double findMedian() {
        if (maxHeap.size() > minHeap.size()) return maxHeap.top();
        return (maxHeap.top() + minHeap.top()) / 2.0;
    }
};
```

**Problems:** #295 Find Median from Data Stream, #480 Sliding Window Median

---

## Pattern 4: Task Scheduling

**When to think about it:**
- "CPU task scheduling with cooldown"
- "Schedule jobs to minimize time"
- "Reorganize string so no two same adjacent"
- Greedily pick the most frequent available task

**Core insight:** Always process the task with highest remaining frequency.
If that task is on cooldown, wait or fill with other tasks.

```cpp
// Task Scheduler (CPU cooldown n)
int leastInterval(vector<char>& tasks, int n) {
    vector<int> freq(26, 0);
    for (char t : tasks) freq[t-'A']++;

    // max-heap of frequencies
    priority_queue<int> pq;
    for (int f : freq) if (f > 0) pq.push(f);

    int time = 0;
    while (!pq.empty()) {
        vector<int> temp;
        int cycle = n + 1;   // one cooldown cycle

        // Pick up to (n+1) tasks in this cycle
        while (cycle-- && !pq.empty()) {
            temp.push_back(pq.top() - 1); pq.pop();
            time++;
        }

        // Re-add tasks that still have remaining count
        for (int t : temp) if (t > 0) pq.push(t);

        // If queue still has tasks, we completed a full cycle
        // If empty, we may have finished early (time already incremented)
        if (!pq.empty()) time += cycle + 1; // fill idle slots
    }
    return time;
}

// Reorganize String (no two same adjacent)
string reorganizeString(string s) {
    vector<int> freq(26, 0);
    for (char c : s) freq[c-'a']++;

    priority_queue<pair<int,char>> pq;
    for (int i = 0; i < 26; i++)
        if (freq[i]) pq.push({freq[i], 'a'+i});

    string res;
    while (pq.size() >= 2) {
        auto [f1, c1] = pq.top(); pq.pop();
        auto [f2, c2] = pq.top(); pq.pop();
        res += c1; res += c2;
        if (f1-1) pq.push({f1-1, c1});
        if (f2-1) pq.push({f2-1, c2});
    }
    if (!pq.empty()) {
        if (pq.top().first > 1) return "";   // impossible
        res += pq.top().second;
    }
    return res;
}
```

**Problems:** #621 Task Scheduler, #767 Reorganize String, #1054 Distant Barcodes, #358 Rearrange String k Distance Apart

---

## Pattern 5: Heap with Lazy Deletion

**When to think about it:**
- Elements need to be removed from heap but heap doesn't support random deletion
- "Sliding window maximum where expired elements stay in heap"
- Maintain a "to-delete" set, skip stale entries when popping

```cpp
// Kth Largest in Stream with deletions
class KthLargest {
    priority_queue<int, vector<int>, greater<int>> minHeap;
    unordered_map<int, int> toDelete;
    int k;

    void clean() {
        while (!minHeap.empty() && toDelete.count(minHeap.top())) {
            toDelete[minHeap.top()]--;
            if (toDelete[minHeap.top()] == 0) toDelete.erase(minHeap.top());
            minHeap.pop();
        }
    }

public:
    KthLargest(int k, vector<int>& nums) : k(k) {
        for (int x : nums) add(x);
    }

    int add(int val) {
        minHeap.push(val);
        if (minHeap.size() > k) {
            toDelete[minHeap.top()]++;
            minHeap.pop();
        }
        clean();
        return minHeap.top();
    }

    void remove(int val) {
        toDelete[val]++;
        clean();
        // Refill if needed
    }
};
```

**Problems:** #703 Kth Largest in Stream, #480 Sliding Window Median (lazy deletion)

---

## Pattern 6: Monotonic Deque (Sliding Window Min/Max)

**When to think about it:**
- "Maximum in every sliding window of size k"
- "Minimum in sliding window"
- Heap works but O(n log n) — deque gives O(n)

**Core insight:** Maintain a deque of indices where values are monotonically decreasing (for max).
Front = current window max. Remove elements outside window from front.
Remove elements smaller than new element from back (they'll never be the max).

```cpp
// Sliding Window Maximum
vector<int> maxSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;   // stores indices, values are decreasing
    vector<int> res;

    for (int i = 0; i < nums.size(); i++) {
        // Remove elements outside window
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();

        // Remove elements smaller than current (they can never be max)
        while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back();

        dq.push_back(i);

        if (i >= k-1) res.push_back(nums[dq.front()]);   // front = max
    }
    return res;
}

// Sliding Window Minimum — flip comparison
vector<int> minSlidingWindow(vector<int>& nums, int k) {
    deque<int> dq;
    vector<int> res;

    for (int i = 0; i < nums.size(); i++) {
        while (!dq.empty() && dq.front() <= i - k) dq.pop_front();
        while (!dq.empty() && nums[dq.back()] > nums[i]) dq.pop_back(); // flip >
        dq.push_back(i);
        if (i >= k-1) res.push_back(nums[dq.front()]);
    }
    return res;
}
```

**Heap vs Deque for sliding window:**
```
Heap:  O(n log k) — easier to code, handles complex comparators
Deque: O(n)       — faster, but only works for simple min/max
Use deque when k is large and performance matters.
```

**Problems:** #239 Sliding Window Maximum, #1438 Longest Subarray with Absolute Diff ≤ Limit, #862 Shortest Subarray with Sum at Least K

---

## Pattern 7: Heap for Greedy / Simulation

**When to think about it:**
- "Minimize/maximize total cost by always making the best local choice"
- "Meeting rooms", "event processing"
- Greedily process events in order, use heap to track ongoing ones

```cpp
// Meeting Rooms II — minimum rooms needed
int minMeetingRooms(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    priority_queue<int, vector<int>, greater<int>> endTimes;  // min-heap of end times

    for (auto& interval : intervals) {
        // If earliest-ending meeting is done, reuse that room
        if (!endTimes.empty() && endTimes.top() <= interval[0])
            endTimes.pop();
        endTimes.push(interval[1]);   // add this meeting's end time
    }
    return endTimes.size();   // rooms still in use = minimum rooms needed
}

// Minimum Cost to Connect Ropes (combine k cheapest each time)
int connectRopes(vector<int>& ropes) {
    priority_queue<int, vector<int>, greater<int>> pq(ropes.begin(), ropes.end());
    int totalCost = 0;

    while (pq.size() > 1) {
        int first  = pq.top(); pq.pop();
        int second = pq.top(); pq.pop();
        int cost   = first + second;
        totalCost += cost;
        pq.push(cost);
    }
    return totalCost;
}

// IPO — maximize capital by selecting k projects
int findMaximizedCapital(int k, int w, vector<int>& profits, vector<int>& capital) {
    int n = profits.size();
    vector<pair<int,int>> projects(n);
    for (int i = 0; i < n; i++) projects[i] = {capital[i], profits[i]};
    sort(projects.begin(), projects.end());   // sort by capital

    priority_queue<int> available;   // max-heap of available profits
    int idx = 0;

    for (int i = 0; i < k; i++) {
        // Add all projects we can now afford
        while (idx < n && projects[idx].first <= w) {
            available.push(projects[idx].second);
            idx++;
        }
        if (available.empty()) break;
        w += available.top(); available.pop();   // pick highest profit
    }
    return w;
}
```

**Problems:** #253 Meeting Rooms II, #502 IPO, #1167 Minimum Cost to Connect Sticks, #1642 Furthest Building You Can Reach, #2402 Meeting Rooms III

---

## The 4 Core Templates to Memorize

```cpp
// 1. KTH LARGEST (min-heap of size k)
priority_queue<int, vector<int>, greater<int>> pq;
for (int x : nums) {
    pq.push(x);
    if (pq.size() > k) pq.pop();
}
return pq.top();  // kth largest

// 2. MERGE K SORTED (min-heap with source tracking)
// {value, list_index, element_index}
priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
for (int i = 0; i < k; i++) pq.push({lists[i][0], i, 0});
while (!pq.empty()) {
    auto [val, i, j] = pq.top(); pq.pop();
    // process val
    if (j+1 < lists[i].size()) pq.push({lists[i][j+1], i, j+1});
}

// 3. TWO HEAPS (running median)
priority_queue<int> lo;                              // max-heap, lower half
priority_queue<int, vector<int>, greater<int>> hi;   // min-heap, upper half
// invariant: lo.size() == hi.size() OR lo.size() == hi.size()+1
// median: lo.size() > hi.size() ? lo.top() : (lo.top()+hi.top())/2.0

// 4. MONOTONIC DEQUE (sliding window max)
deque<int> dq;  // indices, values decreasing
for (int i = 0; i < n; i++) {
    while (!dq.empty() && dq.front() <= i-k) dq.pop_front();
    while (!dq.empty() && nums[dq.back()] < nums[i]) dq.pop_back();
    dq.push_back(i);
    if (i >= k-1) res.push_back(nums[dq.front()]);
}
```

---

## Universal Checklist Before Coding

- [ ] Min-heap or max-heap? Min = `greater<int>`, Max = default
- [ ] Size-k heap: push first, then pop if size > k
- [ ] Two heaps invariant: maxHeap.size() == minHeap.size() or +1
- [ ] Merge k lists: always push the next element from same source
- [ ] Deque sliding window: pop front (out of window), pop back (smaller than new), push back
- [ ] C++ comparator for pq: `return a > b` → min-heap (counter-intuitive, memorize it)

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 215  | Kth Largest Element in Array           | Medium | Kth element           |
| 2  | 347  | Top K Frequent Elements                | Medium | Top K                 |
| 3  | 23   | Merge K Sorted Lists                   | Hard   | Merge K sorted        |
| 4  | 295  | Find Median from Data Stream           | Hard   | Two heaps             |
| 5  | 239  | Sliding Window Maximum                 | Hard   | Monotonic deque       |
| 6  | 973  | K Closest Points to Origin             | Medium | Kth element           |
| 7  | 253  | Meeting Rooms II                       | Medium | Greedy + heap         |
| 8  | 621  | Task Scheduler                         | Medium | Task scheduling       |
| 9  | 767  | Reorganize String                      | Medium | Task scheduling       |
| 10 | 378  | Kth Smallest in Sorted Matrix          | Medium | Merge K / BS          |
| 11 | 703  | Kth Largest in Stream                  | Easy   | Kth element stream    |
| 12 | 502  | IPO                                    | Hard   | Greedy + heap         |
| 13 | 373  | Find K Pairs Smallest Sums             | Medium | Merge K sorted        |
| 14 | 1167 | Min Cost to Connect Sticks             | Medium | Greedy + heap         |
| 15 | 480  | Sliding Window Median                  | Hard   | Two heaps             |

**Study order:** #215 → #347 → #23 → #295 → #239 → #253 → #621 → #502

---

## Company-Wise Most Asked

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #215, #347, #23, #253, #621, #973 |
| **Google** | #295, #239, #23, #502, #480, #373 |
| **Meta** | #215, #347, #23, #253, #767 |
| **Microsoft** | #215, #23, #347, #239, #703 |
| **Bloomberg** | #23, #215, #253, #621, #1167 |

---

## Recent Interview Trends (2023–2025)

### 1. Two Heaps (median) is a Google/Meta senior-level staple
#295 is asked repeatedly. Interviewers follow up: "what if the stream has deletions?"
Know lazy deletion with a to-delete map.

### 2. Heap + Greedy combo is rising
#502 IPO, #1167 Connect Sticks, #1642 Furthest Building — all use a heap
to make greedy choices efficiently. The pattern: sort one dimension, heap on another.

### 3. Monotonic Deque is underrated for O(n) window problems
#239 Sliding Window Maximum is a Google/Meta hard that most candidates solve with heap (O(n log k)).
Knowing the deque approach (O(n)) is a differentiator at senior levels.

### 4. Task Scheduler (#621) appears at Amazon constantly
The mathematical shortcut (idle slots formula) exists, but interviewers also ask for
the simulation approach using a heap. Know both.
