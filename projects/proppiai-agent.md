# ProppiAI-Agent — project rules

Owner-controlled seed for the "ProppiAI-Agent 推进计划" scheduled task (repos ProppiAI/ProppiAI and ProppiAI/ProppiAI-Agent). The repo skill is `.claude/skills/advance-proppiai-agent/SKILL.md` in ProppiAI/ProppiAI; if it is missing, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

## What and why
- Develop the Proppi Agent autonomously and keep the public ProppiAI/ProppiAI-Agent repository in step with the main repository. The Agent is: the authenticated MCP service at https://agents.proppi.ai/api/mcp (also the approved, live ChatGPT app), its widgets, and the Proppi Agent plugin package (offline skills proppi-workflow and proppi-evidence-brief; desktop companion with proppi-account-workflow).
- Goal: a reliable, safe, easy-to-use assistant surface for self-managing landlords in New Zealand and Australia, where every tool does exactly what its description and the skills say, approvals and outcomes are honest, the docs are always true and tidy, and what customers install from ProppiAI-Agent matches the current main repository.
- Code, comments, test names, commit messages, PR text and docs are in English (en-NZ). Report in Chinese.

## Where the code lives (owner, 2026-11-05: "我们需要根据主仓来修改")
- ProppiAI/ProppiAI is the only source: apps/agent (MCP server, tools, widgets under src/server/mcp/widgets, Next app), packages/agent-core, distributions/proppi-agent (public/ = the package source: skills, references, README, CUSTOMER_GUIDE, CHANGELOG, verify.mjs; distribution.json, publication.json, RELEASE.md), scripts/proppi-agent (export, package, source checks, release gates, publisher, with tests), and the server-core/agent-core contracts the tools call.
- ProppiAI/ProppiAI-Agent is GENERATED output. Its default branch `generated-code` is what customers install. It changes ONLY by a verified export from a merged main commit (Public snapshot sync); never hand-edit a file there, never copy anything outside the export allowlist (no private history, business source, internal skills, customer data or credentials).
- Merging to main deploys production through Vercel (agents.proppi.ai included). GitHub Actions is billing-blocked: the local gate is the evidence; say so once in the report while it lasts.

## Git identity (owner, 2026-11-06: "desk@proppi.ai 不能作为 agent repo 的署名 因为 vercel 的 member 不在 只能用我的账号")
- Every commit this task creates, in BOTH repositories, has author AND committer `Leishi L <ldsenow@gmail.com>` — the only identity Vercel's team recognises (RELEASE.md requires it for snapshots too). This is the one owner-approved exception to autonomous-dev §9's no-names rule, and it covers commit metadata only.
- Set it per repository only (`git config user.name` / `user.email` inside each clone, or `git -c ...`), never `--global`. Before pushing, check `git log --format='%an <%ae> | %cn <%ce>' origin/<base>..HEAD` shows only that identity; amend your own unpushed commits if not.
- Never use desk@proppi.ai, agent@proppi.ai, a bot or any other identity. The owner's name never goes into file contents, commit message text, PR titles/bodies, comments, CHANGELOG or docs.

## Authority (owner, 2026-11-05: "自主合并发布 不要我介入只要符合条件")
- Main repo: squash-merge PRs this task opens on branches starting `agent/` when every merge condition holds.
- Public repo: update ProppiAI-Agent's `generated-code` branch with a verified export, as Public snapshot sync describes, without asking. This is the owner-authorised source-snapshot route of RELEASE.md, not a certified Release.
- Never merge or touch anyone else's PR or branch: the separate "Proppi 推进计划" task works in the same main repo on `proppi/` branches; never merge Dependabot (#731 or any dependency bump).
- Not available to this task: the formal Release path. Never publish a main-repo release `proppi-agent-v*`, never dispatch or re-run proppi-agent-release.yml, never run publish.ts with --execute, never create tags or GitHub Releases in either repo, never set the release-enablement variables, never change publication.json, never create or use App credentials. If a formal Release becomes worthwhile, prepare its evidence and say in the report exactly what the owner would have to enable.

## Versions and changelog
- User-visible package changes (skills, references, README, CUSTOMER_GUIDE, plugin/marketplace metadata, mcp.json) need a version entry: the export fails without a complete matching CHANGELOG.md entry. Batch them; before the sync, bump distributions/proppi-agent/distribution.json and the plugin/marketplace versions to the next semver (patch for corrections, minor for new guidance or capabilities) with a complete CHANGELOG entry (Added/Fixed/Breaking changes/Compatibility/Upgrade action, as 1.1.0 is written), in its own reviewed `agent/` PR, following RELEASE.md and the export/release-gate tests. A breaking change for installed customers is an owner decision. Never rewrite an existing version entry or reuse a version with different bytes.

## Setup and gate
- Add ProppiAI/ProppiAI with push access, clone it, `git fetch --depth=300 origin main`, branch from origin/main. Add ProppiAI/ProppiAI-Agent with push access and clone it separately. Set the git identity in both clones before the first commit.
- Read CLAUDE.md, AGENTS.md, .ai/rules/* and the .ai indices you need (CQRS factories, DI boundaries, branded IDs, no ON DELETE CASCADE, no dead compatibility, en-NZ, full country/state names in public copy, tests that pin outcomes); docs/product/one-stop-property-service.md and CLAUDE.md's positioning rules (never imply a service, channel, provider network or completion that does not exist; preparation, external execution and accepted completion are separate milestones; human approval and action receipts stay); docs/ROADMAP.md; distributions/proppi-agent/RELEASE.md (binding for the export) and public/*; ProppiAI-Agent's current release-manifest.json and last commits.
- `npm ci --no-audit --no-fund`.
- LOCAL GATE (baseline on main first): `node scripts/check-all.ts`, `npx vp check`, full `npm run test`, `npm run build -w @proppi/member`, the apps/agent build (and the MCP UI build), the scripts/proppi-agent tests, and `node scripts/proppi-agent/export.ts <sha> <new dir> --preview` + `node <dir>/verify.mjs`.
- Before committing, `git status`: never commit regenerated artifacts (apps/agent/src/server/mcp/widgets/widget-build.generated.json, packages/templates/html/**, .next) — `git restore` them. When a change really alters the MCP tool surface, regenerate the catalog version (`npm run agent:catalog-version`) and update proppi-account-workflow SKILL.md and references/mcp-workflows.md in the same PR.
- Verification boundary: the MCP server running locally with fakes/console providers, tools called through the MCP protocol, widgets rendered in headless Chromium, and the export verified with verify.mjs — never live business calls on a real account.

## Public snapshot sync (after a merge that changes anything the export includes, and at the start of a run if ProppiAI-Agent lags main in exported bytes)
1. Full SHA of the merged main commit (`git rev-parse origin/main` after `git pull --ff-only`).
2. `node scripts/proppi-agent/export.ts <FULL_SHA> <NEW_EMPTY_DIR>` (never --preview for a sync), then `node <DIR>/verify.mjs` and `node <DIR>/verify.mjs --install`; both must pass. Run `publish.ts <FULL_SHA> <DIR> <approval json> --dry-run` when it accepts pending gates; report its gates, never fake approval evidence.
3. In the ProppiAI-Agent clone, from origin/generated-code: replace the tracked tree with exactly the export's files (delete what the export no longer has; nothing else). If nothing but sourceSha would change, do not sync. Re-run both verify commands in the clone.
4. Commit on `sync/<short-sha>` with the git identity and the message pattern of earlier snapshots: "Update Agent <version> generated source snapshot", body "Source: ProppiAI/ProppiAI@<FULL_SHA> (PR #n…)", one line on what changed for customers, and the verified file counts.
5. Open a PR to `generated-code`, check its diff is exactly the export, and merge it yourself so the resulting commit keeps the git identity: prefer a fast-forward (`git push origin sync/<short-sha>:generated-code` once the PR is open and the branch fast-forwards), otherwise squash via the API and check the author; never force-push. Then confirm release-manifest.json's sourceSha equals FULL_SHA, the head author/committer is the git identity, and `node verify.mjs --install` passes on a fresh checkout. If anything fails, do not sync; fix the cause in an `agent/` PR first. If a sync breaks verification after merge, immediately sync again from the previous good export (a normal commit).
6. Record the sync in the ledger (public commit SHA, source SHA, version, files changed). A `sync/` PR does not count against one-PR-at-a-time.

## Ledger and work order
- Ledger: docs/proppi-agent-roadmap.md if it exists on main; otherwise the AI assistant / MCP section of docs/ROADMAP.md (no new docs file). It holds the goal, the ordered item table, the current public snapshot (version, source SHA, public SHA) and the user-visible changes since it.
- Order (adjust after reading the code; one reviewable change per PR; tests first):
  A0 Green main for the agent area: scripts/proppi-agent/mcp-guidance.test.ts fails on origin/main. Fix it only with a real guidance review (diff the tool catalog against proppi-account-workflow and mcp-workflows.md, correct every stale or missing statement, then record the reviewed version). Bumping the reviewed version without that review is forbidden.
  A1 Catch up the public snapshot: list every customer-visible difference between main's distribution (and the MCP tools it describes) and what ProppiAI-Agent ships; land A0 and the version/CHANGELOG PR; then the first sync. Check the export/publisher code and RELEASE.md agree with the git identity rule, with tests.
  A2 Docs truth pass: distributions/proppi-agent/public and the server-side tool descriptions checked against the actual tools, schemas, permissions and outcomes. The run that merges this also merges a code PR.
  A3 Tool inventory and contract tests: every registered MCP tool (grep, not memory) has a test pinning input schema, owner/account scoping, permission check and error shape; a test fails when a tool is added without one, and when the tool surface changes without the guidance being reviewed.
  A4 Approval and outcome honesty: preparation → explicit approval with the current review token/revision → recorded outcome; tests for stale tokens, replays, wrong owner and unknown outcomes; results and widgets never show prepared or approved as done.
  A5 Reliability: idempotency and abort tests, clear auth/expired-session errors, rate-limit and timeout behaviour, large-result paging.
  A6 Widgets: correctness, empty/error states, accessibility and en-NZ copy of every widget, with tests; reproducible widget build.
  A7 Offline skills quality against the product doc and NZ/AU practice (no invented legal rules; capability boundaries stated; hand-off to the connected workflow when an account is needed).
  A8 Export pipeline hardening: tests and mutation checks for source checks, manifest hashes, release-note scope and verify.mjs.
  Then the most valuable gap in correctness, safety or ease of use of the agent surface. Never build a capability the product does not have; prefer no migration, or an additive one.

## Docs cleanup
- The first run on Mondays and Thursdays (Sydney) starts with `agent/docs-cleanup-<date>`, following autonomous-dev §6. Scope: the agent ledger, distributions/proppi-agent/public/*, distributions/proppi-agent/RELEASE.md, agent design docs, the agent entries in .ai indices, and agent-related JSDoc/tool descriptions. Other Proppi docs belong to "Proppi 推进计划". When cleanup changes exported package files, follow Versions and changelog, then sync.

## After each merge
- `git checkout main && git pull --ff-only`, confirm the merge commit; read-only smoke check of the live service (e.g. an unauthenticated request to https://agents.proppi.ai/api/mcp showing the expected auth challenge) and a read-only Vercel MCP check that the deployment for the merge commit was not blocked or failed. If the merge broke production, open a revert PR at once and merge it under the same conditions. A Vercel check blocked by the commit author is a git-identity failure of your own commits: fix it, never work around Vercel.

## Extra merge conditions
- The only allowed gate failure is one that also fails on origin/main without the PR and is outside the area the PR touches.
- Migrations: none, or one already applied in production (check read-only with the Supabase MCP `list_migrations` on project svnegtetolkqtaodprde). Otherwise not merged: `blocked (owner decision: apply migration <file>)`.
- No catalog/guidance gate, export or release gate weakened.

## Project limits
- Production read-only: no Supabase writes or migrations, no Vercel changes, no OpenAI app/directory changes, no live MCP business calls on a real account, no live WhatsApp/email/Trade Me sends. Tests use fakes and console providers.
- No paid model, OCR, embedding, TTS or search APIs; no new paid dependency.
- The generated verify.mjs stays as the export defines it.

## Report extras
- Whether each merge reached production (Vercel deployment state) and the smoke check result.
- The public snapshot: synced or not, version, source SHA → public commit, what customers get; or why it still lags.
- Docs cleanup done this run, if any.
