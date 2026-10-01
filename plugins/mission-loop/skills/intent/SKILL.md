---
name: intent
description: Use when the user wants to capture a feature, change, fix or any other piece of work as a mission intent, asks to write or revise an intent, or types /mission-loop:intent. Produces docs/missions/<slug>/intent.md, the only input a mission run reads.
argument-hint: "[slug]"
---

# intent

Turn a conversation into `docs/missions/<slug>/intent.md`. A mission run builds from this file
alone and never stops to ask a question, so every decision the build needs has to be in it before
it is marked ready.

## Before the conversation

Read the project's own instructions, and as much of the project as your questions need. Ask the
user only what the project cannot tell you.

Look for the mission first: a folder under `docs/missions/`, a branch named `mission/<slug>`, a
worktree under `.worktrees/`. If one exists, show the user what it already holds before asking
anything new.

Agree a slug: short, kebab-case, named after the outcome.

## The conversation

Let the user describe what they want in their own words. Then work out what is missing and ask for
all of it in one batch. One question per message makes them hold the whole mission in their head
for a dozen replies.

Test every milestone with one question: could an agent build this without asking anything? Each
question it would have to ask is a decision the intent is missing. Ask it now, because nobody
answers it later.

When the user leaves a choice to you, make it, write it under Decisions, and tell them you did.

Write what is wanted and what constrains it. Leave the task list and the files to the run, which
works them out against the code.

## The file

Use `intent-template.md` beside this file. All five sections are required.

- **Goals** — who is better off and how, and what is true when the mission is done.
- **Decisions** — every choice already made, written as a fact, numbered.
- **Scope** — what the mission changes, and what it leaves alone.
- **Milestones** — in build order. Each has an outcome and a `Done when` that someone can run or
  observe. A milestone that cannot be checked until a later one is built is not a milestone; merge
  it or cut it differently.
- **Boundaries** — what the run must not touch or change.

Write the file in the project's main working copy. Leave it uncommitted unless the project's
instructions say otherwise; the run carries it onto the mission branch.

## Status

Write `Status: draft`. Read the intent back to the user and correct what you misread.

Change it to `Status: ready` only when the user says it is ready. That word is the only approval a
mission gets.

Once a run has started on an intent, the intent is closed. A change of mind after that is a new
intent.

## Stop and ask if

- You are about to mark an intent ready without the user saying so.
- You are about to write a milestone with no `Done when`, or one nobody could check.
- You are about to leave a section empty because the user did not mention it.
- You are about to write a decision the user never made and did not leave to you.
