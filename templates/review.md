# Final Review — <change name>

**Date:** <YYYY-MM-DD>
**Round:** <N>
**Reviewed range:** <baseline>..<REVIEWED_HEAD short hash>
**Focus range:** <same as reviewed range in round 1; prev_head..REVIEWED_HEAD in round ≥ 2>
**Reviewed HEAD:** <full hash of HEAD when the review ran — Phase 6 archives only if HEAD still equals this>
**Review depth:** <Quick | Standard | Deep> (<auto: reason | set in proposal.md>)
**Verification:** <`<full command>` → exit 0 | none defined>
**Verdict:** <APPROVED | CHANGES REQUESTED | NEEDS DESIGN UPDATE>

## Summary
<reviewer's summary>

## Closure
<round ≥ 2 only; omit in round 1>
- closed: <previous issue>
- open: <previous issue>

## Issues
<bullet list, severity-tagged, ordered [Critical] → [Important] → [Minor]; omitted if APPROVED. NEEDS DESIGN UPDATE uses the heading "## Design Issues".>
- [Critical] <file:line — issue> {class: <kebab-case>}
- [Important] <file:line — issue> {class: <kebab-case>}
- [Minor] <file:line — issue>

## Suggested revision tasks
<CHANGES REQUESTED only; "## Suggested resolutions" for NEEDS DESIGN UPDATE>
- <one line per [Critical] / [Important] issue>

## Notes
<only when Verdict = APPROVED and there are Minor items; skeptic-downgraded items keep their annotation>
- [Minor] <non-blocking observation>

## Pre-existing
<problems the change did not introduce; collected into openspec/backlog.md at archive. Omit if none.>
- [Pre-existing] <file:line — issue>

## Accepted open issues
<only when the user accepted remaining issues at the round-3 cap. Omit otherwise.>
- [Important] <issue> — accepted: <user's reason>

## Dispositions
<APPROVED only, when any item needed one (skeptic-downgraded or accepted). Omit otherwise.>
- <item> → <False positive | Intentional | Not important | Fix later>

## Review stats
- Tasks: <n> (<k> skipped)
- Passed review in one round: <k> / <n> (<pct>%)
- Review rounds per Task: avg <x.x>, max <m>
- Verification failures in Phase 4: <k>
- Final review round: <N>
- Skeptic verification: <candidates / verified / downgraded / moved to pre-existing, or "not run">
- Pre-existing items found: <k>
