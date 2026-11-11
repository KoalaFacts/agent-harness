# Press Floor — project rules

Owner-controlled seed for the "Press Floor 推进计划" scheduled task (repo KoalaFacts/press-floor). The repo skill is `.claude/skills/advance-press-floor/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
the ledger is docs/plans/roadmap.md only if it already exists; otherwise a "Roadmap" section of README.md next to "Remaining boundaries". P5's data-lifecycle design note goes in a section of the closest existing record in docs/decisions/, not a new file. P6's docs/archive/ move is dropped: docs/BASIC-STATUS.md and finished docs/plans/2026-10-* plans are folded into the page that owns any still-true content and deleted; docs/README.md is kept only if it already exists. Owner-decision, owner-acceptance, provenance and evidence docs (AGENTS.md's owner directions, docs/decisions/, COMMONWEALTH-ARMS.md, REFERENCE-CALIBRATION.md, EDITORIAL-PRINCIPLES.md, verification/acceptance records) are kept records. docs/topics/ and content/topics/ belong to the daily topic task and are data: never counted, never touched.

## Verification boundary
the real boundary is the local Worker (Miniflare with local D1/R2/Durable Object) driven in headless Chromium through a room, the composer, review and local issues, with fixtures only — never the real TypeSafe model, Google or pressfloor.org writes. The skills line goes in AGENTS.md, never touching the owner-direction lines.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/press-floor (pressfloor.org): an independent, source-first public policy discussion forum — Purity + TypeScript, Cloudflare Workers, D1 via Drizzle + Zod + the CQRS mediator in cloudflare/cqrs, a Durable Object for realtime, R2 for local-issue photos, Better Auth Google/guest sign-in, TypeSafe AI "Jev" first review.
- Goal: a reliable, safe, easy-to-use forum where several policy rooms run at once, contributions are reviewed before they are public, and the docs always say exactly what is true and are easy to find.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feat/`, `fix/`, `test/`, `docs/`, `refactor/`, `perf/`; squash merge. For this task the PR rhythm replaces AGENTS.md's "work directly on main"; everything else in AGENTS.md stays binding.
- Merging to main does NOT deploy production: deploy.yml's deploy job runs only on workflow_dispatch. Deploying stays the owner's: never dispatch or re-run deploy.yml, room-publish.yml, content-sync.yml, moderator.yml, cloudflare-auth-secret.yml, website-email-routing.yml or any workflow that writes to Cloudflare; never run wrangler against a remote. Read-only public checks of https://pressfloor.org are allowed.
- Every open PR in the repo (Codex, Dependabot, other sessions, the owner's) is in scope for open-PR triage (autonomous-dev §3 step 0).
- The daily "Press Floor 每日选题" task commits scouted topics straight to main (content/topics/<date>/, docs/topics/<date>.md) around 06:52 Sydney. Never edit, move or delete those files, content/sections.json, published room content or locale copy; never touch docs/topics/.

### P3 SETUP AND GATE
- Node 24 and npm 11. Read AGENTS.md (binding), docs/ROOMS-AND-CONTENT.md, docs/EDITORIAL-PRINCIPLES.md, docs/LOCAL-ISSUES.md, docs/CLOUDFLARE-DEPLOYMENT.md, docs/SEARCH-AND-AI-DISCOVERY.md, docs/decisions/.
- Actions here has been billing-blocked since 2026-11-02; the local gate is the evidence.
- LOCAL GATE (what deploy.yml's verify job runs): `npm ci`, `npm test` (types, both bundles, node:test), `npm run test:cloudflare` (Workers/D1 via Miniflare); `npx --no-install playwright install chromium` then `npm run test:browser`, `npm run test:cloudflare:browser`, `npm run test:dev` and the interface/venue/camera check deploy.yml runs; for content-pipeline changes `node scripts/cloudflare-content.mjs sync --dry-run`; the docs checker (scripts/check-docs.ts run with --experimental-transform-types). If a browser suite cannot run here, say exactly which, and do not merge a UI change whose only proof would be that suite.

### P4 LEDGER AND ORDER
- Ledger: per B11 (docs/plans/roadmap.md if it exists, built from the code, README's "Remaining boundaries", the docs and recent PRs).
- Initial order:
  P1 Docs truth pass: README, AGENTS.md factual lines (never the owner-direction lines), docs/* vs the code (routes, limits such as the 100-item snapshot and 50-item pages, workflow names, env/bindings, scripts), one area per PR, in a run that also merges a code PR.
  P2 Saving-is-not-publishing invariant: a test matrix proving pending/held/removed contributions and unapproved issue photos never reach public HTML, counts, permalinks, queues, realtime messages, JSON-LD, /llms.txt, sitemaps or feeds; fix every leak with a regression test.
  P3 Local issues robustness: required-field validation before upload (only if #33 is closed unmerged), orphaned R2 objects after a failed DB write (a safe, reviewed cleanup path never deleting a referenced object; any production run is an owner decision), photo metadata stripping tests for every format.
  P4 Long-lived permalink refresh and history paging in the local demo.
  P5 Data lifecycle: participant-initiated deletion/retention — design note with options first (B11: in an existing decision record); additive schema slices; anything destructive in production is an owner decision.
  P6 Accessibility and ease of use: keyboard paths, focus, labels, error messages across rooms, composer, issues and review pages, with browser tests.
  P7 Security review of Origin/CSRF/cookie/CSP, rate limits and moderator-only routes, with tests for every finding.

### P5 PROJECT RULES
- Extra merge condition — schema: cloudflare/schema.ts changes are additive only, with Workers tests for the new D1 behaviour; a non-additive change, a new :once post-deploy script or any edit to an existing post-deploy script is NOT merged (prepare it, mark blocked with the owner decision).
- Never publish or archive a room, change moderator roles or post real contributions; tests use fixtures; never call the real TypeSafe model or Google. Never write to live D1/R2/KV, set or read secrets, or change Cloudflare, DNS or email routing.
- Keep AGENTS.md's contracts: published @purityjs npm releases with exact versions (framework bugs are fixed in KoalaFacts/Purity, not here); SQLite/D1 only; en-AU only; code-generated scene, no image/sprite/font binaries; Origin/CSRF/cookie/CSP checks; editorial prompts always labelled as the Press Floor desk; fair, non-partisan copy. Never change the owner's visual-acceptance items (reference restoration, host character, emblem) unless the roadmap row says the owner asked; never claim visual fidelity from tests.
- New scripts are TypeScript run with --experimental-transform-types as the repo does.

### P6 HOUSEKEEPING SCOPE
- README.md, AGENTS.md (factual lines only) and docs/** except docs/topics/ and docs/assets/. Superseded material is folded and deleted per B11. On Mondays do a deeper pass: read each doc against the code for one or two areas in rotation (recorded in the PR body) and merge duplicates.

### P7 REPORT EXTRAS
- Which browser suites ran; what the next owner deploy would ship (merged since the last deploy, D1 schema delta, post-deploy scripts).
