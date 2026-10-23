# Decision Register and Open Questions

**Version:** 0.1 proposed · **Status:** pending owner approval. No implementation decision is permanently locked until owner approves v1.0.

## 1. Proposed baseline decisions

| ID | Decision | Rationale | Status / evidence needed |
|---|---|---|---|
| ADR-001 | Private Android APK, no store publishing | Matches intended personal use | Proposed; owner lock required |
| ADR-002 | APK UI bundled; backend remains remote | Keeps provider credentials off device and supports server enforcement | Proposed; validate host availability and APK bridge |
| ADR-003 | React + Vite + TypeScript + Capacitor client | Reuses web UI and remote build workflow | Proposed; vertical-slice device test |
| ADR-004 | Next.js Route Handlers backend | Aligns with RoundTable foundation | Proposed; serverless timeouts/API behavior test |
| ADR-005 | Selectively adapt RoundTable logic | Existing CVP/Blind Jury/Red Team concepts reduce reinvention | Exact commit, tests and licence audit required |
| ADR-006 | OpenRouter adapter first | General gateway if it matches user's key and needed models | Open question: owner's provider/key type (do not ask them to reveal the key itself) |
| ADR-007 | Provider keys only in host environment | Prevent extraction from APK | Non-negotiable proposed guardrail |
| ADR-008 | No durable remote conversation archive in MVP; use a TTL operational store for active run state/idempotency | Required for server-side run caps and step replay prevention; adds temporary data processing | Proposed; Upstash feasibility, TTL and atomic operations must be proven |
| ADR-009 | IndexedDB local history | Browser/Capacitor compatible | Test persistence, Android data policy and migration |
| ADR-010 | One-owner auth, short-lived in-memory token | No public users; minimise token persistence | Must select reviewed password/session implementation and test route coverage |
| ADR-011 | Vercel Hobby candidate | Low-friction Next.js hosting | Verify current terms, personal use, idle behavior, function limits, CORS and no-card path |
| ADR-012 | GitHub Actions build/release | Works without desktop Android Studio | Verify private-repo minutes and Android toolchain; no exposed secrets |
| ADR-013 | No arbitrary tools/agents in MVP | Reduces prompt-injection and side-effect risk | Proposed guardrail |
| ADR-014 | Hard budgets/caps and no paid fallback | Prevent uncontrolled costs | Must test bypasses and estimate uncertainty |
| ADR-015 | English UI first; Unicode/Bengali support | Keeps initial localisation small while permitting user's content | Proposed; inspect Bengali rendering and tests |
| ADR-016 | No telemetry vendor in MVP | Privacy/minimal external dependencies | Proposed; local diagnostics only |
| ADR-017 | Owner-only, no monetization | It is for personal usage | Locked by user requirement once plan approved |

## 2. Owner choices required for v1 lock

These concern safe configuration, not a request to share secrets:

1. **Key provider:** Is the key from OpenRouter, OpenAI, Google Gemini, Anthropic, Groq, or another provider? State the provider name only. Compatibility will be tested without pasting the key into chat.
2. **Repository visibility:** Proposed private repo; GitHub Actions minutes/storage are finite. Accept a free private build quota with manual on-demand builds? Do not make repository public to obtain free CI without explicit consent.
3. **Backend access:** Proposed simple owner password + short-lived token. Accept logging in again each time the app fully restarts?
4. **History:** Proposed local-only history with manual JSON/Markdown export and no cloud sync. Accept that phone loss can lose history unless exported?
5. **Hosting:** Proposed to test Vercel Hobby first if eligible without new paid subscription/card. Accept that free deployment may sleep or become unavailable? If it fails feasibility checks, change host before coding further.
6. **Feature cap:** Proposed 4 seats, up to 2 critique rounds, 12 model calls/run, 4,000 output tokens/call, 3 simultaneous calls maximum. Should limits remain conservative initially? Recommended yes.
7. **UI language:** Proposed English interface with full Unicode/Bengali input support. Additional UI languages can be added later.
8. **Local data sensitivity:** Proposed not to use highly sensitive or regulated data. Accept this boundary?
9. **Temporary run processing:** To enforce multi-stage budgets and prevent replay, active prompts and intermediate outputs are proposed to exist temporarily in a secured Redis service with a maximum 24-hour TTL and deletion after the run where possible. Completed history stays on your phone. Accept this additional temporary data-processing boundary?

## 3. Architecture proof gates before final lock

- Audit exact RoundTable commit, package versions, license and current test/build status.
- Confirm provider adapter matches user's provider; test key only in host secret setting.
- Build a blank Capacitor APK from GitHub Actions and install on owner's actual phone.
- Confirm login-protected API call with mock provider then one explicitly approved real call.
- Prove selected host can support bounded calls under current limits and free terms.
- Verify owner can download milestone ZIP and APK using mobile GitHub interface.

If a gate fails, record a decision change and alternatives. Do not pretend an untested host or build works.
