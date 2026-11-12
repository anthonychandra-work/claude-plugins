---
name: setup
description: Use when the user types /mission-loop:setup, or asks to set a project up for missions, or to write or refresh the project's instruction file. Writes CLAUDE.md where the project has no instruction file, and a private CLAUDE.local.md beside the ones it has.
---

# setup

Give every session in this project the context it works from, in seven sections: what the project
is, how work is done in it, which skills to load, where its knowledge is, its commands, its gates,
and what goes wrong here. Run once per project, and again to bring the file up to date.

## Which file

The folder is the one the session started in. Look for an instruction file, `CLAUDE.md` or
`AGENTS.md`, in that folder and in every folder above it, up to the checkout's root.

| Found | Write | Check against |
| ----- | ----- | ------------- |
| a file in the session's folder that this skill wrote | that file, updated | every other instruction file found |
| no instruction file | `CLAUDE.md` in the session's folder | nothing |
| one or more instruction files | `CLAUDE.local.md` in the session's folder | each one found |

A file this skill wrote starts with the line `<!-- mission-loop:setup -->`. A `CLAUDE.local.md`
in a folder above loads into the same session, so check against it too.

`CLAUDE.local.md` is the user's own. Claude Code loads it beside `CLAUDE.md` in every session
started in its folder or below, and git never carries it to anyone else. The instruction files
already in the project belong to whoever shares it, and this skill leaves them as they are.

Before anything else, say in one line which file you are about to write and why. The user can name
another: to complete a `CLAUDE.md` of their own that this skill did not write, or to write a
`CLAUDE.md` in a folder below one that has an instruction file. Follow that. In a file this skill
did not write, add the sections it lacks and change nothing it already says.

A `CLAUDE.md` in a folder below the root makes that folder a mission scope. Say so before you
write one there. A `CLAUDE.local.md` makes no scope.

## Explore

Read before you ask. In the checkout:

- every instruction file found above, and each file they point to;
- the readme, the contribution guide and the pull-request template;
- the scripts the project defines, its build files, its lockfile, and what its continuous
  integration runs and requires;
- the folders that hold documents: wikis, decisions, rules, research, missions;
- the recent history: how commits are written, how branches are named, which branch is the default;
- the skills and agents this session has, from the project, the user and plugins.

## The seven sections

Use `instruction-format.md` beside this file. Take each section in turn and settle it one of three
ways.

- **Covered.** An instruction file you are checking against already says it. Leave the section
  out. Do not repeat it and do not reword it: both files load into the same session.
- **Found.** The project shows it. Draft the section from what you read.
- **Not found.** Ask the user. Put every question in one batch, each with the answer you
  recommend, worded so that "yes" accepts it.

| Section | Draft it from | Ask the user |
| ------- | ------------- | ------------ |
| Project context | the readme, the layout, the build files | to correct what you misread |
| Working conventions | the instruction files, the contribution guide, the history | what the project does but has not written down |
| Skills | the skills and agents this session has | which of them matter here, and for what |
| Knowledge map | the document folders | which questions they answer |
| Commands | the scripts, the build files, continuous integration | to confirm the command that makes a fresh checkout runnable |
| Gates | what continuous integration requires | which gate must be green, and which is judged against a baseline |
| Things Claude gets wrong | nothing; it starts with its standing line alone | nothing |

Write a command only when a script, a build file, continuous integration or a document shows it.
Where a document already answers a question, point to it and do not copy it.

## In a `CLAUDE.local.md`

- Its first line after the heading says that the project's own instruction files win wherever the
  two differ.
- Working conventions says how the user works here: work enters as a mission through
  `/mission-loop:intent`, the user types `/mission-loop:start`, and the mission's folder is pushed
  with its pull request. Add what the project does but has left unwritten.
- Compare the user's way of working with what the project's files require. Show the user each
  point where they differ, and write none of them into the file as though it were settled. Two
  names cannot bend: a mission's branch is `mission/<slug>` and its folder is under
  `docs/missions/`. When the project requires otherwise, report it as a conflict for the user to
  take to the project.
- If git does not already ignore `CLAUDE.local.md`, add it to the repository's local exclude file.
  Never edit `.gitignore`. If that write is refused, give the user the one command that adds the
  line, and say that git will pick the file up until they run it.

## Ready for a mission

A mission run reads three facts from the instruction files and assumes them when they are missing:

- the command that makes a fresh checkout or worktree runnable;
- the commands for the project's gates: tests, type checks, lint, build;
- how commits and pull requests are written.

Check that the instruction files, the one you are writing included, answer all three. Ask for any
that is missing and write it in. One the user cannot answer: say so in the report.

## Draft, then write

Show the user the whole file as it will be written, and what you left out as covered. Write it
when they accept it.

On a later run, check every section against the instruction files again. Drop a section they now
cover, update a section the project has moved past, and leave every line the user added.

Leave the file uncommitted. Committing a `CLAUDE.md` is the user's decision.

## Report

The file written, the sections in it, the sections left out as covered, each conflict, and any of
the three facts still missing.

## Stop and ask if

- You are about to write a `CLAUDE.md` in or below a folder that has an instruction file, and the
  user did not ask for it.
- You are about to change a line in a file this skill did not write.
- You are about to write a command you did not find in the project.
- You are about to write a convention the project's own files contradict.
