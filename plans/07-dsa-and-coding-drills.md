# 07 — DSA & Practical JS/TS/React/Node Coding Drills

For Senior/Staff Full-Stack loops, interviews split into two coding styles:
1. **Algorithmic DSA (45 min):** Clean TypeScript/JavaScript solutions, complexity analysis, edge cases.
2. **Practical / Machine Coding (45–60 min):** Real-world async JS utilities, React widgets, or Node backend modules.

---

## Part A: High-ROI 30–35 Problem Core (From the 60-Problem List)

> **11+ YOE Triage Rule:** Do **not** grind all 60 problems blindly unless targeting pure FAANG algorithmic loops. Focus first on **~30–35 high-ROI problems** across **Arrays/Hashing, Sliding Window, Binary Search on Answer, Intervals, LRU/Design, Trees, Graphs/Topological Sort, and Heaps**, plus **Multi-Part Stateful Coding** (`study-materials/06`). Deprioritize heavy 2D DP (`Word Break`, `Edit Distance`) unless a target company explicitly tests them.

Solve these in **TypeScript/JavaScript** without IDE autocomplete. Aim for **20–25 min per Medium**, speaking your approach before coding.

### 1. Arrays, Hashing & Two Pointers (10)
- [ ] Two Sum (HashMap baseline)
- [ ] Group Anagrams
- [ ] Top K Frequent Elements (Bucket sort `O(n)` vs. Heap `O(n log k)`)
- [ ] Product of Array Except Self (`O(1)` extra space)
- [ ] Longest Consecutive Sequence (`Set` `O(n)`)
- [ ] Valid Palindrome / Container With Most Water
- [ ] 3Sum (sorting + two pointers + duplicate skipping)
- [ ] Subarray Sum Equals K (Prefix sum + HashMap — *super common!*)
- [ ] Merge Sorted Array / Remove Duplicates in-place
- [ ] Trapping Rain Water (Hard classic — two pointers)

### 2. Sliding Window (6)
- [ ] Best Time to Buy and Sell Stock
- [ ] Longest Substring Without Repeating Characters
- [ ] Longest Repeating Character Replacement
- [ ] Permutation in String
- [ ] Minimum Window Substring (Hard — master frequency map tracking)
- [ ] Sliding Window Maximum (Monotonic Deque)

### 3. Stack & Monotonic Stack (6)
- [ ] Valid Parentheses
- [ ] Min Stack (`O(1)` min)
- [ ] Evaluate Reverse Polish Notation
- [ ] Daily Temperatures (Monotonic decreasing stack)
- [ ] Car Fleet / Asteroid Collision
- [ ] Largest Rectangle in Histogram (or Basic Calculator II for string parsing)

### 4. Binary Search & Intervals (8)
- [ ] Binary Search / Search in Rotated Sorted Array
- [ ] Find Minimum in Rotated Sorted Array
- [ ] Koko Eating Bananas (Binary search on answer space!)
- [ ] Time Based Key-Value Store (HashMap + Binary Search)
- [ ] Merge Intervals
- [ ] Insert Interval
- [ ] Non-overlapping Intervals (Greedy)
- [ ] Meeting Rooms II (Min-heap or sweep line — *very common!*)

### 5. Linked List & Design Structures (6)
- [ ] Reverse Linked List (Iterative & Recursive)
- [ ] Linked List Cycle / Find Duplicate Number (Floyd's slow/fast)
- [ ] Merge Two Sorted Lists / Merge K Sorted Lists
- [ ] Copy List with Random Pointer
- [ ] **LRU Cache** (HashMap + Doubly Linked List — *top-5 most asked for JS/Node!* Note: JS `Map` preserves insertion order, know both `Map` shortcut AND real DLL implementation!)
- [ ] Design Hit Counter / Rate Limiter (Queue or circular bucket array)

### 6. Trees & Tries (10)
- [ ] Maximum Depth / Invert Binary Tree
- [ ] Same Tree / Subtree of Another Tree
- [ ] Binary Tree Level Order Traversal (BFS)
- [ ] Binary Tree Right Side View
- [ ] Validate Binary Search Tree
- [ ] Lowest Common Ancestor of a Binary Tree
- [ ] Kth Smallest Element in a BST
- [ ] Construct Binary Tree from Preorder & Inorder
- [ ] Serialize and Deserialize Binary Tree
- [ ] Implement Trie (Prefix Tree) + Word Search II (Trie + Backtracking)

### 7. Graphs, Topological Sort & Union-Find (8)
*(Critical for DevFlow DAG execution & Sphere task dependency interviews!)*
- [ ] Number of Islands (DFS/BFS)
- [ ] Clone Graph
- [ ] Rotting Oranges (Multi-source BFS)
- [ ] **Course Schedule I & II** (Topological Sort / Kahn's BFS + cycle detection — *practice cold!*)
- [ ] Pacific Atlantic Water Flow
- [ ] Network Delay Time (Dijkstra's shortest path)
- [ ] Number of Connected Components / Redundant Connection (Union-Find with path compression)
- [ ] Word Ladder (Shortest path BFS)

### 8. Heaps & Dynamic Programming (6)
- [ ] Kth Largest Element in an Array / Task Scheduler (Priority Queue / Greedy)
- [ ] Find Median from Data Stream (Two Heaps)
- [ ] Climbing Stairs / House Robber
- [ ] Coin Change (Bottom-up 1D DP)
- [ ] Longest Increasing Subsequence (`O(n log n)` patience sorting or `O(n^2)`)
- [ ] Word Break / Unique Paths / Edit Distance (pick 2)

---

## Part B: 15 Practical JS/TS/React/Node Coding Drills (Must-Master!)

Senior Frontend/Full-Stack interviews often replace a LeetCode round with one of these:

### JavaScript / TypeScript Runtime Utilities
*(Priority 1 = #1, #2, #3, #7, #8 below; skip memorizing #4–#6 polyfills unless targeting a frontend-trivia screen.)*
1. **`debounce(fn, wait, { leading, trailing })` & `throttle(fn, wait)`** — handle `this` binding, args, return `.cancel()` / `.flush()`.
2. **Async Concurrency Limiter (`mapWithConcurrency(items, limit, asyncFn)`)** — runs at most `N` promises concurrently, preserves output order, stops or collects errors cleanly.
3. **Retry with Exponential Backoff + Jitter + AbortSignal** — directly mirrors your backend retry experience!
4. **Deep Clone (`structuredClone` edge cases: circular refs via `WeakMap`, `Date`, `Map`, `Set`, arrays)** & **Deep Equal**.
5. **Typed EventEmitter / PubSub** — `on`, `off`, `once`, `emit` with memory-leak-safe unsubscribe functions.
6. **Promise.all / Promise.allSettled / Promise.race Polyfills** — zero bugs on empty arrays or non-promise primitives.
7. **Token Bucket / Sliding Window Rate Limiter** — in-memory TS class + explain Redis Lua adaptation.
8. **DAG Pipeline Runner (DevFlow style)** — given nodes + edges + async/sync transforms, detect cycles, execute in topological order passing outputs downstream.

### React / Frontend Machine Coding (35–45 min each)
9. **Debounced Typeahead / Autocomplete** — `AbortController` race-condition cancellation, client cache, keyboard navigation (`ArrowUp/Down/Enter/Esc`), ARIA attributes (`role="combobox"`, `aria-activedescendant`).
10. **Server-Paginated, Sortable, Filterable Data Table** — URL query sync, loading skeletons, optimistic row action with rollback on API failure.
11. **Multi-Step Form Wizard with Validation** — persisted draft state, per-step validation, dirty-state navigation guard.
12. **Custom `useQuery`-lite Hook** — cache by key, dedupe concurrent identical requests, stale-while-revalidate, `AbortController`, cleanup on unmount.
13. **Toast / Notification Queue + Portal** — auto-dismiss timers paused on hover, stacking limit, accessible `aria-live` region.
14. **Infinite Scroll / Virtualized List** — `IntersectionObserver` sentinel or windowing math (`startIndex`, `endIndex`, `paddingTop/Bottom`) for 10,000+ rows.
15. **Live Streaming Chat / Generative UI Renderer** — parse streaming chunks, render safe structured table/chart JSON blocks, auto-scroll unless user scrolled up.

---

## Part C: Senior/Staff Code Review & Debugging Drills

Because your resume positions you as the **go-to reviewer for UI and backend changes** and a **root-cause diagnostician**, many Senior/Staff loops include a **PR Review / Bug-Hunt round** instead of (or alongside) LeetCode.

### 10-Point Senior PR Audit Checklist
1. **Correctness & State Transitions:** Can stale fields/actions overwrite newer state? Is there a race condition or missing optimistic lock (`version`)?
2. **Concurrency & Async Cleanup:** Missing `AbortController` in `useEffect` / search? Unhandled `Promise.all` partial failure? Missing React effect cleanup?
3. **Database & Query Shape:** `SELECT *` pulling huge TOAST/JSONB blobs? Unbounded query without `LIMIT`? N+1 inside a loop? Missing index on `WHERE`/`ORDER BY`?
4. **Transactions & Idempotency:** Two DB writes without a transaction? Webhook/cron without unique `idempotency_key` or distributed lock?
5. **Security:** Missing authz/ownership check (`WHERE id = $1 AND user_id = $2`)? `dangerouslySetInnerHTML` XSS? Raw SQL interpolation? Unverified webhook HMAC?
6. **Memory & Performance:** Unbounded `Map`/listener leak? Loading 25k rows into Node/browser memory instead of streaming/paginating?
7. **Error Handling & Observability:** Swallowed `catch {}`? Missing structured log/trace context? Retry loop without backoff/max-attempts?
8. **Type Safety & Validation:** Trusting `req.body as MyType` without runtime Zod parsing? Missing exhaustive `never` check?
9. **React / UI Hygiene:** Derived state duplicated in `useEffect`? Unstable inline object/function props breaking memoized heavy grids? Missing keyboard/ARIA accessibility?
10. **Rollout Safety:** Breaking DB migration without expand-and-contract? Risky behavior missing a feature flag?

### Practice Drill: Spot All Bugs in This 25-Line PR Snippet (Target: < 5 Min)
```ts
// Backend route + React hook from a fictional PR — find at least 7 bugs!
app.post('/api/jobs/:id/assign', async (req, res) => {
  const { assigneeId, role } = req.body;
  const jobs = await db.query('SELECT * FROM jobs WHERE status = $1', ['PENDING']); // 1, 2
  const job = jobs.find((j) => j.id === req.params.id); // 3
  if (!job) return res.status(404).send('Not found');

  await db.query(`UPDATE jobs SET assignee_id = '${assigneeId}', status = 'ASSIGNED' WHERE id = '${job.id}'`); // 4, 5
  await fetch(SALESFORCE_PO_URL, { method: 'POST', body: JSON.stringify({ jobId: job.id }) }); // 6
  res.json({ ok: true, role });
});

function JobSearch({ onSelect }: { onSelect: (j: any) => void }) {
  const [q, setQ] = useState('');
  const [results, setResults] = useState([]);
  useEffect(() => {
    fetch(`/api/jobs?q=${q}`).then(r => r.json()).then(setResults); // 7, 8, 9
  }, [q]);
  return results.map((r, i) => <div key={i} onClick={() => onSelect(r)} dangerouslySetInnerHTML={{ __html: r.title }} />); // 10, 11
}
```
> **Answer Key (check only after spotting them yourself):** (1) `SELECT *` over-fetches TOASTed JSONB + scans all `PENDING` rows instead of `WHERE id = $1`; (2) no auth/authz check (`req.user` / role from body!); (3) unbounded fetch + O(N) JS `.find()` instead of SQL lookup; (4) SQL injection via template string interpolation; (5) race condition—no `AND status = 'PENDING'` / `version` check or `FOR UPDATE`, so two requests double-assign; (6) external Salesforce PO call has no pre-flight eligibility check, timeout, or retry/idempotency; (7) no `AbortController` → stale search race condition; (8) missing `encodeURIComponent(q)` + debounce; (9) no `.ok` check / error handling; (10) array index `key={i}` + `any` type; (11) `dangerouslySetInnerHTML` XSS + non-accessible clickable `<div>` instead of `<button>`.

---

## Coding Interview Execution Checklist

1. **Before typing:** Restate inputs/outputs, constraints (`n` size, duplicates, negative numbers, async errors), and confirm signature.
2. **Think aloud:** State brute-force (`O(n^2)`) → bottleneck → optimal structure (`O(n log n)` / `O(n)`).
3. **Write clean idiomatic TS/JS:** Meaningful variable names, helper functions, guard clauses.
4. **Dry-run manually:** Trace `[ ]`, `[1]`, duplicates, cycle/error path out loud *before* saying "I'm done."
5. **State Time & Space complexity** accurately (including call stack / output size if applicable).
