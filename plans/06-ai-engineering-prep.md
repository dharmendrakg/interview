# 06 — AI Engineering, MCP, Agent Tooling & LLM Apps Prep

In 2026 interviews, **almost everyone claims "I use Copilot/Cursor." Almost nobody can explain how they built an MCP server feeding Zipkin traces to reviewing agents, authored domain skills, shipped a local Ollama generative-UI assistant, AND enforced deterministic verification gates.** This is your sharpest competitive edge—don't hand-wave it.

---

## 1. Model Context Protocol (MCP) & Agent Tooling Architecture

### Concepts You Must Explain Crisp & Deep
1. **What MCP solves:**
   - Standardizes how LLM hosts/IDEs (Copilot Agent Mode, Claude, OpenCode, etc.) discover and invoke **Tools** (executable actions), **Resources** (read-only context), and **Prompts** (reusable workflows) over stdio or Streamable HTTP/SSE using JSON-RPC 2.0—eliminating N×M custom integrations.
2. **Your Node.js MCP Server (PR Diffs + Zipkin Traces):**
   - **Why combine static PR diffs with runtime Zipkin traces?**
     - Static diff review spots syntax/local logic issues, *not* N+1 cascades, fan-out latency, or redundant external calls (like redundant Salesforce PO reservation calls). Pairing diff chunks with actual Zipkin span trees lets a reviewing agent spot latency regressions and architectural smells grounded in real production telemetry.
   - **Design Details Interviewers Probe:**
     - *Context window budgeting:* Raw PR diffs and Zipkin JSON spans can easily blow context limits! Explain how your MCP tools **filter, summarize, and paginate** (e.g., trim lockfiles/snapshots, collapse repetitive spans, return slowest critical-path spans + error tags first).
     - *Schema design:* Narrow, strongly-typed Zod/JSON-Schema tool parameters (`get_pr_diff({ prNumber, pathFilter })`, `get_trace_summary({ traceId, minDurationMs })`) rather than one generic "run arbitrary query" tool.
3. **VS Code Extension for GitHub Copilot Agent Mode (GitHub + Trace + DB tools):**
   - **Security & Blast Radius:**
     - DB tool: strictly **read-only credentials** (`SELECT` only or `EXPLAIN` only), statement timeout (e.g., 2s), row cap (`LIMIT 50`), no PII/secret tables exposed.
     - GitHub/Trace tools: scoped tokens, redaction of bearer tokens/credentials before passing text into LLM context.
     - **Prompt Injection Defense:** External content (PR comments, commit messages, DB string values, offerwall payloads) is **untrusted data**. Never execute state-changing tools automatically from untrusted text without explicit human confirmation ("human-in-the-loop" approval for mutations).

---

## 2. Domain Agent Skills & Verification Harnesses

### Defending Your "AI-First" Engineering Philosophy
Interviewers at senior/staff levels often worry: *"Does 'AI-first' mean shipping sloppy AI slop fast?"* Flip that concern immediately:
- **Context Engineering (Domain Skills):**
  - Generic agents fail on legacy/enterprise migrations because they don't know local invariants (e.g., Sphere's centralized job domain function, Salesforce CDC echo prevention, column-scoped DB projections).
  - Agent skills encode **architecture boundaries, anti-patterns, and verification commands** dynamically when relevant—keeping base prompts lean while preventing recurring mistakes.
- **Verification Pipeline ("AI code is an untrusted first draft"):**
  1. **Type & Static Gates:** Strict TypeScript (`noImplicitAny`, exhaustive unions), ESLint/React Compiler rules, GraphQL Codegen checks.
  2. **Automated Tests:** Vitest unit tests for pure domain logic + Playwright E2E for critical flows & race-condition regressions.
  3. **Structured Review Pass:** Agentic pre-review (using your MCP diff + trace tools) catches obvious smells *before* human review, then human reviewers focus on domain semantics, data modeling, and failure modes.
  4. **Safe Rollout:** Feature flags, canary/staged environments, Zipkin trace + error monitoring before 100% exposure.

---

## 3. Local LLMs (Ollama) + Domain Routing + Generative UI

*(Directly tied to Sphere's conversational LLM assistant for invoices, jobs, permits, finance)*

### Key Technical Trade-Offs to Articulate
1. **Why Local Models (Ollama) vs. Frontier Cloud APIs (Anthropic/OpenAI)?**
   - *Pros:* Sensitive enterprise financial/permit/customer data never leaves VPC/local infra; zero per-token variable bill; no external rate limits.
   - *Cons:* Smaller context windows, weaker multi-step tool calling, higher risk of malformed JSON/hallucinated fields on complex prompts, GPU memory/latency constraints.
2. **Why Domain-Expert Routing Solved Small-Model Limitations:**
   - Giving a local 7B–32B model 40 tools and 4 business domains degrades tool-selection accuracy and bloats the system prompt.
   - **Two-stage Router Pattern:**
     1. Lightweight **Intent Classifier / Router** classifies query into `{ invoices | jobs | permits | finance | out_of_scope }` (with confidence threshold + fallback clarification question).
     2. Selected **Domain Expert** receives *only* that domain's 3–5 typed tools, glossary, and schema examples.
3. **Generative UI (Rendering Tables & Charts Safely):**
   - **Golden Rule:** *Never let the LLM fabricate numbers or SQL directly in the UI.*
   - **Flow:** LLM emits a structured tool call / filter spec (validated with **Zod**) → Backend executes a parameterized query/API → Backend returns **verified row data + metadata** → Frontend renders deterministic React/MUI `<DataTable />` or `<Chart />` components from typed UI descriptors (`{ type: 'table', columns, rows }` or `{ type: 'bar_chart', series }`), streaming tokens/status via Vercel AI SDK or SSE.
   - **Failure Fallback:** If Zod schema validation or JSON parsing fails, retry once with error feedback or fall back cleanly to plain text / guided prompt suggestions.

---

## 4. Vercel AI SDK (DevFlow `wtool.dev`: Anthropic + Groq)

- **Multi-Provider Routing:** Explain *why* using both Anthropic and Groq makes sense:
  - **Groq** for ultra-low-latency streaming (<300ms TTFT) on quick syntax fixes / JSON repairs / interactive UX.
  - **Anthropic (Claude)** for deeper multi-step reasoning and nuanced explanations.
- **Streaming, Prompt Caching & Unit Economics:** `streamText` / `streamObject` in Next.js Route Handlers, prompt caching for static system/skill prefixes (cutting TTFT & input token cost), max token caps, abort handling (`req.signal`), per-tier usage metering in Drizzle/MySQL + Redis rate limits, and comparing cloud token economics vs. local Ollama GPU/latency sizing.
- **Human-in-the-Loop Write Guardrails:** Explicit confirmation state machines for any mutating agent tool, plus practicing **live AI-assisted coding interviews** (steering an agent cleanly, rejecting subtle bugs, and verifying edge cases under a 45-min clock).

---

## 5. Evaluation (Evals) & AI Probe Questions

Expect this Staff-level question: **"How do you know your Ollama router or DevFlow auto-fixer actually improved after changing a prompt or model?"**
- **Offline Eval Suite:** Curate 50–100 golden test cases per domain (invoices, jobs, permits, finance + adversarial/ambiguous queries). Measure:
  1. *Router accuracy* (exact domain match).
  2. *Schema/tool-call validity rate* (% passing Zod without retry).
  3. *Groundedness / faithfulness* (answers match deterministic DB tool outputs).
  4. *Latency (p50 / p95 TTFT & total completion).*
- **Online Telemetry:** User thumbs-up/down, retry rate, fallback-to-human rate, and trace spans in Zipkin/logs.
