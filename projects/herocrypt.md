# HeroCrypt — project rules

Owner-controlled seed for the "HeroCrypt 推进计划" scheduled task (repo KoalaFacts/HeroCrypt). The repo skill is `.claude/skills/advance-herocrypt/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
DEVELOPMENT_ROADMAP.md is the ledger. V1's vector coverage matrix goes in STANDARDS_COMPLIANCE.md itself, not a linked new page. P2 and H1 design notes go in a section of docs/CRYPTO_SPEC.md (OpenPGP ones in docs/OPENPGP_COMPLIANCE_PLAN.md). Wycheproof licence and pinned commit go in the existing THIRD-PARTY-NOTICES.md. P6's docs/archive/ moves become fold-and-delete, and P6's consolidation of overlapping security/testing pages must end with fewer files.

## Verification boundary
the real boundary is the official vectors run through the public API (RFC appendices, NIST/FIPS KATs, Wycheproof) and round-trips with an independent implementation (GnuPG for OpenPGP; the .NET platform primitives where they exist) — never only the library checking itself; the verifier checks each vector's source and pinned version.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroCrypt (public). A .NET cryptography library (netstandard2.0, net8.0, net9.0, net10.0; xUnit v3 on Microsoft.Testing.Platform): hashes/KDFs (SHA, Blake2b, Argon2, PBKDF2, HKDF, Scrypt), AEADs (ChaCha20/XChaCha20-Poly1305, AES-GCM/CCM/SIV/OCB), stream ciphers, RSA/ECDSA/Ed25519/X25519/secp256k1, .NET 10 ML-KEM/ML-DSA/SLH-DSA, an RFC 9580 OpenPGP subset, BIP32/39, Shamir, X3DH/Double Ratchet, and educational protocols.
- Goal: every supported path proven correct against official standards vectors and independent implementations, robust against hostile input, honest and well-organised documentation that states exactly what is qualified, and an easy, safe-by-default API.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feat/`, `fix/`, `test/`, `docs/`, `refactor/`, `perf/`, `ci/`; squash merge. Report green Dependabot PRs; never merge them.
- Never edit, dispatch or re-run publish-nuget.yml or create-release.yml. User-visible changes go in CHANGELOG.md [Unreleased] and docs/migration-guide.md when they affect upgrading.

### P3 SETUP AND GATE
- Read CONTRIBUTING.md, SECURITY.md, PRODUCTION_READINESS.md, STANDARDS_COMPLIANCE.md, TEST_STATUS.md, CHANGELOG.md, DEVELOPMENT_ROADMAP.md, docs/README.md, docs/CRYPTO_SPEC.md, docs/THREAT_MODEL.md, docs/OPENPGP_COMPLIANCE_PLAN.md, tests/HeroCrypt.Tests/TESTING_CONVENTIONS.md, .github/workflows/README.md.
- SDKs: global.json pins 10.0.402 with rollForward disable. dotnet-install.sh `--version 10.0.402` into a scratch folder, plus `--runtime dotnet --channel 8.0` and `9.0` into the same folder; set DOTNET_ROOT and PATH. Node 24 for TypeScript tools.
- LOCAL GATE: `dotnet restore --locked-mode` (after a PackageReference change `--force-evaluate` and commit the lock files); `dotnet build --configuration Release --no-restore /p:TreatWarningsAsErrors=true` (all TFMs incl. netstandard2.0, examples and the netstandard clear consumer); `dotnet test --project tests/HeroCrypt.Tests/HeroCrypt.Tests.csproj -c Release --no-restore` on net8.0, net9.0, net10.0 with test and skipped counts; `dotnet format --verify-no-changes --no-restore`; the existing `python3 -m unittest discover -s scripts -p 'test_release_validation.py'`; the docs checker once it exists.

### P4 LEDGER AND ORDER
- Ledger: DEVELOPMENT_ROADMAP.md (turn the stale phase list into the living ledger in the first PR; keep still-valid old items as rows, say which were dropped and why).
- Initial order:
  D0 Docs truth pass: contradictions between README, SECURITY.md, PRODUCTION_READINESS.md, STANDARDS_COMPLIANCE.md, TEST_STATUS.md and the code (e.g. post-quantum called "reference/educational only" vs .NET 10 native APIs; README's RSA 512–4096 claim vs what the code allows). Never claim audits, certification, FIPS validation or interop not actually tested. The run that merges D0 also merges a code PR.
  V1 Vector coverage matrix per algorithm/API (RFC appendices, NIST CAVP/ACVP, FIPS 203/204/205 KATs, BIP vectors, RFC 9580 samples) with the proving test or "gap", in STANDARDS_COMPLIANCE.md or a linked page. Fetch the specs; do not rely on memory.
  V2 Close vector gaps, one family per PR: Project Wycheproof (licence and pinned commit in THIRD-PARTY-NOTICES.md) for AEADs, Ed25519/X25519, ECDSA incl. secp256k1, RSA-OAEP/PSS/PKCS#1, HKDF. Every vector failure is a real bug: fix with a regression test and a CHANGELOG/migration note when it rejects previously accepted input.
  P1 OpenPGP interoperability: bidirectional pinned-version tests against GnuPG (and OpenPGP.js via a TypeScript harness if feasible) for v4/v6 signatures, RSA/X25519 envelopes, SEIPD v1/v2, in a CI job without new permissions; record exactly what interoperates in PRODUCTION_READINESS.md.
  P2 OpenPGP profile gaps (integrated signed-and-encrypted messages, ECDH v6 sessions), design note first, smallest additive API. Legacy/weak ciphers (3DES, CAST5, IDEA, Blowfish), even decrypt-only, are an owner decision.
  R1 Robustness: fixed-seed fuzz/property tests of every untrusted-input parser (OpenPGP packets, armor, key rings, S2K incl. Argon2 limits, ciphertext framing, BIP39); fix every non-documented exception, hang or unbounded allocation with a regression test.
  S1 Side-channel and secret hygiene: constant-time MAC/tag/password checks, zeroization, no secrets in exceptions or ToString; fix proven issues with tests.
  U1 Ease of use: docs that compile (extend the docs checker to compile ```csharp blocks from README.md and docs/**/*.md against src/HeroCrypt, with a marker for deliberately partial snippets; never delete an example to pass); complete accurate XML docs; clear exception messages.
  H1 Hybrid post-quantum key agreement (X25519 + ML-KEM per the current IETF draft or HPKE profile): design note pinning spec version and vectors; implement only additively, otherwise owner decision.
  F1 Performance: BenchmarkDotNet same-machine A/B with both runs recorded; never weaken security for speed.

### P5 PROJECT RULES
- Never hand-roll a new primitive where the platform or an existing in-repo implementation exists; never loosen a security default or re-enable disabled features (threshold signatures, MPC, PSI, Beaver triples); educational protocols stay labelled educational. Breaking public API changes are an owner decision. No new third-party runtime dependency in src/HeroCrypt; test-only dependencies need a reason and licence check. Test fixtures are test data, never operational keys.
- Extra merge condition: hosted CodeQL/security scans that ran passed; no vector or security default weakened.

### P6 HOUSEKEEPING SCOPE
- README.md, the root *.md files and docs/**. GitHub-recognised files (README.md, SECURITY.md, CONTRIBUTING.md, CHANGELOG.md, LICENSE, THIRD-PARTY-NOTICES.md) stay where they are. Consolidate overlapping topics (e.g. SECURITY.md vs VULNERABILITY_DISCLOSURE.md vs docs/API_SECURITY.md; TEST_STATUS.md vs docs testing pages; README feature list vs STANDARDS_COMPLIANCE.md) into one canonical page with links. Completed or abandoned plans (OPENPGP_COMPLIANCE_PLAN phases done, docs/superpowers plans for shipped features) get a status header and go to docs/archive/; still-open items become ledger rows. docs/README.md is grouped by reader (getting started, choosing an algorithm, production qualification, protocols, contributors). The docs checker also flags plan/spec pages without a status header. Replace scripts/check-xml-docs.sh / quick-doc-check.sh only if the new tool fully covers them. Never loosen a security warning or limitation statement.

### P7 REPORT EXTRAS
- Test counts per TFM; vector sources / spec sections covered by merged PRs.
