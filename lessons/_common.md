---
platform: _common
detect: always
---

# Lessons — any project (platform-independent)

<!-- Entries use the format in lessons/README.md. Ids: L-COMMON-001, L-COMMON-002, … (never reused).
     Written by `/super-spec lessons record` / `harvest`; read by `/super-spec lessons import`. -->

### L-COMMON-001 · partial-copy-no-rollback
- status: observe
- updated: 2026-11-28
- when: code writing or replacing several output files as one set (`os.replace`, `copyfile`, `copy2`, `rename` in a loop)
- symptom: After an error partway (file locked by a viewer, disk full), the output folder holds a mix of new and old files, or an old file is gone with no backup.
- root_cause: Each file is replaced on its own; there is no all-or-nothing step and the backups are deleted on the error path.
- enforce: test — inject an OSError on the n-th replace and assert every file is either all-new or all-old, temps are gone, and a backup that could not be restored is kept.
- review_check: a step that replaces several files as one set restores already-replaced files from backups on any error, removes temps, and never deletes a backup it could not restore.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×5 · proved_red: no

### L-COMMON-002 · untracked-required-asset
- status: observe
- updated: 2026-11-28
- when: .gitignore, build scripts, packaging config (`--add-data`, resource folders), tests reading resource files
- symptom: The app, its build or its tests work on the author's machine but fail on a fresh clone because a required resource file is missing.
- root_cause: A broad ignore pattern (e.g. `*.png`) excludes files the code or the build script needs, so they were never committed.
- enforce: test — a test asserts that every file the build bundles or the code loads at start-up is listed by `git ls-files`.
- review_check: every file the code reads from its resource folder, or the build checks or bundles, is tracked and not excluded by .gitignore.
- sensitive_paths: none
- verify: build and run the tests from a fresh clone.
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×3 · proved_red: no

### L-COMMON-003 · atomic-write-temp-names
- status: observe
- updated: 2026-11-28
- when: code using `os.replace` / rename with a temp file, `tempfile.mkstemp`, names ending in `.tmp`
- symptom: Two writers (two instances, or a write plus a backup copy) overwrite each other's temp file; or, once temp names are random, files left by a crash pile up forever.
- root_cause: A fixed `<name>.tmp` is shared by every writer; random temp names fix that but nothing ever deletes the ones a crash left behind.
- enforce: test — use unique temp names (mkstemp with a recognisable prefix) and, at start-up, delete only files matching this program's exact temp-name shape that are older than a safe age; test that recent and foreign files are kept.
- review_check: atomic writes use a unique temp name, and leftovers of this program's own temp names are cleaned by exact pattern and age, never by a broad glob.
- sensitive_paths: none
- verify: none
- evidence: 1 project(s) · first seen 2026-11 · recurring class ×2 · proved_red: no
