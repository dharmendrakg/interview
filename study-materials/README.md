# Tailored Interview Study Guide — Dharmendra Kumar Gupta

**Target Calibration:** Senior / Staff Full-Stack Engineer (Frontend-Leaning, L5/L6) · Secondary Tracks: Staff Frontend Architect & AI-Native Product Engineer  
**Source Material Analyzed:** [`resume.md`](../resume.md) and [`plans/01`–`09`](../plans/README.md)

---

## Assumptions & Calibration Notes

1. **Target Role Assumption:** Neither `resume.md` nor `plans/` pins a single company loop. Based on your 11+ years of experience (5 years at NortonLifeLock, 6+ years at Comcast as DE4 / Sphere UI Lead & go-to reviewer, and 4 live solo products), this curriculum calibrates to **Senior / Staff (L5–L6) Full-Stack and Frontend-Leaning product/AI engineering loops** at top product companies and remote engineering orgs.
2. **Depth Calibration:** Every chapter skips beginner syntax and jumps straight to **internals, trade-offs, failure modes, and whiteboard-ready mental models** (e.g., PostgreSQL TOAST & `EXPLAIN (ANALYZE, BUFFERS)`, React Compiler dataflow constraints, Next.js 16 `"use cache"` + PPR, V8 heap amplification, MCP context budgeting, and L5→L6 behavioral framing).
3. **Relationship to `plans/`:** Your `plans/` folder defines *what* to prepare and your weekly schedule. This `study-materials/` folder provides the **self-contained technical textbook, gap-closing deep-dives, resume bridges, and full Q&A answer guides** for every theme.

---

## Gap Analysis: Resume vs. Preparation Plan

| Gap / Risk Area | Evidence in `resume.md` & `plans/` | How This Study Guide Closes It |
| :--- | :--- | :--- |
| **1. Unverified Claim Details (`[VERIFY]` Traps)** | `plans/02` flags missing crisp anchors: exact `16×` SQL rewrite & baseline latency, CDC mechanism, MUI v7 adoption scale, Find My Power ML boundary, Norton concrete bug, solo product metrics. | **Chapter 01** turns every resume bullet into a defensible 90-sec + 10-min technical script with an explicit pre-interview `[VERIFY]` resolution worksheet. |
| **2. Stack Breadth vs. Internals Depth** | `resume.md` spans React 19 Compiler, Next.js 14–16 Cache Components, Drizzle, Postgres, MySQL, MongoDB, Redis, GraphQL Yoga, Docker, `ethers.js`, and Ollama. | **Chapters 02, 03 & 09** dissect the exact under-the-hood mechanics interviewers test to separate API users from Staff engineers. |
| **3. Senior → Staff System Design Framing** | Strong execution history at Comcast needs structured 45-min whiteboard framing: capacity math, failure modes, Outbox/CDC echo-loops, RADIO frontend architecture. | **Chapter 04** provides complete 45-min blueprints for all 6 domain-matched designs + 2 unseen classics (Webhooks, Distributed Rate Limiter). |
| **4. Defending "AI-First" With Rigor** | Interviewers will probe hallucinations on local Ollama models, MCP blast radius, prompt injection, and how you evaluate prompt/model changes. | **Chapter 05** covers MCP JSON-RPC/context budgeting, least-privilege security, two-stage domain routing + Zod Generative UI, and offline/online eval suites. |
| **5. Timed Coding & Senior PR Audit Speed** | 11+ yr engineers often fail 45-min blind DSA (Graphs/Topo Sort, Sliding Window, LRU, Trees) or live async JS/React widgets without deliberate drills. | **Chapters 06 & 07** provide TS pattern templates, worked solutions (including DAG topological runner & `AbortController` hooks), and Senior PR bug-hunt drills. |
| **6. Staff Behavioral Calibration & Missing Stories** | `plans/08` leaves two explicit `[FILL IN]` gaps (disagreement/pushback and biggest mistake/failure) and highlights L5 ("I fixed X") vs. L6 ("I codified guardrails") drift. | **Chapter 08** maps L5→L6 upgrades across your story bank and gives structured templates to lock in the missing conflict and failure stories. |

---

## Curriculum Overview

| Chapter # | Title | Focus Area | Study Priority | File |
| :---: | :--- | :--- | :---: | :--- |
| **01** | Resume Forensics & Probe Defense | `[VERIFY]` closure, Comcast Sphere metrics, long-tenure/side-product probes, NortonLifeLock & solo products | **High** | [`01-resume-forensics-and-story-defense.md`](./01-resume-forensics-and-story-defense.md) |
| **02** | Backend Internals, SQL/TOAST & Production Forensics | Node.js event loop & V8 heap, Postgres `EXPLAIN` drill/TOAST/`SKIP LOCKED`, REST idempotency, observability & migrations | **High** | [`02-backend-databases-and-production-forensics.md`](./02-backend-databases-and-production-forensics.md) |
| **03** | React 19, Next.js 14–16 & Design Systems | React Compiler, RSC/`"use cache"`, hydration, TS type-level mastery, MUI v7 vs. Tailwind/shadcn, Web Vitals & Vitest/Playwright | **High** | [`03-frontend-react19-nextjs16-and-design-systems.md`](./03-frontend-react19-nextjs16-and-design-systems.md) |
| **04** | System Design (Backend & RADIO Frontend) | 45-min blueprints, worked capacity math, CDC/Outbox, Auto-Assignment, Data Grid, Webhooks, Rate Limiter & Enterprise AI/RAG | **High** | [`04-system-design-backend-and-frontend.md`](./04-system-design-backend-and-frontend.md) |
| **05** | AI Engineering, MCP, Local LLMs & Evals | MCP + Zipkin, context engineering (KV-cache/tool tax), constrained outputs, Ollama routing, Generative UI & LLM-as-Judge | **High** | [`05-ai-engineering-mcp-and-generative-ui.md`](./05-ai-engineering-mcp-and-generative-ui.md) |
| **06** | DSA Patterns & TypeScript Execution | Patterns + TS templates (Topo Sort, Sliding Window, Binary Search, `MinHeap`, `UnionFind`, LRU) & multi-part stateful coding | **High** | [`06-dsa-patterns-and-ts-execution.md`](./06-dsa-patterns-and-ts-execution.md) |
| **07** | Practical Machine Coding & Senior PR Audits | Async utilities, `useQueryLite`, React machine coding, live tests & 10-point AI-slop PR bug-hunts | **Medium** | [`07-practical-machine-coding-and-pr-audits.md`](./07-practical-machine-coding-and-pr-audits.md) |
| **08** | Behavioral Leadership & Staff Calibration | 60s/2m pitches, L5→L6 reframing, conflict & failure slots, 2026 AI-native leadership probes & scoring rubric | **Medium** | [`08-behavioral-staff-calibration-and-mock-rubric.md`](./08-behavioral-staff-calibration-and-mock-rubric.md) |
| **09** | Specialized Stack & Domain Deep-Dives | `ethers.js` payout reliability, MongoDB ESR & CV pipeline, MV3 extension security, serverless pooling & Docker/feature flags | **Low** | [`09-specialized-stack-and-domain-deep-dives.md`](./09-specialized-stack-and-domain-deep-dives.md) |

---

## How to Use This Study Guide Alongside `plans/`

1. **Before Week 1 Starts (2–3 hours):** Complete **Chapter 01** and resolve every `[VERIFY]` prompt on paper. Never walk into a screen with fuzzy numbers on your own resume.
2. **Follow `plans/01-study-roadmap.md`:** Use Chapters **02–06** as your core technical reading during Track A (4-week) or Track B (8-week) Block 2 study slots.
3. **Daily Coding (Block 1):** Pair the problem list in `plans/07-dsa-and-coding-drills.md` with the TypeScript templates in **Chapter 06** and the practical utilities/widgets in **Chapter 07**.
4. **Weekly Calibration:** Use **Chapter 08** during mock loops to audit whether your answers sound L5 (executor) or L6 (force-multiplier). Study **Chapter 09** selectively when targeting Web3/fintech, browser-security, or ML-adjacent product roles.
