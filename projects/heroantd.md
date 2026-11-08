# HeroAntD — project rules

Owner-controlled seed for the "HeroAntD 推进计划" scheduled task (repo KoalaFacts/HeroAntD). The repo skill is `.claude/skills/advance-heroantd/SKILL.md`; if it is missing on main, the task's first PR creates it from this file (see `skills/autonomous-dev` §1). The task may make these rules stricter in its own skill, never looser.

References in the text below to B2–B12 point at the shared rules that now live in `skills/autonomous-dev` and `skills/verification-first`: B2 → §4 What counts, B3 → §5 authority, B4 → §1 Setup, B5 → §7 Ledger, B6 → §3 The loop, B7 → §6 Housekeeping, B8 → §5 Merge conditions, B9 → §9 Engineering defaults, B10 → §10 Report, B11 → §6 Docs discipline (and the section below), B12 → `verification-first` and §8.

## Docs files in this repo
no docs/ROADMAP.md, docs/COMPONENTS.md or CHANGELOG.md is created if missing. The ledger and the per-component parity table go in a section of README.md (or of an existing docs page if one fits); the Unreleased list is a README section. Storybook stories are product code, not docs, and are unaffected.

## Project

### P1 WHAT AND WHY
- Repo KoalaFacts/HeroAntD (public). A Bun monorepo porting Ant Design v6 (React) to framework-agnostic Web Components: packages/core (Stencil 4, light DOM `shadow: false`, tag = Ant CSS prefix, e.g. `<ant-btn>`; vitest browser specs), packages/tokens (Ant Design CSS and tokens extracted from antd by scripts/), packages/icons (generated icon components), packages/docs (Storybook 10, web-components-vite; packages/docs/vercel.json builds it).
- Goal: a faithful, accessible, easy-to-use Web Components edition of Ant Design v6 — every ported component matching the React API and rendered output as closely as Web Components allow, working in plain HTML and any framework (including native forms), with docs and stories that always say exactly what is true.
- Schedule: every 4 hours (6 runs a day), 50 minutes per run.
- Known blocker (2026-11-06): three runs produced no branch or PR; this session could not reach the repo through the GitHub API. Apply B4's push-access line first.

### P2 AUTHORITY SPECIFICS
- Branch prefixes `feat/`, `fix/`, `test/`, `docs/`, `refactor/`, `ci/`, `deps/`; squash merge.
- Merging to main may make Vercel's Git integration rebuild and publish the Storybook site; that is why the docs build is in the gate. Never use the Vercel tools or CLI, never change Vercel settings.
- Never `npm publish` / `bun publish`; package versions track the Ant Design version and are never changed. Unreleased changes go in the README section B11 names.

### P3 SETUP AND GATE
- Read README.md (the porting guide and its principles: no custom CSS, light DOM, match the React API, slots for ReactNode), biome.json, every package.json, packages/core/stencil.config.ts and vitest.config.ts, packages/core/scripts/scaffold.ts, packages/tokens/scripts/.
- Toolchain: Bun (official installer into a scratch folder) and Node 24; `bun install`. Core's browser specs use @vitest/browser-webdriverio with Chrome: install a headless Chrome/Chromium (e.g. `npx @puppeteer/browsers install chrome@stable` or Playwright's chromium) and point webdriverio at it.
- LOCAL GATE: `bun install --frozen-lockfile` (after a dependency change update and commit bun.lock); `bun run build` (biome check, tokens, icons, core) with zero biome errors; `bun run test` with core's browser specs actually running (record spec and test counts); `bun run build:docs`; `tsc --noEmit` for core and docs once T0 adds it; `bun run check:docs` once it exists.
- verify-heroantd's real boundary: a component rendered in headless Chromium (spec or Storybook story) with its Ant classes, events and keyboard behaviour observed, compared against antd React where a comparison exists.

### P4 LEDGER AND ORDER
- Ledger and per-component parity table: per B11 (README sections). The parity table has one row per Ant Design v6 component (from the antd version in the lockfile — read its exports, not memory) with HeroAntD tag(s), status (ported / partial / missing / not applicable and why), props/events/methods parity versus the React .d.ts (scaffold output), keyboard/ARIA coverage, spec, story. Keep it true in every PR.
- Initial order:
  T0 Runnable gate and CI: core browser specs run headless here and in CI (switch to @vitest/browser-playwright as docs already uses if webdriverio is the obstacle, dropping no spec); type-check scripts; .github/workflows/ci.yml running the gate with read-only permissions and no secrets.
  D0 Docs truth pass: README against the code (scripts, paths, package names, the scaffold flow actually working); a short user quick start (install, loader or custom elements, tokens CSS, a working HTML example) separate from the contributor porting guide, as README sections; ledger, parity table, docs checker. The run that merges D0 also merges a code PR.
  C1 Spec coverage: every component in packages/core/src/components has a spec (65 components vs 62 specs at 2026-11-06) checking rendered Ant classes for main props, events and disabled/loading states.
  P1 Parity, one family per PR (button, input, input-number, select, checkbox, radio, switch, tabs, modal, drawer, dropdown, menu, table, pagination, tooltip/popover first): props/events/methods vs the scaffold .d.ts, rendered DOM/classes vs antd React (extend the ButtonComparison story into an automated DOM-class comparison test where practical); record parity in the parity table.
  F1 Native forms: `formAssociated: true` + ElementInternals (value, name, disabled, required, validity, reset/restore) for input, input-number, select, checkbox, radio, switch, slider, rate, cascader, color-picker, upload, with tests submitting a real <form>.
  A1 Accessibility: keyboard paths, focus management (modal/drawer trap and return, menus, dropdowns, tabs roving tabindex, tree), ARIA per WAI-ARIA APG, with tests; fix biome a11y warnings rather than silencing them.
  M1 Missing v6 components, one per PR by usefulness (Form + Form.Item, DatePicker/TimePicker, AutoComplete, TreeSelect, Transfer, Segmented, Message/Notification imperative API, Carousel, Calendar, Mentions, QRCode, Watermark, Splitter), each via the scaffold flow with spec, story and parity-table row.
  TH1 Theming: dark and compact tokens usable from the docs (Storybook theme switch), tested.
  U1 Ease of use: typed events and custom-element type declarations for consumers, framework usage examples (plain HTML, React 19, Vue).
  DEP Dependency upkeep between PRs (`bun outdated`; antd, Stencil, Storybook, vitest, biome in `deps/` PRs, re-extracting tokens when antd changes and recording CSS deltas; no prerelease in shipped packages — the docs' vite beta may stay until a stable exists, noted in the ledger).
- Every PR that touches a component also updates its Storybook story (controls for every new prop, an example for every new slot/event) and the parity table.

### P5 PROJECT RULES
- Keep the README principles: styles come from @hero-antd/tokens, light DOM, React-API naming per the mapping table. Ant Design is MIT: keep its licence notice where its CSS or code is used; never copy code from non-MIT sources.
- No Chromatic publishing or other paid services.
- Extra merge conditions: the docs build and the browser specs actually ran; a Vercel preview check that ran and failed blocks the merge; breaking changes to an existing component's public API (tag, prop/event/method names or types) are an owner decision unless they fix a React-API mismatch the parity table records — then the Unreleased section carries a migration line.

### P6 HOUSEKEEPING SCOPE
- README.md, docs/, package READMEs and the Storybook docs pages; remove unused Storybook template leftovers (e.g. packages/docs/stories/assets) after checking every reference.

### P7 REPORT EXTRAS
- Whether the browser specs and the docs build ran; components ported / partial / missing from the parity table.
