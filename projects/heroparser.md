# HeroParser — project rules

Owner-controlled seed for the "HeroParser 推进计划" scheduled task (repo KoalaFacts/HeroParser). The repo skill is `.claude/skills/advance-heroparser/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
the ledger is docs/plans/maintenance-ledger.md only if it already exists; otherwise a "Maintenance ledger" section of CHANGELOG.md's [Unreleased] notes or of CLAUDE.md (whichever P6 makes the source). L3's two options go on #167 and in the ledger section, not in a new file. P6's docs/archive/ move is dropped: docs/perf/ notes are benchmark evidence and stay where they are, with at most a one-line status note added at the top; superseded non-evidence notes are folded into their owning page and deleted. The CLAUDE.md/AGENTS.md divergence check is a script and is allowed; the non-source file may become a short pointer to the source by editing it in place.

## Verification boundary
the install branch is `docs/verify-skills`; the skills line goes in whichever of CLAUDE.md/AGENTS.md P6 makes the source (CLAUDE.md until then), and the divergence check must accept it. The real boundary is the CLI and the AOT native binary run on real CSV / fixed-width / Excel / JSONL files on net10.0 and net11.0, plus the Wasm npm package and the browser demo under /HeroParser/demo/ in headless Chromium when they change.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroParser. A .NET CSV / Fixed-Width / Excel / JSONL / HTB parser library with a source generator, a console engine, a CLI, a WebAssembly npm package and a browser demo.
- Goal: .NET 11 readiness before .NET 11 GA, all dependencies current, and documentation that is always true, tidy and organised — preparation only, no new release (owner: "对全部依赖升级 然后开始兼容 net 11 但是我们不需要发布新版本 只是准备").
- Schedule: standard tier — twice a day, 50 minutes per run.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `net11/`, `deps/`, `fix/`, `docs/`; squash merge. Also allowed (owner: "全程你自主开发", "能合并就合并"): PR #167 (`feature/net11-support`, counts as this task's PR) and green Dependabot patch/minor PRs with no findings, merged between PRs. Other people may push to #167: fetch first and merge their commits in; never overwrite or revert their work without a traced reason. Touch #162 only to close it as superseded after #167 merges.
- NO RELEASE: never dispatch or re-run Create Release or any Publish workflow (NuGet, WinGet, Homebrew, Scoop, Snap); never change `<Version>` in Directory.Build.props; never run `npm run sync-version`; never publish the npm package. CHANGELOG [Unreleased] (structure Added / Changed / Fixed / Build and CI / Dependencies / Notes).

### P3 SETUP AND GATE
- Read CLAUDE.md, AGENTS.md, CHANGELOG.md, .github/workflows/README.md.
- SDKs: dotnet-install.sh `--channel 11.0 --quality preview` and `--channel 10.0` into one scratch folder first on PATH, then `dotnet workload install wasm-tools`; add 8.0/9.0 runtimes when needed. Node 24+ and npm 12 (`npm install --prefix <scratch> npm@12`).
- LOCAL GATE: `dotnet restore HeroParser.slnx --force-evaluate`; `dotnet build HeroParser.slnx -c Release` (warnings are errors); `dotnet format whitespace --verify-no-changes HeroParser.slnx`; HeroParser.Tests, Generators.Tests and Console.Tests on net10.0 and net11.0 (`dotnet test --project <proj> -c Release -f <tfm>`); AotTests: `dotnet run` on net11.0 plus `dotnet publish -f net11.0 -r linux-x64` and run the native binary with no IL/trim warnings; when npm/ or Wasm changed: Wasm publish, `npm ci --prefix npm --legacy-peer-deps`, demo build, `vue-tsc -b`, `vp fmt --check`, `node npm/tests/test-wasm.js`; the docs checks (snippet compile + link check) once they exist.

### P4 LEDGER AND ORDER
- Ledger: docs/plans/maintenance-ledger.md (statuses may also include `waiting (on what)`).
- Initial order:
  L1 Merge #167: net11.0 target; global.json floor with rollForward latestFeature + allowPrerelease; CI on the newest 11.0 SDK via setup-dotnet `11.0.x` (no forced dotnet-quality, per the owner's review); NuGet consumer check covers net11.0; dependency upgrades. RC3 is go-live, so merge once CI is green. Then close #162 (CsvHelper 34, superseded) pointing to the merged commit.
  L2 Docs that compile: a TypeScript tool extracts ```csharp blocks from README.md and docs/*.md, compiles them in a scratch project against the repo's projects, honours an explicit marker for deliberately partial snippets, runs as a CI job; fix every snippet (README needed missing usings and a wrong ToJsonLlmChunksAsync call in 2.8.2). Never delete examples to pass.
  L3 Historical-evidence job: owner decision. Write the two options (retire the job, or report "evidence expired" as pass-with-notice) in the ledger and on #167; do not change the job until the owner answers.
  L4 After .NET 11 GA (detect from https://builds.dotnet.microsoft.com/dotnet/release-metadata/11.0/releases.json: `latest-release` without a prerelease suffix): verify CI installed the GA SDK, raise the global.json floor to GA (the 2.9.0 release gate), BenchmarkDotNet to the newest stable 0.16.x if any, update "RC" wording. Until GA this is `waiting`, not blocked.
  L5 .NET 11 analyzer and obsoletion sweep with `-p:AnalysisLevel=latest-all`: fix real issues with tests first; record intentionally ignored diagnostics with reasons in the ledger (suppress in code only as CLAUDE.md allows).
  L6 Dependency upkeep between PRs (`dotnet list HeroParser.slnx package --outdated`, also `--include-prerelease`; `npm outdated --workspaces --include-workspace-root` in npm/): port breaking majors in `deps/` PRs preserving behaviour (as CsvHelper 34 kept v33 quoting with CsvQuoting.WhenNeededOrWhitespace); shipped packages' runtime deps stay stable; keep `onnxruntime-web` at 1.20.1 (override + postinstall patches) and TypeScript 6.x until vue-tsc supports 7, noted in the ledger.
  L7 Docs accuracy audit (README.md, docs/*.md, the npm package README, the browser demo text, CLAUDE.md/AGENTS.md, .github/workflows/README.md vs the code), one area per PR; the demo must keep working under /HeroParser/demo/ (Playwright smoke test when changed).

### P5 PROJECT RULES
- Extra merge-condition exception: the known "Validate Historical Evidence And Optional Fixed Collection" job fails on any PR touching csv-pipe-history.yml because the artifact of run 36732093817 expired; it is not counted against a merge and is never "fixed" by a re-run.
- Codex reviews only when a PR is opened, so do not wait for a new Codex review after a push; check the threads and the diff yourself. The Codex review of the PR must be completed with no open blocking/high/security finding, and every thread (including the owner's account) fixed or answered.
- Never claim performance without CLAUDE.md's same-runner A/B.

### P6 HOUSEKEEPING SCOPE
- docs/README.md index; CLAUDE.md and AGENTS.md are near-copies that drift — make one the source and have the other point to it or carry a checked copy, with a TypeScript CI check that fails when they diverge; move superseded or retired docs/perf/ notes (e.g. the CSV Pipe timing work) to docs/archive/ keeping evidence and benchmark history; version wording stays "coming in 2.9.0" until released.

### P7 REPORT EXTRAS
- .NET 11 status and the SDK CI used; dependency changes and what is held back.
