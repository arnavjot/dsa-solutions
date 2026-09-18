**Whiteboard trap:** The `<` vs `<=` boundary on touching intervals: for Merge Intervals `[1,2],[2,3]` DO merge (`s <= last_end`), but for Meeting Rooms a meeting ending at 10 and one starting at 10 do NOT need two rooms (`heap[0] <= s` frees the room) — pick the comparison per problem, don't copy it blindly. And always sort first; unsorted input is the #1 silent killer here.

---

## 3. Trees, Tries, Heaps / Top-K

> All tree code assumes LeetCode's `TreeNode` (`self.val`, `self.left`, `self.right`). On the whiteboard, write `class TreeNode` once in a corner and move on.

### Tree DFS (recursive)
**Amazon problems (practice in this order):**

| LC# | Problem | Diff | Approach in one line |
|-----|---------|------|----------------------|
| 206 | Reverse Linked List | E | Three pointers prev/cur/next; flip one `.next` per step; return prev |
| 21 | Merge Two Sorted Lists | E | Dummy head + tail pointer; splice the smaller head each step; attach the leftover list |
| 141 | Linked List Cycle | E | Fast (2x) and slow (1x) pointers — they meet iff there's a cycle |
| 19 | Remove Nth Node From End | M | Advance fast n steps from a dummy, then move both until fast hits the end; `slow.next = slow.next.next` |
| 2 | Add Two Numbers | M | Walk both lists with a carry, building result digits `(a + b + carry) % 10` |
| 143 | Reorder List | M | Find middle, reverse second half, then interleave the two halves |
| 138 | Copy List with Random Pointer | M | Pass 1: dict old→new clone; pass 2: wire clone `.next`/`.random` through the dict |
| 23 | Merge k Sorted Lists | H | Min-heap of `(val, i, node)` for each list head; pop smallest, push its `.next` |
| 146 | LRU Cache | M | Hashmap + doubly linked list — covered in depth in **system-design-lld-and-lru.md** |

**Flagship — LC 138 Copy List with Random Pointer (top-5 Amazon question):**
```python
class Node:
    def __init__(self, val=0, next=None, random=None):
        self.val, self.next, self.random = val, next, random

def copyRandomList(head):
    old_to_new = {None: None}         # sentinel so None pointers map cleanly
    cur = head
    while cur:                        # pass 1: clone every node (no wiring)
        old_to_new[cur] = Node(cur.val)
        cur = cur.next
    cur = head
    while cur:                        # pass 2: wire clones via the map
        old_to_new[cur].next = old_to_new[cur.next]
        old_to_new[cur].random = old_to_new[cur.random]
        cur = cur.next
    return old_to_new[head]

# Dry run A->B, A.random=B, B.random=A: pass 1 makes A',B'.
# Pass 2: A'.next=B', A'.random=B'; B'.next=map[None]=None, B'.random=A'. Done.
```

**Whiteboard trap:** Overwriting `.next` before saving it (losing the rest of the list) — always write the reversal as save-next-first (or the tuple-assignment trick); and skipping the dummy head on merges/deletions, which forces ugly special-casing of the first node.

---

### Intervals
**Spot it:** Input is pairs `[start, end]`: "merge overlapping", "insert an interval", "can attend all meetings", "minimum number of rooms", "minimum removals to make non-overlapping". Two master moves: (1) sort by start and merge/greedy, (2) chronological events / sweep line — split into sorted starts and sorted ends (or `(time, +1/-1)` events) and track a running count.

**Template:**
```python
def merge_intervals(intervals):
    intervals.sort()                              # by start
    out = []
    for s, e in intervals:
        if out and s <= out[-1][1]:               # overlaps (or touches) last
            out[-1][1] = max(out[-1][1], e)       # extend
        else:
            out.append([s, e])
    return out

def max_concurrent(intervals):                    # sweep-line count
    events = []
    for s, e in intervals:
        events.append((s, 1))
        events.append((e, -1))
    events.sort()                                 # end(-1) before start(+1) at ties
    best = cur = 0
    for _, d in events:
        cur += d
        best = max(best, cur)
    return best
```

**Amazon problems (practice in this order):**

| LC# | Problem | Diff | Approach in one line |
|-----|---------|------|----------------------|
| 252 | Meeting Rooms | E | Sort by start; conflict iff any `start[i] < end[i-1]` |
| 56 | Merge Intervals | M | Sort by start; extend the last output interval while starts fall inside it |
| 57 | Insert Interval | M | Copy intervals ending before newStart; absorb all overlappers into new via min/max; copy the rest |
| 253 | Meeting Rooms II | M | Min-heap of end times: reuse a room if `heap[0] <= start`, else push a new one; answer = heap size |
| 435 | Non-overlapping Intervals | M | Sort by END; greedily keep intervals that start after the last kept end; removals = n − kept |
| 452 | Min Arrows to Burst Balloons | M | Same greedy as 435 sorted by end — one arrow per group sharing a common point |
| 986 | Interval List Intersections | M | Two pointers; intersection is `[max(starts), min(ends)]` if valid; advance whichever ends first |

**Flagship — LC 253 Meeting Rooms II (a top Amazon question):**
```python
import heapq

def minMeetingRooms(intervals):
    intervals.sort()                  # by start time
    heap = []                         # end times of meetings currently in a room
    for s, e in intervals:
        if heap and heap[0] <= s:     # earliest-ending meeting is over
            heapq.heapreplace(heap, e)    # reuse that room
        else:
            heapq.heappush(heap, e)       # need a new room
    return len(heap)

# Dry run [[0,30],[5,10],[15,20]]: push 30; 5<30 so push 10 -> [10,30];
# 10<=15 so replace 10 with 20 -> [20,30]. Answer: 2 rooms.
```

**Whiteboard trap:** The `<` vs `<=` boundary on touching intervals: for Merge Intervals `[1,2],[2,3]` DO merge (`s <= last_end`), but for Meeting Rooms a meeting ending at 10 and one starting at 10 do NOT need two rooms (`heap[0] <= s` frees the room) — pick the comparison per problem, don't copy it blindly. And always sort first; unsorted input is the #1 silent killer here.

---

## 3. Trees, Tries, Heaps / Top-K

> All tree code assumes LeetCode's `TreeNode` (`self.val`, `self.left`, `self.right`). On the whiteboard, write `class TreeNode` once in a corner and move on.

### Tree DFS (recursive)

**Spot it:** "depth / height / diameter / balanced", "path sum", "same tree / subtree", "lowest common ancestor", "invert / mirror" — anything where the answer for a node is built from the answers of its two children.

**Template:**
```python
def dfs(node):
    if not node:
        return 0                      # base case: null answer (0 / True / None)
    left = dfs(node.left)
    right = dfs(node.right)
    return 1 + max(left, right)       # combine children -> answer for this subtree
```
Two DFS flavors to know cold: (1) **return a value up** (depth, path sum), (2) **update a global while returning something else** (diameter, max path sum return the *downward* chain, update the *through-node* answer on the side).

**Amazon problems (practice in this order):**

| LC# | Problem | Diff | Approach in one line |
|---|---|---|---|
| 104 | Maximum Depth of Binary Tree | E | Return `1 + max(depth(l), depth(r))`; null returns 0 |
| 100 | Same Tree | E | Both null → True; one null or vals differ → False; recurse on both sides |
| 110 | Balanced Binary Tree | E | DFS returns height, or `-1` sentinel once any subtree is unbalanced; bubble `-1` up |
| 543 | Diameter of Binary Tree | E | DFS returns height; at each node update global `best = leftH + rightH` |
| 572 | Subtree of Another Tree | E | At every node of root, run `sameTree(node, subRoot)`; O(n·m) is fine to state |
| 112 | Path Sum | E | Subtract `node.val` from target going down; at a leaf check remainder == 0 |
| 113 | Path Sum II | M | Same subtraction, carry a path list; append a **copy** at matching leaf, pop on backtrack |
| 235 | Lowest Common Ancestor of a BST | M | Walk from root: both keys smaller → go left, both larger → go right, else current node is the LCA |
| 236 | Lowest Common Ancestor of a Binary Tree | M | Postorder: return node if it is p/q or if both left and right recursions found something |
| 124 | Binary Tree Maximum Path Sum | H | DFS returns best downward gain (clamped at 0); update global with `left + node.val + right` |

**Flagship — LC 572 Subtree of Another Tree (huge Amazon favorite):**
```python
def isSubtree(root, subRoot):
    def same(a, b):
        if not a and not b:
            return True
        if not a or not b or a.val != b.val:
            return False
        return same(a.left, b.left) and same(a.right, b.right)

    def dfs(node):
        if not node:
            return False
        if same(node, subRoot):
            return True
        return dfs(node.left) or dfs(node.right)

    return dfs(root)

# Dry run: root=[3,4,5,1,2], subRoot=[4,1,2]:
# same(3,4) fails -> try children; same(4,4): 1==1, 2==2, all leaves match -> True.
```

**Whiteboard trap:** in diameter/max-path-sum, returning the *through-node* value (`left + right + val`) up the recursion instead of the *single downward chain* (`val + max(left, right)`) — the through value can't extend upward.

### Tree BFS (level order)

**Spot it:** "level by level", "right side view", "zigzag", "average/max of each level", "minimum depth", "connect nodes at same level" — anything phrased per-level or nearest-first.

**Template:**
```python
from collections import deque

def level_order(root):
    if not root:
        return []
    q, out = deque([root]), []
    while q:
        level = []
        for _ in range(len(q)):       # len(q) snapshots THIS level's size
            node = q.popleft()
            level.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
        out.append(level)
    return out
```

**Amazon problems (practice in this order):**

| LC# | Problem | Diff | Approach in one line |
|---|---|---|---|
| 637 | Average of Levels in Binary Tree | E | Level loop; append `sum(level)/len(level)` |
| 102 | Binary Tree Level Order Traversal | M | The template verbatim |
| 199 | Binary Tree Right Side View | M | BFS; record the **last** node popped in each level |
| 103 | Binary Tree Zigzag Level Order Traversal | M | Normal BFS; reverse the level list when depth is odd |
| 515 | Find Largest Value in Each Tree Row | M | Level loop; take `max` per level |
| 116 | Populating Next Right Pointers in Each Node | M | Within each level set `prev.next = node` (or O(1) space via existing `next` pointers) |

**Flagship — LC 199 Right Side View:**
```python
from collections import deque

def rightSideView(root):
    if not root:
        return []
    q, out = deque([root]), []
    while q:
        n = len(q)
        for i in range(n):
            node = q.popleft()
            if i == n - 1:            # last node of this level = visible from right
                out.append(node.val)
            if node.left:  q.append(node.left)
            if node.right: q.append(node.right)
    return out

# Dry run: [1,2,3,null,5,null,4]:
# level 0 -> last=1; level 1 -> pops 2,3, last=3; level 2 -> pops 5,4, last=4. Out=[1,3,4].
```