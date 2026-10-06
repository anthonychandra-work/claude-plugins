---
name: start
description: Runs or resumes one mission from its ready intent to one pull request, with no stop in between. Runs only when the user types /mission-loop:start, with or without a slug.
argument-hint: "[slug] [direction for a blocked mission]"
disable-model-invocation: true
---

# start

Take one mission from a ready intent to an open pull request. Between the door check and the pull
request you ask the user nothing. The only other ending is a milestone that failed three times.

You are the conductor. You read what is committed, pick the next step, hand it to an agent, and
record the result. Planning, building and validating each run in an agent of their own, so none of
them carries another's context.

Arguments: `$ARGUMENTS`. The first word is the slug. Anything after it is the user's direction for
a blocked mission. Empty means no slug was given.

## Scope

A project can keep missions for one part of itself beside the missions of the whole. The folder
the session started in decides which kind this run takes.

Walk up from that folder, inside its checkout, to the nearest folder that holds an instruction
file, `CLAUDE.md`. At the checkout's root, or with none found below it, the mission is of the
whole project and every name in this skill is as written. Otherwise that folder is the scope
folder, written as its path from the root, and the mission is scoped:

| | whole project | scope folder `apps/shop` |
| - | ------------- | ------------------------ |
| mission folder | `docs/missions/<slug>/` | `apps/shop/docs/missions/<slug>/` |
| branch | `mission/<slug>` | `mission/apps/shop/<slug>` |
| worktree | `.worktrees/<slug>` | `.worktrees/apps/shop/<slug>` |

Where this skill names one of the three, read it for the mission's scope. The worktree is under
the primary checkout's root in both.

A scoped mission commits files under its scope folder and nowhere else: its documents in its
mission folder, its code beside them. A step that would need a file outside is a failure entry,
never a wider scope.

A session runs the missions of its own scope alone. A slug found only in another scope: say which
folder to start the session in, and stop.

## Pick the mission

A slug was given: that mission.

No slug: list the candidates. A candidate is a mission branch of this scope whose state is not
`done`, or an intent of this scope in the primary checkout marked `Status: ready` with no mission
branch yet. Exactly one: take it. More than one: list each with its state and stop. None: say so
and stop. Never choose between two.

Before you act, say what you are about to do in one line: mission, milestone, state, attempt.
`csv-export: m2 of 3, executing, attempt 1, task 3 of 5.`

## Door check

Run it once, when the mission has no branch yet. The intent is `intent.md` in the mission folder
of the primary checkout. It passes when:

- its status line reads `Status: ready`;
- it has the five sections Goals, Decisions, Scope, Milestones and Boundaries, each filled in, with
  no placeholder text left from the template;
- every milestone has an `Outcome` and a `Done when`;
- no other mission's branch name starts with this mission's followed by `/`, and this mission's
  starts with no other's: git cannot hold both.

If any part fails, name each one and stop. Do not repair the intent. That is a conversation the
user has through `/mission-loop:intent`.

## Where the work happens

The primary checkout is the project's main working copy, the first entry of `git worktree list`.
The mission has one branch and one worktree, named as Scope gives them.

- Everything the mission writes, documents and code, goes in the worktree and is committed on the
  branch.
- The session's working directory is in the primary checkout, so a relative path lands in the
  wrong copy. Use the worktree's absolute path for every file and every command, and give agents
  the same.
- Never switch, rebase, reset or commit in the primary checkout.
- The branch exists and the worktree is gone: add the worktree back from the branch and prepare it
  again. The commits are intact.
- The worktree holds uncommitted changes when you arrive: they belong to a step that was cut off.
  Stash them with a message naming the milestone and the step, then redo that step. Never build on
  them.

## Read the state

Every run starts here, a first run and a resumed one alike. Read what is committed. `state.md` is
the index; the committed files are the evidence, and when the two disagree the files win and you
rewrite `state.md` to match.

| Found | Do |
| ----- | -- |
| no mission branch | door check, then scaffolding |
| the branch, without a committed `spec.md` and `state.md` | scaffolding |
| state `blocked` | see Blocked |
| every milestone `passed` | close |
| anything else | the loop, on the first milestone that is not `passed` |

Inside that milestone, the attempt is the number of failure entries in its `issues.md` since the
latest owner entry, plus one.

| Found in the milestone folder | Do |
| ----------------------------- | -- |
| no `plan.md` or no `validation.md`, or a plan whose `Attempt` is lower than the attempt | plan |
| a plan with unticked tasks | build |
| every task ticked, and no proof or a proof whose `Attempt` is lower | validate |
| a proof for this attempt with `Result: pass` | mark the milestone `passed`, take the next one |
| three failure entries since the latest owner entry | blocked |

## Scaffolding

Yours to do, once per mission.

1. **Base.** Fetch. The base is the remote's default branch, unless the project's instructions name
   another. A project with no remote uses the primary checkout's current commit. Create the branch
   and the worktree from the base. If git does not already ignore `.worktrees/`, add it to the
   repository's local exclude file.
2. **Intent.** If the worktree's copy of the intent is missing or differs from the primary
   checkout's, copy it in and commit it. If the original is untracked, delete it once that commit
   exists, so the eventual merge does not collide with it. The worktree's copy is the intent from
   here on.
3. **Prepare.** Make the worktree runnable the way the project's instructions say: install
   dependencies, generate what the build needs.
4. **Spec.** Read the intent, the project's instructions, and the code the milestones concern.
   Write `spec.md`: the requirements, the assumptions, and for each milestone its tasks and the
   files it will touch. Keep the intent's milestones in their order, with their names, outcomes and
   `Done when`. In a scoped mission every file is under the scope folder.
5. **Overlap.** For every other mission branch not yet merged into the base, of any scope, read
   the file lists in its spec. Record each file both missions touch under Overlap. This never
   stops the run.
6. **State.** Write `state.md` with every milestone `pending`, and with its `Scope` line in a
   scoped mission. Commit `spec.md` and `state.md` together.

The spec decides nothing the intent already decided. Where the intent is silent, decide, and record
the choice under Assumptions with your reason.

## The loop

One milestone at a time, in order. A milestone is planned only after the one before it has passed,
because its plan is researched against the code the earlier milestones left behind.

**Plan.** Set the state to `research-and-planning`. Dispatch `mission-loop:planner`. It commits
`plan.md` and `validation.md`.

**Build.** Set the state to `executing`. Dispatch `mission-loop:executor`. It commits once per
task and ticks the task in the same commit.

- Every task ticked: validate.
- It added a failure entry to `issues.md`: the attempt failed.
- Tasks left unticked and no failure entry: it was cut off. Dispatch it again; this uses no
  attempt.

**Validate.** Dispatch `mission-loop:validator`. It commits `proof.md`.

- `Result: pass`: mark the milestone `passed` and take the next one.
- `Result: fail`: the validator has added a failure entry.

**After a failed attempt.** Fewer than three failure entries: plan again. The planner reads
`issues.md` and revises the plan. Three: blocked.

### Dispatching

Give an agent four facts and nothing else: the worktree's absolute path, the slug, the milestone
folder as its path from the worktree's root, and the attempt number. A scoped mission adds a
fifth, the scope folder. Do not describe the work, summarise an earlier agent's report, or say
what you expect to find. The files carry all of that, and an agent handed a conclusion returns
it confirmed. The validator in particular is told nothing about how the build went.

When an agent returns, check that the file it owes is committed. If it is not, stash what it left
and dispatch it again. A second miss on the same step is a failed attempt: add the failure entry
yourself, signed `conductor`.

## Blocked

Set the state to `blocked` and commit it. Report the milestone, each failure entry in one line, and
the worktree path. Push nothing. Stop.

A blocked mission moves again only on the user's direction. Run again with no direction after the
slug: report the block and stop. With direction: append it to the milestone's `issues.md` as an
owner entry in the user's own words, commit it, and plan again. The attempts start over from that
entry.

## Close

When every milestone has passed:

1. Set the state to `done` and commit it.
2. Push the branch. This is the mission's only push.
3. Open one pull request against the base, following the project's conventions for pull requests.
   Its description gives the goals in two lines, each milestone with its result and the path to its
   proof, every assumption, and every overlap. For a scoped mission it names the scope folder.
4. Report the pull request, the assumptions, the overlaps, and any stash you left behind.

Never merge it. With no remote, or no way to open a pull request from here, stop after the commit
and say that the branch is ready and why there is no pull request.

## Rules that hold throughout

- Only you write `state.md`, and every change to it is its own commit.
- A step is finished when its file is committed, and not before.
- The intent is never edited once scaffolding begins.
- The project's instructions outrank this skill wherever they speak: its commands, its commit and
  pull-request conventions, its base branch, its hooks. Where it has no commit convention, write
  `<type>(<slug>): <what changed>`.
- A hook or a project rule that refuses a write is obeyed by changing the content. Never route
  around it.
- Nothing is pushed before close.
- Planning, building and validating happen in the agents. Do none of them in your own context.

## Shapes

Read `references/file-shapes.md` before writing `spec.md`, `state.md` or an entry in `issues.md`.

## Stop only if

- The door check fails, or there is more than one candidate and no slug.
- The mission is blocked.
- The worktree or the branch cannot be created.
- Going on would need a change in the primary checkout, a rewrite of commits already made, or
  anything outside the worktree that cannot be undone.

Nothing else is a reason to stop or to ask. A missing decision becomes an assumption. A failing
check becomes a failure entry.
