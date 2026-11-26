# 01 — Study Roadmap (4-Week Sprint & 8-Week Deep Track)

Choose the **4-Week Sprint** if you already have interviews scheduled or are actively applying right now. Choose the **8-Week Deep Track** if you are targeting Staff/L6 roles at top product companies and want maximum DSA + System Design polish.

---

## Target Readiness Bar (Exit Criteria)

Before calling yourself interview-ready, pass all 6 gates:
1. **Pitch & Resume Fluency:** Can explain any bullet on `resume.md` in 90 seconds (impact + trade-offs) and survive 10 minutes of hostile "why didn't you do X instead?" follow-ups.
2. **DSA Baseline:** Solve 2 unseen LeetCode Mediums in 40 minutes in TypeScript/JavaScript with clean edge-case handling and complexity analysis—no IDE autocomplete dependence.
3. **Practical Frontend/Full-Stack Coding:** Build a debounced/abortable search UI, an interactive data grid/filter, or an async concurrency queue in 35–45 minutes without stalling.
4. **System Design (Backend + Frontend):** Drive a 45-minute whiteboard session covering requirements, scale math, APIs, data model, failure modes, and UI architecture without needing interviewer nudges.
5. **Database & Debugging Mastery:** Read a Postgres/MySQL `EXPLAIN ANALYZE` plan, spot JSONB/N+1/index issues, and explain Node.js heap/event-loop debugging step by step.
6. **AI Engineering Credibility:** Defend MCP tool boundaries, local vs. cloud LLM trade-offs, structured outputs, hallucination mitigation, and AI code verification gates.

---

## Track A: 4-Week Intensive Sprint (2.5–3 hrs/day weekdays, 4–5 hrs/day weekends)

### Daily Weekday Template (2.5–3 hrs)
- **Block 1 (60 min):** 1–2 coding problems from `07-dsa-and-coding-drills.md` (alternating DSA & JS/React/Node machine coding).
- **Block 2 (60–75 min):** Deep technical topic of the day (`03`, `04`, `05`, or `06`).
- **Block 3 (30–45 min):** Active recall — speak 1 resume story or 1 behavioral question aloud, or review notes without looking.

### Week 1 — Resume Narrative, Core JS/TS/React/Next.js & Coding Restart
- **Mon:** Resume deep-dive (`02`) — audit Comcast metrics & Salesforce migration stories. Coding: Arrays/HashMaps/Sliding Window (3 problems).
- **Tue:** React 19 internals, concurrent rendering, Compiler rules (`03`). Coding: Implement `debounce` / `throttle` + `Promise.all` with concurrency limit.
- **Wed:** Next.js 14–16 App Router, RSC boundaries, caching (`use cache` / Cache Components), Server Actions (`03`). Coding: Two Pointers + Stack (3 problems).
- **Thu:** TypeScript generics, narrowing, type-safe API contracts + MUI v7 / Design System + Browser/Web Security (`03`). Coding: React live widget (typeahead with `AbortController` + keyboard nav).
- **Fri:** Resume deep-dive (`02`) — DevFlow, FaucetWorld, CouponCode/UICraft, Find My Power, NortonLifeLock stories. Coding: Trees/BFS/DFS (3 problems).
- **Sat (Mock + Review):** 1 timed Frontend Machine Coding (45m) + 1 Backend/SQL review (`04` Postgres JSONB & indexing).
- **Sun:** Rest + 60m flashcard/recall review of Week 1 weak spots.

### Week 2 — Backend, Databases, Performance Forensics & System Design Foundations
- **Mon:** Node.js event loop, streams, memory leaks & heap snapshots (`04`). Coding: Graphs / Topological Sort (2–3 problems).
- **Tue:** PostgreSQL & MySQL query optimization (`EXPLAIN ANALYZE`, composite indexes, Drizzle pitfalls, N+1) (`04`). Coding: Async task scheduler / retry with backoff + jitter.
- **Wed:** Redis caching, rate limiting, distributed locks, Auth (SSO, NextAuth vs Better Auth), Webhook idempotency, MongoDB, Zero-downtime DB migrations & Docker/CI/CD (`04`). Coding: Heap / Priority Queue (2–3 problems).
- **Thu:** System Design Framework + Practice Prompt #1 (Salesforce-to-Sphere CDC migration & shared finance APIs) (`05`). Coding: Binary Search on answer (2 problems).
- **Fri:** Frontend System Design Framework + Practice Prompt #3 (Enterprise Data Grid + Design System at scale) (`05`). Coding: LRU Cache / Rate Limiter in TS.
- **Sat (Mock + Review):** Full 45-min System Design mock (Prompt #2: Auto-assignment / retry queue without memory bloat) + behavioral stories (`08`).
- **Sun:** Rest + refine weak areas from Saturday mock.

### Week 3 — AI Engineering, Advanced System Design & Behavioral Polish
- **Mon:** AI Engineering (`06`) — MCP architecture, Zipkin/GitHub tool boundaries, security/prompt injection, Copilot Agent Mode extension story. Coding: Intervals & Greedy (2–3 problems).
- **Tue:** AI Engineering (`06`) — Ollama local routing, Generative UI (tables/charts), Vercel AI SDK, evals & AI code verification gates. Coding: React custom hook (`useQuery`-lite with cache & dedupe).
- **Wed:** System Design Practice Prompt #4 (Real-time Network Telemetry Dashboard) & #5 (DevFlow client-side DAG pipeline) (`05`). Coding: DP basics (2–3 problems).
- **Thu:** Behavioral & Leadership (`08`) — 8-story STAR matrix, Senior vs. Staff framing, code review conflict & cross-team pushback stories. Coding: Trie / String matching (2 problems).
- **Fri:** System Design Practice Prompt #6 (FaucetWorld cron/blockchain payout) + Prompts #7/#8 (Webhook Delivery & Distributed Rate Limiter) (`05`). Coding: Mixed blind medium (2 problems, 45m timed).
- **Sat (Full Loop Simulation):** 1 DSA + 1 System Design + 1 Behavioral back-to-back with strict timer.
- **Sun:** Post-mortem of Saturday simulation; rewrite weak answers.

### Week 4 — Company-Specific Targeting, Mock Loops & Final Sharpening
- **Mon–Tue:** Re-solve the 12 hardest coding problems you struggled with; review all Postgres/React/Node "gotcha" questions.
- **Wed:** Mock Frontend/Full-Stack Machine Coding (45m) + PR Code Review / Bug-Hunt Drill (`07` Part C) + Mock AI/Architecture deep-dive (45m).
- **Thu:** Polish "Tell me about yourself" (60s & 2m), salary/level calibration, and 8–10 smart questions for interviewers (`08`).
- **Fri:** Light review only — read [`09-interview-cheat-sheet.md`](./09-interview-cheat-sheet.md) (metrics, latency numbers, architecture hooks, and STAR reminders).
- **Sat–Sun:** Rest, sleep, light warm-up before interview week.

---

## Track B: 8-Week Thorough Plan (1.5–2 hrs/day)

Use the same module order as Track A, stretching each Track A week across **2 weeks**:
- **Weeks 1–2:** Resume Deep-Dive + DSA Foundations (Arrays, Hashing, Trees, Graphs) + React/Next.js Internals.
- **Weeks 3–4:** Practical JS/TS/React Machine Coding + Node.js/SQL/Redis/Auth Deep-Dive.
- **Weeks 5–6:** System Design (Backend + Frontend, 6 full whiteboards) + AI Engineering Deep-Dive.
- **Weeks 7–8:** Hard DSA Patterns (DP, Tries, Heaps) + Behavioral/Staff Calibration + 4 Full Mock Loops.

---

## Weekly Progress Tracker

Copy this checklist to track completion:

- [ ] **Week 1:** All 10+ resume stories rehearsed aloud; 15 coding problems; React 19 & Next.js 16 notes locked.
- [ ] **Week 2:** Postgres `EXPLAIN` + Node memory/event loop mastered; 3 system designs completed; 15 more coding problems.
- [ ] **Week 3:** AI engineering + MCP/Ollama defenses solid; 3 more system designs; full behavioral matrix complete.
- [ ] **Week 4:** 2+ timed full mock loops completed; weak-spot re-drills done; interview cheat sheet finalized.
