# FactsYOLO — project rules

Owner-controlled seed for the "FactsYOLO 推进计划" scheduled task (repo KoalaFacts/FactsYOLO). The repo skill is `.claude/skills/advance-factsyolo/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
the ledger is docs/plans/td2-roadmap.md only if it already exists; otherwise a "Roadmap" section of the TD2 spec. docs/README.md is not created; README.md indexes the docs. The stage B comparison plan goes in a section of the TD2 spec, and the stage C licence record in the existing MODEL_LICENSES.md. One explicit exception: ADR 0002 (the Burn vs Candle decision the owner asked for) may be a new numbered file in docs/decisions/ only if CONTRIBUTING.md requires numbered decision records for dependency/framework choices; otherwise it is a section of the TD2 spec. Training/benchmark metric JSON artifacts are data, not docs. P7's "superseded records marked historical with a pointer" applies only to kept evidence records, by editing them in place.

## Verification boundary
the install branch is `claude/verify-skills`. The real boundary is the built library called through its C ABI (xtask c/verify and the C# binding) on original synthetic frames, with contract fixtures compared and measured numbers recorded; experiments are verified by their recorded metric JSON, not by the author's summary. No dataset pixels or external weights are ever used as verification inputs.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/FactsYOLO (public). An original, Apache-2.0, Rust-first vision core with a C ABI and thin C# (later Go/TS) bindings, embedded by customers in their own CCTV cameras, NVRs and edge devices on premises. Design: docs/superpowers/specs/2026-11-05-td2-general-detector-design.md (on main once PR #68 merges, otherwise on branch claude/td2-general-detector-design-20261105).
- Task order (owner-approved 2026-11-05): T1 detection (TD2: objects, animals, people, vehicles, faces, licence plates) → T2 licence-plate recognition → T3 scene recognition → T4 face recognition.
- Schedule: standard tier — once a day, 50 minutes per run.
- Equivalent deliverable for B2: merged code, tests, measured evidence or a decision record that moves a ledger item forward; negative results count when recorded.

### P2 AUTHORITY SPECIFICS
- Branch prefix `claude/` (and PR #68); squash merge. Never merge codex/ branches or Dependabot. Housekeeping branch: `claude/docs-hygiene-YYYY-MM-DD`.
- A PR opened in this run may be merged in this run only after all hosted CI on its head completed and passed and any review bot finished; if Codex reviews this repo and has not reviewed the head, wait (up to ~30 minutes in-run, otherwise next run) while the PR is younger than 12 hours.

### P3 OWNER DECISIONS ALREADY MADE (2026-11-05)
- On-prem, customer-installed and customer-operated: no data ever leaves the customer's control; the deployed library has no networking code, writes no files, stores nothing; the host owns decoding, storage, galleries and watch-lists.
- Face and licence-plate recognition are required; scene, animal and object recognition are very important.
- Budget ZERO: only free GitHub-hosted runners (incl. ubuntu-24.04-arm as interim aarch64 host) and this workspace (2 CPUs, no GPU). Kaggle/Colab need the owner's account: prepare notebooks/scripts and ask; never create accounts.
- Framework (Burn vs Candle) is decided by stage B evidence (ADR 0002). The spec's other open items are adopted as written unless evidence shows them wrong (fix the spec in a PR and say why).
- STILL THE OWNER'S: approving recorded licence terms of any external dataset before download (stage C: research and record terms, category mapping and attribution in MODEL_LICENSES.md first); any spending, accounts, publishing weights or SDK packages, tags/Releases, licence or trademark changes.

### P4 SETUP AND GATE
- Clone with GIT_LFS_SKIP_SMUDGE=1. Read CONTRIBUTING.md (binding: provenance, no Ultralytics code/weights, dependency review, written decision before ABI/format/dependency changes, tests first), MODEL_LICENSES.md, docs/decisions/, the TD2 spec, the ledger, docs/testing/m1-evidence.md.
- Install the pinned Rust toolchain via rustup. LOCAL GATE = the README's "Build and verify" commands the workspace can run (cargo test debug/release, contract-fixtures, clippy -D warnings, fmt --check, xtask c/verify where tools exist; experiments/cpu-detector checks when touched). Long runs go to GitHub Actions.

### P5 LEDGER AND ORDER
- Ledger: docs/plans/td2-roadmap.md (work items by spec stage A–J).
- Initial order: (1) merge PR #68; (2) stage B: predeclared comparison plan, a fixed CPU proxy (same seeds, initialization, steps; small conv+BN+SiLU+upsample net on original synthetic data) trained with Burn and Candle in experiments/, gradient agreement vs an independent reference forward, throughput and peak memory on x64 and arm64 runners, dependency/licence review, then ADR 0002; (3) stage C research and licence record (owner gate before download); (4) stage D harness and reduced-scale training on free runners; (5) stage E native td2 backend, NEON/AVX2, aarch64 CI with real execution, no-networking CI gate; then F onward. While an item waits on the owner or CI, advance the next unblocked one (e.g. T2–T4 design, synthetic plate rendering). T4 training stays blocked until a budget exists. TD1 is maintenance-only.
- Long training/benchmark jobs: GitHub Actions workflows, resumable in chunks well under 6 hours, each finished chunk's checkpoint saved (actions/cache keyed by run identity) so a later run resumes from the last chunk; metrics as scalar JSON artifacts; analyse finished runs and record results including negative ones.

### P6 PROJECT RULES
- Never add networking code, telemetry or file writes to the deployed library; never upload customer or dataset images; never commit or upload dataset pixels; never publish weights trained on external data (no Releases, committed weights or public artifacts containing them).
- Never port or translate Ultralytics or other detectors' code, never use their weights; cite papers for ideas. No biometric attribute inference (age, gender, ethnicity, emotion). Face/plate features stay opt-in, off in default builds.
- Helper tools follow repo conventions (Rust xtask) or TypeScript; the docs checker may live in xtask.
- Extra merge condition: hosted CodeQL ran and passed; no evidence threshold weakened.

### P7 HOUSEKEEPING SCOPE
- All Markdown: status statements contradicting main (README "Status", docs/testing/m1-progress.md still saying task 8 is under review, ledger vs merged PRs, spec "Owner decisions"); superseded records marked historical with a pointer to the current one (historical evidence keeps its content); docs/README.md sections: current plan and ledger, specs, decisions, evidence by milestone (M1 native, TD1 experiment historical, TD2), testing how-tos; README and CONTRIBUTING reflect the CCTV/edge, on-prem, TD2 direction without over-claiming.

### P8 REPORT EXTRAS
- Training/benchmark jobs started, resumed or finished and their numbers; items re-scoped or blocked and why.
