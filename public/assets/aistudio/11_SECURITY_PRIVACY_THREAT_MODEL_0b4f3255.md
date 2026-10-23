# Security, Privacy and Threat Model

**Version:** 0.1 proposed · **Status:** release-blocking requirements

## 1. Security goals

Confidentiality of API credentials and local sessions; backend authentication; integrity of council budgets/workflows; predictable use of paid APIs; source and prompt isolation; safe rendering of untrusted model output; recoverability; least privilege.

## 2. Assets and trust boundaries

- API provider keys and account quotas.
- Owner password/session signing key.
- Prompts, model answers, files and local history.
- Server-side policy: allowlisted providers/models, max calls/tokens, timeouts.
- Android APK signing key and release identity.
- GitHub credentials and Actions permissions.
- Provider outputs and external web source contents (untrusted data).

The Android client, imported files, prompts, model output, external web content and browser deep links are untrusted. GitHub Actions artifacts and deployment configuration are sensitive release assets.

## 3. Threat model

| Threat | Example | Required mitigation |
|---|---|---|
| Credential extraction | API key embedded in APK or source map | Keys only in backend secrets; inspect APK and build outputs for secret patterns |
| Unauthorised spend | Public backend endpoint lets outsiders call models | Protect every model route; backend auth and rate limits; test 401/403 on every route |
| Budget bypass | Modified client increases model seats or tokens | Server-signed run plan, schema validation and server-enforced ceilings on every step |
| Brute-force login | Repeated password attempts | Rate limit, progressive delay, safe failure and no verbose auth errors |
| SSRF | Client asks backend to fetch arbitrary provider URL | Provider IDs and base URLs are server allowlisted; no arbitrary URL proxy |
| Prompt injection | Retrieved page tells model to leak data/call tools | Treat retrieved text as untrusted data; no tools/side effects in MVP; separate instructions and data; preserve sources |
| XSS/unsafe Markdown | Model output contains scripts or javascript: links | Raw HTML disabled; sanitize Markdown; validate URL schemes; no arbitrary iframe/embed |
| Malicious import | JSON payload oversized or contains deceptive fields | Byte/depth/count limits, strict schema, escaped rendering, no execution, reject unknown active fields |
| Log leakage | Provider error includes headers/key/prompt | Sanitise error objects and logs; redact credentials; test synthetic canary secrets |
| Session theft | Persistent token in local storage | Token in memory only; HTTPS; short lifetime; no token in exports/logs; logout clears token |
| Dependency compromise | Malicious npm action/dependency | Lockfile, audit reports, pin actions/dependencies to reviewed versions/SHAs where practical |
| Supply-chain/tampered APK | User installs artifact from wrong workflow | Release from owned repo, checksum, signed artifact, build ID and documented source commit |
| Local data exposure | Device loss or unsafe Android backup | No cloud sync; inform owner of local storage exposure; review Android backup rules; app lock is separate from device lock |
| Resource exhaustion | Long prompt or many parallel calls | Input and call caps, rate limits, output caps, timeout, bounded concurrency |

## 4. Authentication and authorisation

One-owner access only. Every API route other than minimal liveness and throttled login must check the session. Do not hide UI routes as the sole protection. The server derives the owner identity from a valid signed token; never trust `userId`, `isAdmin`, `budget`, model name or run state from the client. Re-authentication required after app restart. Protect session-signing secret and owner password as separate environment secrets.

## 5. Secrets management

- Provider keys: backend host secrets only.
- Owner password and session secret: backend host secrets only.
- Android signing keystore/password: stored outside repo and outside ZIP artifacts; maintain an offline recovery copy controlled by owner.
- No secrets in `VITE_*`, `NEXT_PUBLIC_*`, app config, logs, bug reports, screenshots, GitHub issue text or commit history.
- Rotate provider keys immediately after suspected exposure. Revoke old keys at the provider.

## 6. Data handling and privacy

- Prompt and selected attachments are sent to configured provider(s); research mode may send requests to a provider grounding/search system.
- MVP does not log prompt/output content in ordinary application logs and does not use third-party analytics/crash SDKs. A separate TTL-based operational data store is used for active run state and must be access-restricted and purged on completion or expiry.
- Completed conversation history is local; active-run prompts/intermediate outputs are temporarily stored in the secured operational Redis service with short TTL so server-enforced limits, idempotency and multi-stage processing work. User must understand this data leaves the phone and may be accessible to the service operator during the TTL window. Local app data may also be accessible to someone with device access/root/forensic capability.
- Privacy notice before first provider send; explain providers, routing, retention unknowns and model-training/data-processing conditions must be checked against each provider's current terms.
- Do not claim GDPR/CCPA/HIPAA compliance or certification. This is a private personal tool, not designed for protected health data or regulated workflows.

## 7. Security verification gates

Before release: unauthenticated-route matrix; password-throttle test; token expiry test; prompt/size limits; provider/model allowlist; SSRF test; malicious Markdown/import tests; secret scan; APK/source-map scan; log canary scan; package/license audit; TLS and CORS check; no credential in ZIP; exact artifact checksum. Any critical/high security issue blocks release until fixed or explicitly accepted with documented risk (critical issues cannot be accepted for private release).

## 8. Reference baselines

OWASP API Security Top 10: https://owasp.org/API-Security/ · OWASP ASVS: https://owasp.org/www-project-application-security-verification-standard/ · OWASP MASVS: https://mas.owasp.org/MASVS/ · OWASP Cheat Sheet Series: https://cheatsheetseries.owasp.org/
