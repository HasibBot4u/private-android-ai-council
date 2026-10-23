# System Architecture and Technology Decisions

**Version:** 0.1 proposed · **Status:** pending source audit and owner approval

## 1. Architecture decision summary

| Area | Proposed choice | Reason / constraint |
|---|---|---|
| Android UI | React + Vite + TypeScript strict | Fast mobile web UI; bundle assets in APK; phone-operated workflow |
| APK shell | Capacitor Android | Reuses web UI and produces a real APK; build remotely in Actions |
| Backend | Next.js App Router Route Handlers + TypeScript | Compatible with selected RoundTable foundation and server-only provider calls |
| Orchestration | Adapt selected RoundTable logic behind typed interfaces | Strong baseline for CVP, blind jury and red-team modes; audit before reuse |
| Validation | Zod at client and server boundaries; server schema is authoritative | Reject malformed and over-budget requests |
| UI state | Zustand for transient view/run state; domain types separate from components | Small state model; persisted session history stays local in IndexedDB |
| Navigation | React Router for client views | Clear screen hierarchy and mobile back navigation |
| Styling | Tailwind CSS plus semantic design tokens | Consistent theme and responsive layout |
| Markdown | Safe renderer with raw HTML disabled and unsafe URL protocols rejected | Reduce XSS risk from model output/imported content |
| Provider gateway | OpenRouter adapter first; other providers only after compatibility tests | Existing key/provider is not yet known; do not claim all keys are interchangeable |
| Local data | IndexedDB via a maintained wrapper | Completed sessions stay local; no durable cloud conversation archive |
| Hosting candidate | Vercel Hobby for personal/noncommercial use, after proof-of-concept | Simple Next.js deployment; free limits/inactivity/streaming require validation |
| CI | GitHub Actions | Lint, tests, build Android APK and package release artifacts online |
| Testing | Vitest, Testing Library, Playwright, API contract tests, manual Android UAT | Unit through end-to-end coverage |

## 2. Trust boundary

APK includes UI and public backend address only. It contains no provider key, backend signing secret, password, release keystore or fixed admin credential. HTTPS connects to backend. Backend authenticates every protected route and enforces model/provider allowlists, input size, call/token limits and timeouts before contacting a provider. Providers receive only the question and minimum context required by the selected workflow.

## 3. Council engine interface

Use a provider-neutral typed interface with explicit functions:

- `listConfiguredModels()` — only server-approved IDs, capabilities, price metadata and availability.
- `preflightRun(input)` — compute planned steps/calls and validate hard ceilings.
- `startRun(input)` — create a run with a random ID and explicit plan.
- `executeStep(runId, stepId)` — exactly one bounded logical step; provider adapter performs one bounded request unless a recorded retry is allowed.
- `cancelRun(runId)` — cancel queued work and signal best-effort abort of active work.
- `getRunStatus(runId)` — return sanitised stage and error state.

Every step record includes mode, sequence, model ID, call category, timestamps, provider status, input/output token counts when available, estimated/actual known cost, retry number and safe error code. Do not store API keys or unsanitised request headers.

## 4. Execution and serverless limits

Do not run an unbounded multi-round council in a single serverless invocation. Use short, bounded step calls; client controls sequencing and receives progress. Backend revalidates run limits on every step. Use a run-scoped server-signed policy/nonce or equivalent state machine to prevent a modified client from expanding the approved plan. Do not depend on client-supplied counters as budget authority.

The client submits one bounded step at a time, but the server-side TTL ledger is the authority for the approved run plan, consumed call count, idempotency and intermediate stage state. Do not let client-supplied counters or model answers alone determine the next authorised step. Upstash free-tier suitability, no-card onboarding, TTL support, atomicity and quota behavior must pass a proof-of-concept before stack lock. If unsuitable, evaluate a free relational DB or compatible server platform before implementation continues.

## 5. State architecture

- **UI state (memory only):** active selection, transient loading, session token, current run view.
- **Local domain data (IndexedDB):** saved conversations/results, timestamps, settings, imported/exported records.
- **Server configuration:** configured provider IDs/model IDs, secret references, server-side hard limits, owner auth secret.
- **No MVP durable remote conversation archive.** Redis temporarily stores the minimum active-run prompt/context/intermediate stage data required to enforce server-authoritative run state, replay prevention and budgets. TTL is capped at 24 hours and active run records should be deleted after completion/cancellation; no prompt/output in ordinary logs.

## 6. Deployment topology

- `apps/mobile`: Vite-built bundled UI plus Capacitor native Android wrapper.
- `apps/backend` (or root backend after repo audit): Next.js authenticated API and council adapters.
- `packages/contracts`: shared Zod/TypeScript DTO schemas where safe; never share server secrets or Node-only providers into client bundle.
- `tests`: unit, API/security, integration and E2E tests.
- `.github/workflows`: CI, APK build, artifact bundle, release checksum.
- `docs`: approved specifications, ADRs, test reports, changelog and recovery guides.

The final monorepo layout is decided after the RoundTable audit to minimise a risky rewrite and avoid importing unneeded dependencies.

## 7. Technology decision rules

Pin runtime/package versions and commit hashes. Choose the simplest service that satisfies proven needs. No Kubernetes, Docker orchestration, separate queue, durable conversation database, multi-tenant RBAC, GraphQL or microservice split in MVP. A small managed Redis operational ledger is the one proposed external state service and must pass a no-card/free-tier feasibility test. Other infrastructure services are deferred unless an actual need/test demonstrates value. Re-evaluate free-tier hosting and API streaming support at implementation time.
