---
name: super-spec
description: Spec-driven workflow combining superpowers (brainstorm/plan/subagent/TDD/review) with openspec (lifecycle). Triggered by /super-spec, or when user wants a structured multi-phase workflow for a non-trivial change with subagent isolation, per-task review, and final holistic review before archive.
---

# /super-spec

Single-entry workflow for a non-trivial change. End-to-end:

```
brainstorm → propose → plan → apply (per-task subagent + spec/quality review) → final review → archive
```

All artifacts live in `openspec/changes/<name>/`. Defaults: TDD mode is offered, no merge / no push / no PR.

---

## ⚠ EXECUTION MODEL — read this first

**This file is an index, not the workflow itself.** It contains zero implementation details. Every phase, flow, and template lives in a sibling file. Your job as orchestrator is **mechanical dispatch**:

1. Identify which sibling file applies to the current step (use the indices below).
2. Use the **Read tool** to load that file into your context.
3. Follow it **literally**. Do not summarize, paraphrase, or skip steps.
4. Move to the next step only when the current file says you are done.

### NON-NEGOTIABLE RULES

- **NEVER execute a phase or flow from memory.** You will hallucinate details. The sibling file is the source of truth — Read it every time you enter that step, even if you ran it five minutes ago.
- **NEVER infer the contents of a referenced file.** If this file mentions `phases/04-apply.md`, you MUST Read `phases/04-apply.md` before doing anything Phase-4-related. The name is not a description.
- **NEVER batch Reads ahead of time** to "prepare." Read each file at the moment you enter its step, so the freshest content drives execution.
- **If a Read fails** (file missing, permission denied), halt and report the path. Do NOT improvise the missing logic.

A correct invocation of this skill produces a long sequence of `Read tool` calls — one per phase/flow entry, plus per-template lookups. If you find yourself executing without Reading, **stop and Read.**

---

## Invocation

User runs `/super-spec [<name | short description>]`.

1. **Read `flows/pre-flight.md` and execute it.** Halt on any failure. (No exceptions, no shortcuts — this runs every invocation.)
2. Then decide based on argument:
   - **No argument** → ask the user "What change do you want to work on? Describe what to build or fix."
   - **`<name>` matches existing `openspec/changes/<name>/`** → **Read `flows/resume-detection.md` and execute it**, then jump to the phase it tells you.
   - **`<name>` is new, or only a description was given** → enter Phase 1 (which means: Read `phases/01-brainstorm.md` and execute it).

---

## Phase index

Phases run sequentially unless a flow file directs otherwise. **Each row below is an instruction: when you enter the phase, Read the file and execute it line-by-line.** The file is the spec; this row is just a pointer.

| Phase | Action |
|---|---|
| 1. Brainstorm | **Read `phases/01-brainstorm.md` and execute it.** |
| 2. Propose | **Read `phases/02-propose.md` and execute it.** |
| 3. Plan | **Read `phases/03-plan.md` and execute it.** |
| 4. Apply (per-Task loop) | **Read `phases/04-apply.md` and execute it.** Re-Read at the top of every Task iteration — do not loop from memory. |
| 5. Final Review | **Read `phases/05-final-review.md` and execute it.** |
| 6. Archive | **Read `phases/06-archive.md` and execute it.** |

## Flow index (non-linear)

These are not part of the linear sequence. Enter only when the listed trigger fires; when you do, **Read the file and execute it**.

| Trigger | Action |
|---|---|
| Every invocation start (mandatory) | **Read `flows/pre-flight.md` and execute it.** |
| Resuming an existing change | **Read `flows/resume-detection.md` and execute it.** |
| Phase 5 verdict = `NEEDS DESIGN UPDATE` | **Read `flows/recover.md` and execute it.** |
| User says "abort" / "stop this change" | **Read `flows/abort.md` and execute it.** |

## Template index

Phase files instruct you to write artifacts (`proposal.md`, `design.md`, `review.md`, etc.). When you do, **Read the matching template file first** to get its exact structure, then fill it in. Do not invent the structure from memory.

| Artifact | Action |
|---|---|
| `proposal.md` body | **Read `templates/proposal.md`** for the exact section structure. |
| `design.md` body | **Read `templates/design.md`** for the exact section structure. |
| `review.md` body | **Read `templates/review.md`** for the exact section structure. |
| `## Revisions` block (Recover flow) | **Read `templates/revision-block.md`** for the exact section structure. |

---

## Context Isolation Rules (NON-NEGOTIABLE)

These rules deliver the actual context isolation between phases. Do not relax them. They apply to every Phase-4 dispatch and to Phase 5.

1. **Subagent dispatch always uses the Agent tool, never the Skill tool.**
   The Skill tool runs in the current context (no isolation). The Agent tool spawns a fresh context window (full isolation). Skill tool is only used for invoking other skills' interactive logic in the orchestrator's own context (e.g., `superpowers:brainstorming`, `superpowers:writing-plans`, `/simplify` from inside the final-reviewer).

2. **Every subagent prompt is constructed from the corresponding `prompts/*.md` template.**
   **Read the template at dispatch time** (not from memory), substitute the variables, pass it verbatim as the Agent prompt. Do not freehand. Do not append additional context unless a `{...}` placeholder explicitly invites it. The shared `prompts/_isolation-preamble.md` is referenced by every per-role prompt — do not strip it.

3. **Reviewers see git, not implementer narrative.**
   The implementer's response to the orchestrator is one line: `Done: <commit_hash>`. Reviewer prompts contain only the commit hash; reviewers fetch the diff via `git show <hash>` themselves. **Never** paste implementer narrative into a reviewer prompt.

4. **Per-task fresh subagent.**
   Never reuse a subagent across Tasks. Each Task gets a fresh implementer + fresh spec-reviewer + fresh quality-reviewer. Each dispatch is a new Agent tool call.

5. **Strip orchestrator context.**
   The orchestrator's own context contains the brainstorming conversation, all prior Task outputs, all reviewer reports. None of this should appear in a subagent's prompt unless the relevant template explicitly includes a placeholder for it.

6. **Final-reviewer is also fresh.**
   The final-reviewer reads files from disk and runs git commands itself. It receives no per-task review summaries; it forms its own holistic view from `proposal.md + design.md + tasks.md + git diff`.

---

## Dependencies

Documented in `README.md`. Pre-flight only checks the openspec CLI binary; plugin presence is not pre-checked. If a `Skill` invocation fails because a plugin is missing, output:

```
Skill `<name>` not found. This workflow depends on the `superpowers` and `opsx` plugins.
Install via Claude Code plugin manager.
```

then halt.
