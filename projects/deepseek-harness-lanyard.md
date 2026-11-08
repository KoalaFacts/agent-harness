# Lanyard — project rules

Owner-controlled seed for the "Lanyard 推进计划" scheduled task (repo KoalaFacts/deepseek-harness-lanyard). The repo skill is `.claude/skills/advance-deepseek-harness-lanyard/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
no docs/roadmap.md or docs/README.md is created if missing. The ledger/work list and the Unreleased list go in a section of AGENTS.md or an existing docs/ page; the README pair is the index. Security invariants, owner decisions and threat-model points are kept records.

## Verification boundary
the real boundary is a real dsh (latest and next) running with the plugin, reached over the LAN bind with TLS from headless Chromium (the e2e and browser e2e), plus the mutation proofs in P3; verify-lanyard lists every guard with its mutation command. CLAUDE.md is a symlink to AGENTS.md: add the skills line to AGENTS.md.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/deepseek-harness-lanyard (public; npm package @koalafacts/deepseek-harness-lanyard): an out-of-tree `dsh` (DeepSeek Harness) plugin bundle, TypeScript/ESM, composed over the shipped harness through cordis.patch.yml, with exactly one purpose: using the dsh web GUI from a phone on a home network — explicit LAN bind, self-signed TLS, a gate holding every network request to upstream's own browser session, loopback/config pins, pairing QR, keep-awake.
- Goal: that one use case reliable and safe against every current dsh release line (latest and next), every guard proven by a mutation test, and docs (README.md + README.zh.md pair, AGENTS.md, docs/) always true, short and well organised. README.zh.md stays Chinese.
- Schedule: every 4 hours (6 runs a day), 50 minutes per run.
- SCOPE RULE (binding, from AGENTS.md): NOT a general-purpose gateway, reverse proxy or auth layer and must not grow into one. A change that does not serve someone opening the dsh GUI on their phone is out of scope. The plugin issues no credential of its own (authentication is upstream's), changes no harness source, and installing it changes nothing visible. Making the network the default, adding a credential, or widening what an unauthenticated LAN peer can reach are owner decisions. Deleting dead code or duplication with tests proving behaviour counts as progress.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `fix/`, `feat/`, `test/`, `docs/`, `ci/`, `deps/`, `refactor/`; squash merge. Report green Dependabot PRs instead of merging them.
- NO RELEASE: never dispatch or re-run release.yml or publish-npm.yml (not even dry_run), never edit them except for a proven bug with an independent agent review, never create tags or GitHub Releases, never change the version in package.json, never `npm publish`, never touch npm trust or the `production` environment. User-visible changes go in the Unreleased list (B11) and in both READMEs.
- Keep every action pinned by SHA.

### P3 SETUP AND GATE
- Read AGENTS.md (binding; CLAUDE.md is a symlink to it — edit AGENTS.md), README.md, README.zh.md, docs/, .github/workflows/*.yml, the source and tests for the area you touch.
- Toolchain: Node 22+ and pnpm 10 as CI uses (`npm i -g pnpm@10` or corepack); `pnpm install --frozen-lockfile`; Playwright Chromium for the browser e2e.
- LOCAL GATE (record ci.yml and nightly.yml state on main too; a red main or red nightly is fixed first):
  1. `npm run check` (typecheck, vitest, build + verify-build, verify:package).
  2. `npm run test:e2e` and `npm run test:e2e:browser`. If they exit 2 SKIPPED here (no LAN interface, no Playwright, publish wave), the hosted jobs are the evidence; say so. An upstream publish-wave SKIPPED may be re-run once later in the run.
  3. Mutation proof for every guard you add or change, and re-run neighbouring guards' mutations when a nearby rule changes (AGENTS.md "Verification"): break the guard, see the test go red for the intended assertion, restore. `npm run test:mutation:tls` when TLS code is touched.
  4. `node scripts/sync-dsh.ts` check: dsh-* devDependencies match what `dsh@latest` ships.

### P4 LEDGER AND ORDER
- Ledger: docs/roadmap.md only if it exists on main; otherwise per B11. It also keeps the scope rule and evidence (test names, mutation results, CI run links, dsh versions tested).
- Initial order:
  L0 Green main and nightly on both channels (latest and next).
  L1 Ledger + docs truth pass: README.md, README.zh.md and AGENTS.md against the code (supported dsh lines, --host rules on 0.2.0 vs 0.2.1, TLS/publicUrl behaviour, install steps, commands, payload inventory count); README pair in sync. The run that merges this also merges a code PR.
  L2 Upstream compatibility against dsh `next`: for each drift or structural change (startup provider, webserver carrier schema, pairing/connection injection, Remote surface RPCs such as terminal.attach) a failing test first, then the smallest fix keeping every security invariant; keep the peer window rule as AGENTS.md states it.
  L3 Remote-surface classification by enumeration: every RPC/route the current upstream exposes (from the installed packages, by grep, not memory) is classified, with a test that fails when upstream adds an unclassified one.
  L4 Mutation coverage beyond TLS: a TypeScript mutation harness (pattern of scripts/mutate-tls-recovery.ts) for the admission gate, the session check at every seat (register / registerUpgrade / registerFallback), loopback/config pins and payload allowlist; each surviving mutation gets a test that kills it.
  L5 Phone-use reliability within scope: pairing and reconnect after dsh restart, LAN certificate renewal across restarts, keep-awake behaviour, clear failure messages at load for misconfiguration — with tests and browser e2e where it applies.
  L6 Dependency upkeep between PRs: `pnpm outdated`; port breaking majors in a `deps/` PR preserving behaviour; keep the scoped js-yaml override rule from AGENTS.md (delete it only once nothing matches).
- Assert that something worked (a 200 with ok: true, a 101), not merely that the gate stayed out of the way. Self-review the diff before opening (AGENTS.md "Reviewing your own work": enumerate instances with grep, check prose against code). The independent review agent must read any security argument a change rests on.

### P5 PROJECT RULES
- Security invariants in AGENTS.md ("Load-bearing invariants", "Deferring authentication to upstream", "Client-owned paths are configuration") are never weakened; misconfiguration fails loud at load, never falls back to a weaker posture. No new runtime dependency without weighing its tree, stated in the PR. Config is an exported interface plus same-named Schemastery schema.
- Extra merge conditions: every hosted CI check on the head ran and passed (check, e2e, browser, security scans) — a SKIPPED e2e is not a pass; no mutation check, type-check strictness or payload allowlist entry weakened; no security posture loosened; no documented rule or invariant dropped.

### P6 HOUSEKEEPING SCOPE
- README.md and README.zh.md say the same things in the same structure (fix drift either way; the readme specs such as tests/readme-pairing.spec.ts stay green). AGENTS.md keeps rules and invariants; long incident narratives that only explain why a rule exists may shrink to one sentence of reason, but no rule, invariant, boundary statement, threat-model point or owner-decision note is lost or softened — list each removed or moved paragraph and where its content now lives. Delete docs or comments describing removed code after grep confirms it is gone. Housekeeping runs `npm run check`.

### P7 REPORT EXTRAS
- CI and nightly state, dsh latest/next versions tested, mutation results.
