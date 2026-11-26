# Interview Preparation Plan — Dharmendra Kumar Gupta

Tailored directly to [`resume.md`](../resume.md) (**Senior / Staff Full-Stack Engineer · React · TypeScript · Next.js · Node.js · AI-Assisted Engineering**).

---

## 1. Candidate Positioning & Target Roles

With **11+ years** of experience (5 years at NortonLifeLock, 6+ years at Comcast as DE4 / UI Lead & go-to reviewer, plus 4 live solo products), your strongest positioning is **not** a generic "React/Node developer." You fit three high-value archetypes:

1. **Senior / Staff Full-Stack Engineer (Frontend-Leaning)** — Design systems (MUI v7), complex enterprise workflows, Next.js/React architecture, plus strong Node.js/SQL/API design and root-cause debugging.
2. **Staff / Principal Frontend Architect** — Owning shared component libraries, complex data grids/dashboards, state/data-fetching architecture, performance, and cross-team code review standards.
3. **AI-Native Product / Full-Stack Engineer** — Building real LLM product features (Ollama local routing + structured UI, Vercel AI SDK) *and* developer velocity/verification tooling (MCP servers, Zipkin trace integration, VS Code agent extensions, domain skills).

---

## 2. Resume Strength & Risk Audit

### Unfair Advantages (Lean Hard Into These)
- **Concrete Production Forensics:** Exact numbers (`63 KB` JSONB column, `27,876` rows, `10-minute` retry loop, `16×` query speedup). Very few candidates have crisp metrics like these ready to dissect on a whiteboard.
- **Enterprise Migration Reality:** Replacing Salesforce with Sphere + CDC replication back + cross-system finance APIs proves you handle messy, high-stakes business systems—not just greenfield CRUD.
- **Credible AI-First Workflow:** You built actual MCP/Zipkin/VS Code tooling and gated AI output with Vitest/Playwright/feature flags rather than treating AI as autocomplete.
- **End-to-End Builder Proof:** 4 live products (`wtool.dev`, `faucetworld.in`, `couponcode.dev`, `uicraft.dev`) covering auth, payments, crons, blockchain payouts, Redis, and client-side pipelines.

### Likely Interviewer Probe Areas (Close These Gaps First)
1. **Stack Breadth vs. Internals Depth:** Your resume lists Next.js 14–16 (including Cache Components), React 19 (React Compiler), Drizzle, GraphQL Yoga, PostgreSQL, MySQL, MongoDB, Redis, Docker, ethers.js, and Ollama. Expect targeted deep-dives testing whether you know *under-the-hood mechanics* rather than just APIs.
2. **DSA & Time-Boxed Coding:** At 11+ years, big-tech and top product companies still reject strong architects on 45-minute LeetCode Medium/Hard or practical JS/React/Node live coding if rusty.
3. **System Design Framing (Senior vs. Staff):** Moving from "I fixed X / I built Y" to *trade-offs, capacity math, failure modes, schema evolution, observability, and cross-team alignment*.
4. **Honesty & Scoping on AI Claims:** Be ready to clearly explain how you evaluate LLM accuracy/hallucinations (especially on local Ollama models for finance/invoices/permits) and how you verify AI-generated PRs without becoming a bottleneck.

---

## 3. Preparation Playbooks in This Folder

| File | Purpose | When to Use |
| :--- | :--- | :--- |
| [`01-study-roadmap.md`](./01-study-roadmap.md) | 4-week sprint & 8-week deep tracks, daily schedule, exit criteria | Day 0 planning & weekly check-ins |
| [`02-resume-deep-dive-stories.md`](./02-resume-deep-dive-stories.md) | Every resume bullet/project expanded into 2-min + 10-min scripts & probe Qs | First 3 days + before every recruiter/HM screen |
| [`03-frontend-react-nextjs-prep.md`](./03-frontend-react-nextjs-prep.md) | React 19, Compiler, Next.js 14–16, TS, Design Systems, State, Testing | Core frontend rounds & machine coding |
| [`04-backend-databases-performance-prep.md`](./04-backend-databases-performance-prep.md) | Node.js internals, SQL/JSONB/EXPLAIN, Redis, Auth, APIs, Cron/Webhooks | Backend, debugging & full-stack rounds |
| [`05-system-design-prep.md`](./05-system-design-prep.md) | Backend + Frontend architecture frameworks & 6 domain-matched mock designs | System design rounds (L5/L6) |
| [`06-ai-engineering-prep.md`](./06-ai-engineering-prep.md) | MCP, agent tooling, verification harnesses, Ollama vs cloud LLMs, Generative UI | AI-native engineering & HM discussions |
| [`07-dsa-and-coding-drills.md`](./07-dsa-and-coding-drills.md) | High-ROI 60-problem DSA list + 15 JS/TS/React/Node practical coding drills | Daily coding practice |
| [`08-behavioral-and-leadership-prep.md`](./08-behavioral-and-leadership-prep.md) | Pitches, Senior/Staff behavioral matrix, conflict/review stories, mock rubric & Qs | Recruiter, HM, Bar Raiser / Values rounds |
| [`09-interview-cheat-sheet.md`](./09-interview-cheat-sheet.md) | 1-page pre-interview warmup: metrics, numbers, trade-offs & red-flag checklist | Night before & 30 min before every round |

---

## 4. Quick-Start Checklist (First 48 Hours)

- [ ] Decide target track & timeline in [`01-study-roadmap.md`](./01-study-roadmap.md) (4-week sprint vs. 8-week thorough).
- [ ] Read [`02-resume-deep-dive-stories.md`](./02-resume-deep-dive-stories.md) and fill in the `[VERIFY]` prompts with exact mental numbers/details from Comcast & NortonLifeLock (no confidential specifics, just crisp technical facts).
- [ ] Record yourself delivering the 90-second pitch from [`08-behavioral-and-leadership-prep.md`](./08-behavioral-and-leadership-prep.md).
- [ ] Run one timed 45-minute coding baseline (1 DSA Medium + 1 JS/TS utility) from [`07-dsa-and-coding-drills.md`](./07-dsa-and-coding-drills.md) to calibrate speed.
- [ ] Schedule at least 2 mock system design + 2 mock coding interviews (peer, mentor, or AI-simulated whiteboard).
