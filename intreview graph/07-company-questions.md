# Company-Specific Graph Questions (2024–2025)

> Data sourced from LeetCode company tags + interview reports. Sorted by frequency (times reported).

---

## Amazon — Graph Problems

Amazon asks more graph problems than any other company. #200 Islands is their #1 most asked.

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 103 | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS/BFS |
| 41  | [127](https://leetcode.com/problems/word-ladder/) | Word Ladder | BFS implicit graph |
| 36  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS/backtrack |
| 35  | [994](https://leetcode.com/problems/rotting-oranges/) | Rotting Oranges | Multi-source BFS |
| 34  | [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Topological Sort |
| 31  | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 28  | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo Sort / Cycle |
| 19  | [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | Grid DFS |
| 10  | [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | BFS weighted graph |
| 10  | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topological Sort |
| 9   | [126](https://leetcode.com/problems/word-ladder-ii/) | Word Ladder II | BFS + backtrack |
| 9   | [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Shortest Path Binary Matrix | BFS grid |
| 8   | [694](https://leetcode.com/problems/number-of-distinct-islands/) | Number of Distinct Islands | DFS + shape hash |
| 8   | [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS + hashmap |
| 6   | [323](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Connected Components | DSU / DFS |
| 6   | [841](https://leetcode.com/problems/keys-and-rooms/) | Keys and Rooms | DFS |
| 6   | [542](https://leetcode.com/problems/01-matrix/) | 01 Matrix | Multi-source BFS |

**Amazon Tip:** They heavily test BFS + DFS on grids. Rotting Oranges + Number of Islands cover the majority of their patterns. Word Ladder series is a reliable hard-round question.

---

## Meta (Facebook) — Graph Problems

Meta's graph problems skew toward BFS on grids and union-find (accounts merge).

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 76  | [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Shortest Path in Binary Matrix | BFS grid |
| 38  | [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS/DFS + hashmap |
| 34  | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 34  | [721](https://leetcode.com/problems/accounts-merge/) | Accounts Merge | DSU + hashmap |
| 21  | [953](https://leetcode.com/problems/verifying-an-alien-dictionary/) | Verifying Alien Dictionary | Order check (not topo) |
| 18  | [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | Grid DFS |
| 16  | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |
| 15  | [286](https://leetcode.com/problems/walls-and-gates/) | Walls and Gates | Multi-source BFS |
| 13  | [127](https://leetcode.com/problems/word-ladder/) | Word Ladder | BFS implicit |
| 12  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS |
| 9   | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo / Cycle |
| 8   | [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | BFS weighted |
| 6   | [332](https://leetcode.com/problems/reconstruct-itinerary/) | Reconstruct Itinerary | DFS Eulerian path |

**Meta Tip:** #1091 Shortest Path in Binary Matrix is their #1 graph problem — very high frequency. Clone Graph (#133) is consistently asked in phone screens. Accounts Merge (#721) is a signature Meta problem.

---

## Google — Graph Problems

Google focuses on harder combinations: topo sort, weighted paths, implicit graphs.

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 21  | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 11  | [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Topo Sort |
| 10  | [417](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Pacific Atlantic Water Flow | Multi-source DFS |
| 9   | [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | Grid DFS |
| 8   | [743](https://leetcode.com/problems/network-delay-time/) | Network Delay Time | Dijkstra |
| 8   | [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | BFS weighted |
| 7   | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |
| 6   | [542](https://leetcode.com/problems/01-matrix/) | 01 Matrix | Multi-source BFS |
| 5   | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 5   | [130](https://leetcode.com/problems/surrounded-regions/) | Surrounded Regions | Boundary DFS |
| 5   | [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Shortest Path Binary Matrix | BFS grid |
| 5   | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo / Cycle |
| 5   | [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS + hashmap |
| 4   | [332](https://leetcode.com/problems/reconstruct-itinerary/) | Reconstruct Itinerary | Eulerian path |
| 3   | [261](https://leetcode.com/problems/graph-valid-tree/) | Graph Valid Tree | DSU / DFS |
| 3   | [721](https://leetcode.com/problems/accounts-merge/) | Accounts Merge | DSU |
| 2   | [305](https://leetcode.com/problems/number-of-islands-ii/) | Number of Islands II | DSU dynamic |
| 2   | [684](https://leetcode.com/problems/redundant-connection/) | Redundant Connection | DSU |
| 1   | [1020](https://leetcode.com/problems/number-of-enclaves/) | Number of Enclaves | Boundary DFS |

**Google Tip:** Google combines patterns. Expect questions like "grid + Dijkstra" (#1631) or "topo sort + custom comparison". Alien Dictionary (#269) is a Google signature. They also invent novel graph problems in the on-site.

---

## Microsoft — Graph Problems

Microsoft prefers classic, clean problems. Less exotic than Google.

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 42  | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 16  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS |
| 12  | [994](https://leetcode.com/problems/rotting-oranges/) | Rotting Oranges | Multi-source BFS |
| 10  | [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | Grid DFS |
| 10  | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo / Cycle |
| 10  | [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Topo Sort |
| 8   | [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) | Shortest Path Binary Matrix | BFS grid |
| 7   | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 6   | [127](https://leetcode.com/problems/word-ladder/) | Word Ladder | BFS implicit |
| 4   | [694](https://leetcode.com/problems/number-of-distinct-islands/) | Number of Distinct Islands | DFS + shape hash |
| 4   | [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS/DFS |
| 3   | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |
| 2   | [684](https://leetcode.com/problems/redundant-connection/) | Redundant Connection | DSU |
| 2   | [323](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/) | Connected Components | DSU |
| 2   | [261](https://leetcode.com/problems/graph-valid-tree/) | Graph Valid Tree | DSU |

---

## Bloomberg — Graph Problems

Bloomberg keeps it practical — islands and word search dominate.

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 33  | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 14  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS |
| 6   | [133](https://leetcode.com/problems/clone-graph/) | Clone Graph | BFS/DFS |
| 4   | [130](https://leetcode.com/problems/surrounded-regions/) | Surrounded Regions | Boundary DFS |
| 4   | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |
| 2   | [694](https://leetcode.com/problems/number-of-distinct-islands/) | Number of Distinct Islands | DFS |
| 2   | [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | BFS weighted |
| 2   | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo |

---

## Uber — Graph Problems

Uber focuses on search in graphs and Eulerian path problems.

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 36  | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 17  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS |
| 14  | [399](https://leetcode.com/problems/evaluate-division/) | Evaluate Division | BFS weighted |
| 10  | [332](https://leetcode.com/problems/reconstruct-itinerary/) | Reconstruct Itinerary | Eulerian path |
| 7   | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 5   | [305](https://leetcode.com/problems/number-of-islands-ii/) | Number of Islands II | DSU dynamic |
| 3   | [417](https://leetcode.com/problems/pacific-atlantic-water-flow/) | Pacific Atlantic | Multi-source DFS |
| 3   | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |
| 2   | [130](https://leetcode.com/problems/surrounded-regions/) | Surrounded Regions | Boundary DFS |

---

## Apple — Graph Problems

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 11  | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |
| 5   | [127](https://leetcode.com/problems/word-ladder/) | Word Ladder | BFS implicit |
| 5   | [210](https://leetcode.com/problems/course-schedule-ii/) | Course Schedule II | Topo Sort |
| 4   | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 4   | [721](https://leetcode.com/problems/accounts-merge/) | Accounts Merge | DSU |
| 3   | [695](https://leetcode.com/problems/max-area-of-island/) | Max Area of Island | Grid DFS |
| 3   | [207](https://leetcode.com/problems/course-schedule/) | Course Schedule | Topo |

---

## Airbnb — Graph Problems

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 14  | [269](https://leetcode.com/problems/alien-dictionary/) | Alien Dictionary | Topo Sort |

**Airbnb Tip:** Alien Dictionary #269 is their signature graph problem with extremely high frequency. Know it perfectly.

---

## Twitter/X — Graph Problems

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 23  | [79](https://leetcode.com/problems/word-search/) | Word Search | Grid DFS |
| 5   | [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Trie + Grid DFS |
| 4   | [332](https://leetcode.com/problems/reconstruct-itinerary/) | Reconstruct Itinerary | Eulerian path |

---

## DoorDash — Graph Problems

| Freq | # | Problem | Pattern |
|------|---|---------|---------|
| 11  | [286](https://leetcode.com/problems/walls-and-gates/) | Walls and Gates | Multi-source BFS |
| 5   | [317](https://leetcode.com/problems/shortest-distance-from-all-buildings/) | Shortest Distance from All Buildings | Multi-source BFS |
| 5   | [200](https://leetcode.com/problems/number-of-islands/) | Number of Islands | Grid DFS |

---

## Problems That Appear at EVERY Company

These 5 are asked everywhere — do these first:

```
#200  Number of Islands           (Grid DFS)
#207  Course Schedule             (Topo Sort / Cycle)
#133  Clone Graph                 (BFS/DFS + hashmap)
#994  Rotting Oranges             (Multi-source BFS)
#127  Word Ladder                 (BFS implicit graph)
```

---

## Problems Exclusive to Specific Companies

| Problem | Only Asked At |
|---------|--------------|
| [332](https://leetcode.com/problems/reconstruct-itinerary/) Reconstruct Itinerary | Uber, Google, Twitter |
| [269](https://leetcode.com/problems/alien-dictionary/) Alien Dictionary | Airbnb (#1 problem!), Google, Amazon, Meta |
| [743](https://leetcode.com/problems/network-delay-time/) Network Delay Time | Google, Uber |
| [721](https://leetcode.com/problems/accounts-merge/) Accounts Merge | Meta, Apple, Google |
| [1091](https://leetcode.com/problems/shortest-path-in-binary-matrix/) Shortest Path Binary Matrix | Meta (#1 graph problem!), Amazon, Google, Microsoft |
| [417](https://leetcode.com/problems/pacific-atlantic-water-flow/) Pacific Atlantic | Google, Uber |
| [305](https://leetcode.com/problems/number-of-islands-ii/) Islands II (dynamic) | Google, Uber |
| [399](https://leetcode.com/problems/evaluate-division/) Evaluate Division | Google, Meta, Amazon, Uber, Snapchat |
| [286](https://leetcode.com/problems/walls-and-gates/) Walls and Gates | Meta, DoorDash |

---

## 2024–2025 Emerging Trends

### 1. Dijkstra on grids is now mainstream (mid-level+)
- [1631](https://leetcode.com/problems/path-with-minimum-effort/) Path with Min Effort — Dijkstra on 2D grid
- [778](https://leetcode.com/problems/swim-in-rising-water/) Swim in Rising Water — Dijkstra / binary search
- [1514](https://leetcode.com/problems/path-with-maximum-probability/) Path with Max Probability — Dijkstra variant
- These combine two patterns: grid traversal + weighted shortest path

### 2. Bidirectional BFS asked for follow-ups (#127)
- After solving Word Ladder, expect: "Can you optimize it?"
- Answer: bidirectional BFS from both `beginWord` and `endWord` simultaneously
- Reduces time from O(b^d) to O(b^(d/2))

### 3. DSU with size/weight tracking
- [684](https://leetcode.com/problems/redundant-connection/) Redundant Connection — track which edge creates cycle
- [1584](https://leetcode.com/problems/min-cost-to-connect-all-points/) Min Cost Connect Points — Kruskal's MST
- [1319](https://leetcode.com/problems/number-of-operations-to-make-network-connected/) Make Network Connected — DSU components

### 4. "Implicit graph" BFS is consistently tested
- State = some encoding (word, board position, lock combo)
- Neighbors generated by transformation rules (change 1 char, flip 1 bit)
- Problems: #127 Word Ladder, #433 Gene Mutation, #752 Open the Lock, #847 Shortest Path Visiting All Nodes

### 5. Graph + DP combinations (senior/hard rounds)
- [329](https://leetcode.com/problems/longest-increasing-path-in-a-matrix/) Longest Increasing Path — DFS + memo on DAG
- [787](https://leetcode.com/problems/cheapest-flights-within-k-stops/) Cheapest Flights K Stops — Dijkstra or Bellman-Ford with state
- These appear in Google/Meta for senior SWE rounds

### 6. Number of Distinct Islands (#694) rising at Amazon/Microsoft
- DFS + encode the shape of each island as a string/hash
- Tests creative encoding of DFS traversal path
