# Pattern 8: Trie (Prefix Tree)

## Identify This Pattern
- "Implement dictionary / autocomplete"
- "Find words with a given prefix"
- "Word search in a 2D board with a word list"
- "Count words with prefix X"
- "Wildcard / regex matching on words"
- Problems involving PREFIX operations on strings

## Why Trie (not hashset)?
```
Hashset: O(L) search, but NO prefix queries
Trie:    O(L) search, O(L) prefix check, O(L) insert
         Shared prefixes stored only once → space efficient for word lists
```

---

## Templates

### Trie Node + Class
```cpp
struct TrieNode {
    TrieNode* children[26] = {};   // or unordered_map<char, TrieNode*> for non-alpha
    bool isEnd = false;
    int count = 0;                 // optional: word count through this node
};

class Trie {
    TrieNode* root = new TrieNode();

    // Insert word
    void insert(string word) {
        TrieNode* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) cur->children[idx] = new TrieNode();
            cur = cur->children[idx];
        }
        cur->isEnd = true;
    }

    // Search exact word
    bool search(string word) {
        TrieNode* cur = root;
        for (char c : word) {
            int idx = c - 'a';
            if (!cur->children[idx]) return false;
            cur = cur->children[idx];
        }
        return cur->isEnd;
    }

    // Check if any word starts with prefix
    bool startsWith(string prefix) {
        TrieNode* cur = root;
        for (char c : prefix) {
            int idx = c - 'a';
            if (!cur->children[idx]) return false;
            cur = cur->children[idx];
        }
        return true;   // don't check isEnd — prefix only
    }
};
```

### Trie with Wildcard (`.` matches any char)
```cpp
bool searchWithWild(string word, int idx, TrieNode* node) {
    if (!node) return false;
    if (idx == word.size()) return node->isEnd;
    char c = word[idx];
    if (c != '.') {
        return searchWithWild(word, idx + 1, node->children[c - 'a']);
    } else {
        // try all possible characters
        for (auto child : node->children) {
            if (child && searchWithWild(word, idx + 1, child)) return true;
        }
        return false;
    }
}
```

### Trie + DFS on 2D Board (Word Search II pattern)
```cpp
// Build trie from word list
// DFS on board, navigate trie simultaneously
// When trie node isEnd == true → found a word
// Mark cell visited during DFS, unmark on backtrack

void dfs(board, i, j, TrieNode* node, string& path, vector<string>& result) {
    if (node->isEnd) { result.push_back(path); node->isEnd = false; }
    if (out_of_bounds || visited) return;
    char c = board[i][j];
    TrieNode* next = node->children[c - 'a'];
    if (!next) return;           // prune: no word continues with this char

    board[i][j] = '#';           // mark visited
    path.push_back(c);
    // recurse 4 directions
    path.pop_back();
    board[i][j] = c;             // unmark
}
```

---

## Problems

### Medium

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [208](https://leetcode.com/problems/implement-trie-prefix-tree/) | Implement Trie | Build TrieNode with children[26] + isEnd | Foundation — must know cold |
| [648](https://leetcode.com/problems/replace-words/) | Replace Words | Build trie from roots. For each word in sentence, find shortest prefix in trie | Stop at first `isEnd` node during traversal |
| [720](https://leetcode.com/problems/longest-word-in-dictionary/) | Longest Word in Dictionary | Build trie. BFS/DFS — only extend if current prefix IS a word | Every prefix of the answer must itself be a valid word |
| [1268](https://leetcode.com/problems/search-suggestions-system/) | Search Suggestions System | Build trie. At each prefix, collect up to 3 words via DFS | Lexicographic order: use `children[26]` array (not hashmap) |
| [677](https://leetcode.com/problems/map-sum-pairs/) | Map Sum Pairs | Trie node stores sum. Insert adds value to all nodes on path | For prefix sum: accumulate at each node, not just leaf |
| [211](https://leetcode.com/problems/design-add-and-search-words-data-structure/) | Design Add/Search Words | Trie + wildcard `.` → recurse all children when `.` encountered | DFS through trie when wildcard hit |

### Hard

| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [212](https://leetcode.com/problems/word-search-ii/) | Word Search II | Build trie from words. DFS on board + navigate trie simultaneously | Prune: if no trie node for current char, stop. Remove found words from trie to avoid duplicates |
| [336](https://leetcode.com/problems/palindrome-pairs/) | Palindrome Pairs | Insert all words reversed into trie. For each word, check palindrome pairs | Complex — also solvable with hashmap approach |
| [472](https://leetcode.com/problems/concatenated-words/) | Concatenated Words | Insert all words into trie. For each word: word break using trie | DP or DFS + trie to check if word is made of other words |

---

## Word Search II — Important Optimization

```
Naive: for each word, run DFS on board → O(W * N * M * 4^L)
Trie:  one DFS traversal, navigate all words simultaneously → O(N * M * 4^L)

Critical optimization: after finding a word, set node->isEnd = false
(or delete from trie) to avoid outputting duplicates.

Further optimization: prune dead branches — if a TrieNode has no
remaining words (all children null, isEnd false), unlink it from parent.
This prevents re-visiting dead paths.
```

---

## Space vs Time Tradeoffs

| Structure | Space | Search | Prefix | Best For |
|-----------|-------|--------|--------|---------|
| Hashset | O(total chars) | O(L) | O(L * W) scan | Exact word lookup |
| Sorted array | O(total chars) | O(log W) | O(log W) binary search | Static dictionary |
| Trie (array) | O(26 * L * W) worst | O(L) | O(L) | Prefix queries, shared prefixes |
| Trie (hashmap) | O(unique chars * W) | O(L) | O(L) | Non-alphabetic chars |

---

## Common Mistakes

- `search` vs `startsWith`: search needs `isEnd == true` at end. startsWith just needs to reach the end of prefix.
- Forgetting to mark visited cells in Word Search II DFS → infinite loops
- Not removing found words from trie in #212 → duplicate results
- Using `unordered_map` children when lexicographic order matters (use array `children[26]` instead)

## Follow-up Interviewers Love

- "How would you delete a word from the trie?" → Mark isEnd = false; optionally prune childless nodes upward
- "What's the space complexity?" → O(total characters * alphabet size) worst case with array children
- "Can you make autocomplete return top-k suggestions?" → Add a min-heap or count at each node
- "What if words have non-lowercase characters?" → Use `unordered_map<char, TrieNode*>` instead of `children[26]`
