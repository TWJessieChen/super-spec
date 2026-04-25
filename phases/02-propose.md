# Phase 2 — Propose

## HARD-GATE B — Mode Selection

Use AskUserQuestion to ask:
> "Choose mode for this change:
>  - **TDD**: write failing test first, then code (full discipline)
>  - **Simple**: skip new tests, but existing tests must still pass"

Record the answer.

## Derive change name

From the brainstorming conversation, propose a kebab-case name (e.g., "add user auth" → `add-user-auth`). Ask the user to confirm or suggest a different name.

## Create the change directory

Run: `openspec new change <name>`

## Write `proposal.md`

Path: `openspec/changes/<name>/proposal.md`

Structure: see `templates/proposal.md`. Fill in `## What` and `## Why` from the brainstorming conversation; set `## Mode` to the choice from HARD-GATE B.

## Write `design.md`

Path: `openspec/changes/<name>/design.md`

Structure: see `templates/design.md`. Fill in from the design content produced during brainstorming.

## Commit

Stage `openspec/changes/<name>/proposal.md` and `openspec/changes/<name>/design.md`. Commit message:
```
openspec(<name>): propose
```

→ Continue to Phase 3.
