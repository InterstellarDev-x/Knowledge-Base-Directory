# Linked List Patterns — LeetCode C++ Reference

## How to Decide Which Pattern

```
Cycle detection / middle / nth-from-end?    → Fast & Slow Pointers
Edge cases with head changing?              → Dummy Head Node
Reverse a portion or all of the list?       → In-Place Reversal
Two sorted lists need combining?            → Merge Pattern
Problem has nested / recursive structure?   → Recursive Pattern
Need to reorder, split, or rearrange?       → Two-Pointer + Reconnect
```

**The one question to always ask first:**
> "Do I need to look at two positions at once, or track where to reconnect?"
> — if yes, it's almost certainly a two-pointer or dummy-head problem.

---

## C++ Linked List Quick Reference

```cpp
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

// Common setup lines
ListNode* dummy = new ListNode(0);
dummy->next = head;

ListNode* slow = head, *fast = head;
ListNode* prev = nullptr, *cur = head;
```

> **Key rules:**
> - Always check `if (!head || !head->next)` before using `->next`
> - Draw the list on paper, trace pointer movements step by step
> - When modifying next pointers, save the next node BEFORE overwriting

---

## Pattern 1: Fast & Slow Pointers (Floyd's Algorithm)

**When to think about it:**
- "Does the list have a cycle?"
- "Find the middle of the list"
- "Find the start of the cycle"
- "Find nth node from the end"
- Two pointers moving at different speeds through the same list

**Core insight:** Fast pointer moves 2x speed of slow. When fast hits end → slow is at middle.
When they meet → there's a cycle. This O(1) space trick replaces O(n) HashSet.

```
List: 1 -> 2 -> 3 -> 4 -> 5

Step 0: slow=1, fast=1
Step 1: slow=2, fast=3
Step 2: slow=3, fast=5
Step 3: slow=4, fast=null  <- fast hit end, slow is at middle (node 3 for odd length)
```

### Template A: Detect Cycle
```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head, *fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;   // they met -> cycle exists
    }
    return false;
}
```

### Template B: Find Cycle Start
```cpp
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head, *fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            // Phase 2: move one pointer to head, keep other at meeting point
            // They meet again at the cycle start
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow;
        }
    }
    return nullptr;
}
```

### Template C: Find Middle
```cpp
ListNode* findMiddle(ListNode* head) {
    ListNode* slow = head, *fast = head;

    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;   // for even length, returns second middle
}
// Trick: "fast->next->next" version gives second middle
// Use "fast->next" condition for first middle of even-length list
```

### Template D: Nth Node from End
```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;

    ListNode* fast = dummy, *slow = dummy;

    // Advance fast by n+1 steps
    for (int i = 0; i <= n; i++) fast = fast->next;

    // Move both until fast hits end
    while (fast) {
        slow = slow->next;
        fast = fast->next;
    }

    // slow is now just before the node to delete
    slow->next = slow->next->next;
    return dummy->next;
}
```

**Problems:** #141 Linked List Cycle, #142 Cycle II (find start), #876 Middle of Linked List, #19 Remove Nth from End, #234 Palindrome Linked List

---

## Pattern 2: Dummy Head Node

**When to think about it:**
- Head node might be deleted or changed
- You're building a new list from scratch
- Edge cases hit when result list is empty at start
- "Remove elements", "partition list", "merge lists"

**Core insight:** A dummy node before head means you never have to special-case
"what if we need to remove the head?" — the dummy's next IS the real head.

```
Without dummy:              With dummy:
if removing head:           dummy -> 1 -> 2 -> 3
  head = head->next         prev = dummy
  special case!             prev->next = prev->next->next  <- same code always
```

```cpp
// Generic dummy head template
ListNode* solve(ListNode* head) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* cur = dummy;

    while (cur->next) {
        if (shouldRemove(cur->next)) {
            cur->next = cur->next->next;   // skip the node
        } else {
            cur = cur->next;               // advance only when NOT removing
        }
    }

    return dummy->next;
}
```

### Remove All Occurrences of Value
```cpp
ListNode* removeElements(ListNode* head, int val) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* cur = dummy;

    while (cur->next) {
        if (cur->next->val == val)
            cur->next = cur->next->next;   // skip
        else
            cur = cur->next;
    }
    return dummy->next;
}
```

### Partition List (values < x before values >= x)
```cpp
ListNode* partition(ListNode* head, int x) {
    ListNode* lessHead = new ListNode(0);   // dummy for < x chain
    ListNode* moreHead = new ListNode(0);   // dummy for >= x chain
    ListNode* less = lessHead, *more = moreHead;

    while (head) {
        if (head->val < x) { less->next = head; less = less->next; }
        else               { more->next = head; more = more->next; }
        head = head->next;
    }

    more->next = nullptr;          // terminate >= chain
    less->next = moreHead->next;   // connect chains
    return lessHead->next;
}
```

**Problems:** #203 Remove Linked List Elements, #83 Remove Duplicates, #82 Remove Duplicates II, #86 Partition List, #21 Merge Two Sorted Lists

---

## Pattern 3: In-Place Reversal

**When to think about it:**
- "Reverse the entire list"
- "Reverse a sublist from position m to n"
- "Reverse in groups of k"
- No extra space — must reverse by relinking next pointers

**Core insight:** You need three pointers: `prev`, `cur`, `next`.
Save `next` before breaking the link, point `cur->next` backward, advance.

```
Initial:   nullptr <- [prev]   [cur] -> [next] -> ...
After:     nullptr <- [prev] <- [cur]   [next] -> ...
                               new prev  new cur
```

### Template A: Reverse Entire List
```cpp
ListNode* reverseList(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur = head;

    while (cur) {
        ListNode* next = cur->next;  // save next BEFORE breaking link
        cur->next = prev;            // reverse the link
        prev = cur;                  // advance prev
        cur = next;                  // advance cur
    }
    return prev;   // prev is new head
}

// Recursive version
ListNode* reverseList(ListNode* head) {
    if (!head || !head->next) return head;
    ListNode* newHead = reverseList(head->next);
    head->next->next = head;    // node after head points back to head
    head->next = nullptr;       // head's next becomes null
    return newHead;
}
```

### Template B: Reverse Between Positions m and n
```cpp
ListNode* reverseBetween(ListNode* head, int left, int right) {
    ListNode* dummy = new ListNode(0);
    dummy->next = head;
    ListNode* pre = dummy;

    // Step 1: advance pre to node just before position left
    for (int i = 1; i < left; i++) pre = pre->next;

    // Step 2: reverse from left to right using "head insertion" trick
    ListNode* cur = pre->next;
    for (int i = 0; i < right - left; i++) {
        ListNode* next = cur->next;
        cur->next = next->next;        // detach next
        next->next = pre->next;        // insert next at front of reversed section
        pre->next = next;
    }
    return dummy->next;
}
```

### Template C: Reverse in K-Groups
```cpp
// Helper: reverse n nodes starting from head, return new head
ListNode* reverseN(ListNode* head, int k) {
    ListNode* prev = nullptr, *cur = head;
    for (int i = 0; i < k; i++) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur;
        cur = next;
    }
    return prev;
}

ListNode* reverseKGroup(ListNode* head, int k) {
    // Check if k nodes exist
    ListNode* check = head;
    for (int i = 0; i < k; i++) {
        if (!check) return head;   // fewer than k nodes left, don't reverse
        check = check->next;
    }

    ListNode* newHead = reverseN(head, k);   // reverse k nodes
    head->next = reverseKGroup(check, k);    // head is now tail, connect to rest
    return newHead;
}
```

**Problems:** #206 Reverse Linked List, #92 Reverse Linked List II, #25 Reverse Nodes in k-Group, #234 Palindrome Linked List (reverse second half)

---

## Pattern 4: Merge Pattern

**When to think about it:**
- "Merge two sorted lists"
- "Merge k sorted lists"
- "Sort a linked list" (merge sort on linked list)
- Combine multiple sorted sequences efficiently

**Core insight:** Compare heads of both lists, take the smaller, advance that pointer.
Use a dummy head to avoid edge cases on the first node.

### Template A: Merge Two Sorted Lists
```cpp
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    ListNode* dummy = new ListNode(0);
    ListNode* cur = dummy;

    while (l1 && l2) {
        if (l1->val <= l2->val) { cur->next = l1; l1 = l1->next; }
        else                    { cur->next = l2; l2 = l2->next; }
        cur = cur->next;
    }

    cur->next = l1 ? l1 : l2;   // attach remaining nodes
    return dummy->next;
}

// Recursive version (elegant but O(n) stack space)
ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) {
    if (!l1) return l2;
    if (!l2) return l1;
    if (l1->val <= l2->val) { l1->next = mergeTwoLists(l1->next, l2); return l1; }
    else                    { l2->next = mergeTwoLists(l1, l2->next); return l2; }
}
```

### Template B: Merge K Sorted Lists (Min-Heap)
```cpp
ListNode* mergeKLists(vector<ListNode*>& lists) {
    // Min-heap: {value, node}
    auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
    priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

    // Push head of each list
    for (auto* node : lists)
        if (node) pq.push(node);

    ListNode* dummy = new ListNode(0);
    ListNode* cur = dummy;

    while (!pq.empty()) {
        auto* node = pq.top(); pq.pop();
        cur->next = node;
        cur = cur->next;
        if (node->next) pq.push(node->next);  // push next from same list
    }
    return dummy->next;
}
```

### Template C: Sort Linked List (Merge Sort)
```cpp
ListNode* sortList(ListNode* head) {
    if (!head || !head->next) return head;

    // Find middle and split
    ListNode* slow = head, *fast = head->next;  // fast->next: get LEFT middle
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    ListNode* mid = slow->next;
    slow->next = nullptr;          // cut the list in half

    ListNode* left  = sortList(head);
    ListNode* right = sortList(mid);
    return mergeTwoLists(left, right);
}
```

**Problems:** #21 Merge Two Sorted Lists, #23 Merge k Sorted Lists, #148 Sort List

---

## Pattern 5: Two-Pointer Reconnect (Reorder / Rearrange)

**When to think about it:**
- "Reorder list" (interleave first and second half)
- "Rotate list"
- "Odd-even grouping"
- You need to find a split point, modify one half, then reconnect

**Core insight:** Find the structural midpoint or split, operate on each half
separately, then reconnect. Always use fast/slow to find the split.

### Reorder List (1->2->3->4->5 becomes 1->5->2->4->3)
```cpp
void reorderList(ListNode* head) {
    if (!head || !head->next) return;

    // Step 1: find middle
    ListNode* slow = head, *fast = head;
    while (fast->next && fast->next->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: reverse second half
    ListNode* second = slow->next;
    slow->next = nullptr;           // cut
    second = reverseList(second);

    // Step 3: interleave
    ListNode* first = head;
    while (second) {
        ListNode* tmp1 = first->next;
        ListNode* tmp2 = second->next;
        first->next  = second;
        second->next = tmp1;
        first  = tmp1;
        second = tmp2;
    }
}
```

### Odd-Even Linked List (odd-indexed nodes then even-indexed nodes)
```cpp
ListNode* oddEvenList(ListNode* head) {
    if (!head) return head;

    ListNode* odd  = head;
    ListNode* even = head->next;
    ListNode* evenHead = even;      // save even head for reconnect

    while (even && even->next) {
        odd->next  = even->next;    // odd points to next odd
        odd        = odd->next;
        even->next = odd->next;     // even points to next even
        even       = even->next;
    }

    odd->next = evenHead;           // connect odd tail to even head
    return head;
}
```

### Rotate List by k
```cpp
ListNode* rotateRight(ListNode* head, int k) {
    if (!head || !head->next || k == 0) return head;

    // Find length and tail
    int len = 1;
    ListNode* tail = head;
    while (tail->next) { tail = tail->next; len++; }

    k = k % len;
    if (k == 0) return head;

    // Find new tail: (len - k - 1) steps from head
    tail->next = head;              // make circular
    ListNode* newTail = head;
    for (int i = 0; i < len - k - 1; i++) newTail = newTail->next;

    ListNode* newHead = newTail->next;
    newTail->next = nullptr;
    return newHead;
}
```

**Problems:** #143 Reorder List, #328 Odd Even Linked List, #61 Rotate List

---

## Pattern 6: Recursive Linked List

**When to think about it:**
- Problem has a natural recursive structure (process node, then recurse on rest)
- "Add two numbers represented as lists" (carry propagates forward)
- Elegant reverse, clone, or deep-copy problems
- Warning: O(n) stack space — avoid on very long lists

**Core insight:** Treat the list as: `head + recurse(head->next)`.
Process the recursive result FIRST, then handle head on the way back up.

### Add Two Numbers (digits stored in reverse order)
```cpp
ListNode* addTwoNumbers(ListNode* l1, ListNode* l2, int carry = 0) {
    if (!l1 && !l2 && carry == 0) return nullptr;

    int sum  = carry;
    if (l1) { sum += l1->val; l1 = l1->next; }
    if (l2) { sum += l2->val; l2 = l2->next; }

    ListNode* node = new ListNode(sum % 10);
    node->next = addTwoNumbers(l1, l2, sum / 10);
    return node;
}
```

### Copy List with Random Pointer
```cpp
unordered_map<Node*, Node*> mp;

Node* copyRandomList(Node* head) {
    if (!head) return nullptr;
    if (mp.count(head)) return mp[head];    // already cloned

    Node* clone = new Node(head->val);
    mp[head] = clone;                       // map BEFORE recursing (handles cycles)

    clone->next   = copyRandomList(head->next);
    clone->random = copyRandomList(head->random);
    return clone;
}
```

**Problems:** #2 Add Two Numbers, #445 Add Two Numbers II, #138 Copy List with Random Pointer, #206 Reverse (recursive)

---

## Pattern 7: Runner Technique (Two Pointers, Same Direction)

**When to think about it:**
- "Find intersection of two lists"
- Need to align two lists of different lengths
- One pointer "runs ahead" to create an offset

**Core insight:** If list A has length a+c and list B has length b+c (c = common tail),
then a pointer traversing A then B, and another traversing B then A, will meet at
the intersection after a+b+c steps each.

### Intersection of Two Linked Lists
```cpp
ListNode* getIntersectionNode(ListNode* headA, ListNode* headB) {
    ListNode* a = headA, *b = headB;

    // When a reaches end, redirect to headB. Vice versa.
    // They meet at intersection after traveling a+b+c steps each.
    while (a != b) {
        a = a ? a->next : headB;
        b = b ? b->next : headA;
    }
    return a;   // nullptr if no intersection
}
```

### Check Palindrome Linked List
```cpp
bool isPalindrome(ListNode* head) {
    // Step 1: find middle
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }

    // Step 2: reverse second half
    ListNode* rev = nullptr, *cur = slow;
    while (cur) {
        ListNode* next = cur->next;
        cur->next = rev;
        rev = cur;
        cur = next;
    }

    // Step 3: compare first and reversed second half
    ListNode* left = head, *right = rev;
    while (right) {
        if (left->val != right->val) return false;
        left  = left->next;
        right = right->next;
    }
    return true;
}
```

**Problems:** #160 Intersection of Two Linked Lists, #234 Palindrome Linked List

---

## The 5 Most Important Templates to Memorize

These five cover ~80% of all linked list interview questions:

```cpp
// 1. REVERSE (3-pointer iterative)
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr, *cur = head;
    while (cur) {
        ListNode* next = cur->next;
        cur->next = prev;
        prev = cur; cur = next;
    }
    return prev;
}

// 2. FIND MIDDLE (fast/slow)
ListNode* middle(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) { slow = slow->next; fast = fast->next->next; }
    return slow;
}

// 3. MERGE TWO SORTED (dummy head)
ListNode* merge(ListNode* l1, ListNode* l2) {
    ListNode* dummy = new ListNode(0), *cur = dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) { cur->next = l1; l1 = l1->next; }
        else                    { cur->next = l2; l2 = l2->next; }
        cur = cur->next;
    }
    cur->next = l1 ? l1 : l2;
    return dummy->next;
}

// 4. DETECT CYCLE (fast/slow meet)
bool hasCycle(ListNode* head) {
    ListNode* slow = head, *fast = head;
    while (fast && fast->next) {
        slow = slow->next; fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}

// 5. NTH FROM END (two-pointer gap)
ListNode* nthFromEnd(ListNode* head, int n) {
    ListNode* dummy = new ListNode(0); dummy->next = head;
    ListNode* fast = dummy, *slow = dummy;
    for (int i = 0; i <= n; i++) fast = fast->next;
    while (fast) { slow = slow->next; fast = fast->next; }
    return slow->next;
}
```

---

## Universal Checklist Before Coding

- [ ] Can head itself be removed or changed? → use **dummy head**
- [ ] Need cycle detection or middle? → **fast/slow pointers**
- [ ] Reversing any portion? → **prev/cur/next** three-pointer pattern
- [ ] Save `next` BEFORE modifying `cur->next`
- [ ] Check `if (!head || !head->next)` — base case for single/empty list
- [ ] After split: remember to set the tail's `->next = nullptr` (cut the list)
- [ ] Off-by-one: trace through a 3-4 node example by hand before submitting

---

## Must-Do Problem List

| Priority | # | Problem | Difficulty | Pattern |
|---|---|---------|-----------|---------|
| 1  | 206 | Reverse Linked List              | Easy   | In-place reversal        |
| 2  | 21  | Merge Two Sorted Lists           | Easy   | Merge + dummy head       |
| 3  | 141 | Linked List Cycle                | Easy   | Fast/slow pointers       |
| 4  | 876 | Middle of Linked List            | Easy   | Fast/slow pointers       |
| 5  | 19  | Remove Nth Node from End         | Medium | Two-pointer gap          |
| 6  | 142 | Linked List Cycle II             | Medium | Floyd's cycle start      |
| 7  | 160 | Intersection of Two Lists        | Easy   | Runner technique         |
| 8  | 234 | Palindrome Linked List           | Easy   | Middle + reverse         |
| 9  | 92  | Reverse Linked List II           | Medium | Partial reversal         |
| 10 | 143 | Reorder List                     | Medium | Middle + reverse + merge |
| 11 | 328 | Odd Even Linked List             | Medium | Two-pointer reconnect    |
| 12 | 2   | Add Two Numbers                  | Medium | Recursive / iterative    |
| 13 | 86  | Partition List                   | Medium | Dummy head (two chains)  |
| 14 | 148 | Sort List                        | Medium | Merge sort               |
| 15 | 25  | Reverse Nodes in k-Group         | Hard   | Reversal in groups       |
| 16 | 23  | Merge k Sorted Lists             | Hard   | Merge + min-heap         |
| 17 | 138 | Copy List with Random Pointer    | Medium | HashMap / recursive      |
| 18 | 61  | Rotate List                      | Medium | Find tail, make circular |
| 19 | 82  | Remove Duplicates II             | Medium | Dummy head + skip        |
| 20 | 445 | Add Two Numbers II               | Medium | Stack or reverse         |

**Study order:** #206 -> #21 -> #141 -> #876 -> #19 -> #142 -> #234 -> #92 -> #143 -> #25
These ten cover every core pattern.

---

## How Patterns Combine (Hard Problems = Multiple Patterns)

| Problem | Patterns Used |
|---------|--------------|
| #234 Palindrome List | Fast/Slow (middle) + Reversal |
| #143 Reorder List | Fast/Slow (middle) + Reversal + Merge |
| #148 Sort List | Fast/Slow (split) + Merge |
| #25 Reverse k-Group | Reversal + Counting + Reconnect |
| #142 Cycle II | Fast/Slow (detect) + Two-pointer (find start) |

---

## Complexity Reference

| Pattern               | Time     | Space | Key Insight                              |
|-----------------------|----------|-------|------------------------------------------|
| Fast/Slow Pointers    | O(n)     | O(1)  | Two speeds guarantee meeting in cycle    |
| Dummy Head            | O(n)     | O(1)  | Eliminates head-removal edge case        |
| In-Place Reversal     | O(n)     | O(1)  | Three pointers, no extra nodes           |
| Merge Two Sorted      | O(n+m)   | O(1)  | One pass, dummy simplifies start         |
| Merge K Sorted        | O(n log k) | O(k) | Min-heap size k, n total nodes          |
| Merge Sort (Sort List)| O(n log n) | O(log n) | Stack space for recursion depth      |
| Runner Technique      | O(n)     | O(1)  | Two pointers traverse a+b+c each         |
| Recursive             | O(n)     | O(n)  | Stack depth = list length                |

---

## Company-Wise Most Asked Problems (Recent Trends)

| Company | Most Frequently Asked Problems |
|---------|-------------------------------|
| **Amazon** | #2 Add Two Numbers, #206 Reverse, #21 Merge Two Sorted, #141 Cycle, #19 Remove Nth |
| **Google** | #25 Reverse k-Group, #23 Merge k Sorted, #142 Cycle II, #160 Intersection |
| **Meta** | #206 Reverse, #21 Merge Two Sorted, #143 Reorder List, #138 Copy with Random |
| **Microsoft** | #206 Reverse, #19 Remove Nth, #21 Merge Two Sorted, #234 Palindrome |
| **Bloomberg** | #2 Add Two Numbers, #19 Remove Nth, #86 Partition, #82 Remove Duplicates II |
| **Apple** | #141 Cycle, #876 Middle, #234 Palindrome |
| **Uber / Airbnb** | #25 Reverse k-Group, #23 Merge k Sorted |

---

## Recent Interview Trends (2023–2025)

### 1. Two-pointer is the #1 tested skill
Almost every medium/hard linked list question at FAANG boils down to fast/slow or
gap-pointer. Pattern 1 (Fast & Slow) alone covers ~40% of questions asked.

### 2. "Combine patterns" questions are rising
Companies are moving away from single-pattern questions. Expect multi-step problems:

| Problem | Patterns Combined |
|---------|------------------|
| #143 Reorder List | Middle + Reversal + Merge (asked heavily at Meta/Amazon) |
| #234 Palindrome List | Middle + Reversal + Compare |
| #142 Cycle II | Cycle detect + find start (Google favorite) |
| #148 Sort List | Middle + Merge sort |
| #25 Reverse k-Group | Reversal + counting + reconnect |

### 3. Hard problems appear at senior-level interviews
- **#25 Reverse Nodes in k-Group** — Google/Uber for L5+
- **#23 Merge k Sorted Lists** — Google/Amazon for senior roles
- **#138 Copy List with Random Pointer** — Meta specifically, tests HashMap thinking

### 4. Bloomberg asks the most linked list questions
Bloomberg disproportionately asks linked list questions vs other companies —
especially pointer manipulation (#86 Partition, #82 Remove Duplicates II, #19).

### 5. "Explain your pointer logic" is now expected
Interviewers ask you to trace pointer movements step by step. Pure code isn't enough.
Be ready to explain:
- Why you save `next` before overwriting `cur->next`
- Why you use a dummy node vs not
- What happens to your pointers on a 1-node or 2-node list (edge cases)

### 6. LRU Cache (Design) — linked list + hashmap combo
**#146 LRU Cache** (Hard) is consistently one of the most asked design problems
at Google, Meta, Amazon. It uses a doubly linked list + unordered_map together.
This is the bridge between pure linked list and system design rounds.

```cpp
// LRU Cache core structure
struct Node {
    int key, val;
    Node* prev;
    Node* next;
    Node(int k, int v) : key(k), val(v), prev(nullptr), next(nullptr) {}
};

class LRUCache {
    int capacity;
    unordered_map<int, Node*> mp;   // key -> node
    Node* head;                      // dummy head (least recent)
    Node* tail;                      // dummy tail (most recent)

    void remove(Node* node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    void insertAtTail(Node* node) {
        node->prev = tail->prev;
        node->next = tail;
        tail->prev->next = node;
        tail->prev = node;
    }

public:
    LRUCache(int cap) : capacity(cap) {
        head = new Node(0, 0);
        tail = new Node(0, 0);
        head->next = tail;
        tail->prev = head;
    }

    int get(int key) {
        if (!mp.count(key)) return -1;
        remove(mp[key]);
        insertAtTail(mp[key]);
        return mp[key]->val;
    }

    void put(int key, int val) {
        if (mp.count(key)) remove(mp[key]);
        else if ((int)mp.size() == capacity) {
            mp.erase(head->next->key);
            remove(head->next);
        }
        mp[key] = new Node(key, val);
        insertAtTail(mp[key]);
    }
};
```

**#146 LRU Cache is the most important "hard" linked list problem to know.**

---

## Priority Study Plan Based on Company Targets

**For Amazon / Microsoft (SDE-1/SDE-2):**
#206 -> #21 -> #141 -> #19 -> #876 -> #234 -> #2 -> #143

**For Google / Meta (L4/L5):**
All of the above + #142 -> #92 -> #25 -> #23 -> #138 -> #146

**For Bloomberg:**
#206 -> #21 -> #19 -> #2 -> #86 -> #82 -> #83 -> #445
