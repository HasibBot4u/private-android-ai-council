# Hosting, DevOps, CI/CD and Release Architecture

**Version:** 0.1 proposed · **Status:** pending feasibility proof

## 1. Environments

- **Development:** local/remote ephemeral test configuration; use fake provider adapters by default.
- **CI:** GitHub Actions with no provider secrets; lint, types, unit tests, security checks and frontend/Android build.
- **Preview/staging:** optional low-risk deployment for owner acceptance, with separate secret and strict spend cap. Never expose a public unauthenticated preview.
- **Private production:** authenticated backend deployment and manually approved APK. Use separate config from staging.

Because this is a single-owner private app, do not create Kubernetes, multi-region clusters, load balancers, queue clusters or multiple microservices unless a measured need arises.

## 2. Hosting decision process

Vercel Hobby is the initial candidate because the backend is Next.js, but its current terms/limits, owner eligibility, inactivity behavior, route duration, request body limits, CORS and any streaming behavior must be checked at implementation time. The app should work with bounded non-streaming calls if SSE proves unreliable. A different host may be selected after the same proof-of-concept. Do not promise always-on availability on a free plan. Validate that the host can connect to the selected Redis service securely and that timeout/retry behavior preserves idempotency.

Possible alternative: a Cloudflare-based backend only if required SDK/runtime APIs and deployment path are compatible with the selected server code. This is not automatically a drop-in replacement for Next.js. Do not deploy Docker/Kubernetes as a shortcut; it adds operational complexity and possible cost.

## 3. CI pipeline

1. Trigger on pull request/push: install locked Node dependencies; lint; type-check; unit/component tests; API/security tests with mock provider; build backend and mobile web bundle; dependency/secret scan.
2. Trigger mobile package job after web tests pass: install pinned JDK/Android SDK/Capacitor dependencies; build debug APK for branch validation; optionally release APK only on manually approved/tagged workflow.
3. Verify APK file exists, package ID/version are correct, archive contains no `.env`, provider key, session secret, or source-only signing material.
4. Generate SHA-256 checksum, changelog/build metadata, test summary and software/licence inventory.
5. Build a full source/docs ZIP; exclude `.env`, `node_modules`, caches, local histories, secrets and keystore files.
6. Upload artifact with retention compatible with GitHub quota; owner downloads it after successful workflow.
7. A failed security/test gate must not be labelled passing or produce a release tagged as stable.

## 4. Git and release policy

- `main`: last known-good integrated state.
- Feature branches for material changes; small phone-friendly changesets.
- Pull request or equivalent review checklist: requirement IDs, tests, security/cost impact, docs, dependency/licence implications.
- Manual release tag only after UAT approval.
- Pin GitHub Actions to reviewed version/SHA where feasible; third-party actions are code dependencies.
- Use conventional commit prefixes optionally (`feat:`, `fix:`, `docs:`, `test:`, `security:`); commit messages never include secrets.
- Never rely on Replit pushing code to GitHub. After every major milestone supply a ZIP independently.

## 5. Android signing

- Set unique reverse-domain application ID once and lock it before first release.
- Debug APK may use ephemeral debug key; release APK must use a stable owner-controlled signing key so future updates install over the previous version.
- Signing keystore and passwords stored as GitHub Actions secrets/secure file injection or an equally protected method; never committed, logged or bundled in source ZIP.
- If the keystore is lost, a future release cannot update the installed app under the same identity; recovery instructions and offline backup are mandatory.
- Provide an unsigned/debug preview only for non-sensitive internal testing if safe release signing cannot be set up yet.

## 6. Free quota/billing guardrails

- Check current Actions quota for private repositories and artifact-retention limits before build frequency is finalised.
- Public repository only by deliberate owner choice after assessing that all source/docs are safe to publish; never make repo public merely to save minutes without approval.
- No card requirement assumed. If a service requires billing setup, pause and select another feasible option rather than instructing the owner to add a card.
- Provider-side spend controls are supplementary; in-app/server caps remain required.

## 7. Backup and disaster recovery

- Source is reproducible from Git and exported milestone ZIPs.
- Owner downloads and retains each release ZIP and SHA-256 manifest.
- Back up signing key securely offline outside repository/artifacts.
- Export local history JSON when wanted; MVP has no cloud-sync recovery for phone loss.
- Recovery runbook: restore source ZIP/repo → check locked tool versions → restore signing secret securely → run CI → verify checksum → reinstall/upgrade APK.
- Hosting recovery: verify provider uptime/quotas; redeploy known-good commit; rotate compromised secrets; re-run smoke/UAT tests.

## 8. Release channels

Private release only: GitHub artifact download to owner's phone. No Play Store/TestFlight, ASO, public landing page, waitlist, advertising, public onboarding or customer helpdesk in this project.
