# Maintain verify — keep the verify skill and its feature map honest

**Trigger:** the user says "maintain verify", "audit the verify skill", "the feature map is out of date", or runs `/super-spec maintain-verify`. Also offered (once, one line) by `phases/05-final-review.md` §7 when a Run reported `NOT COVERED` scenarios or a step failed because the feature map was wrong. Runs outside any change: it does not need an `openspec/changes/<name>/` and is not part of the six phases.

**Why:** the Run gate is only as good as the verify skill behind it. A feature map rots as soon as the app changes, and a stale map makes Run drive the wrong path and still report PASS. This pass re-checks every feature from source and drives every feature live. (Adapted from pstack `/maintain-verification-skill`.)

## Outcome — say which one

- **clean** — every feature got source and live coverage; nothing to change. No commit.
- **changed** — one commit of proven corrections to the verify skill.
- **blocked** — coverage could not finish, or a proven fix could not land safely. Say exactly what blocked it.

## Edit scope

Only the verify skill's own folder (`.claude/skills/verify*/` — its SKILL.md and any scripts it owns). Never product code. A behavior the map describes that the app no longer has is either doc drift (fix the map) or a product regression (report it, and offer to record it in `openspec/backlog.md`; do not paper over it in the map).

## Pass

0. **Locate the target.** `ls .claude/skills/verify*/SKILL.md */.claude/skills/verify*/SKILL.md`. Several → ask which. None → stop and point at pre-flight §6 (create one from `examples/verify-skill.android/`). Tracked-file changes present (`git status --porcelain --untracked-files=no`) → stop and ask the user to commit or stash first.
1. **Index hygiene.** Read the skill. Check the PROJECT block is filled (no `<…>` placeholders left), every feature-map line has all three parts (feature → path → expected), and there are no duplicates. Fix those directly.
2. **Source wave.** One read-only Agent subagent (`model: sonnet`) per feature-map line, all in one message. Each gets the feature line and the repo, and returns exactly: `summary` (how the feature works, from source), `entry_points` (file:line of the screen / handler), `drift` (`none`, or what the map says vs. what the source does, with file:line), `recipe` (one concrete way to drive it live, with real resource-ids / texts from the layout files). Children never drive the app and never edit files.
3. **Reconcile.** Every feature line has a returned summary. Spot-check each cited drift by opening the cited lines yourself; do not re-prove clean claims. Then look for user-facing features missing from the map: `git log --since="<Last confirmed commit's date>" --name-only` for new activities / fragments / composables / manifest entries / deep links — a feature counts as missing only with a concrete source path.
4. **Live pass (required, even when source looks clean).** You own all driving; follow the skill's own Steps and Rules. One clean start, then drive every feature at least once, serially, merging features that share a screen into one walk where you can. Hold three rules the whole time:
   - Run the skill's **Doctor** before the first drive and after anything surprising; if Doctor fails, relaunch from the clean start before continuing — never drive an instance you have not health-checked since it last surprised you.
   - Evidence captured so far survives every relaunch and cleanup (check it is still in the evidence directory).
   - Nothing a drive started outlives it (force-stop, dismiss dialogs, remove test data you created).
   A feature that cannot be reached is `unreachable` only with the concrete prerequisite (login, entitlement, hardware, Android version) and the route attempted. A Doctor failure caused by the skill itself (wrong package name, stale ready marker) is drift: fix it under edit scope and retry once.
5. **Fix.** Apply only corrections proven by step 3 or 4: map lines, resource-ids, ready marker, Doctor commands, missing features (with the route you actually drove). Update `Last confirmed against source:` to `git rev-parse HEAD`.
6. **Commit** (only when **changed**). Delegate to a haiku Agent subagent (`你是被主 session 委派的子代理，可以直接用 Bash 執行 git 命令，不需要再往下委派。`; `git add` only the verify skill folder; no hook bypass; report stderr on failure): `chore(verify): refresh feature map (<k> fixed, <m> added)`. No push.

## Report

```
maintain-verify: clean | changed | blocked
features: <n> covered from source · <n> driven live · <k> unreachable (<prerequisite>)
drift fixed: <list, one line each — feature: was → now>
added: <features added to the map>
regressions (product, not fixed here): <list, or none>
commit: <hash | none>
```

Offer a weekly scheduled run once, if the user has none: the same pass, report only, no commit unless they approve the diff.
