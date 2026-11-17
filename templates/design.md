# <Change Name> — Design

## Goal
<one sentence>

## Approach
<2-3 sentences>

## Key Decisions
- <design choice with brief rationale>
- ...

## File Structure
- <which files will be created / modified, with one-line responsibility>
- ...

## Verification
<!-- Commands the orchestrator runs itself (machine gates, before any reviewer sees the code).
     Use exact, copy-pasteable commands run from the repo root. Write `(none)` when no such command exists. -->
- **Per-task:** `<fast command: targeted unit tests / lint / compile of the touched module>` — <what it covers>
- **Full:** `<complete command: full build + full test suite>` — <what it covers>
- **Run:** `<command, or skill:verify>` — <which spec-delta scenarios it drives end to end, or `(none): <why the app cannot be run here>`>
<!-- Run = start the real app / service and drive the user-facing flows the spec deltas describe, like a QA
     engineer would (emulator or device via adb / an Android MCP, a browser via CDP, a simulator, an API harness
     against a locally started service). It must print a per-step PASS/FAIL trace and leave evidence
     (screenshots, logs, responses). `skill:<name>` means "invoke the project's `.claude/skills/<name>/` skill".
     Unit tests do not count as Run. -->

## Out of Scope
- <bullet list>
