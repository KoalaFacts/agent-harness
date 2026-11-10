# ai-office — project rules

Owner-controlled seed for the "ai-office 推进计划" scheduled task (repo KoalaFacts/ai-office). The repo skill is `.claude/skills/advance-ai-office/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
docs/ROADMAP.md is the ledger. docs/PROGRESS.md and docs/OFFICE-TEMPLATES.md are not created if missing (run notes go in PR bodies; if docs/PROGRESS.md already exists, it is only compacted, never grown, and may be folded into ROADMAP.md and deleted). Template catalogue and setup-flow docs go in a section of docs/ARCHITECTURE.md or docs/FEATURE-STATUS.md; design notes for blocked steps go in the roadmap step text. Template data files (JSON/YAML and their bilingual strings) are product data, not docs.

## Verification boundary
the install branch is `office/verify-skills`. The real boundary is the built app running locally with a local disposable D1/R2 (never remote), driven in headless Chromium through the template quick-setup flow in zh-CN and en — pick an industry template, answer the questions, see the configured office render — plus owner/workspace isolation checked with two local owners.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/ai-office: a pixel-office app — Next/Vinext on Cloudflare Workers with D1/R2, schema in schema/office-v1.json, services in server/, browser controllers in public/, node:test tests in tests/.
- PRODUCT GOAL (owner, 2026-11-05): AI Office lets any business owner quickly stand up the office they need. We provide industry-optimised office templates; the owner answers a few questions and gets a working office configured for their business. Every step serves that goal; when a choice is open, pick what makes template-based quick setup easier and safer.
- Work follows docs/ROADMAP.md (goal, milestones M0–M6, steps, acceptance criteria, Status table): the first step not done, in milestone and step order, skipping only `blocked (owner decision)` steps. A complete spec the next step can be built from counts as a spec step's deliverable.

### P2 AUTHORITY SPECIFICS
- Branch prefix `office/` (step branches `office/<milestone-step-slug>`; housekeeping `office/docs-tidy-YYYY-MM-DD`); merge method **merge** (merge commit): `-f merge_method=merge -f sha=<head>`. Only `office/` PRs in KoalaFacts/ai-office.
- Codex review wait is 30 minutes here (Codex summary comment showing Completed), then the independent agent review (which also checks owner/workspace isolation).
- Owner decisions are only: deploying or new infrastructure/hosts/domains/identity providers, running anything against live data, credentials, external execution or provider connections, opening sign-up to other owners, and new art that would cost money. For those: do the preparatory work that needs no decision (options and a recommendation in the roadmap step, interfaces, tests against a fake), mark `blocked (owner decision)`, merge that, continue.
- Never deploy, never run wrangler against a remote, never touch live D1/R2 data, never apply a migration anywhere but a local disposable database. Never create or configure credentials, hosts, domains, identity providers, paid infrastructure, provider/assistant connections or external dispatch; no automated external execution.

### P3 SETUP AND GATE
- Read docs/ROADMAP.md, docs/FEATURE-STATUS.md, README.md, docs/ARCHITECTURE.md, docs/SECURITY.md, docs/ENGINEERING-ACCEPTANCE.md, docs/PROGRESS.md if it exists, any CLAUDE.md/AGENTS.md.
- `npm ci --no-audit` (never run `npm audit`). LOCAL GATE: `node --test tests/*.test.mjs`, `./node_modules/.bin/tsc --noEmit --incremental false`, `npm run build`, `npm run lint` (0 errors), `npm run docs:check` once it exists.

### P4 LEDGER AND WORK RULES
- Ledger: docs/ROADMAP.md (Status table + step text; mark `done (PR #n)` only when its acceptance criteria are met; split a step too big for one reviewable PR — more than a few hundred lines or more than one concern — into numbered sub-steps in the same PR as the first sub-step). docs/FEATURE-STATUS.md records what is delivered, with test evidence, and what remains unverified. Docs never claim production, deployment or browser acceptance you did not do.
- Keep existing contracts: schema versions, additive-only migrations under drizzle/, audit/outbox atomicity, owner/workspace isolation, permission records never granting execution. Templates and setup text are bilingual (zh-CN and en).
- Templates reuse the existing pixel sprites: no image or character-art generation.

### P5 PROJECT RULES
- Never commit real credentials, local state, runtime profiles, build outputs or dependencies; public/status.json stays sanitised sample data.
- Housekeeping never deletes code, tests, migrations, schema files, assets, licence files or asset provenance; licences and provenance stay with their assets.
- Extra merge condition: only roadmap or housekeeping work.

### P6 HOUSEKEEPING SCOPE
- Every Markdown and text file outside node_modules, checked against the code: fix stale commands, paths, schema names, feature status and counts; fold obsolete or duplicated docs into their owner and delete them; fix broken relative links. ROADMAP.md: done steps collapse to one line with their PR; no stale "in progress" markers. The first housekeeping PR adds scripts/check-docs.ts (TypeScript, Node) with a test as `npm run docs:check` in the gate (and in CI once CI exists).

### P7 REPORT EXTRAS
- The roadmap status line M0–M6 after this run; owner decisions each with your recommendation.
