# 03 — Frontend, React 19, Next.js 14–16 & Design Systems Prep

Your resume claims **UI leadership, React 19 + React Compiler, Next.js 14–16 (including Cache Components), TypeScript, MUI v7, shadcn/ui, Tailwind, MUI X Data Grid, Zustand, TanStack Query, React Hook Form, GraphQL Codegen, Storybook, Vitest, Playwright**. Expect interviewers to test **internals**, not surface syntax.

---

## 1. React 19 & Concurrent Rendering Internals

### High-Probability Questions & Crisp Answers

1. **How does React Compiler work, and what rules must code follow for it to optimize safely?**
   - **Mechanism:** A build-time Babel/compiler pass that analyzes component/hook dataflow and automatically inserts fine-grained memoization (`memoize` cache slots) for JSX elements, callbacks, and computed values—often more granularly than manual `useMemo`/`useCallback`.
   - **Rules of React required:** Components/hooks must be pure during render, props/state must be immutable (mutating an array/object in place breaks dependency checks), and hooks cannot be called conditionally.
   - **Pitfall:** Third-party libraries or legacy code that mutate refs/objects during render or pass unstable refs; explain how you adopt it incrementally (`"use memo"` / directory opt-in or fixing ESLint `react-hooks` / compiler diagnostics).

2. **React 19 Features You Should Know Cold:**
   - **Actions & `useActionState` / `useFormStatus` / `useOptimistic`:** Async transitions automatically manage pending state, error boundaries, and optimistic rollbacks without manual `isLoading` boilerplate.
   - **`use(promiseOrContext)`:** Reads promises (integrating with `<Suspense>`) and context conditionally inside render (unlike `useContext`).
   - **`ref` as a regular prop:** No more `forwardRef` boilerplate in React 19 (huge win for MUI/design-system wrapper components!).
   - **Transitions (`startTransition` / `useTransition`):** Marks state updates as non-urgent so urgent input/typing stays responsive; React can interrupt and discard stale renders.

3. **Reconciliation, Keys, and Stale Closures:**
   - Why array index keys corrupt state on reorder/insert/delete.
   - How `key` reset pattern works intentionally (e.g., resetting a form when `jobId` changes instead of ugly `useEffect` sync).
   - Stale closures in `setInterval`, event listeners, or async callbacks—fixed via functional updates `setState(prev => ...)`, `useEffectEvent` / refs, or moving resolution server-side (tie to your Sphere task-to-job story!).

---

## 2. Next.js (v14–v16) App Router & Cache Components

Because you explicitly listed **Next.js 14, 15, 16 (Cache Components)** across Sphere and DevFlow, expect version-evolution questions:

1. **RSC (React Server Components) vs. SSR vs. Client Components:**
   - **SSR** renders HTML on the server for initial paint, then hydrates the full component JS on the client.
   - **RSC** executes *only* on the server, never ships its component code/dependencies (like DB drivers, Drizzle, heavy markdown/parsers) to the client bundle, and streams a serialized RSC payload.
   - **Boundary rule:** `'use client'` marks the network/bundle boundary. Server Components can pass serializable props (and even Server Component `children`) into Client Components, whereas importing a Server Component directly inside a `'use client'` file turns it into a client module.

2. **Caching Evolution (Next.js 14 → 15 → 16):**
   - **Next 14:** Aggressive implicit caching (`fetch` cached by default, Full Route Cache, Router Cache) caused surprising stale-data bugs.
   - **Next 15:** Flipped defaults—`fetch` and Route Handlers (`GET`) became uncached by default; dynamic APIs (`cookies()`, `headers()`, `params`) became async.
   - **Next 16 (Cache Components / `'use cache'`):** Explicit opt-in caching at the page, component, or function level using `"use cache"` paired with Partial Prerendering (PPR)—static cached shells stream immediately while dynamic `<Suspense>` holes stream per request, invalidated via `cacheLife` and `cacheTag` / `revalidateTag`.

3. **Server Actions Security Checklist:**
   - Treat every Server Action like a **public POST endpoint**!
   - Always re-verify authentication + authorization inside the action (never trust hidden form fields or client state), validate payloads with Zod, and watch out for Closure Encryption (don't accidentally close over secrets passed down or rely on client invisibility).

---

## 3. Design Systems (MUI v7, shadcn/ui, Tailwind, Storybook)

Tie answers to **leading Sphere's UI & MUI v7 design system**:
- **MUI v7 vs. Tailwind/shadcn trade-offs:**
  - *MUI v7:* Best for enterprise apps needing complex accessible widgets out of the box (MUI X Data Grid, Date Pickers, complex Dialog/Menu keyboard focus traps) and centralized theme tokens across multiple product squads. Watch bundle size (tree-shaking, avoiding barrel imports), CSS specificity/Emotion vs. Pigment CSS / CSS variables.
  - *Tailwind + shadcn/ui:* Copy-owned primitives (Radix/Base UI + Tailwind), zero runtime CSS-in-JS overhead, ideal for consumer/product apps like **DevFlow** and **UICraft** where bespoke visual identity and RSC compatibility matter.
- **Preventing Design System Rot:**
  - Expose semantic variants (`<StatusBadge tone="warning" />`), not raw escape hatches everywhere.
  - Visual regression & interaction tests in Storybook/Playwright; enforce a11y (`aria-*`, focus management, contrast, keyboard navigation).
  - Never leak MUI internals in domain business components if you want smooth major upgrades (v5 → v7).

---

## 4. State Management, Data Fetching & Complex UI Performance

- **Server State vs. Client UI State:**
  - **TanStack Query / GraphQL Codegen:** Owns caching, deduplication, background refetch, stale-while-revalidate, request cancellation (`signal`), optimistic updates. Never mirror server responses into Zustand!
  - **Zustand:** Owns ephemeral cross-component client state (selected map pins, split-pane layout, multi-step wizard draft, active filter panel). Use granular selectors (`useStore(s => s.selectedId)`) to avoid re-renders.
  - **React Hook Form:** Uncontrolled refs for zero-lag typing in massive enterprise forms (e.g., Sphere Job Details / Permits) + Zod schema resolver.
- **MUI X Data Grid & Live Dashboards (25k+ rows, charts, maps):**
  - Server-side pagination, sorting, and filtering for 27k+ row datasets (never ship 27k rows to browser memory!).
  - Row/column virtualization for wide grids; stable `columns` memoization and cell renderers.
  - Live telemetry dashboards: throttle/batch WebSocket or polling updates (`requestAnimationFrame` or 500ms buffer), use Canvas/WebGL for high-density maps/charts rather than thousands of SVG DOM nodes.

---

## 5. TypeScript Mastery & Testing (Vitest + Playwright)

- **TS Patterns to Practice:** Discriminated unions for API states & domain events (`type Result<T> = { ok: true; data: T } | { ok: false; error: AppError }`), `satisfies`, `infer`, mapped/template literal types, exhaustive `never` checks in `switch`, Zod `z.infer` as single runtime+compile-time source of truth.
- **Compiler & Type-System Internals:**
  - *Distributive conditional types:* `T extends U ? X : Y` distributes over union `T` unless wrapped in `[T] extends [U]`.
  - *Variance:* Function parameters are **contravariant** under `strictFunctionTypes`, return types are **covariant**; `readonly` prevents unsafe array mutation covariance bugs.
  - *`NoInfer<T>` (TS 5.4+):* Stops a secondary argument from widening a generic parameter inferred from the primary argument (great for design-system/hook APIs!).
- **Testing Strategy (Your "AI verification" story):**
  - **Vitest:** Pure domain functions (centralized job field handler!), custom hooks, Zod schemas, edge-case state reducers.
  - **Playwright:** Critical user journeys (auth, checkout/Razorpay, form submission, race-condition page-refresh tests), network mocking with HAR/route interception only when testing deterministic failure states.

---

## 6. Core JavaScript, Browser Internals, Web Vitals & Security

Interviewers frequently test **foundational JS & browser mechanics** even at Staff level:

1. **Browser Event Loop & Rendering Pipeline:**
   - One macrotask (`click`, `setTimeout`, network callback) → drain **all** microtasks (`Promise.then`, `queueMicrotask`) → `requestAnimationFrame` callbacks → Style/Layout/Paint/Composite (if frame budget ~16.6ms is due).
   - **Web Workers (DevFlow `wtool.dev`):** Offload CPU-heavy parsing/regex/transforms off the main thread; communication via `postMessage` uses structured cloning (or zero-copy `Transferable` `ArrayBuffer`s).
2. **Core Web Vitals (Measure & Fix):**
   - **LCP (Largest Contentful Paint, <2.5s):** Eliminate render-blocking scripts/CSS, SSR/PPR critical HTML, `next/image` `priority` / `fetchPriority="high"`, preload hero font/asset.
   - **INP (Interaction to Next Paint, <200ms — replaced FID):** Break up >50ms Long Tasks (`scheduler.yield()` / `startTransition`), virtualize huge tables, throttle live telemetry flushes, avoid forced synchronous reflows.
   - **CLS (Cumulative Layout Shift, <0.1):** Explicit `width`/`height` or `aspect-ratio` on media/charts, reserve skeleton heights, `font-display: swap` with size-adjusted fallback.
3. **Browser Security Boundaries (NortonLifeLock + Sphere SSO):**
   - **CORS:** Simple vs. preflight (`OPTIONS`) requests triggered by custom headers (`Authorization`), non-simple `Content-Type` (`application/json`), or `PUT/PATCH/DELETE`. Server must return `Access-Control-Allow-Origin` (never `*` with `Access-Control-Allow-Credentials: true`).
   - **CSP (`Content-Security-Policy`):** Mitigates XSS by restricting script origins (`nonce-...` + `'strict-dynamic'`), banning inline `eval`/scripts, and controlling `frame-ancestors` (clickjacking defense).
   - **`postMessage` & Extension Messaging:** Always verify `event.origin` on the receiver and pass a strict targetOrigin (never `'*'` for sensitive data) on the sender.
   - **Where to store auth tokens:** Prefer `HttpOnly; Secure; SameSite=Lax` cookies via BFF/Next.js server over `localStorage` (any XSS payload can exfiltrate `localStorage`).

---

## 7. Rapid-Fire Frontend Check Questions (Test Yourself)

1. Why does `React.StrictMode` double-invoke effects in dev, and what bug does it expose? *(Missing cleanup—e.g., uncancelled fetch/subscription or duplicate WebSocket listeners.)*
2. How do you cancel an outdated search fetch in React without race conditions? *(`AbortController` passed to `fetch` in `useEffect` cleanup, or TanStack Query `queryFn({ signal })`.)*
3. What causes layout thrashing in the browser, and how do you inspect it? *(Interleaved DOM reads like `offsetHeight` and writes in loops causing forced synchronous reflows; inspect via Chrome DevTools Performance flamegraph purple Layout bars.)*
4. How does a cross-browser extension (Norton Password Manager) avoid leaking secrets to a hostile page's JS? *(Isolated world for content scripts, keeping vault/keys in background service worker, strict `origin` validation, avoiding injecting secrets into global `window`.)*
5. When would you choose WebSockets vs. SSE vs. short/long polling for a Sphere dashboard? *(SSE for unidirectional live status/LLM streaming over HTTP; WebSockets for bidirectional low-latency telemetry; TanStack Query polling for 10–30s enterprise metrics.)*
