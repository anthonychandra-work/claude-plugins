# File shapes

A mission is one folder, on one branch.

```
docs/missions/<slug>/
  intent.md
  spec.md
  state.md
  m1-<name>/
    plan.md
    validation.md
    proof.md
    issues.md
  m2-<name>/
```

A milestone folder is `m<n>-<name>`, taken from the intent's `### M<n> — <name>` heading in
lowercase kebab-case. `issues.md` exists only once an attempt has failed.

A scoped mission's folder is under its scope folder: `apps/shop/docs/missions/<slug>/` for the
scope folder `apps/shop`.

## state.md

```markdown
# Mission: csv-export

State: executing
Milestone: m2-download-link
Attempt: 1
Base: 4c1e9a2
Branch: mission/csv-export

| milestone | status |
| --------- | ------ |
| m1-export-job | passed |
| m2-download-link | executing |
| m3-link-expiry | pending |
```

Each list below is the full set of values. Nothing else is valid.

State: `scaffolding` · `research-and-planning` · `executing` · `done` · `blocked`

Milestone status: `pending` · `planning` · `executing` · `passed` · `blocked`

At most one milestone is `planning`, `executing` or `blocked`. `Milestone` and `Attempt` name it.
With every milestone `passed`, the state is `done` and both lines read `—`.

A scoped mission has one more line, after `Branch`: `Scope: apps/shop`, its scope folder. Its
branch line then reads `Branch: mission/apps/shop/csv-export`. A state file without a `Scope`
line belongs to a mission of the whole project.

## spec.md

```markdown
# Spec: Export invoices as CSV

From `intent.md`. Base `4c1e9a2`.

## Requirements

- R1 — An export holds one row per invoice in the chosen range. (D1, M1)
- R2 — A finished export is reachable for 24 hours, then gone. (D4, M3)

## Assumptions

- A1 — Amounts are written in the account's currency with two decimals; the intent names no
  format. (scaffolding)

## Milestones

### m1-export-job

Outcome: <from the intent, unchanged>

Done when: <from the intent, unchanged>

Tasks:

- <one change that leaves the project working>
- <...>

Files:

- `<path>` — new
- `<path>` — changed

### m2-download-link

<...>

## Overlap

- `<path>` — also touched by `mission/<other-slug>`
```

Every requirement names the decision and the milestone it comes from. An assumption names who made
it: `scaffolding`, or `planner` or `executor` with the milestone. Write `None.` under Overlap when
no file is shared. A path under Files is written from the repository's root, in a scoped mission
too.

The tasks here are an outline. The planner turns them into the checklist the executor ticks, and
may split or reorder them within the milestone's outcome.

## issues.md

```markdown
# Issues: m2-download-link

## Attempt 1 — validator

Failed: V3, V5

<the output that shows the failure>

Observed: <where the behaviour departs from what the check expects>

## Owner — 2026-03-14

<the user's direction, in their own words>
```

A failure entry is headed `## Attempt <n> — <who>`, where who is `validator`, `executor` or
`conductor`. An owner entry is headed `## Owner — <date>`. Entries are appended, never edited.

The attempt for a milestone is the number of failure entries after the latest owner entry, plus
one.

## What you read from the agents' files

The planner, the executor and the validator own `plan.md`, `validation.md` and `proof.md`. You
read three things from them:

- `Attempt: <n>` near the top of `plan.md` and of `proof.md`;
- the task checklist in `plan.md`, where `- [ ]` is open and `- [x]` is built;
- `Result: pass` or `Result: fail` near the top of `proof.md`.

In a scoped mission the proof's table starts with a row `S`, the validator's own check that no
file outside the scope folder changed. It counts toward `Result` like any other row.
