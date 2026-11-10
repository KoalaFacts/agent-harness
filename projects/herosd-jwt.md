# HeroSD-JWT — project rules

Owner-controlled seed for the "HeroSD-JWT 推进计划" scheduled task (repo KoalaFacts/HeroSD-JWT). The repo skill is `.claude/skills/advance-herosd-jwt/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
docs/contributors/roadmap.md is the ledger. C1's conformance matrix and C4's SD-JWT VC design note go in a section of an existing page that documentation-governance.md assigns (e.g. the roadmap or the relevant docs/users/ page), not a new file. P6's retirement of docs/superpowers plans and dated docs/development notes is fold-and-delete, never archive.

## Verification boundary
the real boundary is the spec's own worked examples run through the public API, the AOT console smoke (issue → present → verify), and the AspNetCore handler exercised in a test host with real HTTP requests; the verifier traces each conformance claim to its spec section.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroSD-JWT (public). A .NET SD-JWT (selective disclosure for JWTs) library: core package HeroSdJwt (BCL-only, net8.0/net9.0/net10.0, AOT-safe) and HeroSdJwt.AspNetCore (DI, authentication handler, distributed replay/revocation adapters).
- Goal: a spec-conformant, interoperable, secure-by-default, easy-to-use SD-JWT library with always-true, tidy docs, ready for the 2.0 release.
- Schedule: standard tier — once a day, 50 minutes per run.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feature/`, `fix/`, `docs/`, `refactor/`, `test/` (per CLAUDE.md); squash merge. Other sessions also push `fix/` PRs here: apply the overlap guard. Report green Dependabot PRs.
- Never edit, dispatch or re-run publish-nuget.yml, create-release.yml or publish-approved-2.0.0.yml (that last one runs on any push to main that touches it — never touch that file). User-visible changes in CHANGELOG.md [Unreleased] and docs/users/upgrading-to-2.0.md when they affect upgrading.

### P3 SETUP AND GATE
- Read CLAUDE.md (binding), CONTRIBUTING.md, SECURITY.md, CHANGELOG.md, docs/README.md, docs/maintainers/documentation-governance.md, docs/maintainers/versioning-policy.md, docs/contributors/roadmap.md, the docs/users/ page of the area.
- SDKs: global.json pins 10.0.402 with rollForward disable: dotnet-install.sh `--version 10.0.402` plus `--runtime dotnet --channel 8.0` and `9.0` into one scratch folder; set DOTNET_ROOT and PATH.
- LOCAL GATE: `dotnet restore HeroSD-JWT.slnx` (locked mode; `--force-evaluate` after a PackageReference change, commit lock files); `dotnet build HeroSD-JWT.slnx -c Release` (warnings are errors in src/) incl. examples/ and benchmarks/; `dotnet format --verify-no-changes`; `dotnet test HeroSD-JWT.slnx -c Release` on all three TFMs; an AOT publish smoke of a tiny console app that issues, presents and verifies one SD-JWT with zero IL/trim/AOT warnings (add it as a test project or TypeScript script when first needed); the docs checker (link check + snippet compile) once it exists.

### P4 LEDGER AND ORDER
- Ledger: docs/contributors/roadmap.md (the canonical contributor roadmap; its "Future Improvements" content is stale — turn it into the ledger in the first PR, keeping still-valid items as rows).
- Initial order:
  D0 First docs housekeeping together with or right after the ledger rewrite.
  C1 Spec conformance matrix: the current IETF SD-JWT spec (published RFC if any, otherwise latest draft — fetch it) section by section against the code: disclosure format, digests/_sd, array elements, decoys, `_sd_alg`, key binding JWT (`sd_hash`, `typ`, iat/nonce/aud), presentation format, verification order, security considerations; one row per MUST/SHOULD with the proving test or "gap", placed per governance.
  C2 Close conformance gaps one per PR, pinned with the spec's own examples where given.
  C3 Spec worked examples as test vectors; third-party suites only after their licence is checked and recorded (unclear → owner decision).
  C4 SD-JWT VC readiness: design note for `typ`, `vct`, status, issuer key resolution with the smallest additive API; implement only without breaking the 2.0 surface, otherwise owner decision.
  D1 Docs that compile: the TypeScript docs checker extracts ```csharp blocks from README.md and docs/**/*.md and compiles them against src projects (marker for deliberate partial snippets; never delete an example to pass), plus link/anchor checks, as a ci.yml job without new permissions; fix CLAUDE.md/README claims that are wrong (e.g. README's `Category=` test filters).
  D2 Docs audit: README, docs/users/*, docs/operations/*, the AspNetCore package README and samples vs the code (API names, defaults, error codes, algorithms, frameworks), one area per PR.
  S1 Robustness: extend Fuzz/ and CsCheck property tests to disclosures, presentations, key-binding JWTs and claim paths; fix crashes or unbounded resource use with regression tests.
  U1 Ease of use: complete accurate XML docs, clear VerificationResult messages for every ErrorCode, examples/ build and run.
  P1 Performance with the existing BenchmarkDotNet project, same-machine A/B only, never weakening security.

### P5 PROJECT RULES
- Security defaults are never loosened (none-alg rejection, ExpectedKeyType Asymmetric, fail-closed replay/revocation, constant-time comparisons). Breaking public API changes beyond what upgrading-to-2.0.md documents are an owner decision. No new third-party runtime dependency in HeroSdJwt. Do not refactor Internal/Ed25519 except for a proven bug with a test.

### P6 HOUSEKEEPING SCOPE
- Follow docs/maintainers/documentation-governance.md (single source of truth, audience-first folders, naming): merge duplicated setup/security/verification guidance across README, docs/users and docs/operations; retire finished docs/superpowers/plans and dated docs/development notes (fold true content into canonical pages; delete or archive per governance); keep CLAUDE.md and CONTRIBUTING.md matching real commands and layout. Keep LICENSE, SECURITY.md's reporting process, CHANGELOG history and upgrading-to-2.0.md content intact (fix only wrong facts and links). Housekeeping is additionally due when 5 PRs have merged since the last one.

### P7 REPORT EXTRAS
- Test counts per TFM; spec sections covered by merged PRs.
