# Lessons — record, import and harvest platform lessons

**Trigger:** `/super-spec lessons [record | import | harvest | push]`, or the user says "record this as a platform lesson", "import lessons", "harvest lessons", "push / upload lessons". Also entered from `flows/enforcement-ladder.md` step 5 (record), and from `flows/pre-flight.md` §7 (import). With no sub-command, ask which of the four.

**Why:** a mistake one Android (or Unity, or AOSP…) project paid for should not be paid for again by the next one. Project rules (`REVIEW.md`, lint, tests) stay with their project; the platform part of the lesson is copied into this skill's `lessons/` folder, which travels with super-spec to every machine and every new project. Format and rules: `lessons/README.md` — Read it before any of the sections below.

`<skill>` below means this skill's own directory (where this file lives). `<project>` is the current repo.

## 1. Detect platforms (record, import, harvest)

For each `<skill>/lessons/*.md`, read its frontmatter `detect` globs and match them against `git ls-files` of `<project>`. `_common` always matches. Show the result in one line: `Platforms: android, _common`. The user may add or remove one; use their list.

## 2. Record

Input: one or more candidates — a class handed over by `flows/enforcement-ladder.md` (with its evidence and the level it chose), an escape case file, or the user's own description of something that went wrong.

For each candidate:

1. **Is it a platform lesson?** Apply the test in `lessons/README.md`. If it is about this project's own code or APIs, say so in one line and stop for this candidate (it stays in the project's `REVIEW.md` / backlog).
2. **Find an existing entry.** Search the chosen platform file(s) for the same class, or an overlapping `when:` / symptom. If one exists, this is an **update**: add 1 to the project count only when this is a project the entry has not counted before (compare the evidence text; ask if unsure), raise `enforce` if this project reached a stronger level, set `proved_red: yes` if a red proof ran, set `status: active` once the count reaches 2 or it is proved red, and set `updated:` to today.
3. **Draft** the entry in the README format (new id = highest existing number + 1). Write it generically:
   - `when:` with platform globs / API names, not this project's paths;
   - `symptom` / `root_cause` in platform terms;
   - `review_check` reusing the project's check text if one exists, with project names removed;
   - an artifact block only when it is short and generic (a lint rule skeleton, a test pattern) — never project code.
   New entries start as `status: observe` unless the evidence is an escape that was proved red, or the user says it has hit more than one project.
4. **Privacy pass.** Remove anything on the README's privacy list. Then show the user the exact entry text and ask (AskUserQuestion): **Write it** / **Edit first** (free text) / **Skip**. Nothing is written without this confirmation.
5. **Write** it into `<skill>/lessons/<platform>.md` (replacing `(no lessons recorded yet)` on the first entry), keeping entries in id order.

After all candidates:

- If `<skill>` is a git working tree (`git -C <skill> rev-parse --is-inside-work-tree`), commit only the lesson files there: `git -C <skill> add lessons/ && git -C <skill> commit -m "lessons(<platforms>): <classes>"`. Then ask once: **Upload now** (→ run §5) / **Later** (→ say: `run /super-spec lessons push when you want to upload`). Never push without that answer, and never with a plain `git push` — §5 is the only upload path, because it merges with what other machines uploaded.
- If it is not a git tree (an installed copy), say where the file was written and that `/super-spec lessons push` can upload it (§5 works without a local clone).
- If the skill directory cannot be written at all (a read-only or synced copy), print the confirmed entries in a fenced block with their target file, so they can be added to the super-spec repo by hand (or saved to a file and uploaded with §5 from another machine); still write the lock line in `<project>`.
- In `<project>`, append to `openspec/lessons.lock` (create it with the header from §3): `exported L-<…> (<class>) <YYYY-MM-DD>` — so harvest does not offer the same class again.

## 3. Import

1. Detect platforms (§1). Read every entry of the matching files. Read `<project>/openspec/lessons.lock` if it exists; it looks like:
   ```
   # super-spec lessons lock — which platform lessons this project has imported
   imported L-ANDROID-003 updated=2026-11-20 → review:C12 backlog:"enforce receiver-not-unregistered"
   declined L-ANDROID-004 updated=2026-11-20
   exported L-ANDROID-007 (codec-not-released) 2026-12-02
   ```
2. **Candidates** = entries with `status` active or observe whose id is not in the lock, or whose `updated:` is newer than the date the lock recorded. Skip `retired`. None → say `No new lessons for <platforms>.` and stop.
3. **Show them** as a short table (id · class · status · enforce · evidence count) and ask (AskUserQuestion, multiSelect, up to 4 per question, repeat as needed): which to import. Entries not selected are written to the lock as `declined`, so they are not offered again unless updated.
4. **For each selected entry, install the strongest form the project can take now** — following `flows/enforcement-ladder.md`'s logic, but without new evidence:
   - `enforce` is structure / type / lint / test → append an `[Enforce]` item to `openspec/backlog.md` with the entry's artifact block (adapted to this project's module names), `prove_red_on: lesson L-<…> — none in this project yet` and `then:` as in the ladder. This project has no commit with the mistake yet, so when the item is built, its red proof uses a fixture file that deliberately contains the mistake (`Red proof: fixture:<path> · <command>`, see `templates/design.md`). Also add the interim review check below.
   - `review_check` present → add to `REVIEW.md` (create from `templates/REVIEW-rules.md` if missing): `- [<next free id>] when: <entry's when:, narrowed to this project's paths where obvious> → check: <review_check> (source: lesson L-<…>) · enforced_by: reviewer` — under `## Always check` if the entry is `active` and the user chose "Always" for it, otherwise under `## Observe`. Imported checks do not go through `prompts/rule-skeptic.md` (there is no evidence in this repo yet); they earn promotion like any Observe check.
   - `sensitive_paths` present → append the globs that match at least one path in `git ls-files`; skip the rest.
   - `verify` present and a verify skill exists → append the step to that skill's Doctor or health checks section; no verify skill → add it to the backlog item list as `verify: <step>`.
5. **Write the lock** lines (`imported … → <what was installed>`, `declined …`).
6. **Commit** by delegating to a haiku Agent subagent (`你是被主 session 委派的子代理，可以直接用 Bash 執行 git 命令，不需要再往下委派。`; `git add` only `REVIEW.md` / `openspec/REVIEW.md`, `openspec/backlog.md`, `openspec/lessons.lock` and the verify skill folder, whichever changed; no hook bypass; stderr on failure): `chore(lessons): import <k> platform lessons (<platforms>)`. When called from pre-flight, this lands before the planning commit, so it is not part of the change.

Report: `Imported <k>: <ids> → <n> review checks (<a> Always, <o> Observe), <b> backlog items, <s> sensitive paths, <v> verify steps. Declined <d>.`

## 4. Harvest

Batch version of record, for a project that has used super-spec for a while.

1. Collect candidate classes from `<project>`:
   - `class:` of every case in `openspec/review-evals/*.md`;
   - `{class: …}` lines in `openspec/changes/archive/*/review*.md` and `**Fixed during review:**` lines in `openspec/changes/archive/*/tasks.md`, counted;
   - `REVIEW.md` checks whose `source:` is an escape or a class (not `lesson …` — those came from lessons);
   - `## Enforced elsewhere` lines (classes the project already made impossible — the strongest lessons).
2. Drop `verification-failure`, `unclassified`, `run-failure`, classes already `exported` in the lock, and classes that already exist in the platform files with this project counted.
3. Show the rest with counts and where they came from; ask (multiSelect) which to record. Run §2 steps 1–5 for each selected class (one privacy confirmation per entry), then §2's commit / lock steps once.

## 5. Push — upload local lessons to the super-spec repo

Uploads the lessons on this machine to the GitHub repo super-spec was installed from, merged with what other machines already uploaded. It only ever changes `lessons/` in the repo. Works whether `<skill>` is a git clone or a plain copy.

1. **Find the repo.** `git -C <skill> remote get-url origin` if `<skill>` is a git tree; otherwise read `<skill>/lessons/.upstream` (one line: the repo URL); otherwise ask the user for the URL (free text) and, if `<skill>` is writable, save it to `<skill>/lessons/.upstream`. Default branch: `git ls-remote --symref <url> HEAD` (the `ref: refs/heads/<branch>` line).
2. **Fresh copy of the remote.** `TMP=$(mktemp -d)` and `git clone --depth 1 --branch <branch> <url> "$TMP/super-spec"`. All merging and the commit happen there — never in `<skill>` — so unrelated local commits or edits in `<skill>` are never pushed. Clone fails (auth, network) → report stderr and stop.
3. **Compare entries per platform file.** For each `<skill>/lessons/*.md` (skip `README.md` and `.upstream`), split local and remote files into entries at `### L-` headings and match them:
   | Local entry | Remote | Action |
   |---|---|---|
   | id not in remote, class not in remote | — | **new**: append |
   | id not in remote, same class already in remote under another id | written on another machine | **merge** into the remote entry: add the project count if it is a different project (ask if unsure), keep the stronger `enforce`, `proved_red: yes` wins, `status` per README rules, `updated:` = later date; drop the local id |
   | same id, same class, content differs | — | **update**: field by field, the side with the later `updated:` wins; evidence counts are combined as in merge |
   | same id, different class | id collision | **renumber** the local entry to the next free id in the remote file |
   | same id, identical | — | nothing |
   Remote entries that are not local are kept untouched (they came from other machines). A local platform file that does not exist remotely is copied over, and its row is added to the table in `lessons/README.md` if missing.
4. **Privacy re-check.** Scan every new / merged / updated entry for things the README forbids: application ids that are not generic examples (`[a-z]+\.[a-z0-9_]+\.[a-z0-9_.]+` outside code that names an SDK API), hostnames and URLs other than public docs, e-mail addresses, absolute paths (`/Users/`, `/home/`, `C:\`), and anything that looks like a key or token. List each hit with the entry id.
5. **Show the plan and confirm.** One table: id · platform · action (new / merge into L-… / update / renumber L-… → L-…) · class, plus the privacy hits. Ask (AskUserQuestion): **Upload** / **Fix first** (stop; the user edits the entries, then runs push again) / **Cancel**. Privacy hits default the recommendation to **Fix first**. Nothing is pushed without **Upload**.
6. **Commit and push from the fresh copy.** Write the merged files, then in `$TMP/super-spec`: `git add lessons/`, commit `lessons(<platforms>): <n> new, <m> merged/updated` (add the session's commit attribution lines if it has any), `git push origin HEAD:<branch>`. If the push is rejected because the remote moved, fetch again and redo steps 3–6 once; a second rejection → stop and report. Push refused for permissions → report it as is.
7. **Renumbered ids.** For each `renumber L-a → L-b`: rewrite `exported L-a` → `exported L-b` in `<project>/openspec/lessons.lock` if this run started inside a project (commit that one-line change with the haiku subagent rules from §3 step 6: `chore(lessons): renumber exported lesson ids`). Say plainly that other projects which exported or imported `L-a` from this machine still carry the old id.
8. **Bring `<skill>` up to date.** If `<skill>` is a git tree: `git -C <skill> fetch origin <branch>`. If every local commit ahead of `origin/<branch>` touches only `lessons/` (`git -C <skill> log origin/<branch>..HEAD --name-only`) and `git -C <skill> status --porcelain` is empty, ask once: **Update local copy to match GitHub** (`git -C <skill> reset --keep origin/<branch>` — the local lesson commits are now upstream in merged form) / **Leave it**. Any other local commits → do not touch them; say the local clone has unrelated commits and needs a manual `git pull --rebase`. If `<skill>` is not a git tree, copy the merged `lessons/*.md` back into `<skill>/lessons/` when it is writable.
9. **Clean up** `rm -rf "$TMP"`.

Report:
```
lessons push: <url> <branch> → <commit hash>
new: <ids> · merged: <local → remote ids> · updated: <ids> · renumbered: <a → b>
privacy hits fixed before upload: <k | none>
local copy: updated | left as is (<reason>)
```

## Rules

- This flow never changes product code.
- Writing to `<skill>/lessons/` happens only in record / harvest (and push step 8), only after the user confirms each entry's text.
- Uploading happens only through §5, only after the user picks **Upload**; it never pushes anything outside `lessons/`.
- `lessons/` is never read during the six phases — only here and in pre-flight §7. Once a lesson is imported, the project's own `REVIEW.md` / backlog / verify skill carry it; the run does not consult `lessons/` again.
