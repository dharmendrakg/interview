# Dharmendra Kumar Gupta

**Staff Full-Stack Engineer (Frontend-Leaning · Individual Contributor) · React · TypeScript · Next.js · Node.js · AI Engineering**

Bengaluru, India (open to remote) · [+91 7200939340](tel:+917200939340) · [dhar.kg@gmail.com](mailto:dhar.kg@gmail.com) · [LinkedIn](https://www.linkedin.com/in/dharmendra-kumar-gupta) · [GitHub](https://github.com/dharmendrakg)

---

## Summary

Hands-on Staff-level Individual Contributor with 11+ years of experience driving frontend architecture, design systems, and distributed backend reliability. For 6+ years at Comcast, served as **UI Lead (IC)** and cross-service reviewer for **Sphere** (replacing legacy Salesforce across **[N squads / M engineers]**), driving four domain migrations, shared finance APIs, and production forensics (`16×` API speedup; 3-cause OOM elimination). Built AI-assisted engineering workflows—shipping an MCP trace-review server, VS Code agent tooling, and a privacy-first local LLM assistant gated by tests and staged rollouts—while building and operating automated live web products end to end.

---

## AI-Assisted Engineering

- **Domain agent skills:** Authored codebase skills encoding Sphere's architecture invariants, Salesforce migration playbook, and query/state pitfalls so coding agents produce compliant changes rather than generic boilerplate.
- **MCP & IDE review tooling:** Built a Node.js Model Context Protocol (MCP) server feeding pull-request diffs and distributed traces (Zipkin) to reviewing agents, plus a VS Code extension exposing scoped GitHub, trace, and read-only database tools to GitHub Copilot Agent Mode—catching **[latency/query regressions before merge]**.
- **Deterministic verification gates:** Treats AI-written code as an untrusted first draft, gating every change through strict TypeScript, Vitest unit tests, Playwright E2E suites, structured trace-aware review, and feature-flagged staged rollouts.

---

## Skills

- **Architecture & Leadership:** Frontend/Full-Stack System Design, Design Systems & Governance, Accessibility (WCAG), Cross-Service Code Review, Strangler-Fig & Zero-Downtime Migrations, Observability (Zipkin Distributed Tracing)
- **Frontend:** TypeScript, JavaScript, React 19 (React Compiler), Next.js (App Router v14–v16, Cache Components), Material UI (v5–v7), Tailwind CSS, shadcn/ui, MUI X Data Grid, TanStack Query, Zustand, GraphQL Code Generator, React Hook Form, Storybook
- **Backend & Data:** Node.js, Express, Next.js API Routes & Server Components, REST, GraphQL (Yoga), PostgreSQL (Query Tuning, JSONB/TOAST), MySQL/MariaDB, MongoDB, Drizzle ORM, Redis, CDC Replication, Background Jobs & Rate Limiting, SSO / OAuth2 / OIDC, NextAuth, Better Auth
- **AI Engineering:** Model Context Protocol (MCP), Agent Skills & Plugins, Vercel AI SDK, Ollama (Local LLMs), Structured Outputs (Zod) & Evals, Prompt Injection Guardrails
- **Quality & Delivery:** Vitest, Playwright, Docker, CI/CD, Feature Flags, Staged Rollouts

---

## Experience

### Comcast — Senior Software Engineer (Development Engineer 4)
*Bengaluru, India · Jun 2020 – Present*

*Sphere: Comcast's core platform for planning, funding, and building network expansion, replacing a legacy Salesforce system.*

- **UI Lead & design system (IC):** Serve as UI Lead for Sphere and own its shared component library (`React, TypeScript, MUI v7`), standardizing architecture, accessibility, theme tokens, and reusable patterns across **[N Sphere modules / M engineers]** to reduce cross-squad UI duplication and upgrade friction.
- **Cross-service code review:** Serve as go-to reviewer for both UI and backend changes across Sphere's services, enforcing domain boundaries, query performance, concurrency safety, and rollout hygiene.
- **Production memory forensics:** Traced recurring OOM/memory spikes in a production auto-assignment service to three compounding root causes—a `JSONB` column up to **63 KB** over-fetched on every retry, payload bloat across **27,876 rows**, and a **10-minute retry loop** reloading it before GC settled—and eliminated the spikes via column-scoped queries.
- **Query performance:** Accelerated the intake listing API **16×** (**[~X.Xs → ~Yms p95]**) by profiling PostgreSQL execution plans and rewriting **[the underlying query/pagination structure]**.
- **Domain & concurrency boundaries:** Eliminated a page-refresh race condition by moving task-to-job resolution into an atomic task-based backend API; prevented stale job actions from leaking across updates by centralizing field handling and transition rules in one pure domain function.
- **Platform migration & shared finance APIs:** Led strangler-fig migration of four core domains (Job Details, Projects & Phases, Addressing, Right of Entry/Site Contacts) off Salesforce into Sphere with origin-tagged CDC replication back to Salesforce; designed shared Finance APIs (labor rates, funding approvals, PO reservations) featuring a pre-flight eligibility check that eliminated redundant PO reservation calls on every task assignment.
- **Enterprise LLM assistant:** Built the conversational front-end and routing integration for Sphere's privacy-first LLM assistant (`Next.js 16, React 19, TypeScript, Ollama`), using domain-expert routing across invoices, jobs, permits, and finance to render schema-validated tables and charts on local models.
- **Real-time telemetry dashboards:** Built a network port-connectivity dashboard (`Next.js 15, React 19, GraphQL Code Generator, SSO, Zustand, live charts, map`) and an Ethernet-over-HFC reliability dashboard with smooth viewport updates.
- **Computer vision application:** Built the full-stack ingestion pipeline, async inference job orchestration, MongoDB store, and React/MUI human-in-the-loop verification UI for **Find My Power**, identifying power supplies in Google Street View imagery to reduce manual field checks.

### NortonLifeLock (Symantec) — Software Engineer
*Chennai, India · Jun 2015 – May 2020*

- **Cross-browser extension security:** Engineered autofill, credential-capture, and isolated-world messaging features for **Norton Password Manager** across Chrome, Firefox, and Edge—handling untrusted third-party DOMs, dynamic SPAs, and strict CSP/security boundaries for **[millions of consumer users]**.
- **Web vault & portal engineering:** Built user authentication and vault management portal workflows in **React and TypeScript**, partnering with security teams on zero-regression consumer releases.
- **Core product integration:** Delivered identity-protection features inside **Norton Identity Safe** (Norton AntiVirus), resolving **[1 concrete cross-browser/SPA DOM or performance issue]**.

---

## Products & Engineering Projects

*(Architected as automated, zero-daily-ops systems built outside work hours)*

### DevFlow — [wtool.dev](https://wtool.dev)
*Solo Product · Next.js 16 (Cache Components), React 19 (React Compiler), TypeScript, Tailwind CSS, Drizzle ORM, MySQL*

- Visual developer-tools pipeline platform chaining transforms into client-side DAG workflows (zero server data exposure) with AI explanations and auto-fixes via Vercel AI SDK (Anthropic, Groq) across **[N tools]**.
- Production SaaS stack with Better Auth, Razorpay webhook billing, Resend transactional email, analytics, and automated Vitest + Playwright regression suites.

### FaucetWorld — [faucetworld.in](https://faucetworld.in)
*Solo Product (Since 2019) · Next.js 16, React 19, TypeScript, Express, MySQL, Redis*

- Autonomous rewards platform operating continuously since **2019** (**[uptime / payout volume / active user metric]**) using a Next.js app and Express worker service for scheduled jobs, `ethers.js` blockchain payouts, and Redis-backed rate limiting/anti-abuse.
- Integrates signed third-party offerwalls, an affiliate engine, and an LLM-powered support and trivia assistant with zero daily manual intervention.

*Additional shipped sites: [couponcode.dev](https://couponcode.dev) (automated course-coupon aggregator, since 2022) and [uicraft.dev](https://uicraft.dev) (Next.js/Tailwind UI template marketplace).*

---

## Talks

- **"Astro: The Framework That Lets React Shine Brighter"** — Chennai React Meetup

---

## Education

**Master of Computer Applications (MCA)** — National Institute of Technology Tiruchirappalli (NIT Trichy), 2012–2015
