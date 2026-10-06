# claude-plugins

Plugins for [Claude Code](https://claude.com/claude-code), published as one marketplace named `ac`.

## Install

```shell
/plugin marketplace add anthonychandra-work/claude-plugins
/plugin install mission-loop@ac
```

## mission-loop

Takes a piece of work from a written intent to one pull request without stopping to ask. You
decide everything in a conversation that ends in an intent; the run then writes a spec, and plans,
builds and validates each milestone in turn.

It works in any git project. It reads the project's own instructions for how to test, lint, build
and commit, and carries no knowledge of any stack.

### Commands

`/mission-loop:intent` turns a conversation into an intent with five sections: goals, decisions,
scope, milestones and boundaries. It starts as a draft, and you mark it ready. That is the only
approval a mission gets.

`/mission-loop:start [slug]` runs a mission, or resumes one. With no slug it takes the single
mission that is ready or unfinished; with more than one it lists them and stops. It refuses an
intent that is not ready or is missing a section.

`/mission-loop:status` reports every mission's state, milestone and attempt, and what is waiting on
you. It changes nothing.

### A mission for one part of a project

A project that keeps an instruction file, `CLAUDE.md`, in a folder below its root can keep
missions for that part beside the missions of the whole. The folder a session starts in decides
which: started in `apps/shop/` or below it, the three commands work on that scope, and started at
the root they work on the whole project.

| | whole project | scope folder `apps/shop` |
|---|---|---|
| mission folder | `docs/missions/<slug>/` | `apps/shop/docs/missions/<slug>/` |
| branch | `mission/<slug>` | `mission/apps/shop/<slug>` |
| worktree | `.worktrees/<slug>` | `.worktrees/apps/shop/<slug>` |

A scoped mission commits files under its scope folder and nowhere else. The validator checks that
on every milestone, and a file outside fails the milestone. Work that needs a change outside is a
mission of the whole project, started at the root.

### A run

1. **Scaffolding.** The mission gets one branch and one worktree. The spec lists the requirements,
   the tasks and the files for each milestone, every assumption made where the intent was silent,
   and every file shared with another unfinished mission.
2. **Research-and-planning.** One milestone at a time, a planner researches the code as it stands
   and writes the plan and the validation list. A milestone is planned only after the one before
   it has passed.
3. **Executing.** An executor builds the plan one task and one commit at a time. A separate
   validator, told nothing about the build, runs the validation list and writes the proof with the
   output of every check.

A failed validation is recorded and sent back to planning. A milestone gets its first attempt and
two retries; after the third failure the mission is blocked and reports to you. Running
`/mission-loop:start <slug>` followed by your direction unblocks it.

When every milestone has passed, the branch is pushed once and one pull request is opened, listing
the assumptions and the overlaps. You merge it.

### What a mission leaves on disk

```
docs/missions/<slug>/
  intent.md          what is wanted; closed once the run starts
  spec.md            requirements, assumptions, tasks and files per milestone
  state.md           where the run is
  m1-<name>/
    plan.md          findings and the task checklist
    validation.md    the checks, written before the code
    proof.md         each check with its output
    issues.md        one entry per failed attempt
```

All of it is committed on the mission's branch and arrives with the pull request. A scoped
mission's folder is under its scope folder.

### Resuming

A run keeps nothing in the session. A step counts as done when its file is committed, so a session
that crashes, stops or is lost resumes with the same command: it reads the committed files and
continues from the first step that is not done. Work left uncommitted by the lost session is
stashed and that step is redone.

### Running unattended

The run asks no questions, but Claude Code still asks for permission to edit files and run
commands unless the session's permission mode allows them. Start the session in a mode that does.

A scoped mission's worktree is outside the folder its session started in. A mode that accepts
edits in the session's working folders still asks there, so add `.worktrees/` as a working folder
of that session.

Run one session per mission. Two missions can run at once in two sessions; two sessions on the same
mission write over each other.
