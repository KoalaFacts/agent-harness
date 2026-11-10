# KoalaFactsMarketing — project rules

Owner-controlled seed for the "KoalaFactsMarketing 推进计划" scheduled task (repo KoalaFacts/KoalaFactsMarketing). The repo skill is `.claude/skills/advance-koalafactsmarketing/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
no docs/ROADMAP.md, docs/README.md or docs/database-proposal.md is created. The ledger/work list is a section of docs/STATUS.md; the database proposal (D1) is a section of docs/backend-architecture.md if present, otherwise docs/cloudflare-migration.md; the README is the docs index. docs/content-facts.json is data.

## Verification boundary
the real boundary is the built dist served locally and driven in headless Chromium at 390px and 1280px (pages, keyboard flow, no-JavaScript paths), plus the worker's enquiry routes exercised locally with D1 mocked and the wrangler dry run. The DATABASE BOUNDARY in P2 still holds: verification never executes SQL.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/KoalaFactsMarketing (private): the public site for a physical Koala Facts bakery planned for Chatswood in April 2027 (business events, catering, birthdays, anniversaries, special moments), intended for koalafacts.com.au on Cloudflare Workers + Static Assets, with a guarded enquiry API (D1 + Turnstile, disabled by default). Source in site/ and worker/, generated dist/, native Node 24 .ts scripts and node:test tests.
- Goal: a launch-ready, accurate, accessible, fast website whose code lands on main in small reviewed steps and whose docs say exactly what is true.
- Schedule: standard tier — twice a day, 50 minutes per run.
- Why this task exists: earlier sessions spent two weeks adding status comments to Draft PR #4 (feat/backend-stack-alignment, ~89 files, ~13k lines) without merging anything. Do NOT add more status comments to PR #4 and do NOT grow it; land its useful work on main in small PRs (M1).

### P2 AUTHORITY SPECIFICS
- Branch prefixes `site/`, `fix/`, `test/`, `docs/`, `refactor/`, `ci/`; merge method **merge** (merge commit, as PRs #3 and #5): `-f merge_method=merge -f sha=<head>`. Review wait for a bot is 30 minutes here, then the independent agent review (which also checks enquiry-API safety, accessibility and that no copy invents a fact).
- PR #4 is the owner's own draft; the owner said on it (2026-10-24) to keep it unmerged and to get specific approval before database schema/migration work. Never merge PR #4 or push to its branch. You MAY re-implement or cherry-pick its non-database parts into your own small PRs from current main, crediting the PR #4 commits in each PR body.
- Merging to main does not deploy (no Worker or Workers Builds exists in the Cloudflare account today). If a Workers Builds connection appears, stop merging site/worker changes, report it, and treat merges as production deploys needing the owner's go-ahead.
- DATABASE BOUNDARY (binding until the owner approves otherwise in writing on a PR or in docs/STATUS.md): do not write, change or execute database schema or migrations (migrations/, cloudflare/schema.ts or any Drizzle schema, D1 tables), and do not run tests that execute SQL against a database, not even locally. Existing request-behaviour tests that mock D1 are fine.
- Never run wrangler against a remote (only --dry-run and local); never create or change Cloudflare Workers, Workers Builds, D1, Turnstile, DNS, domains, certificates or GoDaddy records; never set or read secrets; no sign-in attempts. Marketing operations belong in KoalaFacts/project-growth; do not work there.

### P3 SETUP AND GATE
- Read AGENTS.md (binding: facts and content rules, no invented details, no non-functional forms, accessibility, source/output separation), README.md, docs/STATUS.md, docs/design.md, docs/content-facts.json, docs/asset-provenance.md, docs/cloudflare-migration.md, docs/event-website-plan.md, docs/backend-architecture.md if present, PR #4's body and last commits.
- Node 24 (.node-version): if older, `npm i node@24` into a scratch folder first on PATH.
- LOCAL GATE: `npm ci --ignore-scripts`, `npm run typecheck`, `npm test`, `npm run build` (includes scripts/verify.ts), `npx wrangler deploy --env '' --dry-run --outdir .wrangler/dry-run` with WRANGLER_SEND_METRICS=false, check-docs once it exists. For site/ changes also serve dist locally and check the changed pages in headless Chromium (Playwright via npx into a scratch folder) at 390px and 1280px: no horizontal overflow, keyboard focus visible, works without JavaScript where AGENTS.md requires; record what you checked. Never commit dist/, node_modules/, .wrangler/ or screenshots unless a doc deliberately references one.

### P4 LEDGER AND ORDER
- Ledger: a "Roadmap" section of docs/STATUS.md. Every PR updates its row and docs/STATUS.md's current state where it changes.
- Initial order (one reviewable change per PR, a few hundred lines of real change at most):
  R0 Ledger + gate truth: the Roadmap section from STATUS.md, the plan docs and PR #4; STATUS.md refreshed to the real state (main SHA, test counts from your gate run, PR #4 situation). The run that merges R0 also merges a code PR.
  M1 Land PR #4's non-database work in slices from current main, each with tests: (a) Zod validation at the HTTP boundary (worker/enquiries/validation.ts + pure tests) preserving the public contract, zod exact-pinned (AGENTS.md's zero-runtime-dependencies rule is superseded only as far as PR #4's AGENTS.md change says; update AGENTS.md in the same PR); (b) typed command/query handler structure for the enquiry feature without schema changes; (c) offline event-brief download/restore and no-JavaScript fixes; (d) canonical .html routes and linked-asset checks; (e) other site/ fixes and tests from PR #4. Re-verify each against current main; drop what main already has. When all non-database work is on main, leave one short comment on PR #4 listing the merged PRs and what remains (database only), once.
  D1 Database proposal (docs only, no execution, in the section B11 names): exact Drizzle schema, migration SQL, compatibility with migrations/0001_enquiries.sql and existing rows, test plan (local disposable D1 only), rollback, and "what the owner approves". Mark D1 `blocked (owner decision: approve the database proposal section)`; build nothing on it until approved.
  S1 Content and SEO correctness: every page's title, description, canonical, Open Graph, structured data (LocalBusiness/Bakery only with confirmed facts — no address, hours, prices or opening day that docs/content-facts.json does not confirm), sitemap/robots consistent with preview noindex; a test that fails if a page lacks them or states an unconfirmed fact.
  A1 Accessibility: automated axe-core over every built page at mobile and desktop widths, fixing every violation; keyboard flow of the enquiry form and occasion selectors pinned by a test.
  P1 Performance: image sizes/formats, font loading, CSS/JS weight, cache headers in site/_headers; reproducible local before/after numbers; never claim a gain from one noisy run.
  U1 Customer journeys: clearer event/catering/celebration pages and FAQs using only confirmed facts and honest placeholders ("menu to be announced").

### P5 PROJECT RULES
- Never invent business facts (address, hours, menu, prices, opening day, reviews, awards, sourcing, dietary guarantees, photos). Never add a non-functional form, newsletter, booking or ordering flow. An enquiry is never presented as a confirmed booking. Never send messages to third parties.
- Extra merge conditions: the PR crosses no DATABASE BOUNDARY and states no unconfirmed business fact; mergeable_state may be "unstable" only because of billing-blocked checks.

### P6 HOUSEKEEPING SCOPE
- README.md, AGENTS.md (only to correct something now wrong, never to weaken a rule) and docs/**: consolidate overlapping plans into STATUS.md or the owning page and delete the rest after folding; keep docs/STATUS.md to the current state plus a short one-line-per-entry history; remove superseded verification material no current doc cites (e.g. the opening-only verification.json and its 20261025 previews) after checking references. docs/content-facts.json and asset-provenance.md must match what site/ actually shows; a mismatch is a bug fixed in the same PR (or a fix/ PR next if it changes the site). Never delete owner instructions, approval/decision records, licences, provenance still relied on, or evidence a current doc cites.

### P7 REPORT EXTRAS
- How much of PR #4's non-database work is now on main; owner decisions such as approving the database proposal, a Cloudflare connection, or confirmed business facts.
