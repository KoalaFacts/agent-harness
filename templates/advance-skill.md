---
name: advance-<repo>
description: How the scheduled task advances <repo> — goal, headline metric, gate, work order, scope and project rules. Read with agent-harness/autonomous-dev at the start of every run.
---

# Advance <repo>

This skill is the repo-specific half of the scheduled development task; the run loop is `agent-harness/skills/autonomous-dev`. The task keeps this file current through its own PRs. It may make rules stricter, never looser, and never relaxes the hard limits in the scheduled task's prompt.

## Goal and metric
- What the repo is, in two lines.
- The goal, and the **headline metric** that shows progress (with how to measure it).
- What counts as real progress here (the "code PR" equivalent).

## Setup
- Toolchain and exact install commands (scratch folder, pinned versions).
- Known environment traps and their fixes.

## Gate
The exact local gate commands, in order. Warnings are errors unless stated.

## Ledger
Where it lives (an existing file or a section of one).

## Work order
Ordered items, each with id, what, why it matters for users. Split big ones into mergeable sub-steps.

## Authority
- Branch prefixes; merge method (default squash pinned to head SHA).
- First line of every PR body, e.g. "Opened by the <repo> scheduled task."
- Anything narrower than the defaults (others' PRs, Dependabot, adopting stale PRs).
- What merging already deploys, if anything.

## Project rules
Constraints specific to this repo (files never to touch, locked versions, licences, protected registries), extra merge conditions, and owner decisions the task must not take.

## Docs scope
Which existing files hold what; size budgets; what housekeeping checks here.

## Report extras
Anything the report must add for this repo.
