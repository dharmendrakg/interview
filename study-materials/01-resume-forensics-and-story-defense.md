# Chapter 01 — Resume Forensics & Story Defense

**Study Priority:** `High`  
**Target Role Calibration:** Senior / Staff Full-Stack Engineer (Frontend-Leaning, L5/L6)

---

## 1. Objectives & Priority

- **Why This Matters:** At 11+ years, interviewers assume baseline competence and spend 20–40% of technical and hiring-manager rounds **stress-testing your own resume claims**. One vague answer on a headline metric (`16× speedup`, `63 KB JSONB`, `CDC back to Salesforce`) creates instant skepticism across the entire loop.
- **Resume / Plan Gaps Addressed:**
  - Closes every `[VERIFY]` prompt flagged in `plans/02-resume-deep-dive-stories.md` before you enter live screens.
  - Bridges the gap between *listing* an outcome ("eliminated memory spikes") and *defending* it 3 layers deep under hostile follow-ups ("what about TOAST, heap amplification, batching, DLQ, and why tests missed it?").
  - Prevents your 5 years at NortonLifeLock (2015–2020) and 4 solo products from sounding like stale maintenance or weekend side projects.

---

## 2. Core Concepts & Mental Models

### The 3-Layer Story Defense Model
Every major resume bullet must survive three escalating depths of questioning:
1. **Layer 1 — The 90-Second Executive Narrative (Situation → Root Cause/Decision → Impact):** Crisp numbers, clear ownership, zero rambling.
2. **Layer 2 — The Architectural Trade-Off ("Why Not X?"):** Why you chose column projection / task-based API / Strangler Fig / Ollama routing over the naive alternative, and what your choice *cost*.
3. **Layer 3 — Production Forensics & Next-Step Guardrails:** Exact mechanism under the hood (e.g., PostgreSQL TOAST, V8 heap amplification, echo-loop prevention) + what you institutionalized afterward so the class of bug cannot recur (Staff signal).

### The "Honesty Under Probe" Rule
Never invent metrics or claim ownership of an ML model or infra layer someone else built. High-level engineers build instant trust by stating clean boundaries:
- **Bad:** *"We built an ML model for Street View power supplies."* (Collapses when asked about loss functions or training hyperparams.)
- **Strong:** *"I owned the full-stack ingestion, async orchestration, MongoDB storage, and React/MUI human-in-the-loop verification UI, while partnering with [ML team / external vision service] on inference."*

### Pre-Interview `[VERIFY]` Closure Checklist
Before scheduling interviews, fill in concrete mental anchors (staying strictly inside NDA/confidentiality bounds) for the 7 unverified spots from `plans/02`:

| Story | `[VERIFY]` Item to Lock In | Safe Way to Frame It |
| :--- | :--- | :--- |
| **Auto-Assignment Memory Spike** | Did you also ship pagination, backoff, or DLQ alongside column projection? | Distinguish *"what stopped the bleeding immediately (column projection)"* vs. *"follow-up hardening shipped or proposed (cursor batching, backoff, DLQ)"*. |
| **16× Intake Listing Speedup** | Exact baseline → final latency & exact SQL rewrite. | Know whether it was e.g. `~3.2s → ~200ms` and whether the root fix was eliminating a correlated subquery / `OFFSET` / redundant join / `OR` defeating an index. |
| **Salesforce CDC & Finance API** | Exact CDC mechanism (logical decoding/Debezium/outbox table → SFDC Bulk/REST/Platform Events) & failure retry. | Explain the exact sync boundary and how partial failures / governor limits were buffered or retried. |
| **MUI v7 Design System** | Approximate consumer count (squads/modules/engineers) & primary pain solved. | E.g., *"Shared across N Sphere modules / M engineers, unifying tables/forms/theme tokens during MUI v5→v7."* |
| **Find My Power (CV)** | Which model/API ran inference & how false positives/negatives were handled. | Clarify your pipeline/UX boundary + confidence threshold + human verification step before field dispatch. |
| **Solo Products** | 1–2 real operational metrics (`faucetworld.in` uptime/payout volume, `wtool.dev` tool count/test count). | Concrete numbers turn side projects into proof of product & operational ownership. |
| **NortonLifeLock (2015–2020)** | 1 specific feature or gnarly bug owned in Password Manager / React+TS portal. | E.g., SPA autofill event dispatch, cross-browser extension messaging race, or portal auth flow. |

### Handling the Two Classic Long-Tenure & Builder-Risk Questions
Interviewers reviewing a 6+ year Comcast tenure alongside 4 live solo products almost always test two subtle concerns:
1. **"Why are you leaving Comcast after 6+ years?"**
   - *Trap:* Complaining about legacy systems, slow teammates, or management.
   - *Strong Framing:* *"I've had tremendous scope on Sphere—leading UI/design system, migrating 4 core domains off Salesforce, and building AI review tooling—and I'm proud of what we shipped. Now I'm looking for my next inflection point: a high-ownership Senior/Staff environment where AI-native product engineering, deep full-stack architecture, and tight feedback loops are core to the business."*
2. **"You run 4 live products (`wtool.dev`, `faucetworld.in`, etc.)—how do we know they won't compete with your day job?"**
   - *Strong Framing:* *"They're intentionally architected to be zero-touch and automated—FaucetWorld has run autonomously on `node-cron` + Redis + MySQL since 2019 with no daily manual work, and DevFlow runs client-side with automated Vitest/Playwright gates. Building real products on nights/weekends is actually how I sharpen production instincts—payment webhooks, Next.js 16 Cache Components, and Vercel AI SDK patterns—that I bring straight back to my team."*

---

## 3. Resume Tie-Ins

### Tie-In A: Comcast Sphere — Auto-Assignment Memory Spikes (Flagship Forensic Story)
- **Raw Metrics:** `63 KB` JSONB column × `27,876` rows × `10-minute` retry loop → eliminated via column-scoped queries.
- **Why It Works at Staff Level:**
  - `27,876 × 63 KB` is `~1.75 GB` of raw UTF-8 JSON per retry cycle. In PostgreSQL, values > ~2 KB live out-of-line in **TOAST tables**; `SELECT *` forces PostgreSQL to read/decompress TOAST pages, stream GBs over the socket, and trigger **3×–10× V8 heap amplification** during `JSON.parse()` (strings, hidden classes, old-space promotion).
  - Because retries ran every 10 minutes, the heap never fully settled before the next sweep arrived.
  - **Fix & Guardrail:** Column projection stopped PostgreSQL from touching TOAST completely. Follow up by explaining defense-in-depth (bounded `FOR UPDATE SKIP LOCKED LIMIT 100` batches, moving cold blobs to a `1:1` detail table, exponential backoff + DLQ, and adding a "no `SELECT *` on blob tables" rule to PR reviews & domain agent skills).

### Tie-In B: Comcast Sphere — API Boundaries & Domain Purity
- **Task-to-Job Refresh Race + Stale Job Actions:**
  - *Problem 1:* Client resolved `task → job` via staggered async calls; page refresh or fast navigation triggered out-of-order responses and stale UI state. Moving resolution into a single **task-based backend API** eliminated the waterfall and race window at the source.
  - *Problem 2:* Multiple update endpoints/handlers mutated job fields inconsistently, letting stale actions leak through. Centralizing field handling into **one pure domain function** created a single source of truth for state transitions and field sanitization (pair with optimistic concurrency `version` checks in `WHERE`).

### Tie-In C: Comcast Sphere — Strangler Fig Migration & Finance Eligibility Gate
- Migrated **Job Details, Projects & Phases, Addressing, Right of Entry/Site Contacts** off Salesforce into Sphere with **CDC replication back** to Salesforce.
- Shared Finance APIs (labor rates, funding approvals, PO reservations) included a **pre-flight eligibility check** that spared Salesforce a redundant PO reservation call on every task assignment—cutting external latency (`~100–500 ms` per avoided call), failure surface, and Salesforce API governor-limit burn.

### Tie-In D: AI Tooling + NortonLifeLock + 4 Live Products
- **AI Tooling:** Domain agent skills + Node.js MCP server (PR diffs + Zipkin traces) + VS Code extension for Copilot Agent Mode + Ollama local assistant (domain-expert router → verified tables/charts) + Vitest/Playwright/feature-flag gates.
- **NortonLifeLock (2015–2020):** Frame as **hostile-DOM security engineering**—isolated worlds between background service worker (crypto/session), content scripts (untrusted third-party DOMs, Shadow DOM, synthetic React/Angular events), and React/TS popup/portal UIs under strict CSP.
- **Solo Products:** `wtool.dev` (DevFlow: Next.js 16 Cache Components, React 19 Compiler, client-side DAG privacy, Better Auth, Razorpay, Resend, Vercel AI SDK), `faucetworld.in` (autonomous since 2019: `node-cron`, MySQL, Redis, `ethers.js`, offerwalls, LLM support), `couponcode.dev` (since 2022), `uicraft.dev`.

---

## 4. Practice Questions & Answer Guides

### Q1 (Foundational — Screen / HM): *"Walk me through the hardest production bug you diagnosed at Comcast."*
- **What Interviewers Test:** Root-cause rigor vs. guessing; do you verify with data or throw RAM at containers?
- **Structured Answer Guide:**
  1. **Context (15s):** Production auto-assignment service on Sphere was hitting recurring OOM/memory spikes.
  2. **Diagnosis (30s):** Instead of bumping container memory, I correlated heap growth timing with DB query profiles and retry cadence. Found **3 compounding factors**: `SELECT *` pulling an unused `63 KB` JSONB column, `27,876` rows fetched in a single retry sweep, and a `10-minute` retry timer compounding allocation before GC settled.
  3. **Fix & Why It Worked (25s):** Switched to column-scoped queries. Because Postgres stores >2 KB JSONB out-of-line in TOAST, omitting the column skipped TOAST reads/decompression, cut `~1.75 GB` of wire transfer, and avoided `3×–10×` V8 `JSON.parse()` heap bloat.
  4. **Staff Close (20s):** Added/recommended bounded cursor batching + backoff/DLQ, and encoded "no `SELECT *` on JSONB tables" into our PR review checklist and domain agent skills.

### Q2 (Intermediate — Architecture Probe): *"In your Salesforce-to-Sphere migration, how did bi-directional or reverse CDC avoid infinite sync loops and partial-failure drift?"*
- **What Interviewers Test:** Real distributed-systems scars vs. buzzword name-dropping.
- **Structured Answer Guide:**
  1. **Strangler Fig Boundary:** Migrated domain slices incrementally (Job Details, Projects & Phases, Addressing, ROE/Site Contacts) while legacy Salesforce workflows still depended on read/write visibility.
  2. **Echo-Loop Prevention:** Tag every mutation with `origin_system` (`SPHERE` vs. `SFDC`) and a `correlation_id` / payload hash. CDC relays filter out or no-op changes whose origin matches the target system.
  3. **Governor Limits & Ordering:** Partition events by `job_id` so per-entity updates stay ordered; coalesce rapid bursts before calling Salesforce Bulk/REST APIs; enforce idempotency keys on both sides.
  4. **Drift Recovery:** Dead-letter queue for poison payloads + scheduled reconciliation job comparing entity timestamps/checksums between Sphere and Salesforce.

### Q3 (Deep-Dive — API & Domain Judgment): *"Why wasn't a client-side `AbortController` enough for the task-to-job refresh race? Why build a new task-based API?"*
- **What Interviewers Test:** Distinguishing local symptom patches from clean system boundaries.
- **Structured Answer Guide:**
  1. **Why Client-Only Fixes Leak:** Even with `AbortController` or TanStack Query cancellation, client-side resolution still required waterfall calls (`fetch task → extract jobId → fetch job`), duplicated resolution rules across entry points, and broke on deep-linked page refreshes before state hydrated.
  2. **Backend Boundary Fix:** Moving `task → job` resolution into a dedicated task-based API made the operation atomic in one round-trip, removed the client state machine entirely, and guaranteed consistent mapping across web, refresh, and future consumers.
  3. **Complementary Hygiene:** Still keep `AbortController`/TanStack Query cancellation on search/typeahead UIs, and pair backend mutations with centralized pure domain field validation + optimistic locking (`version`).

### Q4 (Deep-Dive — Skeptical Probe): *"You list 4 solo products, 6+ years at Comcast, and AI tooling. How do you know your solo products are actually engineered for reliability rather than weekend demos?"*
- **What Interviewers Test:** Operational maturity, security, and unit economics.
- **Structured Answer Guide:**
  - Anchor on **FaucetWorld (`faucetworld.in`, running autonomously since 2019)** and **DevFlow (`wtool.dev`)**:
    - *FaucetWorld:* Automated `node-cron` workers + `ethers.js` payouts require distributed cron locks (Redis), nonce serialization + tx confirmation tracking, HMAC/IP verification on offerwall callbacks, and Redis rate limiting against bot abuse—otherwise it bleeds real funds overnight.
    - *DevFlow:* Runs transforms client-side for zero-data-leakage privacy and zero server compute cost, while gating accounts (Better Auth), payments (Razorpay webhook idempotency), and Anthropic/Groq AI quotas via Drizzle/MySQL + Vitest/Playwright suites.

---

## 5. Quick-Review Cheat Sheet

- **Comcast Memory Spike:** `63 KB` JSONB (Postgres TOAST >2 KB) × `27,876` rows (`~1.75 GB` raw → `3×–10×` V8 heap) × `10-min` retry → fixed via column-scoped queries (+ batching/backoff/DLQ/skills).
- **16× Intake Speedup:** SQL execution plan profiling (`EXPLAIN ANALYZE`) + query rewrite (verify exact before/after ms + SQL shape).
- **Race & Stale Actions:** Task-based backend endpoint killed refresh race; single pure domain function stopped stale job action leaks.
- **Salesforce + Finance:** Strangler Fig (4 domains) + CDC back to SFDC (origin tagging stops echo loops) + pre-flight eligibility gate spared redundant PO reservation calls.
- **NortonLifeLock (2015–2020):** Cross-browser Password Manager extension (isolated worlds, hostile DOM autofill, CSP) + React/TS portals.
- **Solo Products:** `wtool.dev` (Next 16, React 19 Compiler, client-side privacy, Better Auth, Razorpay, Anthropic/Groq, Vitest/Playwright) + `faucetworld.in` (autonomous since 2019, Redis, MySQL, `node-cron`, `ethers.js`).
