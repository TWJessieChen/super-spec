# Final Review — <change name>

**Date:** <YYYY-MM-DD>
**Round:** <N>
**Reviewed range:** <baseline>..<REVIEWED_HEAD short hash>
**Focus range:** <same as reviewed range in round 1; prev_head..REVIEWED_HEAD in round ≥ 2>
**Reviewed HEAD:** <full hash of HEAD when the review ran — Phase 6 archives only if HEAD still equals this>
**Review depth:** <Quick | Standard | Deep> (<auto: reason | set in proposal.md>)
**Review rules:** <REVIEW.md @ <RULES_BASE short hash> | none at baseline> · change modifies review rules: <yes | no>
**Verification:** <`<full command>` → exit 0 | none defined>
**Run:** <`<run line>` → PASS (<k> scenarios) | could not run — <reason> | none — <reason>>
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

## Skeptic ledger
<only when skeptics voted (Deep native-verify, or the ultracode panel). One line per blocking candidate BEFORE the vote, including ones voted down. Omit otherwise.>
- [<original severity>] <file> — <description> {class: <kebab-case>} · lens: <lens or reviewer> · confirms: <0|1|2|unverified> · evidence: confirm L<1-4|–> / refute L<1-4|–> → <final severity>
<evidence = the highest `evidence_level` the skeptics reported on each side (1 claim · 2 line pointed at · 3 failure walked through · 4 ran something). Recorded only — the vote rules do not use it.>

## Review stats
- Tasks: <n> (<k> skipped)
- Passed review in one round: <k> / <n> (<pct>%)
- Review rounds per Task: avg <x.x>, max <m>
- Verification failures in Phase 4: <k>
- Final review round: <N>
- Skeptic verification: <candidates / verified / downgraded / moved to pre-existing, or "not run">
- Blocking issues by evidence: <L4: k · L3: k · L2: k · L1: k — counting each final [Critical]/[Important] at its confirm level; or "not run">
- Pre-existing items found: <k>
- Checks triggered: <ids from REVIEW.md @ RULES_BASE whose `when:` matched, e.g. A3, A8 | none>
- Observe hits: <observe:<id> notes reported by reviewers | none>
