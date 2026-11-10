# FactsPDF — project rules

Owner-controlled seed for the "FactsPDF 推进计划" scheduled task (repo KoalaFacts/FactsPDF). The repo skill is `.claude/skills/advance-factspdf/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
docs/mvp-roadmap.md is the ledger. User-visible changes go only in the existing draft release notes (docs/releases/0.1.0-preview.1.md); no new release-notes file. P6's move to docs/history/ is dropped: the finished milestone and execution ledgers (docs/*-ledger.md, the m9–m12 notes) are evidence records and stay where they are, unchanged, and pages that cite them keep linking to them; any other superseded note is folded into its owning page and deleted. site/data/compatibility.json and its translation layers are product data, not docs.

## Verification boundary
the real boundary is the CLI and the `Pdf.Write()` SDK rendering the examples/*.html into PDFs, with byte-parity against main (P3.3) and the rendered pages rasterised and looked at for visible changes; the feature map follows site/data/compatibility.json's areas.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/factspdf. A .NET 10 C# HTML-to-PDF engine with a fluent SDK `Pdf.Write()`, a CLI, Native AOT and browser-wasm probes, and the "Can I Render" compatibility site under site/.
- Goal (owner: "一直开发 直到我们可以成为一个可靠的 mvp 产品"): a RELIABLE MVP — correct and deterministic output, every untrusted input either renders or fails with FactsPdfException, bounded memory and time, a user-friendly SDK, near-zero allocation on the hot path, and docs that say exactly what is true.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `perf/`, `feat/`, `fix/`, `test/`, `docs/`, `ci/`; squash merge (owner: "通过了就合并 pr").
- Never dispatch publish-nuget.yml or any publish workflow. User-visible changes go in the release notes under docs/releases/ that are still marked draft/unreleased (currently 0.1.0-preview.1.md), kept unreleased.
- Exception to one-PR-at-a-time: while a code PR waits only on CI you may build the next one locally.

### P3 SETUP AND GATE
- Read AGENTS.md (binding), CLAUDE.md, docs/feature-notes.md, docs/development.md, docs/compatibility-site.md, docs/mvp-roadmap.md.
- .NET 10 SDK via dotnet-install.sh `--channel 10.0`; Node 24 for site/; the existing Python scripts/ tooling (e.g. png_fixtures.py) stays usable.
- LOCAL GATE:
  1. Build EVERY csproj (`git ls-files '*.csproj'`) in Release with warnings as errors, skipping only examples/first-use (needs a local feed); CI also builds probes and acceptance projects outside the solution.
  2. `dotnet test tests/FactsPDF.Tests -c Release` plus the CLI tests.
  3. Byte-parity against main: CLI from main (git worktree) vs branch over every examples/*.html with three option sets — defaults; DejaVu TrueType with `--subset-fonts --page-numbers --header-text H --title T`, images from `python3 scripts/png_fixtures.py`, `--page-size letter --landscape --margin 20mm`; `--page-furniture examples/business-page-furniture.json --page-width 148mm --page-height 210mm` with a fully embedded font. Compare exit codes and bytes; any difference must be intended and explained in the PR. P0 turns this into a TypeScript script.
  4. When site/ data changes, the site's own tests and validation; Can I Render data changes need a new chained English translation layer (docs/compatibility-site.md) plus updated snapshot pins.

### P4 LEDGER AND ORDER
- Ledger: docs/mvp-roadmap.md. Expected order after Z1 (#177): P0 TypeScript parity script (+ CI job against the merge base if it fits an existing workflow without new permissions); Z2 content streams written as ASCII bytes into pooled buffers; Z3 TextStyle single-private-copy like BoxStyle (#176); Z4 CSS tokenizer/parser/adapter allocations; Z5 layout-phase allocations (the 200-row table spends ~2.4 MB in layout); R1 soak and limits (every Max* limit, cancellation on every output path: ToBytes, ToStream, ToStreamAsync, ToBufferWriter, ToFile, ToFileAsync); R2 SDK usability (complete XML docs, README quick start and examples compile, clear FactsPdfException messages with source location for every FPDF code); R3 fixed-seed fuzzing of CSS stylesheets and page furniture (RobustnessFuzzTests), fixing every non-FactsPdfException crash; R4 packaging (`dotnet pack` metadata, README, no stray files, documentation XML; examples/first-use against the packed package from a local feed); R5 the most valuable gap found.
- Each Z item measures allocations per phase before and after (GC.GetAllocatedBytesForCurrentThread per phase and an EventListener on GCAllocationTick by type), lowers tests/FactsPDF.Tests/AllocationBudgetTests budgets to about 15% above the new measurement (never raises them), passes parity, and claims timing only from the existing same-harness benchmark workflow (box-layout-resources.yml).
- A new capability also updates site/data/compatibility.json, docs/feature-notes.md and the area's docs page, as AGENTS.md says.

### P5 PROJECT RULES
- Licensing files (LICENSE.md, LICENSING.md, COMMERCIAL.md, docs/license-history.md) are never edited; add no legal disclaimers. The copyright holder is Koala Facts Trading Pty Ltd. Extension methods follow shared/ExtensionsToString.cs. Extra merge condition: parity passes and allocation budgets were not raised.

### P6 HOUSEKEEPING SCOPE
- docs/, README.md and public API doc comments: fix statements the code no longer matches (removed APIs such as PdfConverter/PdfOptions, old option names, wrong test counts, stale "next step" sections); move finished milestone and execution ledgers (docs/*-ledger.md, m9–m12 notes) into docs/history/ with their evidence and pointers from pages that cite them. Follow AGENTS.md's documentation rules: README gets no per-feature sections and the Can I Render link stays near the top.

### P7 REPORT EXTRAS
- Allocation before/after and parity result for each merged PR.
