# Purity — project rules

Owner-controlled seed for the "Purity 推进计划" scheduled task (repo KoalaFacts/Purity). The repo skill is `.claude/skills/advance-purity/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
docs/plans/surpass-solid.md is the ledger (its "Last docs tidy" line is the housekeeping date). The audit, gap and roadmap docs (docs/code-quality-audit-2026-11-05.md, docs/solid-v2-gap-analysis.md, docs/solid-v2-gaps.json, docs/solid-competitor-roadmap.md, docs/framework-capabilities.md) are edited in place; a finished one is folded into the ledger or framework-capabilities.md and deleted. Handoff notes such as NEXT_HANDOFF.md are folded and deleted. One explicit exception: each SV2 capability may get one new numbered ADR in docs/decisions/ (the repo's existing decision-record convention, needed for API semantics), and only one per capability; everything else about it goes in that ADR or the ledger. benchmark/history and recorded benchmark runs are data.

## Verification boundary
the verifier verdict is separate from P5's Codex review rule and does not replace it. The real boundary is a built app (CLI scaffold or the benchmark pages) running in headless Chromium — JIT and strict AOT, SSR then hydration, DOM mutation counts — and, for performance, benchmark/run-bench.ts against Solid as P4 describes; the skills line goes in the root CLAUDE.md (keep AGENTS.md/GEMINI.md pointing to it).

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/Purity (PurityJS). Goal: SURPASS SolidJS v2 — clearly differentiated (Web Components + Shadow DOM, direct DOM templates with no-build JIT and CSP-safe strict AOT, zero runtime dependencies, modular entries, non-lossy SSR/hydration/islands), very easy to use and excellent in performance.
- ADRs are in English.
- Equivalent deliverable for B2: merged code, tests, measured improvements or docs that change what a user can do.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `perf/`, `feat/`, `fix/`, `test/`, `docs/` opened by this task, plus the PR for branch `audit/code-quality-2026-11-05` (the audit this plan starts from; #346 was closed and a successor PR from the same branch carries it); squash merge. Housekeeping branch: `docs/tidy-YYYY-MM-DD`. Other sessions and Dependabot work here.
- Publishing to npm stays the owner's: never run or dispatch the publish workflow, never bump versions for a release, never tag or create Releases.

### P3 SETUP AND GATE
- Node 24 and npm 11 (if older, `npm i node@24 npm@11` into a scratch folder first on PATH). Read CLAUDE.md and the CLAUDE.md/README of every package you touch, the ledger, the audit doc (on main once the audit PR merges, otherwise on its branch), the solid-v2 gap docs, docs/framework-capabilities.md and the ADRs in docs/decisions/ for the area. CLAUDE.md's package table and paths may be stale until CQ-10 lands — trust the tree.
- LOCAL GATE: `npm ci`, `npm run check`, `npm run build:packages`, `npm test` (all workspaces), `npm run check:bundle`; plus the browser/memory scripts relevant to what you touched (test:browser:*, test:ssr:memory, test:inspector:memory) when Chromium is available; scripts/check-docs.ts (`node --experimental-strip-types`) once it exists, added to `npm run check` once it passes on main.

### P4 LEDGER AND ORDER
- Ledger: docs/plans/surpass-solid.md (id, what, why it beats or matches Solid, status, evidence: test names, benchmark numbers with environment).
- Order (one PR each unless trivially small):
  1. Audit items in the audit doc's section 4 order: CQ-06+07+10 (compiler alias in vitest configs, de-duplicate tests, fix both CLAUDE.md files, record the rc.20 comparator change in the solid-v2 target/gap docs) → CQ-04+03 (swapSlot text fast path keeping undefined/false → empty; per-binding last-value memo initialised to a private sentinel; MutationObserver mutation-count tests per binding kind; input focus/selection preservation test) → CQ-01+02 (each(): keys array in EachState, head/tail trimming, LIS on the changed window only, incremental owner update) → CQ-05 (shared accessor methods; reduce row-ownership churn while keeping registrations for dynamic row roots — root-dynamic-unmount.test.ts must stay green) → CQ-09 (type-check tests for every package in CI, typed Vite Plugin surface) → CQ-08, CQ-11–13.
  2. Then the SV2 capability rows in the roadmap's order: SV2-003 async derivation, SV2-006 error/reset boundaries, SV2-004 client loading/retained content, SV2-005 property-granular stores, SV2-008 current keyed index, SV2-007 async transactions/optimistic, SV2-009/010 independent streaming and early interactivity, SV2-017 owner composition, SV2-013 diagnostics. Each starts with its ADR (B11 exception: API, semantics, intentional differences from Solid, acceptance tests); then implementation slices, each a mergeable PR with value on its own.
  3. Ease of use with every capability: typed public API, README/docs examples, CLI scaffold update when it changes how apps are written, clear dev-mode errors. Fewer, composable APIs.
  4. Performance evidence: for every perf PR run benchmark/run-bench.ts for purity and solid on the affected pages (CHROMIUM_PATH if Playwright's browser is absent; warmup 3, iterations ≥5) before and after; record medians and the raw run file in the PR and ledger. Re-run when a difference is under ~10%; never claim a win from one noisy run.
- Tests first; cover error/disposal paths; keep generate / generateSSR / generateHydrate in lockstep; review your own diff adversarially BEFORE opening, because Codex reviews only the opening commit.

### P5 PROJECT RULES (stricter than B6.5 and B8.3)
- Review: Codex (chatgpt-codex-connector) reviews only the commit a PR was opened with. Its summary for the opening commit must show Completed (wait up to 30 minutes after opening; if it never completes, the PR waits for the next run — no independent-agent substitute here), and every finding it raised plus every other review comment is fixed in a later commit with its thread resolved, or answered with a traced reason and resolved. A re-review of later commits is not required. Never close and reopen a PR or push empty commits to get Codex or CI to run again.
- Every hosted CI check on the head ran and passed (a job cancelled by runner infrastructure may be re-run once; a second failure is real).
- Bundle budgets: never raise an existing budget to pass; a new capability that needs bytes goes into an opt-in entry with its own new budget stated in the PR; unavoidable growth of an existing entry is an owner decision.
- Removing a genuinely duplicated test is fine when the PR names the surviving copy.
- Stuck rule: if this task's open PR is still not mergeable at the start of its THIRD run, or the same failure survived two fix attempts, shrink it to the green mergeable part (split the rest) or close it with a comment, mark the ledger item blocked with what would unblock it, and move on. If hosted CI or Codex is unavailable for more than a day, say so once and keep the single PR ready.
- Benchmark claims name the exact Solid version, Purity commit, browser and machine; never rewrite benchmark/history.

### P6 HOUSEKEEPING SCOPE
- Every *.md: broken relative links and anchors, references to files/exports that no longer exist, ADR numbering/index gaps (the checker finds these); stale facts against the tree and package manifests (package lists and paths, versions, test/ADR/scenario counts, commands, comparator versions), preferring wording that does not go stale (link to the manifest). Consolidate overlapping or superseded docs into the current one and delete the rest (B11). Never delete or renumber ADRs (mark superseded ones in their status line). Agent-facing files (root and package CLAUDE.md, AGENTS.md, GEMINI.md) match the tree exactly, stay short and point to the canonical doc. Update the ledger's "Last docs tidy" line.

### P7 REPORT EXTRAS
- Bundle and benchmark deltas for each merged PR; items marked blocked and what would unblock them.
