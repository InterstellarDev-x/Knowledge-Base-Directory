# Pattern 6: Interval Problems

## Identify This Pattern
- "Merge overlapping intervals"
- "Insert a new interval"
- "Meeting rooms — can all attend? / minimum rooms needed?"
- "Non-overlapping intervals — minimum removals"
- "Employee free time"
- Input: list of [start, end] pairs

## Core Insight
```
Sort by start time → intervals that might overlap are adjacent.
Two intervals [a,b] and [c,d] overlap iff c <= b  (where a <= c after sorting).
Merge: new interval = [a, max(b,d)]
```

---

## Templates

### Merge Intervals
```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());  // sort by start
    vector<vector<int>> res;

    for (auto& interval : intervals) {
        if (res.empty() || res.back()[1] < interval[0]) {
            res.push_back(interval);           // no overlap → add new
        } else {
            res.back()[1] = max(res.back()[1], interval[1]);  // overlap → extend end
        }
    }
    return res;
}
```

### Insert Interval (into sorted, non-overlapping list)
```cpp
vector<vector<int>> insert(vector<vector<int>>& intervals, vector<int>& newInterval) {
    vector<vector<int>> res;
    int i = 0, n = intervals.size();

    // 1. Add all intervals that end before newInterval starts
    while (i < n && intervals[i][1] < newInterval[0])
        res.push_back(intervals[i++]);

    // 2. Merge all overlapping intervals
    while (i < n && intervals[i][0] <= newInterval[1]) {
        newInterval[0] = min(newInterval[0], intervals[i][0]);
        newInterval[1] = max(newInterval[1], intervals[i][1]);
        i++;
    }
    res.push_back(newInterval);

    // 3. Add remaining
    while (i < n) res.push_back(intervals[i++]);
    return res;
}
```

### Meeting Rooms I — Can attend all?
```cpp
// Sort by start. Check if any consecutive pair overlaps.
bool canAttendMeetings(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    for (int i = 1; i < intervals.size(); i++)
        if (intervals[i][0] < intervals[i-1][1]) return false;
    return true;
}
```

### Meeting Rooms II — Minimum rooms needed
```cpp
// Method 1: Min-heap of end times
int minMeetingRooms(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());
    priority_queue<int, vector<int>, greater<>> endTimes;  // min-heap
    for (auto& interval : intervals) {
        if (!endTimes.empty() && endTimes.top() <= interval[0])
            endTimes.pop();          // reuse this room
        endTimes.push(interval[1]); // allocate room, track its end
    }
    return endTimes.size();  // rooms in use = answer
}

// Method 2: Events (start/end as +1/-1) — sweep line
int minMeetingRooms(vector<vector<int>>& intervals) {
    vector<pair<int,int>> events;
    for (auto& i : intervals) {
        events.push_back({i[0], 1});   // start → +1
        events.push_back({i[1], -1});  // end → -1
    }
    sort(events.begin(), events.end());  // sort by time; -1 before +1 at same time
    int rooms = 0, maxRooms = 0;
    for (auto& [t, v] : events) {
        rooms += v;
        maxRooms = max(maxRooms, rooms);
    }
    return maxRooms;
}
```

### Non-Overlapping Intervals (minimum removals)
```cpp
// Greedy: sort by END time. Keep interval with earliest end.
// Discard any interval that overlaps with the last kept.
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(), [](auto& a, auto& b){ return a[1] < b[1]; });
    int removed = 0, lastEnd = INT_MIN;
    for (auto& i : intervals) {
        if (i[0] >= lastEnd) lastEnd = i[1];  // no overlap, keep
        else removed++;                         // overlap, remove current
    }
    return removed;
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [252](https://leetcode.com/problems/meeting-rooms/) | Meeting Rooms I (Premium) | Sort by start, check consecutive overlap | Any overlap → false |
| [228](https://leetcode.com/problems/summary-ranges/) | Summary Ranges | Track start of range, extend while consecutive | Transition on gap |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [56](https://leetcode.com/problems/merge-intervals/) | Merge Intervals | Sort by start, extend back()[1] on overlap | `if res.back()[1] >= cur[0]` → merge |
| [57](https://leetcode.com/problems/insert-interval/) | Insert Interval | 3-phase: before, merge, after | Overlap condition: `cur[1] >= new[0] && cur[0] <= new[1]` |
| [253](https://leetcode.com/problems/meeting-rooms-ii/) | Meeting Rooms II (Premium) | Min-heap of end times OR events sweep line | Heap size = rooms in use at any point |
| [435](https://leetcode.com/problems/non-overlapping-intervals/) | Non-Overlapping Intervals | Sort by END time. Greedy: always keep earliest-ending | This maximizes non-overlapping intervals kept |
| [452](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/) | Minimum Arrows to Burst Balloons | Same as non-overlapping — sort by end, merge when overlap | Each arrow covers the common end region |
| [986](https://leetcode.com/problems/interval-list-intersections/) | Interval List Intersections | Two pointers on two sorted lists. Intersection = [max(a,c), min(b,d)] | Advance pointer with smaller end |
| [1288](https://leetcode.com/problems/remove-covered-intervals/) | Remove Covered Intervals | Sort by start (desc by end on tie). A covers B if A.start ≤ B.start and A.end ≥ B.end | Count not covered |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [759](https://leetcode.com/problems/employee-free-time/) | Employee Free Time (Premium) | Merge all intervals, find gaps | Sort all intervals together, find gaps between merged |
| [715](https://leetcode.com/problems/range-module/) | Range Module | Sorted map of intervals. Add/remove/query with upper_bound | `map<int,int>` where key=start, value=end |

---

## Sweep Line Technique

```
Convert interval events to timestamped events:
  start → +1
  end   → -1

Sort events by time (tie-break: -1 before +1 if end=start means no overlap)
Sweep through: running sum = number of active intervals at each moment.
Maximum running sum = answer for Meeting Rooms II.

This generalizes to: count points covered by most intervals, etc.
```

## Sort by Start vs Sort by End

```
Sort by START when:
  - Merging intervals (you process left to right, extending rightward)
  - Inserting an interval (find where it belongs)

Sort by END when:
  - Greedy interval scheduling (keep earliest-ending non-overlapping)
  - Minimum removals (#435)
  - Minimum arrows (#452)

Intuition for sort-by-end greedy:
  Keeping the interval that ends earliest leaves maximum room for future intervals.
```

## Common Mistakes

- Merge Intervals: not sorting first — adjacent intervals might not overlap after sorting
- Non-Overlapping: sorting by START instead of END — gives wrong greedy choice
- Meeting Rooms II: at a tie (one ends when another starts), if the problem says non-overlapping → end before start means okay, so sort with `-1` events before `+1`
- Insert Interval: forgetting the 3-phase structure — trying to do it in one loop leads to edge case bugs

## Follow-up Interviewers Love

- "Meeting Rooms II — two approaches?" → Min-heap and sweep line. Explain both.
- "Non-overlapping vs meeting rooms — what's the relationship?" → `intervals - removals = max non-overlapping = min rooms needed` when every interval must be attended
- "What if intervals are given as a stream?" → Use a balanced BST (ordered_set) or segment tree
