# 04 — Backend, Databases, Performance & Security Prep

Your resume highlights **Node.js, Express, PostgreSQL, MySQL/MariaDB, MongoDB, Drizzle ORM, Redis, REST, GraphQL (Yoga), NextAuth, Better Auth, node-cron, Razorpay, ethers.js, Zipkin distributed tracing**, plus hard production debugging (`63 KB JSONB`, `27,876 rows`, `16× query speedup`).

---

## 1. Node.js Internals & Production Memory Forensics

### Must-Know Concepts
1. **Event Loop Phases:**
   - Timers (`setTimeout`/`setInterval`) → Pending I/O callbacks → Idle/prepare → Poll (incoming I/O) → Check (`setImmediate`) → Close callbacks.
   - **Microtask Queue (`process.nextTick` & Promise `.then`):** Drained *between every phase*! Starving the event loop with recursive `nextTick` or heavy synchronous `JSON.parse`/crypto blocks all HTTP requests and healthchecks.
2. **Memory Leaks & V8 Heap Debugging (Tie directly to your Comcast story):**
   - **Common Node leak patterns:** Unbounded in-memory caches/Maps, unremoved `EventEmitter` listeners, closures holding large request/DB buffers, or batch jobs loading huge TOASTed JSON payloads into arrays (`27,876 × 63 KB`).
   - **How to diagnose:** Metrics (`process.memoryUsage()` — RSS vs. `heapUsed` vs. `external`/`arrayBuffers`), `--heapsnapshot` / Chrome DevTools Memory profiler (compare 2 snapshots taken before/after load using *Comparison* view → look for retained objects & dominator tree), or `--prof` / clinic.js for CPU/event-loop lag.
3. **Streams & Backpressure:**
   - Use `stream.pipeline` / async iterators + DB cursors when exporting or processing tens of thousands of rows so memory stays `O(batch_size)` instead of `O(total_rows)`.
4. **Scaling Node:**
   - Stateless horizontal scaling behind load balancer; `worker_threads` only for CPU-bound work (image processing, crypto, AST parsing), never for I/O. Graceful shutdown (`SIGTERM`: stop accepting connections, drain active requests, close DB/Redis pools).

---

## 2. PostgreSQL & MySQL Query Optimization (Your Strongest Backend Card)

### Deep-Dive Concepts Tied to Comcast
1. **Reading `EXPLAIN (ANALYZE, BUFFERS)`:**
   - Know the difference between **Estimated vs. Actual rows** (stale stats → `ANALYZE` / skewed data), **Seq Scan vs. Index Scan vs. Index-Only Scan vs. Bitmap Heap Scan**.
   - Watch **Buffers (`shared hit` vs. `read`)** and **Sort Method (`quicksort` vs. `external merge Disk`)** and **Nested Loop** multiplying outer rows × inner work.
2. **PostgreSQL JSONB & TOAST Internals:**
   - Columns > ~2 KB get compressed and stored out-of-line in a **TOAST table**.
   - **Why `SELECT *` kills performance:** Even if an index scan finds rows fast, fetching a 63 KB JSONB column triggers random TOAST table reads + decompression + network transfer + JS parsing. **Covering indexes (`INCLUDE`)** or **column-scoped queries** avoid heap/TOAST fetches completely.
   - **JSONB indexing:** `GIN` index (`jsonb_path_ops` for `@>` containment queries), or expression B-tree index `(payload->>'status')` for equality/range on hot keys. When JSONB gets queried heavily or grows huge, promote hot fields to real typed columns or split cold blobs into a separate `1:1` detail table!
3. **Common Query Rewrites (16× Speedup Playbook):**
   - Correlated subquery per row → `JOIN` / window function / lateral join.
   - `OFFSET 50000 LIMIT 50` → **Keyset (cursor) pagination** `WHERE (created_at, id) < ($1, $2) ORDER BY created_at DESC, id DESC LIMIT 50`.
   - `OR` across different columns breaking index usage → `UNION ALL` (or BitmapOr check).
   - Leading wildcard `LIKE '%foo%'` → `pg_trgm` GIN trigram index or full-text `tsvector`.
   - Drizzle ORM pitfalls: N+1 queries inside loops, accidental `db.select()` (fetches all columns including huge JSONB!), missing `limit`, or unindexed foreign keys.
4. **Transactions, Locks & Concurrency:**
   - Isolation levels (`Read Committed` default in Postgres vs. `Repeatable Read` default in MySQL InnoDB; `Serializable` / `SELECT ... FOR UPDATE SKIP LOCKED` for job queues!).
   - **Auto-assignment job queue pattern:** `SELECT id FROM tasks WHERE status = 'pending' FOR UPDATE SKIP LOCKED LIMIT 100` prevents two worker pods from grabbing the same task!
   - Optimistic locking (`UPDATE jobs SET ..., version = version + 1 WHERE id = $1 AND version = $2`) to eliminate lost updates & stale job actions.

---

## 3. Caching (Redis), Rate Limiting & Distributed Coordination

- **Caching Patterns:**
  - *Cache-Aside (Lazy Loading):* Read Redis → miss → read DB → write Redis with TTL. Watch **Cache Stampede / Thundering Herd** (fix with single-flight coalescing, jittered TTL, or probabilistic early expiration).
  - *Write-Through / Event Invalidation:* Invalidate or update cache on domain mutation.
- **Rate Limiting (FaucetWorld & Public APIs):**
  - Token Bucket or Sliding Window Log/Counter in **Redis via atomic Lua script** (avoids race between `GET` and `INCR`).
- **Distributed Cron Locking (FaucetWorld `node-cron`):**
  - Problem: If Express runs 2+ instances, `node-cron` fires twice!
  - Fix: Redis `SET lock:payout_job <token> NX PX 60000` (with safe token-checked Lua unlock) or PostgreSQL advisory locks / BullMQ repeatable jobs, plus **idempotent ledger state** in MySQL so even a double trigger cannot double-pay.

---

## 4. API Design, GraphQL, Auth & Payments/Webhooks

### REST vs. GraphQL (GraphQL Yoga + Codegen)
- **Why GraphQL on Sphere dashboards:** Multiple widgets/maps need nested graph slices without over-fetching or multiple round trips; GraphQL Codegen gives end-to-end TypeScript safety.
- **GraphQL Pitfalls & Defenses:**
  - **N+1 problem** → `DataLoader` (per-request batching + caching).
  - **Query depth/complexity DoS** → max depth limit, query cost analysis, persisted queries in prod.
  - **Caching:** Harder at HTTP/CDN edge than REST `GET` unless using persisted GET queries or field-level caching.

### Auth (SSO, NextAuth, Better Auth)
- **Session vs. Stateless JWT:** Stateful server/DB/Redis sessions give instant revocation (logout, stolen token, role downgrade); stateless JWTs scale without DB lookup per request but require short expiry + refresh token rotation + reuse detection.
- **Enterprise SSO (OIDC / SAML):** Redirect flow, PKCE for OAuth2, `state` + `nonce` validation to stop CSRF/replay, mapping IdP groups/claims to internal RBAC permissions on the backend (never trust client-sent role headers).
- **Web Security Big 4:** XSS (`HttpOnly; Secure; SameSite=Lax` cookies, CSP, React escaping + sanitize any `dangerouslySetInnerHTML`), CSRF (SameSite cookies + CSRF token / Origin header check), SQLi (parameterized queries / Drizzle bindings), SSRF (allow-list URLs if fetching external user URLs).

### Webhooks, Payments (Razorpay) & Blockchain (`ethers.js`)
- **Webhook Idempotency (Razorpay / Offerwalls):**
  - Verify HMAC signature using raw request body (`crypto.timingSafeEqual`).
  - Store `provider_event_id` with a **UNIQUE constraint** in DB inside the same transaction that credits the user/marks order paid. Return `200 OK` on duplicate deliveries.
- **Blockchain Payouts (`ethers.js`):**
  - Never block HTTP requests on tx mining. Enqueue payout intent → serialize signer `nonce` (avoid nonce collisions across concurrent txs) → handle stuck tx (replace-by-fee / bump gas) → wait for N block confirmations before marking finalized; reconcile chain state via background poller.

---

## 5. Observability & Distributed Tracing (Zipkin)

- How trace propagation works: `traceparent` / B3 headers (`X-B3-TraceId`, `X-B3-SpanId`) passed across API Gateway → Node services → DB spans → Salesforce calls.
- Explain how you use Zipkin in real debugging (and in your MCP review server!): separating **network/queue wait** vs. **DB query duration** vs. **external Salesforce call** vs. **serialization**.

---

## 6. MongoDB, Zero-Downtime Migrations & Production Delivery (Docker / CI/CD / Feature Flags)

Because your resume lists **MongoDB, PostgreSQL, MySQL/MariaDB, Drizzle ORM, Docker, CI/CD, and feature flags**, know these operational patterns cold:

1. **MongoDB vs. PostgreSQL / MySQL (Tie to Find My Power vs. Sphere):**
   - **Why MongoDB fit Find My Power:** Polymorphic/evolving ML inference outputs (variable bounding boxes, Street View camera metadata, confidence scores) where embedding documents avoids multi-table joins on read.
   - **MongoDB Gotchas to Know:** 16 MB BSON doc limit, unbounded array growth (anti-pattern → move to separate collection with references), compound index **ESR rule** (Equality → Sort → Range), write concern (`w: "majority"`) vs. read concern trade-offs.
   - **When Postgres wins:** Relational integrity, ACID transactions across entities (Sphere finance/jobs, double-entry ledgers), complex analytical joins, and `JSONB` when you only need semi-structured attributes alongside strict foreign keys.

2. **Zero-Downtime Database Migrations (Expand-and-Contract Pattern):**
   - Never run breaking `RENAME COLUMN`, `DROP COLUMN`, or `NOT NULL` without a default (or heavy table-lock) directly during deploy while old app pods are still serving traffic!
   - **4 Safe Steps:**
     1. *Expand:* Add new column/table as nullable (or `ADD COLUMN ... DEFAULT` safely in modern Postgres/MySQL `INSTANT`).
     2. *Dual-Write:* Deploy app writing both old & new columns; backfill historical rows in throttled batches (avoid replication lag / lock contention!).
     3. *Cutover Reads:* Switch reads to new column (often behind a feature flag).
     4. *Contract:* Once old pods are gone, drop old column/triggers.
   - **Index Safety:** Always `CREATE INDEX CONCURRENTLY` in PostgreSQL (avoids blocking writes on large production tables like your 27k+ row tables).

3. **Connection Pooling in Next.js / Node.js:**
   - Traditional Express servers hold a bounded `pg.Pool` (`max: 10–20` per pod).
   - **Next.js / Serverless / Edge hazard:** Serverless functions spin up many concurrent instances and exhaust Postgres `max_connections`! Fix via **PgBouncer / RDS Proxy / Hyperdrive** (transaction pooling mode—watch out for prepared statements compatibility!).

4. **Docker, CI/CD & Feature Flags:**
   - **Docker for Next.js / Node:** Multi-stage build (`deps` → `build` → minimal `runner` with non-root `node` user), `output: 'standalone'` in `next.config` to shrink image size 5–10×, explicit `HEALTHCHECK` + `SIGTERM` graceful drain.
   - **Feature Flags & Staged Rollouts (Your AI verification gate):** Decouple *deployment* from *release*. Use flags as instant kill switches for risky migrations or AI-assisted features, clean up stale flags after 100% rollout so dead branches don't rot.
