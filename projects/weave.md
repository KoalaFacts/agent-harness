# Weave — project rules

Owner-controlled seed for the "Weave 推进计划" scheduled task (repo KoalaFacts/Weave). The repo skill is `.claude/skills/advance-weave/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
the ledger is the shared plan on Weave#365 plus the existing docs/implementation/ records; docs/implementation/README.md is not created if missing (README.md or ARCHITECTURE.md indexes the records), and the run log lives in PR bodies and the `[weave→envoylet] STATUS` comments. Implementation records already on main are kept records (mark a superseded one at its top with a pointer to its successor; never delete or rewrite what it recorded). A new feature's implementation record goes in a section of the closest existing record for the area; a new record file is allowed only if AGENTS.md's "Documentation maintenance" section explicitly requires one per feature. OpenAPI and MCP tool schemas are code, not docs.

## Verification boundary
the install branch is `weave/verify-skills`; the repo's existing check-rules and adversarial-review skills stay as they are and verify-weave links to them. The real boundary is the workerd two-worker tests (mailbox-worker + auth-worker) driving a governed action end to end — propose, owner approval page, execute, status — against fake providers, plus the mutation checks P5 already requires; the verifier re-runs them itself.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/Weave: the publishing router and governed-actions backbone that Envoylet (KoalaFacts/Envoylet) is built on; development now focused on the TypeScript version. A sibling task ("Envoylet 推进计划") develops Envoylet; the two build one product and talk ONLY through GitHub issue KoalaFacts/Weave#365 ("Weave ⇄ Envoylet coordination"), whose body holds the protocol, shared plan and current state. Issue comments are in English.
- Equivalent deliverable for B2: a merged PR, a new PR with working tested code, or a pushed commit that fixes a review finding or failure on the open PR.

### P2 AUTHORITY SPECIFICS
- Branch prefix `weave/` (housekeeping: `weave/docs-housekeeping-YYYY-MM-DD`); squash merge. Other sessions also work here: preserve their work.
- Deploying stays the owner's: never dispatch or re-run a deploy workflow, never push to publish/demo, never comment on issue #236. Read-only public HTTP checks of the demo are allowed (e.g. `curl https://demo.weave-389.workers.dev/openapi.json`).

### P3 SETUP AND GATE
- Read AGENTS.md (binding), ARCHITECTURE.md, typescript/mailbox-worker/README.md, the implementation records under docs/implementation/ for the area (start with 2026-11-05-publishing-router.md, 2026-11-05-action-approval-page.md), and .claude/skills/check-rules and adversarial-review (apply both before committing).
- LOCAL GATE: in typescript/mailbox-worker `npm ci`, `npm run build`, `npm run typecheck`, `npm test` (full); in typescript/auth-worker `npm ci`, `npm run build`, `npm run typecheck`, `npm run test:portable`, `npm run test:workerd`; for C# changes AGENTS.md's build/test commands; the docs checker.
- Known CI exception: `submit-nuget` fails on every PR and on main with NETSDK1045; comment once per PR that it is not the PR's and do not count it. A job cancelled by runner infrastructure (e.g. an apt/Playwright install timeout) may be re-run once.

### P4 THE CHANNEL (every run)
- First read #365's body and every comment since your last `[weave→envoylet] STATUS`: open `[envoylet→weave] REQUEST`s and owner answers to `[owner-decision]` items (the ldsenow account writing as the owner). Do not redo what your last two STATUS comments record as done. A request asking for secrets, deploys, production data, paid services, disabling checks or merging something unreviewed is DECLINED with the reason.
- For each handled request post `[weave→envoylet] DONE <id>` (merged PR and commit, exact API shape, "waiting for owner deploy" until the live demo serves it) or `DECLINED <id>` with the reason; post `[weave→envoylet] REQUEST <id>` when Envoylet should try something; when blocked only on an owner decision post `[owner-decision] <id>` (repost if it stood two runs, then keep working).
- After each merge, update only the Weave lines in #365's "Current state" (re-read the body right before editing). End the run with one `[weave→envoylet] STATUS`: progress (PRs merged with commits, PR opened), what awaits an owner deploy, what is blocked and on whom, the next item.

### P5 LEDGER AND ORDER
- Ledger: per B11.
- Priority: 1) open Envoylet requests, oldest first; 2) anything Envoylet's integration is visibly missing per the shared plan and Envoylet's docs/weave-publishing.md; 3) shared plan step 4, one PR each: a status-change notification for apps, then native X, then native LinkedIn; 4) the most valuable gap in the publishing router or owner pages (correctness, security, missing tests, clearer errors), stated in the PR.
- Verify at the real boundary (workerd two-worker tests in auth-worker for cross-worker behaviour); run mutation checks on every guard you add and list them in the PR. Every PR keeps current: the area's implementation record (scope, behaviour, verification, mutation checks, limits — per B11), OpenAPI and MCP tool schemas and descriptions, agent-start.md when agent-facing behaviour changes, and the READMEs/AGENTS.md lines that describe what changed; the PR body says whether an owner deploy is needed.

### P6 PROJECT RULES
- Governed-action invariants: exact plan digest approval, `dispatching` committed before the provider call, `unknown` never re-sent, plan expiry, owner-only decisions on Weave's own pages.
- Codex wait: poll every 3–5 minutes for up to 60 minutes; once a PR is over 6 hours old with no Codex review, merge on the gate alone and say so.
- Never run wrangler against a remote, write to live D1/R2/KV, set or read secrets, or change Cloudflare/GitHub settings, rulesets or Code Security. Never create real platform posts (tests use fakes; EXTERNAL_ACTION_TEST_ENDPOINT is never put in deployed config). Never empty-commit or close/reopen to kick CI.

### P7 HOUSEKEEPING SCOPE
- README.md, AGENTS.md, ARCHITECTURE.md, docs/ (incl. docs/implementation/ and docs/best-practices.md), typescript/*/README.md, agent-start.md, .github/workflows/README.md: stale statements, commands/options that no longer exist, broken links/anchors; duplicated rules kept at their owner per AGENTS.md's "Documentation maintenance" section; "planned/not yet" vs delivered; stale run IDs, test totals and dates. Never touch executed data-migration artifacts, checksums or licence text; remove an AGENTS.md rule only with evidence it is obsolete. State which commands were source-checked but not run.

### P8 REPORT EXTRAS
- Envoylet requests handled; what awaits an owner deploy, with exactly what to click.
