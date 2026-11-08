# agent-harness

A small, self-evolving harness for autonomous scheduled development with Claude Code.

The idea: a scheduled task should be a **thin trigger**, not a 30 KB rulebook. Knowledge about *how* to work lives in skills; knowledge about *a specific repo* lives in that repo and is improved by the agent itself through PRs. Only the hard limits — what the agent must never do — stay in the trigger, where the agent cannot edit them.

```
trigger prompt (≈10 lines, owner-controlled)     what must never happen; which repo; time budget
        │
        ├── agent-harness/skills/autonomous-dev       how a run works (shared by every repo)
        ├── agent-harness/skills/verification-first   how to prove a change works (shared)
        ├── agent-harness/projects/<name>.md          the owner's project rules: scope, authority, order (per repo, owner-controlled)
        │
        └── <repo>/.claude/skills/advance-<repo>      goal, metric, gate, order, scope (per repo, self-evolving)
            <repo>/.claude/skills/verify-<repo>       feature map, real checks, traps (per repo, self-evolving)
```

## Layers and who may change them

| Layer | Lives in | Changed by |
|---|---|---|
| Hard limits | the scheduled task's prompt | the owner only |
| Run loop, verification method | this repo (`skills/`) | the owner merges; agents may only open PRs |
| Project rules (scope, authority, gate, order) | this repo (`projects/`) | the owner merges; agents may only open PRs |
| Repo goal, gate, work order, verify map | each repo's `.claude/skills/` | the agent, through its own PRs |

A rule in a lower layer can make things stricter, never looser. If a repo skill ever conflicts with a hard limit in the trigger, the trigger wins.

## Contents

- [`skills/autonomous-dev/SKILL.md`](skills/autonomous-dev/SKILL.md) — the run loop: one PR at a time, time budget, review, merge conditions, docs discipline, reporting, self-evolution.
- [`skills/verification-first/SKILL.md`](skills/verification-first/SKILL.md) — done criteria, real-behaviour evidence, independent verdict.
- [`projects/`](projects/) — one file per scheduled task: the owner's project rules, seeding each repo's `advance-<repo>` skill.
- [`templates/advance-skill.md`](templates/advance-skill.md) — template for a repo's `advance-<repo>` skill.
- [`templates/trigger-prompt.md`](templates/trigger-prompt.md) — the thin scheduled-task prompt.

## Adopting it in a repo

1. Add the project rules as `projects/<name>.md`.
2. Replace the scheduled task's prompt with `templates/trigger-prompt.md`, filled in, scheduled every 4 hours.

The first runs create `advance-<repo>` (from the project rules, per `templates/advance-skill.md`) and `verify-<repo>` if they are missing, and point the repo's `AGENTS.md` / `CLAUDE.md` at them.

## License

Apache-2.0 — see [LICENSE](LICENSE).
