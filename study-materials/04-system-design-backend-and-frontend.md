# Chapter 04 — System Design (Backend & RADIO Frontend)

**Study Priority:** `High`  
**Target Role Calibration:** Senior / Staff Full-Stack Engineer (L5/L6)

---

## 1. Objectives & Priority

- **Why This Matters:** System Design is the primary leveling gate between Senior (L5) and Staff (L6). Interviewers look for **structured scoping, back-of-the-envelope math, clean data modeling, proactive bottleneck/failure-mode analysis, and trade-off articulation** across both Backend/Distributed Systems and Frontend Architecture.
- **Resume / Plan Gaps Addressed:**
  - Closes **System Design Framing (Senior vs. Staff)** (`plans/README.md`) and **Gate 4 (`plans/01`)**.
  - Turns your real projects (Salesforce→Sphere CDC, Auto-Assignment Engine, MUI X Data Grid, Live Connectivity Dashboard, DevFlow DAG builder, FaucetWorld payouts) plus 2 unseen classics (Multi-Tenant Webhooks, Distributed Rate Limiter) into crisp 45-minute whiteboard blueprints.

---

## 2. Core Concepts & Mental Models

### A. 45-Minute Execution Blueprints

| Phase | Backend / Full-Stack (45 min) | Frontend Architecture — RADIO (45 min) |
| :--- | :--- | :--- |
| **0–5 min** | **Clarify Scope & SLAs:** Top 3 flows, QPS, data volume, read/write ratio, consistency vs. availability, legacy constraints. | **R — Requirements:** Personas, devices, real-time vs. polling, offline, SEO vs. authenticated app, a11y/i18n. |
| **5–8 min** | **Capacity Math:** `1M req/day ≈ 12 avg RPS` (peak `3×`), `1M rows × 1 KB = 1 GB`, payload × batch size. | **A — Architecture:** Rendering (RSC/SSR/PPR/CSR), BFF (REST/GraphQL), state boundaries (URL/Server/Client), modular vs. MFE. |
| **8–18 min** | **High-Level Design & Data Model:** APIs, core tables/indexes, storage choice (Postgres vs. MySQL vs. Mongo vs. Redis), async queues/CDC. | **D & I — Data Model & Interface:** Normalized cache, cursor pagination, optimistic/delta shapes, design-system component contracts. |
| **18–35 min** | **Proactive Deep Dives (2–3):** Races, idempotency, CDC echo loops, backpressure, cache stampede, hot partitions. | **O — Optimizations:** Virtualization, WebGL/Canvas vs. DOM, Web Workers, throttling, Core Web Vitals (`LCP/INP/CLS`). |
| **35–45 min** | **Failure Modes & Rollout:** Circuit breakers, DLQs, expand-and-contract migrations, feature flags, Zipkin/alerts. | **Resilience & Observability:** Error boundaries, offline/reconnect UX, RUM telemetry, a11y/E2E gates. |

### B. Universal Scale & Storage Rules of Thumb
- **Latencies:** Redis `~0.5–1 ms` | Indexed Postgres/MySQL (same AZ) `~2–10 ms` | External REST (Salesforce/Stripe) `~100–500 ms`.
- **Worked 3-Minute Whiteboard Math Example (Sphere Intake + Telemetry):**
  - Assume `50k` daily active field/ops users × `40` API reads/day = `2M req/day` ≈ `24 avg RPS` (peak `~75 RPS`; cache hit 80% → `15 DB QPS`).
  - Telemetry writes: `20k` network nodes emitting a `500 B` heartbeat every `15s` = `~1,333 writes/sec` (`~667 KB/sec` ≈ `57 GB/day` raw → batch/downsample into Timescale/ClickHouse or rollups rather than hammering OLTP Postgres!).
  - Always check payload × batch size: `27,876 rows × 63 KB = 1.75 GB`—a single unprojected background sweep dwarfs normal user traffic!
- **Storage vs. Queue Choice:**
  - Use PostgreSQL `FOR UPDATE SKIP LOCKED` for moderate background queues (`<500 jobs/sec`) when you want ACID coupling with domain tables and zero extra queue infra.
  - Move to **Kafka / SQS** when you need multi-consumer event replay, strict per-key partition ordering at high throughput, or decoupled cross-service streaming.
- **Three Traps to Call Out Proactively:**
  1. **Read-Replica Lag ("Read-Your-Own-Writes"):** Route a mutating user's reads to primary briefly (or return the mutated entity directly in the mutation response).
  2. **SSE / WebSocket Fan-Out Across Pods:** A single Node pod holds ~10k–50k idle sockets; publish events over **Redis Pub/Sub** (or NATS) so Pod A reaches clients connected to Pod B.
  3. **Retry Storms / Thundering Herd:** Fixed-interval retries synchronize outages; always use **exponential backoff + full jitter + circuit breaker + DLQ**.

### C. Six Domain-Matched + Two Unseen Design Blueprints

#### Blueprint 1: Legacy CRM (Salesforce) → Modern Platform Migration (CDC + Shared Finance APIs)
```text
  Sphere UI ──► Sphere API ──► [TX: Update Jobs + Insert outbox_events (origin='SPHERE')]
                                         │
                                         ▼
                              Debezium / Outbox Relay ──► Kafka (partitioned by job_id)
                                                                 │
                                                                 ▼
  Salesforce ◄── Coalescing Worker (Bulk/REST, echo filter) ◄────┘
```
- **Core Mechanisms:** Strangler Fig slice-by-slice cutover; **Transactional Outbox** prevents dual-write inconsistency between Postgres and Kafka; `origin_system` + `correlation_id` stops infinite bi-directional echo loops; per-`job_id` partitioning preserves ordering; **pre-flight eligibility gate** avoids wasting `100–500 ms` and Salesforce API quota on ineligible PO reservations; nightly checksum reconciliation repairs drift.

#### Blueprint 2: High-Throughput Auto-Assignment & Retry Engine
- **Core Mechanisms:** Split `tasks` (narrow hot table: `id, status, attempt_count, next_run_at, priority`) from `task_payloads` (`63 KB` JSONB blob fetched only during execution); claim batches with `FOR UPDATE SKIP LOCKED LIMIT 100`; retry transient external failures using `min(cap, base * 2^attempt) + random_jitter`; route poison pills to a **Dead-Letter Queue (DLQ)** after N attempts instead of spinning every 10 minutes forever.

#### Blueprint 3: Enterprise Data Grid & Design System Architecture (Frontend RADIO)
- **Core Mechanisms:** URL `searchParams` (`cursor`, `sort`, `filters`) as source of truth; backend returns `{ rows, nextCursor, facets, allowedActions }` so stale client action logic cannot drift; **MUI X Data Grid** row/column virtualization; TanStack Query optimistic inline edits with `version`-conflict (`409`) rollback; monorepo `@org/ui` package with Storybook + Playwright/axe-core gates.

#### Blueprint 4: Real-Time Network Connectivity & Reliability Dashboard
- **Core Mechanisms:** Initial shell via Next.js RSC + SSO validation; map renders >10k nodes via server-side **geohash/quadtree clustering** by zoom + bounding-box query on **Canvas/WebGL** (Mapbox/Deck.gl, never 50k SVG nodes); SSE/WebSocket deltas buffered in a `ref` and flushed to Zustand/charts every `500ms` so main thread stays under `16ms`.

#### Blueprint 5: Browser-Based Visual DAG Pipeline Builder (`DevFlow` / `wtool.dev`)
- **Core Mechanisms:** Client-side DAG validated with **Kahn's topological sort** (cycle detection before run); each node runs as a pure `(input, config) => Result` step inside a **Web Worker** (prevents regex catastrophic backtracking or 50 MB JSON parsing from freezing the React main thread, supports worker termination timeout); explicit opt-in boundary before sending user-approved snippets to Next.js `/api/ai` (Groq for low-latency repair, Anthropic for deep explanation).

#### Blueprint 6: Automated Rewards & Blockchain Payout Engine (`FaucetWorld`)
- **Core Mechanisms:** Immutable **double-entry `ledger_entries`** inside MySQL ACID transactions; offerwall webhook verification (`HMAC` + IP allow-list + `UNIQUE(provider_tx_id)`); Redis sliding-window rate limits + anomaly hold queue; distributed-locked `node-cron` payout worker with serialized Ethereum signer `nonce` manager, gas-bump replacement for stuck txs, and N-confirmation finalization watcher.

#### Blueprint 7 (Unseen Classic): Multi-Tenant Webhook & Notification Delivery Platform
- **Architecture:** Ingestion API persists event + `idempotency_key` → partitioned queue → **per-tenant token-bucket rate limiter** (*noisy neighbor isolation* so Tenant A's 100k spike doesn't delay Tenant B) → delivery worker signs payload (`HMAC-SHA256` + timestamp header against replay) with strict `5s` timeout → tiered retry queues (`1m, 5m, 30m, 2h` + DLQ) + **per-endpoint circuit breaker** when a customer URL returns sustained 5xx.

#### Blueprint 8 (Unseen Classic): Distributed Rate Limiter (API Gateway + Redis)
- **Architecture:** Stateless gateway pods call an **atomic Redis Lua script** implementing Sliding-Window Counter or Token Bucket (`<2ms` overhead; eliminates `GET`→`SET` race conditions). Return `429 Too Many Requests` with `Retry-After` & `X-RateLimit-Remaining`. **Degradation trade-off:** Fail-open with local in-memory fallback for non-critical read APIs if Redis blips; fail-closed for auth/payout endpoints.

#### Blueprint 9 (2026 Modern Classic): Enterprise AI Assistant (RAG + Structured DB Tools + RBAC)
*(Directly extends your Sphere Ollama Assistant & DevFlow AI)*
- **Core Problem:** Answer natural-language questions across both **structured operational DBs** (invoices, jobs, permits) and **unstructured SOP/policy docs**, respecting user RBAC permissions without hallucinating numbers.
- **Key Architecture:**
  - **Two-Stage Intent & Tool Router:** Classify query → route to *Structured Tool Execution* (parameterized Postgres/API queries returning verified rows) vs. *Hybrid Document Retrieval* (BM25 lexical + dense vector embeddings fused via Reciprocal Rank Fusion + reranker).
  - **Pre-Retrieval ACL Filtering:** Never filter permissions *after* LLM generation! Enforce `tenant_id` / `role` metadata filters **inside** the vector/BM25 query and DB queries (`WHERE org_id = $userOrg`).
  - **Streaming & Guardrails:** Stream SSE tokens + typed Generative UI blocks (`Zod`-validated); attach clickable source citations (`docId + chunk`); enforce per-user token/RPM budgets in Redis; log traces/evals (TTFT, schema validity, thumbs-down) asynchronously.

---

## 3. Resume Tie-Ins

- **Blueprint 1 & 2** directly elevate your Comcast Sphere migration, finance eligibility check, and `63 KB × 27,876` auto-assignment story from "what I fixed" to "how I architect reliable distributed workflows."
- **Blueprint 3 & 4** directly frame your Sphere UI Lead role, MUI v7 design system governance, and Next.js 15/GraphQL/Zustand connectivity dashboard under the RADIO frontend rubric.
- **Blueprint 5 & 6** turn `wtool.dev` and `faucetworld.in` into concrete proofs of Web Worker concurrency, financial ledger integrity, and autonomous cron/blockchain systems.

---

## 4. Practice Questions & Answer Guides

### Q1 (Foundational — Backend): *"When designing a background job processor for 200 jobs/sec in a Postgres + Node.js stack, would you start with Postgres `SKIP LOCKED` or Kafka? Justify the trade-off."*
- **Answer Guide:**
  - **Start with Postgres `SKIP LOCKED`** at 200 jobs/sec if jobs are tightly coupled to existing relational state (like Sphere task assignment). Why: transactional consistency (update domain row + enqueue job in one ACID tx), zero new broker ops overhead, and easy dead-letter querying via SQL.
  - **Move to Kafka/SQS when:** Throughput climbs into thousands/sec, multiple independent consumer groups need replayable event logs (like CDC replication to Salesforce + analytics + audit), or long-lived high-volume streams would bloat Postgres MVCC vacuuming.

### Q2 (Intermediate — Frontend RADIO): *"Design an enterprise data grid supporting 50,000 rows, saved team filter views, inline editing, and optimistic updates."*
- **Answer Guide:**
  - **Requirements & Architecture:** URL `searchParams` own active sort/filter/cursor; saved views stored server-side by `viewId` and hydrate URL state on load.
  - **Data & Contract:** Cursor-based pagination + backend-computed `allowedActions` per row + `version` token.
  - **Optimizations:** MUI X Data Grid virtualization, memoized cells, `AbortController`/TanStack Query cancellation on rapid filter changes, optimistic `onMutate` patch with snapshot rollback on `409 Conflict` / `5xx`, and `aria-live` announcements for inline save errors.

### Q3 (Deep-Dive — Distributed Systems): *"In a multi-tenant webhook delivery platform, one tenant's endpoint hangs for 30 seconds and starts dropping packets. How do you stop them from taking down delivery for all other tenants?"*
- **Answer Guide:**
  1. **Strict Timeouts:** Enforce short connect (`1–2s`) and total request (`5s`) timeouts so workers aren't pinned indefinitely.
  2. **Bulkhead / Per-Tenant Quotas:** Separate dispatch queues or enforce per-tenant concurrency caps & token buckets before workers dequeue.
  3. **Circuit Breaker:** Track rolling 5xx/timeout rate per `endpoint_id` in Redis; trip circuit `OPEN` for e.g. 5 minutes, diverting events to delayed retry/DLQ and notifying the tenant instead of burning worker threads.

### Q4 (Deep-Dive — Fintech / Ledger): *"Why is `UPDATE users SET balance = balance + 10` dangerous in a rewards/payment system, and what schema replaces it?"*
- **Answer Guide:**
  - **Why It Fails:** Destroys auditability, cannot detect double-credits on webhook retries, and suffers lost updates or race conditions under concurrent spend/earn transactions.
  - **Correct Design:** **Double-entry / immutable `ledger_entries` table** (`id, account_id, delta, direction, reference_type, reference_id, idempotency_key UNIQUE, created_at`) inside an ACID transaction (`SELECT balance FROM accounts WHERE id = $1 FOR UPDATE` before debiting to enforce non-negative balance). Account balance can be derived or materialized atomically alongside the ledger insert.

### Q5 (2026 Modern — AI Architecture): *"Design an enterprise AI assistant that answers questions across both live Postgres finance/job tables and internal PDF policy docs without leaking unauthorized data or hallucinating figures."*
- **Answer Guide:**
  1. **Query Routing:** Lightweight classifier routes structured data questions (invoice totals, job statuses) to **parameterized Postgres/API tools**, and unstructured policy questions to **Hybrid RAG (BM25 + vector + reranker)**.
  2. **RBAC Pre-Filtering (Never Post-Filter!):** Pass the user's `org_id` and RBAC roles from the verified server session into *both* SQL `WHERE` clauses and vector-index metadata filters *before* top-K chunks enter the LLM context.
  3. **No Fabricated Math:** SQL/backend tools compute all aggregations and return verified rows + typed Generative UI blocks (`Zod`-validated); policy answers must cite `doc_id` + chunk excerpt.
  4. **Reliability & Cost:** Stream via SSE with `req.signal` abort handling, Redis per-user RPM/token rate limits, and offline golden evals in CI.

---

## 5. Quick-Review Cheat Sheet

- **Backend 45-Min Flow:** Scope/SLA (4m) → Math (3m) → High-level + APIs (10m) → Schema/DB (8m) → Deep-dives (15m) → Failures/Rollout (5m).
- **Frontend RADIO:** **R**equirements → **A**rchitecture → **D**ata Model → **I**nterface → **O**ptimizations.
- **CDC Echo Loops:** Transactional Outbox + `origin_system` + `correlation_id` + per-entity partition ordering + nightly reconciliation.
- **Retry Hygiene:** Narrow scheduling table (`SKIP LOCKED`) + exponential backoff + full jitter + DLQ + circuit breaker.
- **Live Maps/Charts:** Geohash server clustering + WebGL/Canvas + `ref` buffer flushed every `500ms`.
- **Rate Limiter:** Atomic Redis Lua script; fail-open (reads) vs. fail-closed (auth/payments).
