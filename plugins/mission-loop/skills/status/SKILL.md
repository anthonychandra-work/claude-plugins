---
name: status
description: Use when the user types /mission-loop:status, or asks where a mission stands, which missions are running, blocked or finished, or what is waiting on them. Read-only.
---

# status

Report where every mission stands. Read only, and cheap. Never a step toward doing the work.

## Read

- Every mission branch: `mission/<slug>` for a mission of the whole project, and
  `mission/<scope folder>/<slug>` for a scoped one, where the slug is the last part of the name.
  From each, its committed `state.md` in its mission folder: `docs/missions/<slug>/`, under the
  scope folder for a scoped mission. Read it from the branch; a mission needs no worktree to be
  reported.
- Whether each of those branches is already merged into the project's default branch.
- Every `intent.md` in a mission folder of the primary checkout that has no mission branch, and
  its status line.
- When the project's host has a command-line tool here, whether a `done` mission has an open pull
  request.

Nothing else. No specs, plans, proofs or source code.

A session reports its own scope. Its scope folder is the nearest folder, at or above the one the
session started in, that holds an instruction file, `CLAUDE.md`. Below the repository's root, the
session reads that scope's branches and intents alone. At the root it reads those of the whole
project and of every scope.

## Report

Three parts, in this order.

One line per mission: slug, its scope folder when it has one, state, the milestone it is on out of
how many, and the attempt. A blocked mission also gets the heading of its latest failure entry. A
merged mission is reported as merged.

One line per intent that has not started: slug, its scope folder when it has one, and `draft` or
`ready`.

Last, anything waiting on the user, named as an action they take: a blocked mission to give
direction on, a pull request to merge, a draft intent to finish. Nothing waiting: say so in those
words. A mission that only needs `/mission-loop:start` run again is not waiting on the user.

## Hold the line

The committed files are the source. Report what they say, and say that is what you did.

Uncommitted work in a worktree is not progress. Mention that it is there and leave it alone.

A state that looks wrong is worth naming as a doubt. Do not correct it and do not open the code to
settle it; the next `/mission-loop:start` reads the files and fixes the state itself.

Never continue into the work.
