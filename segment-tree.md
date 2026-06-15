# Segment Tree & Binary Indexed Tree (BIT/Fenwick) — LeetCode C++ Reference

## When Do You Need These?

```
Range sum queries, NO updates?                     → Prefix sum (O(1) query)
Point updates + range queries?                     → BIT / Segment Tree
Range updates + range queries?                     → Segment Tree with lazy propagation
Range min/max queries?                             → Segment Tree (BIT doesn't support this easily)
Counting inversions / rank queries?                → BIT
Dynamic range queries with inserts?                → Segment Tree
Coordinate compression needed?                     → BIT after compression
```

**The key decision:**
> BIT = simpler code, O(n log n), handles sums/counts, point update + prefix query
> Segment Tree = more powerful, handles any associative operation (min, max, sum, GCD), range update + range query

---

## Binary Indexed Tree (Fenwick Tree)

### Core Idea
Every index `i` in the BIT stores the sum of a range determined by the lowest set bit of `i`.
- Update: propagate upward via `i += i & (-i)` (add lowest set bit)
- Query: propagate downward via `i -= i & (-i)` (remove lowest set bit)

```
Indices:   1   2   3   4   5   6   7   8
           |___|   |___|   |___|   |___|
               |_______|       |_________|
                       |_________________|

BIT[4] stores sum of [1..4]
BIT[6] stores sum of [5..6]
BIT[8] stores sum of [1..8]
```

### BIT Template — Point Update, Prefix Sum Query

```cpp
struct BIT {
    int n;
    vector<int> tree;

    BIT(int n) : n(n), tree(n+1, 0) {}

    // Add val to position i (1-indexed)
    void update(int i, int val) {
        for (; i <= n; i += i & (-i))
            tree[i] += val;
    }

    // Sum of [1, i] (prefix sum)
    int query(int i) {
        int sum = 0;
        for (; i > 0; i -= i & (-i))
            sum += tree[i];
        return sum;
    }

    // Sum of [l, r] (range sum)
    int query(int l, int r) {
        return query(r) - query(l-1);
    }
};

// Usage
BIT bit(n);
bit.update(3, 5);     // add 5 at position 3
bit.query(1, 5);      // sum of positions 1..5
```

### BIT Template — Build from Array

```cpp
BIT(vector<int>& nums) : n(nums.size()), tree(nums.size()+1, 0) {
    for (int i = 0; i < nums.size(); i++)
        update(i+1, nums[i]);   // O(n log n) total
}

// OR: O(n) build
BIT buildFast(vector<int>& nums) {
    BIT bit(nums.size());
    for (int i = 1; i <= nums.size(); i++) {
        bit.tree[i] += nums[i-1];
        int j = i + (i & -i);
        if (j <= nums.size()) bit.tree[j] += bit.tree[i];
    }
    return bit;
}
```

---

## Segment Tree

### Core Idea
Binary tree where each node covers a range `[l, r]`.
- Leaf nodes: single elements
- Internal nodes: result of merge(left child, right child)
- Query / Update: O(log n)
- Flexible: works for sum, min, max, GCD, any associative operation

```
Array: [1, 3, 5, 7, 9, 11]
Segment Tree (sum):
                [1..6]=36
              /           \
        [1..3]=9       [4..6]=27
        /      \        /      \
   [1..2]=4  [3]=5  [4..5]=16  [6]=11
    /    \          /      \
  [1]=1 [2]=3   [4]=7    [5]=9
```

### Segment Tree Template — Sum (Iterative)

```cpp
struct SegTree {
    int n;
    vector<int> tree;

    SegTree(int n) : n(n), tree(2*n, 0) {}

    SegTree(vector<int>& nums) : n(nums.size()), tree(2*nums.size()) {
        // Load leaves
        for (int i = 0; i < n; i++) tree[n+i] = nums[i];
        // Build internal nodes bottom-up
        for (int i = n-1; i > 0; i--) tree[i] = tree[2*i] + tree[2*i+1];
    }

    // Update position pos (0-indexed) to val
    void update(int pos, int val) {
        tree[pos + n] = val;
        for (int i = (pos+n) >> 1; i >= 1; i >>= 1)
            tree[i] = tree[2*i] + tree[2*i+1];
    }

    // Sum of [l, r) — note: half-open interval
    int query(int l, int r) {
        int res = 0;
        for (l += n, r += n; l < r; l >>= 1, r >>= 1) {
            if (l & 1) res += tree[l++];
            if (r & 1) res += tree[--r];
        }
        return res;
    }
};
```

### Segment Tree Template — Min/Max (Recursive, more flexible)

```cpp
struct SegTree {
    int n;
    vector<int> tree;

    SegTree(int n) : n(n), tree(4*n, INT_MAX) {}

    void build(vector<int>& nums, int node, int l, int r) {
        if (l == r) { tree[node] = nums[l]; return; }
        int mid = (l + r) / 2;
        build(nums, 2*node, l, mid);
        build(nums, 2*node+1, mid+1, r);
        tree[node] = min(tree[2*node], tree[2*node+1]);  // change for max or sum
    }

    void update(int node, int l, int r, int pos, int val) {
        if (l == r) { tree[node] = val; return; }
        int mid = (l + r) / 2;
        if (pos <= mid) update(2*node, l, mid, pos, val);
        else            update(2*node+1, mid+1, r, pos, val);
        tree[node] = min(tree[2*node], tree[2*node+1]);
    }

    int query(int node, int l, int r, int ql, int qr) {
        if (ql > r || qr < l) return INT_MAX;  // out of range (identity for min)
        if (ql <= l && r <= qr) return tree[node];  // fully inside
        int mid = (l + r) / 2;
        return min(query(2*node, l, mid, ql, qr),
                   query(2*node+1, mid+1, r, ql, qr));
    }

    // Wrapper calls
    void build(vector<int>& nums) { build(nums, 1, 0, n-1); }
    void update(int pos, int val) { update(1, 0, n-1, pos, val); }
    int query(int l, int r) { return query(1, 0, n-1, l, r); }
};
```

### Segment Tree with Lazy Propagation — Range Update + Range Query

**Use when:** "Add/set a value to all elements in range [l, r], then query range sum"

```cpp
struct LazySegTree {
    int n;
    vector<long long> tree, lazy;

    LazySegTree(int n) : n(n), tree(4*n, 0), lazy(4*n, 0) {}

    void pushDown(int node, int l, int r) {
        if (lazy[node] != 0) {
            int mid = (l + r) / 2;
            tree[2*node]   += lazy[node] * (mid - l + 1);
            tree[2*node+1] += lazy[node] * (r - mid);
            lazy[2*node]   += lazy[node];
            lazy[2*node+1] += lazy[node];
            lazy[node] = 0;
        }
    }

    // Add val to all elements in [ql, qr]
    void update(int node, int l, int r, int ql, int qr, long long val) {
        if (ql > r || qr < l) return;
        if (ql <= l && r <= qr) {
            tree[node] += val * (r - l + 1);
            lazy[node] += val;
            return;
        }
        pushDown(node, l, r);
        int mid = (l + r) / 2;
        update(2*node, l, mid, ql, qr, val);
        update(2*node+1, mid+1, r, ql, qr, val);
        tree[node] = tree[2*node] + tree[2*node+1];
    }

    long long query(int node, int l, int r, int ql, int qr) {
        if (ql > r || qr < l) return 0;
        if (ql <= l && r <= qr) return tree[node];
        pushDown(node, l, r);
        int mid = (l + r) / 2;
        return query(2*node, l, mid, ql, qr) +
               query(2*node+1, mid+1, r, ql, qr);
    }

    // Wrappers (0-indexed)
    void update(int l, int r, long long val) { update(1, 0, n-1, l, r, val); }
    long long query(int l, int r) { return query(1, 0, n-1, l, r); }
};
```

---

## Pattern 1: Range Sum Queries with Updates

**BIT is better here** — simpler, same complexity.

```cpp
// 307. Range Sum Query - Mutable
class NumArray {
    BIT bit;
    vector<int> nums;

public:
    NumArray(vector<int>& nums) : bit(nums.size()), nums(nums) {
        for (int i = 0; i < nums.size(); i++)
            bit.update(i+1, nums[i]);
    }

    void update(int index, int val) {
        bit.update(index+1, val - nums[index]);  // add delta
        nums[index] = val;
    }

    int sumRange(int left, int right) {
        return bit.query(left+1, right+1);
    }
};
```

---

## Pattern 2: Counting / Rank Queries

**BIT shines here** — count elements less than x efficiently.

```cpp
// Count inversions in array — O(n log n)
// An inversion is a pair (i, j) where i < j but nums[i] > nums[j]
int countInversions(vector<int>& nums) {
    // Coordinate compress to [1, n]
    vector<int> sorted = nums;
    sort(sorted.begin(), sorted.end());
    sorted.erase(unique(sorted.begin(), sorted.end()), sorted.end());
    auto compress = [&](int x) {
        return lower_bound(sorted.begin(), sorted.end(), x) - sorted.begin() + 1;
    };

    BIT bit(sorted.size());
    int inversions = 0;
    for (int x : nums) {
        int rank = compress(x);
        // Count how many already-inserted elements are > x
        inversions += bit.query(sorted.size()) - bit.query(rank);
        bit.update(rank, 1);
    }
    return inversions;
}

// 315. Count of Smaller Numbers After Self
vector<int> countSmaller(vector<int>& nums) {
    vector<int> sorted = nums;
    sort(sorted.begin(), sorted.end());
    sorted.erase(unique(sorted.begin(), sorted.end()), sorted.end());
    int m = sorted.size();

    auto compress = [&](int x) {
        return lower_bound(sorted.begin(), sorted.end(), x) - sorted.begin() + 1;
    };

    BIT bit(m);
    vector<int> res(nums.size());
    for (int i = nums.size()-1; i >= 0; i--) {  // process right to left
        int rank = compress(nums[i]);
        res[i] = bit.query(rank-1);              // count elements < nums[i]
        bit.update(rank, 1);
    }
    return res;
}
```

---

## Pattern 3: Coordinate Compression + BIT/Segment Tree

**When to use:** Values are large (up to 10^9) but count is small (up to 10^5).
Compress values to indices [1, n] first, then use BIT.

```cpp
// Template for coordinate compression
vector<int> compress(vector<int>& nums) {
    vector<int> sorted = nums;
    sort(sorted.begin(), sorted.end());
    sorted.erase(unique(sorted.begin(), sorted.end()), sorted.end());

    vector<int> compressed(nums.size());
    for (int i = 0; i < nums.size(); i++)
        compressed[i] = lower_bound(sorted.begin(), sorted.end(), nums[i])
                        - sorted.begin() + 1;  // 1-indexed
    return compressed;
}

// 493. Reverse Pairs: count pairs where nums[i] > 2 * nums[j], i < j
int reversePairs(vector<int>& nums) {
    // Collect all unique values + 2*vals for BIT indexing
    vector<long long> vals;
    for (int x : nums) { vals.push_back(x); vals.push_back(2LL * x); }
    sort(vals.begin(), vals.end());
    vals.erase(unique(vals.begin(), vals.end()), vals.end());

    auto idx = [&](long long x) {
        return lower_bound(vals.begin(), vals.end(), x) - vals.begin() + 1;
    };

    BIT bit(vals.size());
    int res = 0;
    for (int x : nums) {
        // Count how many already-inserted j have nums[j]*2 < x
        res += bit.query(idx((long long)x - 1));  // prefix count < x
        bit.update(idx(2LL * x), 1);
    }
    return res;
}
```

---

## Pattern 4: Segment Tree for Dynamic Range Queries

**Problems that explicitly need Segment Tree (BIT can't easily do these):**
- Range min/max queries with point updates
- Range assignment / range add with range sum

```cpp
// 218. The Skyline Problem — use a multiset (or segment tree for larger inputs)
// 699. Falling Squares — range max queries with range set updates

// Generic range max with point update
struct MaxSegTree {
    int n;
    vector<int> tree;
    MaxSegTree(int n) : n(n), tree(2*n, 0) {}

    void update(int pos, int val) {
        tree[pos + n] = val;
        for (int i = (pos+n) >> 1; i >= 1; i >>= 1)
            tree[i] = max(tree[2*i], tree[2*i+1]);
    }

    int query(int l, int r) {  // [l, r)
        int res = 0;
        for (l += n, r += n; l < r; l >>= 1, r >>= 1) {
            if (l & 1) res = max(res, tree[l++]);
            if (r & 1) res = max(res, tree[--r]);
        }
        return res;
    }
};
```

---

## BIT vs Segment Tree Comparison

| Feature | BIT (Fenwick) | Segment Tree |
|---------|---------------|--------------|
| Code length | ~15 lines | ~30-50 lines |
| Space | O(n) | O(4n) |
| Build time | O(n log n) | O(n) |
| Point update | O(log n) | O(log n) |
| Range query | O(log n) | O(log n) |
| Range update | Hard (difference BIT) | Easy (lazy prop) |
| Range min/max | Not supported | Supported |
| 2D version | Possible (BIT2D) | Complex |

**Rule of thumb:**
- Use BIT for: sum/count, point update, prefix query
- Use Segment Tree for: min/max, range update, range query

---

## The 3 Templates to Memorize

```cpp
// 1. BIT — Point Update, Prefix Sum
struct BIT {
    int n; vector<int> t;
    BIT(int n) : n(n), t(n+1, 0) {}
    void update(int i, int v) { for(; i<=n; i+=i&-i) t[i]+=v; }
    int query(int i) { int s=0; for(; i>0; i-=i&-i) s+=t[i]; return s; }
    int query(int l, int r) { return query(r)-query(l-1); }
};

// 2. Segment Tree — Point Update, Range Sum (iterative)
struct SegTree {
    int n; vector<int> t;
    SegTree(int n) : n(n), t(2*n,0) {}
    void update(int p, int v) { t[p+=n]=v; for(p>>=1; p>=1; p>>=1) t[p]=t[2*p]+t[2*p+1]; }
    int query(int l, int r) {
        int s=0;
        for(l+=n,r+=n; l<r; l>>=1,r>>=1) {
            if(l&1) s+=t[l++];
            if(r&1) s+=t[--r];
        }
        return s;
    }
};

// 3. Coordinate Compression
auto compress = [](vector<int>& nums) {
    vector<int> s = nums;
    sort(s.begin(), s.end());
    s.erase(unique(s.begin(), s.end()), s.end());
    for(auto& x : nums) x = lower_bound(s.begin(), s.end(), x) - s.begin() + 1;
};
```

---

## Universal Checklist Before Coding

- [ ] Do I need point update + range query → BIT
- [ ] Do I need range update + range query → Segment Tree with lazy
- [ ] Do I need range min/max → Segment Tree (BIT doesn't support)
- [ ] Are values too large? → Coordinate compression first
- [ ] BIT is 1-indexed — remember to convert 0-indexed to 1-indexed
- [ ] Segment tree size: iterative uses `2*n`, recursive uses `4*n`
- [ ] Lazy propagation: always call `pushDown` before traversing children

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 307  | Range Sum Query - Mutable              | Medium | BIT basic             |
| 2  | 315  | Count of Smaller Numbers After Self    | Hard   | BIT + coordinate comp |
| 3  | 493  | Reverse Pairs                          | Hard   | BIT + coordinate comp |
| 4  | 327  | Count of Range Sum                     | Hard   | Seg Tree / merge sort |
| 5  | 218  | The Skyline Problem                    | Hard   | Seg Tree / multiset   |
| 6  | 699  | Falling Squares                        | Hard   | Lazy seg tree         |
| 7  | 406  | Queue Reconstruction by Height         | Medium | BIT inversion variant |
| 8  | 1649 | Create Sorted Array through Insertions | Hard   | BIT order stats       |
| 9  | 2179 | Count Good Triplets in Array           | Hard   | BIT counting          |
| 10 | 308  | Range Sum Query 2D - Mutable           | Hard   | 2D BIT                |

**Study order:** #307 → #315 → #493 → then harder problems

---

## Company-Wise Most Asked

| Company | Most Frequently Asked |
|---------|----------------------|
| **Google** | #315, #327, #218, #493 |
| **Amazon** | #307, #315, #493 |
| **Meta** | #315, #493, #218 |
| **Microsoft** | #307, #315 |

---

## Interview Reality Check

Segment Tree / BIT problems appear at:
- **Google, Meta senior/staff roles** — expected to implement from scratch
- **FAANG on-site rounds** — sometimes as follow-up after simpler approach
- **Competitive programming backgrounds** — interviewers test these directly

For **most interviews**: if a problem is solvable with a sorted structure, prefix sum, or heap — use that instead. Segment Tree is the last resort for problems where range queries need updates.

**Key signal that you need BIT/Seg Tree:**
"The array gets modified AND you need range queries" — this exact combination.

---

## Recent Trends (2023–2025)

- BIT counting inversions appears in Google phone screens
- Range update / range query (lazy seg tree) is more common in system design follow-ups
- Coordinate compression is tested independently at Amazon (they give large value arrays)
- LeetCode has added more BIT/Fenwick problems (1649, 2179) — these are now mid-frequency
