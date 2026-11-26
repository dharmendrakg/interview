# Chapter 02 — Backend Internals, SQL/TOAST & Production Forensics

**Study Priority:** `High`  
**Target Role Calibration:** Senior / Staff Full-Stack Engineer (Frontend-Leaning, L5/L6)

---

## 1. Objectives & Priority

- **Why This Matters:** Your strongest backend credibility comes from production forensics (`63 KB JSONB`, `27,876 rows`, `16× query speedup`), cross-system finance APIs, and running real Drizzle/MySQL/Redis/Express services. Interviewers will test whether you truly understand **PostgreSQL storage/query execution, Node.js V8/Event Loop mechanics, distributed locks/rate limiting, webhook idempotency, and zero-downtime migrations**.
- **Resume / Plan Gaps Addressed:**
  - Closes **Stack Breadth vs. Internals Depth** (`plans/README.md`) across Node.js, PostgreSQL, MySQL/InnoDB, Drizzle ORM, Redis, GraphQL Yoga, Auth, and Docker.
  - Equips you to pass **Gate 5 (`plans/01`)**: reading `EXPLAIN (ANALYZE, BUFFERS)` cold and diagnosing Node.js heap/event-loop stalls step by step.

---

## 2. Core Concepts & Mental Models

### A. Node.js Event Loop, V8 Memory & Streams

```text
  ┌───────────────────────────┐
  │ timers (setTimeout)       │◄──┐
  └─────────────┬─────────────┘   │
  ┌─────────────▼─────────────┐   │  Between EVERY phase transition:
  │ pending I/O callbacks     │   ├──► drain process.nextTick() queue
  └─────────────┬─────────────┘   │    then drain Promise microtask queue!
  ┌─────────────▼─────────────┐   │
  │ poll (incoming I/O, DB)   │   │  (Blocking CPU work or recursive nextTick
  └─────────────┬─────────────┘   │   starves I/O, healthchecks & timers)
  ┌─────────────▼─────────────┐   │
  │ check (setImmediate)      │   │
  └─────────────┬─────────────┘   │
  ┌─────────────▼─────────────┐   │
  │ close callbacks           ├───┘
  └───────────────────────────┘
```

- **Memory Anatomy (`process.memoryUsage()`):**
  - `rss`: Total OS resident memory (code, stack, V8 heap, C++ buffers).
  - `heapTotal` / `heapUsed`: V8 managed JS objects (Young Gen / Scavenger vs. Old Gen / Mark-Sweep-Compact).
  - `external` / `arrayBuffers`: C++ objects & `Buffer`s (`pg` wire buffers, file streams) outside V8 heap.
- **Why JSON Bloat Kills Node Containers:**
  - A `63 KB` JSONB string across `27,876` rows (`~1.75 GB` UTF-8) arrives in `external` socket buffers, duplicates as JS strings, expands into V8 objects/maps/arrays (`3×–10×` memory amplification), survives young-gen scavenger when held in a retry array, promotes to **Old Space**, and triggers stop-the-world Major GC pauses or container OOM (`SIGKILL 137`).
- **Heap Debugging Workflow:**
  1. Monitor `heapUsed` sawtooth vs. climbing floor + event loop lag (`perf_hooks.monitorEventLoopDelay`).
  2. Capture two `--heapsnapshot` files (after warm-up, then after load) or inspect via `node --inspect` + Chrome DevTools **Comparison** view.
  3. Sort by `#Delta` / **Retained Size** and trace the **Retainer / Dominator tree** (usually an unbounded `Map`, unremoved `EventEmitter` listener, closure over `req`/`res`, or unpaginated query array).
- **Streams & Backpressure:** For exports or bulk processing, use `pg-cursor` / async iterators piped through `stream.pipeline(...)` so memory stays `O(batch_size)` rather than `O(N)`.
- **Graceful Shutdown (`SIGTERM`):** Stop accepting new HTTP connections (`server.close()`), let in-flight requests drain up to a deadline, close DB/Redis pools, and exit cleanly—essential in Kubernetes/Docker rolling deploys.

### B. PostgreSQL Internals: TOAST, `EXPLAIN (ANALYZE, BUFFERS)`, Indexes & Locks

- **TOAST (The Oversized-Attribute Storage Technique):**
  - Postgres pages are `8 KB`. Columns exceeding ~`2 KB` (`JSONB`, `TEXT`) are compressed and moved out-of-line into a companion `pg_toast` table, leaving only an ~18-byte pointer in the main heap tuple.
  - **Key Insight:** If your query selects only scalar columns (`SELECT id, status FROM tasks`), Postgres **never reads or decompresses the TOAST table**. `SELECT *` forces random TOAST I/O + decompression on every matching row.
  - **Fix Beyond Projection:** Promote hot JSONB keys to typed columns or expression indexes `(payload->>'status')`, use `GIN` (`jsonb_path_ops`) for `@>` containment queries, and split cold blobs (`63 KB` payloads) into a `1:1` `task_payloads` table.

- **Reading `EXPLAIN (ANALYZE, BUFFERS)` Like a Staff Engineer:**
  - **Estimated vs. Actual Rows (`rows=10` vs. `actual rows=50000`):** Stale `ANALYZE` stats or correlated columns causing bad join plans (e.g., Nested Loop chosen instead of Hash Join).
  - **Scan Types:** `Index Only Scan` (fastest—heap fetches = 0 when visibility map is clean) > `Index Scan` > `Bitmap Heap Scan` (good for medium selectivity / combining indexes) > `Seq Scan` (fine for tiny tables or >15–20% table reads; fatal on large hot lookups).
  - **`BUFFERS` (`shared hit` vs. `read`):** `hit` = RAM cache; `read` = physical disk I/O. High `hit` count (millions of buffers) still burns CPU—often from an N+1 Nested Loop or missing index!
  - **`Sort Method: external merge Disk`:** Query exceeded `work_mem`, spilling sort/hash to disk.

- **Annotated `EXPLAIN (ANALYZE, BUFFERS)` Drill (Spot the 3 Bottlenecks):**
  ```text
  Limit  (cost=18420.12..18425.40 rows=25 width=840) (actual time=1920.4..1921.1 rows=25 loops=1)
    Buffers: shared hit=142800 read=38410
    -> Sort  (actual time=1920.3..1920.5 rows=25 loops=1)
         Sort Method: external merge  Disk: 48120kB
         -> Seq Scan on intake_jobs j  (actual rows=185000 loops=1)
              Filter: (status = 'OPEN' AND region = 'WEST')
              Rows Removed by Filter: 620000
              SubPlan 1
                -> Aggregate  (actual rows=1 loops=185000)
                     -> Index Scan using idx_tasks_job on tasks t  (actual rows=4 loops=185000)
  ```
  1. *Width `840` + `read=38410` + `48 MB` Disk Sort:* Fetching wide/JSONB columns *before* `LIMIT 25` forces sorting huge rows on disk!
  2. *`Seq Scan` removing 620k rows:* Missing partial/composite index `ON intake_jobs (region, created_at DESC, id DESC) WHERE status = 'OPEN'`.
  3. *`SubPlan 1` (`loops=185000`):* Correlated subquery executed 185,000 times before `LIMIT 25`! **Fix:** Keyset/indexed `LIMIT 25` CTE *first*, then join `tasks` counts only for the 25 returned IDs (typically delivers `10×–30×` speedups like your `16×` intake win).

- **High-ROI SQL Rewrites (The 16× Speedup Playbook):**
  1. **Deep Pagination:** `OFFSET 50000 LIMIT 50` scans and discards 50,000 rows → **Keyset Pagination:** `WHERE (created_at, id) < ($1, $2) ORDER BY created_at DESC, id DESC LIMIT 50` with a composite index `(created_at DESC, id DESC)`.
  2. **Correlated Subqueries:** `SELECT ..., (SELECT count(*) FROM items WHERE items.job_id = j.id)` runs per outer row → aggregate once via `LEFT JOIN (SELECT job_id, count(*) ... GROUP BY job_id)` or window functions.
  3. **`OR` Across Distinct Columns:** `WHERE a = $1 OR b = $2` often degrades to `Seq Scan` → `UNION ALL` (or deduplicated `UNION`) uses separate indexes on `a` and `b`.
  4. **Drizzle ORM Traps:** `db.select()` with no column list emits `SELECT *` (pulling TOAST blobs!); relational `with:` queries can over-fetch or trigger N+1 patterns if not checked against generated SQL.

- **Locks, Queues & Concurrency:**
  - **Job Queue Without Redis:**
    ```sql
    WITH claimed AS (
      SELECT id FROM jobs
      WHERE status = 'PENDING' AND next_run_at <= now()
      ORDER BY next_run_at ASC
      FOR UPDATE SKIP LOCKED
      LIMIT 100
    )
    UPDATE jobs j SET status = 'RUNNING', attempts = attempts + 1
    FROM claimed c WHERE j.id = c.id
    RETURNING j.id, j.task_type;
    ```
    `SKIP LOCKED` lets multiple Node pods claim disjoint batches with zero lock contention.
  - **Optimistic Locking (Stale Job Actions):**
    `UPDATE jobs SET ..., version = version + 1 WHERE id = $1 AND version = $2` → `rowCount === 0` returns `409 Conflict`.
  - **Postgres vs. MySQL/InnoDB:** Postgres defaults to `Read Committed` (no gap locks, MVCC creates dead tuples requiring `VACUUM`); MySQL InnoDB defaults to `Repeatable Read` with **clustered primary keys** (use monotonic `BIGINT` or `UUIDv7` for InnoDB PKs—random `UUIDv4` fragments B-trees!).

### C. Redis Patterns, Webhooks, Auth & Zero-Downtime Migrations

- **Redis Patterns:**
  - **Cache-Aside & Stampede Defense:** If a hot key expires under burst traffic, 500 requests hit the DB simultaneously. Fix with **single-flight request coalescing** (in-process/distributed lock), **jittered TTLs**, or **stale-while-revalidate**.
  - **Atomic Rate Limiting:** Always use a **Redis Lua script** (or `MULTI` with care) so reading count + checking limit + incrementing + setting `EXPIRE` executes atomically on Redis's single thread.
  - **Distributed Cron Lock:** `SET lock:payout <uuid_token> NX PX 60000`; release only via Lua comparing `ARGV[1] == redis.call('GET', KEYS[1])` so a slow worker never deletes another worker's lock.

- **GraphQL Yoga + DataLoader:**
  - GraphQL resolves fields recursively (`Job → Tasks → Assignee`), creating classic N+1 query storms.
  - Attach a **per-request `DataLoader`** in Yoga context to batch keys in one event-loop tick (`WHERE id = ANY($1)`) and cache within that single request. Protect public/authenticated endpoints with max query depth, complexity cost limits, and persisted queries.

- **Auth & Web Security Boundaries:**
  - **Stateful Sessions (Better Auth / Redis) vs. JWT:** Stateful sessions allow instant revocation (role downgrade, stolen device logout); stateless JWTs require short TTLs (`5–15m`) + rotating refresh tokens with **reuse detection**.
  - **Enterprise SSO (OIDC):** Enforce `PKCE`, `state` (CSRF protection), and `nonce` (replay protection); map IdP groups to RBAC permissions **server-side**.
  - **Webhook & Payment Idempotency (Razorpay / Offerwalls):**
    1. Verify HMAC on **raw request body** using `crypto.timingSafeEqual`.
    2. Insert `provider_event_id` into a table with a `UNIQUE` constraint **inside the same SQL transaction** that credits balance or marks order paid. Return `200 OK` on conflict (`23505`).

- **Zero-Downtime DB Migrations (Expand-and-Contract):**
  1. **Expand:** Add nullable new column / new table; create indexes with `CREATE INDEX CONCURRENTLY` (never lock writes on large prod tables).
  2. **Dual-Write & Backfill:** Deploy app writing both old & new fields; backfill old rows in throttled batches (`LIMIT 1000` + sleep) to avoid replication lag.
  3. **Cutover Reads:** Switch reads to new schema (behind a feature flag).
  4. **Contract:** Drop old column only after all old app pods have drained.

- **REST API Contracts, Idempotency & Observability (Staff Signals):**
  - **Mutating REST Idempotency:** Support `Idempotency-Key` headers on POST endpoints (e.g., PO reservation / payout creation); store `(key, request_hash, response_status, response_body)` in Redis/DB—reject mismatched payloads with `422` and replay identical retries safely. Use `ETag` + `If-Match` for HTTP optimistic concurrency (`412 Precondition Failed`).
  - **Backward-Compatible Evolution:** Additive optional fields only; never rename/remove JSON fields or tighten enum inputs without versioning or dual-reading; return standardized error shapes (`RFC 9457` Problem Details).
  - **Observability Beyond Zipkin Spans:**
    - *RED / USE Metrics:* Rate, Errors, Duration (`p50/p95/p99` histograms—never averages!) for services; Utilization, Saturation, Errors for DB pools/Node event-loop lag.
    - *Cardinality Trap:* Never put unbounded IDs (`job_id`, `user_id`, raw URLs) into Prometheus metric labels (explodes TSDB memory); keep high-cardinality IDs in **structured logs** and **Zipkin trace tags**.
    - *Tail vs. Head Sampling:* Head sampling decides at trace start (misses rare 5xx/slow traces); tail sampling buffers briefly at collector and keeps 100% of errors + slow outliers (`>1s`).

---

## 3. Resume Tie-Ins

- **Comcast Auto-Assignment Spike:** Anchor on `SELECT *` + TOAST (`63 KB`) + `27,876` rows + `10-min` retry + V8 heap amplification. Show how separating hot queue metadata from cold JSONB blobs + `SKIP LOCKED` solves this structurally.
- **Comcast 16× Intake Listing Speedup:** Pair `EXPLAIN (ANALYZE, BUFFERS)` thinking with keyset pagination, removing correlated subqueries, and checking Drizzle-generated SQL.
- **Comcast Cross-System Finance API & Stale Job Fix:** Connect `version`-based optimistic locking and state-machine guards to stopping stale job actions; connect Zipkin B3/`traceparent` spans to isolating slow external Salesforce PO calls vs. local DB spans.
- **FaucetWorld (`node-cron`, MySQL, Redis) & DevFlow (Razorpay, Drizzle, Better Auth):** Use FaucetWorld to illustrate Redis distributed cron locks + MySQL double-entry idempotent ledgers; use DevFlow to illustrate Razorpay raw-body HMAC + `UNIQUE(event_id)` transactions and `pg_pool`/serverless connection pooling (`PgBouncer` / RDS Proxy).

---

## 4. Practice Questions & Answer Guides

### Q1 (Foundational): *"Explain what happens in PostgreSQL and Node.js when an endpoint runs `SELECT *` on 25,000 rows containing a 60 KB JSONB column."*
- **Answer Guide:**
  1. **PostgreSQL:** Values > ~2 KB are stored compressed out-of-line in `pg_toast`. `SELECT *` forces random TOAST page fetches + LZ/pglz decompression for all 25k rows, blowing shared buffers and sending ~1.5 GB over TCP.
  2. **Node.js / V8:** `pg` allocates wire buffers (`external`), decodes UTF-8 strings, and `JSON.parse()` builds nested V8 heap objects (`3×–10×` amplification). Holding the result array promotes objects to Old Space, stalling the event loop during Major GC or triggering container OOM.
  3. **Fix:** Column projection (`SELECT id, status`), `LIMIT`/keyset pagination or DB cursor streaming, and isolating cold blobs into a separate `1:1` table.

### Q2 (Intermediate): *"You look at an `EXPLAIN (ANALYZE, BUFFERS)` plan and see `Nested Loop` with `actual rows=1` estimated, `actual rows=45000` real, and `Buffers: shared hit=920000`. What happened and how do you fix it?"*
- **Answer Guide:**
  - **Diagnosis:** Severe cardinality misestimation (estimated 1 row, actual 45,000) tricked the planner into picking a `Nested Loop` instead of a `Hash Join` or `Merge Join`. For each of the 45k outer rows, it probed the inner index/heap ~20 times (`~920k` buffer hits = massive CPU burn even though data was cached in RAM).
  - **Root Causes & Fixes:** Run `ANALYZE` (stale stats or autovacuum lagging); check for correlated predicates (add `CREATE STATISTICS` extended stats); rewrite non-sargable predicates (`WHERE lower(email) = ...` without an expression index, or `OR`/correlated subqueries) so accurate selectivity estimates unlock a `Hash Join`.

### Q3 (Deep-Dive): *"In FaucetWorld, how do you guarantee that a `node-cron` payout worker running across 3 Express containers neither skips a payout nor double-pays a user?"*
- **Answer Guide:**
  1. **Single-Runner Coordination:** Acquire a Redis lock (`SET lock:payout <uuid> NX PX 60000`) or use Postgres `FOR UPDATE SKIP LOCKED` at the row level so multiple pods can't claim the same `PENDING` payout rows.
  2. **Idempotent Financial State (Source of Truth):** Redis locks can expire if GC pauses a worker! The ultimate safety net is an **ACID MySQL transaction**: transition `payouts.status` with `WHERE id = ? AND status = 'PENDING'`, and write an immutable `ledger_entries` row keyed by a unique `idempotency_key`. Even a duplicate cron trigger hits a no-op / unique constraint.
  3. **External Side-Effect Safety:** Claim row as `PROCESSING` *before* calling `ethers.js`, persist `tx_hash`, and reconcile confirmations asynchronously.

### Q4 (Deep-Dive): *"You need to rename and backfill a non-null status column on a 10-million-row Postgres table with zero downtime. Walk through the migration steps."*
- **Answer Guide:**
  - Use **Expand-and-Contract**:
    1. **Expand:** `ALTER TABLE ... ADD COLUMN new_status TEXT;` (nullable, metadata-only instant lock; acquire short `lock_timeout` so it doesn't queue behind long queries!). Add indexes with `CREATE INDEX CONCURRENTLY`.
    2. **Dual-Write:** Deploy app code writing both `old_status` and `new_status` (falling back to `old_status` on reads).
    3. **Backfill:** Update historical rows in small throttled batches (`LIMIT 2000` with brief pauses) so autovacuum and replication lag stay healthy.
    4. **Validate & Cutover:** Add `NOT NULL` safely via `ADD CONSTRAINT ... CHECK (new_status IS NOT NULL) NOT VALID`, then `VALIDATE CONSTRAINT` (avoids long `ACCESS EXCLUSIVE` scan lock!). Flip reads to `new_status` behind a feature flag.
    5. **Contract:** Drop `old_status` in a later deploy after old pods terminate.

---

## 5. Quick-Review Cheat Sheet

- **Node Event Loop:** Microtasks (`nextTick`, Promises) drain between *every* phase—sync CPU (`JSON.parse` on 1.5 GB) blocks everything.
- **V8 Heap Debug:** `process.memoryUsage()` + 2 `--heapsnapshot`s in DevTools Comparison view → sort by Retained Size / `#Delta`.
- **Postgres TOAST (~2 KB):** Never `SELECT *` on blob/JSONB tables; split cold blobs or use covering indexes (`INCLUDE`).
- **Query Fixes:** `OFFSET` → keyset `(created_at, id) < ($1, $2)`; correlated subquery → `JOIN`/`LATERAL`; `OR` → `UNION ALL`; `LIKE '%x%'` → `pg_trgm`.
- **Concurrency:** Job queue = `FOR UPDATE SKIP LOCKED LIMIT N`; optimistic update = `WHERE id=$1 AND version=$2` (`409 Conflict`).
- **Webhooks & Locks:** Raw-body `crypto.timingSafeEqual` + `UNIQUE(provider_event_id)` in same DB tx; Redis Lua for atomic rate limits & token-checked unlocks.
- **Migrations:** Expand (nullable + `CONCURRENTLY`) → Dual-write + throttled backfill → `NOT VALID` + `VALIDATE` constraint → Cutover → Contract.
