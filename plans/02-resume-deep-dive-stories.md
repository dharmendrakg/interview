# 02 — Resume Deep-Dive Stories & Probe Defense

Every line on your resume is an invitation for an interviewer to spend 15 minutes drilling into it. Below are **2-minute crisp scripts**, **architectural trade-offs**, **follow-up traps**, and `[VERIFY]` prompts for facts you should confirm cleanly in your head before interviews.

---

## Story 1: Production Auto-Assignment Memory Spikes (Comcast Sphere)

> **Resume claim:** Traced recurring memory spikes in a production auto-assignment service to three compounding causes (a JSONB column of up to 63 KB over-fetched on every retry, payload bloat across 27,876 rows, a retry loop reloading it every 10 minutes) and eliminated them with column-scoped queries.

### 90-Second Pitch
- **Situation:** A production auto-assignment background service kept hitting recurring OOM / memory spikes in production.
- **Investigation:** Instead of blindly increasing container memory limits, I analyzed heap/memory behavior alongside the DB queries executed during retries and distributed traces.
- **Root Causes (3 compounding factors):**
  1. The service was running broad `SELECT *` / entity-level fetches that included a `JSONB` payload column up to **63 KB** per row, even though the assignment/retry logic only needed a few scalar IDs/status fields.
  2. The retry batch was sweeping **27,876 rows** at once (`~63 KB × 27k` + JS object/JSON parse amplification in V8 heap easily balloons into hundreds of MBs/GBs per cycle).
  3. A **10-minute retry loop** kept re-loading the full dataset repeatedly before GC / heap pressure stabilized, compounding memory spikes.
- **Fix & Impact:** Replaced broad fetches with **column-scoped queries** (projecting only required columns), eliminating the spike without hardware scaling.

### Deep-Dive Follow-Up Traps (Be Ready For These!)
1. **"Why did 27,876 × 63 KB blow up a Node.js container even more than raw byte size?"**
   - Raw UTF-8 JSON bytes in transit + Postgres wire buffer + `JSON.parse()` creating V8 heap objects (strings, hidden classes, nested arrays/maps, GC old-space promotion) typically adds **3×–10× memory amplification** over raw JSON size.
2. **"Why didn't you just paginate or stream, or fix the retry condition?"**
   - Explain defense-in-depth: (a) Column projection immediately stopped transferring/parsing unneeded TOASTed JSONB data; (b) `[VERIFY]` Did you also add batching/pagination, backoff, or dead-lettering for poisoned rows so 27,876 rows aren't retried identically every 10 minutes? Be honest about what shipped vs. what you'd add next (bounded cursor pagination, exponential backoff + max retry cap/DLQ, separating large blobs into a cold table).
3. **"How does PostgreSQL store a 63 KB JSONB column?"**
   - Explain **TOAST** (The Oversized-Attribute Storage Technique, threshold ~2 KB): Postgres compresses/moves large JSONB values out-of-line to a TOAST table. If you don't select that column, Postgres **never reads the TOAST table**—huge I/O + network + heap savings!

---

## Story 2: 16× Intake Listing API Speedup (Comcast Sphere)

> **Resume claim:** Made the intake listing API 16× faster by rewriting its query.

### 90-Second Pitch
- **Situation:** The intake listing endpoint—a critical entry screen for Sphere users—was sluggish as data grew.
- **Action:** Profiled the SQL execution plan, identified the bottleneck in how joins/filters/aggregations/subqueries were structured, and rewrote the query.
- **Result:** **16× latency reduction** on the listing API.

### `[VERIFY]` Before Interview
- Recall the **exact baseline & final latency** (e.g., `~3.2s -> ~200ms` or `4.8s -> 300ms`) and **exact SQL change** (e.g., correlated subquery -> CTE/lateral/window function, `OFFSET` -> keyset, `OR` preventing index scan -> `UNION ALL`, or avoiding redundant joins before `LIMIT`).
- **Follow-up trap:** *"Did you change indexes, ORM query generation, or pagination too? How did you verify identical results?"* (Explain comparing row counts/checksums against prod-like data and checking `EXPLAIN (ANALYZE, BUFFERS)`).

---

## Story 3: Race Condition & Stale Actions Elimination (Comcast Sphere)

> **Resume claim:** Removed a page-refresh race condition by resolving task-to-job mapping in a task-based API instead of in the client; stopped stale job actions from leaking through job updates by centralizing field handling in one domain function.

### Why Interviewers Love This Story
It shows **API/domain boundary judgment**, not just UI coding.
- **Part A (Race condition):** Resolving task→job mapping client-side caused multiple async requests / stale closures / out-of-order responses on refresh or fast navigation. Moving the resolution to a single **task-based backend endpoint** eliminated split client state, waterfall calls, and race windows (also mention `AbortController` / TanStack Query cancellation as complementary client hygiene).
- **Part B (Stale job actions):** Multiple update paths were independently mutating/forwarding job fields, allowing stale actions to overwrite newer state. Centralizing field mapping/validation into **one pure domain function** created a single source of truth for allowed transitions and field sanitization.
- **Follow-up trap:** *"What if two users update the same job concurrently on the backend?"* -> Discuss **optimistic concurrency control** (`version` / `updated_at` check in `WHERE` returning `409 Conflict`) and state-machine guards.

---

## Story 4: Salesforce Migration, CDC & Shared Finance APIs (Comcast Sphere)

> **Resume claim:** Migrated Job Details, Projects & Phases, Addressing, and Right of Entry/Site Contacts off Salesforce into Sphere, with change-data-capture replication back to Salesforce. Designed finance APIs shared by Salesforce and Sphere (labor rates, funding approvals, purchase-order reservations), including an eligibility check that spares Salesforce a redundant reservation call on every task assignment.

### Talking Points
1. **Strangler Fig Migration Pattern:** Why migrate module-by-module (Job Details, Projects & Phases, Addressing, ROE/Site Contacts) instead of big-bang cutover.
2. **CDC Replication Back to Salesforce:** Why bi-directional or reverse sync is hard—infinite sync loops (use origin/actor metadata or idempotency keys), Salesforce API governor limits & bulkification, schema/field mismatch, out-of-order events, and reconciliation jobs.
3. **Finance Eligibility Check:** Previously, every task assignment triggered a PO reservation call to Salesforce/finance even when ineligible. Putting a lightweight **pre-flight eligibility check** in the shared API eliminated redundant external calls, cutting latency, failure surface, and Salesforce API quota usage.
- **`[VERIFY]`:** Know roughly what CDC mechanism was used (e.g., Postgres logical decoding / Debezium / Kafka / custom outbox table -> Salesforce REST/Bulk/Platform Events) and how partial failures were retried.

---

## Story 5: UI Leadership, Design System (MUI v7) & Dashboards (Comcast Sphere)

> **Resume claim:** Lead UI development for Sphere and own its design system (React, TypeScript, MUI v7). Go-to reviewer for UI and backend changes across Sphere's services. Built network port-connectivity dashboard (Next.js 15, React 19, GraphQL Code Generator, SSO, Zustand, live charts, map), Ethernet-over-HFC reliability dashboard, AI hiring prototype, Next.js 14 PoC, Find My Power.

### Talking Points
- **Design System Governance:** How you avoid MUI v7 upgrade pain and component sprawl across multiple Sphere modules (theme tokens, wrapped primitives vs. leaky `sx` overrides, Storybook documentation, breaking-change codemods/PR checks, accessibility defaults).
- **Port-Connectivity Dashboard:** Why `GraphQL Code Generator` (end-to-end typed queries/fragments catching schema drift at build time), why `Zustand` (ephemeral UI/map selection & filter state distinct from server cache), how live charts + maps stayed smooth (memoization, throttling updates, clustering/viewport filtering on maps).
- **Code Review Bar:** What makes you the "go-to reviewer" for both UI and backend? Mention reviewing for **failure modes, query plans, concurrency/races, domain boundaries, and maintainability**, not nitpicking formatting.
- **`[VERIFY]`:** Confirm approximate adoption scale of Sphere's MUI v7 design system (how many engineers/squads/modules consume it) and what concrete problem it solved (e.g., inconsistent UI, slow onboarding, duplicated table/form logic).

### Sub-Story 5B: Find My Power (Street View + Computer Vision)
> **Resume claim:** Built Find My Power (React, Material UI, Node.js, MongoDB), which identifies power supplies in Google Street View imagery with a machine-learning model to cut manual field checks.

- **Clarify Your Exact Boundary:** Be crisp on what *you* owned vs. the ML team/model (e.g., full-stack app, Street View image ingestion/geocoding pipeline, async inference job orchestration, MongoDB storage, and React/MUI verification UI).
- **Hard Problems to Highlight:**
  1. **Async Inference & API Quotas:** Fetching Street View tiles/panoramas at scale, batching/throttling external Google Maps requests, decoupling slow ML inference from the UI via job status tracking.
  2. **Why MongoDB Here:** Unstructured/variable detection payloads (bounding boxes, confidence scores, camera headings/pitch, Street View metadata) fit document storage naturally compared to rigid relational tables.
  3. **Human-in-the-Loop UX:** Showing confidence thresholds, bounding-box overlays, and side-by-side map/Street View review so field engineers confirm detections before dispatching trucks.
- **`[VERIFY]`:** Know roughly which ML model/API ran inference (custom TensorFlow/PyTorch/SageMaker/YOLO or external Vision API) and how false positives/negatives were handled.

---

## Story 6: AI-Assisted Engineering Tooling (MCP + Zipkin + Copilot + Skills)

> **Resume claim:** Domain agent skills; Node.js MCP server feeding PR diffs and distributed traces (Zipkin) to reviewing agents; VS Code extension exposing GitHub, trace and database tools to GitHub Copilot Agent Mode; verification pipeline (tests, structured review, feature flags, staged rollout).

### Talking Points (Your Biggest Differentiator)
- **Why generic AI fails in enterprise codebases:** Agents hallucinate patterns when they lack architectural invariants (e.g., Sphere's Salesforce CDC rules, centralized domain functions, column-scoped query norms).
- **Solution Stack:**
  1. **Domain Skills:** Codified Sphere's migration playbook, architecture, and known pitfalls into structured instructions loaded by coding agents.
  2. **MCP Server (Node.js):** Exposes read-focused tools (`get_pr_diff`, `get_zipkin_trace`, etc.) via Model Context Protocol so reviewing agents inspect real runtime latency/span breakdowns alongside static code diffs.
  3. **VS Code Extension for Copilot Agent Mode:** Gives engineers safe, scoped GitHub, Zipkin trace, and DB inspection tools right inside the IDE.
  4. **Verification Gate:** AI output = untrusted first draft -> Vitest unit tests + Playwright E2E + structured review pass + feature flags/staged rollouts.
- **Follow-up trap:** *"Isn't giving an AI DB/GitHub tools dangerous?"* -> Discuss **least privilege** (read-only DB roles, parameterized/allow-listed queries, PII redaction, no direct prod writes, human approval before PR merge).

---

## Story 7: LLM Assistant (Ollama Local Models)

> **Resume claim:** Built conversational front-end of an LLM assistant for Sphere (Next.js 16, React 19, TypeScript, Ollama): domain-expert routing for invoices, jobs, permits and finance, with answers rendered as tables and charts on local models.

### Talking Points
- **Why Local (Ollama):** Data privacy/compliance (internal enterprise finance, invoices, permits can't leak to public APIs) and predictable cost.
- **Domain-Expert Routing:** Why one giant prompt fails on smaller local models -> intent classifier / router dispatches to specialized domain prompts + narrow tool schemas (invoices vs. jobs vs. permits vs. finance).
- **Generative UI (Tables & Charts):** How you stop local models from breaking JSON/UI rendering -> strict JSON schema / structured outputs, Zod validation on the server/client boundary, fallback to markdown/error card if validation fails, and deterministic data fetching (LLM picks tool/params, *backend executes query*, UI renders typed table/chart from verified rows—never let the LLM invent financial numbers!).

---

## Story 8: Solo Live Products (DevFlow, FaucetWorld, CouponCode, UICraft)

### DevFlow (`wtool.dev`)
- **Architecture:** Client-side execution for developer tool pipelines (privacy + zero compute cost for transforms) + Drizzle/MySQL for user accounts/saved flows + Better Auth + Razorpay + Resend + Vercel AI SDK (Anthropic & Groq for AI explanations/auto-fixes) + Vitest/Playwright.
- **Technical hook:** Next.js 16 Cache Components + React 19 React Compiler. Know how client-side DAG/pipeline state executes cleanly and how Groq (fast inference) vs. Anthropic (complex reasoning/fixes) can be routed.

### FaucetWorld (`faucetworld.in`, since 2019 — 7+ years running!)
- **Architecture:** Next.js 16 frontend + Express worker/API service (`node-cron`), MySQL + Redis, `ethers.js` blockchain payouts, third-party offerwalls/affiliate system, LLM chatbot/trivia/support.
- **Technical hook:** Running autonomously since 2019! Expect questions on **anti-abuse/fraud prevention** (Redis rate limiting, IP/device checks, offerwall callback signature verification), **payout reliability** (`ethers.js` nonce management, gas spikes, idempotent ledger entries so a cron restart never double-pays), and **cron locking** (Redis distributed lock so two instances don't run the same job).

### CouponCode (`couponcode.dev`, since 2022) & UICraft (`uicraft.dev`)
- **CouponCode:** Automated ingestion + deduplication of discounted courses, scheduled social/site publishing via crons, SEO/SSR caching for fast organic landing pages, handling expired/invalid coupons cleanly.
- **UICraft:** Packaging reusable Next.js/Tailwind/React templates & components—clean theming tokens, zero-config DX, responsive/a11y defaults, and digital license/delivery flow.
- **`[VERIFY]`:** Pick 1–2 concrete operational stats you're comfortable sharing (e.g., FaucetWorld uptime/payouts/active users, DevFlow number of tools/tests) so these read like real businesses, not weekend demos.

---

## Story 9: NortonLifeLock / Symantec (2015–2020)

> **Resume claim:** Norton Password Manager cross-browser extension, React/TS portal pages, Norton Identity Safe.

### 90-Second Pitch & Why These 5 Years Matter
Don't let 2015–2020 sound like "ancient history" or generic maintenance—consumer security extensions operate in **one of the harshest frontend environments on the web**:
- **Cross-Browser Extension Architecture:** Separate security boundaries between **Background / Service Worker** (holds session/crypto state), **Content Scripts** (isolated DOM access on untrusted third-party sites), and **Popup / Portal UI** (React + TypeScript), communicating via structured message passing.
- **Hostile DOM Autofill Heuristics:** Third-party login/checkout forms use Shadow DOM, SPAs with synthetic React/Angular input events, dynamic iframes, and anti-bot obfuscation—autofill has to work without breaking site layouts or leaking credentials to malicious page scripts.
- **Security & Scale Mindset:** Working on a flagship consumer product shipped to millions of users meant strict CSP, origin checks, zero tolerance for XSS/credential leaks, and rigorous cross-browser compatibility (Chrome, Firefox, Edge, Safari/IE transitions).
- **`[VERIFY]` Before Interview:** Recall **one specific feature or hard bug** you owned at Norton (e.g., a React+TS web portal workflow, an autofill/form-capture bug on modern SPAs, or cross-browser messaging/performance fix) so you have a crisp concrete story if asked.
