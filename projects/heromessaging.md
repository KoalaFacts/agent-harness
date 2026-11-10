# HeroMessaging — project rules

Owner-controlled seed for the "HeroMessaging 推进计划" scheduled task (repo KoalaFacts/HeroMessaging). The repo skill is `.claude/skills/advance-heromessaging/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
the ledger is docs/plans/replacement-roadmap.md only if it already exists; otherwise a "Roadmap" section of docs/masstransit-replacement-acceptance.md. P5's "API change stated in an ADR" means a new section of the closest existing ADR in docs/adr/, not a new ADR file. P6's reorganisation creates no new folders (no docs/guides/, no docs/archive/): finished or superseded plans and one-off analyses are folded into the live page that owns any still-true content and then deleted; git mv only into folders that already exist. ADRs, acceptance evidence and retained benchmark results stay where they are.

## Verification boundary
the real boundary is real RabbitMQ and PostgreSQL (Docker locally, otherwise the hosted integration jobs) with two separate OS processes killed at declared points and the business ledger checked, as the acceptance doc requires; the feature map rows are the acceptance rows F01–F25 and P01–P05.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroMessaging (public). A .NET messaging library (net8.0/net9.0/net10.0): in-process commands/queries/publish, PostgreSQL and SQL Server storage (Outbox, Inbox, Saga), a RabbitMQ transport, serialization plugins, source generators, observability and a NativeAOT consumer.
- Goal: a credible MassTransit replacement for the workloads it claims — better performance, lower resource use, a friendly public API — proven scenario by scenario as docs/masstransit-replacement-acceptance.md requires.
- Schedule: standard tier — once a day, 50 minutes per run.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `fix/`, `feat/`, `test/`, `perf/`, `docs/`, `ci/`; squash merge. Never touch codex/ branches (e.g. #118), Dependabot or other sessions' work.
- Never dispatch or re-run create-release.yml, publish-nuget.yml or any publish workflow. CHANGELOG.md [Unreleased].

### P3 SETUP AND GATE
- Read CLAUDE.md (binding), CONTRIBUTING.md, docs/README.md, docs/masstransit-replacement-acceptance.md (inventory F01–F25, P01–P05), docs/superpowers/specs/2026-10-24-competitive-foundation-design.md and plans/, docs/adr/, docs/audit/, docs/testing-guide.md, the package README of the area.
- SDKs: dotnet-install.sh with the SDK in global.json (`--channel 10.0`) plus `--runtime dotnet --channel 8.0` and `9.0`.
- LOCAL GATE: `dotnet build HeroMessaging.slnx -c Release` with warnings as errors (also NativeAotConsumer and any project CI builds outside the solution); unit tests (`--filter-not-trait Category=Integration`) for touched projects and core HeroMessaging.Tests on net8.0/net9.0/net10.0; integration tests (PostgreSQL, SQL Server, RabbitMQ) locally when Docker is available, otherwise the hosted run-integration-tests / Build and Test jobs are the evidence; publish and run NativeAotConsumer when dispatch, generators or trimming-relevant code change; benchmarks only for perf claims via the existing workflows or CLAUDE.md's same-harness A/B; the docs checker.
- RED MAIN FIRST: the "Build and Test" workflow on main was failing on 2026-11-05.

### P4 LEDGER AND ORDER
- Ledger: docs/plans/replacement-roadmap.md; every row names the acceptance rows it moves (F../P..), and every PR also updates that row in docs/masstransit-replacement-acceptance.md (only its status vocabulary; "Passed" needs a named scenario, exact configuration, executable verification and retained result).
- Initial order (the acceptance doc's first route: RabbitMQ + PostgreSQL, two independent processes):
  R0 Green main.
  D0 First housekeeping as a full docs reorganisation (see P6), right after R0.
  R1 F09 consumer retries/redelivery (attempt limit and backoff govern RabbitMQ direct-consumer failure; terminal disposition and retry state across redelivery).
  R2 F10 dead letters and replay (retained destination with failure context, safe replay, operator docs).
  R3 F06 PostgreSQL Outbox (commit/rollback with business state, competing workers, lease expiry/fencing, process death around broker confirmation).
  R4 F07 transactional external publish on top of R3.
  R5 F08 durable Inbox acceptance (persist-before-ACK, duplicates, multi-host ownership, crash recovery, retention/poison policy).
  R6 Two-process reliability harness (separate OS processes killed at declared points, business ledger) running R3–R5 evidence in hosted CI.
  R7 F24 consumer testing API; R8 F16 remote request/response; R9 F17 persistent scheduling; R10 F14 Saga state plus outgoing work; R11 F12 SQL Server external Outbox.
  R12 P01–P05 equivalent-work benchmarks against MassTransit, same machine and workflow, with resource use; negative results recorded.

### P5 PROJECT RULES
- Do not edit LICENSE. Preserve existing public signatures unless the item is an API change stated in an ADR. Fix warnings in code rather than suppressing them. Extra merge condition: hosted CodeQL/security scans that ran passed; no coverage or performance gate weakened.

### P6 HOUSEKEEPING SCOPE
- All Markdown (README.md, CLAUDE.md, CONTRIBUTING.md, docs/**, package READMEs, tests/README.md). The first pass reorganises: finished or superseded plans and one-off analyses (candidates: RINGBUFFER_IMPLEMENTATION_PLAN.md, CSharp13-Improvements.md, code-quality-improvement-plan.md, idempotency-implementation-plan.md, rate-limiting-implementation-plan.md, production-readiness-analysis.md, dated audits) go to docs/archive/ with a header naming what superseded them; live guides into a clear structure (docs/guides/, docs/adr/, docs/plans/, docs/audit/, docs/archive/) with git mv and every inbound link fixed; stale facts fixed (CLAUDE.md's "Last updated 2025-11-07" and "C# 12" lines, commands, project lists, frameworks, workflows). Never delete or rewrite ADRs, acceptance evidence, retained benchmark results or CHANGELOG history; negative results stay findable.

### P7 REPORT EXTRAS
- Acceptance rows moved by each merged PR; test counts per framework.
