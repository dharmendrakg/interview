# Dharmendra Kumar Gupta

**Senior Full-Stack Engineer · React · TypeScript · Next.js · Node.js**

Bengaluru, India (open to remote) · [+91 7200939340](tel:+917200939340) · [dhar.kg@gmail.com](mailto:dhar.kg@gmail.com) · [LinkedIn](https://www.linkedin.com/in/dharmendra-kumar-gupta) · [GitHub](https://github.com/dharmendrakg)

---

## Summary

Senior full-stack engineer with 11+ years in web development, including 6+ on Sphere at Comcast as UI lead and go-to reviewer for UI and backend changes. Works AI-first: built the agent skills, review server and editor tooling that let AI coding agents work on the platform, and gates AI-assisted changes with tests, review and staged rollouts. Diagnoses problems across services, from a 16× query speedup to memory spikes traced to three root causes, and builds LLM features, including an assistant that runs on local models. Builds and runs four live products end to end.

---

## AI-Assisted Engineering

- **Agent skills:** Wrote domain skills that teach AI coding agents Sphere's architecture, migration playbook and known pitfalls, so their output fits the codebase instead of being generic.
- **Review tooling:** Built a Node.js MCP server that feeds pull-request diffs and distributed traces (Zipkin) to reviewing agents, and a VS Code extension that exposes GitHub, trace and database tools to GitHub Copilot Agent Mode.
- **Verification:** Treats AI-written code as a first draft: unit and end-to-end tests, a structured review pass before opening a pull request, and releases behind feature flags or to staged environments first.

---

## Skills

| Category | Technologies & Practices |
| :--- | :--- |
| **Languages** | TypeScript, JavaScript (ES2015+), SQL |
| **Frontend** | React 19, React Compiler, Next.js (App Router, v14–v16), Tailwind CSS, Material UI (v5–v7), shadcn/ui, MUI X Data Grid, Zustand, TanStack Query, React Hook Form, Storybook |
| **Backend** | Node.js, Express, Next.js API routes and server components, REST and GraphQL APIs, NextAuth, Better Auth, node-cron |
| **Data** | PostgreSQL, MySQL/MariaDB, MongoDB, Drizzle ORM, Redis |
| **AI** | LLM apps (Vercel AI SDK, Ollama), Model Context Protocol (MCP), GitHub Copilot tools, agent skills and plugins, prompt engineering |
| **Quality and delivery** | Vitest, Playwright, Docker, CI/CD, feature flags, code review |

---

## Experience

### Comcast — Senior Software Engineer (Development Engineer 4)
*Bengaluru, India · Jun 2020 – Present*

*Sphere: Comcast's platform for planning, funding and building network expansion, replacing a legacy Salesforce system.*

- **UI leadership:** Lead UI development for Sphere and own its design system (React, TypeScript, MUI v7), the shared component layer for Sphere's modules.
- **Code review:** Go-to reviewer for UI and backend changes across Sphere's services.
- **Root-cause analysis:** Traced recurring memory spikes in a production auto-assignment service to three compounding causes (a JSONB column of up to 63 KB over-fetched on every retry, payload bloat across 27,876 rows, a retry loop reloading it every 10 minutes) and eliminated them with column-scoped queries.
- **Performance:** Made the intake listing API 16× faster by rewriting its query.
- **Design decisions:** Removed a page-refresh race condition by resolving task-to-job mapping in a task-based API instead of in the client; stopped stale job actions from leaking through job updates by centralizing field handling in one domain function.
- **Cross-system API design:** Designed finance APIs shared by Salesforce and Sphere (labor rates, funding approvals, purchase-order reservations), including an eligibility check that spares Salesforce a redundant reservation call on every task assignment.
- **Platform migration:** Migrated Job Details, Projects & Phases, Addressing, and Right of Entry/Site Contacts off Salesforce into Sphere, with change-data-capture replication back to Salesforce.
- **LLM assistant:** Built the conversational front-end of an LLM assistant for Sphere (Next.js 16, React 19, TypeScript, Ollama): domain-expert routing for invoices, jobs, permits and finance, with answers rendered as tables and charts on local models.
- **Dashboards and prototypes:** Built a network port-connectivity dashboard (Next.js 15, React 19, GraphQL Code Generator, SSO, Zustand, live charts, map) and an Ethernet-over-HFC reliability dashboard; also the UI for an AI hiring prototype and a Next.js 14 proof of concept for Sphere (TanStack Query, GraphQL Yoga, NextAuth).
- **Computer vision:** Built Find My Power (React, Material UI, Node.js, MongoDB), which identifies power supplies in Google Street View imagery with a machine-learning model to cut manual field checks.

### NortonLifeLock (Symantec) — Software Engineer
*Chennai, India · Jun 2015 – May 2020*

- Built features, UI and fixes for **Norton Password Manager**, a consumer password manager shipped as a cross-browser extension.
- Developed portal pages with **React and TypeScript** alongside the extension.
- Contributed feature development to **Norton Identity Safe**, a module of Norton AntiVirus.

---

## Projects

### DevFlow — [wtool.dev](https://wtool.dev)
*Own product · Next.js 16 (Cache Components), React 19 (React Compiler), TypeScript, Tailwind CSS, Drizzle ORM, MySQL*

- Developer-tools platform where tools chain into visual pipelines, with AI explanations and auto-fixes powered by the Vercel AI SDK (Anthropic, Groq); tools run client-side so data never leaves the browser.
- Runs as a real product: accounts (Better Auth), payments (Razorpay), transactional email (Resend), usage analytics, and unit and end-to-end test suites (Vitest, Playwright).

### FaucetWorld — [faucetworld.in](https://faucetworld.in)
*Own product since 2019 · Next.js 16, React 19, TypeScript, Express, MySQL, Redis*

- Rewards platform that runs itself: a Next.js app plus a Node.js/Express service for scheduled jobs (node-cron), blockchain payouts (ethers.js) and email, with no daily manual work.
- Integrates third-party offerwalls and link shorteners, runs its own affiliate system, and adds an LLM-powered chatbot, trivia challenge and support assistant.

### CouponCode — [couponcode.dev](https://couponcode.dev)
*Own product since 2022 · Next.js, React, Tailwind CSS*

- Aggregates discounted online-course coupons and publishes them on schedule to the site and social media through cron jobs.

### UICraft — [uicraft.dev](https://uicraft.dev)
*Digital marketplace · Next.js templates and UI components*

- Sells premium Next.js templates, themes and UI components.

---

## Talks

- **"Astro: The Framework That Lets React Shine Brighter"** — Chennai React Meetup

---

## Education

**Master of Computer Applications (MCA)** — National Institute of Technology Tiruchirappalli (NIT Trichy), 2012–2015

---

## Languages

English (professional) · Hindi (native)