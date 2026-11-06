# agent-harness

A small, self-evolving harness for autonomous scheduled development with Claude Code.

The idea: a scheduled task should be a **thin trigger**, not a 30 KB rulebook. Knowledge about *how* to work lives in skills; knowledge about *a specific repo* lives in that repo and is improved by the agent itself through PRs. Only the hard limits — what the agent must never do — stay in the trigger, where the agent cannot edit them.

```
trigger prompt (≈10 lines, owner-controlled)     what must never happen; which repo; time budget
        │
        ├── agent-harness/skills/autonomous-dev       how a run works (shared by every repo)
        ├── agent-harness/skills/verification-first   how to prove a change works (shared)
        │
        └── <repo>/.claude/skills/advance-<repo>      goal, metric, gate, order, scope (per repo, self-evolving)
            <repo>/.claude/skills/verify-<repo>       feature map, real checks, traps (per repo, self-evolving)
```

## Layers and who may change them

| Layer | Lives in | Changed by |
|---|---|---|
| Hard limits | the scheduled task's prompt | the owner only |
| Run loop, verification method | this repo (`skills/`) | the owner merges; agents may only open PRs |
| Repo goal, gate, work order, verify map | each repo's `.claude/skills/` | the agent, through its own PRs |

A rule in a lower layer can make things stricter, never looser. If a repo skill ever conflicts with a hard limit in the trigger, the trigger wins.

## Contents

- [`skills/autonomous-dev/SKILL.md`](skills/autonomous-dev/SKILL.md) — the run loop: one PR at a time, time budget, review, merge conditions, docs discipline, reporting, self-evolution.
- [`skills/verification-first/SKILL.md`](skills/verification-first/SKILL.md) — done criteria, real-behaviour evidence, independent verdict.
- [`templates/advance-skill.md`](templates/advance-skill.md) — template for a repo's `advance-<repo>` skill.
- [`templates/trigger-prompt.md`](templates/trigger-prompt.md) — the thin scheduled-task prompt.

## Adopting it in a repo

1. Copy `templates/advance-skill.md` to `<repo>/.claude/skills/advance-<repo>/SKILL.md` and fill it in from the code.
2. Point the repo's `AGENTS.md` / `CLAUDE.md` at `advance-<repo>` and `verify-<repo>`.
3. Replace the scheduled task's prompt with `templates/trigger-prompt.md`, filled in.

The first run creates `verify-<repo>` if it is missing.

## License

Apache-2.0 — see [LICENSE](LICENSE).
