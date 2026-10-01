---
name: status
description: Use when the user types /mission-loop:status, or asks where a mission stands, which missions are running, blocked or finished, or what is waiting on them. Read-only.
---

# status

Report where every mission stands. Read only, and cheap. Never a step toward doing the work.

## Read

- Every branch named `mission/<slug>`, and from each its committed `docs/missions/<slug>/state.md`.
  Read it from the branch; a mission needs no worktree to be reported.
- Whether each of those branches is already merged into the project's default branch.
- Every `docs/missions/<slug>/intent.md` in the primary checkout that has no mission branch, and
  its status line.
- When the project's host has a command-line tool here, whether a `done` mission has an open pull
  request.

Nothing else. No specs, plans, proofs or source code.

## Report

Three parts, in this order.

One line per mission: slug, state, the milestone it is on out of how many, and the attempt. A
blocked mission also gets the heading of its latest failure entry. A merged mission is reported as
merged.

One line per intent that has not started: slug, and `draft` or `ready`.

Last, anything waiting on the user, named as an action they take: a blocked mission to give
direction on, a pull request to merge, a draft intent to finish. Nothing waiting: say so in those
words. A mission that only needs `/mission-loop:start` run again is not waiting on the user.

## Hold the line

The committed files are the source. Report what they say, and say that is what you did.

Uncommitted work in a worktree is not progress. Mention that it is there and leave it alone.

A state that looks wrong is worth naming as a doubt. Do not correct it and do not open the code to
settle it; the next `/mission-loop:start` reads the files and fixes the state itself.

Never continue into the work.
