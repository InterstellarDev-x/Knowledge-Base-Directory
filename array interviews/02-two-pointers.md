# Pattern 2: Two Pointers

## Identify This Pattern
- Sorted array + find pair/triplet with sum = target
- "Remove duplicates", "move zeros", "remove element in-place"
- "Container with most water", "trapping rain water"
- Two ends converging OR slow/fast pointer in same direction

## Two Sub-Patterns
```
Opposite ends (l=0, r=n-1):  converge inward. Works on SORTED arrays.
Same direction (slow/fast):  fast scans ahead, slow marks "good" position.
```

---

## Templates

### Opposite Ends — sorted array pair sum
```cpp
int l = 0, r = n - 1;
while (l < r) {
    int sum = nums[l] + nums[r];
    if (sum == target) { /* found */ l++; r--; }
    else if (sum < target) l++;
    else r--;
}
```

### Same Direction — write pointer (slow = next write position)
```cpp
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (/* keep nums[fast] */) {
        nums[slow++] = nums[fast];
    }
}
// slow = new length
```

### 3Sum (fix one + two pointers on the rest)
```cpp
sort(nums.begin(), nums.end());
for (int i = 0; i < n - 2; i++) {
    if (i > 0 && nums[i] == nums[i-1]) continue;  // skip duplicate i
    int l = i+1, r = n-1;
    while (l < r) {
        int s = nums[i] + nums[l] + nums[r];
        if (s == 0) {
            result.push_back({nums[i], nums[l], nums[r]});
            while (l < r && nums[l] == nums[l+1]) l++;  // skip dup l
            while (l < r && nums[r] == nums[r-1]) r--;  // skip dup r
            l++; r--;
        } else if (s < 0) l++;
        else r--;
    }
}
```

### Trapping Rain Water
```cpp
int l = 0, r = n-1, lMax = 0, rMax = 0, water = 0;
while (l < r) {
    if (height[l] < height[r]) {
        lMax = max(lMax, height[l]);
        water += lMax - height[l++];  // water = lMax - current (lMax is the bottleneck)
    } else {
        rMax = max(rMax, height[r]);
        water += rMax - height[r--];
    }
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [26](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) | Remove Duplicates from Sorted Array | Slow/fast: copy fast to slow when different | `slow` = last unique written |
| [27](https://leetcode.com/problems/remove-element/) | Remove Element | Slow = next valid position. Copy only non-val | Same write-pointer pattern |
| [283](https://leetcode.com/problems/move-zeroes/) | Move Zeroes | Write non-zeros to slow position, fill rest with 0 | Or swap non-zero with slow pointer |
| [167](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) | Two Sum II (sorted) | Opposite ends. Sum < target → l++, else r-- | Input IS sorted — no hashmap needed |
| [977](https://leetcode.com/problems/squares-of-sorted-array/) | Squares of Sorted Array | Two pointers from ends; place larger square at end | Squared values are largest at both ends |
| [344](https://leetcode.com/problems/reverse-string/) | Reverse String | Swap l and r, converge | Most basic two-pointer |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [15](https://leetcode.com/problems/3sum/) | 3Sum | Sort + fix i + two pointers for pair | Skip duplicates at all 3 levels |
| [16](https://leetcode.com/problems/3sum-closest/) | 3Sum Closest | Same as 3Sum but track minimum abs diff | No early exit — must find closest |
| [11](https://leetcode.com/problems/container-with-most-water/) | Container With Most Water | Move the SHORTER side inward (taller side can't help unless shorter grows) | `area = min(h[l], h[r]) * (r-l)` |
| [18](https://leetcode.com/problems/4sum/) | 4Sum | Sort + fix i + fix j + two pointers | 4 levels of dedup. O(n³) |
| [75](https://leetcode.com/problems/sort-colors/) | Sort Colors | Dutch National Flag (see Pattern 5) | Three-pointer partition |
| [80](https://leetcode.com/problems/remove-duplicates-from-sorted-array-ii/) | Remove Duplicates II (allow 2) | Keep if `nums[fast] != nums[slow-2]` | Slow pointer offset by 2 |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [42](https://leetcode.com/problems/trapping-rain-water/) | Trapping Rain Water | Two pointers from ends. Process the shorter side first | Water at position = min(lMax, rMax) - height. The side with smaller max is the bottleneck |
| [844](https://leetcode.com/problems/backspace-string-compare/) | Backspace String Compare | Two pointers from END of both strings. Skip backspaced chars | Process # by counting them; skip next char for each |

---

## Key Insight: Why Move the Shorter Side in #11

```
Container area = min(height[l], height[r]) × (r - l)
Moving the TALLER side inward:
  - width decreases by 1
  - height can only stay same or decrease (min still limited by shorter side)
  → area can only decrease or stay same → no point moving taller

Moving the SHORTER side inward:
  - width decreases by 1
  - height MIGHT increase (shorter side might find something taller)
  → area might increase → this is the only productive move
```

## Optimizations & Common Mistakes

**Common mistakes:**
- 3Sum: not skipping duplicates at l and r AFTER finding a triplet — leads to duplicate results
- Trapping Rain Water: computing water before updating lMax/rMax
- Two Sum II: using a hashmap when the array is already sorted (two pointers is O(n) and O(1) space)

**Optimizations:**
- 3Sum: early exit if `nums[i] > 0` (sorted — no triplet can sum to 0 if smallest is positive)
- 3Sum: skip if `nums[i] == nums[i-1]` avoids reprocessing same pivot
- Remove duplicates: use `slow` instead of `erase` — O(n) vs O(n²)

**Follow-up interviewers love:**
- "What's the time complexity of 3Sum?" → O(n²) after sorting
- "What if the array isn't sorted for 3Sum?" → Sort first: O(n log n). Doesn't change the O(n²) overall.
- "Can you do trapping rain water with O(1) extra space?" → Two pointers is already O(1)
- "What if we want 4Sum or kSum?" → Recursively reduce k-sum to (k-1)-sum with a fixed element
