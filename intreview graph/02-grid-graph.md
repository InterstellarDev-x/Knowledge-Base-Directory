# Pattern 2: Grid as Graph (Islands / Flood Fill)

## Identify This Pattern
- 2D grid with cells (0/1, 'W'/'L', chars)
- "Number of islands", "max area", "flood fill"
- "Surrounded regions", "enclaves", "reachable from border"
- "Shortest path in grid" → BFS on grid
- "Number of distinct islands" (count unique shapes)

## Recognition Signal
> Any 2D grid where you need to explore connected regions → grid DFS/BFS.
> Each cell = graph node. Adjacent cells = edges.

---

## Templates

### Grid DFS (in-place visited marking)
```cpp
int dr[] = {-1, 1, 0, 0};
int dc[] = {0, 0, -1, 1};

void dfs(vector<vector<char>>& grid, int r, int c) {
    int m = grid.size(), n = grid[0].size();
    if (r < 0 || r >= m || c < 0 || c >= n) return;
    if (grid[r][c] != '1') return;           // not land or already visited
    grid[r][c] = '0';                         // mark visited in-place
    for (int d = 0; d < 4; d++)
        dfs(grid, r + dr[d], c + dc[d]);
}
// Drive: for each cell, if '1' → dfs, count++
```

### Grid BFS (shortest path in grid)
```cpp
queue<pair<int,int>> q;
vector<vector<int>> dist(m, vector<int>(n, -1));
q.push({sr, sc}); dist[sr][sc] = 0;

while (!q.empty()) {
    auto [r, c] = q.front(); q.pop();
    for (int d = 0; d < 4; d++) {
        int nr = r + dr[d], nc = c + dc[d];
        if (nr<0||nr>=m||nc<0||nc>=n) continue;
        if (dist[nr][nc] != -1) continue;     // visited
        if (grid[nr][nc] == '1') continue;    // obstacle
        dist[nr][nc] = dist[r][c] + 1;
        q.push({nr, nc});
    }
}
```

### Multi-Source BFS (start from ALL sources simultaneously)
```cpp
queue<pair<int,int>> q;
// Push ALL sources first
for (int i = 0; i < m; i++)
    for (int j = 0; j < n; j++)
        if (isSource(grid[i][j])) { dist[i][j] = 0; q.push({i,j}); }
// Then normal BFS from all of them at once
```

### Boundary DFS (save border-connected cells)
```cpp
// Pattern: start DFS from ALL border cells, mark as "safe"
// Then sweep grid: anything not marked safe → flip/capture
for (int i = 0; i < m; i++) { dfs(board, i, 0); dfs(board, i, n-1); }
for (int j = 0; j < n; j++) { dfs(board, 0, j); dfs(board, m-1, j); }
// Now sweep: 'S' → restore, 'O' → capture
```

### Distinct Islands (encode DFS traversal path as shape)
```cpp
void dfs(grid, r, c, path, direction) {
    // mark visited
    for each direction d:
        if valid and unvisited:
            path += d;           // encode direction taken
            dfs(grid, nr, nc, path, d);
            path += 'b';         // backtrack marker (critical for uniqueness)
}
// Insert path string into a set — each unique string = unique shape
```

---

## Problems

### Easy

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | DFS from each unvisited '1', mark visited | Count DFS starts = count islands |
| [733](https://leetcode.com/problems/flood-fill/) | Flood Fill | DFS from source pixel, change color | Check for same color before recursing to avoid infinite loop when new color == old color |
| [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | DFS returns area of connected component | Track max across all starts |
| [1971](https://leetcode.com/problems/find-if-path-exists-in-graph/) | Find if Path Exists | Simple BFS/DFS | Foundation |

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [994](https://leetcode.com/problems/rotting-oranges/) | Rotting Oranges | Multi-source BFS: push ALL rotten oranges at start | After BFS, check if any fresh remain |
| [542](https://leetcode.com/problems/01-matrix/) | 01 Matrix | Multi-source BFS: push ALL 0-cells at start | BFS from 0s outward — same as Rotting Oranges |
| [417](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Pacific Atlantic Water Flow | Reverse: DFS from Pacific border cells, then Atlantic border cells, find intersection | Going "uphill" instead of downhill simplifies the logic |
| [130](https://leetcode.com/problems/surrounded-regions/) | Surrounded Regions | Boundary DFS: mark border-connected 'O' as safe, capture rest | Never capture cells reachable from border |
| [1020](https://leetcode.com/problems/number-of-enclaves/) | Number of Enclaves | Boundary DFS: remove all land reachable from border | Count remaining land cells |
| [286](https://leetcode.com/problems/walls-and-gates/) | Walls and Gates | Multi-source BFS from all gates (0 cells) simultaneously | Same template as rotting oranges |
| [694](https://leetcode.com/problems/number-of-distinct-islands/) | Number of Distinct Islands | DFS + encode path with direction + backtrack markers | String in set = shape fingerprint |
| [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Shortest Path Binary Matrix | BFS from (0,0) to (n-1,n-1) through 0-cells | 8-directional movement. Check start/end cells |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [827](https://leetcode.com/problems/making-a-large-island/) | Making a Large Island | Label each island with unique id + size. For each 0-cell: check distinct neighbors, sum their sizes + 1 | DSU or DFS labeling first, then greedy check |
| [329](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) | Longest Increasing Path | DFS + memoization (each cell has one answer) | Grid forms a DAG (only go to strictly larger cells) |
| [317](https://leetcode.com/problems/shortest-distance-from-all-buildings/) | Shortest Distance from All Buildings | BFS from each building, accumulate distances at each empty cell | Track reach count — valid cell must be reached from ALL buildings |

---

## Key Techniques

### When to use BFS vs DFS on grid
```
Just need to visit all connected cells?    → DFS (simpler)
Need shortest distance to something?       → BFS
Multiple sources, expand simultaneously?   → Multi-source BFS
```

### The Boundary DFS trick
```
Problem says: "cells NOT connected to border are captured/enclosed"
Solution: flip it — mark all cells REACHABLE from border as safe
Pattern: 
  1. DFS/BFS from all 4 border edges
  2. Mark reachable cells with a temp marker
  3. Sweep grid: marked=keep, unmarked=capture
```

### Multi-Source BFS pattern
```
ANY time the question is: "minimum distance from X to nearest Y"
WHERE there are MULTIPLE Y sources
→ Push ALL Y sources into queue FIRST, then BFS
Examples: Rotting oranges, 01 Matrix, Walls and Gates
```

---

## Common Mistakes

- **Flood fill (#733):** When new color == old color, it causes infinite recursion. Check `if oldColor == newColor return` first.
- **Shortest path binary matrix (#1091):** Start cell or end cell might be blocked (value 1). Check this edge case immediately.
- **Distinct islands (#694):** Forgetting the backtrack marker in the path encoding. Two different shapes can produce the same string without it.
- **Pacific Atlantic (#417):** Don't DFS from each cell downward — too complex. Always reverse: DFS from ocean borders upward.
- **Making large island (#827):** When checking 0-cell neighbors, must deduplicate island IDs (same island reachable from multiple directions).

## Follow-up Interviewers Love

- "What if we have 8-directional movement?" → Add 4 diagonal directions to dr/dc arrays
- "What if the grid is too large to fit in memory?" → Discuss chunking / external BFS
- "How would you count distinct islands?" → #694, encode DFS path as shape fingerprint
- "Can you solve 01 Matrix / Rotting Oranges without modifying the input?" → Use separate dist array instead of in-place marking
