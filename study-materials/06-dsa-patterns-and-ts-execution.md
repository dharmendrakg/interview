# Chapter 06 — DSA Patterns & TypeScript Execution

**Study Priority:** `High`  
**Target Role Calibration:** Senior / Staff Full-Stack Engineer (L5/L6)

---

## 1. Objectives & Priority

- **Why This Matters:** As `plans/README.md` warns, big-tech and top product companies still reject 11+ year Staff architects if they stall on a 45-minute blind LeetCode Medium/Hard in TypeScript/JavaScript. The goal is **pattern recognition in <2 minutes** and bug-free execution without IDE autocomplete.
- **Resume / Plan Gaps Addressed:**
  - Closes **DSA & Time-Boxed Coding** (`plans/README.md`) and **Gate 2 (`plans/01`)** ("2 unseen Mediums in 40 minutes in TS/JS").
  - Supplies reusable TypeScript pattern templates—especially **Graphs / Topological Sort** (directly tied to DevFlow DAG execution & Sphere task dependencies), **LRU Cache**, **Sliding Window**, **Binary Search on Answer**, **Monotonic Stack**, **Trees**, and **Heaps/DP**.

---

## 2. Core Concepts & Mental Models

### A. Pattern Recognition Trigger Table

| Problem Signal | Go-To Pattern | Target Complexity |
| :--- | :--- | :--- |
| "Subarray sum equals K", "pairs with target", "duplicates" | **HashMap / Prefix Sum** | `O(n)` time, `O(n)` space |
| "Longest/shortest contiguous substring/subarray with constraint" | **Sliding Window** (expand `right`, shrink `left` while invalid) | `O(n)` time |
| "Next greater element", "daily temperatures", "histogram", "valid parens" | **Monotonic Stack** | `O(n)` time |
| "Sorted/rotated array" OR "minimize the maximum / feasible check `canDo(mid)`" | **Binary Search** (Index or Answer Space) | `O(n log M)` or `O(log n)` |
| "Overlapping intervals", "meeting rooms", "merge ranges" | **Sort by Start (or End) + Greedy / Sweep Line / Min-Heap** | `O(n log n)` |
| "O(1) get/put eviction" | **LRU Cache (`Map` in JS or HashMap + Doubly Linked List)** | `O(1)` get & put |
| "Hierarchy, paths, serialization, BST validity, LCA" | **DFS (Recursion/Pre-Postorder) or BFS (Level Queue)** | `O(n)` time |
| "Prerequisites, build order, cycle detection, DAG" | **Topological Sort (Kahn's BFS in-degree or 3-color DFS)** | `O(V + E)` |
| "Top K", "streaming median", "shortest weighted path" | **Heap (Priority Queue) / Dijkstra** | `O(n log k)` / `O(E log V)` |
| "Optimal count/cost with overlapping subproblems" | **1D/2D DP** (define state + transition + base case) | `O(n)`–`O(n·m)` |

### B. Essential TypeScript Templates (Memorize These Shapes)

#### 1. Kahn's Topological Sort (DAG Execution & Cycle Detection)
```ts
// Directly applicable to Course Schedule I/II AND DevFlow DAG runner!
function topoOrder(n: number, edges: [number, number][]): number[] {
  const adj: number[][] = Array.from({ length: n }, () => []);
  const inDegree = new Array<number>(n).fill(0);
  for (const [u, v] of edges) {
    adj[u].push(v);
    inDegree[v]++;
  }
  const queue: number[] = [];
  for (let i = 0; i < n; i++) if (inDegree[i] === 0) queue.push(i);

  const order: number[] = [];
  let head = 0; // O(1) dequeue instead of Array.shift() O(n)!
  while (head < queue.length) {
    const u = queue[head++];
    order.push(u);
    for (const v of adj[u]) {
      if (--inDegree[v] === 0) queue.push(v);
    }
  }
  return order.length === n ? order : []; // empty => cycle detected
}
```
> **JS/TS Pitfall:** Never use `queue.shift()` inside BFS on large graphs (`O(n)` per shift → `O(n^2)` total). Use a `head` pointer!

#### 2. Sliding Window + Frequency Map
```ts
function lengthOfLongestSubstring(s: string): number {
  const lastSeen = new Map<string, number>();
  let left = 0, best = 0;
  for (let right = 0; right < s.length; right++) {
    const ch = s[right];
    const prev = lastSeen.get(ch);
    if (prev !== undefined && prev >= left) left = prev + 1;
    lastSeen.set(ch, right);
    best = Math.max(best, right - left + 1);
  }
  return best;
}
```

#### 3. Binary Search on Answer Space (Koko Eating Bananas / Capacity Allocation)
```ts
function minCapacity(weights: number[], days: number): number {
  let lo = Math.max(...weights);
  let hi = weights.reduce((a, b) => a + b, 0);
  const feasible = (cap: number): boolean => {
    let used = 1, cur = 0;
    for (const w of weights) {
      if (cur + w > cap) { used++; cur = 0; }
      cur += w;
    }
    return used <= days;
  };
  while (lo < hi) {
    const mid = lo + ((hi - lo) >> 1);
    if (feasible(mid)) hi = mid;
    else lo = mid + 1;
  }
  return lo;
}
```

#### 4. LRU Cache (Explain JS `Map` Shortcut AND Real DLL)
- **JS/TS Note:** `Map` preserves insertion order (`map.keys().next().value` is the oldest key), giving an `O(1)` 15-line solution (`delete` + `set` moves key to MRU). Always state aloud: *"In TypeScript, `Map` preserves insertion order for O(1) LRU; if you'd prefer language-agnostic pointers, I can implement the Doubly Linked List + HashMap."*

#### 5. MinHeap in JS/TS (No Built-in `heapq`!)
```ts
class MinHeap<T> {
  private data: T[] = [];
  constructor(private cmp: (a: T, b: T) => number) {}
  get size() { return this.data.length; }
  peek(): T | undefined { return this.data[0]; }

  push(val: T): void {
    this.data.push(val);
    let i = this.data.length - 1;
    while (i > 0) {
      const p = (i - 1) >> 1;
      if (this.cmp(this.data[i], this.data[p]) >= 0) break;
      [this.data[i], this.data[p]] = [this.data[p], this.data[i]];
      i = p;
    }
  }

  pop(): T | undefined {
    if (this.data.length === 0) return undefined;
    const top = this.data[0];
    const last = this.data.pop()!;
    if (this.data.length > 0) {
      this.data[0] = last;
      let i = 0;
      const n = this.data.length;
      while (true) {
        let best = i, l = 2 * i + 1, r = 2 * i + 2;
        if (l < n && this.cmp(this.data[l], this.data[best]) < 0) best = l;
        if (r < n && this.cmp(this.data[r], this.data[best]) < 0) best = r;
        if (best === i) break;
        [this.data[i], this.data[best]] = [this.data[best], this.data[i]];
        i = best;
      }
    }
    return top;
  }
}
```

#### 6. Union-Find (Path Compression + Union by Rank) & Multi-Part Stateful Coding (2026 Trend)
```ts
class UnionFind {
  private parent: number[];
  private rank: number[];
  components: number;
  constructor(n: number) {
    this.parent = Array.from({ length: n }, (_, i) => i);
    this.rank = new Array<number>(n).fill(0);
    this.components = n;
  }
  find(x: number): number {
    if (this.parent[x] !== x) this.parent[x] = this.find(this.parent[x]);
    return this.parent[x];
  }
  union(a: number, b: number): boolean {
    let ra = this.find(a), rb = this.find(b);
    if (ra === rb) return false;
    if (this.rank[ra] < this.rank[rb]) [ra, rb] = [rb, ra];
    this.parent[rb] = ra;
    if (this.rank[ra] === this.rank[rb]) this.rank[ra]++;
    this.components--;
    return true;
  }
}
```
- **2026 "Anti-Cheat" Stateful Design Coding Prompts:** Expect 2–3 incremental requirements stacked on one class rather than isolated puzzles—e.g. **Time-Based KV Store** (`set/get` → add `timestamp` binary search → add TTL eviction), **In-Memory File/Permissions Tree** (`Trie` + inheritance), or **Formula/DAG Cell Evaluator** (parse `=A1+B2` → topological eval + cycle detection → reactive cache invalidation on `setCell`).

### C. TypeScript Coding Interview Pitfalls
- `arr.sort()` sorts **lexicographically** (`[10, 2]` stays `[10, 2]`)! Always write `arr.sort((a, b) => a - b)`.
- `-0` vs `0` and integer division: use `Math.trunc(a / b)` or `((lo + hi) >> 1)` (only safe up to `2^31 - 1`; use `Math.floor(lo + (hi - lo) / 2)` for larger numbers).
- Prefer `Map` / `Set` over plain `{}` objects for dynamic keys (avoids `__proto__` / `"constructor"` key collisions and gives `.size` in `O(1)`).

---

## 3. Resume Tie-Ins

- **DevFlow (`wtool.dev`) DAG Execution:** Directly connects to **Topological Sort (Kahn's BFS) + Cycle Detection**—use this when asked about graph problems or designing workflow pipelines.
- **Comcast Sphere Task Dependencies & Auto-Assignment:** Connects to **Intervals / Task Scheduler / Priority Queue** (scheduling bounded retries, merging time windows, or resolving task dependency graphs).
- **Comcast Port-Connectivity Map / Network Reliability:** Connects to **BFS/DFS graph traversal & Dijkstra (`Network Delay Time`)** for modeling reachability and latency across network nodes.

---

## 4. Practice Questions & Answer Guides

### Q1 (Foundational — Prefix Sum + HashMap): *Subarray Sum Equals K*
- **Prompt:** Given `nums: number[]` (may contain negatives!) and `k: number`, return the count of contiguous subarrays summing to `k`.
- **Why Sliding Window Fails:** Negative numbers break monotonicity—shrinking the window doesn't reliably reduce the sum.
- **Solution Outline (`O(n)` time, `O(n)` space):**
  - Maintain `prefixSum` and `freq = new Map([[0, 1]])`.
  - At each `x`, `prefix += x`. Any earlier prefix equal to `prefix - k` forms a valid subarray ending here, so `count += freq.get(prefix - k) ?? 0`.
  - Increment `freq.set(prefix, (freq.get(prefix) ?? 0) + 1)`.
  - **Edge cases:** `k = 0`, single element `[k]`, negative numbers canceling out.

### Q2 (Intermediate — Graph / Topo Sort): *Course Schedule II / Task Dependency Order*
- **Prompt:** Given `numTasks` and dependency pairs `[task, prereq]`, return a valid execution order or `[]` if cyclic.
- **Solution Outline (`O(V + E)`):**
  - Build directed edge `prereq -> task` and `inDegree[task]++`.
  - Push all `inDegree === 0` nodes into queue; pop with `head` pointer, append to `order`, decrement neighbors' `inDegree`, enqueue new zeros.
  - Return `order.length === numTasks ? order : []`.
  - **Follow-up (DevFlow Parallelism):** *"How do you maximize parallel execution?"* Run level-by-level BFS—every batch currently in the queue has all dependencies satisfied and can run via `Promise.all` (or concurrency limiter)!

### Q3 (Intermediate — Design / Cache): *Implement an `LRUCache(capacity)` with `O(1)` `get` and `put`*
- **Solution Outline (`O(1)` time, `O(capacity)` space):**
  - Using JS `Map`:
    - `get(key)`: if absent return `-1`; `val = map.get(key)`; `map.delete(key); map.set(key, val); return val`.
    - `put(key, val)`: if `map.has(key)` `map.delete(key)`; `map.set(key, val)`; if `map.size > cap` `map.delete(map.keys().next().value!)`.
  - **DLL Version Readiness:** Sentinel `head` (MRU) and `tail` (LRU) dummy nodes eliminate null-pointer edge cases on insert/remove; `Map<number, Node>` maps key → DLL node.

### Q4 (Deep-Dive — Hard / Two Pointers or Heaps): *Find Median from Data Stream (or Sliding Window Maximum)*
- **Prompt (Median Stream):** Design `addNum(num)` (`O(log n)`) and `findMedian()` (`O(1)`).
- **Solution Outline:**
  - Maintain **MaxHeap `low`** (smaller half) and **MinHeap `high`** (larger half), keeping `low.size >= high.size` and `low.size - high.size <= 1`.
  - On `addNum(x)`: push to `low`, pop `low.max` and push to `high` (guarantees ordering invariant `low.max <= high.min`); if `high.size > low.size`, pop `high.min` back to `low`.
  - `findMedian()`: if `low.size > high.size` return `low.peek()`; else `(low.peek() + high.peek()) / 2`.

---

## 5. Quick-Review Cheat Sheet

- **Execution Order:** Clarify constraints (`n`, negatives, duplicates) → Brute force → Optimal `O(...)` → Code clean TS → Dry-run `[]`, `[1]`, duplicates, cycle/error case.
- **JS/TS Traps:** `arr.sort((a, b) => a - b)` (never bare `.sort()`); BFS `head++` instead of `.shift()`; `Map`/`Set` over `{}` for arbitrary keys.
- **Top-5 Must-Cold Problems:** `Subarray Sum = K`, `Longest Substring No Repeats`, `Course Schedule II (Kahn's BFS)`, `LRU Cache`, `Meeting Rooms II / Kth Largest (MinHeap)`.
- **DP Checklist:** State (`dp[i]`), Transition, Base Case, Iteration Order, Space Optimization (`O(n)` → `O(1)`).
