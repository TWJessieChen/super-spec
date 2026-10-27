# REVIEW.md

<!--
Project review rules for /super-spec reviewers (task-reviewer, final-reviewer, panel lenses).
Put this file at the repo root or at openspec/REVIEW.md. Every reviewer reads it on every review,
so keep it short — link to longer docs instead of pasting them.

What belongs here: what failure looks like in THIS project, what to always check, which paths
are risky, and what to ignore. What does not belong here: general coding advice the model
already knows. When the same review comment keeps coming back, add it under "Always check"
(super-spec Phase 6 proposes these for you).
-->

## What failure looks like here
<!-- 3–6 bullets. Concrete, project-specific. Examples:
- A resource acquired from the platform (handle, id, registration) is not released on every exit path.
- Docs that describe behavior the code no longer has ("docs that lie").
- Config that silently routes to the wrong environment. -->
- 

## Always check
<!-- Project-specific checks reviewers should apply on top of their generic rubric.
     One line each, phrased as what to look for. -->
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
