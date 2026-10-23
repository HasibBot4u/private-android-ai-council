# Data Model, API Contracts and State Transitions

**Version:** 0.1 proposed · **Status:** pending approval

## 1. Core domain entities

| Entity | Fields (conceptual) | Storage |
|---|---|---|
| Conversation | id, title, createdAt, updatedAt, tags, schemaVersion | IndexedDB local |
| CouncilRun | id, conversationId, mode, prompt, state, planSummary, startedAt, finishedAt, stopReason | IndexedDB local; safe run state received from server |
| RunStep | id, runId, sequence, stage, seatLabel, modelId, state, start/end time, retry count, safeErrorCode | Redis operational state while run is active; local after receipt |
| ModelAnswer | id, stepId, text, provider/model display name, usage metadata if returned | IndexedDB local |
| PeerEvaluation | candidate IDs, randomised labels, criteria, ranking, explanation | IndexedDB local |
| Objection | id, targetClaimId/text, challenger, challenge, evidenceRefs, response, disposition | IndexedDB local |
| Claim | id, text, status, explanation, sourceRefIds, checkingMethod | IndexedDB local |
| SourceRef | id, title, URL, publisher/domain, retrievedAt, publishedAt if known, retrievalMethod | IndexedDB local |
| UsageSummary | planned/used calls, tokens where known, cost estimate, actual cost when known, unknowns | IndexedDB local |
| AppSettings | theme, defaultMode, UI preferences, server endpoint, configured non-secret choices | IndexedDB/local config; never store auth token or provider key in export |

The MVP has no durable remote conversation archive. A short-lived Redis operational store holds only the minimum active-run state/context needed to prevent replay and continue stages. It must expire automatically on success/failure/cancel (with an upper-bound TTL as fallback); store no completed chat archive there. Sensitive session details are never emitted to analytics or ordinary server logs.

## 2. Data retention and exports

- Completed local history persists until the owner deletes it or clears app data. Active run context/intermediate output is stored temporarily in an authenticated Redis operational service, removed on completion/cancel where practical and subject to a maximum 24-hour TTL.
- The owner may export a single/all conversations as Markdown or versioned JSON.
- Export omits password, access token, provider credentials, private endpoint secrets and environment config.
- Import limits file size/depth/record count, validates schema version and normalises URLs; it renders text only and executes nothing.
- Clear-history action requires confirmation; OS backup behavior must be checked during Android build so private sessions are not unintentionally backed up to a personal cloud account.

## 3. Initial REST API

Base path: `/api/v1`. HTTPS required. JSON DTOs are versioned; all route handlers validate input/output.

| Method + path | Auth | Purpose |
|---|---|---|
| GET `/health` | Public but minimal | Liveness/build version only; no env data |
| POST `/auth/login` | Public, throttled | Exchange owner password for short-lived session |
| POST `/auth/logout` | Required | Revoke/invalidate session where supported |
| GET `/models` | Required | List allowlisted provider/model capabilities and price metadata |
| POST `/runs/preflight` | Required | Validate plan and return limits/cost unknowns before any model call |
| POST `/runs` | Required | Create a bounded run plan/identifier; no model calls before preflight accepted |
| POST `/runs/{runId}/steps/{stepId}` | Required | Execute one allowed bounded step or return existing idempotent result |
| GET `/runs/{runId}` | Required | Return safe current status/progress |
| POST `/runs/{runId}/cancel` | Required | Cancel not-yet-started work / request provider abort |
| POST `/provider-check` | Required, limited | Test a configured provider/model with minimal request |

Never accept arbitrary URLs or raw credentials from a run request. For the login endpoint, apply rate limiting before expensive work. For run/step endpoints, enforce request size, prompt length, model allowlist, planned call count, token caps, per-run concurrency and timeouts server-side.

## 4. Authentication session policy

- One-owner password configured as a host secret; never commit it.
- Password compared using a secure password hashing/verification approach supported by the deployment environment (not plaintext comparison in app code).
- Short-lived signed session token with issuer/audience/expiry; secret remains in backend environment.
- Client token held in app memory only; closing/restarting app means logging in again.
- No refresh token persisted on disk in MVP. Logout clears memory. Server secret rotation invalidates existing tokens.
- All private routes require a valid token and reject expired/replayed/incompatible sessions.

Redis operational records may also track login attempt limits, active session revocation/generation, approved run plan, consumed step IDs, budget reservations and idempotency. No durable completed-chat archive is stored there. This service is an external data processor and must be disclosed to the owner.

## 5. API contract conventions

- Standard error shape: `{ "error": { "code": "SAFE_CODE", "message": "Human-readable message", "retryable": false, "requestId": "opaque-id" } }`.
- Do not return stack traces, env values or raw provider error payloads.
- Use explicit idempotency keys for run creation and step execution so rapid repeats do not silently multiply calls.
- Timeouts and partial output must be represented in a typed status.
- See `contracts/openapi.yaml` for the initial OpenAPI skeleton; refine schemas during implementation without silently changing approved semantics.
