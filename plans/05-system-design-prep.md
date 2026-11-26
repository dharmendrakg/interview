# 05 — System Design Prep (Backend + Frontend)

At 11+ years (Senior / Staff), system design is graded on **clarity of framing, trade-off reasoning, failure handling, data modeling, and knowing where bottlenecks actually live**. Expect either a **Backend/Full-Stack System Design** or a **Frontend Architecture Design**.

---

## 1. Universal 45-Minute Interview Structure

### Backend / Full-Stack (45 min)
1. **Clarify Scope & Constraints (4–5 min):** Functional requirements (top 3 flows), non-functional (QPS, data volume, read/write ratio, latency SLA, consistency vs. availability, legacy constraints).
2. **Back-of-the-Envelope Math (3 min):** QPS (peak = 3× avg), storage growth/year, payload size (remember your `63 KB × 27k` intuition!).
3. **High-Level Architecture & APIs (8–10 min):** Clients, API Gateway/BFF, core services, DBs, async queues/CDC, key REST/GraphQL endpoints.
4. **Data Model & Storage Choice (8 min):** Tables/collections, primary keys, indexes, why Postgres vs. MySQL vs. MongoDB vs. Redis.
5. **Deep Dives & Bottlenecks (15 min):** Drive this proactively! Pick 2–3 hardest parts: race conditions, idempotency, CDC sync loops, cache stampede, backpressure, observability, **SLIs/SLOs & error budgets**.
6. **Failure Modes, Governance & Rollout (4–5 min):** What breaks when downstream (e.g., Salesforce/payment provider) is slow or down? Circuit breakers, DLQs, feature flags, phased migration, plus **Staff governance** (RFC/ADR trade-offs, build-vs-buy-vs-wrap, incremental tech-debt retirement).

### Frontend Architecture (45 min — RADIO Framework)
1. **R — Requirements:** Users, devices, offline/realtime needs, SEO vs. authenticated app, a11y/i18n.
2. **A — Architecture:** Rendering (RSC/SSR/CSR/PPR), BFF/API layer (REST vs. GraphQL), state ownership (URL vs. Server Cache vs. Client Store), micro-frontends vs. modular monorepo.
3. **D — Data Model:** Normalized client cache, pagination/cursor shapes, optimistic updates, WebSocket/SSE delta shapes.
4. **I — Interface (APIs & Component Contracts):** Design-system primitives, props/slots, error/loading boundaries.
5. **O — Optimizations:** Bundle splitting, virtualization, image/font loading, debouncing/cancellation, Core Web Vitals (LCP, INP, CLS), telemetry.

---

## 2. Six Domain-Matched Mock Design Prompts (Practice All 6!)

### Prompt 1: Legacy CRM (Salesforce) to Modern Platform Migration with Bi-Directional Sync
*(Directly mirrors Comcast Sphere!)*
- **Core Problem:** Migrate 4 complex enterprise domains off Salesforce without downtime while legacy modules still depend on Salesforce data.
- **Key Architecture:**
  - **Strangler Fig + BFF/Domain API:** Route migrated capabilities to Sphere Postgres; keep unmigrated ones proxied.
  - **Outbox Pattern + CDC:** Write domain mutation + `outbox_events` row in **one ACID Postgres transaction** → Debezium/relay publishes to Kafka/queue → sync worker pushes batched updates to Salesforce Bulk/REST API.
  - **Hard Problems to Highlight:**
    1. **Preventing Infinite Echo Loops:** Tag every change with `source_system` (`SPHERE` vs. `SFDC`) + `correlation_id`; ignore echoed updates with matching origin/hash.
    2. **Salesforce Rate Limits & Out-of-Order Events:** Coalesce/debounce rapid updates per entity ID before hitting Salesforce; partition queue by `job_id` to preserve per-entity ordering.
    3. **Reconciliation:** Nightly drift-detection job comparing checksums/timestamps between Sphere and Salesforce.

---

### Prompt 2: High-Throughput Background Auto-Assignment & Retry Engine
*(Directly mirrors your memory-spike fix + production robustness!)*
- **Core Problem:** Assign tens of thousands of incoming tasks to field/ops resources reliably, with retries on transient external failures (e.g., finance eligibility / PO checks).
- **Key Architecture:**
  - **Lean Job Table vs. Blob Table:** Keep hot scheduling columns (`id, status, attempt_count, next_run_at, lock_owner, task_type`) in a narrow table; move 63 KB+ JSONB payloads to a separate `task_payloads` table fetched *only* by workers that actually execute the payload.
  - **Queueing & Locking:** Either Postgres `FOR UPDATE SKIP LOCKED` with bounded batch size (`LIMIT 100`) or Redis/Kafka worker queues.
  - **Retry Strategy:** Exponential backoff with full jitter (`next_run_at = now + min(cap, base * 2^attempt) + rand`) + **Dead-Letter Queue (DLQ)** after N attempts so poisoned rows never spin forever every 10 minutes.
  - **Pre-Flight Eligibility Gate:** Check cheap local rules/cached eligibility before calling slow external finance/PO reservation APIs.

---

### Prompt 3: Enterprise Data Grid + Design System Architecture (Frontend System Design)
*(Mirrors Sphere UI Lead + MUI X Data Grid + 27k+ records)*
- **Core Problem:** Support 20+ complex operational views with filtering, inline edits, bulk actions, saved views, and strict accessibility across multiple teams.
- **Key Architecture:**
  - **Server-Driven Grid Contract:** URL query params (`?cursor=...&sort=...&filter=...`) as source of truth so views are shareable/refresh-safe; backend returns rows + facet counts + capabilities/permissions per row.
  - **Stale Action Prevention:** Backend computes `allowedActions` / centralized domain validator; UI disables invalid actions AND backend enforces state-machine transitions.
  - **Performance:** Column/row virtualization (`MUI X Data Grid`), memoized cell renderers, optimistic inline edit with TanStack Query rollback on `409 Conflict`, code-split heavy drawers/modals.
  - **Design System Governance:** Versioned `@org/ui` monorepo package wrapping MUI v7 theme tokens + Storybook + automated Playwright/axe-core a11y checks.

---

### Prompt 4: Real-Time Network Connectivity & Reliability Dashboard
*(Mirrors your Next.js 15 + GraphQL + Zustand + Live Charts + Map dashboard)*
- **Core Problem:** Visualize live port connectivity and regional HFC reliability across thousands of network nodes on an interactive map + time-series charts.
- **Key Architecture:**
  - **Map Scale:** Never render 50k DOM markers! Use server-side **geohash/quadtree clustering** by zoom + bounding-box viewport queries, rendered on WebGL/Canvas (Mapbox/Deck.gl).
  - **Live Telemetry Updates:** SSE/WebSocket stream pushing delta metrics every few seconds; buffer incoming ticks in a ref/queue and flush to Zustand/charts on a 500ms–1s cadence to keep main thread <16ms frames (protecting INP).
  - **Data Fetching:** Initial shell via SSR/RSC + SSO token validation; GraphQL Codegen typed queries for node drill-down drawer; TanStack Query `staleTime` + window focus throttling.

---

### Prompt 5: Browser-Based Visual Pipeline Builder (DevFlow `wtool.dev`)
*(Frontend + Hybrid AI System Design)*
- **Core Problem:** Let developers chain JSON/Base64/JWT/Regex/Diff transforms into a visual DAG running 100% locally in the browser, with optional AI auto-fix/explanation.
- **Key Architecture:**
  - **Execution Engine:** Topological sort of DAG nodes; each node is a pure function `(input, config) => Result<Output, Error>` executed in a **Web Worker** (prevents huge payloads/regex catastrophic backtracking from freezing the React UI thread, with execution timeout cancellation).
  - **Zero-Data-Leakage Privacy Boundary:** Local transforms never call servers; only explicit "Explain / Fix with AI" sends user-approved snippet/error to Next.js API route → Vercel AI SDK (Groq for low-latency fixes, Anthropic for deep explanations), with rate limiting via Better Auth + Redis + Razorpay tier quotas.
  - **Persistence:** Next.js 16 Cache Components for public tool docs/templates; Drizzle + MySQL for saved user pipelines.

---

### Prompt 6: Automated Rewards & Blockchain Payout Engine (FaucetWorld)
*(Backend / Distributed Reliability Design)*
- **Core Problem:** Run scheduled crypto payouts, third-party offerwall callbacks, and affiliate rewards reliably with strong fraud resistance.
- **Key Architecture:**
  - **Double-Entry Ledger (MySQL ACID):** Never just `UPDATE users SET balance = balance + X`. Write immutable `ledger_entries (id, user_id, delta, type, idempotency_key)` + update balance atomically.
  - **Offerwall Callback Security:** Verify provider HMAC/IP allow-list + unique `tx_id` idempotency key.
  - **Anti-Abuse:** Redis sliding-window rate limits, velocity anomaly flags (payout hold queue for suspicious spikes).
  - **Payout Worker:** `node-cron` guarded by distributed lock pulls `PENDING` withdrawals in batches → single signer nonce manager submits via `ethers.js` → tracks `tx_hash` → confirmation watcher transitions `BROADCASTED` → `CONFIRMED` (or safe retry if dropped/underpriced).

---

## 3. Core Building Blocks & Back-of-the-Envelope Cheat Sheet

### Latency & Scale Numbers (Keep in Your Head)
- **Redis / In-memory lookup:** ~0.5–1 ms | **Indexed Postgres/MySQL query (same AZ):** ~2–10 ms | **External REST call (e.g., Salesforce/Stripe):** ~100–500 ms.
- **1 day = 86,400 sec (~10⁵ sec):** `1M req/day ≈ 12 avg RPS` (peak ~35–50 RPS); `100M req/day ≈ 1,200 avg RPS` (peak ~3,600 RPS).
- **Storage sanity check:** `1M rows × 1 KB = 1 GB`. `27,876 rows × 63 KB = ~1.75 GB` raw payload!

### Storage & Queue Decision Matrix
- **PostgreSQL:** Default for relational integrity, ACID transactions, JSONB + GIN/expression indexes, `FOR UPDATE SKIP LOCKED` job queues.
- **MySQL / MariaDB:** Great OLTP workhorse (`InnoDB` clustered PK—keep PKs sequential like UUIDv7/BIGINT to avoid page splits!).
- **MongoDB:** Polymorphic/nested documents (ML detections, CMS blocks), horizontal sharding, low join needs.
- **Redis:** Sub-ms cache, atomic Lua rate limiters, distributed locks, Pub/Sub fan-out, ephemeral state (not primary source of truth for money).
- **Kafka / SQS vs. Postgres Queue:** Use `SKIP LOCKED` up to moderate throughput (~hundreds/sec) to avoid extra infra; move to Kafka/SQS when you need multi-consumer replay, strict partition ordering at high throughput, or decoupled event streaming.

### Classic Architectural Traps to Call Out Proactively
1. **Read-Replica Lag ("Read-Your-Own-Writes"):** After a user updates a Sphere job, a subsequent `GET` hitting a replica may return stale data! Fix: route creator's reads to primary for N seconds (session/LSN watermark) or return updated entity in mutation response.
2. **Real-Time Fan-Out at Scale:** One Node pod holds ~10k–50k idle SSE/WebSocket connections; scale horizontally using **Redis Pub/Sub** (or NATS/Kafka) so Pod A publishing an event reaches a user connected to Pod B.
3. **Thundering Herd / Retry Storms:** Downstream outage + fixed retries = synchronized traffic spikes. Always use **exponential backoff + full jitter + circuit breaker**.

---

## 4. Two Classic Unseen System Design Prompts (Beyond Your Resume)

### Prompt 7: Multi-Tenant Webhook & Notification Delivery Platform
- **Problem:** Deliver millions of webhook/email events reliably with per-tenant rate limits, retries, and delivery logs.
- **Key Design:** Ingestion API validates + persists event (`idempotency_key`) → partitioned queue → per-tenant token bucket rate limiter (so one noisy tenant doesn't starve others — *noisy neighbor isolation*) → worker signs payload (`HMAC-SHA256` + timestamp header to stop replay) with strict 5s HTTP timeout → tiered retry queues (`1m, 5m, 30m, 2h` + DLQ) + circuit breaker when a customer endpoint returns 5xx continuously.

### Prompt 8: Distributed Rate Limiter (API Gateway + Redis)
- **Problem:** Enforce `100 req/min` per API key and `10 req/sec` per IP across 20 stateless Node/API gateway pods with <2 ms overhead.
- **Key Design:** Sliding-Window Counter or Token Bucket implemented as an **atomic Redis Lua script** (avoids `GET`→`SET` race conditions); fail-open vs. fail-closed trade-off if Redis is unreachable (fail-open with local in-memory fallback for non-billing routes; fail-closed for auth/payout endpoints); return standard `429 Too Many Requests` + `Retry-After` & `X-RateLimit-Remaining` headers.
