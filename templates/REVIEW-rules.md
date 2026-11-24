# REVIEW.md

<!--
Project review rules for /super-spec reviewers (task-reviewer, final-reviewer, panel lenses).
Put this file at the repo root or at openspec/REVIEW.md. Every reviewer reads it on every review,
so keep it short — link to longer docs instead of pasting them.

What belongs here: what failure looks like in THIS project, what to always check, which paths
are risky, and what to ignore. What does not belong here: general coding advice the model
already knows, and anything a compiler, linter or test can enforce — a text rule only works when
a reviewer reads it, so mechanical checks belong in code (super-spec's flows/enforcement-ladder.md).
When the same review comment keeps coming back, super-spec first tries to make it impossible
(structure / type / lint / test) and only then proposes a check here — new checks start under "Observe".

Reviewers read this file from the commit BEFORE the change started, never from the change itself:
editing it inside a change only takes effect from the next change, and the edit is itself reviewed.
-->

## What failure looks like here
<!-- 3–6 bullets. Concrete, project-specific. Examples:
- A resource acquired from the platform (handle, id, registration) is not released on every exit path.
- Docs that describe behavior the code no longer has ("docs that lie").
- Config that silently routes to the wrong environment. -->
- 

## Always check
<!-- Project-specific checks reviewers apply on top of their generic rubric. One line each:
       - [C<n>] when: <globs and/or diff keywords, comma-separated> → check: <what to look for> (source: <escape case, PR or incident>) · enforced_by: reviewer
     `enforced_by: reviewer` says nothing but this text enforces it; `reviewer (pending: <backlog heading>)` says
     a stronger check (lint, test, type, structure) is queued in openspec/backlog.md and this line is an interim
     watch. When a check here is violated again, that is a repeat: the next fix must be a stronger level.
     Reviewers apply an entry only when its `when:` matches the diff (a glob matches a changed path, a keyword
     appears in an added or removed line), so a long list stays cheap and quiet. An entry without `when:` applies
     to every change — keep those few. `source:` says why the check exists; a check nobody can explain is a
     candidate for deletion. Example:
       - [C1] when: **/*Service*.kt, bindService, registerReceiver → check: every bind/register has its unbind/unregister on every teardown path (source: escape 2026-10-03-player-leak) · enforced_by: reviewer -->
- 

## Observe
<!-- Trial checks: same format as "Always check". Reviewers apply them but a hit is only ever a
     non-blocking note (`[Minor] observe:<id> …`), so you can see what a rule would flag before it can block
     anyone. Promote an entry to "Always check" once its hits have been real problems; delete it if they were
     noise. New checks proposed by super-spec land here first. -->
- 

## Enforced elsewhere
<!-- The rule table: classes that used to be review checks and are now enforced by code. Reviewers do NOT
     apply these (the machine already does); the list exists so everyone can see who enforces what, and so a
     class is not re-added as text. One line each:
       - <class> → <structure:… | type:… | lint:<rule id> | test:<path>> (since <change name>)
     Delete a line only when the mistake can no longer happen at all (e.g. the API it was about is gone). -->
- 

## Sensitive paths
<!-- Globs. A change touching any of these is reviewed at Deep depth regardless of size:
     full multi-lens panel (ultracode) or skeptic verification (native). -->
sensitive_paths:
- `.github/workflows/**`
- 

## Quick paths
<!-- Globs that only ever need a Quick review when a change touches nothing else.
     Documentation (*.md, *.txt, *.rst, docs/**) is already Quick by default. -->
quick_paths:
- 

## Skip
<!-- Globs reviewers do not review at all (generated code, vendored deps, lockfiles). -->
skip_paths:
- 

## Noise
- At most 5 `[Minor]` items per review.
- Style or naming comments only when they break a rule written here or in CLAUDE.md.
- Every finding cites `file:line`. No evidence, no finding.
- Summaries lead with the count of blocking issues; when there are none, say so in one line.
