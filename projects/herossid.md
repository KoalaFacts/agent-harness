# HeroSSID — project rules

Owner-controlled seed for the "HeroSSID 推进计划" scheduled task (repo KoalaFacts/HeroSSID). The repo skill is `.claude/skills/advance-herossid/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
no docs/ROADMAP.md, docs/README.md or docs/CHANGELOG.md is created if missing. The ledger and each VISION milestone's done/remaining status go in a section of the existing docs/VISION.md; README.md is the docs index (its capability table shortened to one line per area as D1 says); Unreleased goes in a README section if no changelog exists. P6's "short redirect line at old paths" is dropped: consolidated profile docs are deleted outright after their content is folded in and every inbound doc link and code comment is updated. S1/X1/K1/R1 designs go in a section of the existing docs/superpowers spec for the area, not a new spec file.

## Verification boundary
the real boundary is the API host running locally (PostgreSQL via Docker/Testcontainers when available) driven through a full credential flow over HTTP — issue, hold, present, verify, revoke — and, for exchange protocols, an independent tool or wallet library actually run against it; the feature map is grouped by VISION milestone.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroSSID (public), .NET 10 / C#. Mission (docs/VISION.md): a trustworthy, portable identity-credential foundation for people, organisations and agents, judged by verified credential flows and independent interoperability. VISION milestones: trust foundation, operable lifecycle, independent exchange, privacy and holder control, pilot readiness.
- Schedule: standard tier — twice a day, 50 minutes per run.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feat/`, `fix/`, `test/`, `docs/`, `refactor/`, `ci/`; squash merge. Never dispatch publish-nuget.yml or any publish workflow. User-visible changes in docs/CHANGELOG.md "Unreleased" (create it if missing).

### P3 SETUP AND GATE
- Read docs/VISION.md, TESTING.md, docs/testing-guide.md, docs/superpowers/ (specs and plans), .github/workflows/ci.yml.
- .NET 10 SDK via dotnet-install.sh `--channel 10.0` into a scratch folder; Node 24 for the docs checker.
- LOCAL GATE: `dotnet restore`, `dotnet build HeroSSID.sln -c Release` (warnings stay errors where the repo sets it), every test project ci.yml runs (unit, API, credentials, data, DID operations, contract), the docs checker. Integration tests (Docker/Testcontainers PostgreSQL) run if Docker is available; otherwise the hosted CI job is the evidence.

### P4 LEDGER AND ORDER
- Ledger: docs/ROADMAP.md, which also holds each VISION milestone with what is done (evidence: tests, PRs) and what remains; every work row names the milestone gap it closes.
- ANTI-SPRAWL RULE: the repo already has dozens of narrow "opt-in profile" variants (docs/ ~60 files, ~550 KB; a very long README capability table). Do NOT add another narrow variant, pin, cutoff or HTTP wrapper of an existing flow unless it closes a concrete VISION milestone gap, named in the PR. Consolidation that removes duplication (code or docs) with tests proving behaviour unchanged counts as progress.
- Initial order:
  D1 First housekeeping + roadmap: docs/ROADMAP.md from the code and VISION; the docs checker (also flags code identifiers in backticks that no longer exist in src/); TESTING.md counts from a real gate run (it claims 174); docs/README.md index grouped by VISION milestone; README capability table shortened to one line per area linking to details, losing no boundary statement.
  S1 Real selective disclosure: SD-JWT VC (pin the exact IETF version) — issuer disclosures, holder selection, key binding JWT, verifier checks; tampering, digest-mismatch, duplicate-disclosure and omitted-claim tests with spec vectors where available. Replace the mock SD-JWT path rather than adding a parallel one.
  X1 Standard exchange: OpenID4VCI 1.0 issuance slice (offer, pre-authorized code, token, credential endpoint, nonce/proof), then OpenID4VP 1.0 presentation slice (DCQL, direct_post, nonce/audience binding), each with rejection and replay tests; claim conformance only from an independent tool or wallet library actually run.
  K1 Signing-key lifecycle: signer abstraction for HSM/KMS without exporting keys (software impl + fake remote signer in tests), rotation with historical verification, recovery tests.
  R1 W3C Bitstring Status List publication and verification, replacing ad-hoc snapshot formats where safe, compatibility documented.
  O1 Operability: multi-instance challenge consumption and restart tests against PostgreSQL, health/readiness endpoints, startup config validation.
  C1 Consolidation of duplicated verifier/denylist/pinning paths behind shared policy types where tests prove identical behaviour; delete dead code and its docs.

### P5 PROJECT RULES
- Use the platform or well-reviewed libraries; never hand-roll primitives. New dependencies need a reason and licence check in the PR. No real credentials or private keys in the repo; test keys are generated in tests.
- Extra merge condition: while Actions is billing-blocked, do not merge a change to the database layer or integration-tested paths unless integration tests ran somewhere (locally with Docker or hosted).

### P6 HOUSEKEEPING SCOPE
- Consolidate overlapping profile docs into one page per area (e.g. the jwt-vc-json-* and vcdm20-* families), keeping every boundary, security limit and non-claim statement, with a short redirect line at old paths that code comments or other docs link to. Report docs file count and total size before/after.

### P7 REPORT EXTRAS
- The VISION milestone gap each merged PR closes; docs file count and size after housekeeping.
