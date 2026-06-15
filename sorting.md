# Sorting Patterns — LeetCode C++ Reference

## How to Decide Which Sort / Pattern

```
Need a stable, general-purpose sort?                → std::sort (introsort, O(n log n))
Elements in small known range (0 to k)?             → Counting Sort O(n+k)
Sort by multiple keys / custom comparator?          → std::sort + lambda
Find kth largest/smallest efficiently?              → QuickSelect O(n) avg
Intervals: merge overlapping, check conflicts?      → Sort by start + sweep
Sort nearly-sorted or small arrays?                 → Insertion Sort
Count inversions / merge two sorted halves?         → Merge Sort
Cyclic array with 0,1,2 values?                     → Dutch National Flag
Relative order within groups (even/odd, pos/neg)?  → Two-pointer partition
```

**The one question to always ask first:**
> "Do I actually need a full sort, or do I just need the kth element / a specific order?"
> — Full sort is O(n log n). QuickSelect, counting sort, or a heap can often do better.

---

## C++ Sorting Quick Reference

```cpp
#include <algorithm>

vector<int> v = {5, 2, 8, 1, 9};

sort(v.begin(), v.end());                        // ascending  O(n log n)
sort(v.begin(), v.end(), greater<int>());        // descending
sort(v.begin(), v.end(), [](int a, int b) {      // custom comparator
    return a < b;
});

// Partial sort — only first k elements sorted
partial_sort(v.begin(), v.begin() + k, v.end());

// nth_element — kth element in sorted position, rest unsorted O(n) avg
nth_element(v.begin(), v.begin() + k, v.end());
// v[k] is now the (k+1)th smallest element

// Check if sorted
is_sorted(v.begin(), v.end());

// Stable sort — preserves relative order of equal elements
stable_sort(v.begin(), v.end());
```

---

## Algorithm 1: Bubble Sort

**When to think about it:**
- Almost never in production — O(n²)
- Useful to know: can detect if array is already sorted in O(n)
- Stable sort — equal elements keep their original order

**Core insight:** Repeatedly bubble the largest unsorted element to its correct position.
One pass without any swap → array is sorted.

```cpp
void bubbleSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        bool swapped = false;
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        if (!swapped) break;   // already sorted — O(n) best case
    }
}
```

| Best | Average | Worst | Space | Stable |
|------|---------|-------|-------|--------|
| O(n) | O(n²)   | O(n²) | O(1)  | Yes    |

---

## Algorithm 2: Selection Sort

**When to think about it:**
- Almost never — O(n²) always
- Minimum number of swaps (exactly n-1) — useful when write cost is high
- Not stable

**Core insight:** Find the minimum of the unsorted portion, swap it to its position.
Unlike bubble, selection sort always does exactly n-1 swaps regardless of input.

```cpp
void selectionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        int minIdx = i;
        for (int j = i + 1; j < n; j++)
            if (arr[j] < arr[minIdx]) minIdx = j;
        swap(arr[i], arr[minIdx]);
    }
}
```

| Best  | Average | Worst | Space | Stable |
|-------|---------|-------|-------|--------|
| O(n²) | O(n²)   | O(n²) | O(1)  | No     |

---

## Algorithm 3: Insertion Sort

**When to think about it:**
- Small arrays (n ≤ 20) — very fast in practice due to low overhead
- Nearly sorted arrays — O(n) best case
- Online algorithm — can sort as elements arrive
- Used internally by std::sort for small subarrays

**Core insight:** Build sorted array one element at a time. Each new element is
"inserted" into its correct position by shifting larger elements right.

```cpp
void insertionSort(vector<int>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];   // shift right
            j--;
        }
        arr[j + 1] = key;          // insert at correct position
    }
}
```

| Best | Average | Worst | Space | Stable |
|------|---------|-------|-------|--------|
| O(n) | O(n²)   | O(n²) | O(1)  | Yes    |

---

## Algorithm 4: Merge Sort

**When to think about it:**
- Need guaranteed O(n log n) — quicksort has O(n²) worst case
- Count inversions in an array
- Sort a linked list (random access not needed)
- External sorting (data too large for memory)
- Stable sort required

**Core insight:** Divide array in half, sort each half recursively, merge two sorted halves.
The merge step does all the real work — runs in O(n) per level, O(log n) levels total.

```
[5, 2, 8, 1, 9, 3]
   /              \
[5, 2, 8]      [1, 9, 3]
  /    \          /    \
[5,2] [8]       [1,9]  [3]
 / \              / \
[5] [2]          [1] [9]

merge up:
[2,5] [8]       [1,9] [3]
[2,5,8]         [1,3,9]
[1,2,3,5,8,9]
```

```cpp
void merge(vector<int>& arr, int l, int mid, int r) {
    vector<int> left(arr.begin() + l, arr.begin() + mid + 1);
    vector<int> right(arr.begin() + mid + 1, arr.begin() + r + 1);

    int i = 0, j = 0, k = l;
    while (i < left.size() && j < right.size())
        arr[k++] = (left[i] <= right[j]) ? left[i++] : right[j++];
    while (i < left.size())  arr[k++] = left[i++];
    while (j < right.size()) arr[k++] = right[j++];
}

void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    merge(arr, l, mid, r);
}
```

### Count Inversions (classic merge sort application)
```cpp
// Inversion: i < j but arr[i] > arr[j]
// Count during merge: when right[j] < left[i], ALL remaining left elements
// are greater than right[j] → add (left.size() - i) inversions

long long mergeCount(vector<int>& arr, int l, int r) {
    if (l >= r) return 0;
    int mid = l + (r - l) / 2;
    long long count = mergeCount(arr, l, mid) + mergeCount(arr, mid + 1, r);

    vector<int> left(arr.begin() + l, arr.begin() + mid + 1);
    vector<int> right(arr.begin() + mid + 1, arr.begin() + r + 1);

    int i = 0, j = 0, k = l;
    while (i < left.size() && j < right.size()) {
        if (left[i] <= right[j]) arr[k++] = left[i++];
        else {
            count += left.size() - i;   // all remaining left > right[j]
            arr[k++] = right[j++];
        }
    }
    while (i < left.size())  arr[k++] = left[i++];
    while (j < right.size()) arr[k++] = right[j++];
    return count;
}
```

| Best       | Average    | Worst      | Space  | Stable |
|------------|------------|------------|--------|--------|
| O(n log n) | O(n log n) | O(n log n) | O(n)   | Yes    |

---

## Algorithm 5: Quick Sort

**When to think about it:**
- Best average-case performance in practice (cache friendly)
- In-place sorting (O(log n) stack space)
- std::sort uses introsort (quicksort + heapsort + insertion sort)
- Partition logic is the key building block for QuickSelect

**Core insight:** Pick a pivot. Partition array so everything left of pivot is smaller,
everything right is larger. Pivot is now in its final position. Recurse on both sides.

```
[3, 6, 8, 10, 1, 2, 1]  pivot = 1 (last element)
 ^                   ^
Partition: [1, 1, 8, 10, 6, 2, 3] → pivot 1 at index 0
                                     recurse on [8,10,6,2,3]
```

```cpp
int partition(vector<int>& arr, int l, int r) {
    int pivot = arr[r];   // choose last element as pivot
    int i = l - 1;        // i tracks boundary of smaller elements

    for (int j = l; j < r; j++) {
        if (arr[j] <= pivot) {
            i++;
            swap(arr[i], arr[j]);
        }
    }
    swap(arr[i + 1], arr[r]);   // put pivot in correct position
    return i + 1;               // return pivot index
}

void quickSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int pivotIdx = partition(arr, l, r);
    quickSort(arr, l, pivotIdx - 1);
    quickSort(arr, pivotIdx + 1, r);
}

// Randomized pivot — avoids O(n²) worst case on sorted input
int partitionRandom(vector<int>& arr, int l, int r) {
    int randIdx = l + rand() % (r - l + 1);
    swap(arr[randIdx], arr[r]);
    return partition(arr, l, r);
}
```

| Best       | Average    | Worst | Space    | Stable |
|------------|------------|-------|----------|--------|
| O(n log n) | O(n log n) | O(n²) | O(log n) | No     |

---

## Algorithm 6: Heap Sort

**When to think about it:**
- Guaranteed O(n log n) with O(1) space (unlike merge sort)
- When you need in-place + guaranteed worst case
- Finding k largest/smallest elements (partial heap sort)

**Core insight:** Build a max-heap, then repeatedly extract max to end of array.
Two phases: heapify O(n), then n extractions each O(log n).

```cpp
void heapify(vector<int>& arr, int n, int i) {
    int largest = i;
    int left = 2 * i + 1, right = 2 * i + 2;

    if (left  < n && arr[left]  > arr[largest]) largest = left;
    if (right < n && arr[right] > arr[largest]) largest = right;

    if (largest != i) {
        swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

void heapSort(vector<int>& arr) {
    int n = arr.size();

    // Build max-heap — start from last non-leaf node
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    // Extract elements one by one
    for (int i = n - 1; i > 0; i--) {
        swap(arr[0], arr[i]);     // move current max to end
        heapify(arr, i, 0);       // restore heap for reduced size
    }
}
```

| Best       | Average    | Worst      | Space | Stable |
|------------|------------|------------|-------|--------|
| O(n log n) | O(n log n) | O(n log n) | O(1)  | No     |

---

## Algorithm 7: Counting Sort

**When to think about it:**
- Elements are integers in a **known, small range** [0, k]
- "Sort characters", "sort digits", "sort array with values 0 to 100"
- k is small — O(n + k) beats O(n log n) when k << n

**Core insight:** Count frequency of each value, then reconstruct sorted array.
No comparisons needed — this is why it beats the O(n log n) comparison-sort lower bound.

```
arr = [4, 2, 2, 8, 3, 3, 1]
count[1]=1, count[2]=2, count[3]=2, count[4]=1, count[8]=1
output: [1, 2, 2, 3, 3, 4, 8]
```

```cpp
void countingSort(vector<int>& arr, int maxVal) {
    vector<int> count(maxVal + 1, 0);

    for (int x : arr) count[x]++;

    int idx = 0;
    for (int val = 0; val <= maxVal; val++)
        while (count[val]-- > 0) arr[idx++] = val;
}

// Stable counting sort (preserves relative order — needed for radix sort)
vector<int> countingSortStable(vector<int>& arr, int maxVal) {
    vector<int> count(maxVal + 1, 0);
    for (int x : arr) count[x]++;
    for (int i = 1; i <= maxVal; i++) count[i] += count[i - 1];  // prefix sum

    vector<int> output(arr.size());
    for (int i = arr.size() - 1; i >= 0; i--)  // reverse for stability
        output[--count[arr[i]]] = arr[i];
    return output;
}
```

| Best    | Average | Worst   | Space  | Stable |
|---------|---------|---------|--------|--------|
| O(n+k)  | O(n+k)  | O(n+k)  | O(n+k) | Yes    |

---

## Algorithm 8: Radix Sort

**When to think about it:**
- Sort integers or strings by digit/character positions
- Large numbers but manageable number of digits
- O(d * (n + k)) where d = number of digits, k = base (10 or 26)

**Core insight:** Sort by least significant digit first, using stable counting sort.
After d passes, the array is fully sorted.

```
arr = [170, 45, 75, 90, 802, 24, 2, 66]

Sort by ones:  [170, 90, 802, 2, 24, 45, 75, 66]
Sort by tens:  [802, 2, 24, 45, 66, 170, 75, 90]
Sort by hundreds: [2, 24, 45, 66, 75, 90, 170, 802]
```

```cpp
void radixSort(vector<int>& arr) {
    int maxVal = *max_element(arr.begin(), arr.end());

    for (int exp = 1; maxVal / exp > 0; exp *= 10) {
        vector<int> output(arr.size());
        vector<int> count(10, 0);

        for (int x : arr) count[(x / exp) % 10]++;
        for (int i = 1; i < 10; i++) count[i] += count[i - 1];
        for (int i = arr.size() - 1; i >= 0; i--) {
            int digit = (arr[i] / exp) % 10;
            output[--count[digit]] = arr[i];
        }
        arr = output;
    }
}
```

| Best      | Average   | Worst     | Space  | Stable |
|-----------|-----------|-----------|--------|--------|
| O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes    |

---

## Pattern 1: QuickSelect — Kth Largest / Smallest

**When to think about it:**
- "Find kth largest element"
- "Find median"
- Don't need full sort — just the kth position
- O(n) average, O(n²) worst (use randomized pivot)

**Core insight:** After partitioning, pivot is at its final index.
If pivot index == k, we're done. Otherwise recurse on ONE side only (not both like quicksort).

```
Find 3rd largest in [3,2,1,5,6,4]
After partition with pivot=4: [3,2,1,4,6,5] → pivot at index 3
kth largest = index (n-k) from left = index 3 → need index 3
Done! answer = 4
```

```cpp
int quickSelect(vector<int>& nums, int l, int r, int k) {
    if (l == r) return nums[l];

    // Randomized pivot
    int pivotIdx = l + rand() % (r - l + 1);
    swap(nums[pivotIdx], nums[r]);

    int pivot = nums[r], i = l - 1;
    for (int j = l; j < r; j++)
        if (nums[j] <= pivot) swap(nums[++i], nums[j]);
    swap(nums[++i], nums[r]);

    if (i == k) return nums[i];
    if (i < k)  return quickSelect(nums, i + 1, r, k);
    return          quickSelect(nums, l, i - 1, k);
}

int findKthLargest(vector<int>& nums, int k) {
    return quickSelect(nums, 0, nums.size() - 1, nums.size() - k);
}

// Alternative: use a min-heap of size k — O(n log k)
int findKthLargestHeap(vector<int>& nums, int k) {
    priority_queue<int, vector<int>, greater<int>> pq;  // min-heap
    for (int x : nums) {
        pq.push(x);
        if (pq.size() > k) pq.pop();   // keep only k largest
    }
    return pq.top();
}
```

**When to use QuickSelect vs Heap:**
```
QuickSelect: O(n) avg — best when you can modify the array
Heap of k:   O(n log k) — best when streaming data or can't modify array
```

**Problems:** #215 Kth Largest Element, #347 Top K Frequent Elements, #692 Top K Frequent Words, #973 K Closest Points to Origin

---

## Pattern 2: Custom Comparator Sorting

**When to think about it:**
- Sort objects by multiple fields
- Non-standard ordering (sort by frequency, then alphabetically)
- "Largest number", "meeting rooms", "task scheduler"

**Core insight:** Define what "a should come before b" means.
Be careful: comparator must be a **strict weak ordering** (irreflexive, asymmetric, transitive).
`comp(a, a)` must return `false`, else undefined behavior.

```cpp
// Sort intervals by start time
vector<vector<int>> intervals = {{1,3},{0,2},{5,7}};
sort(intervals.begin(), intervals.end(), [](auto& a, auto& b) {
    return a[0] < b[0];   // sort by start time
});

// Sort strings: shorter first, alphabetical for ties
sort(words.begin(), words.end(), [](const string& a, const string& b) {
    return a.size() != b.size() ? a.size() < b.size() : a < b;
});

// Sort by frequency descending, then by value ascending
sort(nums.begin(), nums.end(), [&](int a, int b) {
    return freq[a] != freq[b] ? freq[a] > freq[b] : a < b;
});

// Largest Number (#179) — sort so concatenation is maximized
sort(strs.begin(), strs.end(), [](const string& a, const string& b) {
    return a + b > b + a;   // compare concatenation order
});
```

### Sort Structs / Pairs
```cpp
// Sort pairs: first by second element desc, then first element asc
vector<pair<int,int>> v = {{1,3},{2,1},{3,3},{4,2}};
sort(v.begin(), v.end(), [](auto& a, auto& b) {
    return a.second != b.second ? a.second > b.second : a.first < b.first;
});

// Custom struct sort
struct Job { int start, end, profit; };
vector<Job> jobs;
sort(jobs.begin(), jobs.end(), [](const Job& a, const Job& b) {
    return a.end < b.end;   // sort by end time (for interval DP)
});
```

**Problems:** #179 Largest Number, #56 Merge Intervals, #252 Meeting Rooms, #1235 Max Profit in Job Scheduling

---

## Pattern 3: Interval Sorting — Merge / Schedule

**When to think about it:**
- "Merge overlapping intervals"
- "Minimum meeting rooms needed"
- "Non-overlapping intervals" / "remove minimum intervals"
- Sort by start time, then sweep

**Core insight:** After sorting by start time, intervals can only overlap with their
immediate predecessor in the sorted order. One linear sweep is enough.

### Merge Overlapping Intervals
```cpp
vector<vector<int>> merge(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end());  // sort by start
    vector<vector<int>> res;

    for (auto& interval : intervals) {
        // No overlap with last — add new interval
        if (res.empty() || res.back()[1] < interval[0])
            res.push_back(interval);
        else
            // Overlap — extend the end of last interval
            res.back()[1] = max(res.back()[1], interval[1]);
    }
    return res;
}
```

### Meeting Rooms II (minimum rooms needed)
```cpp
int minMeetingRooms(vector<vector<int>>& intervals) {
    vector<int> starts, ends;
    for (auto& i : intervals) { starts.push_back(i[0]); ends.push_back(i[1]); }
    sort(starts.begin(), starts.end());
    sort(ends.begin(), ends.end());

    int rooms = 0, maxRooms = 0, j = 0;
    for (int i = 0; i < starts.size(); i++) {
        if (starts[i] < ends[j]) rooms++;    // new meeting starts before one ends
        else { rooms--; j++; }               // a meeting ended, free a room
        maxRooms = max(maxRooms, rooms);
    }
    return maxRooms;
}
```

### Non-overlapping Intervals (minimum removals)
```cpp
int eraseOverlapIntervals(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(), [](auto& a, auto& b) {
        return a[1] < b[1];   // sort by END time (greedy: keep earliest-ending)
    });

    int removed = 0, lastEnd = INT_MIN;
    for (auto& interval : intervals) {
        if (interval[0] >= lastEnd) lastEnd = interval[1];  // no overlap, keep
        else removed++;                                       // overlap, remove
    }
    return removed;
}
```

**Problems:** #56 Merge Intervals, #57 Insert Interval, #252 Meeting Rooms, #253 Meeting Rooms II, #435 Non-overlapping Intervals, #452 Min Arrows to Burst Balloons

---

## Pattern 4: Dutch National Flag (3-Way Partition)

**When to think about it:**
- "Sort array of 0s, 1s, and 2s"
- Partition array into exactly 3 groups in one pass
- "Move all zeros to left, all twos to right"

**Core insight:** Three pointers: `low`, `mid`, `high`.
- Everything before `low` = 0
- `low` to `mid-1` = 1
- `high+1` to end = 2
- `mid` to `high` = unprocessed

```
[2,0,2,1,1,0]
 l           h
 m

Step: arr[mid]=2 → swap with high, high--
Step: arr[mid]=0 → swap with low, low++, mid++
...until mid > high
```

```cpp
void sortColors(vector<int>& nums) {
    int low = 0, mid = 0, high = nums.size() - 1;

    while (mid <= high) {
        if (nums[mid] == 0) {
            swap(nums[low++], nums[mid++]);  // 0: send to front
        } else if (nums[mid] == 1) {
            mid++;                           // 1: already in middle
        } else {
            swap(nums[mid], nums[high--]);   // 2: send to back (don't increment mid)
        }
    }
}
```

**Why not increment `mid` when swapping with `high`:**
The element swapped from `high` is unknown — it needs to be processed.
But the element swapped from `low` is always 1 (since `low` only advances past processed 1s),
so `mid` can safely advance.

**Problems:** #75 Sort Colors, #280 Wiggle Sort, #905 Sort Array by Parity

---

## Pattern 5: Cyclic Sort

**When to think about it:**
- Array contains numbers in range [1, n] or [0, n]
- "Find missing number", "find all duplicates", "find the duplicate"
- Each number belongs at a specific index — place it there directly

**Core insight:** Number `x` belongs at index `x-1` (for 1-indexed).
Keep swapping until each number is at its correct index. One pass O(n).

```
arr = [3, 1, 5, 4, 2]
idx:   0  1  2  3  4

i=0: arr[0]=3 → should be at index 2, swap(arr[0], arr[2]) → [5,1,3,4,2]
i=0: arr[0]=5 → should be at index 4, swap(arr[0], arr[4]) → [2,1,3,4,5]
i=0: arr[0]=2 → should be at index 1, swap(arr[0], arr[1]) → [1,2,3,4,5]
i=0: arr[0]=1 → correct, i++
...
```

```cpp
// Place each number at its correct index
void cyclicSort(vector<int>& nums) {
    int i = 0;
    while (i < nums.size()) {
        int correct = nums[i] - 1;           // where nums[i] should go
        if (nums[i] != nums[correct])
            swap(nums[i], nums[correct]);    // put it there
        else
            i++;                             // already correct, move on
    }
}

// Find missing number
int missingNumber(vector<int>& nums) {
    int i = 0;
    while (i < nums.size()) {
        if (nums[i] < nums.size() && nums[i] != nums[nums[i]])
            swap(nums[i], nums[nums[i]]);
        else i++;
    }
    for (int i = 0; i < nums.size(); i++)
        if (nums[i] != i) return i;
    return nums.size();
}

// Find all duplicates
vector<int> findDuplicates(vector<int>& nums) {
    int i = 0;
    while (i < nums.size()) {
        int correct = nums[i] - 1;
        if (nums[i] != nums[correct]) swap(nums[i], nums[correct]);
        else i++;
    }
    vector<int> res;
    for (int i = 0; i < nums.size(); i++)
        if (nums[i] != i + 1) res.push_back(nums[i]);
    return res;
}
```

**Problems:** #268 Missing Number, #448 Find All Disappeared Numbers, #442 Find All Duplicates, #41 First Missing Positive, #287 Find the Duplicate Number

---

## Pattern 6: Sort + Two Pointer

**When to think about it:**
- "Two sum in sorted array", "three sum", "four sum"
- "Container with most water", "closest pair"
- Sort first, then use two pointers to find pairs efficiently

**Core insight:** After sorting, you can binary search or two-pointer sweep.
For pair sum: left pointer starts at 0, right at end. If sum < target, move left right.
If sum > target, move right left.

```cpp
// Two Sum in sorted array
vector<int> twoSumSorted(vector<int>& nums, int target) {
    int l = 0, r = nums.size() - 1;
    while (l < r) {
        int sum = nums[l] + nums[r];
        if (sum == target) return {l, r};
        if (sum < target) l++;
        else r--;
    }
    return {};
}

// Three Sum (all unique triplets summing to 0)
vector<vector<int>> threeSum(vector<int>& nums) {
    sort(nums.begin(), nums.end());
    vector<vector<int>> res;

    for (int i = 0; i < nums.size() - 2; i++) {
        if (i > 0 && nums[i] == nums[i-1]) continue;   // skip duplicate i

        int l = i + 1, r = nums.size() - 1;
        while (l < r) {
            int sum = nums[i] + nums[l] + nums[r];
            if (sum == 0) {
                res.push_back({nums[i], nums[l], nums[r]});
                while (l < r && nums[l] == nums[l+1]) l++;  // skip duplicate l
                while (l < r && nums[r] == nums[r-1]) r--;  // skip duplicate r
                l++; r--;
            } else if (sum < 0) l++;
            else r--;
        }
    }
    return res;
}
```

**Problems:** #167 Two Sum II, #15 3Sum, #18 4Sum, #16 3Sum Closest, #259 3Sum Smaller

---

## The 5 Most Important Sort Templates to Memorize

```cpp
// 1. MERGE SORT (stable, guaranteed O(n log n))
void mergeSort(vector<int>& arr, int l, int r) {
    if (l >= r) return;
    int mid = l + (r - l) / 2;
    mergeSort(arr, l, mid);
    mergeSort(arr, mid + 1, r);
    // merge arr[l..mid] and arr[mid+1..r]
    vector<int> tmp;
    int i = l, j = mid + 1;
    while (i <= mid && j <= r)
        tmp.push_back(arr[i] <= arr[j] ? arr[i++] : arr[j++]);
    while (i <= mid)  tmp.push_back(arr[i++]);
    while (j <= r)    tmp.push_back(arr[j++]);
    for (int k = l; k <= r; k++) arr[k] = tmp[k - l];
}

// 2. QUICKSELECT (kth smallest/largest in O(n) avg)
int quickSelect(vector<int>& a, int l, int r, int k) {
    int pivot = a[r], i = l - 1;
    for (int j = l; j < r; j++) if (a[j] <= pivot) swap(a[++i], a[j]);
    swap(a[++i], a[r]);
    if (i == k) return a[i];
    return i < k ? quickSelect(a, i+1, r, k) : quickSelect(a, l, i-1, k);
}

// 3. DUTCH NATIONAL FLAG (sort 0s, 1s, 2s in one pass)
void dnf(vector<int>& a) {
    int lo = 0, mid = 0, hi = a.size() - 1;
    while (mid <= hi) {
        if      (a[mid] == 0) swap(a[lo++], a[mid++]);
        else if (a[mid] == 1) mid++;
        else                  swap(a[mid], a[hi--]);
    }
}

// 4. MERGE INTERVALS (sort by start, sweep)
vector<vector<int>> mergeIntervals(vector<vector<int>>& v) {
    sort(v.begin(), v.end());
    vector<vector<int>> res;
    for (auto& i : v) {
        if (res.empty() || res.back()[1] < i[0]) res.push_back(i);
        else res.back()[1] = max(res.back()[1], i[1]);
    }
    return res;
}

// 5. CYCLIC SORT (place each number at its correct index)
void cyclicSort(vector<int>& a) {
    int i = 0;
    while (i < a.size()) {
        int c = a[i] - 1;
        if (a[i] != a[c]) swap(a[i], a[c]);
        else i++;
    }
}
```

---

## Universal Checklist Before Coding

- [ ] Do I need full sort or just kth element? → QuickSelect / partial_sort
- [ ] Are values in a small range? → Counting Sort
- [ ] Need stable sort (preserve relative order)? → Merge Sort / stable_sort
- [ ] Interval problem? → Sort by start time first
- [ ] 0/1/2 values only? → Dutch National Flag
- [ ] Numbers in range [1,n] → cyclic sort
- [ ] Custom ordering? → lambda comparator (ensure strict weak ordering)
- [ ] `comp(a, a)` must return `false` — or you get undefined behavior

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 75   | Sort Colors                         | Medium | Dutch National Flag      |
| 2  | 215  | Kth Largest Element in Array        | Medium | QuickSelect / Heap       |
| 3  | 56   | Merge Intervals                     | Medium | Sort + sweep             |
| 4  | 179  | Largest Number                      | Medium | Custom comparator        |
| 5  | 15   | 3Sum                                | Medium | Sort + two pointer       |
| 6  | 268  | Missing Number                      | Easy   | Cyclic sort / XOR        |
| 7  | 435  | Non-overlapping Intervals           | Medium | Sort by end + greedy     |
| 8  | 347  | Top K Frequent Elements             | Medium | Heap / bucket sort       |
| 9  | 253  | Meeting Rooms II                    | Medium | Sort starts + ends       |
| 10 | 973  | K Closest Points to Origin          | Medium | QuickSelect / Heap       |
| 11 | 148  | Sort List                           | Medium | Merge sort on linked list|
| 12 | 442  | Find All Duplicates in Array        | Medium | Cyclic sort              |
| 13 | 41   | First Missing Positive              | Hard   | Cyclic sort              |
| 14 | 493  | Reverse Pairs                       | Hard   | Merge sort + count       |
| 15 | 315  | Count of Smaller Numbers After Self | Hard   | Merge sort + index       |
| 16 | 57   | Insert Interval                     | Medium | Interval merge           |
| 17 | 452  | Min Arrows to Burst Balloons        | Medium | Sort by end + greedy     |
| 18 | 287  | Find the Duplicate Number           | Medium | Cyclic sort / Floyd's    |
| 19 | 912  | Sort an Array                       | Medium | Merge / Quick / Heap     |
| 20 | 164  | Maximum Gap                         | Hard   | Radix sort / bucket      |

**Study order:** #75 -> #215 -> #56 -> #15 -> #268 -> #435 -> #347 -> #41 -> #493
These nine cover every pattern including hard applications.

---

## Algorithm Comparison Table

| Algorithm      | Best       | Average    | Worst      | Space    | Stable | Use When |
|----------------|------------|------------|------------|----------|--------|----------|
| Bubble Sort    | O(n)       | O(n²)      | O(n²)      | O(1)     | Yes    | Nearly sorted, detect sorted |
| Selection Sort | O(n²)      | O(n²)      | O(n²)      | O(1)     | No     | Minimize swaps |
| Insertion Sort | O(n)       | O(n²)      | O(n²)      | O(1)     | Yes    | Small or nearly sorted |
| Merge Sort     | O(n log n) | O(n log n) | O(n log n) | O(n)     | Yes    | Stable, guaranteed, linked list |
| Quick Sort     | O(n log n) | O(n log n) | O(n²)      | O(log n) | No     | General purpose, in-place |
| Heap Sort      | O(n log n) | O(n log n) | O(n log n) | O(1)     | No     | In-place + guaranteed |
| Counting Sort  | O(n+k)     | O(n+k)     | O(n+k)     | O(k)     | Yes    | Small integer range |
| Radix Sort     | O(d(n+k))  | O(d(n+k))  | O(d(n+k))  | O(n+k)   | Yes    | Fixed-length integers/strings |

---

## Company-Wise Most Asked Problems

| Company | Most Frequently Asked |
|---------|----------------------|
| **Amazon** | #56 Merge Intervals, #215 Kth Largest, #347 Top K Frequent, #75 Sort Colors |
| **Google** | #315 Count Smaller, #493 Reverse Pairs, #179 Largest Number, #973 K Closest |
| **Meta** | #56 Merge Intervals, #215 Kth Largest, #253 Meeting Rooms II, #15 3Sum |
| **Microsoft** | #75 Sort Colors, #56 Merge Intervals, #148 Sort List, #268 Missing Number |
| **Bloomberg** | #56 Merge Intervals, #57 Insert Interval, #253 Meeting Rooms II, #179 Largest Number |
| **Uber** | #973 K Closest Points, #215 Kth Largest, #253 Meeting Rooms II |

---

## Recent Interview Trends (2023–2025)

### 1. Merge Intervals is the single most asked sort problem
#56 appears at Amazon, Google, Meta, Bloomberg. Know it cold.
Variants: insert interval (#57), non-overlapping (#435), meeting rooms (#252/#253).

### 2. Kth element in two forms — QuickSelect AND heap
Companies ask #215 both ways. QuickSelect for O(n) avg, heap for O(n log k).
Know when to use each — interviewers ask "can you do better?" after the heap solution.

### 3. Merge sort applications over merge sort itself
#493 Reverse Pairs and #315 Count of Smaller Numbers are Google favorites —
they use merge sort's counting logic, not just the sort. Understand the inversion count pattern.

### 4. Custom comparator design is a core skill
#179 Largest Number tests your understanding of strict weak ordering.
Always test your comparator: `comp(a,a)` must return false.

### 5. Cyclic sort is underrated but covers many "easy" problems
#268, #448, #442, #41, #287 all fall under cyclic sort.
Recognize the pattern: "array of n elements with values in [1,n]" → cyclic sort.

### 6. Bucket sort / frequency sort pattern is rising
#347 Top K Frequent (bucket sort by frequency), #451 Sort Characters by Frequency —
use an array of buckets indexed by count. O(n) for these problems.

```cpp
// Bucket sort by frequency — O(n)
vector<int> topKFrequent(vector<int>& nums, int k) {
    unordered_map<int,int> freq;
    for (int x : nums) freq[x]++;

    vector<vector<int>> buckets(nums.size() + 1);  // bucket[i] = nums with freq i
    for (auto& [val, cnt] : freq) buckets[cnt].push_back(val);

    vector<int> res;
    for (int i = buckets.size() - 1; i >= 0 && res.size() < k; i--)
        for (int x : buckets[i]) res.push_back(x);
    return res;
}
```
