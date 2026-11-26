# Chapter 07 — Practical Machine Coding & Senior PR Audits

**Study Priority:** `Medium`  
**Target Role Calibration:** Senior / Staff Full-Stack Engineer (Frontend-Leaning, L5/L6)

---

## 1. Objectives & Priority

- **Why This Matters:** Many Senior/Staff Full-Stack and Frontend loops replace one LeetCode round with **45-minute practical JS/TS/React machine coding** or a **Senior PR Code Review / Bug-Hunt round**. Because your resume explicitly claims **"go-to reviewer for UI and backend changes"** and **"UI Lead"**, interviewers expect effortless execution on async cancellation, concurrency control, accessible widgets, and spotting subtle production bugs.
- **Resume / Plan Gaps Addressed:**
  - Closes **Gate 3 (`plans/01`)** and operationalizes **Parts B & C of `plans/07-dsa-and-coding-drills.md`**.
  - Turns conceptual knowledge (retries, `AbortController`, DAG execution, PR audits) into muscle-memory TypeScript/React implementations.

---

## 2. Core Concepts & Mental Models

### A. Must-Cold Async JS / TypeScript Runtime Utilities

#### 1. Async Concurrency Limiter (`mapWithConcurrency`)
- Used everywhere: Find My Power Street View batching, Salesforce bulk sync, API fan-out without DDoSing downstream.
```ts
async function mapWithConcurrency<T, R>(
  items: readonly T[],
  limit: number,
  worker: (item: T, index: number) => Promise<R>
): Promise<R[]> {
  if (limit < 1) throw new RangeError('limit must be >= 1');
  const results = new Array<R>(items.length);
  let nextIndex = 0;

  async function runLane() {
    while (nextIndex < items.length) {
      const idx = nextIndex++; // safe: synchronous increment before await!
      results[idx] = await worker(items[idx], idx);
    }
  }

  const lanes = Array.from({ length: Math.min(limit, items.length) }, runLane);
  await Promise.all(lanes);
  return results;
}
```
- **Why Interviewers Probe This:** Does `idx = nextIndex++` race? No—JS is single-threaded between `await` boundaries. Also be ready to add `AbortController` so if one lane rejects, remaining lanes stop picking up new items.

#### 2. Retry with Exponential Backoff, Full Jitter & `AbortSignal`
```ts
async function retryWithBackoff<T>(
  fn: (signal: AbortSignal) => Promise<T>,
  { maxAttempts = 4, baseMs = 200, maxMs = 5000, signal }: {
    maxAttempts?: number; baseMs?: number; maxMs?: number; signal?: AbortSignal;
  } = {}
): Promise<T> {
  let lastErr: unknown;
  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    signal?.throwIfAborted();
    try {
      return await fn(signal ?? new AbortController().signal);
    } catch (err) {
      lastErr = err;
      if (signal?.aborted || attempt === maxAttempts - 1) throw err;
      const exp = Math.min(maxMs, baseMs * 2 ** attempt);
      const delay = Math.random() * exp; // Full jitter stops retry storms
      await new Promise<void>((resolve, reject) => {
        const timer = setTimeout(() => {
          signal?.removeEventListener('abort', onAbort);
          resolve();
        }, delay);
        const onAbort = () => { clearTimeout(timer); reject(signal?.reason); };
        signal?.addEventListener('abort', onAbort, { once: true });
      });
    }
  }
  throw lastErr;
}
```

#### 3. Debounce (with `cancel()` / `flush()` / `this` preservation) & Deep Clone (`WeakMap` for Cycles)
- For `debounce`: store `timerId`, `lastArgs`, `lastThis`; clear timer on every call; handle `leading` edge (`invokeNow = leading && !timerId`) vs. `trailing` edge cleanly.
- For `deepClone`: handle primitives, `null`, `Date`, `RegExp`, `Map`, `Set`, `Array`, plain objects, and check `seen = new WeakMap()` *before* recursing so circular references never stack-overflow.

#### 4. Reference `useQueryLite` Hook (Deduplication + SWR + Cancellation)
```ts
type Entry<T> = { data?: T; error?: Error; updatedAt: number; promise?: Promise<T> };
const cache = new Map<string, Entry<unknown>>();

function useQueryLite<T>(key: string, fetcher: (signal: AbortSignal) => Promise<T>, staleTimeMs = 10_000) {
  const [, forceRender] = useReducer((x: number) => x + 1, 0);
  useEffect(() => {
    const now = Date.now();
    const existing = cache.get(key) as Entry<T> | undefined;
    if (existing?.data !== undefined && now - existing.updatedAt < staleTimeMs) return;
    if (existing?.promise) { existing.promise.finally(forceRender); return; }

    const controller = new AbortController();
    const promise = fetcher(controller.signal)
      .then((data) => { cache.set(key, { data, updatedAt: Date.now() }); return data; })
      .catch((err) => {
        if (err?.name !== 'AbortError') cache.set(key, { ...existing, error: err as Error, updatedAt: existing?.updatedAt ?? 0 });
        throw err;
      })
      .finally(forceRender);

    cache.set(key, { ...existing, promise });
    return () => controller.abort();
  }, [key, staleTimeMs]);

  return cache.get(key) as Entry<T> | undefined;
}
```
> **Live Follow-Up Expectancy:** In 2026 machine coding, expect the final 10 minutes to ask *"Write 3 Vitest/Playwright tests for this."* Always test: (1) deduplicating concurrent mounts with the same `key`, (2) aborting stale request when `key` changes mid-flight, and (3) serving stale data while revalidating on error.

### B. React Machine Coding Playbook (35–45 Min Standard)

| Widget | Non-Negotiable Senior Signals |
| :--- | :--- |
| **Debounced Typeahead / Combobox** | `AbortController` cancels in-flight fetch on query change/unmount; `Map` client cache; `role="combobox"`, `aria-expanded`, `aria-activedescendant`, `ArrowUp/Down/Enter/Escape`. |
| **Custom `useQuery`-lite** | Module-level `cache` + `inFlightPromises` map (deduplicates concurrent mounts with same key!), `staleTime`, `AbortController`, subscriber notification via `Set<() => void>` or `useSyncExternalStore`. |
| **Server-Paginated Grid** | Sync `page/cursor/sort/q` with URL; skeleton state; optimistic row mutation with rollback + error toast on `409/5xx`; memoized row actions. |
| **Toast Queue + Portal** | `createPortal` to `document.body`; max visible stack; pause auto-dismiss timer on `onMouseEnter`/`onFocus` (track remaining ms!); `aria-live="polite"`. |
| **Virtualized List (10k+ rows)** | Fixed item height `H`, `scrollTop`, `startIndex = Math.max(0, Math.floor(scrollTop / H) - overscan)`, `endIndex`, `translateY(startIndex * H)` inside total-height spacer. |
| **Streaming Generative UI** | Buffer SSE chunks cleanly; validate structured JSON blocks with Zod before mounting `<Table />` / `<Chart />`; auto-scroll only if user is already near bottom (`scrollHeight - scrollTop - clientHeight < 40`). |

### C. The 10-Point Senior PR & "AI-Slop" Audit Framework
In 2026 interviews, PR review rounds often present **plausible-looking AI-generated code** with subtle architectural defects. Scan systematically in **two passes**:
1. **Security & Authz (Pass 1):** Auth/ownership check (`WHERE id = $1 AND org_id = $2`), SQL string interpolation, `dangerouslySetInnerHTML`, webhook HMAC, trusting `role` from `req.body`, SSRF on user-supplied URLs.
2. **Data & Concurrency (Pass 1):** `SELECT *` on TOAST/JSONB, unbounded fetch without `LIMIT`, N+1 inside loops, missing transaction, missing optimistic `version` / `FOR UPDATE` race, missing `AbortController`, unawaited or fail-fast `Promise.all` where `Promise.allSettled` is required.
3. **Resilience & AI-Specific Smells (Pass 2):** Hallucinated library flags/options, silent `catch {}`, missing Zod boundary validation, fixed retries without backoff/jitter, external calls inside DB transactions (holding locks open for 300ms+!).
4. **React / UI & Rollout (Pass 2):** Array index `key={i}`, `any` types, mutating arrays/objects in place (breaks React Compiler!), derived state in `useEffect`, non-semantic `<div onClick>`, breaking DB migration without expand-and-contract or feature flag.

---

## 3. Resume Tie-Ins

- **Comcast Go-To UI & Backend Reviewer:** Frame the 10-Point PR Audit Checklist as your actual mental model when reviewing Sphere PRs—and the exact rules you encoded into your **domain agent skills** and **MCP review server**.
- **Comcast Task-to-Job Race & Find My Power:** Tie `AbortController` + `mapWithConcurrency` directly to eliminating client fetch races and throttling Google Street View / ML inference pipelines.
- **DevFlow (`wtool.dev`):** Connect the **DAG Pipeline Runner** (topological sort + async/sync pure step execution + error isolation) and **Streaming Generative UI** drills directly to your live products.

---

## 4. Practice Questions & Answer Guides

### Q1 (JS/TS Machine Coding): *"Build a `runDagPipeline(nodes, edges, initialInput)` engine in TypeScript (like DevFlow) that detects cycles and executes independent branches concurrently."*
- **Solution Outline:**
  1. Compute `inDegree` and adjacency list from `edges` (`from -> to`).
  2. Run **level-by-level Kahn's BFS**: start `ready` with all `inDegree === 0` nodes.
  3. While `ready.length > 0`: run all `ready` nodes concurrently via `Promise.all(ready.map(id => executeNode(id, inputsFor(id))))`, store outputs, decrement children's `inDegree`, collect next wave's `inDegree === 0` nodes.
  4. After loop, if `executedCount < nodes.length`, throw `CycleDetectedError` (ideally validated *before* side effects via a dry topological pass first!).
  5. **Edge cases:** disconnected components, diamond dependencies (node waits for *all* parents' outputs), per-node error wrapping.

### Q2 (React Machine Coding): *"Implement a `useDebounceSearch` / Typeahead component that never displays stale results when fast typing causes out-of-order network responses."*
- **Key Implementation Talking Points:**
  - Debounce `query` (`250–300ms`) with timer cleanup.
  - Inside effect (or query trigger), create `const controller = new AbortController()`, pass `{ signal: controller.signal }` to `fetch`, and return `() => controller.abort()` in cleanup.
  - Ignore `err.name === 'AbortError'` in `catch` so aborted requests don't flash error banners.
  - Check client cache `Map<string, Result[]>` before hitting network; add keyboard nav (`activeIndex` clamped `0..items.length-1`, reset on query change) + ARIA combobox attributes.

### Q3 (Senior PR Review Drill): *"Review a 20-line endpoint + React hook (`plans/07` Part C style) that does `SELECT * FROM jobs WHERE status='PENDING'`, template-string `UPDATE`, unguarded external PO `POST`, and `useEffect` `fetch` feeding `<div onClick dangerouslySetInnerHTML>`."*
- **Structured Critique (How to Deliver Aloud in <4 Min):**
  - **Blockers (Security & Data Loss):** SQL injection via template string interpolation; missing authn/authz & trusting `role` from `req.body`; `dangerouslySetInnerHTML` XSS.
  - **High-Severity Bugs (Correctness & Scale):** `SELECT *` pulls full TOAST blobs and loads *all* `PENDING` rows to filter one ID in JS instead of `WHERE id = $1`; race condition allowing double-assignment without `AND status = 'PENDING'` / `version` / `FOR UPDATE`; external Salesforce PO call has no pre-flight eligibility check, timeout, or idempotency.
  - **Frontend Hygiene:** `useEffect` lacks `AbortController`, `encodeURIComponent(q)`, debounce, and `res.ok` check; array index `key={i}`; `<div onClick>` lacks keyboard/button semantics; `any` type.

---

## 5. Quick-Review Cheat Sheet

- **Concurrency Limiter:** Pool of `min(limit, items.length)` async workers pulling `const i = nextIndex++` before `await`.
- **Retry:** Exponential backoff + full jitter (`Math.random() * min(cap, base * 2^attempt)`) + `AbortSignal` cleanup.
- **Typeahead:** Debounce + `AbortController` + ignore `AbortError` + client cache + keyboard/ARIA (`role="combobox"`).
- **`useQuery`-lite:** Shared cache + `inFlight` promise map for dedupe + `staleTime` + subscriber cleanup.
- **PR Audit Order:** (1) Authz/SQLi/XSS → (2) `SELECT *`/N+1/Races/Tx → (3) Timeouts/Idempotency/Zod → (4) React keys/a11y/Rollout flag.
