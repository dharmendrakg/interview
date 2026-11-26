# 09 — 1-Page Pre-Interview Cheat Sheet

Read this **30 minutes before any interview** to lock in crisp numbers, architectural hooks, and execution discipline.

---

## 1. Your 6 Signature Metrics & Proof Points
1. **11+ Years Experience:** 5 yrs NortonLifeLock (cross-browser Password Manager + React/TS portals) → 6+ yrs Comcast DE4 (Sphere UI Lead + go-to UI/backend reviewer) + 4 live solo products.
2. **3-Cause Memory Spike (Comcast Auto-Assignment):** `SELECT *` over-fetching a **63 KB JSONB** column (Postgres TOAST) × **27,876 rows** × **10-min retry loop** → eliminated with column-scoped queries.
3. **16× Intake Listing Speedup:** SQL execution plan profiling + query rewrite.
4. **Race Condition & Stale Actions:** Moved task→job resolution to task-based backend API (killed refresh race); centralized job field handling in one pure domain function.
5. **Salesforce Migration & Finance APIs:** Strangler-fig migration (Job Details, Projects & Phases, Addressing, ROE/Site Contacts) + CDC replication back + pre-flight eligibility check sparing redundant PO reservation calls.
6. **AI-First + Verification:** Domain skills + Node.js MCP server (PR diffs + Zipkin traces) + Copilot Agent Mode VS Code extension + Ollama local domain-router (invoices/jobs/permits/finance → verified tables/charts), gated by Vitest, Playwright, review & feature flags.

---

## 2. Quick Technical Numbers & Rules of Thumb
- **Latencies:** Redis `~1 ms` | Indexed DB query `~2–10 ms` | External REST (Salesforce/Stripe) `~100–500 ms`.
- **QPS Math:** `1M req/day ≈ 12 avg RPS` (peak `~35–50 RPS`). `1M rows × 1 KB = 1 GB`.
- **Core Web Vitals:** `LCP < 2.5s` | `INP < 200ms` | `CLS < 0.1`.
- **Postgres TOAST Threshold:** `~2 KB` (never `SELECT *` on tables with large JSONB/text!).
- **Queue Pattern:** `FOR UPDATE SKIP LOCKED LIMIT N` + exponential backoff + full jitter + DLQ.
- **Zero-Downtime DB Migration:** Expand (nullable) → Dual-write & backfill → Cutover reads → Contract (`CREATE INDEX CONCURRENTLY`).
- **Security Non-Negotiables:** `HttpOnly; Secure; SameSite=Lax` cookies, parameterized SQL, Zod at every boundary (Server Actions, LLM outputs, webhooks), `crypto.timingSafeEqual` + `UNIQUE(provider_event_id)` for webhooks.

---

## 3. Fast Decision Matrix (Say the Trade-Off Out Loud!)
- **MUI v7 vs. Tailwind/shadcn:** MUI v7 = enterprise speed, complex accessible widgets (Data Grid), centralized theme across squads; Tailwind/shadcn = zero CSS-in-JS runtime, bespoke product UX, RSC-friendly.
- **TanStack Query vs. Zustand:** Server cache/dedupe/refetch = TanStack Query; ephemeral client UI/map/filter state = Zustand. Never duplicate server data in Zustand!
- **Next.js 14 → 15 → 16:** Implicit caching (14) → uncached by default + async request APIs (15) → explicit `"use cache"` Cache Components + PPR + `cacheTag` (16).
- **Ollama Local vs. Cloud LLM:** Local = enterprise data privacy & zero per-token cost, needs domain routing + Zod + deterministic backend queries; Cloud (Anthropic/Groq) = stronger reasoning / ultra-low TTFT.
- **Postgres vs. MongoDB vs. Redis:** Postgres = ACID + relational + JSONB; MongoDB = polymorphic/nested docs (ML detections); Redis = sub-ms cache, Lua rate limit, locks, Pub/Sub.

---

## 4. Coding & System Design Round Reminders
- **Coding:** Clarify constraints → state brute force & optimal `O(...)` → write clean TS/JS → **dry-run `[ ]`, `[1]`, duplicates, error/cycle path** before finishing.
- **System Design:** Scope & SLA (4m) → Quick math (3m) → High-level + APIs (8m) → Schema & indexes (8m) → Deep-dives: races, idempotency, bottlenecks (15m) → Failures & rollout (5m).
- **Behavioral:** Lead with **Staff framing**—not just *"I fixed X"*, but *"I fixed X, codified the invariant in domain boundaries/skills/tests, and aligned teams so it never recurred."*
