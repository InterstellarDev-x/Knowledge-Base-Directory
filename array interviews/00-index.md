# Array Interview Master Index

## Pattern Files

| File | Pattern | Difficulty | Most Asked At |
|------|---------|-----------|--------------|
| [01-sliding-window.md](01-sliding-window.md) | Sliding Window | Easy–Hard | All companies |
| [02-two-pointers.md](02-two-pointers.md) | Two Pointers | Easy–Medium | Meta, Amazon, Google |
| [03-prefix-sum.md](03-prefix-sum.md) | Prefix Sum | Easy–Medium | Amazon, Google, Meta |
| [04-kadane-variants.md](04-kadane-variants.md) | Kadane's + Variants | Medium–Hard | All companies |
| [05-dutch-flag-partition.md](05-dutch-flag-partition.md) | Dutch National Flag + Partition Algorithms | Easy–Medium | Microsoft, Amazon |
| [06-interval-problems.md](06-interval-problems.md) | Interval / Meeting Rooms | Medium–Hard | Google, Amazon, Bloomberg |
| [07-in-place-tricks.md](07-in-place-tricks.md) | In-Place / Cyclic Sort / Floyd's | Medium–Hard | Amazon, Microsoft |
| [08-greedy-array.md](08-greedy-array.md) | Greedy on Arrays | Medium–Hard | Amazon, Google |
| [09-company-questions.md](09-company-questions.md) | Company-Specific Lists (2024–2025) | All | All |

---

## Pattern Decision Tree

```
Contiguous subarray/substring with condition?  → Sliding Window
Pairs/triplets in sorted array?                → Two Pointers
Range sum query / count subarrays with sum k?  → Prefix Sum
Maximum/minimum sum subarray?                  → Kadane's
3-way partition (0s, 1s, 2s)?                  → Dutch National Flag
Overlapping intervals → merge/count?           → Sort by start + sweep
Find missing/duplicate with O(1) space?        → Cyclic Sort / Floyd's
Jump game / greedy decisions left→right?       → Greedy
```

## Must-Do 15

```
#53   Maximum Subarray                → Kadane's
#1    Two Sum                         → Hashmap
#15   3Sum                            → Two Pointers
#11   Container With Most Water       → Two Pointers
#42   Trapping Rain Water             → Two Pointers
#560  Subarray Sum Equals K           → Prefix Sum + Hashmap
#238  Product of Array Except Self    → Prefix/Suffix product
#56   Merge Intervals                 → Interval
#76   Minimum Window Substring        → Sliding Window
#239  Sliding Window Maximum          → Monotonic Deque
#152  Maximum Product Subarray        → Kadane variant
#75   Sort Colors                     → Dutch National Flag
#287  Find the Duplicate Number       → Floyd's Cycle
#41   First Missing Positive          → Cyclic Sort
#55   Jump Game                       → Greedy
```
